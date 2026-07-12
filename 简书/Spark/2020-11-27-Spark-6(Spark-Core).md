>spark练习题
>处理数据上的分组和业务需求上的分组
##1.案例topN(要点使用模式匹配重新分组)

```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
//需求：统计出每一个省份广告被点击次数的TOP3
//数据的示例:
//1516609143869 4 6 1 11
//1516609143869 3 6 49 7
//1516609143869 8 3 4 18
//1516609143869 8 8 69 14
//1516609143869 0 6 51 29
//1516609143869 5 3 59 2
//1516609143869 8 4 66 25

//需求:时间戳 省份 用户 城市 广告
object Test7 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test7").setMaster("local[*]"))
    val rdd = sc.textFile("src/main/resources/top.txt")
    //由于shuffle传输的字节越多性能越差,只需要需求的数据
    val maprdd: RDD[((String, String), Int)] = rdd.map(data => {
      val th = data.split(" ")
      //结构为 ((省份,城市),广告数)
      ((th(1), th(3)), th(4).toInt)
    })
    //使用效率高的reducebykey,求取出(省份,城市)分组内的所有广告数
    val reducerdd: RDD[((String, String), Int)] = maprdd.reduceByKey(_ + _)
    //使用case 模式匹配重新分组
    val mapardd = reducerdd.map(data => data match {
      case ((province, city), add) => (province, (city, add))
    })
    val grouprdd: RDD[(String, Iterable[(String, Int)])] = mapardd.groupByKey()
    //获取前三
    val result: RDD[(String, List[(String, Int)])] = grouprdd.mapValues(data => data.toList.sortBy(_._2)(Ordering.Int.reverse).take(3))
    result.collect.foreach(println)
    sc.stop()
  }
}
```
##2.基础练习题(过滤求和,最值问题,平均值的多解法效率,join)
```
12 宋江 25 男 chinese 50
12 宋江 25 男 math 60
12 宋江 25 男 english 70
12 吴用 20 男 chinese 50
12 吴用 20 男 math 50
12 吴用 20 男 english 50
12 杨春 19 女 chinese 70
12 杨春 19 女 math 70
12 杨春 19 女 english 70
13 李逵 25 男 chinese 60
13 李逵 25 男 math 60
13 李逵 25 男 english 70
13 林冲 20 男 chinese 50
13 林冲 20 男 math 60
13 林冲 20 男 english 50
13 王英 19 女 chinese 70
13 王英 19 女 math 80
13 王英 19 女 english 70
```

**1.一共有多少个小于20岁的人参加考试？**
```
 val result = rdd.map(data => data.split(" ")).filter(x => x(2).toInt < 20).groupBy(_(1)).count()
```
**2.一共有多少个女生参加考试？**
```
 val result = rdd.map(_.split(" ")).filter(_ (3) == "男").groupBy(_(1)).count()
```
**3.语文科目的平均成绩是多少？**
```
    val sc = new SparkContext(new SparkConf().setAppName("test1").setMaster("local[*]"))
    val rdd = sc.textFile("src/main/resources/sanguo.txt")
    rdd.map(data => {
      val number = data.split(" ")
      ((number(4), number(5).toInt))
    }).filter(_._1 == "chinese").aggregateByKey((0, 0))(
      (u, v) => ((u._1 + v), u._2 + 1),
      (x, x1) => ((x._1 + x1._1), (x._2 + x1._2))
    ).mapValues(data => data._1 / data._2).collect.foreach(println)
    sc.stop()
```
**4.13班数学最高成绩是多少**
```
object Test2 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test1").setMaster("local[*]"))
    val rdd = sc.textFile("src/main/resources/sanguo.txt")
    val strings: Int = rdd.map(_.split(" ")).filter(_ (0) == "13").filter(_ (4) == "math").map(_ (5).toInt).max()
      //.sortBy(_.toInt,false).take(1)
    println(strings)
    sc.stop()
  }
}
```
**5.总成绩大于150分的12班的女生有几个？**
```
object Test3 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test1").setMaster("local[*]"))
    val rdd = sc.textFile("src/main/resources/sanguo.txt")
    val filtrdd: RDD[Array[String]] = rdd.map(_.split(" ")).filter(data => data(0).equals("12") && data(3).equals("女"))
    val rerdd: RDD[(String, Int)] = filtrdd.map(data => (data(1), data(5).toInt)).reduceByKey(_ + _)
    rerdd.collect.foreach(println)
    val result: Long = rerdd.filter(_._2 > 150).count()
    println(result)
    sc.stop()
  }
}
```
**6.总成绩大于150分，且数学大于等于70，且年龄大于等于19岁的学生的平均成绩是多少？**

```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

//总成绩大于150分，且数学大于等于70，且年龄大于等于19岁的学生的平均成绩是多少？
object Test4 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test1").setMaster("local[*]"))
    val data = sc.textFile("src/main/resources/sanguo.txt")
    //首先求取总成绩大于150的姓名和总成绩
    val mapdd = data.map(txt => {
      val strings = txt.split(" ")
      (strings(1), (strings(4),strings(5).toInt))
    })
    //获取总成绩和科目数
    val aggrdd: RDD[(String, (Int, Int))] = mapdd.aggregateByKey((0, 0))(
      (u, v) => ((u._1 + v._2), u._2 + 1),
      (x, x1) => ((x._1 + x1._1), x._2 + x1._2)
    )
    //得到的是总成绩大于150的姓名和平均值
    val re1: RDD[(String, Int)] = aggrdd.filter(_._2._1 > 150).mapValues(data => data._1 / data._2)
    //(王英,73)
    //(宋江,60)
    //(杨春,70)
    //(李逵,63)
    //(林冲,53)

    //且数学大于等于70，且年龄大于等于19岁的学生
    val maprdd: RDD[Array[String]] = data.map(_.split(" "))
    val filtrdd: RDD[Array[String]] = maprdd.filter(data => data(2).toInt >= 19 && data(4).equals("math") && data(5).toInt >= 70)
    val re2: RDD[(String, Int)] = filtrdd.map(data => {
      (data(1), data(5).toInt)
    })
    //join连接
    val result: RDD[(String, Int)] = re1.join(re2).map(data => {
      (data._1, data._2._1)
    })
    result.collect.foreach(println)
    sc.stop()
    //(王英,73)
    //(杨春,70)
  }
}
```
