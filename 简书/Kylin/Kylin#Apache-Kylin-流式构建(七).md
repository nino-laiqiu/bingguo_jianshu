##1.为什么要进行流式构建
实时数据更新是一种普遍的需求，快速更新的数据分析能够帮助分析师快速地判断业务的变化趋势，从而在风险仍然可控的阶段做出决策。在监控领域，通常需要非常实时的数据更新来抓捕异常的数据特征，这样一来，对数据的延迟需求可能是秒级甚至是毫秒级别的。
##2.准备流式数据
#####数据格式
Kylin假设在流式构建中数据以消息流的形式传递给流式构建引擎。消息流中的每条消息都需要包含以下内容：
1. 维度信息。
2. 度量信息。
3. 业务时间戳。

在消息流中，每条消息的数据结构都应该相同，并且可以用同一个分析器实例将每条消息中的维度、度量及时间戳信息提取出来。目前默认的分析器为
org.apache.kylin.source.kafka.TimedJsonStreamParser，该分析器会假设每条消息为一个单独的JSON，所有的信息以键值对形式保存在该JSON之中。它还假设键值对中存在一个特殊的键值代表消息的业务时间，我们将该键值称为业务时间戳。该键值的键名是可配的，其值为长整数的Unix Time时间类型（时间是自1970-01-0100：00：00开始的毫秒数，也称Unix Epoch time）。

业务时间戳的粒度对于在线多维分析而言可能过高，无法在时间维度上完成深度的聚合。因此，Kylin允许用户挑选一些从业务时间戳上衍生出来的时间维度（Derived Time Dimension）

#####消息队列
流式构建的用户需要使用Kafka的Producer将数据源源不断地加入某个Topic中，并且将Kafka的一些基本信息（如Broker节点信息和Topic名称）告知流式构建任务。流式构建任务在启动时会启动Kafka客户端，然后根据配置向Kafka集群读取相应的Topic中的消息，并进行预处理计算。
#####创建Schema
由于Kylin对查询客户端暴露的是ANSI SQL接口，因此用户最终将以SQL接口来查询流式构建的数据。对于全量构建和增量构建，它们的源数据是Hive中的某些表，因此Kylin可以从Hive中导入这些表的Schema，用户查询的时候也可以像直接查询Hive表一样使用相应的SQL语法。但是进行流式构建的问题是，数据是以键值对或文本的形式传入消息队列的，并不像Hive一样存在可以用来导入的表定义。但是由于消息队列中的键值对是基本固定的，甚至包括衍生时间维度一经选择也已经是固定的，因此可以创造出一个虚拟的表，用表中的各个列来对应消息队列中的维度、度量及衍生时间维度。在查询时，用户会直接对这张虚拟的表发起各种SQL查询，就好像这张表真实存在一样。

##3.设计流式Cube
1. 在创建Model对话框的设置中，一般选择最小粒度的衍生时间维度作为分区时间列，这样可减少Segment之间的时间重叠（对于流式构建，因为Segment之间使用Kafka offset做物理分区，所以时间范围允许有重叠）
2. 设置Cube的自动合并时间。因为流式构建需要频繁地构建较小的Segment，为了不给存储器造成太大压力，也为了获取较好的查询性能，需要通过自动合并将已有的多个小Segment合并成一个较大的Segment。所以，这里设置一个层级的自动合并时间：0.5小时、4小时、1天、7天、28天。此外，由于在很多流式场景中，用户只
关心过去一段时间的热数据，因此可以设置保留时间，如30天，这样30天之前的Segment数据就可以被Kylin自动舍弃
3. Aggregation Groups设置中，可以把衍生时间维度设置为Hierarchy关系，减少非必要计算

##4.流式构建原理
1. Kylin对于Hive表的构建，会启动一个hive命令，创建一张临时外表，然后将需要的字段和时间范围的数据，抽取到特定的HDFS目录下。同样地，对于Kafka数据源，Kylin也可以启动一个在YARN上的任务，将Kafka topic中特定范围的数据以期望的格式（默认为sequence文件格式）写到特定的HDFS目录中。考虑到Kafka topic通常分为多个分区（partition），因此**Kylin可以启动多个mapper，每个mapper消费一个partition上对应的消息区间，并行地读取数据，从而加快处理速度**
2. 解决了流数据的并行消费问题后，Kylin还需解决如何界定segment之间的数据界限问题。通常情况下，如果要按照时间范围进行处理的话，会额外读取一段时间范围的数据（称为margin），以尽可能补全数据然而，即便有margin的存在，也不能完全避免在极限情况下个别消息被遗漏，导致统计上的误差。Kylin的Segment划分进行重构，让它可以**支持非时间类型的切分**。在Kafka中，虽然消息不是严格按时间顺序增长的，但是每个消息都有唯一的offset值（long类型），这个值是消费消息的主要索引，是只增且不可修改的。考虑到一个topic虽然可以有多个partition，但每个partition的offset是独立的，那么**Kylin可以将一次消费的各个partition的起点offset加和，作为此Segment的起点符号，将此次消费的各partition的终点offset加和，作为此Segment的终点符号。**KYLIN在创建模型时选择的时间分区列已经成为逻辑上的索引，在流式Cube中，Kylin允许两个segment之间的时间范围有重叠（当然offset不能重叠），用于在查询时快速定位segment

