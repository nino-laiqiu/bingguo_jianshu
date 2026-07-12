##1.  CUBE的构建
#####1.新建项目
由顶部菜单栏进入 Model 页面，然后点击 Manage Projects。
![新建项目](https://upload-images.jianshu.io/upload_images/9049859-4e1d8a9d1f0d87c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.同步Hive表
在顶部菜单栏点击 Model，然后点击左边的 Data Source 标签，它会列出所有加载进 Kylin 的表，点击 Load Table 按钮。
#####3.新建 Data Model
创建 cube 前，需定义一个数据模型。数据模型定义了一个星型（star schema）或雪花（snowflake schema）模型。一个模型可以被多个 cube 使用。

![新建 Data Model](https://upload-images.jianshu.io/upload_images/9049859-99571e865af63ca1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
【可选】点击 Add Lookup Table 按钮添加一个 lookup 表。选择表名和关联类型（内连接或左连接）
![Add Lookup Table](https://upload-images.jianshu.io/upload_images/9049859-1d4ac8e7871555fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击 New Join Condition 按钮，左边选择事实表的外键，右边选择 lookup 表的主键。如果有多于一个 join 列重复执行。
![New Join Condition](https://upload-images.jianshu.io/upload_images/9049859-5d661646a9f2b3a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Dimensions 页面允许选择在子 cube 中用作维度的列，然后点击 Columns 列，在下拉框中选择需要的列。
![Dimensions](https://upload-images.jianshu.io/upload_images/9049859-a94bd00c62df53f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击 “Next” 到达 “Measures” 页面，选择作为 measure 的列，其只能从事实表中选择。
![Measures](https://upload-images.jianshu.io/upload_images/9049859-b1f2442e07076f7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**点击 “Next” 到达 “Settings” 页面，如果事实表中的数据每日增长，选择 Partition Date Column 中相应的 日期列以及日期格式，否则就将其留白。**

【可选】选择是否需要 “time of the day” 列，默认情况下为 No。如果选择 Yes, 选择 Partition Time Column 中相应的 time 列以及 time 格式
![Partition Time Column](https://upload-images.jianshu.io/upload_images/9049859-6093f6e1f147951b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
【可选】如果在从 hive 抽取数据时候想做一些筛选，可以在 Filter 中输入筛选条件。
#####4.新建 Cube
**Cube 信息**
cube 名字可以使用字母，数字和下划线（空格不允许）。Notification Email List 是运用来通知job执行成功或失败情况的邮箱列表。Notification Events 是触发事件的状态。
![cube](https://upload-images.jianshu.io/upload_images/9049859-2c1a20c3adcf0882.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**维度**
点击 Add Dimension，在弹窗中显示的事实表和 lookup 表里勾选输入需要的列。Lookup 表的列有2个选项：“Normal” 和 “Derived”（默认）。“Normal” 添加一个普通独立的维度列，“Derived” 添加一个 derived 维度，derived 维度不会计算入 cube，将由事实表的外键推算出。
![Add Dimension](https://upload-images.jianshu.io/upload_images/9049859-3bd4a94dd4ccd688.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**度量**
点击 +Measure 按钮添加一个新的度量。
**根据它的表达式共有8种不同类型的度量：SUM、MAX、MIN、COUNT、COUNT_DISTINCT TOP_N, EXTENDED_COLUMN 和 PERCENTILE。请合理选择 COUNT_DISTINCT 和 TOP_N 返回类型，它与 cube 的大小相关。**

DISTINCT_COUNT
这个度量有两个实现：
1）近似实现 HyperLogLog，选择可接受的错误率，低错误率需要更多存储；
2）精确实现 bitmap（具体限制请看 https://issues.apache.org/jira/browse/KYLIN-1186）

TOP_N
TopN 度量在每个维度结合时预计算，它比未预计算的在查询时间上性能更好；需要两个参数：一是被用来作为 Top 记录的度量列，Kylin 将计算它的 SUM 值并做倒序排列；二是 literal ID，代表最 Top 的记录，例如 seller_id；

**更新设置**
Auto Merge Thresholds: 自动合并小的 segments 到中等甚至更大的 segment。如果不想自动合并，删除默认2个选项。

Volatile Range: 默认为0，会自动合并所有可能的 cube segments，或者用 ‘Auto Merge’ 将不会合并最新的 [Volatile Range] 天的 cube segments。

Retention Threshold: 只会保存 cube 过去几天的 segment，旧的 segment 将会自动从头部删除；0表示不启用这个功能。

Partition Start Date: cube 的开始日期
![更新设置*](https://upload-images.jianshu.io/upload_images/9049859-cef0914fc47f3764.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**高级设置**
**Aggregation Groups**: Cube 中的维度可以划分到多个聚合组中。默认 kylin 会把所有维度放在一个聚合组，当维度较多时，产生的组合数可能是巨大的，会造成 Cube 爆炸；如果你很好的了解你的查询模式，那么你可以创建多个聚合组。在每个聚合组内，使用 “Mandatory Dimensions”, “Hierarchy Dimensions” 和 “Joint Dimensions” 来进一步优化维度组合。

**Mandatory Dimensions**: 必要维度，用于总是出现的维度。例如，如果你的查询中总是会带有 “ORDER_DATE” 做为 group by 或 过滤条件, 那么它可以被声明为必要维度。这样一来，所有不含此维度的 cuboid 就可以被跳过计算。

**Hierarchy Dimensions**: 层级维度，例如 “国家” -> “省” -> “市” 是一个层级；不符合此层级关系的 cuboid 可以被跳过计算，例如 [“省”], [“市”]. 定义层级维度时，将父级别维度放在子维度的左边。

**Joint Dimensions**:联合维度，有些维度往往一起出现，或者它们的基数非常接近（有1:1映射关系）。例如 “user_id” 和 “email”。把多个维度定义为组合关系后，所有不符合此关系的 cuboids 会被跳过计算。

**Rowkeys**: 是由维度编码值组成。”Dictionary” （字典）是默认的编码方式; 字典只能处理中低基数（少于一千万）的维度；如果维度基数很高（如大于1千万), 选择 “false” 然后为维度输入合适的长度，通常是那列的最大长度值; 如果超过最大值，会被截断。请注意，如果没有字典编码，cube 的大小可能会非常大。

你可以拖拽维度列去调整其在 rowkey 中位置; 位于rowkey前面的列，将可以用来大幅缩小查询的范围。通常建议将 mandantory 维度放在开头, 然后是在过滤 ( where 条件)中起到很大作用的维度；如果多个列都会被用于过滤，**将高基数的维度（如 user_id）放在低基数的维度（如 age）的前面。**

**Mandatory Cuboids: 维度组合白名单。确保你想要构建的 cuboid 能被构建。**

Cube Engine: cube 构建引擎。有两种：MapReduce 和 Spark。如果你的 cube 只有简单度量（SUM, MIN, MAX)，建议使用 Spark。**如果 cube 中有复杂类型度量（COUNT DISTINCT, TOP_N），建议使用 MapReduce。**

Advanced Dictionaries: “Global Dictionary” 是用于精确计算**COUNT DISTINCT**的字典, 它会将一个非 integer的值转成 integer，以便于 bitmap 进行去重。如果你要计算 COUNT DISTINCT 的列本身已经是 integer 类型，那么不需要定义 Global Dictionary。 Global Dictionary 会被所有 segment 共享，因此支持在跨 segments 之间做上卷去重操作。请注意，Global Dictionary 随着数据的加载，可能会不断变大。

“Segment Dictionary” 是另一个用于精确计算 COUNT DISTINCT 的字典，与 Global Dictionary 不同的是，它是基于一个 segment 的值构建的，因此不支持跨 segments 的汇总计算。如果你的 cube 不是分区的或者能保证你的所有 SQL 按照 partition_column 进行 group by, 那么你应该使用 “Segment Dictionary” 而不是 “Global Dictionary”，这样可以避免单个字典过大的问题。

请注意：”Global Dictionary” 和 “Segment Dictionary” 都是单向编码的字典，仅用于 COUNT DISTINCT 计算(将非 integer 类型转成 integer 用于 bitmap计算)，他们不支持解码，因此不能为普通维度编码。

Advanced Snapshot Table: 为全局 lookup 表而设计，提供不同的存储类型。

Advanced ColumnFamily: 如果有超过一个的COUNT DISTINCT 或 TopN 度量, 你可以将它们放在更多列簇中，以优化与HBase 的I/O。

**重写配置**
![重写配置](https://upload-images.jianshu.io/upload_images/9049859-54a6e39f687300af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![示例](https://upload-images.jianshu.io/upload_images/9049859-3ba230e74fa98d19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##2.  CUBE构建的需求评估
cube的构建是解决特定的业务问题的需要评估业务需求需要的表和当前cube是否满足,基于这样的流程,需要设计事件流来规范我们的开发流程,大致如下

评估当前的业务需求
评审需求的解决方案
接口是否满足本次需求(是否能够复用接口)
确定该需求设计的hive表,评估本次需求的底层表是否需要开发
1. 需要开发(底层表)
评估是否依赖于外部数据
**继续确定底层表是否是新增字段还是开发新的底层表**
梳理底层的逻辑
2. 无需开发
确定现有的cube是否支持(评估cube面向的业务问题)
如果支持,梳理cube设计的维度和度量信息,开发SQL,对接上游使用方
**如果不支持,梳理此次需求所涉及的维度和度量,确定当前的底层表是否load,即model是否创建,继续基于当前的load表,评估cube面向的业务问题以及创建cube,继续调试cube与优化**



