##1.关于多版本
versions 的设置存放多个不同时期戳的相同数据
shell命令只能删除全部的版本数据,对于某版本的删除采用 addcolumns()方法


##2.读取数据流程(读比写慢)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b2bae62fd57d2609.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-5203c917563debdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**1.client先访问zookeeper,获取meta表在位于哪个regionsever
2.访问对应regionsever,获取meta对应的regionsever.根据读数据请求的rowkey,查询出目标对应数据在哪个regionsever中的region,并将该table的region信息以及meta表的信息缓存在客户端的meta eache,方便下一次访问
3.与目标regionsever进行通讯
4.分别在block cache(读缓存),memstore 和 store File中查询目标数据,并将查询到的数据进行合并,此处所有数据指向同一条数据的不同版本(time stamp) 或者不同的类型
5.将从文件中查询的数据块(block hfile 数据存储单元,默认大小64kb)缓存到block cache
6.将合并的最终结果返回给客户端**

##3.元数据表 .META.和 -ROOT-
   HBase内部维护着两个元数据表，分别是-ROOT- 和 .META. 表。他们分别维护者当前集群所有Region 的列表、状态和位置。
(1) .META. 记录用户表的 Region 信息，可以有多个 Region，.META.会随需要被Split。
(2) -ROOT- 记录 .META. 表的 Region 信息，只有一个 Region，-ROOT-永不会被Split。
   -ROOT- 表包含.META. 表的Region 列表，因为.META. 表可能会因为超过Region 的大小而进行分裂，所以-ROOT-才会保存.META.表的Region索引，-ROOT-表是不会分裂的。而.META. 表中则包含所有用户Region（user-space Region）的列表。表中的项使用Region 名作为键。Region 名由所属的表名、Region 的起始行、创建的时间以及对其整体进行MD5 hash值。
Table1实在太大了，它的Region实在太多了，.META.为了存储这些Region信息，花费了大量的空间，自己也需要划分成多个Region。这就意味着可能有多个RegionServer在管理.META.。怎么办？在ZooKeeper里面存储所有管理.META.的RegionServer地址让Client自己去遍历？HBase并不是这么做的。
HBase的做法是用另外一个表来记录.META.的Region信息，就和.META.记录用户表的Region信息一模一样。这个表就是-ROOT-表。这也解释了为什么-ROOT-和.META.拥有相同的表结构，因为他们的原理是一模一样的。
(-ROOT-存储 .META.在按个regionsever中,因为可能 .META.太大了,一个regionsever来负责 .META.会增加负担)

##4.Region 定位流程
Client 访问用户数据之前需要访问 ZooKeeper，然后访问 -ROOT- 表，接着访问 .META. 表，最后才能找到用户数据的位置去访问。中间需要多次网络操作，不过 Client 端会执行 Cache 缓存。
**(1) 客户端client首先连接到ZooKeeper这是就要先查找-ROOT-的位置。
(2) 然后client通过-ROOT-获取所请求行所在范围所属的.META.Region的位置。
(3) client接着查找.META.Region来获取user-space Region所在的节点和位置。
(4) 接着client就可以直接和管理者那个Region的RegionServer进行交互。**


##5.Compact过程
memstore是hfile在内存中的体现
flush
多个小型的hfile(在列族中所有的列存储在相同的hfile)
合并过程:
minor compaction
major compaction

##6.数据真正的删除时间
![image.png](https://upload-images.jianshu.io/upload_images/9049859-73d98d6bfc19942e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-fda6998c733eab3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(这里flush没有删除,是因为flush重新生成了一个文件.文件没有合并,同一个文件的相同数据或者相同列族属性的被删除)

##7.spilt
当region的某个列族超过默认10G时进行自动拆分,不是按列族进行切分的,而是按照rowkey进行切分的
(数据热点问题,所以要进行预分区)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e9770f6ef600e110.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##8.hbase简单优化

(1).自带高可用
![image.png](https://upload-images.jianshu.io/upload_images/9049859-700ebbf95f0a70b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(2).预分区
(3).rowkey的设计

