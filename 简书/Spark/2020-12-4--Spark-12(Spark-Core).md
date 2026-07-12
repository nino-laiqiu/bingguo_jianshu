>distinct算子  数据写入mysql  topN案例的性能分析

##1.spark基本概念的复习
RDD:
是一个抽象数据集，RDD中不保存要计算的数据，保存的是元数据（描述信息），即数据的描述信息和运算逻辑，比如数据要从哪里读取，怎么运算等
RDD可以认为是一个代理，你对RDD进行操作，相当于在Driver端先是记录下计算的描述信息，然后生成Task，将Task调度到Executor端才执行真正的计算逻辑

DAG:
是对多个RDD转换过程和依赖关系的描述
触发Action就会形成一个完整的DAG，一个DAG就是一个Job
一个Job中有一到多个Stage，一个Stage对应一个TaskSet，一个TaskSet中有一到多个Task

Job:
Driver向Executor提交的作业
触发一次Acition形成一个完整的DAG
一个DAG对应一个Job

Stage:
Stage执行是有先后顺序的，先执行前的，在执行后面的
一个Stage对应一个TaskSet
一个TaskSet中的Task的数量取决于Stage中最后一个RDD分区的数量

##2.重写distinct算子
默认的distinct算子操作效率太低,改写算子使用treeset的去重功能
```
def mydistinct(iter: Iterator[(String, Int)]): Iterator[String] = {
  iter.foldLeft(Set[String]())((CurS, item) => CurS + item._1).toIterator
}
```

##3.top算子分析(队列)与shuffle
##4.数据写入mysql与log日志
//画图解释
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9af7b26d39db25b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**方法一:把excutor数据传入到driver端**
//写入到MySQL的第一种方式（将Executor端的数据通过网络手机到Driver端，在Driver端将数据写入到数据库中的）
//这种方式适用于少量数据（聚合运算）
```
import java.sql.{Connection, Date, DriverManager, PreparedStatement, Statement}

import com.alibaba.fastjson.{JSON, JSONException}
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
import org.slf4j.{Logger, LoggerFactory}
//形如
//{"oid":"o129", "cid": 3, "money": 300.0, "longitude":115.398128,"latitude":35.916527}
//{"oid":"o130", "cid": 2, "money": 100.0, "longitude":116.397128,"latitude":39.916527}
//{"oid":"o131", "cid": 1, "money": 100.0, "longitude":117.394128,"latitude":38.916527}
//{"oid":"o132", "cid": 3, "money": 200.0, "longitude":118.396128,"latitude":35.916527}
//1,家具
//2,手机
//3,服装

//try-with-resources语法
object Test1 {
  private val logger: Logger = LoggerFactory.getLogger(this.getClass)

  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("order").setMaster("local[*]"))
    val rdd = sc.textFile(args(0))
    //读取要join的数据
    val rdd1 = sc.textFile(args(1))
    //读取业务数据
    val operationrdd: RDD[(Int, Double)] = rdd.map(data => {
      //使用temp来接收,如果不用怎么获取try中的数据
      var temp: (Int, Double) = null
      try {
        val json = JSON.parseObject(data)
        val cid = json.getInteger("cid").toInt
        val money = json.getDouble("money").toDouble
        temp = (cid, money)
      }
      catch {
        case e: JSONException => logger.error("json解析错误" + e)
      }
      temp
    })
    //过滤脏数据
    val filter = operationrdd.filter(_ != null)
    //把money相加
    val conbinerdd: RDD[(Int, Double)] = filter.combineByKey(
      v => v,
      (u: Double, v: Double) => u + v,
      (x1: Double, x2: Double) => x1 + x2
    )
    //读取join的数据
    val joinrdd: RDD[(Int, String)] = rdd1.map(data => {
      val strings = data.split(",")
      (strings(0).toInt, strings(1).toString)
    })
    val value: RDD[(Int, (Double, Option[String]))] = conbinerdd.leftOuterJoin(joinrdd)
    //使用jdbc创建连接
    var conn: Connection = null
    var statement: PreparedStatement = null
    //在diver端创建
    try {
      conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db_demo4?characterEncoding=utf8", "root", "123456")
      //创建Statement
      //发现问题:SQL语句insert不会写了.....
      statement = conn.prepareStatement("insert  into  tb_myorder_spark values(?,?,?,?)")
      value.collect.foreach(data => {
        statement.setDate(1, new Date(System.currentTimeMillis()))
        statement.setInt(2, data._1)
        statement.setDouble(3, data._2._1)
        statement.setString(4, data._2._2.getOrElse("未知").toString)
        statement.executeUpdate()
      })
    } catch {
      case e: Exception => logger.error("jdbc" + e)
    }
    finally {
      if (statement != null && conn != null) {
        statement.close()
        conn.close()
      }
    }
    sc.stop()
  }
}
```
**方法二:jdbc连接在excutor端创建**
注意jdbc连接要在excutor
```
    value.foreachPartition(data => {
      var conn: Connection = null
      var statement: PreparedStatement = null
      try {
        conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db_demo4?characterEncoding=utf8", "root", "123456")
        //创建Statement
        statement = conn.prepareStatement("insert  into  tb_myorder_spark values(?,?,?,?)")
        data.foreach(data1 => {
          statement.setDate(1, new Date(System.currentTimeMillis()))
          statement.setInt(2, data1._1)
          statement.setDouble(3, data1._2._1)
          statement.setString(4, data1._2._2.getOrElse("未知").toString)
          statement.executeUpdate()
        })
      }
      catch {
        case e: Exception => logger.error("jdbc" + e)
      }
      finally {
        if (statement != null && conn != null) {
          statement.close()
          conn.close()
        }
      }
    })
```

