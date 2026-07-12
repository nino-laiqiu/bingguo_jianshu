>内容来源于贝壳DMP实践

##1. 用户分群方式介绍和思路
用户的标签数据一般存储在多张hive表中,在进行用户圈包,会涉及join的逻辑,限制了人群包数据的产出速度.使用标签进行用户分群，其本质还是集合之间的交、并、补运算。如果能够将符合每个标签取值的用户群提都提前构建出来，即构建好标签 - 用户的映射关系，在得到人群包的标签组合后直接选取对应的集合，通过集合之间的交 / 并 / 补运算即可得到最终的目标人群。bitmap 是用于存储标签 - 用户的映射关系的比较理想的数据结构之一。ClickHouse 目前也已经比较稳定的支持了 bitmap 数据结构，为基于 bitmap 的用户分群实现提供了基础。
![流程图](https://upload-images.jianshu.io/upload_images/9049859-7b8f5db0912591a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

整个方案主要包含以下几个技术问题：
1. 如何针对亿级用户构建全局连续唯一数字 ID 标识 join_id？
2. 如何设计 bitmap 生成规则使其适用于 DMP 上所有的画像标签？
3. 如何将 Hive 表中的关系型数据以 bitmap 的形式保存到 ClickHouse 表中？
4. 如何将标签之间的与 / 或 / 非逻辑转化成 bitmap 之间的交 / 并 / 补运算并生成 bitmap SQL？

#####亿级用户构建全局连续唯一数字 ID
hive 提供了基础的 row_number() over() 函数，但是在操作亿级别行的数据时，会造成数据倾斜，受限于 Hadoop 集群单机节点的内存限制，无法成功运行。为此提出了一种针对亿级行大数据量的全局唯一连续数字 ID 生成方法。其核心思想如下：
![核心思想](https://upload-images.jianshu.io/upload_images/9049859-42ea9cd16ca12399.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

具体的做法:由于亿级数据不支持全局row_number()排序,可考虑把一张大表按照一定的规则进行分拆,对每个子表打标签,然后分配id,对于第 1 个子数据集（M = 1）的数据，其最终行号是 1，2，3，4，…，N1；对于第 2 个子数据集（M = 2）的数据，其最终行号是 1 + N1，2 + N1，3 + N1，4 + N1，…，N2 + N1....以此类推

DMP 所有的画像数据最终汇总到了 4 张 Hive 表中，分别保存用户的基本信息（base 表）、偏好信息（prefer 表）、行为信息（action 表）和设备信息（device 表）。构建好 join_id 后，还需要将 join_id 关联到用户画像表中，产出构建 bitmap 所需要的 bitmap_hive 表。到此也就完成了 Hive 数据层的准备工作。
#####Bitmap 的设计
**1. 标签梳理**

..|单一标签|复合标签
--|:--:|--:
枚举类型|枚举类型单一标签|枚举类型复合标签
连续值类型|连续值类型单一标签|连续值类型复合标签
日期类型|日期类型单一标签|日期类型复合标签

**2. bitmap 的构建和运算转换**

具体的见原文档(非公有部分),注意边界值的处理(运算的转化)
这里的处理思想非常有意思 https://cloud.tencent.com/developer/news/683175

**3. Bitmap_CK 表的设计**
bitmap 数据是通过 Spark 任务以序列化的方式写入到 CH 中的，为此我们再 CH 中创建了一个 null 引擎的表，bitmap 的类型为 string。然后以 null 引擎的表为基础创建了一个物化视图表，通过 base64Decode() 函数将 String 类型的 bitmap 转换成 CH 中的 AggregateFunction(groupBitmap, UInt32) 数据结构，最后以物化视图表为物理表，创建分布式表用于数据的查询。同时为了减少 CH 集群的处理压力，我们还进行了一个优化，即在 null 引擎表之前创建了一个 buffer 引擎的表，数据最先写入 buffer 引擎的表，积攒到一定的时间 / 批次后，数据会自动写入到 null 引擎的表。

**4. Hive 的关系型数据到 CH 的 bitmap 数据**
Spark 任务中，先通过 spark SQL 将所需 hive 数据读取，保存在 DataSet<Row> 中。根据不同的标签类型设计的规则使用 spark 聚合算子进行运算。处理逻辑如下：

..|单一标签|复合标签
--|:--:|--:
枚举类型|对于某个标签，读取标签的取值列和用户 ID 列，使用 spark 聚合算子将取值相同的用户 ID 进行聚合，最终对于该标签的每个取值，得到等于该取值的用户 ID 集合。将数据版本、hive 表名、标签名称、标签取值和用户 ID 集合写入 CK。|复合标签是由多个标签组合而成的，其中有且只有一个主要标签，至少包含一个次要标签。对于复合标签中多个标签的顺序问题，约定将这些标签以字典序排列。复合标签的处理和单一标签的处理思路一致，即将主要标签按照单一标签的处理逻辑进行处理，所有标签字段按照字典序排列后作为标签名称，标签值也按照对应的顺序排列后作为标签取值。
连续值类型|对于某个标签，读取标签的取值列和用户 ID 列，对数据进行 flatMap 操作，（如对于取值是 3 的一行，flatMap 后将成为 0、1、2、3 共 4 行数据），对 flatMap 后的数据使用 spark 聚合算子将取值相同的用户 ID 进行聚合，最终对于该标签的每个取值，得到等于该取值的用户 ID 集合。将数据版本、hive 表名、标签名称、标签取值和用户 ID 集合写入 CK。
日期类型|对于某个标签，读取标签的取值列和用户 ID 列，对数据进行 flatMap 操作，（如今天是 2020-04-12，对于取值是 2020-04-10 的一行，flatMap 后将成为 2020-04-12、2020-04-11、2020-04-10 共 3 行数据），对 flatMap 后的数据使用 spark 聚合算子将取值相同的用户 ID 进行聚合，最终对于该标签的每个取值，得到等于该取值的用户 ID 集合。将数据版本、hive 表名、标签名称、标签取值和用户 ID 集合写入 CK。

**在这个过程中，我们还使用了 bitmap 的循环构建、spark 任务调优、异常重试机制、bitmap 构建后的数据验证等方法来提高任务的运行速度和稳定性。**

**5. bitmap SQL 的生成**
通过处理人群包的标签组合，确定所需要的 bitmap 以及这些 bitmap 之间的逻辑关系（下图红线标识），最终生成的 bitmap SQL 示例如下图所示。同时通过使用 GLOBAL IN 代替比较耗时的 GLOBAL ANY INNER JOIN，CH SQL 运行效率也有了大幅度的提升。
对于Push消息类的服务需要通过接口获取人群中的数据用于消息发送。由于ClickHouse定位还是OLAP，不适合大量地在线调用，所以需要将人群的数据导入到Mongodb中来提供在线服务调用。为优化分页查询带来的性能问题，在导入Mongodb时为每个版本的每条数据生成一个自增的ID，同时对这个ID建立索引，在查询时根据页数计算出每一页数据的ID范围，然后再根据索引来查询数据，能保证千万级的分页查询平均响应时间在100ms以内。
![示例](https://upload-images.jianshu.io/upload_images/9049859-b922258cd35585b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一些对于bitmap的优化https://cloud.tencent.com/developer/news/680214
RoaringBitmap论文
 https://arxiv.org/pdf/1402.6407.pdf    https://arxiv.org/pdf/1603.06549.pdf
>拓展文章
https://cloud.tencent.com/developer/news/683175
https://www.infoq.cn/article/D2uR9BB5Fbq49vZJihYi
https://mp.weixin.qq.com/s?__biz=MzIyMTg0OTExOQ==&mid=2247485738&idx=1&sn=71d61dae19c6d6111e25e420207600f9&chksm=e8373a5adf40b34c072a79d6e05d2e1d87c7b78adfcabd30f8b0df09b3ccab79db13d2c4733c&scene=21#wechat_redirect
https://cloud.tencent.com/developer/news/680214
http://mysql.taobao.org/monthly/2018/08/09/
