>案例分析、回顾与优化
组内 区内 排序  聚合
未解决的问题:使用reducebykey乱序问题(怎么定义合理分区),以及关于在内存排序和shuffle排序的适用场景

##1.连续登陆案例(工具类的使用)
(方法一:groupbykey)
(方法二:使用分区repartitionAndSortWithinPartitions或者mappartition)
注意排序:全局排序和组内排序
注意数据:不完整和去重
```
//连续登陆案例,工具类的使用
object Test4 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getCanonicalName)
    if (args(0).toBoolean) {
      conf.setMaster("local[*]")
    }
    val sc = new SparkContext(conf)
    val rdd = sc.textFile(args(1))
    val maprdd: RDD[(String, String)] = rdd.map(data => {
      val txt = data.split(",")
      val people = txt(0)
      val date = txt(1)
      (people, date)
    })
    //我要排序,直接sort是全局排序,使用mapvalue
    //规范化数据,求取减去index的日期
    //flatMapValues扁平化,每个value值与key结合
    val value: RDD[(String, (String, String))] = maprdd.groupByKey().flatMapValues(datas => {
      //在组内进行排序
      val disduplicationList = datas.toSet.toList.sorted
      val sdf = new SimpleDateFormat("yyyy-MM-dd")
      val calendar = Calendar.getInstance()
      //设置一个index用来记录
      var index = 0
      disduplicationList.map(data => {
        val date: Date = sdf.parse(data)
        //设置日期
        calendar.setTime(date)
        calendar.add(Calendar.DATE, -index)
        index += 1
        (data, sdf.format(calendar.getTime))
      })
    })
    val result: RDD[(String, (Int, String, String))] = value.map(data => {
      ((data._1, data._2._2), data._2._1)
    }).groupByKey().mapValues(data => {
      data.toList.sorted
      val size = data.size
      val head = data.head
      val last = data.last
      (size, head, last)
    }).filter(data => data._2._1 >= 3).map(data => (data._1._1, data._2))
    result.collect().foreach(println)
    sc.stop()
  }
}
```
![SQL转化](https://upload-images.jianshu.io/upload_images/9049859-1abe0dba802b79fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.流量统计(怎么设置合理的标记值)
```
//形如如下数据,同一个id的末尾时间-初始时间大于10min就再分一组
//3,2020/2/18 14:39,2020/2/18 15:35,20
//3,2020/2/18 15:36,2020/2/18 15:24,30
object Test2 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("流量统计")
    val boolean = args(0).toBoolean
    if (boolean) {
      conf.setMaster("local[*]")
    }
    val sc = new SparkContext(conf)
    val rdd = sc.textFile(args(1))
    val maprdd: RDD[(String, (String, String, String))] = rdd.map(data => {
      val txt = data.split(",")
      val id = txt(0)
      val starttime = txt(1)
      val lasttime = txt(2)
      val money = txt(3)
      (id, (starttime, lasttime, money))
    })
    //由于要对同一个id进行操作,考虑把对id进行分组
    val groupbyrdd = maprdd.groupByKey()
    val value: RDD[(String, (String, String, Int, Long))] = groupbyrdd.flatMapValues(data => {
      val sdf = new SimpleDateFormat("yyyy/MM/dd hh:mm")
   //   val parstime = Calendar.getInstance()
       data.map(txt => {
        val start: Date = sdf.parse(txt._1)
        val last: Date = sdf.parse(txt._2)
        //怎么进行两个日期相减?
        //获取两个日期之间相差多少秒
        val l = (last.getTime - start.getTime) / (60 * 1000)
        //如果日期相减是小于10min的我就放在集合中,只要有一个日期相减大于10,就把存入的数据全部取出来??
        if (l < 10) {
          (txt._1, txt._2, txt._3.toInt, 0L)
        }
        else {
          //由于l可能会重复,所以我采用时间戳来表示
          ((txt._1, txt._2, txt._3.toInt,last.getTime ))
        }
      })
    })
    value.map(data => {
      ((data._1,data._2._4),(data._2._1,data._2._2,data._2._3))
    }).reduceByKey((x,y)=>{
          var sum =  x._3 + y._3
          var min = if (x._1 > y._1)  y._1 else x._1
          var max  = if (x._2 > y._2)  x._2   else y._2
      (min,max,sum)
    })
    value.collect().foreach(println)
    sc.stop()
  }
}
```
(**使用时间戳和number作为标签值**)
```
import java.text.SimpleDateFormat

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

//数据如下
//1,2020/2/18 15:24,2020/2/18 15:35,40
//1,2020/2/18 16:06,2020/2/18 16:09,50
//1,2020/2/18 17:21,2020/2/18 17:22,60
object Test2 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName)
    if (args(0).toBoolean) {
      conf.setMaster("local[*]")
    }
    val sc = new SparkContext(conf)
    val rdd = sc.textFile(args(1))
    val maprdd = rdd.map(data => {
      val flow = data.split(",")
      val id = flow(0)
      val starttime = flow(1)
      val lasttime = flow(2)
      val money = flow(3).toInt
      (id, (starttime, lasttime, money))
      //这里的分组是必要的吗
    }).groupByKey()

    val flatmaprdd: RDD[(String, (String, String, Int, Long))] = maprdd.flatMapValues(data => {
      val sdf = new SimpleDateFormat("yyyy/MM/dd hh:mm")
      //定义一个便签值来区分什么时候发生了小于10min的情况
      var number: Long = 0L
      data.map(num => {
        val startdate = sdf.parse(num._1)
        val lastdate = sdf.parse(num._2)
        val differtime = (lastdate.getTime - startdate.getTime) / (60 * 1000)
        if (differtime <= 10) {
          (num._1, num._2, num._3, number)
        }
        else {
          //怎么定义最后一个值,这个值是唯一的
          number += 1L
          (num._1, num._2, num._3, System.currentTimeMillis() + number)
        }
      })
    })
    //重新确定key的值
    val map1rdd = flatmaprdd.map {
      case (x, (y, z, p, q)) => ((x, q), (y, z, p))
    }
    map1rdd.collect().foreach(println)

    //使用reducebykey获取sum , start last
    map1rdd.reduceByKey((x, y) => {
      var sum = x._3.toInt + y._3.toInt
      var first = if (x._1 < y._1) x._1 else y._1
      var last = if (x._2 > y._2) x._2 else y._2
      (first, last, sum)
    }
    ).map(data => (data._1._1, data._2._1, data._2._2, data._2._3)).sortBy(data => data).collect().foreach(println)
    //flatmaprdd.collect().foreach(println)
    sc.stop()

  }
}

```
```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

//2,2020/2/18 16:32,2020/2/18 16:40,40
//2,2020/2/18 16:44,2020/2/18 17:40,50
//3,2020/2/18 14:39,2020/2/18 15:35,20
//3,2020/2/18 15:36,2020/2/18 15:55,30
//数据已经排序过
object Test3 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName)
    if (args(0).toBoolean) {
      conf.setMaster("local[*]")
    }
    val sc = new SparkContext(conf)
    val rdd = sc.textFile(args(1))
    //首先把数据格式化
    val groupbykeyrdd = rdd.mapPartitions(txt => {
      txt.map(data => {
        val traffic = data.split(",")
        val id = traffic(0)
        val starttime = traffic(1)
        val lasttime = traffic(2)
        val flux = traffic(3).toInt
        (id, (starttime, lasttime, flux))
      })
    }).groupByKey()

    val flatrdd: RDD[((String, Int), (Long, Long, Int))] = groupbykeyrdd.flatMapValues(data => {
      val sdf = new SimpleDateFormat("yyyy/MM/dd HH:mm")
      //考虑使用一个temp变量来接收上一个lasttime
      //考虑使用flag变量来作为标签,如果时间戳相减<10min就为0 否则就为1
      //考虑使用一个sum变量来作为分组的标准
      val sorted = data.toList.sorted
      var temp = 0L
      var flag = 0
      var sum = 0
      sorted.map(txt => {
        val startdate = sdf.parse(txt._1).getTime
        val lastdate = sdf.parse(txt._2).getTime
        //怎么处理第一个值
        if (temp != 0L) {
          if ((startdate - temp) / (60 * 1000) < 10) {
            flag = 0
          }
          else {
            flag = 1
          }
        }
        sum += flag
        temp = lastdate
        (startdate, lastdate, txt._3, sum)
      })
    }).map {
      case (x, (y, z, p, q)) => ((x, q), (y, z, p))
    }
   val mypartition = new Mypartition(flatrdd.partitions.length)
    val value: RDD[((String, Int), (Long, Long, Int))] = flatrdd.reduceByKey(mypartition,(x, y) => {
      var firsttime = Math.min(x._1, y._1)
      var lasttime = Ordering[Long].max(x._2, y._2)
      var sum = x._3 + y._3
      (firsttime, lasttime, sum)
    })

  val result=  value.mapPartitions(txt => {
      val sdf = new SimpleDateFormat("yyyy/MM/dd HH:mm")
      txt.map(data  => {
        val time1: String = sdf.format(data._2._1)
        val time2: String = sdf.format(data._2._2)
        (data._1._1,time1,time2,data._2._3)
      })
    }).map(data => ((data._1,data._2),(data._3,data._4))).sortByKey()
    result.collect().foreach(println)
    Thread.sleep(Int.MaxValue)
    sc.stop()
  }
}
class  Mypartition(partitonlengh: Int) extends  Partitioner{
  override def numPartitions: Int =  partitonlengh
  override def getPartition(key: Any): Int = {
        key.asInstanceOf[((String, Int))]._1.toInt  / partitonlengh
  }
}
//关于排序如果不借用shuffle算子,就只能在内存中排序了,对于每一个分区有不同组的数据,我的最终目的是对组的排序,我的方法是在内存中排序只能选择重写分区器,一个分组就是一个分区,但是这样会造成task的数量过多,借助shuffle算子groupbykey同样能达成效果

```

##3.店铺月份金额(怎么优化排序)
(使用分组->mapvalue->使用一个变量结构sum值)
(使用分区->每个id一个区->使用自定义partititioner->mappartition取出每个分区的数据)
需求:按照月份分组,按照月份进行累加
```
//数据如下类型,需求获取每个月份的总金额并且迭代增加
//1,2020-01-02,20
//1,2020-01-03,20
//1,2020-01-04,20
object Test1 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName)
    if (args(0).toBoolean) {
      conf.setMaster("local[*]")
    }
    val sc = new SparkContext(conf)
    //数据的结构: ((1,2020-1),20)
    val rdd = sc.textFile(args(1))
    val maprdd = rdd.map(data => {
      val txt = data.split(",")

      val shopid = txt(0)
      val date = txt(1).split("-")
      val turnover = txt(2).toInt
      ((shopid, (date(0) + "-" + date(1))), turnover)
    })
    val reducebyrdd: RDD[((String, String), Int)] = maprdd.reduceByKey(_ + _)
    //是否可以避免掉??
    val re = reducebyrdd.map({
      case ((x, y), z) => ((x), (y, z))
    }).groupByKey().map(data => {
      //使用一个变量来接收一个累加值
      val sorted: List[(String, Int)] = data._2.toList.sorted
      var sum = 0
      sorted.map(num => {
        sum += num._2
        (data._1, (num._1, sum))
      })
    })
    re.collect().foreach(println)
    sc.stop()
  }
}
```