##5.topN的多种写法(隐式转换的使用 自定排序规则 分治 分区器)
**--分析流程与适用场景**
```
topN案例,形如如下数据
http://javaee.51doit.cn/xiaozhang
http://javaee.51doit.cn/laowang
http://javaee.51doit.cn/laowang
http://javaee.51doit.cn/laowang
```

方法1:分组放到list中使用scala的排序规则
```
object Test2 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]"))
    //分组topN
    val rdd = sc.textFile("src/main/resources/topN.txt")
    //格式化数据
    val initial = rdd.map(data => {
      val result = data.split("/")
      val name = result(3)
       //注意是否包含首尾
      val course = result(2).substring(0, result(2).indexOf("."))
      ((course.toString, name.toString), 1)
    })
    //这里是两次shuffle,聚合操作和重新分区的操作
    val value: RDD[(String, Iterable[((String, String), Int)])] = initial.reduceByKey(_ + _).groupBy(data => data._1._1)
    //对value值操作,如何排序?转list放到内存中分组取topN,排序take方法取topN
    val result: RDD[(String, List[((String, String), Int)])] = value.mapValues(data => data.toList.sortBy(data => data._2)(Ordering.Int.reverse).take(3))
    result.collect.foreach(println)
    sc.stop()
  }
}
```
方法2:分治,top算子(循环但是触发了多次job)
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Test3 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]"))
    //分组topN
    val rdd = sc.textFile("src/main/resources/topN.txt")
    //格式化数据
    val initial = rdd.map(data => {
      val result = data.split("/")
      val name = result(3)
      //注意是否包含首尾
      val course = result(2).substring(0, result(2).indexOf("."))
      ((course.toString, name.toString), 1)
    })
    //得到了每个分组的value值的总和
    val reducerdd = initial.reduceByKey(_ + _)
    //分治思想获取每个key._1的topN
    var list = List[String]("bigdata", "javaee")
    //分治排序方法一:sortby
    for (subject <- list) {
      /*      //过滤出每个科目的数据,排序获取take
            val filterrdd = reducerdd.filter(data => data._1._1 == subject)
            //按照value值来排序
            //sortby是如何排序的?默认是升序
            val tuples = filterrdd.sortBy(data => data._2, false).take(3).toList
            println(tuples)*/
      //分治排序方法二:方法二使用top取值
      //top的使用,是获取每个分区内的topN之后聚合取topN
      //首先过滤出每个分组的数据
      val filterrdd: RDD[((String, String), Int)] = reducerdd.filter(data => data._1._1 == subject)
      implicit val sort = Ordering[Int].on[((String, String), Int)](t => t._2)
      //top是使原本降序的规则升序
      val result = filterrdd.top(3).toList
      println(result)
    }
    sc.stop()
  }
}
```
方法3:使用treeset的排序功能,取分区前topN(遗留问题,不正确发生了去重)
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable

object Test4 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]"))
    val rdd = sc.textFile("src/main/resources/topN.txt")
    val initial = rdd.map(data => {
      val result = data.split("/")
      val name = result(3)
      val course = result(2).substring(0, result(2).indexOf("."))
      ((course.toString, name.toString), 1)
    })
    val reducerdd = initial.reduceByKey(_ + _)
    //分组相同的key必然落到同一个分区!!!
    //使用set的自动排序
    val grouprdd: RDD[(String, Iterable[(String, Int)])] = reducerdd.map(data => (data._1._1, (data._1._2, data._2))).groupByKey()
    //每个组一个set
    val value: RDD[(String, List[(String, Int)])] = grouprdd.mapValues(data => {
      //隐式转换,降序
      implicit val sortedSet = Ordering[Int].on[(String, Int)](t => t._2).reverse
      val set = mutable.Set[(String, Int)]()
      data.foreach(tr => {
        set += tr
        if (set.size > 3) {
          set -= set.last
        }
      })
      set.toList.sortBy(x => -x._2)
    })
    value.collect.foreach(println)
    sc.stop()
  }
}
```

