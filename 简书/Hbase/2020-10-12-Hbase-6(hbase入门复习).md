##1.写数据流程

##2.读数据流程

##3.数据导入到hbase表中(两种方法)

##4.hbase shell命令与api

##5.Region的结构

Region按大小分割的，每个表开始只有一个region，随着数据增多，region不断增大，当增大到一个阈值的时候，region就会等分会两个新的region，之后会有越来越多的region

Region是Hbase中分布式存储和负载均衡的最小单元，不同Region分布到不同RegionServer上

Region虽然是分布式存储的最小单元，但并不是存储的最小单元。Region由一个或者多个Store组成，每个store保存一个columns family；每个Strore又由一个memStore和0至多个StoreFile组成

StoreFile包含HFile；memStore存储在内存中，StoreFile存储在HDFS上。

##6.Hbase基本组件说明

>https://blog.csdn.net/woshiwanxin102213/article/details/17584043?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160242390119725271761758%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160242390119725271761758&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2

疑问:
相同的rowkey存储在一个region上,我读取写入数据不会发生多线程问题吗?

