> RDD算子 map  mapPartitions  mapPartitionsWithIndex

## 1.map算子演示
(效率性能低,类似于io流中一个字节一个字节读,要考虑缓冲区)
(读取集合数据集中:注意分区的并行.分区数与设置的core之间的关系)
(实践分别设置local 和 numslices 分别为(1,1)(1,2)(2,1))
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Test1 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("test1").setMaster("local[2]")
    val sc = new SparkContext(conf)
    val v: RDD[Int] = sc.makeRDD(List[Int](1, 2, 3, 4),2)
    val v1 = v.map(data => {
      println(">>>>>>>" + data)
      data
    })
    val v2 = v1.map(data => {
      println("!!!!" + data)
      data
    })
    v2.collect()
    sc.stop()
  }
}
```
##2.mapPartitions 算子演示

```
 val test = new SparkConf().setMaster("local[*]").setAppName("test2")
    val sc = new SparkContext(test)
    val rdd: RDD[Int] = sc.makeRDD(List[Int](1, 2, 3, 4, 5, 6),2)
    //指定分区,">>>>" 打印了俩次,
    val mapp = rdd.mapPartitions(iter => {
      println(">>>>>>")
      iter.map(_ * 1)
    })
    mapp.collect().foreach(println)
    sc.stop()
  }
```
(组内获取最大值)
```
    val conf = new SparkConf().setAppName("test3").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val RDD = sc.makeRDD(List[Int](1, 2, 3, 4, 5, 6),2)
    val mapprdd = RDD.mapPartitions(iter => {
      println(">>>>>")
      List[Int](iter.max).iterator
    })
    mapprdd.collect.foreach(println)
    sc.stop()
  }
```
##3.mapPartitionsWithIndex算子演示
获取分组为1的元素
```
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("test4").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val RDD = sc.makeRDD(List[Int](1, 2, 3, 4),2)
    val map = RDD.mapPartitionsWithIndex((index, iter) => {
      if (index == 1) {
        iter
      }
      else {
        Nil.iterator
      }
    })
    map.collect.foreach(println)
    sc.stop()
  }
```
```
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("test5").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val RDD = sc.makeRDD(List[Int](1, 2, 3, 4, 5))
    val result = RDD.mapPartitionsWithIndex((index, iter) => {
      iter.map(data => {
        (index, data)
      }
      )
    })
    result.collect.foreach(println)
    sc.stop()
  }
```
