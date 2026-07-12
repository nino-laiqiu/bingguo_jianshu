>DAG基本概念 分区算子与shuffle 交集和差集实现   partitionBy实现

##1.基本概念
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2e95bb98520bc6ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**一个job是一个DAG
一个task是类的实例
taskset等同于stage**

##2.交集,差集
**交集**
```
object Test2 {
  def main(args: Array[String]): Unit = {
    val sc: SparkContext = new SparkContext(new SparkConf().setAppName("test5").setMaster("local[*]"))
    val rdd1 = sc.makeRDD(List(1, 2, 3, 4, 5))
    val rdd2 = sc.makeRDD(List(4, 5, 6, 7, 8))
    val rdd = rdd2.map(data => (data, null))
    val result = rdd1.map(data => (data, null)).cogroup(rdd).filter {
      case (_, (ws, vs)) => ws.nonEmpty && vs.nonEmpty
    }.keys
    result.collect.foreach(println)
    sc.stop()
  }
}
```
**差集**
底层使用hash去重
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.immutable.HashMap
import scala.collection.{immutable, mutable}

object Test4 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("").setMaster("local[*]"))
    val rdd1 = sc.makeRDD(List(1, 2,2,3, 3, 4, 5))
    val rdd2 = sc.makeRDD(List(4, 5, 5, 6, 7))
    val map1rdd1: RDD[(Int, Iterable[Null])] = rdd1.map(data => (data, null)).groupByKey()
    val map1rdd2: RDD[(Int, Iterable[Null])] = rdd2.map(data => (data, null)).groupByKey()
    var hash = mutable.HashMap[Int, Iterable[Null]]()
    map1rdd1.collect.foreach(data => hash += data)
    map1rdd2.collect.foreach(data => hash.remove(data._1) )
    println(hash)
    val list: immutable.Seq[(Int, Null)] = hash.toList.flatMap(data => {
      data._2.map(data1 => (data._1, data1))
    })
    list.map(data => data._1).foreach(println)
    sc.stop()
  }
}
```
##3.分区算子
```
//partitionBy实现
object Test3 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("").setMaster("local[*]"))
    var rdd: RDD[(String, Int)] =sc.makeRDD(List(("A",1),("B",2),("D",9),("A",1)),2)
    val value: ShuffledRDD[String, Int, Int] = new ShuffledRDD[String, Int, Int](rdd, new HashPartitioner(rdd.partitions.length+1))
    println(value.partitions.length)
    sc.stop()
  }
}
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-db12967afbe7bde7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4.Driver和 Executor

