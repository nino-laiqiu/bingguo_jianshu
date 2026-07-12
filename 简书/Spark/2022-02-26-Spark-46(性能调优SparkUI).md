##Spark UI 一级入口
![一级入口简介](https://upload-images.jianshu.io/upload_images/9049859-a407cb818529091b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Spark UI 一级入口](https://upload-images.jianshu.io/upload_images/9049859-4eb4fbfd49ffaa60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##Executors
Executors Tab 的主要内容如下，主要包含“Summary”和“Executors”两部分。这两部分所记录的度量指标是一致的，其中“Executors”以更细的粒度记录着每一个 Executor 的详情，而第一部分“Summary”是下面所有 Executors 度量指标的简单加和
![Executor Metrics](https://upload-images.jianshu.io/upload_images/9049859-1dcc101fb8ba8156.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对于 Executors 页面中每一个 Metrics 的具体数值，它们实际上是 Tasks 执行指标在 Executors 粒度上的汇总。

##Environment
![Environment Metrics](https://upload-images.jianshu.io/upload_images/9049859-2985c3835c6ed6e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##Storage
Cached Partitions 与 Fraction Cached 分别记录着数据集成功缓存的分区数量，以及这些缓存的分区占所有分区的比例。当 Fraction Cached 小于 100% 的时候，说明分布式数据集并没有完全缓存到内存（或是磁盘），对于这种情况，我们要警惕缓存换入换出可能会带来的性能隐患。
基于 Storage 页面提供的详细信息，我们可以有的放矢地设置与内存有关的配置项，如 spark.executor.memory、spark.memory.fraction、spark.memory.storageFraction，从而有针对性对 Storage Memory 进行调整。
![Storage Metrics](https://upload-images.jianshu.io/upload_images/9049859-614142adbb685bbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##SQL
一级入口页面，以 Actions 为单位，记录着每个 Action 对应的 Spark SQL 执行计划。我们需要点击“Description”列中的超链接，才能进入到二级页面，去了解每个执行计划的详细信息。
#####Exchange
它们的作用是对申请编码数据与中签编码数据做 Shuffle
![Shuffle Metrics](https://upload-images.jianshu.io/upload_images/9049859-ee6af9c1cf8ecc42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####Sort
可以看到，“Peak memory total”和“Spill size total”这两个数值，足以指导我们更有针对性地去设置 spark.executor.memory、spark.memory.fraction、spark.memory.storageFraction，从而使得 Execution Memory 区域得到充分的保障。
![Sort Metrics](https://upload-images.jianshu.io/upload_images/9049859-11d52a1be7569fd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####Aggregate
对于 Aggregate 操作，Spark UI 也记录着磁盘溢出与峰值消耗，即 Spill size 和 Peak memory total。这两个数值也为内存的调整提供了依据
##Jobs
对于 Jobs 页面来说，Spark UI 也是以 Actions 为粒度，记录着每个 Action 对应作业的执行情况。我们想要了解作业详情，也必须通过“Description”页面提供的二级入口链接。

##Stages
tages 页面，更多地是一种预览，要想查看每一个 Stage 的详情，同样需要从“Description”进入 Stage 详情页

#####Event Timeline
Event Timeline，记录着分布式任务调度与执行的过程中，不同计算环节主要的时间花销。图中的每一个条带，都代表着一个分布式任务，条带由不同的颜色构成。其中不同颜色的矩形，代表不同环节的计算时间
![不同环节的计算时间](https://upload-images.jianshu.io/upload_images/9049859-75297c08c115c995.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####Summary Metrics
对于这些详尽的 Task Metrics，难能可贵地，Spark UI 以最大最小（max、min）以及分位点（25% 分位、50% 分位、75% 分位）的方式，提供了不同 Metrics 的统计分布。这一点非常重要，原因在于，这些 Metrics 的统计分布，可以让我们非常清晰地量化任务的负载分布。
![不同环节的计算时间](https://upload-images.jianshu.io/upload_images/9049859-ed3320ebee469389.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Spill（Memory）和 Spill（Disk）这两个指标。Spill，也即溢出数据，它指的是因内存数据结构（PartitionedPairBuffer、AppendOnlyMap，等等）空间受限，而腾挪出去的数据。Spill（Memory）表示的是，这部分数据在内存中的存储大小，而 Spill（Disk）表示的是，这些数据在磁盘中的大小。因此，用 Spill（Memory）除以 Spill（Disk），就可以得到“数据膨胀系数”的近似值，我们把它记为 Explosion ratio。有了 Explosion ratio，对于一份存储在磁盘中的数据，我们就可以估算它在内存中的存储大小，从而准确地把握数据的内存消耗。

#####Tasks
![Tasks度量指标](https://upload-images.jianshu.io/upload_images/9049859-b5b84e9fd80d72f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
