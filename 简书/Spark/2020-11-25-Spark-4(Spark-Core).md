>RDD算子:flatmap  glom  groupby  fliter   sample   distinct  coalesce
  
尚未回答的问题
>为什么shuffle中下游的分区数是由上游决定的,以及如何决定
上游部分聚合完落到hdfs上的数据,上游还会读取吗
如何获取分区的数量
局部聚合是如何完成的
reducebykey和groupbykey的区别
compactbuffer是怎么样的数据结构
对于算子中  shuffleRdd  aggregator方法的重写
去重算子底层是如何实现的
课程中方法重写代码的书写
对案例的解答

## 1.flatmap算子的示例(扁平化)
```
 val sc = new SparkContext(new SparkConf().setAppName("tset").setMaster("local[*]"))
    val rdd: RDD[List[Int]] = sc.makeRDD(List[List[Int]](List(1, 2), List(3, 4)))
    val result: RDD[Int] = rdd.flatMap(data => data)
    result.collect.foreach(println)
    sc.stop()
```
```
   val sc = new SparkContext(new SparkConf().setAppName("tset").setMaster("local[*]"))
    val rdd: RDD[String] = sc.makeRDD(List[String]("haddoop yarn", "spark scala"))
    val result: RDD[String] = rdd.flatMap(data => data.split(" "))
    result.collect.foreach(println)
    sc.stop()
```
```
    val sc = new SparkContext(new SparkConf().setAppName("tset").setMaster("local[*]"))
    val RDD = sc.makeRDD(List(List(1, 2), 3, List(4, 5)))
    //模式匹配
    val result: RDD[Any] = RDD.flatMap {
      case list: List[_] => list
      case num => List(num)
    }
    result.collect.foreach(println)
    sc.stop()
```

##2.glom算子演示(包装为数组形式)

```
  val sc = new SparkContext(new SparkConf().setAppName("tset").setMaster("local[*]"))
    val RDD = sc.makeRDD(List[Int](1, 2, 3, 4, 5, 6))
    val result: RDD[Array[Int]] = RDD.glom().map(data => data)
    result.collect.foreach(data => println(data.mkString(",")))
    sc.stop()
```
```
    //求取分区的最大值的和
    val sc = new SparkContext(new SparkConf().setAppName("test").setMaster("local[*]"))
    val rdd = sc.makeRDD(List[Int](1, 2, 3, 4, 5, 6), 2)
    val glomrdd: RDD[Array[Int]] = rdd.glom()
    val result: RDD[Int] = glomrdd.map(data => {
      data.max
    })
    println(result.collect.sum)
    sc.stop()
```

##3.groupby(key的类型为所判断的类型)
```
    val sc = new SparkContext(new SparkConf().setAppName("test2").setMaster("local[*]"))
    val rdd = sc.makeRDD(List[String]("hadoop,yarn,mapreduce", "java,scala,node.js"), 2)
    val flatrdd: RDD[String] = rdd.flatMap(data => data.split(","))
    //key是 charat(0)
    //  val result: RDD[(Char, Iterable[String])] = flatrdd.groupBy(data => data.charAt(0))
    //key是true和false
    val res: RDD[(Boolean, Iterable[String])] = flatrdd.groupBy(data => data.contains("s"))
    res.collect.foreach(println)
    sc.stop()
```
```
    val sc = new SparkContext(new SparkConf().setAppName("test").setMaster("local[*]"))
    //读取文本获取,时间内的点击量
    val rdd = sc.textFile("src/main/resources/aa.txt")
    val value: RDD[(String, Int)] = rdd.map(data => {
      val str: Array[String] = data.split(" ")
      //获取时间
      val str1 = str(3)
      val format = new SimpleDateFormat("dd/mm/yyyy:HH:MM:SS")
      val str2: Date = format.parse(str1)
      val format1 = new SimpleDateFormat("HH")
      val str3: String = format1.format(str2)
      (str3, 1)
    }).groupBy(_._1).map(data => (data._1, data._2.size))
    value.collect.foreach(println)
    sc.stop()
```
## 4.fliter算子的示例(数据倾斜)
##5.sample算子的示例(伯努利和泊松分布)
withReplacement：表示抽出样本后是否在放回去，true表示会放回去，这也就意味着抽出的样本可能有重复
```
   val sc = new SparkContext(new SparkConf().setAppName("test3").setMaster("local[*]"))
    val RDD = sc.makeRDD(List[Int](1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
    val result = RDD.sample(
      true,
      0.4
    )
    result.collect.foreach(println)
    sc.stop()
  }
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0299fa45dd8dcb83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 6.distinct算子底层与scala中distinct的区别
spark中的去重
```
partitioner match {
      case Some(_) if numPartitions == partitions.length =>
        mapPartitions(removeDuplicatesInPartition, preservesPartitioning = true)
      case _ => map(x => (x, null)).reduceByKey((x, _) => x, numPartitions).map(_._1)
    }
```
scala中的去重(调用的是hashset)
```
default Object distinct() {
      boolean isImmutable = this instanceof scala.collection.immutable.Seq;
      if (isImmutable && this.lengthCompare(1) <= 0) {
         return this.repr();
      } else {
         Builder b = this.newBuilder();
         HashSet seen = new HashSet();
         Iterator it = this.iterator();
         boolean different = false;
```

#7.coalesce(分区)
(缩减分区数,要指定true否则是没有shuffle的,数据倾斜,想要扩大分区就必须指定trueshuffle spark提供了一个repartition算子扩大分区)
```
    val sc = new SparkContext(new 
    SparkConf().setAppName("test3").setMaster("local[*]"))
    val RDD = sc.makeRDD(List[Int](1, 2, 3, 4, 1, 4, 7, 8, 9),3)
    RDD.coalesce(2,true).saveAsTextFile("src/aa")
    sc.stop()
```
##8.repartition(扩大分区)  