方法4:**重写分区器(可避免数据倾斜,但是task多了),重点如何获取要分区的数量**
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{Partitioner, SparkConf, SparkContext}

import scala.collection.immutable.HashMap
import scala.collection.mutable

object Test5 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]"))
    //分组topN
    val rdd = sc.textFile("src/main/resources/topN.txt")
    //格式化数据
    val initial = rdd.map(data => {
      val result = data.split("/")
      val name = result(3)
      //注意是否包含首尾
      val course = result(2).substring(0, result(2).indexOf("."))
      ((course.toString, name.toString), 1)
    })
    //首先聚合
    val aggrdd: RDD[((String, String), Int)] = initial.aggregateByKey(0)(_ + _, _ + _)
    //触发一次actor获取科目的总数
    val collect: Array[String] = aggrdd.map(data => data._1._1).distinct.collect
    //方法五基于方法四要groupbykey会产生shuffle的优化,使用自定义分区器,有点避免了shuffle但是增加了task
    val mypartitioner = new Mypartitioner(collect)
    val pardd: RDD[((String, String), Int)] = aggrdd.partitionBy(mypartitioner)
    //已经重新分区了,每个分区只有特定的key值
    //一批一批传输
    val value = pardd.mapPartitions(data => {
      implicit val sout = Ordering[Int].on[((String, String), Int)](t => t._2)
      val set = mutable.Set[((String, String), Int)]()
      data.foreach(it => {
        set += it
        if (set.size > 3) {
          set -= set.last
        }
      })
      set.toList.sortBy(data => -data._2).iterator
    })
    value.collect.foreach(println)
  }
}

class Mypartitioner(val collect: Array[String]) extends Partitioner {
  private val stringToInt = new mutable.HashMap[String, Int]()
  var label = 0
  for (key <- collect) {
    stringToInt(key) = label
    label += 1
  }

  override def numPartitions: Int = collect.length

