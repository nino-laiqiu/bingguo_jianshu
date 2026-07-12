##1.注意事项
1. 并发控在5个以内，batchnum 控制在百万以内，防止数据的丢失，任务失败
2. datax，stream lead数据并不是事务操作，需要添加前置SQL， truncate table
3. starrocks并不会自动创建分区，需要提前创建分区，如下方法
```
ALTER TABLE ${table} ADD PARTITION if not exists p${nextPartition}
VALUES [('${zdt.add(2,2).format("yyyy-MM-01")}'),('${zdt.add(2,3).format("yyyy-MM-01")}'));
```
```
PARTITION BY RANGE(`d`)
(PARTITION p000001 VALUES [('0000-01-01'), ('2019-01-01')),
PARTITION p201901 VALUES [('2019-01-01'), ('2019-02-01')),
PARTITION p201902 VALUES [('2019-02-01'), ('2019-03-01')))
```
```
PARTITION p1 VALUES LESS THAN ('2021-01-31'),
PARTITION p2 VALUES LESS THAN ('2021-02-28'),
```
4. starrocks读取hive时，hive表正在更新，starrocks会报元数据找不到
5. 注意建表时varchar类型不要越值

##2. 业务
##3.明细模型更新方法
1. 增量添加
2. 全量覆盖
需要一张正式表和一张临时表，先将数据导入临时表，然后原子更新swap
3. 增量覆盖
[分区原子更新](https://docs.starrocks.com/zh-cn/main/faq/Others#%E6%95%B0%E6%8D%AE%E6%81%A2%E5%A4%8D%E4%B8%AD%E5%8E%9F%E5%AD%90%E6%9B%BF%E6%8D%A2%E8%A1%A8%E5%88%86%E5%8C%BA%E5%8A%9F%E8%83%BD%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95)
在正式表上建立临时分区，将数据导入导临时分区，然后REPLACE替换
```
ALTER TABLE table1
REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
```
注意不论是全量覆盖还是增量覆盖中都存在了DDL语法,这都要走JDBC网址
##4. 待解决的问题
1. 更新时是否影响查询
2. datax导数据文件数量过多，是什么原因导致的，以及如何解决这个问题
3. starrocks tablet是如何决定的
4. 文件数是怎么决定的
![示例，为什么这么多的文件](https://upload-images.jianshu.io/upload_images/9049859-388a715df740df08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
5. java.io.FileNotFoundException: File does not exist,这个问题是什么导致的，我的hive表没有在更新也会报这个问题的
6. 我的桶个数设置为10个，文件数为365个，但是桶个数设置为64个，文件数就突然膨胀到14万个了，为什么？？？
7. StarRocks导入报错close index channel failed解决方案  [close index channel failed](https://blog.csdn.net/Yuchen914/article/details/122983180)
8. com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure,在进行增量数据覆盖更新的时候报错，后来发现DDL语法操作的网址和导入数据的网址是不一致的，连接的时候要注意端口
