> 聚类运算Redis版 
> 非聚类运算 hbase版 Phoenix版求最大值

Redis单机版是支持事务的,但是集群版是不支持事务的

## 回顾Redis的事务操作
```

object JedisTransactionTest {

  def main(args: Array[String]): Unit = {

    //数据库连接
    val jedis = new Jedis("linux03", 6379)
    jedis.auth("123456")

    //开启jedis的pipeline（整体来执行）
    var pipeline: Pipeline = null
    try {
      pipeline = jedis.pipelined()
      //开启多个操作
      pipeline.multi()

      pipeline.hincrBy("wc", "aaa", 18)
      //Redis Hincrby 命令用于为哈希表中的字段值加上指定增量值。
      //val i = 1 / 0
      pipeline.hset("dot", "bbb", "19")
      //将哈希表 key 中的域 field 的值设为 value 。
      //如果 key 不存在，一个新的哈希表被创建并进行HSET 操作。
      //如果域 field 已经存在于哈希表中，旧值将被覆盖
      //提交事务
      pipeline.sync()
      pipeline.exec()
    } catch {
      case e: Exception => {
        e.printStackTrace()
        pipeline.discard()
        //回滚事务
      }
    } finally {
      pipeline.close()
      jedis.close()
    }
  }
}

```
## Ridis版聚类运算
```
//连接池
object JedisConnectionPool {

  val config = new JedisPoolConfig()
  config.setMaxTotal(5)
  config.setMaxIdle(2)
  config.setTestOnBorrow(true)

  val pool = new JedisPool(config, "linux03", 6379, 10000, "123456")

  def getConnection(): Jedis = {
    pool.getResource
  }

}
```
```
//获取历史偏移量

object OffsetUtils {

  def queryHistoryOffsetFromRedis(appName: String, groupId: String): Map[TopicPartition, Long] = {
    val historyOffset = new mutable.HashMap[TopicPartition, Long]()
    val jedis = JedisConnectionPool.getConnection()
    val res: util.Map[String, String] = jedis.hgetAll(appName + "_" + groupId)
    import scala.collection.JavaConverters._

    for (t <- res.asScala) {
      val tp = t._1
      val offset = t._2.toLong
      val fields = tp.split("_")
      val topic = fields(0)
      val partition = fields(1).toInt
      val topicPartition = new TopicPartition(topic, partition)
      historyOffset.put(topicPartition, offset)
    }
    jedis.close()


    historyOffset.toMap
  }
}
```
```
//保证ExactlyOnce，精确一次性语义（要求数据处理且被处理一次）
object KafkaWordCountToRedis {

  def main(args: Array[String]): Unit = {

    val appName = this.getClass.getSimpleName
    val groupId = "g008"
    val conf = new SparkConf().setAppName(appName).setMaster("local[*]")
    val ssc = new StreamingContext(conf, Seconds(5))
    ssc.sparkContext.setLogLevel("WARN")

    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> "linux03:9092,linux04:9092,linux05:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> groupId,
      "auto.offset.reset" -> "earliest",
      "enable.auto.commit" -> (false: java.lang.Boolean) //不自动提交偏移量
    )

    val topics = Array("wordcount18")

    val historyOffset: Map[TopicPartition, Long] = OffsetUtils.queryHistoryOffsetFromRedis(appName, groupId)


    //直连：sparkstreaming用来读取数据的task（kafka的消费者），直接连接到kafka指定topic的leader分区中
    val lines: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream(
      ssc,
      LocationStrategies.PreferConsistent, //位置策略
      ConsumerStrategies.Subscribe[String, String](topics, kafkaParams, historyOffset) //消费策略
    )

    //调用完createDirectStream，返回的第一个DStream进行操作，因为只有第一手的DStream有偏移量
    lines.foreachRDD(rdd => {
      //在指定的时间周期内，kafka中有新的数据写入
      if (!rdd.isEmpty()) {
        //获取偏移量
        val offsetRanges: Array[OffsetRange] = rdd.asInstanceOf[HasOffsetRanges].offsetRanges


        val lines: RDD[String] = rdd.map(_.value())

        val reduced: RDD[(String, Int)] = lines.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)

        //将计算好的结果收集到Driver端，然后写入到Redis
        val res: Array[(String, Int)] = reduced.collect()


        val jedis = JedisConnectionPool.getConnection()
        var pipeline: Pipeline = null
        try {
          pipeline = jedis.pipelined()
          pipeline.multi()

          //目的：是为了实现ExactlyOnce，将计算好的结果和偏移量在同一个事物中同时写入到Redis
          res.foreach(t => {
            pipeline.hincrBy("dot", t._1, t._2)
          })

          offsetRanges.foreach(o => {
            val topic = o.topic
            val partition = o.partition
            val untilOffset = o.untilOffset
            pipeline.hset(appName + "_" + groupId, topic + "_" + partition, untilOffset.toString)
          })

          pipeline.exec()
          pipeline.sync()
        } catch {
          case e: Exception => {
            e.printStackTrace()
            pipeline.discard()
          }
        } finally {
          pipeline.close()
          jedis.close()
        }

      }
    })

    ssc.start()
    ssc.awaitTermination()

  }
}
```
##非聚类运算
hbase 只支持行级事务
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c93e6edd1aab2f9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

