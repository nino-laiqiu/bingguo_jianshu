##flink是如何保存累计值的?

flink是一种有状态的流计算框架，其中说的状态包括两个层面：
1) operator state 主要是保存数据在流程中的处理状态，用于确保语义的exactly-once。
2) keyed state  主要是保存数据在计算过程中的累计值。

##Connect与 Union 区别
Union之前两个流的类型必须是一样，Connect可以不一样，在之后的coMap中再去调整成为一样的。
Connect只能操作两个流，Union可以操作多个
#####1.map flatmap filter

##Reduce说明
KeyedStream → DataStream：一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果。



#####2.keyBy   reduce  split     select    connect    union  

代码课堂练习:
```
import org.apache.flink.api.common.functions.ReduceFunction
import org.apache.flink.streaming.api.scala._


case class Counter_reset(ID: Int, stamp: Long, once: Double)

object Transform1 {
  def main(args: Array[String]): Unit = {
    val environment = StreamExecutionEnvironment.getExecutionEnvironment
    environment.setParallelism(4)
    var path = "C:\\Users\\hp\\IdeaProjects\\Flink_Scala\\src\\main\\resources\\trans1"
    val filterText = environment.readTextFile(path)
    val value = filterText
      .map(data => {
        val strings = data.split("\\s+")
        Counter_reset(strings(0).toInt, strings(1).toLong, strings(2).toDouble)
      })
      //split
      .split((t1) => {
        if (t1.ID > 3) Seq("da") else Seq("xiao")
      })
    //分组
    //.keyBy("ID")
    //reduce方法
    /*  .reduce((reset,newreset1)=>{
          Counter_reset(reset.ID,newreset1.stamp,reset.once.min(newreset1.once))
      })*/
    // .reduce(new MyreduceFuction)
    //split
    /* .split((t1)=>{
         if (t1.ID>3) return Seq("da") else return Seq("xiao")
     })*/
    val value1 = value.select("da")
    val value2 = value.select("xiao")
    val value3 = value.select("da", "xiao")

    val value4 = value1.map(data =>
      Counter_reset(data.ID, data.stamp, "1".toDouble)
    )
    value4.print("value4")
    //合流操作
    val value5 = value4.connect(value3)
    val value6 = value5.map(data1 => (data1.ID,data1.stamp),
      data2 => ()
    )
     value6.print("value6")

    //union  必须类型一致
    val value7 = value1.union(value3)
   // value7.print("value7")

    // .min("once")
    // value.print()
    //    value1.print("da")
    //    value2.print("xiao")
    //    value3.print("either")
    environment.execute("mytext")
  }
}

//reduce方法也可以传入自定义方法
class MyreduceFuction extends ReduceFunction[Counter_reset] {
  override def reduce(t: Counter_reset, t1: Counter_reset): Counter_reset = {
    Counter_reset(t.ID, t1.stamp, t.once.max(t1.once))
  }
}

```
