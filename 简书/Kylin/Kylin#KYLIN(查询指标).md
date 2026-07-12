##1. 为什么需要查询指标？
要保证Kylin服务的高可靠，高可用，高性能。为了向用户提供Kylin查询服务的SLA，我们必须要有相应的查询指标。
##2. 查询指标的具体内容和含义
为了能够根据查询服务节点，业务线，具体业务来统计相关查询指标，我们将查询指标分为了Server，Project，Cube3个层次。

以QueryCount 这个指标为例，这个指标最终会生成3种相关的Metrics：

```scala
Hadoop:name=Trip,service=Kylin.QueryCount 
Hadoop:name=TI_pro,service=Kylin.QueryCount 
Hadoop:name=TI_pro,service=Kylin,sub=htl.QueryCount

其中Trip代表整个server,
TI_pro代表project name,
htl 代表 cube name.
```

#####Kylin关键查询指标说明
```scala
QueryCount：查询总次数；
QueryFailCount：查询失败总次数；
QuerySuccessCount：查询成功总次数；
CacheHitCount： 查询缓存命中次数；
QueryLatency{60s,360s,3600s}{50,75,90,95,99}thePercentile：1分钟，10分钟，1小时内的查询百分位时延；（这3个时间间隔可以通过参数kylin.query.metrics.percentiles.intervals来配置）
QueryLatencyAvgTime，QueryLatencyIMaxTime，QueryLatencyIMinTime：查询时延的均值，最大值，最小值；
ScanRowCount： 查询读取HBase行数的相关指标；
ResultRowCount：查询返回结果行数的相关指标。
```
##3. 查询指标的日常作用
通过Kylin查询日报我们可以了解每个Query Server， 每个Project的查询总次数，查询失败率，查询时延等查询信息。通过查询的监控Screen我们可以实时掌握Kylin的查询服务情况。

##4. 收集Kylin查询指标
在 Kylin 1.5.4版中，我们可以通过将参数kylin.query.metrics.enabled设为true来将查询指标收集到JMX。

查询指标收集到JMX后，我们就可以通过任意的JMX Metrics收集工具来收集Kylin的查询指标，将查询指标上报到公司的监控系统中。唯一需要注意的是，Kylin的查询指标是分为Server，Project，Cube3个层次的，而这种层次关系是通过动态的ObjectName来实现的，所以在获取ObjectName时需要正则匹配。

内容来源于https://blog.bcmeng.com/post/kylin-query-metrics.html
