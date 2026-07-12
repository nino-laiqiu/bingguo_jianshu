> SQL的结构 
udf udaf的简单使用

##1.结构的学习
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9dc5d919d1594c5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2330304e9092af9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

问题:RDD DataFrame DataSet之间的关系

**转换操作:**
```
object Test1 {
  def main(args: Array[String]): Unit = {
    //Todo 环境
    val conf = new SparkConf().setAppName("SQL1").setMaster("local[*]")
    val session = SparkSession.builder().config(conf).getOrCreate()
    import session.implicits._
    //todo DataFrame执行
    //1.DataFrame => sql
    val dsql = session.read.json("src/main/resources/1.json")
    //dsql.show()
    // dsql.createOrReplaceTempView("user")
    //session.sql("select * from user").show()
    //2.DataFrame => DSL
    // dsql.select("name").show()
    //dsql.select($"name").show()
    // dsql.select('age + 1,'age).show()
    //TODO DataSet执行
    //val seq = Seq(1, 2, 3, 4, 5)
    //val ds = seq.toDS()
    //ds.show()
    //TODO RDD => DataFrame
    val rdd = session.sparkContext.makeRDD(List(("小明", 12, 2000), ("小华", 14, 9000)))
    val frame: DataFrame = rdd.toDF("name", "age", "money")
    //val rdd1: RDD[Row] = frame.rdd
    //frame.show()
    //rdd1.foreach(println)
    //TODO DataFrame => DataSet
    // val value: Dataset[User] = frame.as[User]
    // value.show()
    // val frame1: DataFrame = value.toDF()
    //TODO  RDD => DataSet
    val value1: Dataset[User] = rdd.map(data => data match {
      case (x, y, z) => User(x, y, z)
    }).toDS()
    // value1.show()
    // val rdd2: RDD[User] = value.rdd
    session.stop()
  }
}

case class User(name: String, age: Int, money: Double)
```

#2.UDF函数
```
//简单的UDF函数
object Test2 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val spark = SparkSession.builder().config(conf).getOrCreate()
    val framejson = spark.read.json("src/main/resources/1.json")
    framejson.createOrReplaceTempView("user")
    spark.udf.register("udfname",(name:String) => {
         "Name:" + name
    })
    spark.sql("select udfname(name) , age from user").show()
    spark.stop()
  }
}
```

##3.UDAF
**弱类型实现**
```
object Spark_Udaf1 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val spark = SparkSession.builder().config(conf).getOrCreate()
    val frame = spark.read.json("src/main/resources/1.json")
    frame.createOrReplaceTempView("user")
    spark.udf.register("avgAge", new MyAvg)
    val sql: String = "select avgAge(age) from user"
    spark.sql(sql).show()
    Thread.sleep(Int.MaxValue)
    spark.stop()
  }
}

//统计年龄的总和,统计数据的条数
class MyAvg extends UserDefinedAggregateFunction {
  //输入数据的类型
  override def inputSchema: StructType = {
    StructType(Array(StructField("age", LongType)))
  }

  //缓冲区的类型
  override def bufferSchema: StructType = {
    //第一个值:年龄的总和  第二个值:条数的总和
    StructType(Array(StructField("total", LongType), StructField("count", LongType)))
  }

  //输出的数据的类型
  override def dataType: DataType = LongType

  //函数是否稳定
  override def deterministic: Boolean = true

  //缓冲区初始数据
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0) = 0L
    buffer(1) = 0L
  }

  //更新缓冲区
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    buffer(0) = buffer.getLong(0) + input.getLong(0)

    buffer.update(1, buffer.getLong(1) + 1)
    //上一个相当于下一个,底层调用了update方法
    println(buffer.getLong(1))
  }

  //缓冲区的合并,存在多个缓存区
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    //todo (x,y) =>第一个值是初始值或者中间值
    buffer1.update(0, buffer1.getLong(0) + buffer2.getLong(0))
    buffer1.update(1,buffer1.getLong(1) + buffer2.getLong(1))
  }

  //输出数据的类型
  override def evaluate(buffer: Row): Any = {
    println(buffer.getLong(0))
    println(buffer.getLong(1))
    buffer.getLong(0) / buffer.getLong(1)
  }
}
```
**强类型实现**
```
object Spark_Udaf2 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val spark = SparkSession.builder().config(conf).getOrCreate()
    val frame = spark.read.json("src/main/resources/1.json")
    frame.createOrReplaceTempView("user")
    spark.udf.register("myavg", functions.udaf(new MyAvg1))
    spark.sql("select myavg(age) from user").show()
    spark.stop()
  }
}
case class Buff(var total: Long, var count: Long)

class MyAvg1 extends Aggregator[Long, Buff, Long] {
  override def zero: Buff = {
    Buff(0L, 0L)
  }
  //根据输入的数据更新缓冲区数据
  override def reduce(buff: Buff, input: Long): Buff = {
    buff.total = buff.total + input
    buff.count = buff.count + 1
    buff
  }
  //合并多个缓冲区数据
  override def merge(buff1: Buff, buff2: Buff): Buff = {
    buff1.total = buff1.total + buff2.total
    buff1.count = buff1.count + buff2.count
    buff1
  }
  //输出的结果
  override def finish(reduction: Buff): Long = {
    reduction.total / reduction.count
  }
  //下面的两个是序列化
  override def bufferEncoder: Encoder[Buff] = Encoders.product

  override def outputEncoder: Encoder[Long] = Encoders.scalaLong
}
```
