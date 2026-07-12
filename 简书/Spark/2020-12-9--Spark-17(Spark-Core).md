>算子分析与性能
shuffleRDD的学习



对RDD操作本质上是对RDD每个分区的操作;
一个分区就是一个迭代器,task对分区的迭代器进行操作

##1.map和filter
```
object MapAndFilterDemo {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("mapfliter").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
    val map: MapPartitionsRDD[Double, Int] = new MapPartitionsRDD[Double,Int](rdd, (_, _, iter) => iter.map(data => {
      data.toString.toDouble * 10
    }))
    val filter = new MapPartitionsRDD[Double, Double](map, (_, _, iter) => iter.filter(data => data > 4))
    filter.collect().foreach(println)
    sc.stop()
  }
}
```
##2.map和mappartition
>( f: (TaskContext, Int, Iterator[T]) => Iterator[U],)---iter
( new MapPartitionsRDD[U, T](this, (_, _, iter) => iter.map(cleanF)))
从从迭代器中拿去数据,调用一次map算子,每个分区一条数据执行一次map操作)
```
object MapDemo {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("mapfliter").setMaster("local[*]"))
    val rdd = sc.makeRDD(List[Int](1, 2, 3, 4, 5, 6, 7, 8, 9, 10), 3)
    //map算子每个数据都要调用一次效率低下
    rdd.map(data => {
      val parid = TaskContext.get().partitionId()
      val stid = TaskContext.get().stageId()
      (parid, stid, data * 10)
    }).saveAsTextFile("maptest")
    sc.stop()
  }
}
```
> (_: TaskContext, _: Int, iter: Iterator[T]) => cleanedF(iter)
对于mappartition方法,是将数据以一个分区为单位取出,一个分区就是一个迭代器,虽然后续仍要对一个个数据进行操作,但这个是对数据的操作,不同于map算子,map算子是将数据取出,例如如下一共打印了3次,不同于map方法有多少数据就打印多少次
**每条数据数据处理一次,但是会反复使用被创建起来的连接,处理完成关闭连接返回迭代器但是对于map,没一条数据就会使用一个连接,使用连接关闭连接**
```
object MapPartitionDemo {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("mapfliter").setMaster("local[*]"))
    val rdd = sc.makeRDD(List[Int](1, 2, 3, 4, 5, 6, 7, 8, 9, 10), 3)
    rdd.mapPartitions(it => {
      println("******")
      val parid = TaskContext.getPartitionId()
      //这里的map方法是对迭代器中每个数据进行操作
      val tuples: Iterator[(Int, Int)] = it.map(data => {
        (parid, data * 10)
      })
      tuples
    }).saveAsTextFile("mappartition")
  }
}
```
##3.groupbykey和 groupby

>(groupby虽然比groupbykey灵活但是在shuffle的时候却要传输更多的数据,groupby是把全部的数据都放在了集合中,而对于groupbykey只是把value值放在了集合中)

```
object GroupByKeyDemo {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("groupbykey").setMaster("local[*]"))
    val maprdd = sc.makeRDD(List("spark", "scala", "java", "js", "flink",
      "spark", "scala", "java", "js", "flink",
      "spark", "scala", "java", "js", "flink")).map(data => (data, 1))
    maprdd.groupByKey()
    val shufflerdd = new ShuffledRDD[String, Int, CompactBuffer[Int]](maprdd, new HashPartitioner(maprdd.partitions.length))
    //设置局部不聚合
    shufflerdd.setMapSideCombine(false)
    //将每个组内的元素的value值放到集合中
    val createCombiner = (v: Int) => CompactBuffer(v)
    //把组内其他元素添加到集合中
    val mergeValue = (buf: CompactBuffer[Int], v: Int) => buf += v
    //在下游进行合并
    val mergeCombiners = (c1: CompactBuffer[Int], c2: CompactBuffer[Int]) => c1 ++= c2
    val groupbukeyrdd = shufflerdd.setAggregator(new Aggregator[String, Int, CompactBuffer[Int]](
      createCombiner, mergeValue, mergeCombiners
    ))
    groupbukeyrdd.collect().foreach(println)
    sc.stop()
  }
}
```
```
//groupkey把要分组的元素当做key,原来的整个元素当成value值(网络传输效率低)
 this.map(t => (cleanF(t), t)).groupByKey(p)
```

##4.reducebykey
```
object ReducebukeyDemo {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("groupbykey").setMaster("local[*]"))
    val maprdd = sc.makeRDD(List("spark", "scala", "java", "js", "flink",
      "spark", "scala", "java", "js", "flink",
      "spark", "scala", "java", "js", "flink"),2).map(data => (data, 1))

    val createCombiner = (x: Int) => x
    val mergeValue = (x: Int, y: Int) => x + y
    val mergeCombiners = (x1: Int, x2: Int) => x1 + x2
   maprdd.combineByKey(
      (x => x), (x: Int, y: Int) => x + y, (x1: Int, x2: Int) => x1 + x2
    ).saveAsTextFile("combinebykey")
    //使用shuffleRDD
    val shufflerdd = new ShuffledRDD[String, Int, Int](maprdd, new HashPartitioner(maprdd.partitions.length))
    //是否局部聚合
    shufflerdd.setMapSideCombine(true)
    shufflerdd.setAggregator(new Aggregator[String, Int, Int](
      createCombiner, mergeValue, mergeCombiners
    )).saveAsTextFile("shufflerdd")
    sc.stop()
  }
}
```

##5.join
```
object JoinDemo {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("groupbykey").setMaster("local[*]"))
    val rdd1 = sc.makeRDD(List(("spark", 1), ("scala", 2), ("kafka", 4), ("flink", 9), ("spark", 1)))
    val rdd2 = sc.makeRDD(List(("spark", 3), ("scala", 4), ("kafka", 5), ("strom", 2), ("strom", 4)))
    //cogroup的效果如:(spark,(CompactBuffer(1, 1),CompactBuffer(3))),考虑使用for循环,
    val cordd: RDD[(String, (Iterable[Int], Iterable[Int]))] = rdd1.cogroup(rdd2)
   //flatMapValues把集合扁平化了,避免了空值的出现
    val joinrdd: RDD[(String, (Int, Int))] = cordd.flatMapValues(data => {
      for (it <- data._1; it2 <- data._2) yield (it, it2)
    })
    joinrdd.collect().foreach(println)
    sc.stop()
  }
}
```
## 6.fullOuterJoin
```
object FullJoinDemo {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("groupbykey").setMaster("local[*]"))
    val rdd1 = sc.makeRDD(List(("spark", 1), ("scala", 2), ("kafka", 4), ("flink", 9), ("spark", 1)))
    val rdd2 = sc.makeRDD(List(("spark", 3), ("scala", 4), ("kafka", 5), ("strom", 2), ("strom", 4)))
    val cprdd = rdd1.cogroup(rdd2)
    cprdd.flatMapValues {
      case (vs, Seq()) => vs.iterator.map((_,None))
      case (Seq(), ws) => ws.iterator.map((None,_))
      case (vs, ws) => for (it <- vs.iterator; it2 <- ws.iterator) yield (Some(it), Some(it2))
    }.collect().foreach(println)
    sc.stop()
  }
}
```

