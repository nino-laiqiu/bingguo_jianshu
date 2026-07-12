>实践RDD的容器,使用conbinebykey和treeset实现topN
RDD内部执行了多少次和什么有关系
cache/persist  和 checkpoint分析流程

![image.png](https://upload-images.jianshu.io/upload_images/9049859-3659125a6d097a7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

sorbbykey的采样详情与Wordcount执行流程分析
shuffle的中间结果

##topN案例的补充,使用conbinebykey算子来优化
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable

//使用conbinebykey来实现topN
object Test2 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]"))
    //分组topN
    val rdd = sc.textFile("src/main/resources/topN.txt")
    //格式化数据
    val initial = rdd.map(data => {
      val result = data.split("/")
      val name = result(3) trim()
      //注意是否包含首尾
      val course = result(2).trim().substring(0, result(2).trim().indexOf("."))
      ((course.toString, name.toString), 1)
    })
    val caserdd: RDD[(String, (String, Int))] = initial.reduceByKey(_ + _).map {
      case ((x, y), z) => (x, (y, z))
    }

    //获取同一key值的topN
    //传递一个数组,返回一个treeset
    val value = caserdd.combineByKey(createCombiner, mergeValue, mergeCombiners)
    //描述 C-V  (V,C) - V (V,V) -V
    value.collect.foreach(println)
    sc.stop()

  }

  def createCombiner(namescore: (String, Int)) = {
    println("------")
    implicit var sort = Ordering[Int].on[(String, Int)](t => t._2).reverse
    val tuples = mutable.TreeSet[(String, Int)]()
    tuples += namescore
    tuples
  }

  //这个方法执行了多少次?局部聚和
  def mergeValue(tree: mutable.TreeSet[(String, Int)], namescore: (String, Int)) = {
    println("+++++++")
    tree += namescore
    if (tree.size > 3) {
      tree -= tree.last
    }
    tree
  }

  //聚合合并不同分区相同key的值,局部聚合在shuffle之前
  def mergeCombiners(tree: mutable.TreeSet[(String, Int)], tree1: mutable.TreeSet[(String, Int)]) = {
    println("???????")
    implicit var sort = Ordering[Int].on[(String, Int)](t => t._2).reverse
    val tuples = mutable.TreeSet[(String, Int)]()
    for (x <- tree) {
      tuples += x
      if (tuples.size > 3) {
        tuples -= tuples.last
      }
    }
    for (y <- tree1) {
      tuples += y
      if (tuples.size > 3) {
        tuples -= tuples.last
      }
    }
    tuples
  }
}
```
