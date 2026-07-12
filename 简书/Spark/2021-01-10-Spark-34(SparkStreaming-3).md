>聚类运算保证exactlyonce语义 (mysql版)


##1.聚类运算
把偏移量写入到mysql,如果在executor写入,那么是多个task一起写入,所以是同时维护多个task的事务
由于是聚类运算,那么考虑把偏移量\偏移量写入在diver端执行
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ba27b4525203e735.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
计算成功就提交事务
失败则回滚事务(计算结果回滚,偏移量回滚,然后重启,仅接着上一次的偏移量重新读取)

##2.mysql事务回顾
测试如下:
事务分为手动提交和自动提交
自动提交是不精确的
手动提交:测试:设置表count的count字段为主键(不允许重复),测试重复提交相同的主键和 在两次提交中增加 var x = 1/0 ,我们会发现在执行到这个部分的时候,直接到 catch ,catch 中有事务的回滚,导致上面的结果无法提交
补充:在同一个database中,多个表之间的提交也支持事务的操作
```
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

##3.把数据和偏移量写入到mysql中来保证exactlyonce

德鲁伊连接池工具
```
object Druid {
  val properties = new Properties()
   properties.load(new FileInputStream(new File("src/main/resources/properties") ))
  val source: DataSource = DruidDataSourceFactory.createDataSource(properties)

  def getConnection(): Connection = {
    val con :Connection  = null
    try {
      source.getConnection
    } catch {
      case  exception: Exception=> exception.printStackTrace()
    }
    con
  }
}
```
//注意是你的数据库
```
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost/db_demo2
username=root
password=123456
filters=stat
initialSize=2
maxActive=300
maxWait=60000
timeBetweenEvictionRunsMillis=60000
minEvictableIdleTimeMillis=300000
validationQuery=SELECT 1
testWhileIdle=true
testOnBorrow=false
testOnReturn=false
poolPreparedStatements=false
maxPoolPreparedStatementPerConnectionSize=200
```

**auto.offset.reset值含义解释**
earliest 
当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费 
latest 
当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据 
none 
topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常

**代码**
```
import java.sql.{Connection, DriverManager, PreparedStatement}

import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka010.{ConsumerStrategies, HasOffsetRanges, KafkaUtils, LocationStrategies}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object Mysql_Set {
  def main(args: Array[String]): Unit = {
    var APPname = "rby_case"
    var topic_name: String = "first"
    var group_id = "G1"
   //Wordcount案例

    val ssc = new StreamingContext(new SparkConf().setMaster("local[*]").setAppName(APPname), Seconds(5))
    ssc.sparkContext.setLogLevel("WARN")
    val topic = Array("first")
    //配置kafka的参数
    val kafkaParams = Map[String, Object](
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
      ConsumerStrategies.Subscribe[String, String](topic, kafkaParams) //消费策略
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

**mysql  偏移量表和数据表**
```
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


##4.不足   下次再次重启,没有读取偏移量,改进一下
**获取历史偏移量**
```
//获取历史的偏移量
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

**在原来代码的基础上添加读取历史偏移量的操作**
```
   //读取历史偏移量
     val historyOffset: Map[TopicPartition, Long] = OffsetUtils.queryHistoryOffsetFromMySQL(APPname, group_id)

    //配置kafka的参数
    val kafkaParams = Map[String, Object](
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

```
