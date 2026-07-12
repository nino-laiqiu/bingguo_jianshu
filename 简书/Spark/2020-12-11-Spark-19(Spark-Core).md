##1.scala中debug体验迭代器的执行
迭代器的调用是递归的,执行方法要执行,要找到那个迭代器,这个迭代器要执行也必须要向上一个迭代器要数据,类似于栈的结构,先进后出,所以最后一个迭代器要获取数据,前面的迭代器,必须要迭代完全部的数据

##2.线程池的复习
newFixedThreadPool（int nThread）固定大小线程池:
线程池里的线程数量始终维持在一个固定值，当有一个新的任务进来后，如果线程池中有空闲线程，就立刻调出线程执行任务；否则就会把任务交到等待队列中，等待有空闲线程后再调度执行

 newCachedThreadPool（） 可缓冲线程池
法得到的线程池里线程数量是不固定的，会根据实际情况调整线程池里的线程数量，如果线程池里有空闲线程，当有任务被提交后，会优先使用线程池里的空闲线程，如果线程池里所有的线程都有任务在身，此时又有新任务提交，那么线程池就会新建一个新的空闲线程来执行任务

代码:
```
```

##3.spark执行流程的复述与源码分析
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9c84bc4f42d53233.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**0.反射,spark-submit脚本会执行一个spark-submit的类
1.sparksubmit将任务的信息发送给master
2.master会根据提交的application的资源信息过滤出符合条件的worker,然后进行分配cores
3.master向worker通信,需要的资源信息,application,driver信息发送给worker
4.worker启动executor
5......
6.executor跟driver反向注册
7.driver创建RDD Action算子会调用runjob触发执行
8.根据最后一个RDD往前推,根据依赖关系stage,RDD递归类似于类似于栈的结构,递归的出口是没有父RDD
9.先提交前面的stage,再提交后面的stage,一个stage对应一个stageset,一个stageset有多个task(shuffleMaptask Resulttask),然后把taskset传递给tasksecheduler
10.tasksecheduler将taskset中的task进行序列化,然后根据executor的资源情况,将序列化的task发送给executor
11.将序列化的taskdesccription(里面包装着task)发送给executor
12.将taskdesccription反序列化用taskrunner包装放在线程池中
13.将taskdesc中的task也反序列化然后调用task的run方法,传入到taskcontext中,然后根据具体的task类型,如果是shufflemaptask,就调用其runtask
14.将数据先应用分区器返回ID,然后写入到APPendonlymap的内存中,默认达到5M溢写到磁盘,生成两个文件一个索引文件和数据文件
15.mappartitionRDD向shuffledRDD要数据,shuffleRDD获取shuffledReader,从上游拉取属于自己分区的数据,然后进行全局的聚合,最后将聚合的结果写入到hdfs中**
