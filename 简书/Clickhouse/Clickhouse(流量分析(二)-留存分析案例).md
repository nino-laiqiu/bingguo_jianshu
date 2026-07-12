##1. 留存分析定义
[神策用户分析模型——留存分析的使用方法](https://www.sensorsdata.cn/blog/jie-xi-chang-jian-de-shu-ju-fen-xi-mo-xing-liu-cun-fen-xi/)


##2. 留存分析
[ClickHouse留存分析工具十亿数据秒级查询方案](https://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649748113&idx=1&sn=2f0ea3ba734f29e0c3df8a88fb73058d&chksm=bed361ea89a4e8fca033561d71e2a12220ed973bebdd779d5d1c6737dd6a3eb6aac72f08fb2d&mpshare=1&scene=1&srcid=0907f5wcsH5LNZHJdlKlP633&sharer_sharetime=1599468519603&sharer_shareid=9099e0c56e9df70effb9bf70b32d2e8c&version=3.0.25.1802&platform=win#rd)

[高效压缩位图RoaringBitmap的原理与应用](https://www.jianshu.com/p/818ac4e90daf)

[留存函数（retention）](https://help.aliyun.com/document_detail/216948.html)


#####方案一. Roaringbitmap

一般来说，求留存率的做法就是两天的用户求交集,join的速度会比较慢。假若每一个用户都可以表示成一个32位的无符号整型，用bitmap的形式去存储，S1和S2的求交过程就是直接的一个位比较过程，这样速度会得到巨大的提升。而Roaringbitmap对数据进行了压缩，其求交的速度在绝大部分情况下比bitmap还要快，因此这里我们考虑使用Roaringbitmap的方法来对计算留存的过程进行优化。

这里的bitmap编码相关可以参考一下  (bitmap编码在CDP中的应用 )
https://cloud.tencent.com/developer/news/683175

[明细圈人函数](https://help.aliyun.com/document_detail/216947.html)

**(1).生成用户映射**

构建一个映射表 mem_mapping_tf,把各类uid映射为全局唯一的一个32位的无符号整型,这里涉及两个问题,一个是idmapping(全域数据打通)的问题,保证准确性,当然我们映射其他的id,例如是设备id等,在CDP中的多id投放策略,idmapping之后还有一个ONEID,做起来容易,做好还是困难的,idmapping是一个工程(具体参考一下神策数据是怎么做的,之前看过一遍讲的不错),第二个问题,这张映射表怎么实现全局唯一??(id体系的建设)

如何针对亿级用户构建全局连续唯一数字 ID 标识?
1. 将全部亿级数据按照一定的规则分成多个子数据集（假设共有 M 个子数据集，每个子数据集各有 Ni 行数据，M >= 1，Ni >= 1，1<= i <= M）
2. 单个节点上使用行号生成函数 ROW_NUMBER() OVER (PARTITION BY col_1 ORDER BY col_2 ASC/DESC) 生成行号。每个子数据集中的行号都是从 1 开始，最大的行号为 Ni。
3. 对于第 1 个子数据集（M = 1）的数据，其最终行号是 1，2，3，4，…，N1；对于第 2 个子数据集（M = 2）的数据，其最终行号是 1 + N1，2 + N1，3 + N1，4 + N1，…，N2 + N1；对于第 3 个子数据集（M = 3）的数据，其最终行号是 1 + N1 + N2，2 + N1 + N2，3 + N1 + N2，4 + N1 + N2，…，N3 + N1 + N2；…以此类推

**(2).数据转换**

将原始行为数据中的uid映射为oneid
这一步的转化在spark/hive中完成

**(3).导入ck并压缩数据**

可能有什么坑??但是我不知道,前段时间用clickhouse导数据丢了..查了一下是主键的问题,其他的问题需要实践一下
```
#测试
CREATE TABLE nonodb.mid_action_bit_tf
(
eventTime DateTime,
event_type String,
oneid UInt32
)
ENGINE = MergeTree
ORDER BY (eventTime,oneid)
SETTINGS index_granularity = 8192;

insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:15:00','浏览',1);
insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:20:00','浏览',1);
insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:25:00','浏览',1);
insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:26:00','浏览',1);
insert into nonodb.mid_action_bit_tf values ('2022-01-03 11:29:00','下单',1);
insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:29:00','浏览',2);
insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:29:00','点击',2);
insert into nonodb.mid_action_bit_tf values ('2022-01-03 11:29:00','下单',2);
insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:29:00','浏览',3);
insert into nonodb.mid_action_bit_tf values ('2022-01-02 11:29:00','浏览',4);


CREATE TABLE nonodb.adm_action_bit_tf
(
eventTime Date,
event_type String,
user_bit AggregateFunction(groupBitmap,UInt32)
)
ENGINE = AggregatingMergeTree
ORDER BY eventTime
SETTINGS index_granularity = 10;

#压缩数据
insert into nonodb.adm_action_bit_tf
select  toDate(eventTime) ,
        event_type,
        groupBitmapState(oneid)
from  nonodb.mid_action_bit_tf
where toDate(eventTime) >= '2022-01-01' and toDate(eventTime) <= '2022-01-03'
group by  toDate(eventTime),event_type ;
```
**(4).查询**

ck中bitmap函数的应用


#####方案二. retention
该函数将一组条件作为参数，类型为1到32个UInt8类型的参数，用来表示事件是否满足特定条件。
```
retention(cond1, cond2, ..., cond32);
```
返回结果的表达式。返回值包括：
1，条件满足。
0，条件不满足。

```
SELECT
    sum(r[1]) AS r1,
    sum(r[2]) AS r2,
    r2/r1
FROM
(
select oneid,
       retention(toDate(eventTime)='2022-01-02' and event_type = '浏览',toDate(eventTime)='2022-01-03' and event_type = '下单') AS r
       from  nonodb.mid_action_bit_tf where   toDate(eventTime) >= '2022-01-02' and  toDate(eventTime)  <= '2022-01-03'
           group by oneid );
```
```
#案例:用户留存
SELECT
     "访问日期",
    SUM(r[1]) AS "第1天活跃用户",
    SUM(r[2])/SUM(r[1]) AS "次日留存",
    SUM(r[3])/SUM(r[1]) AS "3日留存",
    SUM(r[4])/SUM(r[1]) AS "7日留存"
FROM
    (
        WITH 
            first_day_table AS ( SELECT TIMESTAMP '2022-11-25 00:00:00' AS first_day)
        SELECT
            user_id,
            retention(
                DATE(event_time) = (SELECT DATE(first_day) FROM first_day_table),
                DATE(event_time) = (SELECT DATE(first_day +  INTERVAL 1 DAY) FROM first_day_table),
                DATE(event_time) = (SELECT DATE(first_day +  INTERVAL 2 DAY) FROM first_day_table),
                DATE(event_time) = (SELECT DATE(first_day +  INTERVAL 6 DAY) FROM first_day_table)
                ) AS r
            FROM    nonodb.mid_action_bit_tf
            WHERE   (event_time >= TIMESTAMP '2022-11-25 00:00:00')
            AND     (event_time <= TIMESTAMP '2022-11-25 00:00:00' + INTERVAL '6 day')
            GROUP BY user_id
        ) 
;
```
对比位图函数,还是位图函数方便的...
```
select event_type,bitmapAndCardinality(
	neighbor(bmp,-1), bmp
) retetion
FROM (
	select event_type,groupBitmapState(sipHash64(oneid)) bmp FROM nonodb.mid_action_bit_tf
	where   toDate(eventTime) >= '2022-01-02' and  toDate(eventTime)  <= '2022-01-03'
	GROUP BY event_type order by event_type 
)
```
#####方案三. 拉链表
从数据建模上考虑解决留存分析的问题:拉链表
步骤一: dw.traffic_aggr_session会话表计算今天登录的用户guid

步骤二:昨天的活跃表与今天的日活表full join;计算的规则:

first_dt guid range_start 规则是一致的,只要昨天有那就是昨天的,否则今天的(这种情况是新用户了)

range_end 规则:如果昨天登录了,今天没有登录,那就昨天日期,连续中断要封存,如果昨天没有但是有那就今天的(新用户),其他情况一律是昨天日期(昨天用户今天没有登录的情形,封闭区间保持原样)

步骤三:一种情形没有full jion上:之前存在的用户今天登陆的( max(range_end) != ‘9999-12-31’),所以要union all

从活跃表中获取这种用户的guid和first_dt与日活表left semi join
```
-- 类拉链表
create table dw.user_act_range(
first_dt    string,
guid        string,
range_start string,
range_end   string
)
partitioned by (dt string)
stored as parquet
tblproperties ("parquet.compress"="snappy");
```
```
-- 获取今日登陆的人数
-- 如果range_end都封闭了,但是今天登陆了,是不会出现在这里的,考虑使用 union all
select
  nvl(pre.first_dt, cur.cur_time) as first_dt,
  -- 如果之前存在那就取之前,之前不存在那就是新用户了
  nvl(pre.guid, cur.guid) as guid,
  nvl(pre.range_start, cur.cur_time) as range_start,
  case when pre.range_end = '9999-12-31'
  and cur.cur_time is null then pre.pre_time --  昨天登录,今天没有登录
  when pre.range_end is null then cur.cur_time -- 新用户(jion不上的)
  else pre.range_end -- 封闭区间保持原样
  end as range_end
from
  (
    select
      first_dt,
      guid,
      range_start,
      range_end,
      dy as pre_time
    from
      dw.user_act_range
    where
      dy = '2020-12-10'
  ) pre full
  join (
    select
      guid,
      max(dy) as cur_time
    from
      dw.traffic_aggr_session
    where
      dy = '2020-12-11'
    group by
      guid
  ) cur on pre.guid = cur.guid
union all
  -- 从会话层获取今日登陆的用户
  -- range_end封闭且今日登陆的情况
select
  first_dt as first_dt,
  o1.guid as guid,
  '2020-12-11' as range_start,
  '9999-12-31' as range_end
from
  (
    select
      guid,
      first_dt
    from
      dw.user_act_range
    where
      dy = '2020-12-10'
    group by
      guid,
      first_dt
    having
      max(range_end) != '9999-12-31'
  ) o1 -- 从会话层取出今天登陆的所有用户
  left semi
  join (
    select
      guid
    from
      dw.traffic_aggr_session
    where
      dy = '2020-12-11'
    group by
      guid
  ) o2 on o1.guid = o2.guid
```
```
-- 报表分析-连续活跃区间分布
-- 取最大的活跃区间

with x as (
select 
max(datediff(if(range_end='9999-12-31','2020-12-29',range_end),if(date_sub('2020-12-29' , 30)< range_start,range_start,date_sub('2020-12-29',30)))) as num
from 
dw.user_act_range
where dy='2020-12-17' and date_sub('2020-12-29',30) <= range_end
group by guid
)
select 
count(if(num<=10,1,null)) as continous_10,
count(if(num>10 and num<=20 ,1,null)) as continous_20,
count(if(num>20 and num<=30 ,1,null)) as continous_30
from 
x
```
#####bitMap思想
```
计算方案
步骤一:去重按照guid datime
步骤二:按照guid来分组聚合
使用sum(pow(2,datediff(时间,dt))) 类型为double,sum的结果是登陆数的和
把上述的结果转换为bin() 说明最近登陆的在bitmap的最右边,考虑使用lpad()来补全
考虑使用reverse来反转,这样最左边就是最近登陆的天数
步骤三:更新这个bitmap表
与今天的表进行join,on guid
对这个bitmap进行补全,补全的规则如下:
substr(1,1,30),今天和bitmap表的都有则+1 今天有bitmap表无则是新的guid 今天无bitmap表有说明没有登录+0
```

##3. Clickhouse常用位图函数
[ClickHouse之BitMap的使用](https://blog.csdn.net/qq_44666176/article/details/114291109)


#####1. bitmapBuild  无符号整数构建位图对象
```
select bitmapBuild([1,2,3,4,5])
```
#####2. bitmapToArray  将位图对象转化为整数数组
```
select bitmapToArray(user_bit)  from nonodb.adm_action_bit_tf where eventTime = '2022-01-02' and event_type = '浏览';
```
#####3. bitmapSubsetInRange   将位图指定范围转化为另外一个位图,相当于截取数据，左闭右开
```
select bitmapToArray(bitmapSubsetInRange(user_bit,toUInt32(1), toUInt32(4)))     from nonodb.adm_action_bit_tf where eventTime = '2022-01-02' and event_type = '浏览';
```
#####4. bitmapSubsetLimit  将位图指定范围转化为另外一个位图,(位图函数，起始点，限制条数)
```
select bitmapToArray(bitmapSubsetLimit(user_bit,toUInt32(1), toUInt32(4)))     from nonodb.adm_action_bit_tf where eventTime = '2022-01-02' and event_type = '浏览';
```
#####5. bitmapAnd  为两个bitmap对象进行与操作，参数只能是两个bitmap,取交集
```
select   bitmapToArray(bitmapAnd((select user_bit from nonodb.adm_action_bit_tf where eventTime = '2022-01-02' and event_type = '浏览'),(select 
user_bit from nonodb.adm_action_bit_tf where eventTime = '2022-01-03' and event_type = '下单')))
```
#####6. bitmapOr  为两个bitmap对象进行或操作，参数也同样只能是两个bitmap,取并集
```
select   bitmapToArray(bitmapOr((select user_bit from nonodb.adm_action_bit_tf where eventTime = '2022-01-02' and event_type = '浏览'),(select user_bit from nonodb.adm_action_bit_tf where eventTime = '2022-01-03' and event_type = '下单')))
```
#####7. groupBitmapMergeState  为多个bitmap对象进行或操作
```
官网未找到哇...groupBitmapState,做什么用的???(位图的构造方法)

select   bitmapToArray(groupBitmapMergeState(user_bit))  from nonodb.adm_action_bit_tf where eventTime = '2022-01-03' and event_type = '下单';
```
#####8.  bitmapXor 两个位图进行异或操作，返回一个新位图对象   去除两者重复值，其它值合并
```
select bitmapToArray(bitmapXor(bitmapBuild([1,2,3,4,5,6]), bitmapBuild([3,4,5,6,7,8]))) as res;
```
#####9.  bitmapAndnot 计算两个位图的差异，第一个位图剔除第二个位图相同值后的数值结果
```
select bitmapToArray(bitmapAndnot(bitmapBuild([1,2,3,4,5,6]), bitmapBuild([3,4,5,6,7,8]))) as res;
```
#####10. bitmapAndCardinality 两个位图进行与操作，得到对应位图对象元素的数量
```
SELECT bitmapAndCardinality(bitmapBuild([1,2,3,4,5,6]), bitmapBuild([3,4,5,6,7,8])) AS res;
```

