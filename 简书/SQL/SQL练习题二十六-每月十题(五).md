##301.值班排表
```
create table date_student (
     d  varchar(4)
);

create  table  if not exists  student1 (
     student varchar(5)
);

insert  into  date_student values ('0101'),('0102'),('0103'),('0104'),('0105'),('0106'),('0107');
insert  into  student1 values ('小明'),('小华'),('小红');
```
```
如上有日期和姓名表,要求依次组合生成一张排班表
0101	小明
0102	小华
0103	小红
0104	小明
0105	小华
0106	小红
0107	小明
```
```
select d,student
from (
         select d, row_number() over () %3  as rn
         from date_student
     ) t1
join (
     select student,row_number() over () %3 as rw
     from student1
    ) t2
    on t1.rn  = t2.rw
order by d ;

```
##302.TOP_N条件下计算最近14天趋势
SQL要求在kylin和presto中都能跑通

```
需求

有表test.table_0526
字段 常驻地resident_prov_name 人次person_num 时间 d 
现在要求获取到上海的前20的客源地(按人次获取客源地),获取这些客源地最近14天的热度趋势(以用户输入的结束时间-14天为准,关于热度趋势的计算就是获取占比即可)
```
请求的区间必须大于14天,否则无法返回14天的趋势
```
SELECT resident_prov_name, d, CAST(person_nums * 1.0 / SUM(person_nums) OVER (PARTITION BY resident_prov_name ) AS DECIMAL(9, 4)) AS ratio
FROM (
    SELECT resident_prov_name, d, person_nums
    FROM (
        SELECT resident_prov_name, d, person_nums, ROW_NUMBER() OVER (PARTITION BY resident_prov_name ORDER BY d DESC) AS rn
        FROM (
            SELECT resident_prov_name, d, person_nums
            FROM (
                SELECT resident_prov_name, d, person_nums, dense_rank() OVER (ORDER BY rw DESC) AS rn
                FROM (
                    SELECT resident_prov_name, d, person_nums, SUM(person_nums) OVER (PARTITION BY resident_prov_name ) AS rw
                    FROM (
                        SELECT resident_prov_name, d, SUM(person_num) AS person_nums
                        FROM  test.table_0526
                        WHERE d >= @startTime
                            AND d <= @endTime
                            AND to_country_name = @country
                            AND to_prov_name = @province
                            AND to_city_name = @city
                            AND resident_prov_name IS NOT NULL
                        GROUP BY resident_prov_name, d
                    ) a
                ) b
            ) c
            WHERE rn <= 20
        ) d
    ) e
WHERE rn <= 14
) f
```
如果用户请求的区间小于14天,也返还给用户14天的数据,怎么写(下面的这种写法查询一年的可以,但是时间区间过长kylin就报错了)
```
SELECT resident_prov_name,
       d,
       CAST(person_nums * 1.0 / SUM(person_nums) OVER (PARTITION BY resident_prov_name) AS DECIMAL(9, 4)) AS ratio
FROM
  (SELECT resident_prov_name,
          d,
          SUM(person_num) AS person_nums,
          ROW_NUMBER() OVER (PARTITION BY resident_prov_name
                             ORDER BY d DESC) AS rn
   FROM test.table_0526
   WHERE d >= '2019-01-01'
     AND d <= @endTime
     AND to_country_name = @country
     AND to_prov_name = '上海'
     AND to_city_name = '上海'
     AND resident_prov_name IN
       (SELECT resident_prov_name
        FROM ctripdi_prodb.adm_order_allbu_share_day
        WHERE d >= @startTime
          AND d <= @endTime
          AND to_country_name = @country
          AND to_prov_name = @province
          AND to_city_name = @city
          AND resident_prov_name IS NOT NULL
        GROUP BY resident_prov_name
        ORDER BY SUM(person_num) DESC
        LIMIT 20)
   GROUP BY resident_prov_name,
            d) a
WHERE rn <= 14
```
##303.同比计算
给定时间区间,对比去年相同区间,在top_20 view_spot_name中计算同比
```
select view_spot_name,(lag_persons - persons) * 1.0 /persons as ratio
from
(
SELECT  view_spot_name,persons,LAG(persons,1) over(partition by view_spot_name order by lag1 ) as lag_persons
FROM (
(
   SELECT 1 as lag1,view_spot_name,sum(person_num) AS persons
   FROM test.table0527
   WHERE d >= '2020-05-01'
     AND d<='2021-05-10'
     AND to_country_name = '中国'
     AND to_prov_name = '上海'
     AND to_city_name = '上海'
     AND view_spot_name IS NOT NULL
   GROUP BY view_spot_name
   ORDER BY sum(person_num) DESC
   LIMIT 20
    )
     
    union all
    (
        SELECT 2,view_spot_name,sum(person_num)
        FROM test.table0527
        WHERE
            d >= concat(CAST(CAST(substring('2020-05-01', 1, 4) AS int) - 1 AS varchar), substring('2020-05-01', 5, 7))
            AND d <= concat(CAST(CAST(substring('2021-05-10', 1, 4) AS int) - 1 AS varchar), substring('2021-05-10', 5, 7))
            AND to_country_name = '中国'
            AND to_prov_name = '上海'
            AND to_city_name = '上海'
            AND view_spot_name IS NOT NULL
        GROUP BY view_spot_name
    )
    ) a 
    ) b  where lag_persons is not null
```
这里的问题是
```
concat(CAST(CAST(substring('2020-05-01', 1, 4) AS int) - 1 AS varchar), substring('2020-05-01', 5, 7))
```
不走rowkey,如果字段的枚举值过大就会无法执行,所以比较有效的方法是与前端沟通让他们传入时间
或者在底表先进行计算,改变粒度(之前的粒度为天,现在为月份)

