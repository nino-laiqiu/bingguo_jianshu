> RDD算子:sortBy,交集,并集,差集,拉链,partitionBy,aggregateByKey,reduceByKey groupByKey,聚合算子,join算子,连接分组算子
>比较RDD算子的性能

##1.sortBy(可能存在shuffle,不能改变分区数量,可指定升序降序)
(true是升序)
(默认按字典顺序排序)
```
    val sc = new SparkContext(new SparkConf().setAppName("...").setMaster("local[*]"))
    val rdd = sc.makeRDD(List[Int](1, 6,2,4,5,3), 2)
    rdd.sortBy(data=>data).saveAsTextFile("src/aa")
    sc.stop()
```
```
    val sc = new SparkContext(new SparkConf().setAppName("...").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(("11", 1), ("22", 2),("11",2),("2",11)),1)
    rdd.sortBy(data=>data._1.toInt,true).saveAsTextFile("src/aa")
    sc.stop()
```
##2.双value类型(注意数据类型,差集的顺序,分区数据一致性)
 **交集**
```
    val sc = new SparkContext(new SparkConf().setAppName("...").setMaster("local[*]"))
    val S1 = sc.makeRDD(List(1, 2, 3, 4))
    val S2 = sc.makeRDD(List(3, 4, 5, 6))
    val result: RDD[Int] = S1.intersection(S2)
    result.collect.foreach(println)
    sc.stop()
```
**并集(共有的数据不会去重,去重可用set序列)**
```
    val sc = new SparkContext(new SparkConf().setAppName("...").setMaster("local[*]"))
    //交集
    val S1 = sc.makeRDD(List(1, 2, 3, 4))
    val S2 = sc.makeRDD(List(3, 4, 5, 6))
    val result = S1.union(S2)
    result.collect.foreach(println)
    sc.stop()
```
**差集(不同的角度,结果不同)**
```
   val sc = new SparkContext(new SparkConf().setAppName("...").setMaster("local[*]"))
    //交集
    val S1 = sc.makeRDD(List(1, 2, 3, 4))
    val S2 = sc.makeRDD(List(3, 4, 5, 6))
    val result = S1.subtract(S2)
    result.collect.foreach(println)
    sc.stop()
```
**拉链(分区数据的一致性)**
**Can't zip RDDs with unequal numbers of partitions**
```
    val sc = new SparkContext(new SparkConf().setAppName("...").setMaster("local[*]"))
    val S1 = sc.makeRDD(List(1, 2, 3, 4))
    val S2 = sc.makeRDD(List(3, 4, 5, 6))
    val result = S1.zip(S2)
    result.collect.foreach(println)
    sc.stop()
```
##3.partitionBy算子(重分区)
(要把数据转换成k,v类型,partitionby针对k,v类型)
```
    var sc = new SparkContext(new SparkConf().setAppName("test2").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6), 2)
    val maprdd: RDD[(Int, Int)] = rdd.map(data => (data, 1))
    val result: RDD[(Int, Int)] = maprdd.partitionBy(new HashPartitioner(2))
    result.saveAsTextFile("src/aa")
    sc.stop()
```
**如果重分区的分区器与当前RDD的分区器一致怎么办?**
例如:
```
val result: RDD[(Int, Int)] = maprdd.partitionBy(new HashPartitioner(2))
        .partitionBy(new HashPartitioner(2))
```
解答:HashPartitioner重写了equals方法,self.partitioner == Some(partitioner)会进行比较,如果相等还是调用当前的分区器
```
  override def equals(other: Any): Boolean = other match {
    case h: HashPartitioner =>
      h.numPartitions == numPartitions
    case _ =>
      false
  }
```
```
 if (self.partitioner == Some(partitioner)) {
      self
    } else {
      new ShuffledRDD[K, V, V](self, partitioner)
    }
```
**其他的分区器**
PythonPartitioner  RangePartitioner  HashPartitioner   sortBykey

##4.reduceByKey  groupByKey groupBy算子的示例
reduceByKey groupByKey 是针对(k,v)类型的 ,前者是聚合 后者是非聚合的结果
groupByKey  groupBy 的key就是数据的key 后者是自己指定的
```
    val sc = new SparkContext(new SparkConf().setAppName("test4").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(("A", 1), ("A", 2), ("A", 3), ("A", 4), ("B", 1)))
    val result1: RDD[(String, Int)] = rdd.reduceByKey((s1, s2) => s1 + s2)
    val result2: RDD[(String, Iterable[Int])] = rdd.groupByKey()
    val result3: RDD[(String, Iterable[(String, Int)])] = rdd.groupBy(_._1)
```

