### SparkStreaming入门案例

```scala
//wordcount案例
object WordCount {
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



### SparkStreaming概念

SparkStreaming是一个SparkCore API的一个扩展.底层基于SparkCore,是一个高吞吐可扩展可容错的分布式实时流式计算框架

原理:Driver端将任务划分微批次小任务(周期性)提交到spark计算引擎中执行,任务一直在运行,在sparkcore上对driver端的增强

Dstream可以从很多的数据源创建,对Dstream进行操作,本质上是对Dstream中某个时间点的RDD进行操作,对RDD进行操作,本质上是对每个RDD的分一个分区进行操作,Dstream上的方法也分为Transformation和Action,还有一些方法既不是Transformation也不是Action,比如foreachRDD,调用Transformation后可以生成一个新的Dstream

特点:可通错性 高吞吐 可扩展 丰富的API 可以保证精确一次性语义,但是SparkStreaming是一个微批次准实时的流式计算系统,由于在Driver端定期生成微批次的job,然后在调度到executor中,会有一定的延迟性



### SparkStreaming 整合kafka

#### 方案一:Receiver方式

类似于接自来水,但是如果自来水流速过快就会出现问题,为了保证数据安全性,要将数据写入磁盘记录日志中(write ahead log),效率低.使用kafka老版本的消费API,将偏移量写入到zookeeper中,效率低

#### 方案二:直连kafka

建议使用kafka0.10及以上的版本

一个kafka Topic分区对应一个RDD的分区,即RDD的分区的数量和kafka的分区时一一对应的,生成的task直连kafka Topic的leader分区拉取数据,一个消费者task对应一个kafka的活跃分区

在Driver端调用foreachRDD获取Dstram中的kafkaRDD,传入的foreachRDD,传入的foreachRDD方法的函数在Driver端会周期性的执行.KafkaDstream必须先调用foreachRDD方法或者Tansform,才能获取kafkaRDD,只有kafkaRDD中才有偏移量.kafkaRDD实现了HasoffsetRanger接口,可以获取kafka分区的偏移量,一个批次处理完毕之后,偏移量可以写入到kafak Redis mysql中

#####  自动提交偏移量

      自动提交偏移量到主题 consumer_offsets  比较省事但是假设出现一种情况:消费者消费完数据之后挂掉了,偏移量信息没有来得及更      新,不太准确,会重复消费数据,无法保证数据的一致性

(去掉即可)

```
 "enable.auto.commit" -> (false: java.lang.Boolean) //不让消费者自动提交偏移量
```

```
  inputDStream.asInstanceOf[CanCommitOffsets].commitAsync(offsetRanges)
```



##### 手动提交偏移量

###### 官方提供的异步提交方法

      一个队列实现的commitAsync 但是这会出现如下的问题,例如还没有运算完,就提交了偏移量或者数据处理失败了,但是偏移量却提交了

```scala
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



###### 聚类运算Mysql优化版

```scala
//获取历史的偏移量

import org.apache.kafka.common.TopicPartition
import scala.collection.mutable
object OffsetUtils {
  def queryHistoryOffsetFromMySQL(appName: String, groupId: String): Map[TopicPartition, Long] = {

    val offsetMap = new mutable.HashMap[TopicPartition, Long]()
    var statement: PreparedStatement = null
    var connection: Connection = null
    var resultSet: ResultSet = null
    try {
      //查询语法不需要事务
      connection = Druid.getConnection()
      statement = connection.prepareStatement("select  topic_partition ,offset from db_demo2.kafka_offset where app_gid = ? ")
      statement.setString(1, appName + "_" + groupId)
      resultSet = statement.executeQuery()
      while (resultSet.next()) {
        val topic_partition = resultSet.getString(1)
        val offset = resultSet.getLong(2)
        val strings = topic_partition.split("_")
        val topic = strings(0)
        val partition = strings(1).toInt
        val topicPartition = new TopicPartition(topic, partition)
        offsetMap(topicPartition) = offset

      }
    } catch {
      case exception: Exception => new RuntimeException("查询错误")
    } finally {
      if (resultSet != null) {
        resultSet.close()
      }
      if (statement != null) {
        statement.close()
      }
      if (connection != null) {
        connection.close()
      }

    }
    //返回map
    offsetMap.toMap
  }
}
```



```scala
//mysql的事务回顾
import java.sql.{Connection, Driver, DriverManager, PreparedStatement}

//连接mysql
object Test5 {
  def main(args: Array[String]): Unit = {
    var connection: Connection = null
    var preparedStatement: PreparedStatement = null
    try {
      //mysql的事务时自动提交的,无法保证精确性
      connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/db_demo2", "root", "123456")
      //自动提交事务
      connection.setAutoCommit(false)
      //设置Word为主键,主键不允许重复
      preparedStatement = connection.prepareStatement("insert into count values(?,?)")
      preparedStatement.setString(1, "flink1")
      preparedStatement.setInt(2, 2)
      preparedStatement.executeUpdate()
      var x = 1 / 0
      preparedStatement.setString(1, "scala1")
      preparedStatement.setInt(2, 3)
      preparedStatement.executeUpdate()
      //提交事务
      connection.commit()
    } catch {
      case exception: Exception => exception.printStackTrace()
        //回滚事务
        connection.rollback()
    } finally {
      if (preparedStatement != null) {
        preparedStatement.close()
      }
      if (connection != null) {
        connection.close()
      }
    }
  }
}
```



