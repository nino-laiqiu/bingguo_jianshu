##1. 漏斗分析定义
[神策用户分析模型——漏斗分析的使用方法](https://www.sensorsdata.cn/blog/20180509/)

##2. 有序漏斗windowFunnel
[Clickhouse数据模型之有序漏斗分析](https://www.jianshu.com/p/48d5da97248c)

[Hologres漏斗分析函数](https://help.aliyun.com/document_detail/216948.html)

[Java UDF StarRocks Docs](https://docs.starrocks.com/zh-cn/main/using_starrocks/JAVA_UDF#%E6%AD%A5%E9%AA%A4%E4%BA%8C%EF%BC%9A%E5%BC%80%E5%8F%91-udwf-%E5%87%BD%E6%95%B0)

```
windowFunnel(window, [mode])(timestamp, cond1, cond2, ..., condN)
```
#####参数
window — 滑动窗户的大小，单位是秒。
mode - 这是一个可选的参数。
‘strict’ - 当 ‘strict’ 设置时，windowFunnel()仅对唯一值应用匹配条件。
timestamp — 包含时间的列。 数据类型支持： 日期, 日期时间 和其他无符号整数类型（请注意，即使时间戳支持 UInt64 类型，它的值不能超过Int64最大值，即2^63-1）。
cond — 事件链的约束条件。 UInt8 类型。

如果数据在不同的完成点具有多个事件链，则该函数将仅输出最长链的大小

#####数据
```
CREATE TABLE nonodb.log_action_tf
( uid Int32,
event_type String,
eventTime DateTime )
ENGINE = MergeTree
PARTITION BY uid
ORDER BY (uid, eventTime)
SETTINGS index_granularity = 8192;



insert into nonodb.log_action_tf values(1,'浏览','2022-01-02 11:00:00');
insert into nonodb.log_action_tf values(1,'点击','2022-01-02 11:10:00');
insert into nonodb.log_action_tf values(1,'下单','2022-01-02 11:20:00');
insert into nonodb.log_action_tf values(1,'支付','2022-01-02 11:30:00');
insert into nonodb.log_action_tf values(2,'下单','2022-01-02 11:00:00');
insert into nonodb.log_action_tf values(2,'支付','2022-01-02 11:10:00');
insert into nonodb.log_action_tf values(1,'浏览','2022-01-02 11:00:00');
insert into nonodb.log_action_tf values(3,'浏览','2022-01-02 11:20:00');
insert into nonodb.log_action_tf values(3,'点击','2022-01-02 12:00:00');
insert into nonodb.log_action_tf values(4,'浏览','2022-01-02 11:50:00');
insert into nonodb.log_action_tf values(4,'点击','2022-01-02 12:00:00');
insert into nonodb.log_action_tf values(5,'浏览','2022-01-02 11:50:00');
insert into nonodb.log_action_tf values(5,'点击','2022-01-02 12:00:00');
insert into nonodb.log_action_tf values(5,'下单','2022-01-02 11:10:00');
insert into nonodb.log_action_tf values(6,'浏览','2022-01-02 11:50:00');
insert into nonodb.log_action_tf values(6,'点击','2022-01-02 12:00:00');
insert into nonodb.log_action_tf values(6,'下单','2022-01-02 12:10:00');
```
#####示例
```
SELECT
    uid,
    windowFunnel(1800)(eventTime, event_type = '浏览', event_type = '点击', event_type = '下单', event_type = '支付') AS level
FROM
(
    SELECT
        eventTime,
        event_type,
        uid
    FROM nonodb.log_action_tf
)
GROUP BY uid

user  level
4	2
3	1
2	0
5	2
1	4
6	3

```

[ClickHouse数组函数](http://www.hnbian.cn/posts/b66ad81b.html)

[漏斗分析模型](https://blog.51cto.com/u_14291117/5277689)

分析"2022-01-02"这天 路径为“浏览->点击->下单->支付”的转化情况
```
select
        uid,
        arrayWithConstant(level, 1) levels,
        arrayJoin(arrayEnumerate( levels )) level_index
       from (
 SELECT
          uid,
          windowFunnel(1800)(
            eventTime,
            event_type = '浏览',
            event_type = '点击' ,
            event_type = '下单',
            event_type = '支付'
          ) AS level
        FROM (
          SELECT  eventTime,  event_type , uid
          FROM nonodb.log_action_tf
          WHERE toDate(eventTime) = '2022-01-02'
        ) t1
group by uid)

4	[1,1]	1
4	[1,1]	2
3	[1]	1
5	[1,1]	1
5	[1,1]	2
1	[1,1,1,1]	1
1	[1,1,1,1]	2
1	[1,1,1,1]	3
1	[1,1,1,1]	4
6	[1,1,1]	1
6	[1,1,1]	2
6	[1,1,1]	3

--- ---
select
       level_index,count(1) as ct
       from
           (
               select
        uid,
               arrayWithConstant(level, 1) levels,
               arrayJoin(arrayEnumerate(levels)) level_index
       from (
 SELECT
          uid,
          windowFunnel(1800)(
            eventTime,
            event_type = '浏览',
            event_type = '点击' ,
            event_type = '下单',
            event_type = '支付'
          ) AS level
        FROM (
          SELECT  eventTime,  event_type , uid
          FROM nonodb.log_action_tf
          WHERE toDate(eventTime) = '2022-01-02'
        ) t1
group by uid) t2
           )t3 group by level_index order by  level_index

--- ---
SELECT  transform(level_index,[1,2,3,4],['浏览','点击','下单','支付'],'其他') as event,
        count(1)
FROM (
select
        uid,
               arrayWithConstant(level, 1) levels,
               arrayJoin(arrayEnumerate(levels)) level_index
       from (
 SELECT
          uid,
          windowFunnel(1800)(
            eventTime,
            event_type = '浏览',
            event_type = '点击' ,
            event_type = '下单',
            event_type = '支付'
          ) AS level
        FROM (
          SELECT  eventTime,  event_type , uid
          FROM nonodb.log_action_tf
          WHERE toDate(eventTime) = '2022-01-02'
        ) t1
group by uid) t2 )
group by level_index
ORDER BY level_index ;
```
这个函数看起来很强大,但是少了点什么,我理解的流量分析滑动窗口不太一样

痛点:很显然,如果数据量超过100亿往上,clickhouse大概就拉了,比较好的方法还是结合bitmap进行编码,这里有篇文章可以参考一下的

[每天数百亿用户行为数据，美团点评怎么实现秒级转化分析？ ](https://tech.meituan.com/2018/03/20/user-funnel-analysis-design-build.html)


#####转化率计算
[neighbor](https://cloud.tencent.com/developer/article/1612576)
[uniqCombined | ClickHouse Docs](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/uniqcombined/)
[housepower/olap2018: 易观第二届OLAP漏斗算法大赛](https://github.com/housepower/olap2018)

```
select
    level,
    uniqCombined(uid) AS pv,
    neighbor(pv, -1) AS prev_pv,
    if (prev_pv = 0, -1, round(pv / prev_pv * 100, 3)) AS rate
        from (
SELECT
          uid,
          windowFunnel(1800)(
            eventTime,
            event_type = '浏览',
            event_type = '点击' ,
            event_type = '下单',
            event_type = '支付'
          ) AS level
        FROM (
          SELECT  eventTime,  event_type , uid
          FROM nonodb.log_action_tf
          WHERE toDate(eventTime) = '2022-01-02'
        ) t1
group by uid ) WHERE level > 0  GROUP BY level order by  level ;

```
上面这种是这样的,如果一个uid路径是4层,那么他可能走了第一层,着看产品的口径了,如果只算走了第一步的是1,走了四步的只算作4,那么就是上面这种口径
```
select
    level,
    sum(uid) AS pv,
    neighbor(pv, 1) AS prev_pv,
    if (prev_pv = 0, -1, round(pv / prev_pv * 100, 3)) AS rate
        from (
select
       level_index as level,count(1) as uid
       from
           (
               select
        uid,
               arrayWithConstant(level, 1) levels,
               arrayJoin(arrayEnumerate(levels)) level_index
       from (
 SELECT
          uid,
          windowFunnel(1800)(
            eventTime,
            event_type = '浏览',
            event_type = '点击' ,
            event_type = '下单',
            event_type = '支付'
          ) AS level
        FROM (
          SELECT  eventTime,  event_type , uid
          FROM nonodb.log_action_tf
          WHERE toDate(eventTime) = '2022-01-02'
        ) t1
group by uid) t2
           )t3 group by level_index order by  level_index)WHERE level > 0  GROUP BY level order by  level ;
```
##3. 无序漏斗分析
[groupArray | ClickHouse Docs](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/grouparray/)
[Array Functions | ClickHouse Docs](https://clickhouse.com/docs/en/sql-reference/functions/array-functions/#array-count)
[Clickhouse中的Array类型](https://www.jianshu.com/p/53480b149035)
```
案例:
select  groupArray(num)
from (
     select 1 as num union all  select  2 union all  select  3
         )

select  arrayCount(x -> x=1,[1,1,2,2] )

```
```
SELECT day           AS day,
       sum(level1_pv) AS level1_pv,
       sum(level2_pv) AS level2_pv,
       sum(level1_uv) as level1_uv,
       sum(level2_uv) as level2_uv
from (
select toDate(eventTime) as day,
       uid,
       groupArray(event_type) as events,
       arrayCount(x-> x = '浏览', events)  as level1_pv,
       if(has(events, '浏览'), arrayCount(x-> x = '点击', events),0) as level2_pv,
       hasAll(events, ['浏览'])  as level1_uv,
       hasAll(events, ['浏览','点击'])  as level2_uv
from nonodb.log_action_tf
where toDate(eventTime) >= '2021-01-01'
group by uid,toDate(eventTime) ) group by day order by  day;
```
