##逐层算法(Layer Cubing)
我们知道，一个N维的Cube，是由1个N维子立方体、N个(N-1)维子立方体、N*(N-1)/2个(N-2)维子立方体、......、N个1维子立方体和1个0维子立方体构成，总共有2^N个子立方体组成，在逐层算法中，按维度数逐层减少来计算，每个层级的计算（除了第一层，它是从原始数据聚合而来），是基于它上一层级的结果来计算的。

比如，[Group by A, B]的结果，可以基于[Group by A, B, C]的结果，通过去掉C后聚合得来的；这样可以减少重复计算；当 0维度Cuboid计算出来的时候，整个Cube的计算也就完成了。

**此算法的Mapper和Reducer都比较简单。Mapper以上一层Cuboid的结果（Key-Value对）作为输入。由于Key是由各维度值拼接在一起，从其中找出要聚合的维度，去掉它的值成新的Key，并对Value进行操作，然后把新Key和Value输出，进而Hadoop MapReduce对所有新Key进行排序、洗牌（shuffle）、再送到Reducer处；Reducer的输入会是一组有相同Key的Value集合，对这些Value做聚合计算，再结合Key输出就完成了一轮计算。**

每一轮的计算都是一个MapReduce任务，且串行执行； 一个N维的Cube，至少需要N次MapReduce Job。

此算法充分利用了MapReduce的能力，处理了中间复杂的排序和洗牌工作，故而算法代码清晰简单，易于维护；

受益于Hadoop的日趋成熟，此算法对集群要求低，运行稳定；在内部维护Kylin的过程中，很少遇到在这几步出错的情况；即便是在Hadoop集群比较繁忙的时候，任务也能完成。

当Cube有比较多维度的时候，所需要的MapReduce任务也相应增加；由于Hadoop的任务调度需要耗费额外资源，特别是集群较庞大的时候，反复递交任务造成的额外开销会相当可观；

由于**Mapper不做预聚合**，此算法会对Hadoop MapReduce输出较多数据; 虽然已经使用了Combiner来减少从Mapper端到Reducer端的数据传输，所有数据依然需要通过Hadoop MapReduce来排序和组合才能被聚合，无形之中增加了集群的压力;

**对HDFS的读写操作较多**：由于每一层计算的输出会用做下一层计算的输入，这些Key-Value需要写到HDFS上；当所有计算都完成后，Kylin还需要额外的一轮任务将这些文件转成HBase的HFile格式，以导入到HBase中去；

总体而言，该算法的效率较低，尤其是当Cube维度数较大的时候

##快速Cube算法(Fast Cubing)
“逐段”(By Segment) 或“逐块”(By Split) 算法

该算法的主要思想是，**对Mapper所分配的数据块，将它计算成一个完整的小Cube 段（包含所有Cuboid）；**每个Mapper将计算完的Cube段输出给Reducer做合并，生成大Cube，也就是最终结果.新算法的核心思想是清晰简单的，就是最大化利用Mapper端的CPU和内存，对分配的数据块，将需要的组合全都做计算后再输出给Reducer；由Reducer再做一次合并（merge），从而计算出完整数据的所有组合。如此，经过一轮Map-Reduce就完成了以前需要N轮的Cube计算

在Mapper内部， 也可以有一些优化，下图是一个典型的四维Cube的生成树；第一步会计算**Base Cuboid（所有维度都有的组合）**，再基于它计算减少一个维度的组合。基于parent节点计算child节点，可以重用之前的计算结果；当计算child节点时，需要parent节点的值尽可能留在内存中； 如果child节点还有child，那么递归向下，所以它是一个深度优先遍历。当有一个节点没有child，或者它的所有child都已经计算完，这时候它就可以被输出，占用的内存就可以释放。

![优化](https://upload-images.jianshu.io/upload_images/9049859-65f0c63ba871d684.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Fast Cubing的优点：

总的IO量比以前大大减少。 
此算法可以脱离Map-Reduce而对数据做Cube计算，故可以很容易地在其它场景或框架下执行，例如Streaming 和Spark。

Fast Cubing的缺点：

代码比以前复杂了很多： 由于要做多层的聚合，并且引入**多线程机制**，同时还要估算JVM可用内存，当内存不足时需要将数据暂存到磁盘，所有这些都增加复杂度。
对Hadoop资源要求较高，用户应尽可能在Mapper上多分配内存；如果内存很小，该算法需要频繁借助磁盘，性能优势就会较弱。在极端情况下（如数据量很大同时维度很多），任务可能会由于超时等原因失败；

##总结
如果每个Mapper之间的**key交叉重合度较低**，fast cubing更适合；因为Mapper端将这块数据最终要计算的结果都达到了，Reducer只需少量的聚合。另一个极端是，每个Mapper计算出的key跟其它 Mapper算出的**key深度重合**，这意味着在reducer端仍需将各个Mapper的数据抓取来再次聚合计算；如果key的数量巨大，该过程IO开销依然显著。对于这种情况，Layered-Cubing更适合

Kylin在计算Cube之前对数据进行采样，在“fact distinct”步，**利用HyperLogLog模拟去重**，估算每种组合有多少不同的key，从而计算出每个Mapper输出的数据大小，以及所有Mapper之间数据的重合度，据此来决定采用哪种算法更优。在对上百个Cube任务的时间做统计分析后，Kylin选择了7做为默认的算法选择阀值(参数kylin.cube.algorithm.layer-or-inmem-threshold)：如果各个Mapper的小Cube的行数之和，大于reduce后的Cube行数的7倍，采用Layered Cubing, 反之采用Fast Cubing。如果用户在使用过程中，更倾向于使用Fast Cubing，可以适当调大此参数值，反之调小。

1、如果每个Mapper之间的key交叉重合度较低，fast cubing更适合；因为Mapper端将这块数据最终要计算的结果都达到了，Reducer只需少量的聚合。另一个极端是，每个Mapper计算出的key跟其它 Mapper算出的key深度重合，这意味着在reducer端仍需将各个Mapper的数据抓取来再次聚合计算；如果key的数量巨大，该过程IO开销依然显著。对于这种情况，Layered-Cubing更适合。

2、在对上百个Cube任务的时间做统计分析后，Kylin选择了7做为默认的算法选择阀值(参数kylin.cube.algorithm.auto.threshold)：如果各个Mapper的小Cube的行数之和，大于reduce后的Cube行数的8倍(各个Mapper的小Cube的行数之和 / reduce后的Cube行数 > 7)，采用Layered Cubing, 反之采用Fast Cubing(本质就是各个Mapper之间的key重复度越小，就用Fast Cubing，重复度越大，就用Layered Cubing)


