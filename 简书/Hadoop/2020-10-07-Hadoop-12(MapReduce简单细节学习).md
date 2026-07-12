#####MapReduce简介
Mapreduce 是一个分布式运算程序的编程框架，是用户开发“基于 hadoop 的数据分析 应用”的核心框架。
Mapreduce 核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的 分布式运算程序，并发运行在一个hadoop 集群上


#####hadoop四大组件
HDFS：分布式存储系统
MapReduce：分布式计算系统
YARN： hadoop 的资源调度系统
Common： 以上三大组件的底层支撑组件，主要提供基础工具包和 RPC 框架等


#####为什么需要 MapReduce？
引入 MapReduce 框架后，开发人员可以将绝大部分工作集中在业务逻辑的开发上，而将 分布式计算中的复杂性交由框架来处理

#####优缺点:
易于编程
良好的扩展性
高容错性
适合PB级别以上海量数据的离线处理

**1.难以实时计算（MapReduce处理的是存储在本地磁盘上的离线数据）
2.不能流式计算（流式计算的输入数据是动态的 MapReduce的输入数据是静态的）
3.难以DAG(有向图:多个应用程序存在依赖关系,后一个应用程序的输入为前一个的输出)计算(MapReduce这些并行计算大都是基于非循环的数据流模型，也就是说，一次计算过程中，不同计算节点之间保持高度并行，这样的数据流模型使得那些需要反复使用一个特定数据集的迭代算法无法高效地运行)**

#####由官方案例要解决的问题
maptask如何工作
reducetask如何工作
maptask如何控制分区 排序
maptask reducetask之间如何链接

![image.png](https://upload-images.jianshu.io/upload_images/9049859-6a15c3d59d3017a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**MapReduce进程:
MrAPPmaster 负责这个程序的进程调度和状态协调
maptask
reducetask**


driver流程
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c22d3f3ba269353c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##数据块与数据切片
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7c0de85e21ce9481.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(一般情况下一个数据块一个map的,换句话说:只有一个map处理这个数据块,如果数据切片数据块为俩个则俩个map并行处理)
![(数据切片要与数据块大小保持一致)](https://upload-images.jianshu.io/upload_images/9049859-13ed596d7fdff1f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##原码简单摘录
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e82f2d96fd4db0a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b6e81ba993dc694a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##切片概述!!!!(重要-怎么处理小文件)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-8d1c2b698c75128b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-557c787633c7f553.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-1fae31a518913c2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a8de6bf08cae516d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b96d896ca3d02941.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##fileinputformat实现类
![默认](https://upload-images.jianshu.io/upload_images/9049859-022822338b41ad97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-607b43a2a5cbe31e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-fe22259853ce6505.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-95ed43687ab9e1e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/9049859-969e8d0dda82584e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0249273d4c87ed08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(这是一个难点)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-67bd2818c2e93a91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##分区规则
![image.png](https://upload-images.jianshu.io/upload_images/9049859-6ff0982f5ab4cd28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##排序规则
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2568bcaf8b143a74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##combiner(也就重写reduce方法 只是把这个类放在setcombiner)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-fe63fdf7fbcae807.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-08de4cc4bba5b0df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##Outputformat

![image.png](https://upload-images.jianshu.io/upload_images/9049859-a053de5aba7684da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2186fc5e788ac8f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-20d4aae1421d1ef4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ca1668e1ff40adb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##join
![image.png](https://upload-images.jianshu.io/upload_images/9049859-305d413b82a526de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-8cd8499928b59847.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(map阶段join)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-31566fe1c656be46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


































