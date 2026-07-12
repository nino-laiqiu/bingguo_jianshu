以下是本人Flink学习的相关内容,对于本人,目前来说flink分为两个阶段,第一个阶段是2020年下半年到2021年第一季度,学习的主要载体是 ⟪基于Apache Flink的流处理⟫这本书,b站上的视频等...但是上班也用不上Flink一直从事的是离线开发的工作....SparkSQL BOY和OLAPSQL BOY....想体验一下FlinkSQL BOY...第二个阶段是目前这个阶段,我准备把Flink给捡起来,学习的主要载体是⟪Flink内核原理与实现⟫这本书,flink官网,别人博客的成果等...

#第二个阶段🤑
##⭐Flink入门
[1. Flink概览:简单介绍了Flink的特点,框架架构,运行时架构](https://www.jianshu.com/p/e83718325309)

[2. DataStream的一个案例和使用模板](https://www.jianshu.com/p/cf24cd2b5944)

[3. Flink Source的使用和类型](https://www.jianshu.com/p/c899d8ad66b1)
Source类型例如有:从文件中读取,从集合中读取,Socket文本流,从Kafka中读取等

[4. Flink Transformation的使用和类型,及富函数的简单使用说明](https://www.jianshu.com/p/0a288d2358ac)
Transformation类型例如有:map,filter,flatmap,keyBy,max&maxBy,reduce

[5. Flink 数据分区,Sink的使用和类型,及简单的介绍了时间和窗口的应用](https://www.jianshu.com/p/90ae1e10af76)
数据分区包含:随机和轮询分区,广播,全局分区,及自定义重分区等
Sink类型例如有:写入文件系统,写入Kafka(重点),写入Ridis,写入ES,写入Mysql,写入自定义Sink等
时间语义包含:处理时间和事件时间,介绍了水位线的特性,生成策略,水位线的传递,及延迟数据的处理
简单介绍了窗口的分类:滚动窗口,滑动窗口,会话窗口(合并)等,及简单使用了窗口的功能(这一块是重点)

[6. Flink 处理函数,定时器/触发器的简单应用,多流转换,状态编程,状态生存时间TTL,容错机制](https://www.jianshu.com/p/ee1d675c27c6)

[7. Flink TableAPI/SQL入门](https://www.jianshu.com/p/6140065d77f1)

##⭐FlinkSQL
[1. Flink TableAPI/SQL任务的代码结构,动态输出表转化为输出数据,SQL中指定时间属性](https://www.jianshu.com/p/46f10a336986)

[2. Flink TableAPI/SQL DDL/DML语法](https://www.jianshu.com/p/0e3b7ff01eb0)


#第一个阶段😁
##⭐⟪基于Apache Flink的流处理⟫
这一块有什么用:没什么用了

[1. Flink  状态化流处理概述](https://www.jianshu.com/p/460d0e08b3bc)
[2. Flink  流处理基础](https://www.jianshu.com/p/0bae795e411a)
[3. Flink  Flink架构](https://www.jianshu.com/p/c829d9c49ee9)
[4. Flink  开发环境](https://www.jianshu.com/p/d5d7ef4ee063)
[5. Flink  DataStream 1.7版](https://www.jianshu.com/p/51af6a9e39c8)
[6. Flink  基于时间和窗口的算子](https://www.jianshu.com/p/cb52a0e9cd20)
[7. Flink  有状态算子和应用](https://www.jianshu.com/p/b92f42c02c7f)
[8. Flink  读写外部系统](https://www.jianshu.com/p/b57f1326ff2c)
[9. Flink  搭建Flink运行流式应用](https://www.jianshu.com/p/7ffe4b08937a)
[10. Flink  流式应用运维](https://www.jianshu.com/p/e70b87db10ec)

##⭐行业实践
搁置状态....后面还是要看看Datafun的视频的

[1. Flink 单点恢复和checkpoint优化](https://www.jianshu.com/p/bac2ee6f01ff)
[2. Flink 调度优化](https://www.jianshu.com/p/ed69be42ba22)

##⭐课程
这一块有什么用:主要目前企业开发用的还是FlinkSQL,这个课程作用不是很大了,下面的东西其实我也花费了2-3个月的时间,并且有些知识非常有可取之处,比如说Flink的异步IO,自定义触发器,自己实现一个算子或者状态,FlinkShuffle,状态的应用等

[1. Flink   WorldCount案例](https://www.jianshu.com/p/255743355d63)
[2. Flink  任务调度原理 批/流处理系统的介绍](https://www.jianshu.com/p/37f2d58a7551)
[3. Flink  读取数据的方式](https://www.jianshu.com/p/30d9fe166397)
[4. Flink   Transform](https://www.jianshu.com/p/331268a94bb7)
[5. Flink Sink](https://www.jianshu.com/p/605841e526c5)
[6. Flink 快速入门](https://www.jianshu.com/p/817049e0c00a)
[7. Flink  Source](https://www.jianshu.com/p/8e2b23131f68)
[8. Flink Sink](https://www.jianshu.com/p/e80b07d66219)
[9.  Flink    Transform 一](https://www.jianshu.com/p/d4f79feb6b1d)
[10. Flink  Transform 二](https://www.jianshu.com/p/896d15dfaac1)
[11. Flink  窗口概念](https://www.jianshu.com/p/e373df62a1b3)
[12. Flink  窗口函数](https://www.jianshu.com/p/0f2b045fc35c)
[13. Flink  Joins](https://www.jianshu.com/p/0d3c4a4ae150)
[14. Flink  State](https://www.jianshu.com/p/12064fed3e4a)
[15. Flink  ProcessingTime/侧流输出](https://www.jianshu.com/p/4f366d22fd6b)
[16. Flink TTL](https://www.jianshu.com/p/69cb35a9055a)
[17. Flink  Broadcast/QueryableKey](https://www.jianshu.com/p/6080254b5d49)
[18. Flink  异步IO](https://www.jianshu.com/p/ec89bcd13bd6)
[19. Flink  CheckPoint](https://www.jianshu.com/p/2c116d325d79)
[20. Flink ExactlyOnce语义 ](https://www.jianshu.com/p/785dddd2389b)
[21. Flink  实时日志采集架构](https://www.jianshu.com/p/96a73338ee43)
[22. Flink 实时日志需求案例](https://www.jianshu.com/p/9070d5ce8b77)
[23. Flink  实时业务需求案例](https://www.jianshu.com/p/e5fcd292d5d8)
[24. Flink  离线API](https://www.jianshu.com/p/05af04d31416)
[25. Flink  TableAPI 一](https://www.jianshu.com/p/7829d5475c6d)
[26. Flink  TableAPI 二](https://www.jianshu.com/p/a1de7c4a290e)
[27. Flink  水位线](https://www.jianshu.com/p/383591e3df57)
[28. Flink  shuffle](https://www.jianshu.com/p/286d659615d1)
[29. Flink 电商用户行为分析案例 一](https://www.jianshu.com/p/5cd643749bf4)
[30. Flink  电商用户行为分析案例 二](https://www.jianshu.com/p/2d1d15483937)
[31. Flink  电商用户行为分析案例 三](https://www.jianshu.com/p/f7e6895b91cb)



















