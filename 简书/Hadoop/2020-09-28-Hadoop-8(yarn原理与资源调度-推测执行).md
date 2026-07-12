##补充 MapReduce对象重用问题(非常重要!!!!!!!)

Mapreduce计算，输出时key的值都是一个，而且都是最后一个put进入的值！
原因：
Key-Value对象的重用导致的：Key是一个引用，它在栈中，指向堆中一个对象，同样Value也是如此。虽然reduce方法会反复执行多次，但key和value相关的对象只有两个，key和value的引用也是只有两个，reduce会反复重用这两个对象。所以put进去的key指向的对象只有一个，对象的值为最后一个put进去的值，所以输出的key的值都为最后一个。

##2.yarn框架

(1)resourcemanager(RM)
处理客户端请求
监控nodemanager
启动或监控applicationmaster
资源的分配与调度
(2)nodemanager(NM)
管理单个节点上的资源
处理来自MR的命令
处理来自AM命令
(3)applicationmaster(AM)
负责数据的切块
为应用程序中请资源并分配给内部任务
任务的监控与容错
(4)container
是yarn中的资源抽象,它封装了某个节点的多维度资源(内存 CPU 磁盘 网络)

##yarn工作机制
**有疑问 container什么玩意? 如果申请了内存等,就不能用了吗 容器调度器,假设一个任务只要3个机子,还有其他机子没使用,这个job正在运行,则其他job能运行吗 还有我有个误区,以为有些机子map可能跑不了,以为可能要处理的数据过多,但是我忽略了每个数据块的大小是有限制的,也就是说map一次跑的数据是有限制的,所以问题只是机制处理能力的问题???**

1.MR程序提交到客户端所在节点
2.MR向RM申请一个AM,RM返回一个AM资源提交路径(hdfs)和一个AM_ID
3.客户端提交job运行所在资源到提交路径(hdfs),资源提交完毕,向RM申请运**mrAppmaster**
4.RM将用户的请求初始化为一个task任务,放在调度队列中
5.某个空闲的nodemanger领取task任务.然后创建容器container,  **mrappmaster**
mrappmaster根据节点信息决定开几个map和reduce
6.mrappmaster下载job资源到所在nodemanger,并且向RM申请运行maptask容器
7.RM把这些请求同样放到队列中,空闲的nodemanger领取任务,创建container容器
8.mrappmaster发送运行程序脚本到申请了任务的nodemanger
9.完成map阶段mrappmaster继续向RM申请reducetask程序

![image.png](https://upload-images.jianshu.io/upload_images/9049859-28432a05b22c4d2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##YARN资源调度
在Yarn中有三种调度器可以选择：FIFO Scheduler ，Capacity Scheduler，FairS cheduler。


>FIFO Scheduler把应用按提交的顺序排成一个队列，这是一个先进先出队列，在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。

>Capacity调度器，有一个专门的队列用来运行小任务，但是为小任务专门设置一个队列会预先占用一定的集群资源，这就导致大任务的执行时间会落后于使用FIFO调度器时的时间。


>Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。


##任务的推测执行
一个作业由若干map和reduce任务构成,因为硬件老化,软件bug,某些任务可能运行非常慢
推测执行算法原理
![image.png](https://upload-images.jianshu.io/upload_images/9049859-aaf156bf951f2920.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

