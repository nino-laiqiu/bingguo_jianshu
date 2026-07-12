>Wordcount入门案例
sparkstreaming直连kafka的注意事项和步骤(自动提交偏移量和有缺陷的手动提交偏移量的方法)
spark checkpoint尽量不要用,效果极差

##1.sparkstreaming Wordcount案例
```
//wordcount案例
object Test3 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Wordcount").setMaster("local[*]")
    val sc = new SparkContext(conf)
    //要传入一个时间间隔
    val ssc = new StreamingContext(sc, Seconds(5))
    ssc.sparkContext.setLogLevel("WARN")
    //val context = new StreamingContext(conf, Seconds(5))
    //底层直接new Sparkcontext  this(StreamingContext.createNewSparkContext(conf), null, batchDuration)
    val value: ReceiverInputDStream[String] = ssc.socketTextStream("localhost", 8888)
    val result: DStream[(String, Int)] = value.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)
    //DStream 是对rdd的分装
    //action
    result.print()
    ssc.start()
    //让程序一直运行
    ssc.awaitTermination()
  }
}
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-92cc7bbf9f1d7eb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.sparkstreaming 整合 kafka
两种方法:第一种类似于接自来水,但是会有一个问题自来水的流速过快怎么办

第二种:直连kafka

自动提交偏移量到主题 consumer_offsets  比较省事但是假设出现一种情况:消费者消费完数据之后挂掉了,偏移量信息没有来得及更新,不太准确,会重复消费数据,无法保证数据的一致性

手动提交偏移量:没有提交偏移量,再次启动会重复读取数据

要点:每个分区都会记录这个分区的行偏移量,周期性的调用RDD,注意偏移量是行偏移量

偏移量的信息如下:
```
topic + first + partition + 0 + from-off-set  + 2 + end-off-set + 2 
topic + first + partition + 1 + from-off-set  + 2 + end-off-set + 3 
```

>kafka官方提供的异步提交方法,是由一个队列实现的commitAsync => 
但是这会出现如下的问题,例如
还没有运算完,就提交了偏移量
或者数据处理失败了,但是偏移量却提交了


>实际上我们总会对DStream进行一些运算，这时我们可以借助DStream的transform()算子
特别需要注意，在转换过程中不能破坏 RDD 分区与 Kafka 分区之间的映射关系。亦即像map() / mapPartitions()这样的算子是安全的，而会引起shuffle或者repartition的算子，如reduceByKey() / join() / coalesce()等等都是不安全的

例如:
```
topic + first + partition + 0 + from-off-set  + 2 + end-off-set + 2 
topic + first + partition + 1 + from-off-set  + 2 + end-off-set + 3 
(777,1)(00,1)(7777,3)(77,1)21/01/08 23:29:02 WARN ProcfsMetricsGetter: Exception when trying to compute pagesize, as a result reporting of ProcessTree metrics is stopped
topic + first + partition + 1 + from-off-set  + 3 + end-off-set + 3 
topic + first + partition + 0 + from-off-set  + 2 + end-off-set + 3 
(iii,2)(00,1)(ii,1)(uu,1)
```


**代码**
```
import com.google.common.eventbus.Subscribe
import org.apache.commons.codec.StringDecoder
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.sql.catalyst.expressions.Second
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010.{CanCommitOffsets, ConsumerStrategies, ConsumerStrategy, HasOffsetRanges, KafkaUtils, LocationStrategies, LocationStrategy}
import org.apache.spark.streaming.{Seconds, StreamingContext, dstream, kafka010}
import org.apache.spark.{SparkConf, SparkContext}
import org.sparkproject.guava.eventbus.Subscribe

//kafka集成
object Test2 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("kafka").setMaster("local[*]"))
    sc.setLogLevel("WARN")
    val ssc = new StreamingContext(sc, Seconds(5))
    ssc.sparkContext.setLogLevel("WARN")
    val preferredHosts = LocationStrategies.PreferConsistent
    //主题,一到多个
    val topic = Array("first")
    //配置kafka的参数
    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> "linux03:9092,linux04:9092,linux05:9092",
      //key value序列化
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "earliest", //earliest last

      // -- 手动提交偏移量
      "enable.auto.commit" -> (false: java.lang.Boolean) //不让消费者自动提交偏移量
    )
    val inputDStream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream(ssc,
      LocationStrategies.PreferConsistent, //位置策略
      // 这个策略会将分区分布到所有可获得的executor上。如果你的executor和kafkabroker在同一主机上的话，可以使用PreferBrokers，
      // 这样kafka leader会为此分区进行调度。最后，如果你加载数据有倾斜的话可以使用PreferFixed，
      // 这将允许你制定一个分区和主机的映射（没有指定的分区将使用PreferConsistent 策略）
      ConsumerStrategies.Subscribe[String, String](topic, kafkaParams) //消费策略
    )
    inputDStream.foreachRDD { rdd =>
      //进行判断是否为空
      if (!rdd.isEmpty()) {
        val offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges

        //查看偏移量信息
        for (elem <- offsetRanges) {
          //注意这种写法
          println(s"topic : ${elem.topic} + partition : ${elem.partition} + from-off-set : ${elem.fromOffset} + end-off-set : ${elem.untilOffset} ")
        }

        // 在实际上我们总会对DStream进行一些运算，这时我们可以借助DStream的transform()算子
        //在这里调用rdd的算子,打印的结果出现在executor

        //Cannot use map-side combining with array keys 测试出现如下问题原因是没有使用flatmap 却使用了 reducebykey
        rdd.map(_.value()).flatMap(_.split("\\s+")).map((_, 1)).reduceByKey(_ + _).foreach(print)
        //在转换过程中不能破坏 RDD 分区与 Kafka 分区之间的映射关系。亦即像map() /
        // mapPartitions()这样的算子是安全的，而会引起shuffle或者repartition的算子，
        // 如reduceByKey() / join() / coalesce()等等都是不安全的

        // 确保结果都已经正确且幂等地输出了
        //在driver端异步的更新偏移量 这个批次然后更新偏移量
        inputDStream.asInstanceOf[CanCommitOffsets].commitAsync(offsetRanges)

      }
    }
    //K,V 类型
    // val value = inputDStream.map(_.value()).flatMap(_.split("\\s+")).map((_, 1)).reduceByKey(_ + _)
    // value.print()
    ssc.start()
    ssc.awaitTermination()
  }
}
```
上面的kafka提供的方法还是不够完美,可以采用写入到mysql或者zookeeper之中

##3.补充:为什么不用checkpoint

Spark Streaming的checkpoint机制无疑是用起来最简单的，checkpoint数据存储在HDFS中，如果Streaming应用挂掉，可以快速恢复
但是，如果Streaming程序的代码改变了，重新打包执行就会出现反序列化异常的问题。这是因为checkpoint首次持久化时会将整个jar包序列化，以便重启时恢复。重新打包之后，新旧代码逻辑不同，就会报错或者仍然执行旧版代码
要解决这个问题，只能将HDFS上的checkpoint文件删掉，但这样也会同时删掉Kafka的offset信息，就毫无意义了。
