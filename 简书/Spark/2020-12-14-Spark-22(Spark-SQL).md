##1.数据的加载与保存
```
object Test1 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val spark = SparkSession.builder().config(conf).getOrCreate()
    //TODO 读取
    //spark.read.format("json").load("src/main/resources/1.json").show()
    //TODO 读取 CSV文件
     spark.read.format("csv").option("inferSchema", true.toString).option("header", "true")
      .load("src/main/resources/data1.csv").show()
    //val frame = spark.sql("select * from json.`src/main/resources/1.json`")
    //TODO 写入 mode类型
    //frame.write.format("json").mode("overwrite").save("src/main/resources/2.json")
    Thread.sleep(Int.MaxValue)
  }
}
```

##2.sparkSQL与mysql
```
object Test2 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val spark = SparkSession.builder().config(conf).getOrCreate()
    val frame: DataFrame = spark.read.format("jdbc")
      .option("url", "jdbc:mysql://localhost/db_demo4?useUnicode=true&characterEncoding=utf-8")
      .option("dbtable", "logs")
      .option("user", "root")
      .option("password", "123456")
      .load()

    frame.write.format("jdbc")
      .option("url", "jdbc:mysql://localhost/db_demo4?useUnicode=true&characterEncoding=utf-8")
      .option("dbtable", "logs1")
      .option("user", "root")
      .option("password", "123456")
      .mode("append")
      .save()
  }
}
```
```
object Test1 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val spark = SparkSession.builder().config(conf).getOrCreate()
    // spark.sql("select  date_format('2020-2-01 20:20' ,'yyyy-MM-dd HH:mm');").show()
    val schema = new StructType(Array(
      StructField("id", DataTypes.IntegerType),
      StructField("start_time", DataTypes.StringType),
      StructField("last_time", DataTypes.StringType),
      StructField("flow", DataTypes.DoubleType),
    ))
    val frame = spark.read.format("csv")
      .schema(schema)
      .option("header", true.toString)
      .load("src/main/resources/data1.csv")
      frame.write.format("jdbc").option("url", "jdbc:mysql://localhost/db_demo4?useUnicode=true&characterEncoding=utf-8")
        .option("dbtable", "tb_data2")
        .option("user", "root")
        .option("password", "123456").save()
  }
}
```

```
select
id,min(starttime),max(lasttime),sum(flow)
from
(
select
id, starttime, lasttime, flow,
sum(number1) over( partition by id order by starttime  rows between unbounded preceding and current row) as timeorder
from (
select
id, starttime, lasttime, flow, if(number<10,0,1) as number1
from
(
select
id, starttime, lasttime, flow, flagtime ,if(flagtime=0,0,(UNIX_TIMESTAMP(lasttime,'yyyy/M/dd HH:mm')- UNIX_TIMESTAMP(starttime,'yyyy/M/dd HH:mm'))/60) as number
from (
select
*,
lag(lasttime,1,0) over (partition by id order by lasttime) as flagtime
from tb_data1 ) t1 ) t2 ) t3  ) t4
group by  id,timeorder;
```