数据在kafka中已经有唯一的id,可作为hbase中的rowkey 
每一行的最后一个数据记录偏移量,即可无需读取,查找以一个组内的最大的偏移量即可
但是hbase是不支持分组查询的
解决方案:
1.自定义协处理器
2.使用Phoenix来SQL查询
**如果消费者多读了,但是hbase的id是唯一的,所以会把先前的数据给覆盖掉**

```
  //每一次启动该程序，都要从Hbase查询历史偏移量
  def queryHistoryOffsetFromHbase(appid: String, groupid: String): Map[TopicPartition, Long] = {

    val offsets = new mutable.HashMap[TopicPartition, Long]()

    val connection = DriverManager.getConnection("jdbc:phoenix:linux03,linux04,linux05:2181")

    //分组求max，就是求没有分分区最（大）新的偏移量
    val ps = connection.prepareStatement("select \"topic_partition\", max(\"offset\") from \"t_orders\" where \"appid_groupid\" = ? group by \"topic_partition\"")

    ps.setString(1, appid + "_" + groupid)

    //查询返回结果
    val rs: ResultSet = ps.executeQuery()

    while(rs.next()) {

      val topicAndPartition = rs.getString(1)

      val fields = topicAndPartition.split("_")
      val topic = fields(0)
      val partition = fields(1).toInt

      val offset = rs.getLong(2)

      offsets.put(new TopicPartition(topic, partition), offset)

    }

    offsets.toMap
  }
```
```
/**
  * https://www.jianshu.com/p/f1340eaa3e06
  *
  * spark.task.maxFailures
  * yarn.resourcemanager.am.max-attempts
  * spark.speculation
  *
  * create view "t_orders" (pk VARCHAR PRIMARY KEY, "offset"."appid_groupid" VARCHAR, "offset"."topic_partition" VARCHAR, "offset"."offset" UNSIGNED_LONG);
  * select max("offset") from "t_orders" where "appid_groupid" = 'g1' group by "topic_partition";
  *
  */
object KafkaToHbase {

  def main(args: Array[String]): Unit = {


    //true a1 g1 ta,tb
    val Array(isLocal, appName, groupId, allTopics) = args

    val conf = new SparkConf()
      .setAppName(appName)

    if (isLocal.toBoolean) {
      conf.setMaster("local[*]")
    }

    val sc = new SparkContext(conf)
    sc.setLogLevel("WARN")

    val ssc: StreamingContext = new StreamingContext(sc, Milliseconds(5000))

    val topics = allTopics.split(",")

    //SparkSteaming跟kafka整合的参数
    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> "linux03:9092,linux04:9092,linux05:9092",
      "key.deserializer" -> classOf[StringDeserializer].getName,
      "value.deserializer" -> "org.apache.kafka.common.serialization.StringDeserializer",
      "group.id" -> groupId,
      "auto.offset.reset" -> "earliest", //如果没有记录偏移量，第一次从最开始读，有偏移量，接着偏移量读
      "enable.auto.commit" -> (false: java.lang.Boolean) //消费者不自动提交偏移量
    )

    //查询历史偏移量【上一次成功写入到数据库的偏移量】
    val historyOffsets: Map[TopicPartition, Long] = OffsetUtils.queryHistoryOffsetFromHbase(appName, groupId)

    //跟Kafka进行整合，需要引入跟Kafka整合的依赖
    //createDirectStream更加高效，使用的是Kafka底层的消费API，消费者直接连接到Kafka的Leader分区进行消费
    //直连方式，RDD的分区数量和Kafka的分区数量是一一对应的【数目一样】
    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](
      ssc,
      LocationStrategies.PreferConsistent, //调度task到Kafka所在的节点
      ConsumerStrategies.Subscribe[String, String](topics, kafkaParams, historyOffsets) //指定订阅Topic的规则, 从历史偏移量接着读取数据
    )

//    var offsetRanges: Array[OffsetRange] = null
//
//    val ds2 = kafkaDStream.map(rdd => {
//      //获取KakfaRDD的偏移量(Driver)
//      offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
//      rdd.value()
//    })

    kafkaDStream.foreachRDD(rdd => {

      if (!rdd.isEmpty()) {

        //获取KakfaRDD的偏移量(Driver)
        val offsetRanges: Array[OffsetRange] = rdd.asInstanceOf[HasOffsetRanges].offsetRanges

        //获取KafkaRDD中的数据
        val lines: RDD[String] = rdd.map(_.value())

        val orderRDD: RDD[Order] = lines.map(line => {
          var order: Order = null
          try {
            order = JSON.parseObject(line, classOf[Order])
          } catch {
            case e: JSONException => {
              //TODO
            }
          }
          order
        })
        //过滤问题数据
        val filtered: RDD[Order] = orderRDD.filter(_ != null)

        //调用Action
        filtered.foreachPartition(iter => {
          if(iter.nonEmpty) {
            //将RDD中每一个分区中的数据保存到Hbase中
            //创建一个Hbase的连接
            val connection: Connection = HBaseUtil.getConnection("linux03,linux04,linux05", 2181)
            val htable = connection.getTable(TableName.valueOf("t_orders"))

            //定义一个ArrayList，并且指定长度，用于批量写入put
            val puts = new util.ArrayList[Put](100)

            //可以获取当前Task的PartitionID，然后到offsetRanges数组中取对应下标的偏移量，就是对应分区的偏移量
            val offsetRange: OffsetRange = offsetRanges(TaskContext.get.partitionId()) //在Executor端获取到偏移量

            //遍历分区中的每一条数据
            iter.foreach(order => {
              //取出想要的数据
              val oid = order.oid
              val money = order.money
              //封装到put中
              val put = new Put(Bytes.toBytes(oid))
              //data列族，offset列族
              put.addColumn(Bytes.toBytes("data"), Bytes.toBytes("money"), Bytes.toBytes(money))
              //put.addColumn(Bytes.toBytes("data"), Bytes.toBytes("province"), Bytes.toBytes(province))

              //分区中的最后一条，将偏移量区出来，存到hbase的offset列族
              if(!iter.hasNext) {
                put.addColumn(Bytes.toBytes("offset"), Bytes.toBytes("appid_groupid"), Bytes.toBytes(appName + "_" + groupId))
                put.addColumn(Bytes.toBytes("offset"), Bytes.toBytes("topic_partition"), Bytes.toBytes(offsetRange.topic + "_" + offsetRange.partition))
                put.addColumn(Bytes.toBytes("offset"), Bytes.toBytes("offset"), Bytes.toBytes(offsetRange.untilOffset))
              }
              //将封装到的put添加到puts集合中
              puts.add(put)
              //满足一定大小，批量写入
              if(puts.size() == 100) {
                htable.put(puts)
                puts.clear() //清空puts集合
              }
            })
            //将不满足批量写入条数的数据在写入
            htable.put(puts)
            //关闭连接
            htable.close()
            connection.close()
          }
        })

      }

    })

    ssc.start()

    ssc.awaitTermination()

  }
}
```
