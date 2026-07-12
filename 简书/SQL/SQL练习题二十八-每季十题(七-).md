>坚持了半年,没啥精力找SQL题了,感觉基本上除了递归,函数应用,UDF/UDAF就没啥可写的了,这里就把每月10题改成每季十题吧.......其实也就是凑个数

>计划把之前找的SQL题分模块归纳一下,计划第四季度完成

>上班有时候看了一下算法大佬写的SQL,其实也就那样,主要就是他们的表多,就是逻辑复杂一点,也没什么复杂SQL处理

##321. 用户关系探索
设表名：table0
现有城市网吧访问数据，字段：网吧id，访客id（身份证号），上线时间，下线时间：
规则1: 如果有两个用户在一家网吧的前后上下线时间在10分钟以内，则两人可能认识
规则2: 如果这两个用户在三家以上网吧出现【规则1】的情况，则两人一定认识

该城市上网用户中两人一定认识的组合数。
```
create table table0(
    wid string,
    uid string,
    ontime string,
    offtime string
)
row format delimited fields terminated by '\t'
```
```
insert into table0 values (1,110001,'2020-01-01 12:10:00','2020-01-01 12:30:00');
insert into table0 values (1,110001,'2020-01-01 12:35:00','2020-01-01 12:40:00');
insert into table0 values (1,110002,'2020-01-01 12:50:00','2020-01-01 12:55:00');
insert into table0 values (1,110001,'2020-01-01 13:00:00','2020-01-01 13:10:00');
insert into table0 values (1,110003,'2020-01-01 12:15:00','2020-01-01 13:15:00');

insert into table0 values (2,110001,'2020-01-02 12:10:00','2020-01-02 12:30:00');
insert into table0 values (2,110001,'2020-01-02 12:35:00','2020-01-02 12:40:00');
insert into table0 values (2,110002,'2020-01-02 12:50:00','2020-01-02 12:55:00');
insert into table0 values (2,110003,'2020-01-02 13:00:00','2020-01-02 13:00:00');

insert into table0 values (3,110001,'2020-01-03 12:10:00','2020-01-03 12:30:00');
insert into table0 values (3,110003,'2020-01-03 12:15:00','2020-01-03 12:40:00');
insert into table0 values (3,110001,'2020-01-03 12:50:00','2020-01-03 12:55:00');
insert into table0 values (3,110002,'2020-01-03 13:00:00','2020-01-03 13:10:00');
```
```
select count(uid) as com_cnt
from(
    select uid
          ,count(1) as flag
    from(
        select wid
              ,uid
              ,abs(unix_timestamp(ontime,'yyyy-MM-dd HH:mm:ss')-unix_timestamp(lag(ontime,1,'1970-01-01 08:00:00') over(partition by wid order by ontime),'yyyy-MM-dd HH:mm:ss')) / 60 ontime_diff
              ,abs(unix_timestamp(offtime,'yyyy-MM-dd HH:mm:ss')-unix_timestamp(lag(offtime,1,'1970-01-01 08:00:00') over(partition by wid order by offtime),'yyyy-MM-dd HH:mm:ss')) / 60 offtime_diff
        from table0
    ) m
    where ontime_diff<=10 and offtime_diff<=10
    group by uid
) n
where flag>=3
```
这解法貌似不对把,如果出现在不同网吧的两个人不是同两个人,但是求取的时候就认为他们是同两个人??这里应该要lag用户id进行concat拼接,这里的难点就是排序了,可以去重后group by wid来拼接uid

或者这样,对uid继续排序后拼接
```sql
select  concat_ws(',',sort_array(array('1','3','2')))
```
>https://mp.weixin.qq.com/s/U1YiZF8eRlF7yQtCXxEQzw

