##kylin3之前的TopN实现原理

看一个典型的 Top－N 查询示例。该查询是选择在 2015 年 10 月 1 日，地址在北京，销售商品按价格之和排序（倒序），找前 100 个。

![示例](https://upload-images.jianshu.io/upload_images/9049859-0589183815ccd588.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 Kylin v1.5 之前，SQL 中的 group by 列，需声明成维度，所以这个 Cube 的维度中要有日期，地点和商品名，度量是 SUM(PRICE) 。图中展示了一个这样设计 Cube。因为商品的基数很大，计算的 cuboid 的行数会很多；而度量值 SUM(PRICE) 是非排序的，因此需要将这些纪录都从存储器读到 Kylin 查询引擎中（内存）， 然后再排序找出最高的纪录；这样的解决办法总开销较大。
![用普通度量处理 Top N 查询](https://upload-images.jianshu.io/upload_images/9049859-b9bac884ccfb20ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Kylin 选择了 Space－Saving 算法 ，以及它的一个衍生版 Parallel Space－Saving，并在此之上做了特定的优化，有了 Top－N 之后，Cube 的设计会比以前简单很多，因为像刚才的商品名会被挪到 Measure 中去，在 Measure 里按 Sum 值做倒序，只保留最大的若干值
![使用 Top N 度量的 Cube](https://upload-images.jianshu.io/upload_images/9049859-2df7469fbb06eac6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

值得一提的是需要用多少空间运算 Top－N。简单来说存储空间越多准确率越高。我们通过使用生成一些样本数据然后用 Space－Saving 计算，并且跟真实结果做比较，发现 50 倍空间对于普通的数据分布是够用的。也即，用户需要 Top 100 的结果，Kylin 对于每种组合条件值，保留 Top 5000 的纪录, 并供以后再次合并。这样即使多次合并， Top100 依然是比较接近真实结果


Top－N 的优点：因为它只保留 Top 的记录，会让 Cube 空间大幅度减少，而查询性能大大提升。在一个典型的例子里，改用 Top-N 后，Cube 的大小减少了 90%，而查询时间则只有以前的 10% 不到。
缺点是它可能是近似的结果（当 50 倍空间也无法容纳所有基数的时候）。如果业务场景需要绝对精确的话，它可能不适合。

Top－N 误差率由很多因素决定的：
数据的分布：数据分布越陡，误差越小。
算法使用的空间：如果对精度要求高的话，可以选择用更多的空间换取更精准的准确率 。在实际使用中，可以做一些比较以了解误差情况。

##kylin3之后的TopN实现原理
当前Kylin4的TopN UDAF注册是在org.apache.kylin.engine.spark.job.CuboidAggregator#aggInternal, 代码如下：
```
def aggInternal(ss: SparkSession,
                  dataSet: DataFrame,
                  dimensions: util.Set[Integer],
                  measures: util.Map[Integer, FunctionDesc],
                  isSparkSql: Boolean): DataFrame = {
      //省略
      measure.expression.toUpperCase(Locale.ROOT) match {
        //省略
        case "TOP_N" =>
          // Uses new TopN aggregate function
          // located in kylin-spark-project/kylin-spark-common/src/main/scala/org/apache/spark/sql/udaf/TopN.scala
          val schema = StructType(measure.pra.map { col =>
            val dateType = col.dataType
            if (col == measure) {
              StructField(s"MEASURE_${col.columnName}", dateType)
            } else {
              StructField(s"DIMENSION_${col.columnName}", dateType)
            }
          })

          if (reuseLayout) {
            new Column(ReuseTopN(measure.returnType.precision, schema, columns.head.expr)
              .toAggregateExpression()).as(id.toString)
          } else {
            new Column(EncodeTopN(measure.returnType.precision, schema, columns.head.expr, columns.drop(1).map(_.expr))
              .toAggregateExpression()).as(id.toString)
          }
       //省略
        case _ =>
          max(columns.head).as(id.toString)
      }
    }.toSeq
//省略
    if (reuseLayout) {
      val columns = NSparkCubingUtil.getColumns(dimensions) ++ measureColumns(dataSet.schema, measures)
      df.select(columns: _*)
    } else {
      df
    }
  }
```


其实TopN最初的实现的在org.apache.kylin.engine.spark.job.TopNUDAF，但是可以看到目前TopN的实现是在org.apache.spark.sql.udaf.BaseTopN.scala，最新的实现主要针对旧的实现修复了性能问题，详情可以查看[KYLIN-4760](https://links.jianshu.com/go?to=https%3A%2F%2Fissues.apache.org%2Fjira%2Fbrowse%2FKYLIN-4760)。

Kylin 4.0的TopN是通过Spark UDAF的方式实现的，以下是实现类接口之间的关系，可以看到最终实现的是BaseTopN，继承的是TypedImperativeAggregate。然后BaseTopN又有两个子类，分别是EncodeTopN和ReuseTopN，当从平表（FlatTable)开始构建的时候，FlatTable中没有构建过TopN，这里会调用EncodeTopN，再次之后从已经构建好的cuboid构建下一层cuboid的时候会调用ReuseTopN，避免重复计算，接口关系图如下：
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0154ee8868f1f5f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

继承TypedImperativeAggregate实现TopN，而不是UserDefineAggregateFunction主要是因为UserDefinedAggregateFunction 是把 catalyst 内部 internalRow 类型转换为了 Row 类型，然后使用用户自己的 update 方法处理，然后TypedImperativeAggregate需要自己做序列化、反序列化处理，少了一层转换。

#####TopNCounter介绍
前面提到Space-Saving算法是在TopNCounter中实现的，此处我们对TopNCounter的实现进行一个简要的介绍。BaseTopN对象初始化的时候会创建TopNCounter对象，用户保存计算过程中符合TopN条件的行，对应于Spark UDAF的概念是aggregate buffer。update，merge，eval都是处理的TopNCounter。TopNCounter在初始化的时候需要指定容量, 大小建议为N * TopNCounter.EXTRA_SPACE_RATE, 其中N为TopN定义的大小，EXTRA_SPACE_RATE为建议额外空间调整参数，默认为10， 也就是说如果定义的topn(10,4), 那么TopNCounter的初始化大小则为10 * 10 = 100 。

TopN的处理流程可以见下图：
![image.png](https://upload-images.jianshu.io/upload_images/9049859-fcc405d425a3ea25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

update()主要将传入的行通过TopNCounter.offer() 将一行的内容插入到TopNCounter对象中，merge则是对两个经过update()操作的group进行去重合并，最后在eval()的时候调用TopNCounter.sortAndRetain()来排序和调整TopNCounter大小，最终得到聚合结果。


Kylin 4.0目前使用的是parquet进行存储，我们定义topn(10,4)， TopNCounter.EXTRA_SPACE_RATE 设置为1。cuboid中维度和度量列明的映射关系为：
0 -> seller_id
1 -> item_id
2 -> id
3 -> price
4 -> Count
5 -> TopN

如下是只有TopN和只有SUM的cuboid内容：

![image.png](https://upload-images.jianshu.io/upload_images/9049859-8f3d867d5dfdf249.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

值得注意的是第二行，Count为11，但是实际上TopN列只存储了10个值，这是因为TopNCounter的容量只有10 * EXTRA_SPACE_RATE = 10， 超过10的内容不会被存储，这也是当前TopN存在误差的原因所在。可以看到TopN将计算的维度和group by的维度放到了一起，然后用数组的形式进行存储。
![image.png](https://upload-images.jianshu.io/upload_images/9049859-fc1559cea83ae29a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于sum度量，kylin则是直接存储的sum后的聚合值。



>参考，其实是抄了一边
 [Apache Kylin的Top-N近似预计算-InfoQ](https://www.infoq.cn/article/2016/08/Apache-Kylin-Top-N)
[Kylin 4.0 TopN Introduction CN - Kylin 4.0 TopN Introduction CN - Apache Software Foundation](https://cwiki.apache.org/confluence/display/KYLIN/Kylin+4.0+TopN+Introduction+CN)
[Apache Kylin权威指南（九）：Top_N 度量优化-InfoQ](https://www.infoq.cn/article/oP3XOlGl5a6qNGEH3Yf3)



问题：
1. 对于Kylin3，是哪个维度会移入到 Measure中(不定义为维度实现，我感觉怎么是个锁)，多个维度移入到 Measure又是如何实现
2. TopNCounter中的方法调用流程