##5.aggregateByKey 和 reduceByKey  算子的示例   以及简化版foldByKey
(分区内  分区间)
(前者分区内部分聚合的规则和分区间聚合的规则可不一致)
(后者分区内部分聚合的规则和分区间聚合的规则是一致的)
foldByKey是简化版的aggregateByKey,分区内和分区间的规则是一致的
```
    val sc = new SparkContext(new SparkConf().setAppName("test4").setMaster("local[4]"))
    val rdd = sc.makeRDD(List(("a", 2), ("b", 4), ("b", 1), ("a", 2), ("a", 8), ("c", 1)), 2)
    rdd.reduceByKey(_ + _).saveAsTextFile("src/aa")
    //(a,12)
    //(c,1)
    //(b,5)
    rdd.aggregateByKey(0)(
      (x, y) => Math.max(x, y), (x, x1) => x + x1
    ).saveAsTextFile("src/bb")
    //(a,10)
    //(c,1)
    //(b,4)
    sc.stop()
  }
 //比较的第一个值为0
 rdd.foldByKey(0)(_+_).saveAsTextFile("/src/cc")
```
**aggregateByKey算子求平均数,对第一个参数的理解**
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Test5 {
  def main(args: Array[String]): Unit = {
    // def aggregateByKey[U: ClassTag](zeroValue: U)(seqOp: (U, V) => U, combOp: (U, U) => U)
    //其中的U是初始值 U是传递过来的值
    //求字母key的平均值
    var sc = new SparkContext(new SparkConf().setAppName("test").setMaster("local[*]"))
    val list = List(("a", 2), ("b", 3), ("a", 4), ("b", 1), ("c", 8), ("a", 3))
    val rdd = sc.makeRDD(list)
    //第一个0叠加总数 第二个0叠加次数
    //类型(Int, Int)与传入的(0,0)是一致的
    val value: RDD[(String, (Int, Int))] = rdd.aggregateByKey((0, 0))(
      //在分区内进行叠加
      (u, v) => (u._1 + v, u._2 + 1),
      //x是初始值和计算后的结果,x1是要叠加的值
      (x, x1) => (x._1 + x1._1, x._2 + x1._2)
    )
    //value值的第一个是总和,第二个值个数
    //方法一mapvalue
    val result: RDD[(String, Int)] = value.mapValues(data => data match {
      case (x, y) => x / y
    })
    //方法二
    val result1: RDD[(String, Int)] = value.map(data => (data._1, (data._2._1 / data._2._2)))
    result1.collect.foreach(println)
    sc.stop()
  }
}
```
##6.combineByKey算子的示例(传入的第一个参数的作用是指定初始值的类型)

```
 def combineByKey[C](
      createCombiner: V => C,
      mergeValue: (C, V) => C,
      mergeCombiners: (C, C) => C): RDD[(K, C)] = self.withScope {
    combineByKeyWithClassTag(createCombiner, mergeValue, mergeCombiners)(null)
  }
```
```
 def main(args: Array[String]): Unit = {
    // def aggregateByKey[U: ClassTag](zeroValue: U)(seqOp: (U, V) => U, combOp: (U, U) => U)
    //其中的U是初始值 U是传递过来的值
    //求字母key的平均值
    val sc = new SparkContext(new SparkConf().setAppName("test").setMaster("local[*]"))
    val list = List(("a", 2), ("b", 3), ("a", 4), ("b", 1), ("c", 8), ("a", 3))
    val rdd = sc.makeRDD(list)
    //第一个0叠加总数 第二个0叠加次数
    //类型(Int, Int)与传入的(0,0)是一致的
    val value: RDD[(String, (Int, Int))] = rdd.combineByKey(
      //在分区内进行叠加
      //V ->  U 的转换
      v => (v,1),
      // U 的类型是(v,1)
      (u:(Int,Int), v) => (u._1 + v, u._2 + 1),
      //x是初始值和计算后的结果,x1是要叠加的值
      (x:(Int,Int), x1) => (x._1 + x1._1, x._2 + x1._2)
    )
    //value值的第一个是总和,第二个值个数
    //方法一mapvalue
    val result: RDD[(String, Int)] = value.mapValues(data => data match {
      case (x, y) => x / y
    })
    //方法二
    val result1: RDD[(String, Int)] = value.map(data => (data._1, (data._2._1 / data._2._2)))
    result1.collect.foreach(println)
    sc.stop()
  }
```
##7.区别四种聚合算子
```
object Test6 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test").setMaster("local[*]"))
    val list = List(("a", 2), ("b", 3), ("a", 4), ("b", 1), ("c", 8), ("a", 3))
    val rdd = sc.makeRDD(list)

    //四种聚合算子
    rdd.reduceByKey(_+_).collect.foreach(println)
    rdd.aggregateByKey(0)((x,y)=>x+y,(x,x1)=>x+x1).collect.foreach(println)
    //foldByKey算子是对aggregateByKey分区内操作的简化版
    rdd.foldByKey(0)(_+_).collect.foreach(println)
    //combineByKey算子是aggregateByKey第一个参数的简化
    rdd.combineByKey(x=>x,(x:Int,y)=>x+y,(x:Int,x1:Int)=>x+x1).collect.foreach(println)
    sc.stop()

  }
}
```
```
reduceByKey 
                 combineByKeyWithClassTag[V](
				 (v: V) => v,
				 func,
				 func, 
				 partitioner)			 
aggregateByKey				 
                combineByKeyWithClassTag[U](
				(v: V) => cleanedSeqOp(createZero(), v),
                cleanedSeqOp, 
				combOp,
				partitioner)
foldByKey 
                combineByKeyWithClassTag[V](
				(v: V) => cleanedFunc(createZero(), v),
                cleanedFunc, 
				cleanedFunc, 
				partitioner)	
combineByKey    combineByKeyWithClassTag(
                createCombiner, 
				mergeValue, 
				mergeCombiners)
				(null) 
```

##8. join()    leftOuterJoin()     rightOuterJoin()  join连接算子
连接时可能产生笛卡尔积

##9.cogroup连接分组
在一个 (K, V) 对的 Dataset 上调用时，返回多个类型为 (K, (Iterable<V>, Iterable<W>)) 的元组所组成的 Dataset。

