##1.常用行动算子
```
import org.apache.spark.{SparkConf, SparkContext}

object Test9 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test10").setMaster("local[*]"))
    val rdd = sc.makeRDD(List(4, 5, 6, 1, 2, 3))
    //reduce聚合
    println(rdd.reduce(_ + _))
    //collect 收集返回数组类型
    val ints: Array[Int] = rdd.collect()
    println(ints.mkString(","))
    //count求总的次数
    //sum算子在excutor进行相加,返回结果最后在Driver端相加
    println(rdd.count())
    println(rdd.sum())
    //获取第一个
    println(rdd.first())
    //获取前topN
    val ints1: Array[Int] = rdd.take(2)
    println(ints1.mkString(","))
    //获取排序过的topN
    rdd.takeOrdered(3).foreach(println)
    //排序,指定排序规则
    rdd.takeOrdered(3)(Ordering.Int.reverse).foreach(println)
    //
    val intToLong: collection.Map[Int, Long] = rdd.countByValue()
    println(intToLong)//Map(5 -> 1, 1 -> 1, 6 -> 1, 2 -> 1, 3 -> 1, 4 -> 1)
  }
}
```



##2.辨析aggregateByKey(转换算子)   aggregate(行动算子)
(注意初始值参与分区内和分区间对于行动算子),(注意初始值在分区内调用的次数,,发现转换算子是一个组调用一次,行动算子是一个分区调用一个,最后聚合的时候再次调用一次,当分区内和分区间的需求一致时使用fold代替)
```
import org.apache.spark.{SparkConf, SparkContext}

object Test8 {
  def main(args: Array[String]): Unit = {
    val sc: SparkContext = new SparkContext(new SparkConf().setAppName("test5").setMaster("local[*]"))
    var  rdd =sc.makeRDD(List(("A",11),("B",11),("C",11),("A",111)),2)
    //初始值参与每个区内的每个组的运算,
    //组的粒度的聚合
    rdd.aggregateByKey(10)(_+_,_+_).collect.foreach(println)
    //初始值参与分区内和分区间
    //全局聚合
    val result = rdd.aggregate(10)((x, y) => x + y._2, (x1, x2) => x1 + x2)
    println(result)
    sc.stop()
  }
}
```
```
    val sc: SparkContext = new SparkContext(new SparkConf().setAppName("test5").setMaster("local[*]"))
    var  rdd = sc.makeRDD(List(1,2,3,4),2)
    val result: Int = rdd.aggregate(10)(_ + _, _ + _)
    println(result)//40
    sc.stop()
```
##3.Wordcount的实现方式(11种)
##4.save行动算子
##5.foreach(方法与算子)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c7c10afd9507d81f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
