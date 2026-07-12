Apache Kylin有着卓越的可扩展架构。总体架构上的三大依赖——**数据源、计算引擎和存储引擎都有清晰的接口**，保证了ApacheKylin可以方便地接入最新的数据源或切换计算存储技术，从而**跟随大数据生态圈一起演进**。
##1. 可扩展式架构
#####可扩展架构工作原理
![Kylin可扩展架构](https://upload-images.jianshu.io/upload_images/9049859-a59feb2e37ed2b45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
每一个Cube都可以设定自己的数据源、计算引擎和存储引擎，这些设定信息均保存在Cube元数据中。在构建Cube时，首先由工厂类创建数据源、计算引擎和存储引擎对象。这三个对象均独立创建，相互之间没有关联
![工厂类创建数据源和引擎对象](https://upload-images.jianshu.io/upload_images/9049859-6f6f9ce7c73a4cb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
要把它们串联起来需要使用适配器设计模式。计算引擎好比一块主板，主控整个Cube的构建过程。它以数据源为输入，以存储为Cube的输出，因此定义了IN和OUT两个接口。
![数据源和储存引擎适配IN/OUT接口](https://upload-images.jianshu.io/upload_images/9049859-4115a7d297b2e349.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####三大主要接口
**数据源接口**
```
public interface ISource extends Closeable {
 // 返回一个ISourceMetadataExplorer，用来获取数据源中的元数据信息
 ISourceMetadataExplorer getSourceMetadataExplorer();
 // 适配指定的构建引擎接口。返回一个对象，实现指定的IN接口。
 <I> I adaptToBuildEngine(Class<I> engineInterface);
 // 返回一个ReadableTable，用来顺序读取一个表。
 IReadableTable createReadableTable(TableDesc tableDesc, String uuid);
 // 返回一个SourcePartition，用来记录要构建的数据范围
 SourcePartition enrichSourcePartitionBeforeBuild(IBuildable buildable, SourcePartition srcPartition);
 // 返回一个ISampleDataDeployer，用来将示例数据部署到数据源中
 ISampleDataDeployer getSampleDataDeployer();
 // 卸载已加载到项目中的表
 void unloadTable(String tableName, String project) throws IOException;
}
```
1. getSourceMetadataExplorer：返回一个ISourceMetadataExplorer，用来读取数据源中数据库、表等的元数据信息。
2. adaptToBuildEngine：适配指定的构建引擎接口。返回一个对象，实现指定的IN接口。该接口主要由计算引擎调用，要求数据源向计算引擎适配。如果数据源无法提供指定接口的实现，则适配失败，Cube构建将无法进行。
3. createReadableTable：返回一个ReadableTable，用来顺序读取一个表。除了构建引擎之外，有时查询引擎也需要顺序访问数据维表的内容，用来创建维度字典或维表快照，因而查询引擎也会调用此方法。
4. enrichSourcePartitionBeforeBuild：返回一个SourcePartition，用来记录要构建的数据范围。
5. getSampleDataDeployer：返回一个ISampleDataDeployer，用来将Apache Kylin
的示例数据部署到数据源中，以便用户能够快速入门。·unloadTable：卸载已加载到项目中的数据源表。
6. unloadTable：卸载已加载到项目中的数据源表。

**存储引擎接口**
```
public interface IStorage {
 // 适配指定的构建引擎接口。返回一个对象，实现指定的OUT接口。
 public <I> I adaptToBuildEngine(Class<I> engineInterface);
 // 创建一个查询对象IStorageQuery，用来查询给定的IRealization。
 public IStorageQuery createQuery(IRealization realization);
}
```
1. adaptToBuildEngine：适配指定的构建引擎接口。返回一个对象，实现指定的OUT接口。该接口主要由计算引擎调用，要求存储引擎向计算引擎适配。如果存储引擎无法提供指定接口的实现，则适配失败，Cube构建将无法进行。
2. createQuery：创建一个查询对象IStorageQuery，用来查询给定的IRealization。简单来说就是返回一个能够查询指定Cube的对象。IRealization是在Cube之上的一个抽象。其主要的实现类就是Cube，此外还有叫作Hybrid的虚拟实现（使用多个Cube来共同完成一个查询）

**计算引擎接口**
目前的计算引擎在设计上都是采用批处理，因此叫作IBatchCubingEngine。流式处理是另一类数据处理模式，在实时计算和实时分析中有广泛应用
```
public interface IBatchCubingEngine {
 //返回一个IJoinedFlatTableDesc，记录构建指定的CubeSegment过程中要用到的平表结构信息。
 public IJoinedFlatTableDesc getJoinedFlatTableDesc(CubeSegment newSegment);
 // 返回一个工作流计划，用以构建指定的CubeSegment。
 public DefaultChainedExecutable createBatchCubingJob(CubeSegment newSegment,
String submitter);
 // 返回一个工作流计划，用以合并指定的CubeSegment。
 public DefaultChainedExecutable createBatchMergeJob(CubeSegmentmergeSegment,
String submitter);
 // 返回一个工作流计划，用以优化指定的CubeSegment。
 public DefaultChainedExecutable createBatchOptimizeJob(CubeSegment
optimizeSegment, String submitter);
 // 指明该计算引擎的IN接口。
 public Class<?> getSourceInterface();
 // 指明该计算引擎的OUT接口。
 public Class<?> getStorageInterface();
}
```
1. getJoinedFlatTableDesc：返回一个IJoinedFlatTableDesc，记录构建指定的CubeSegment过程中要用到的平表结构信息。构建第一阶段，会把事实表和维表连接为一张大表，也称为平表，这里返回的IJoinedFlatTableDesc就定义记录着这个平表的结构。
2. createBatchCubingJob：返回一个工作流计划，用以构建指定的CubeSegment。这里的CubeSegment是一个刚完成初始化，但还不包含数据的CubeSegment。返回的DefaultChainedExecutable是一个工作流的描述对象。它将被保存并由工作流引擎在稍后调度执行，完成Cube构建。
3. createBatchMergeJob：返回一个工作流计划，用以合并指定的CubeSegment。这里的CubeSegment是一个待合并的CubeSegment，它的区间横跨多个已有的CubeSegment。返回的工作流计划一样会在稍后被调度执行，执行的过程中将多个现有的CubeSegment合并为一个，以降低Cube的碎片化程度。
4. createBatchOptimizeJob：返回一个工作流计划，用以优化指定CubeSegment。这里的CubeSegment是一个已被构建好的待优化的CubeSegment。返回的工作流计划一样
会在稍后被调度执行。
5. getSourceInterface：指明该计算引擎的IN接口。
6. getStorageInterface：指明该计算引擎的OUT接口。

**描述**
1. Rest API接受到执行（构建/合并）CubeSegment的请求。
2. EngineFactory根据Cube元数据定义，创建IBatchCubingEngine对象，并调用其上的createBatchCubingJob（或createBatchMergeJob、createBatchOptimizeJob）方法。
3. IBatchCubingEngine根据Cube元数据定义，通过SourceFactory和StorageFactory创建出相应的数据源ISource和存储IStorage对象。
4. IBatchCubingEngine调用ISource上的adaptToBuildEngine方法，传入IN接口，要求数据源向自己适配
5. IBatchCubingEngine调用IStorage上的adaptToBuildEngine方法，传入OUT接口，要求存储引擎向自己适配。
6. 适配成功后，计算引擎协同数据源和存储引擎计划Cube构建的具体步骤，将结果以工作流（DefaultChainedExecutable）的形式返回。
7. 执行引擎将在稍后执行工作流，完成Cube构建。

##2. 计算引擎扩展
#####EngineFactory
每一个构建引擎必须实现接口IBatchCubingEngine，并在EngineFactory中注册实现类。只有这样才能在Cube元数据中引用该引擎，否则会在构建Cube时出现“No realization found”（中文含义是“找不到实现”）的错误
注册的方法是通过配置$KYLIN_HOME/conf/kylin.properties在其中添加一行构建引擎的申明
```
kylin.job.engine.2=org.apache.kylin.engine.mr.MRBatchCubingEngine2
kylin.job.engine.4=org.apache.kylin.engine.spark.SparkBatchCubingEngine2
```
#####MRBatchCubingEngine2
```
public class MRBatchCubingEngine2 implements IBatchCubingEngine {
 //返回一个IJoinedFlatTableDesc，记录构建指定的CubeSegment过程中要用到的平表结构信息。
 public IJoinedFlatTableDesc getJoinedFlatTableDesc(CubeSegment newSegment) {
 return new CubeJoinedFlatTableDesc(newSegment);
 }
 // 返回一个工作流计划，用以构建指定的CubeSegment。
 public DefaultChainedExecutable createBatchCubingJob(CubeSegment
newSegment, String submitter) {
 return new BatchCubingJobBuilder2(newSegment, submitter).build();
 }
 // 返回一个工作流计划，用以合并指定的CubeSegment。
 public DefaultChainedExecutable createBatchMergeJob(CubeSegment
mergeSegment, String submitter) {
 return new BatchMergeJobBuilder2(mergeSegment, submitter).build();
 }
 // 返回一个工作流计划，用以优化指定的CubeSegment。
 public DefaultChainedExecutable createBatchOptimizeJob(CubeSegment
optimizeSegment, String submitter) {
 return new BatchOptimizeJobBuilder2(optimizeSegment, submitter).build();
 }
 // 指明该计算引擎的IN接口。
 public Class<?> getSourceInterface() {
 return IMRInput.class;
 }
 // 指明该计算引擎的OUT接口。
 public Class<?> getStorageInterface() {
 return IMROutput2.class;
 }
}
```
######BatchCubingJobBuilder2
```
public class BatchCubingJobBuilder2 extends JobBuilderSupport {
 ......
 private final IMRBatchCubingInputSide inputSide;
 private final IMRBatchCubingOutputSide2 outputSide;
 ......
 public CubingJob build() {
 logger.info("MR_V2 new job to BUILD segment " + seg);
 final CubingJob result = CubingJob.createBuildJob(seg, submitter, config);
 final String jobId = result.getId();
 final String cuboidRootPath = getCuboidRootPath(jobId);
 // Phase 1: Create Flat Table & Materialize Hive View in Lookup Tables
 inputSide.addStepPhase1_CreateFlatTable(result);
 // Phase 2: Build Dictionary
 result.addTask(createFactDistinctColumnsStep(jobId));
 if (isEnableUHCDictStep()) {
 result.addTask(createBuildUHCDictStep(jobId));
 }
 result.addTask(createBuildDictionaryStep(jobId));
 result.addTask(createSaveStatisticsStep(jobId));
 // add materialize lookup tables if needed
 LookupMaterializeContext lookupMaterializeContext = addMaterializeLookupTableSteps(result);
 outputSide.addStepPhase2_BuildDictionary(result);
 if (seg.getCubeDesc().isShrunkenDictFromGlobalEnabled()) {
 result.addTask(createExtractDictionaryFromGlobalJob(jobId));
 }
 // Phase 3: Build Cube
 addLayerCubingSteps(result, jobId, cuboidRootPath); // layer cubing,
only selected algorithm will execute
 addInMemCubingSteps(result, jobId, cuboidRootPath); // inmem cubing,
only selected algorithm will execute
 outputSide.addStepPhase3_BuildCube(result);
 // Phase 4: Update Metadata & Cleanup
 result.addTask(createUpdateCubeInfoAfterBuildStep(jobId,
lookupMaterializeContext));
 inputSide.addStepPhase4_Cleanup(result);
 outputSide.addStepPhase4_Cleanup(result);
 return result;
 }
 ......
}
```
整个构建过程是一个子任务依次串行执行的过程，这些子任务又被分为四个阶段。

1. 第一阶段：创建平表。
这一阶段的主要任务是预计算连接运算符，把事实表和维表连接为一张大表，也称平表。这部分工作通过调用数据源接口来完成，因为数据源一般有现成的计算表连接方法，高效且方便，没有必要在计算引擎中重复实现。
2. 第二阶段：创建字典。
创建字典由多个子任务完成，分别是**抽取列值、为高基数列构建字典、创建字典、保存统计信息、物化维表、从全局字典得到构建要用到的字典。**是否使用字典用户可以在创建Cube的时候进行选择，使用字典的好处是有很好的数据压缩率，可以减少存储空间，同时提升读的速度。缺点是构建字典需要占用较多的内存资源，创建维度基数超过千万的Cube时容易造成内存溢出。虽然可以通过调换外存来解决，但也是以降低速度为代价。如果Cube用到的维度表是视图，会根据这个视图物化一张临时表，之后的构建都会使用与维表视图相对应的临时表。
3. 第三阶段：构建Cube。
第二版MR引擎带有两种构建Cube的算法，分别是分层构建和快速构建。它们对于不同的数据分布各有优劣，区别主要在于**数据通过网络洗牌的策略**。由于网络是大多数Hadoop集群的“瓶颈”，不同的洗牌策略往往决定了构建速度。两种算法的子任务被全部加入工作流计划中，在执行时会根据源数据的统计信息自动选择一种算法，未被选择的算法的子任务会被自动跳过。在构建Cube的最后阶段调用了存储引擎接口，存储引擎负责将计算完的Cube放入存储。
4. 第四阶段：更新元数据和清理临时数据。
最后阶段，Cube已经构建完毕，任务引擎首先添加子任务更新Cube元数据，然后分别调用数据源接口和存储引擎接口对临时数据进行清理。可以看到整个构建过程由构建引擎主导，由它负责调度数据源和存储引擎。除了计算Cube的主要任务是由构建引擎完成的，前期的创建平表和数据导入等操作则是由数据源完成的，Cube保存由存储引擎完成。三者协同，缺一不可。

#####IMRInput
```
public interface IMRInput {
 // 返回一个IMRTableInputFormat对象，用来从数据源读取指定的关系表
 public IMRTableInputFormat getTableInputFormat(TableDesc table);
 // IMRTableInputFormat是一个辅助接口，用于帮助Mapper读取数据源中的一张表
 public interface IMRTableInputFormat {
 // 配置给定MapReduce任务的InputFormat
 public void configureJob(Job job);
 // 解析Mapper的输入对象，返回关系表的一行
 public String[] parseMapperInput(Object mapperInput);
 // 获取输入分片的signature标识
 public String getInputSplitSignature(InputSplit inputSplit);
 }
 ......
}
```
```
public interface IMRInput {
 ......
 // 返回一个辅助对象（接口就在下面），参与创建一个CubeSegment的构建工作流
 public IMRBatchCubingInputSide getBatchCubingInputSide(CubeSegment seg);
 // 本辅助接口代表数据输入端参与创建构建CubeSegment的工作流。
 // 主要负责从数据源提取数据并创建一张临时平表（第一阶段），
 // 然后再工作流的末尾清除这张临时表（第四阶段）。
 public interface IMRBatchCubingInputSide {
 // 返回一个IMRTableInputFormat，帮助MR任务读取之前创建的平表
 public IMRTableInputFormat getFlatTableInputFormat();
 // 由构建引擎调用，要求数据源在工作流中添加步骤完成平表的创建
 public void addStepPhase1_CreateFlatTable(DefaultChainedExecutable jobFlow);
 // 清理收尾，清除已经没用的平表和其他临时对象
 public void addStepPhase4_Cleanup(DefaultChainedExecutable jobFlow);
 }
}
```
#####IMROutput2
```
public interface IMROutput2 {
 // 返回一个IMRBatchCubingOutputSide2对象，参与创建指定CubeSegment的工作流
 public IMRBatchCubingOutputSide2 getBatchCubingOutputSide(CubeSegment seg);
 // 本辅助接口代表数据输出端参与创建构建CubeSegment的工作流。
 //包含四个方法，前三个方法由构建引擎分别在字典创建后、Cube计算完后和清尾阶段调用。
 public interface IMRBatchCubingOutputSide2 {
 // 构建引擎在字典创建后调用，存储引擎可以在这里完成预备存储的初始化工作
 public void addStepPhase2_BuildDictionary(DefaultChainedExecutable jobFlow);
 // 构建引擎在Cube计算完成后调用，存储引擎保存Cube数据
 public void addStepPhase3_BuildCube(DefaultChainedExecutable jobFlow, String cuboidRootPath);
 // 构建引擎在收尾阶段调用，清理存储端的任何垃圾
 public void addStepPhase4_Cleanup(DefaultChainedExecutable jobFlow);
 // 返回IMROutputFormat，在配置MapReduce任务的OutputFormat时被调用。
 public IMROutputFormat getOuputFormat();
 }
}
```
##3. 数据源扩展
实现数据源首先要实现ISource接口
```
public class HiveSource implements ISource {
 @Override
 public <I> I adaptToBuildEngine(Class<I> engineInterface) {
 if (engineInterface == IMRInput.class) {
 return (I) new HiveMRInput();
 } else if (engineInterface == ISparkInput.class) {
 return (I) new HiveSparkInput();
 } else {
 throw new RuntimeException("Cannot adapt to " + engineInterface);
 }
 }
 @Override
 public IReadableTable createReadableTable(TableDesc tableDesc, String uuid) {
 // hive view must have been materialized already
 if (tableDesc.isView()) {
 KylinConfig config = KylinConfig.getInstanceFromEnv();
 String tableName = tableDesc.getMaterializedName(uuid);
 tableDesc = new TableDesc();
 tableDesc.setDatabase(config.getHiveDatabaseForIntermediateTable());
 tableDesc.setName(tableName);
 }
 return new HiveTable(tableDesc);
 }
 @Override
 public SourcePartition enrichSourcePartitionBeforeBuild(IBuildablebuildable,
SourcePartition srcPartition) {
 SourcePartition result = SourcePartition.getCopyOf(srcPartition);
 if (srcPartition.getTSRange() != null) {
 result.setSegRange(null);
 }
 return result;
 }
 @Override
 public ISampleDataDeployer getSampleDataDeployer() {
 return new HiveMetadataExplorer();
 }
 @Override
 public void unloadTable(String tableName, String project) throws IOException {
 }
 @Override
 public void close() throws IOException {
 // not needed
 }
}
```
##4. 存储扩展
```
public class HBaseStorage implements IStorage {
 ......
 @Override
 public <I> I adaptToBuildEngine(Class<I> engineInterface) {
 if (engineInterface == IMROutput2.class) {
 return (I) new HBaseMROutput2Transition();
 } else if (engineInterface == ISparkOutput.class) {
 return (I) new HBaseSparkOutputTransition();
 } else {
 throw new RuntimeException("Cannot adapt to " + engineInterface);
 }
 }
 ......
 @Override
 public IStorageQuery createQuery(IRealization realization) {
 if (realization.getType() == RealizationType.CUBE) {
 ......
 return ret;
 } else {
 throw new IllegalArgumentException("Unknown realization type " +
realization.getType());
 }
 }
}
```
```
public class HBaseMROutput2Transition implements IMROutput2 {
 @Override
 public IMRBatchCubingOutputSide2 getBatchCubingOutputSide(final CubeSegment seg) {
 boolean useSpark = seg.getCubeDesc().getEngineType() == IEngineAware.ID_SPARK;
 // TODO need refactor
 final HBaseJobSteps steps = useSpark ? new HBaseSparkSteps(seg) : new HBaseMRSteps(seg);
 return new IMRBatchCubingOutputSide2() {
 @Override
 public void addStepPhase2_BuildDictionary(DefaultChainedExecutable jobFlow) {
 jobFlow.addTask(steps.createCreateHTableStep(jobFlow.getId()));
 }
 @Override
 public void addStepPhase3_BuildCube(DefaultChainedExecutable jobFlow)
 jobFlow.addTask(steps.createConvertCuboidToHfileStep(jobFlow.getId()));
 jobFlow.addTask(steps.createBulkLoadStep(jobFlow.getId()));
 }
 @Override
 public void addStepPhase4_Cleanup(DefaultChainedExecutable jobFlow) {
 steps.addCubingGarbageCollectionSteps(jobFlow);
 }
 @Override
 public IMROutputFormat getOuputFormat() {
 return new HBaseMROutputFormat();
 }
 };
 }
 ......
}
```
##5. 聚合类型扩展
```
// 类型T是聚合数据的类型
abstract public class MeasureTypeFactory<T> {
 // 创造一个MeasureType实例，依据为指定的聚合函数和聚合数据类型
 abstract public MeasureType<T> createMeasureType(String funcName, DataType dataType);
 // 返回支持的聚合函数，比如"COUNT_DISTINCT"
 abstract public String getAggrFunctionName();
 // 返回支持的聚合数据类型，比如"hllc"
 abstract public String getAggrDataTypeName();
 // 返回聚合数据类型的序列化器，注意序列化器的实现必需线程安全
 abstract public Class<? extends DataTypeSerializer<T>>
getAggrDataTypeSerializer();
 ......
}

```
#####聚合类型的实现（MeasureType）
```
// 类型T是聚合数据的类型
abstract public class MeasureType<T> {
 // 检查用户定义的FunctionDesc是否有效
 public void validate(FunctionDesc functionDesc) throws IllegalArgumentException {
 return;
 }
 // 该聚合数据类型是否需要较大的内存
 public boolean isMemoryHungry() {
 return false;
 }
 // 聚合是否只应用在Base Cuboid上
 public boolean onlyAggrInBaseCuboid() {
 return false;
 }
 ......
}
```
1. MeasureType的泛型参数T代表聚合数据类型。比如，以HyperLogLog为例，它的聚合数据类型是HyperLogLogPlusCounter。
2. validate()方法校验传入的FunctionDesc（度量定义中聚合函数的部分）是否合法。在创建Cube的过程中这个方法会被多次调用，检查用户定义的度量是否正确，比如数据的精度在有效范围内等。如果校验失败，该方法应该抛出IllegalArgumentException。
3. isMemoryHungry()报告该聚合数据在运算时是否需要较多的内存。一些基本的聚合函数比如SUM和COUNT在计算时只需要几个字节，然而类似HyperLogLogPlusCounter的大型数据结构可能一个就需要10KB甚至100KB内存。需要在内存分配上给予特别对待。
4. onlyAggrInBaseCuboid()定义该聚合运算是否只发生在Base Cuboid上。如果是,那么在其他Cuboid上该聚合函数将被跳过。

##6. 维度编码扩展
#####维度编码工厂（DimensionEncodingFactory）
```
public abstract class DimensionEncodingFactory {
 ......
 // 返回所支持的编码名称
 abstract public String getSupportedEncodingName();
 // 返回一个新的维度编码实例
 abstract public DimensionEncoding createDimensionEncoding(String encodingName, String[] args);
}
```
#####维度编码实现（DimensionEncoding）
Apache Kylin对维度编码有以下基本要求：
1）等长。编码后，所有维度值的代码长度相同。
2）双向。编码和原来的值可以双向互换。
3）保序。编码二进制大小与原值的大小保持一致。
```
// 注意维度编码是可以序列化的
public abstract class DimensionEncoding implements Externalizable {
 ......
 // 判断指定的代码是否代表NULL
 public static boolean isNull(byte[] bytes, int offset, int length) {
 ......
 }
 // 获得固定的代码长度
 abstract public int getLengthOfEncoding();
 // 转换给定的维度值（以byte形式表示的字符串）为编码
 abstract public void encode(byte[] value, int valueLen, byte[] output, int outputOffset);
 // 转换给定的编码为维度值（以String形式返回）
 abstract public String decode(byte[] bytes, int offset, int len);
 // 返回一个DataTypeSerializer，以序列化器的接口实现同样的编码解码功能
 abstract public DataTypeSerializer<Object> asDataTypeSerializer();
}
```
