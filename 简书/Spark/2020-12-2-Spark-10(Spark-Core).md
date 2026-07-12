>Wordcount案例四种方式实现(使用元组的排序功能),累加器的应用

##京东电商业务分析
## Wordcount
方法一:使用缓存+cogroup
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

//说明:字段之间以"_"划分,订单数量和支付数量有多个以","划分
//订单和支付中存放的是订单的编号
object Test1 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("jdany").setMaster("local[*]"))
    val rdd: RDD[String] = sc.textFile("src/main/resources/user_visit_action.txt")
    //共用数据是rdd缓存
    rdd.cache()
    //统计点击数量
    val clickorderrdd: RDD[(String, Int)] = rdd.filter(data => {
      val strip = data.split("_")
      strip(6) != "-1"
    }).map(data => {
      val strip = data.split("_")
      (strip(6), 1)
    }).combineByKey(
      //(u,v)u是初始值
      v => v,
      (u: Int, v) => u + v,
      (x1: Int, x2: Int) => x1 + x2
    )
    //统计订单数量
    //订单可能有多个之间","划分
    val orderorderrdd: RDD[(String, Int)] = rdd.filter(data => {
      val strip = data.split("_")
      strip(8) != "null"
    }).flatMap(data => {
      val strip = data.split("_")
      val txt = strip(8).split(",")
      txt.map(data1 => (data1, 1))
    }).reduceByKey(_ + _)
    //统计支付数量
    val payorderrdd: RDD[(String, Int)] = rdd.filter(data => {
      val strip = data.split("_")
      strip(10) != "null"
    }).flatMap(data => {
      val strip = data.split("_")
      strip(10).split(",").map(data => (data, 1))
    }).foldByKey(0)(_ + _)

    //使用连接分组: 结构为  (A,(B,C,D) 因为元组的比较是先比较前面然后在比较后面的数据
    val resultrdd: RDD[(String, (Iterable[Int], Iterable[Int], Iterable[Int]))] = clickorderrdd.cogroup(orderorderrdd, payorderrdd)
    //因为已经求过和了所以迭代器中只有一个数
    val value: RDD[(String, (Int, Int, Int))] = resultrdd.map(data => {
      (data._1, (data._2._1.toList.sum, data._2._2.toList.sum, data._2._3.toList.sum))
    })
    val topN: Array[(String, (Int, Int, Int))] = value.sortBy(data => data._2,false).take(10)
     println(topN.mkString(","))
    sc.stop()
  }
}
```
方法二:使用格式化+union
```
    val value = clickorderrdd.map(data =>
      (data._1, (data._2, 0, 0))
    )
    val value1 = orderorderrdd.map(data =>(data._1,( 0, data._2, 0)))
    val value2 = payorderrdd.map(data => (data._1,(0, 0, data._2)))
    val value3: RDD[(String, (Int, Int, Int))] = value.union(value1).union(value2)
    //union的只是连接
    //reducebykey上一个下一个?
    val value4: RDD[(String, (Int, Int, Int))] = value3.reduceByKey((x1, x2) =>
      (x1._1 + x2._1, x1._2 + x2._2, x1._3 + x2._3)
    )
    val tuples: Array[(String, (Int, Int, Int))] = value4.sortBy(_._2, false).take(10)
    println(tuples.mkString(""))
    sc.stop()
```
方法三:使用if判断
```
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object Test2 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("jdany").setMaster("local[*]"))
    val rdd: RDD[String] = sc.textFile("src/main/resources/user_visit_action.txt")
    val result: RDD[(String, (Int, Int, Int))] = rdd.flatMap(data => {
      val strip = data.split("_")
      if (strip(6) != "-1") {
        List((strip(6), (1, 0, 0)))
      }
      else if (strip(8) != "null") {
        strip(8).split(",").map(data => (data, (0, 1, 0)))
      }
      else if (strip(10) != "null") {
        strip(10).split(",").map(data => (data, (0, 0, 1)))
      }
      else {
        Nil
      }
    })
    val value = result.reduceByKey((x, x1) => {
      (x._1 + x1._1, x._2 + x1._2, x._3 + x1._3)
    })
    val tuples = value.sortBy(_._2, false).take(10)
    println(tuples.mkString(","))
    sc.stop()
  }
}
```
方法四:累加器
```
```
## 跳转率的分析
