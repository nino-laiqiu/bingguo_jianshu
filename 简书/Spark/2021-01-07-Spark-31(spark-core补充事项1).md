##1.spark 和 MapReduce Wordcount案例哪个快,为什么
MapReduce在map端溢写环形缓冲区会排序,影响性能


##2.standlone 和 yarn 模式 worker中的executor
两个executor是不能相互访问的

## 3.spark中的名词的关联
application:
一个application中可以触发多次action,触发一次action形成一个DAG,一个DAG对应一个job
job:
driver向executor提交作业
一个DAG对应一个job
stage:
即,任务执行阶段,一个stage对应一个taskset,一个taskset中task数量取决于stage中最后一个RDD分区的数量
taskset:
保存同一种计算多个task集合,保存同一种计算逻辑,多个task集合
task:
类的实例:有属性:从哪里读取数据  有方法:如何计算
dag:
有向无环图
触发action就形成一个完整的dag
![image.png](https://upload-images.jianshu.io/upload_images/9049859-909ec4c7bed85a41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##4. take会触发一次action

## 5.spark中的序列化问题,map和mappartition的适用场所

##6.standlone模式 中堆外内存

##7.对于任务的提交

##8.mappartition中创建mysql的连接,怎么关闭才安全,避免闭包问题
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ec5689307c68343f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

