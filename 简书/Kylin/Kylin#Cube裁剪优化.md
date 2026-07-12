  随着维度数目的增加，Cuboid 的数量会爆炸式地增长。为了缓解 Cube 的构建压力，Apache Kylin 引入了一系列的高级设置，帮助用户筛选出真正需要的 Cuboid。**这些高级设置包括聚合组（Aggregation Group）、联合维度（Joint Dimension）、层级维度（Hierachy Dimension）和强制维度（Mandatory Dimension）**

##1. 聚合组（Aggregation Group）
用户根据自己关注的维度组合，可以划分出自己关注的组合大类，这些大类在 Apache Kylin 里面被称为聚合组。如下中展示的 Cube，如果用户仅仅关注维度 AB 组合和维度 CD 组合，那么该 Cube 则可以被分化成两个聚合组，分别是聚合组 AB 和聚合组 CD。如下图生成的 Cuboid 数目从 16 个缩减成了 8 个。
![示例一](https://upload-images.jianshu.io/upload_images/9049859-44116fc5e5688f7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


用户关心的聚合组之间可能包含相同的维度，例如聚合组 ABC 和聚合组 BCD 都包含维度 B 和维度 C。这些聚合组之间会衍生出相同的 Cuboid，例如聚合组 ABC 会产生 Cuboid BC，聚合组 BCD 也会产生 Cuboid BC。这些 Cuboid不会被重复生成，一份 Cuboid 为这些聚合组所共有如下
![示例二](https://upload-images.jianshu.io/upload_images/9049859-5658568087931d98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. 联合维度（Joint Dimension）
用户有时并不关心维度之间各种细节的组合方式，例如用户的查询语句中如果会出现则仅仅会出现 group by A, B, C，而不会出现 group by A, B 或者 group by C 等等这些细化的维度组合。这一类问题就是联合维度所解决的问题。例如将维度 A、B 和 C 定义为联合维度，Apache Kylin 就仅仅会构建 Cuboid ABC，而 Cuboid AB、BC、A 等等Cuboid 都不会被生成。最终的 Cube 结果如下所示，Cuboid 数目从 16 减少到 4 [2^(4-3+1)=4]。
![示例三](https://upload-images.jianshu.io/upload_images/9049859-d7d1bdfe38f18498.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**联合维度：将几个维度视为一个维度。**

适用场景：

               1 可以将确定在查询时一定会同时使用的几个维度设为一个联合维度(ABC 都是一起出现的，当做一个维度，也可以不出现)。

               2 可以将基数很小的几个维度设为一个联合维度。

               3 可以将查询时很少使用的几个维度设为一个联合维度。

**优化效果：将N个维度中x个维度设置为关联维度，则这N个维度组合成的cuboid个数会从2 ^ N次方减少到2^(N-X+1)**

##3. 层级维度（Hierarchy Dimension）
用户选择的维度中常常会出现具有层级关系的维度。例如对于国家（country）、省份（province）和城市（city）这三个维度，从上而下来说国家／省份／城市之间分别是一对多的关系。也就是说，用户对于这三个维度的查询可以归类为以下三类:
```
group by country

group by country, province（等同于group by province）

group by country, province, city（等同于 group by country, city 或者group by city）
```
层次维度：具有一定层次关系的维度。

      使用场景：像年，月，日；国家，省份，城市这类具有层次关系的维度。

      优化效果：将N个维度中X个维度设置为层级维度，则cuboid个数减少到 (X+1)*2^(N-X)

![示例四](https://upload-images.jianshu.io/upload_images/9049859-08896f2d338fdcda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

例如，Cuboid [A,C,D]=Cuboid[A, B, C, D]，Cuboid[B, D]=Cuboid[A, B, D]，因而 Cuboid[A, C, D] 和 Cuboid[B, D] 就不必重复存储
![示例五](https://upload-images.jianshu.io/upload_images/9049859-f4b082f59e5531cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4. 强制维度 （Mandatory Dimension）
用户有时会对某一个或几个维度特别感兴趣，所有的查询请求中都存在group by这个维度，那么这个维度就被称为强制维度，只有包含此维度的Cuboid会被生成

强制维度：所有cuboid必须包含的维度

      适用场景：可以将确定在查询时一定会使用的维度设为强制维度。例如，时间维度。

      

      优化效果：将一个维度设为强制维度，则cuboid个数直接减半, 减少到2^(N-1)，如果将X个维度设为强制维度，则cuboid个数减少到2^(N-X)。

![示例六-A](https://upload-images.jianshu.io/upload_images/9049859-d7ba32ccd9dbf31d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据本系列的原理介绍，在Kylin的高级设置中，用户可以根据查询需求对Cube构建预计算的结果进行优化（剪枝），从而减少占用的存储空间。 而优化得当的Cube可以在占用尽量少的存储空间的同时提供极强的查询性能

##5. Cube、Cuboid 和 Cube Segment
1. Cube (或Data Cube)，即数据立方体，是一种常用于数据分析与索引的技术；它可以对原始数据建立多维度索引。通过 Cube 对数据进行分析，可以大大加快数据的查询效率
2. Cuboid 在 Kylin 中特指在某一种维度组合下所计算的数据。
3. Cube Segment 是指针对源数据中的某一片段，计算出来的 Cube 数据。通常数据仓库中的数据数量会随着时间的增长而增长，而 Cube Segment 也是按时间顺序来构建的。
