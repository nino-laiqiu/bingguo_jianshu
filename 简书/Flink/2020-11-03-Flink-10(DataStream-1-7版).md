##1.构建一个典型的flink流式应用需要的步骤
1. 设置执行环境
2. 从数据源中读取一条或多条流
3. 通过一系列流式转换来实现应用逻辑
4. 选择性地将结果输出到一个或者多个数据汇中
5. 执行程序
##2.转换操作
我们将datastreamAPI的转换分为四类
1. 作用于单个事件的基本转换
2. 针对相同键值事件keyedstream转换
3. 将多条数据流合并为一条或者一条数据流拆分为多条流的转换
4. 对流中的事件进行重新组织的分发转换

#####基本转换
1. map
2. filter
3. flatmap(filter和map的泛化)
#####基于KeyedStream的转换
1. keyby
keyby转换通过指定键值的方式将一个DataStream转化为keyedstream,流中的事件会根据各自键值被分到不同的分区,这样一来,有着相同键值的事件一定会在后续算子的同一个任务上处理
2. 滚动聚合
3. reduce
只对有限键值域使用滚动聚合

#####多流转换
1. union
合并两条或者多条类型相同的datastream,union在执行过程中,来自两条流的事件会以FIFO的方式合并,其顺序无法得到任务保证
2. connect comap coflatmap
在流处理中,合并两条数据流中的事件时一个非常普遍的需求,默认情况下,connect方法不会使两条输入流的事件之间产生任何关联,这不是我们希望看到的,为了实现确定性的转换,connect可以与keyby和broadcast结合使用
3. split和select
spilt转换是union转换的逆操作,它将输入流分隔成两条或者多条类型和输入流相同的输出流,每一个到来的事件都可以被发往零个,一个或者多个输出流

#####分发转换
1. 随机(shuffle)
2. 轮流(rebalance)
3. 重调(rescale)
轻量级的负载均衡方法
**2/3的区别在于,轮流会在所有发送任务和接收任务之间建立通信通道,而重调中每个发送任务只会和下游算子的部分建立通道**
4. 广播(broadcast)
5. 全局(global)
6. 自定义

##3.设置并行度
默认并行度
设置并行度
##4.类型
**注意:使用了lambda函数或者泛型函数类型,则必须显式指定类型信息才能启动应用或者提高其性能**
#####支持的数据类型
1. 原始数据类型
2. java和scala元组
3. scala样例类
4. pojo(包括avro生成的类)
5. 一些特殊的类型

#####为数据类型创建类型信息
flink类型系统的核心类是typeinformation,它为系统生成序列化器和比较器提供了必要的信息
#####显式提供类型信息
##5.定义键值和引用字段
#####字段位置
#####字段表达式
#####键值选择器

##6.实现函数
#####函数类
#####lambda函数
#####富函数(☆)
在使用富函数的时候,可以对函数的生命周期实现两个额外的方法
1. open 初始方法,通常用于只做一次的设置的工作
2. close 终止方法,通常用于清理和释放资源
3. 此外，还可以使用getRuntimeContext()方法来访问函数的RuntimeContext，从RuntimeContext中获取一些信息，例如函数的并行度，访问分区状态的方法等
```
class MyFlatMap extends RichFlatMapFunction[Int, (Int, Int)] {
//子任务的index
var subTaskIndex = 0

override def open(configuration: Configuration): Unit = {
subTaskIndex = getRuntimeContext.getIndexOfThisSubtask
// 以下可以做一些初始化工作，例如建立一个和HDFS的连接
}

override def flatMap(in: Int, out: Collector[(Int, Int)]): Unit = {
if (in % 2 == subTaskIndex) {
out.collect((subTaskIndex, in))
}
}

override def close(): Unit = {
// 以下做一些清理工作，例如断开和HDFS的连接。
}
}
```
```
class MyRichMapFunction extends RichMapFunction[String, String] {
  var startTime: Long = _

  override def open(parameters: Configuration): Unit = {
    startTime = System.currentTimeMillis()
  }

  override def map(in: String): String = {
    // 每条记录的处理时间
    val str: String = in + "处理时间:" + System.currentTimeMillis()
    s"开始时间:$startTime, 当前数据$str"
  }

  override def close(): Unit = {}
}
```
