##1. 为什么要增量构建
全量构建可以看作增量构建的一种特例：在全量构建中，Cube中只存在唯一的一个Segment，该Segment没有分割时间的概念，因此也就没有起始时间和结束时间。全量构建和增量构建各有适用的场景，用户可以根据自己的业务场景灵活切换
![增量构建和全量构建的比对](https://upload-images.jianshu.io/upload_images/9049859-d6739ab354ec1b44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(注意更新俩字的含义,kylin是不保存历史状态的,如果是增量更新就是更新指定的时间区间)
整体来说，增量构建的Cube上的查询会比全量构建Cube上的查询做更多的运行时聚合，而这些运行时聚合都发生在单点的查询引擎上，因此，通常来说，增量构建的Cube上的查询会比全量构建的Cube上的查询慢一些。

对于小数据量的Cube或经常需要全表更新的Cube，使用全量构建可以以少量的重复计算降低生产环境中的维护复杂度，减少运维精力。而对于大数据量的Cube，如一个包含两年历史数据的Cube，如果使用全量构建每天更新数据，那么每天为了新数据而重复计算过去两年的数据会严重增加查询成本，在这种情况下需要考虑使用增量构建。
(这里对每日新增的描述,是每日新增,但是对于拉链表来说也是每日新增,那么应该如何选择??)

##2. 设计增量Cube
#####设计增量Cube的条件
并非所有的Cube都适合进行增量构建，Cube的定义必须包含一个时间维度，用来分割不同的Segment，我们将这样的维度称为**分割时间列（Partition Date Column）**。尽管由于历史原因该命名中存在“date”字样，但是分割时间列可以是Hive中的Date类型、Timestamp类型还可以是String类型。无论是哪种类型，Kylin都要求用户显式地指定分割时间列的数据格式。

满足了设计增量Cube的条件之后，在进行增量构建时将增量部分的起始时间和结束时间作为增量构建请求的一部分提交给Kylin的任务引擎，任务引擎会根据起始时间和结束时间从Hive中抽取相应时间的数据，并对这部分数据做预计算处理，然后将预计算的结果封装成新的Segment，保存相应的信息到元数据和存储引擎中。一般来说，增量部分的起始时间等于Cube中最后一个Segment的结束时间。
(对于动态分区表来说,我们并不遵守这规范)
#####增量Cube的创建
1. Model层面的设置
2. Cube层面的设置

##3. 触发增量构建
#####Web GUI触发
在Web GUI上触发Cube的增量构建与触发全量构建的方式基本相同。在Web GUI的Model页面中，选中想要增量构建的Cube，点击“Action”→“Build”命令

#####构建相关的REST API
http://kylin.apache.org/docs/howto/howto_build_cube_with_restapi.html
http://kylin.apache.org/docs/howto/howto_use_restapi.html

##4. 管理Cube碎片
增量构建的Cube每天都可能还有新的增量，这样的Cube日积月累最终可能包含上百个Segment，导致运行时的查询引擎需要聚合多个Segment的结果才能返回正确的查询结果，最终会使查询性能受到严重的影响。从存储引擎的角度来说，大量的Segment会带来大量的文件，这些文件会充斥系统为Kylin所提供的命名空间，给存储空间的多个模块带来巨大的压力，如Zookeeper、HDFS Namenode等。在这种情况下，我们有必要采取措施控制Cube中Segment的数量，如可以自动/手动合并Segment、清理老旧无用的Segment。


#####合并Segment

Kylin提供了一种简单的机制来控制Cube中Segment的数量——合并Segment。在Web GUI中选择需要进行Segment合并的Cube，点击“Action→Merge”命令，然后在对话框中选择需要合并的Segment，可以同时合并多个Segment，但是这些Segment必须是连续的。

#####自动合并
在Cube Designer的“Refresh Settings”的页面中有“Auto Merge Thresholds”“Volatile Range”和“RetentionThreshold”三个设置项可以用来帮助管理Segment碎片。虽然这三项设置还不能完美地解决所有业务场景的需要，但是灵活地搭配使用这三项设置可以大大减少对Segment进行管理的工作量

“Auto Merge Thresholds”允许用户设置几个层级的时间阈值，层级越靠后，时间阈值越大。举例来说，用户可以为一个Cube指定层级（7天、28天），每当Cube中有新的Segment状态变为“READY”的时候，会触发一次系统自动合并的尝试。系统首先会尝试最大一级的时间阈值，结合上面例子中设置的层级（7天、28天），系统会查看是否能把连续的若干个Segment合并成一个超过28天的较大Segment，在挑选连续Segment的过程中，如果有的Segment本身的时间长度已经超过28天，那么系统会跳过该Segment，从它之后的Segment中挑选连续的累计超过28天的Segment。当没有满足条件的连续Segment能够累计超过28天时，系统会使用下一个层级的时间阈值重复此寻找的过程。每当有满足条件的连续Segment被找到，系统就会触发一次自动合并Segment的构建任务，在构建任务完成后，新的Segment被设置为“READY”状态，自动合并的整个尝试过程则需要重新执行。

#####保留Segment
从碎片管理的角度来说，自动合并是将多个Segment合并为一个Segment，以达到清理碎片的目的。保留Segment从另一个角度帮助实现了碎片管理，那就是及时清理不再使用的Segment。在许多业务场景中只会对过去一段时间内的数据进行查询，如对于某个只显示过去1年数据的报表，支持它的Cube，事实上，只需要保留过去一年的Segment。由于数据往往在Hive中已经备份，因此无须再在Kylin中备份超过一年的历史数据。

在这种情况下，我们可以将“Retention Threshold”设置为“365”。每当有新的Segment状态变为“READY”的时候，系统会检查每一个Segment：如果它的结束时间距离最晚的一个Segment的结束时间已经大于“Retention Threshold”，那么这个Segment将被视为无须保留，且系统会自动从Cube中删除这个Segment。

如果启用了“Auto Merge Thresholds”，使用“RetentionThreshold”的时候需要注意，不能将“Auto Merge Thresholds”的最大层级设置得太高。假设我们将“Auto Merge Thresholds”的最大一级设置为“1000”天，而“Retention Threshold”为“365”天，那么受自动合并的影响，新加入的Segment会不断地被自动合并到一个越来越大的Segment之中，更糟糕的是，这会不断地更新这个大Segment的结束时间，最终导致这个大Segment永远不会被释放。因此，**推荐自动合并的最大一级的时间不要超过1年**。
（这个自动合并是怎么做到的???）

#####数据持续更新
在实际应用场景中，我们常常遇到这样的问题：由于ETL过程的延迟，业务需要每天刷新过去N天的Cube数据。举例来说，客户有一个报表每天都需要更新，但是每天源数据的更新不仅包含当天的新数据，还包括了过去7天数据的补充。一种比较简单的方法是，每天在Cube中增量构建一个长度为一天的Segment，这样过去7天的数据会以7个Segment的形式存在于Cube之中。Cube的管理员除了每天创建一个新的Segment代表当天的新数据（BUILD操作）以外，还需要对代表过去7天的7个Segment进行刷新（REFRESH操作，Web GUI上的操作及REST API参数与BUILD类似，这里不再详细展开）。这样的方法固然有一定的作用，但是每天为每个Cube触发的构建数量太多，容易造成Kylin的任务队列堆积大量未能完成的任务

上述方案的另一个弊端是每天一个Segment会使Cube中迅速累积大量的Segment，需要Cube管理员手动地对超过7天的Segment进行合并，期间还必须避免将7天的Segment一起合并了。举例来说，假设现在有100个Segment，每个Segment代表过去的一天的数据，Segment按照起始时间排序。在合并时，我们只能挑选前面93个Segment进行合并，如果不小心把第94个Segment也一起合并了，当我们试图刷新过去7天（94-100）的Segment时，会发现为了刷新第94天的数据，不得不将1~93的数据一并重新进行计算，因为此时第94天的数据已经和1～93这93天的数据糅合在一个Segment之中了，这是一种极大的资源浪费。更糟糕的是，即使使用之前介绍的自动合并的功能，类似的问题也仍然存在,但在Kylin v2.3.0及之后的版本中，增加了一种机制能够阻止自动合并试图合并近期N天的Segment，那就是“Volatile Range”。

“Volatile Range”的设置非常简单，在Cube Designer的“Refresh Setting”中，在“Volatile Range”后面的文本框中输入最近的不想要合并的Segment的天数。举个例子，如果设置“VolatileRange”为“2”，“Auto Merge Thresholds”为“7”，现有01-01到01-08的数据即8个Segment，自动合并不会被开启，直到01-09的数据进来，才会触发自动合并，将01-01到01-07的数据合并为一个Segment。
(不是很明白,假设我有一个cube我每天都更新最近60天的数据,那么我应该如何设置合并)