这个答案是肯定没问题的,就是自连接的SQL性能较差
```
设表名：table0
字段：wid ， uid ，ontime ，offtime
select
    id,
    count(distinct wid) c
from
    (select
        wid,
        concat(t0.uid,t1.uid) as id
    from
        (select
            wid,
            uid,
            unix_timestamp(ontime,'yyyyMMdd HH:mm:ss') as ontime,
            unix_timestamp(offtime,'yyyyMMdd HH:mm:ss') as offtime
        from 
            table0
        )t0
    join
        (select
            wid,
            uid,
            unix_timestamp(ontime,'yyyyMMdd HH:mm:ss') as ontime,
            unix_timestamp(offtime,'yyyyMMdd HH:mm:ss') as offtime
        from 
            table0
        )t1
    on t0.wid=t1.wid
        and t0.uid>t1.uid
        and (abs(t0.ontime-t1.ontime)<10*60 or abs(t0.offtime-t1.offtime)<10*60)
        
    )t0
group by 
    id
having
    c>=3
```
##322. 留存用户
分别统计6.1日活跃玩家
流失率次日回归率 6/2未登陆但是6/3日登陆的玩家 / 6/2未登陆的玩家总数
流失率3日回归率 6/2-6/3未登陆但是6/4日登陆的玩家 / 6/2-6/3未登陆的玩家总数
流失率30日回归率 6/2-6/30未登陆但是7/1日登陆的玩家 / 6/2-6/30未登陆的玩家总数
```
id   login_date
用户id   用户登陆时间
一天同一个用户会登陆多次
```

```
-- 由于uid过多,随机选取1000个uid来分析

with x as (
select  distinct  id from app_lo_log where  login_date >= '2021-06-01' and login_date <= '2021-06-02' limit 1000 )

select
       login_date,allover,
sum(allover) over (rows  between  unbounded preceding and 1 preceding)
from (
-- 统计每天的回归玩家
         select count(id) as allover,
                login_date
         from (
-- 取用户在区间最早的登录时间,原因是最早的登录时间不在某个时间区间内,那么这个用户属于login_date的回归玩家
                  select id,
                         date(min(login_date)) as login_date
                  from app_lo_log
                  where login_date >= '2021-06-02'
                    and login_date <= '2021-07-02'
                    and id in (select id from x)
                  group by id
              ) t1
         group by login_date
     ) t2 
order by  login_date
```
同学提供的写法,但是感觉在求取比率的时候分母不太对,分母求取的应该是在当前之前从未登陆的人,也不太好求取因为存在从未登陆的人(不限制结束时间),另一个问题就是窗口函数改成`sum(allover) over (rows  between  current row and unbounded following )`或者`sum(allover) over (rows  between  1 following and unbounded following )`应该更符合业务方提出的需求的含义


##323. 偶数对
有一串字符串,有几个类似的,00,11,22,33这样的对子,比如"14,55,00"导出的结果就是2;"14,01,02"导出的结果就是0,使用SQL来解决

思路就是偶数对和11的关系
mysql不支持split函数比较麻烦,这里使用spark
```
select
id ,sum(if(v1%11 = 0 ,1,0))
from 
(
select 
id,v1 
from 
(
SELECT id,v 
FROM (
    SELECT 1 as id , '19,21,32,22|22,9' as st 
) t
    LATERAL VIEW explode(split(st, '\\|')) tf AS v  
    ) t1 
    LATERAL VIEW explode(split(v, ',')) tf1 AS v1
    ) t2 
    group by id
```
能不能改进一下解析一步到位?
>mysql实现行转列的功能 https://www.cnblogs.com/gered/p/10797012.html
##324. 控制窗口大小的使用
有所遗忘,关键平时也没有这种需求的......感觉一天不如一天了.......

**PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点**

