DSL案例

##数据源
JSON:
数据格式丰富,可以嵌套结构
传输,储存了冗余的数据
CSV
储存数据相对紧凑
schema信息不完整,结构比较单一


ORC File文件结构
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a820c9ca2d82d920.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Parquet
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7d0eaf4416c23a0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 理解一行数据(我傻了?)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-05bc54c82c41abfd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

行列分隔符
行分隔符
一行数据就是一个表全部字段的一行数据


# DSL风格
注意1.spark.2不支持的语法 date_sub()  使用expr(date_sub()) 支持spark2x 3x
注意2.lag()开窗.默认值,bug,要使用expr()语法
注意3.groupby之后使用agg(聚合函数) 如果直接使用一个聚合函数则只能用一个函数
注意4.window的静态工厂方法 在控制窗口和窗口大小的应用,例如:rangebetween
注意5.使用drop()来删除不需要的字段,从而避免了select的使用
注意6.别名的使用,$"" ' 的事项

##案例一:连续登陆天数
```
object Test5 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val sprak = SparkSession.builder().config(conf).getOrCreate()
    import sprak.implicits._
    import org.apache.spark.sql.functions._

    val schema = new StructType(Array(
      StructField("id", DataTypes.StringType),
      StructField("start_time", DataTypes.StringType),
    ))
    val frame = sprak.read.format("csv").schema(schema).option("header", false.toString).load("src/main/resources/date1.csv")
    frame.distinct().select(
      'id,
      'start_time,
      dense_rank().over(Window.partitionBy("id").orderBy("start_time")) as ("rn")
    ).select(
      'id,
      'start_time,
      date_sub('start_time, 'rn) as "last_time", //spark.2不支持
      // expr("date_sub(start_time,rn) as last_time")
    ).groupBy(
      'id,
      'last_time
    ).agg(
      min("start_time") as "min_time",
      max("start_time") as "max_time",
      count("id") as "times"
    ).where('times >= 3).show()
  }
}
```

![image.png](https://upload-images.jianshu.io/upload_images/9049859-1abe0dba802b79fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##案例二:上面的
```
object Test4 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val sprak = SparkSession.builder().config(conf).getOrCreate()
    import sprak.implicits._
    import org.apache.spark.sql.functions._
    //
    val schema = new StructType(Array(
      StructField("id", DataTypes.IntegerType),
      StructField("start_time", DataTypes.StringType),
      StructField("last_time", DataTypes.StringType),
      StructField("flow", DataTypes.DoubleType)
    ))
    val frame = sprak.read.format("csv").schema(schema).option("header", true.toString).load("src/main/resources/data1.csv")

    frame.select(
      'id,
      'start_time,
      'last_time,
      'flow,
       expr("lag(last_time,1,start_time) over(partition by id order by start_time) as lag_time")
    ).select(
      'id,
      'start_time,
      'last_time,
      'flow,
      expr("if(to_unix_timestamp(start_time,'yyyy/M/dd HH:mm') - to_unix_timestamp(lag_time,'yyyy/M/dd HH:mm') > 600, 1, 0) as flag")
    ).select(
      'id,
      'start_time,
      'last_time,
      'flow,
      sum($"flag").over(Window.partitionBy($"id").orderBy($"start_time").rangeBetween(Window.unboundedPreceding,Window.currentRow)).as("rn")
    ).groupBy("id","rn")
      .agg(
        min("start_time") as "start_time",
        max("last_time")  as "last_time",
        sum("flow") as "flow"
      ).drop("rn")
      .orderBy("id").show()
  }
}
```

##案例三:实现sum的开窗累加
```
object Test6 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")
    val sprak = SparkSession.builder().config(conf).getOrCreate()
    import sprak.implicits._
    import org.apache.spark.sql.functions._
    val schema = new StructType(Array(
      StructField("id", DataTypes.StringType),
      StructField("start_time", DataTypes.StringType),
      StructField("money", DataTypes.DoubleType),
    ))
    val frame = sprak.read.format("csv").schema(schema).option("header", false.toString).load("src/main/resources/shop.csv")
    frame.select(
      'id,
      substring('start_time, 0, 7) as "time",
      'money
    ).groupBy('id, 'time).agg(sum($"money")as("sum_money"))
     .select(
        'id,
        'time,
        sum($"sum_money").over(Window.partitionBy("id").orderBy("time").rangeBetween(Window.unboundedPreceding,Window.currentRow))as("sum_")
      ).orderBy("id","time").show()
  }
}
```
if(,,)默认值要赋值的标准?给予什么