![按offset切分的segment](https://upload-images.jianshu.io/upload_images/9049859-daec5be99648440c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 第三个问题是如何解决Kafka消息和Hive表的join需求。针对这种情况，为了简单，把Kafka消息写到HDFS后，Kylin将它描述成一张临时的Hive表，然后使用HiveQL将它与同在Hive中的维度表进行join操作，类似于对普通Hive数据源的处理。
![kafka数据源和hive数据源的join](https://upload-images.jianshu.io/upload_images/9049859-268e4567705866d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##5. 触发流式构建
将Kafka topic用作数据源，第一次触发Cube构建的时候，Kylin会自动从Kafka集群获取该topic最早的offset和当下最新的offset，随后启动任务消费这个范围内的数据。当触发下次构建时，**Kylin会从上次构建的结束位置开始，消费到当下的最新offset。**因此，用户几乎不用关心构建的范围，就仿佛点击一个按钮，告诉Kylin“请帮我把截至目前的新消息构建进Cube”一样。用户只需要掌握触发构建的时间点即可，如是每1小时构建一次，还是每15分钟构建一次。此外，构建的频率也是可以随时调整的，如在白天业务人员需要及时看到数据的时候，每15分钟构建一次，在夜晚和周末的时候降至小时级别。
#####单次触发构建
消费的Kafka消息的offset范围，包括每个partition的起始offset、结束offset等，都会被自动记录在segment的元数据中

#####自动化多次触发
如果流式构建频繁，比如每五分钟构建一次，那么一天就要构建数百次，这样一来，每次都进行手动触发显然就不切实际了。由于单次触发可以通过API的方式执行，因此用户可以结合自己喜欢的调度工具来完成自动化的触发。随着5分钟构建一次的Segment不断堆积，自动合并也会被触发，**Kylin会使用增量构建中的合并构建把小Segment陆续合并成大Segment，以保证查询性能。**对于合并操作，Kylin使用与增量构建中相同的方式进行合并，不再依赖于特殊的流式构建引擎。

#####初始化构建起点
如果用户想从某个特定offset点开始构建，也可以通过调用Rest API指定每个partition的offset起点来完成。考虑到Kafka offset对用户透明，一般这样的场景并不多。如果用户的Kafka上暂存的数据量很大，不想进行全量构建，那么可以使用另一个API把Kylin构建的起点从topic的最早时间点移动到当下时间点
#####其他操作
流式构建中，往往前一个segment还没有构建完成，后一个segment就已经开始构建了；如果前一个构建任务失败，并且因为种种原因被用户舍弃，那么它对应的segment就会被舍弃，从而在连续的segment中间会留下一个没有数据的offset区间，我们将这个没有数据的offset区间称为空洞。为了方便用户填上数据空洞，Kylin提供了额外的REST API来检查空洞和补洞

检查空洞API的运行脚本如下：
```
curl -X GET --user ADMIN:KYLIN -H "Content-Type: application/json;charset=utf-8"
http://localhost:7070/kylin/api/cubes/{your_cube_name}/holes
```
如果此API返回为空，代表没有空洞，否则返回一个segment数组，每个segment代表一个数据空洞
补洞API的运行脚本如下：
```
curl -X PUT --user ADMIN:KYLIN -H "Content-Type: application/json;charset=utf-8"
http://localhost:7070/kylin/api/cubes/{your_cube_name}/holes
```
此API会自动为每个数据空洞触发一个构建任务，返回任务数组；如果没有数据空洞，则返回为空。

#####出错处理
1. Kafka版本不匹配：从Kylin 2.5版本以后，要求Kafka版本为1.0.0或以上；
2. Kafka broker配置有误：在配置数据源表的时候，用户需要输入Kafka broker信息，包括broker机器名、端口等；需确保此机器名和端口是能够从Hadoop集群的各个节点访问到Kafka broker的，否则会获取数据失败。
3. 如果自动合并构建出错，会导致新产生的Segment迅速堆积，Cube的查询性能也会下降。因此，每当合并构建出错时，管理员需要及时查看合并失败的原因，排除故障并在Web GUI中恢复该合并构建。合并构建的失败往往与流式构建本身没有直接的关系，因为合并不是流式构建引擎的专有功能。出错的原因往往和增量构建一样，问题出在Hadoop集群本身。


补充一个kylin一个注意事项,在求%占比的时候,在kylin不要使用下面这个,kylin对这个支持不够好
```
concat(cast(p * 1.0/sum(p) over() as varchar),'%')
```
应该使用下面这个
```
cast(p * 1.0/sum(p) over() as decimal(9,4) )
```
