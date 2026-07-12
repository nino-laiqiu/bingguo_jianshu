>聚合算子的区别 ,flatmapvalue的实现原理 各种join的实现(使用cogroup)(模式匹配的应用) 分组求和的多种方式
(考虑算子的性能和效率)

##1.复习4个聚合算子
**aggregateBykey:分区内的规则与分区间的规则可以不一致
reduceBykey:无法指定初始值,分区内和分区间的规则是一致的
foldBykey:是aggregateBykey的简化版,分区内和分区间的规则是一致的
ConbineBykey:比较与aggregateBykey的初始值,更符合聚合的含义,第一个参数指定返回的RDDvalue值类型**

##2.keys,values,mapValues、flatMapValues
```
object Test1 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("t").setMaster("local[*]"))
    val rdd: RDD[(String, Int)] = sc.makeRDD(List(("A", 1), ("B", 8), ("C", 7), ("A", 7)))
    // def keys: RDD[K] = self.map(_._1)
    val keysrdd: RDD[String] = rdd.keys
    val valrdd: RDD[Int] = rdd.values
    val mvrdd: RDD[(String, Int)] = rdd.mapValues(data => data)
    keysrdd.collect.foreach(println)
    sc.stop()
  }
}
```
flatMapValues源码:
```
    val cleanF = self.context.clean(f)
    new MapPartitionsRDD[(K, U), (K, V)](self,
      (context, pid, iter) => iter.flatMap { case (k, v) =>
        cleanF(v).map(x => (k, x))
      },
      preservesPartitioning = true)
  }
```
##3.分组求和(groupByKey reduceByKey flatMapValues )
```
//groupByKey 
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Test2 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test2").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(("小明", "1,2,3"), ("小明", "4,5,6"), ("小莫", "2,3,4")))
    //分组求总和
    val value: RDD[(String, Iterable[String])] = rdd.groupByKey()
    val value1: RDD[(String, Iterable[Array[Int]])] = value.mapValues(data => {
      data.map(data => {
        val strings: Array[String] = data.split(",")
        val ints: Array[Int] = strings.map(data => data.toInt)
        ints
      })
    })
    val result = value1.map(data => (data._1, data._2.map(_.sum).sum))
    result.collect.foreach(println)
    sc.stop()
  }
}
```
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Test3 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test2").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(("小明", "1,2,3"), ("小明", "4,5,6"), ("小莫", "2,3,4")))
    val mvrdd: RDD[(String, Array[String])] = rdd.mapValues(data => data.split(","))
    val maprdd: RDD[(String, Int)] = mvrdd.map(data => (data._1, data._2.map(data => data.toInt).sum))
    val result: RDD[(String, Int)] = maprdd.reduceByKey(_ + _)
     result.collect.foreach(println)
    sc.stop()
  }
}
```
```
//flatMapValues
object Test4 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test2").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(("小明", "1,2,3"), ("小明", "4,5,6"), ("小莫", "2,3,4")),5)
    val fmvrdd: RDD[(String, Int)] = rdd.flatMapValues(data => data.split(",").map(_.toInt))
    val value: RDD[(String, Int)] = fmvrdd.reduceByKey(_ + _)
    value.collect.foreach(println)
    sc.stop()
  }
}
```
```
//flatMapValues的实现
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Test5 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test2").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(("小明", "1,2,3"), ("小明", "4,5,6"), ("小莫", "2,3,4")),2)
    val result: RDD[Array[(String, String)]] = rdd.map(data => {
      data match {
        case (x, y) => {
          //返回的是数组
          val tuples: Array[(String, String)] = y.split(",").map(data => (x, data))
          tuples
        }
      }
    })
    val value: RDD[(String, String)] = result.flatMap(data => data)
      value.collect.foreach(println)
     //result.collect.foreach(println)
    sc.stop()
  }
}
```
##4.用cogroup实现内连接 左(右)外连接  全连接
**(flatmapvalue和mapvalue)**
```
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object Test6 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("t").setMaster("local[*]"))
    val rdd: RDD[(String, Int)] = sc.makeRDD(List(("A", 1), ("B", 8), ("C", 7), ("A", 7)))
    val rdd1 = sc.makeRDD(List(("A", "60"), ("A", "80"), ("C", "90"), ("F", "100")))
    //connectandgroup方法
    // val cagrdd: RDD[(String, (Iterable[Int], Iterable[String]))] = rdd.cogroup(rdd1)
    // cagrdd.collect.foreach(println)
    //(A,(CompactBuffer(1, 7),CompactBuffer(60, 80)))
    //(B,(CompactBuffer(8),CompactBuffer()))
    //(C,(CompactBuffer(7),CompactBuffer(90)))
    //(F,(CompactBuffer(),CompactBuffer(100)))
    //rdd.join(rdd1).collect.foreach(println)
    //实现join
    val value: RDD[(String, (Int, String))] = rdd.cogroup(rdd1).flatMapValues(x => {
      //为什么可以这样,数据是一条一条传过来的,只要数据为空就跳过,从而实现join的效果
      for (num <- x._1; num2 <- x._2) yield (num, num2)
    })
    value.collect.foreach(println)
    sc.stop()
  }
}
```

```

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object Test6 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("t").setMaster("local[*]"))
    val rdd: RDD[(String, Int)] = sc.makeRDD(List(("A", 1), ("B", 8), ("C", 7), ("A", 7)))
    val rdd1 = sc.makeRDD(List(("A", "60"), ("A", "80"), ("C", "90"), ("F", "100")))
    val cagrdd: RDD[(String, (Iterable[Int], Iterable[String]))] = rdd.cogroup(rdd1)
    val value = cagrdd.flatMapValues(data => {
      data match {
        case (vs, Seq()) => vs.iterator.map(data => (data, None))
        case (vs, ws) => for (x <- vs.iterator; y <- ws.iterator) yield (x, y)
      }
    })
    value.collect.foreach(println)
    sc.stop()
  }
}
```
```
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object Test6 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("t").setMaster("local[*]"))
    val rdd: RDD[(String, Int)] = sc.makeRDD(List(("A", 1), ("B", 8), ("C", 7), ("A", 7)))
    val rdd1 = sc.makeRDD(List(("A", "60"), ("A", "80"), ("C", "90"), ("F", "100")))
    //实现leftoutjoin
    val cagrdd: RDD[(String, (Iterable[Int], Iterable[String]))] = rdd.cogroup(rdd1)
    val value = cagrdd.flatMapValues(data => {
      data match {
        case (vs, Seq()) => vs.iterator.map(data => (data, None))
        case (Seq(),ws) => ws.iterator.map(data => (None,data))
        case (vs, ws) => for (x <- vs.iterator; y <- ws.iterator) yield (x, y)
      }
    })
    value.collect.foreach(println)
    sc.stop()
  }
}
```
(io.Serializable 源码是包装为Some())
```
import java.io

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object Test7 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("t").setMaster("local[*]"))
    val rdd: RDD[(String, Int)] = sc.makeRDD(List(("A", 1), ("B", 8), ("C", 7), ("A", 7)))
    val rdd1 = sc.makeRDD(List(("A", "60"), ("A", "80"), ("C", "90"), ("F", "100")))
    val cagrdd: RDD[(String, (Iterable[Int], Iterable[String]))] = rdd.cogroup(rdd1)
    //使用if来判断
    val value: RDD[(String, (Int, io.Serializable))] = cagrdd.flatMapValues(data => {

      if (data._1.isEmpty && data._2.nonEmpty) {
        data._2.map(data => (None, data))
      }
      if (data._1.nonEmpty && data._2.isEmpty) {
        data._1.map(data => (data, None))
      }
      else {
        for (vs <- data._1; ws <- data._2) yield (vs, ws)
      }
    })
    value.collect.foreach(println)
    sc.stop()
  }
}
```