```sql
-- 数据表和偏移量表
CREATE TABLE db_demo2.kafka_word(
   word varchar(20) PRIMARY KEY,
   count int
);


create table db_demo2.kafka_offset(
      app_gid varchar(20),
      topic_partition varchar(20),
      offset int,
      PRIMARY KEY(app_gid,topic_partition)
);
```



```scala
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.TopicPartition
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka010.{ConsumerStrategies, HasOffsetRanges, KafkaUtils, LocationStrategies}
import org.apache.spark.streaming.{Seconds, StreamingContext}

//在代码的基础上添加读取历史偏移量的操作
object Mysql {
  def main(args: Array[String]): Unit = {
    var APPname = "rby_case"
    var topic_name: String = "first"
    var group_id = "G1"

    val ssc = new StreamingContext(new SparkConf().setMaster("local[*]").setAppName(APPname), Seconds(5))
    ssc.sparkContext.setLogLevel("WARN")
    val topic = Array("first")
    //读取历史偏移量
     val historyOffset: Map[TopicPartition, Long] = OffsetUtils.queryHistoryOffsetFromMySQL(APPname, group_id)

    //配置kafka的参数
    val kafkaParams: Map[String, Object] = Map[String, Object](
      "bootstrap.servers" -> "linux03:9092,linux04:9092,linux05:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> group_id,
      "auto.offset.reset" -> "earliest", //earliest last,为什么要使用earliest
      "enable.auto.commit" -> (false: java.lang.Boolean) //不让消费者自动提交偏移量
    )

    val inputDStream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream(
      ssc,
      LocationStrategies.PreferConsistent, //位置策略
      ConsumerStrategies.Subscribe[String, String](topic, kafkaParams,historyOffset) //消费策略
    )

    inputDStream.foreachRDD(rdd => {
      if (!rdd.isEmpty()) {
        //获取偏移量
        val offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges

        //执行逻辑
        val value = rdd.map(_.value()).flatMap(_.split("\\s+")).map((_, 1)).reduceByKey(_ + _)
        //把计算结果收集到driver端
        val tuples = value.collect()
        //目的：是为了实现ExactlyOnce，将计算好的结果和偏移量在同一个事物中同时写入到MySQL
        //建立mysql连接把数据和偏移量写入到分别mysql的两个表中
        var connection: Connection = null
        var statement: PreparedStatement = null
        var statement1: PreparedStatement = null
        try {
          connection =  DriverManager.getConnection("jdbc:mysql://localhost:3306/db_demo2", "root", "123456")

          //开启事务
          connection.setAutoCommit(false)

          //mysql不支持更新操作,使用ON DUPLICATE KEY UPDATE来实现,此处的count+只适用于本次的reducebykey
          statement = connection.prepareStatement("insert  into  kafka_word(word, count) VALUES (?,?) ON DUPLICATE KEY UPDATE count = count + ?")

          //将计算好的结果写入到mysql中
          tuples.foreach(data => {
            statement.setString(1, data._1)
            statement.setInt(2, data._2)
            statement.setInt(3, data._2)
            //写入到mysql中
            statement.executeUpdate()
            //如果数据过多可以把数据先收藏起来在写入到mysql中
            // statement.addBatch()
          })
          //addBatch循环完之后把把攒起来的数据写入到mysql中
          // statement.executeBatch()

          statement1 = connection.prepareStatement("INSERT INTO kafka_offset (app_gid, topic_partition, offset) VALUES (?,?,?) ON DUPLICATE KEY UPDATE offset = ?")

          for (elem <- offsetRanges) {
            statement1.setString(1, APPname + "_" + group_id)
            statement1.setString(2, elem.topic + "_" + elem.partition)
            statement1.setLong(3, elem.untilOffset)
            statement1.setLong(4, elem.untilOffset)
            statement1.executeUpdate()
          }
          //提交事务
          connection.commit()

        } catch {
          case exception: Exception =>
            //捕获事务
            exception.printStackTrace()
            //要成功都成功,要失败都失败
            connection.rollback() //回滚事务
            //停止这个Application
            ssc.stop(true)
        } finally {
          if (statement1 != null) {
            statement1.close()
          }

          if (statement != null) {
            statement.close()
          }
          if (connection != null) {
            connection.close()
          }
        }
      }
    })
    ssc.start()
    ssc.awaitTermination()
  }
}
```



###### 聚类运算Redis版

```scala
//Redis事务复习

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



```scala
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



```scala
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



```scala
/保证ExactlyOnce，精确一次性语义（要求数据处理且被处理一次）
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



###### 非聚类运算Hbase版
数据在kafka中已经有唯一的id,可作为hbase中的rowkey
每一行的最后一个数据记录偏移量,即可无需读取,查找以一个组内的最大的偏移量即可
但是hbase是不支持分组查询的
解决方案:
1.自定义协处理器
2.使用Phoenix来SQL查询
**如果消费者多读了,但是hbase的id是唯一的,所以会把先前的数据给覆盖掉**
//每一次启动该程序，都要从Hbase查询历史偏移量

```
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
```
```
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
