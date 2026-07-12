##1. Cube Planner
#####为什么要引入Cube Planner
#####Cube Planner算法介绍
Cuboid的成本主要包含如下两个方面。
构建成本：取决于该维度组合的数据行数。
查询成本：取决于查询该Cuboid需要扫描的行数。

**贪心算法**
贪心算法使用多轮迭代，每次选出当前状态下最优的一种维度组合并将其加入推荐列表。
![维度组合示例](https://upload-images.jianshu.io/upload_images/9049859-a0fc2b3ca6dd8ca2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![层级关系示例](https://upload-images.jianshu.io/upload_images/9049859-299136be1f92d05d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一步，因为维度组合a是Base Cuboid，默认一定会被预计算，所以将a加入推荐列表。在当前只有a被计算的情况下，查询剩余所有维度组合的数据都只能通过它们的祖先a，成本均为100（扫描a所需的行数）。
第二步，估算b的收益比。如果预计算b，那么对b和其子孙d、e、g、h这5种维度组合的查询都可以通过扫描50行的b Cuboid来得到。相比直接扫描a，查询成本从100减至50，因此预计算b对整个Cube的增益为(100–50)*5，而效益比就是增益再除以b的行数：(100–50)*5/50=5。
![估计物化b的收益](https://upload-images.jianshu.io/upload_images/9049859-8d49fb8d579f0ef3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第三步，按第二步的方法依次计算c、d、e、f、g、h的效益比
![收益比](https://upload-images.jianshu.io/upload_images/9049859-f4e775b713e1cc45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可见g是效益比最高的维度组合，因此第一轮将维度组合g加入推荐列表中。
![第一轮迭代](https://upload-images.jianshu.io/upload_images/9049859-9f7f5b6bfd9af437.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第四步，重复第二、三步，每轮迭代都加入一种维度组合。

**基因算法**
基因算法的核心思想是交叉变异，优胜劣汰，主要步骤包括“选择”→“交叉”→“变异”。假设每个Cuboid都是一个基因，一种Cuboid组合是一个染色体，染色体用01字符串表示，其中1表示该Cuboid在这个Cube里出现，0表示不出现。例如：Cuboid Set{O1，O2，O3，O4，O5，O6，O7，O8，O9，O10}，染色体{0110011001}表示这个Cube包含了{O2，O3，O6，O7，O10}这5个Cuboid。每个染色体的Fitness计算方法与贪心算法类似。

算法步骤如下：
第一步，初始化种群
随机生成一些01字符串作为初始种群，代表一些可能的Cuboid组合。
第二步，选择（selection）
采用轮盘赌选择算法（roulette wheel selection）挑选出两个染色体，某个染色体被选中的概率和它的Fitness成正比。
第三步，交叉（crossover）
两个父染色体以一定的规则交换部分基因，生成新的染色体。
第四步，变异（mutation）
在交叉得到的结果中以一定比例随机选择某一位进行反转，并将结果加入种群。

#####使用Cube Planner
Cube Planner会根据优化前的维度组合数量来选择是使用贪心算法还是基因算法。默认情况下，有223以上个维度组合的Cube使用基因算法进行优化；维度组合数量为28～223个的Cube使用贪心算法进行优化，维度组合数量少于28=256的则不做优化。上述阈值由“kylin.cube.cubeplanner.algorithm-threshold-greedy”和“kylin.cube.cubeplanner.algorithm-threshold-genetic”这两个参数共同决定。

##2. System Cube
为了更好地监控Kylin的运行，Apache Kylin自v2.3.0版本引入了System Cube来收集运行时的各种指标数据。System Cube中存储着各种指标数据，通过这些指标我们可以了解每个查询服务器的使用情况，每个项目的查询总次数、查询失败率、查询时延等查询信息，配合Dashboard就可以实时掌握Kylin的查询服务情况。
##3. 仪表盘
为了更好地管理项目，分析人员一定需要了解Cube使用过程中的各项指标，如某个Cube每一天的查询数量、平均查询延迟及每GB源数据的平均构建时间等。所有这些重要的Cube使用数据都可以在Kylin仪表盘（Dashboard）中找到。

....待补充细节4.15日完成
