spark中的join策略 大概可以分为三种,shuffle join 和broadcast join  非常常见,这是介绍一下桶join(hash join)

#####1.shuffle join (Common Join)
#####2.broadcast join 
#####3.bucket map join
1） set hive.optimize.bucketmapjoin = true;
2） 一个表的bucket数是另一个表bucket数的整数倍
3） bucket列 == join列
4） 必须是应用在map join的场景中
5）如果表不是bucket的，只是做普通join
#####4.Sort-Merge-Bucket
1）
set hive.auto.convert.sortmerge.join=true;
set hive.optimize.bucketmapjoin = true;
set hive.optimize.bucketmapjoin.sortedmerge = true;
set hive.auto.convert.sortmerge.join.noconditionaltask=true;
2） 小表的bucket数=大表bucket数
3） Bucket 列 == Join 列 == sort 列
4） 必须是应用在bucket mapjoin 的场景中
```
创建桶和hash桶数量,这里为4
CREATE TABLE bucketed_user (id INT) name STRING) 
CLUSTERED BY (id) INTO 4 BUCKETS; 
```
两个表join的时候，小表不足以放到内存中，但是又想用map side join这个时候就要用到bucket Map join。其方法是两个join表在join key上都做hash bucket，并且把你打算复制的那个（相对）小表的bucket数设置为大表的倍数。这样数据就会按照join key做hash bucket。小表依然复制到所有节点，Map join的时候，小表的每一组bucket加载成hashtable，与对应的一个大表bucket做局部join，这样每次只需要加载部分hashtable就可以了

**注意**
hive并不检查两个join的表是否已经做好bucket且sorted，需要用户自己去保证join的表，否则可能数据不正确。有两个办法
1）hive.enforce.sorting 设置为true
2）手动生成符合条件的数据，通过在sql中用distributed c1 sort by c1 或者 cluster by c1
表创建时必须是CLUSTERED且SORTED，如下
create table test_smb_2(mid string,age_id string)
CLUSTERED BY(mid) SORTED BY(mid) INTO 500 BUCKETS

>[https://cwiki.apache.org/confluence/download/attachments/27362054/Hive+Summit+2011-join.pdf](https://cwiki.apache.org/confluence/download/attachments/27362054/Hive+Summit+2011-join.pdf)

#####5.hive sql中的group by & Distribute by & partition by & cluster by & partitioned by & clustered by
group by & partition by & Distribute by 首先一定要记住group by分组之后是会组内聚合的而后两者仅仅是分组了，并未有聚合操作

partition by是分区 Distribute by 可以理解为分簇
partition by是分区 区内排序用order by
Distribute by 可以理解为分簇 簇内排序用sort by 另外当 distribute by 和 sort by 后的字段相同时，可以使用 cluster by 方式

partitioned by (分区名 string) 按所分区名分区建表使用
clustered by(列名)  按列分桶建表使用
