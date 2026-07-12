##1.RDD血缘关系
![image.png](https://upload-images.jianshu.io/upload_images/9049859-adc327f640e6c71d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-94a2e3d6b0f66bf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-80fccffa69299a55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c618e7032d638e94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.持久化:cache  persist  checkpoint(检查点)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-69ad21abb4ee5af3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import org.apache.spark.storage.StorageLevel
import org.apache.spark.{SparkConf, SparkContext}

object Test5 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("").setMaster("local[*]"))
    sc.setCheckpointDir("src/ff")
    val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6, 1, 2, 3, 4))
    val mardd = rdd.map(data => {
      //每个collect执行了一次,重复操作
      println("*******")
      (data, 1)
    })
    //考虑持久化
    //只能持戒化到内存
    // mardd.cache()
    //可持久化到磁盘
    // mardd.persist(StorageLevel.DISK_ONLY)
    mardd.checkpoint()
    mardd.groupByKey().collect.foreach(println)
    mardd.reduceByKey(_ + _).collect.foreach(println)
    sc.stop()
  }
}
```

##3.自定义分区器(模式匹配)
```
import org.apache.spark.{Partitioner, SparkConf, SparkContext}

object Test6 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("test6").setMaster("local[*]"))
    val rdd = sc.makeRDD(List("scala", "spark", "scala", "flink"), 2)
    val myrdd = rdd.map(data => (data, 1)).partitionBy(new MyPartitioner)
    myrdd.saveAsTextFile("src/bbb")
    sc.stop()

  }
}

class MyPartitioner extends Partitioner {
  override def numPartitions: Int = 2

  override def getPartition(key: Any): Int = {
    key match {
      case "scala" => 1
      case _ => 0
    }
  }
}
```
##4.累加器

##5.广播变量