```sql
#例如
select name,orderdate,cost,
sum(cost) over() as fullagg, --所有行相加
sum(cost) over(partition by name) as fullaggbyname, --按name分组，组内数据相加
sum(cost) over(partition by name order by orderdate) as fabno, --按name分组，组内数据累加 
sum(cost) over(partition by name order by orderdate rows between unbounded preceding and current row) as mw1   --和fabno一样,由最前面的起点到当前行的聚合 
sum(cost) over(partition by name order by orderdate rows between 1 preceding and current row) as mw2,   --当前行和前面一行做聚合 
sum(cost) over(partition by name order by orderdate rows between 1 preceding and 1 following) as mw3,   --当前行和前边一行及后面一行 
sum(cost) over(partition by name order by orderdate rows between current row and unbounded following) as mw4  --当前行及后面所有行 
over (order by score range between 2 preceding and 2 following) 窗口范围为当前行的数据幅度减2加2后的范围内的数据求和。
from order; 
```
##325. 母店下所有子店
有表dimhtl
```
htlid   type   typeid
子酒店id   如果是0就是母酒店,不是0就是子酒店   母酒店id
```
现在求取某母酒店下所有的子酒店
```
select
t1.htlid,t2.htlid
from
dimhtl t1
join  (select * from dimhtl where type <> 0) t2
on t1.type = 0
and t1.typeid = t2.typeid
```

##326. 流失率口径
最近有算有个城市流失率,流失率口径 =这段时间内这些预定人预定了其他城市订单的自然人 /某段时间内取消过去这个城市的订单,但是没有去这个城市的自然人(预订人)

我们不从算法的行程角度来看看问题,就单纯地从数仓的角度回答,对于这个口径的较高执行效率的SQL写法
##327.小数点前0丢失的解决方法
[小数点前0丢失的解决方法](https://www.jianshu.com/p/950110ec52e9)
```
select
substring(ratio,1,7) || '%'  as ratio
from
(
select case when position('.' in ratio) = 1 then '0'|| ratio else ratio end as ratio
) t1 
```
##328.两种分组top类型需求
1. 求取客流量在top10城市的,年龄区间占比
```
           上海   北京  广州 ......
60前
60后
70
80
90
....
```
2. 求取城市行政区下top10的客源省份的占比
```
               上海   北京  广州 ......
黄浦区
浦东新区
长宁区
...... 
```
其实这两类需求只是主次不同,第一个需求列为1,第二个需求行为1

##329. 同时存在的AID
有表如下
```
AID  BID
1     2
1     5
2     1
2     3
2     5
3     4
3     2
3     5
```
列AID和BID是一对多的关系,希望将BID中同时存在3和5的AID找出来,预计的结果是AID = 2 

很简单,having判断一下数量,注意的细节是同一个AID对于重复的BID的处理
```
having sum(case when  bid = 3 then 1 esle 0 end ) >= 1 and 
sum(case when  bid = 5 then 1 esle 0 end ) >= 1
```
##330. Type类型均分
```
Type Num
A  2
B  3
C  2
```
希望将每个Type按Num进行均等分,结果如下
```
A 1
A 1
B 1
B 1
B 1
C 1
C 1
```
方法有很多:比如使用递归来做,构建row_number() 自增键,使用炸裂函数等

方法一:使用hive中的炸裂函数
```
with x as (
select 'A' as type  ,2 as Num  
union all 
select 'B' , 3 
union all 
select 'C' ,2 
)
select
type,'1' AS m 
from 
x 
LATERAL VIEW POSEXPLODE(SPLIT(SPACE(Num - 1 ),' ')) AS pos,val
```
SPACE 函数用于生成由N个空格字符的字符串，split按照' ' 进行切分，注意Num数问题

方法二:使用mysql的递归CTE用法,在大数据中递归式一般是不用的,这里复习一下mysql的递归用法
>https://www.yiibai.com/mysql/recursive-cte.html

示例:
```
WITH RECURSIVE cte_count (n)
AS (
      SELECT 1
      UNION ALL
      SELECT n + 1
      FROM cte_count
      WHERE n < 3
    )
SELECT n
FROM cte_count;

```
参考:
```
WITH RECURSIVE sql_0904 (Type,Num)
as (
    select  Type,Num from T0904
    union all
    select  Type ,Num -1 from sql_0904 where Num >= 2
    )
select  Type ,1  FROM sql_0904 ORDER BY Type;
```
补充:
同期在kylin中的写法
```
concat(cast(cast(substring('2021-01-01',1,4) as int ) -1 as varchar),
substring('2021-01-01',5,7))
```
未来28天在kylin中的写法
```
TIMESTAMPADD(DAY, 28, CURRENT_DATE) 
```