  override def getPartition(key: Any): Int = {
    //获取科目
    val i = key.asInstanceOf[(String, String)]._1
    stringToInt(i)
  }
}
```
方法五:调用reduceByKey时传入自定义的分区器

```
import org.apache.spark.{Partitioner, SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

import scala.collection.mutable

object Test6 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]"))
    //分组topN
    val rdd = sc.textFile("src/main/resources/topN.txt")
    //格式化数据
    val initial = rdd.map(data => {
      val result = data.split("/")
      val name = result(3)
      //注意是否包含首尾
      val course = result(2).substring(0, result(2).indexOf("."))
      ((course.toString, name.toString), 1)
    })
    //首先先不reducebykey,其实这种方法是多余的,只要初始的数据结构是(course,(name.number)就大可不必
    //触发一次行动算子算出科目去重
    val collect: Array[String] = initial.map(data => data._1._1).distinct.collect
    val partition = new MyPartition(collect)
    val value: RDD[((String, String), Int)] = initial.reduceByKey(partition, _ + _)
    val res: RDD[((String, String), Int)] = value.mapPartitions(it => {
      //定义一个可排序的集合TreeSet
      implicit val ord: Ordering[((String, String), Int)] = Ordering[Int].on[((String, String), Int)](t => t._2).reverse
      val sorter = new mutable.TreeSet[((String, String), Int)]()
      //变量迭代器
      it.foreach(t => {
        //将当前的这一条数据放入到treeSet中
        sorter.add(t)
        if (sorter.size > 3) {
          //移除最小的
          val last = sorter.last
          sorter -= last
        }
      })
      sorter.iterator
    })
    println(res.collect().toBuffer)
    sc.stop()
  }
}
class MyPartition(val collect: Array[String]) extends Partitioner {
  private val rules = new mutable.HashMap[String, Int]()
  var index = 0
  //初始化一个分区的规则
  for (sb <- collect) {
    rules(sb) = index
    index += 1
  }

  override def numPartitions: Int = collect.size

  override def getPartition(key: Any): Int = {
    val value = key.asInstanceOf[(String, String)]._1
    rules(value)
  }
}
```
方法6:shuffleRDD
```
import org.apache.spark.rdd.ShuffledRDD
import org.apache.spark.{Partitioner, SparkConf, SparkContext}

import scala.collection.mutable

object Test7 {
  def main(args: Array[String]): Unit = {
    //使用shuffleRDD
    //val result = countsAndSubjectTeacher.repartitionAndSortWithinPartitions(subjectPartitioner)
    //按照某种格式进行操作,然后重新采用另一种格式
    val sc = new SparkContext(new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]"))
    val rdd = sc.textFile("src/main/resources/topN.txt")
    val initial = rdd.map(data => {
      val result = data.split("/")
      val name = result(3)
      //注意是否包含首尾
      val course = result(2).substring(0, result(2).indexOf("."))
      ((course.toString, name.toString), 1)
    })
    //首先聚和
    val reducerdd = initial.reduceByKey(_ + _)
    //执行一次actor算子获取共有多少课程
    val collect: Array[String] = reducerdd.map(data => data._1._1).distinct.collect
    val ypartition = new MYpartition(collect)
    //使用元组的排序特性,重新格式化
    val value = reducerdd.map {
      case ((x, y), z) => ((z, (x, y)),null)
    }
    //按照第二元素重新分区,使其在同一个分区中
    implicit  val sort = Ordering[Int].on[((Int,(String,String)))](t => t._1).reverse
    //怎么定义shuffle的类型
    val value1 = new ShuffledRDD[(Int, (String, String)), Null, Null](value, ypartition)
    value1.setKeyOrdering(sort)
    value1.keys.collect.foreach(println)
  }
}

class MYpartition(val collect: Array[String]) extends Partitioner {
  private val map: mutable.Map[String, Int] = mutable.Map[String, Int]()
  var label = 0
  for (th <- collect) {
    map(th) = label
    label += 1
  }

  override def numPartitions: Int = collect.size

  override def getPartition(key: Any): Int = {
    val value = key.asInstanceOf[(Int, (String, String))]._2._1
    map(value)
  }
}
```
(具体场景具体的分析方法)