另外kylin和presto的时间处理函数
```
kylin
TIMESTAMAPADD(DAY,-14,date '2021-01-01')

presto
date_add('day',-14,cast('2021-01-01' as date)
date_add('day',-14, date'2021-01-01' )
```
##304.分区表批量补数
新建的一张分区表要补数,一般是一天一天的补数,但是这样比较麻烦,采用动态分区的方法来补数,效率将大大提升,set动态分区的参数,d>= 分区的初始时间,需要注意的是要先跑几个分区对比一下数据,当然这种动态分区补数的方法也是有缺陷的,动态分区的时间不是分区d就不能使用,因为可能其他的时间维度与分区不是一一对应的

如下一般的set参数
```
set hive.merge.mapfiles=true;      --map端
set hive.merge.mapredfiles=true;       --reduce端
set hive.map.aggr=true;
SET hive.auto.convert.join=true;
SET hive.exec.max.dynamic.partitions.pernode=9000;
SET hive.exec.max.dynamic.partitions=9000;
set hive.exec.parallel=true;
```

##306.TOP_20%条件下计算词频
```
有表test_table2,包含 master_hotel_name(酒店名称),,tag_name(评价词),
score(酒店评分),pplrt_num(评价词的词频)

现在的需求是要统计前20%酒店(按评分来排序)的评价词的词频
test_table2是我加工的表,来自于一张评价词词频表和酒店相关扩展表(包含酒店名称,评分等)

```
```
SELECT tag_name as tag,sum(pplrt_nums) as pplrt_sum
FROM (
    SELECT master_hotel_name,tag_name,pplrt_nums, rn * 1.0 / counts AS a
    FROM (
        SELECT master_hotel_name,tag_name, score,pplrt_nums, ROW_NUMBER() OVER (ORDER BY score desc ) AS rn, COUNT(1) OVER () AS counts
        FROM (
            SELECT master_hotel_name,tag_name,SUM(score) as score ,SUM(pplrt_num) AS pplrt_nums
            FROM  test_table2
            WHERE d >=  '2021-05-27' and d <= '2021-05-27'
            GROUP BY master_hotel_id, master_hotel_name,tag_name
        )
    )
)
WHERE a <= 0.2 
group by tag_name
ORDER by pplrt_sum desc
LIMIT 50
```
##307.每隔15min的记录数
```
CREATE TABLE if not exists  test_0521(
ID INT ,
Times datetime
);

INSERT INTO test_0521
VALUES(1,'2021-05-24 11:01:45'),
      (2,'2021-05-24 11:03:15'),
      (3,'2021-05-24 11:05:34'),
      (4,'2021-05-24 11:09:23'),
      (5,'2021-05-24 11:17:45'),
      (6,'2021-05-24 11:19:15'),
      (7,'2021-05-24 11:29:34'),
      (8,'2021-05-24 11:37:23');
```
```
如何用SQL查询出每个15分钟的记录数,预期结果如下
2021-05-24 11:00:00,4
2021-05-24 11:15:00,3
2021-05-24 11:30:00,1
```
.....用mysql写着实有点复杂,这边的主要问题就是对时间的处理,不知道mysql有没有特定的时间处理函数集
```
select
cast(concat(substr(date_add(date_sub( Times ,INTERVAL minute(Times) minute ),
INTERVAL floor(minute(Times) / 15) * 15  minute),1,17),'00') as datetime),
count(1)
from test_0521
group by 
cast(concat(substr(date_add(date_sub( Times ,INTERVAL minute(Times) minute ),
INTERVAL floor(minute(Times) / 15) * 15  minute),1,17),'00') as datetime);
```
我的思路如下,获取时间的分钟数与15进行比较,这样就正确处理了时间分钟,接着要对时间的秒进行处理,我这里是使用substr来切分concat来补充
##308.时间段重叠取数问题
```
某直播业务会记录主播开播及关播时间。需要计算出每天的峰值及峰值出现时间。数据表如下,粒度为每秒
| user_id | start_time | end_time |
| ------- | ------------------- | ------------------- |
| 1 | 2020-01-07 01:03:01 | 2020-01-07 03:03:01 |
| 2 | 2020-01-07 01:05:01 | 2020-01-07 02:03:01 |
| 3 | 2020-01-07 01:09:01 | 2020-01-07 05:03:01 |
| 1 | 2020-01-07 09:10:01 | 2020-01-07 10:03:01 |
```
这道题主要考虑就是写一个循环,hive中提供了一个udaf函数,之前写过
```
WITH a AS (
        SELECT '2021-04-20' AS depart_time, '2021-04-10' AS arrive_time
        UNION ALL
        SELECT '2021-03-11', '2021-03-09'
    )
SELECT pos
FROM a
    LATERAL VIEW POSEXPLODE(SPLIT(SPACE(DATEDIFF(TO_DATE(a.depart_time), TO_DATE(a.arrive_time)) - 1), ' ')) tf AS pos,val 
```
把时间转换为时间戳即可比较,然后在转过来
```
select unix_timestamp('2020-09-15 10:35:33');
select from_unixtime(1000000000);
```
##309.连续空余座位
```
CREATE TABLE T0527
(
seat_id INT,
free INT
)

INSERT INTO T0527 VALUES 
(1,1),
(2,0),
(3,1),
(4,1),
(5,1)
```
seat_id 字段是一个自增的整数，free 字段是布尔类型（'1' 表示空余， '0' 表示已被占据）。连续空余座位的定义是大于等于 2 个连续空余的座位
对于如上样例，你的查询语句应该返回如下结果。
```
3
4
5
```
```
select seat_id
from (
         select SEAT_ID,
                FREE,
                LAG(free, 1, 0) over (ORDER BY seat_id)  AS LAG_1,
                LEAD(free, 1, 0) over (order by seat_id) as lead_1
         from T0527
     ) t1
where free <> 0 and (lead_1 =1 or LAG_1 =1 );
```
```
select  seat_id
from (
         select seat_id,free,
                seat_id - lag(seat_id, 1) over (order by seat_id) as lag_1
         from T0527
     ) t1
where free<> 0 and lag_1 =1 ;
```
方法三可使用自连接判断abs
##310.发货单号的最值
```
CREATE TABLE T0520 
(
shipid INT,
paydate DATE,
payno INT
)

INSERT INTO T0520 VALUES(1001,'2020/11/2',5);
INSERT INTO T0520 VALUES(1001,'2020/11/2',3);
INSERT INTO T0520 VALUES(1001,'2020/11/3',1);
INSERT INTO T0520 VALUES(1001,'2020/11/3',3);
INSERT INTO T0520 VALUES(1002,'2020/11/9',1);
INSERT INTO T0520 VALUES(1002,'2020/11/9',4);
INSERT INTO T0520 VALUES(1002,'2020/11/8',3);
INSERT INTO T0520 VALUES(1002,'2020/11/8',2);
```
使用两种方法,查询出每个发货单号(shipid)，最早付款时间(paydate)和最小付款单号(payno)

```
select shipid, paydate, payno
from (
         select shipid,
                paydate,
                payno,
                row_number() over (partition by shipid order by paydate asc ,payno asc ) as rn
         from T0520
     ) t1
where  rn = 1;
```

```
select a.shipid,a.paydate,min(a.payno)
from T0520 a
join (
    select shipid,min(paydate) as min_date
    from T0520
    group by shipid
    ) b on a.paydate = b.min_date and a.shipid = b.shipid
group by a.shipid,a.paydate;
```
