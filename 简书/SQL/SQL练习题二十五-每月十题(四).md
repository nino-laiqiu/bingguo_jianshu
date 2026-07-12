##291.数据补充
```
create table null_attribution_lag(
stock_id string,
date_cal string,
price double
)
row format delimited fields terminated by ",";


data

300121,2020-10-01,10.65
300121,2020-10-02,11.65
300121,2020-10-03,12.15
300121,2020-10-04,10.00
300121,2020-10-05,10.99
300121,2020-10-06,
300121,2020-10-07,
300121,2020-10-08,10.01
300125,2020-10-01,8.65
300125,2020-10-02,9.65
300125,2020-10-03,9.11
300125,2020-10-04,
300125,2020-10-05,8.34
300125,2020-10-06,
300125,2020-10-07,
300125,2020-10-08,10.65

load data local inpath "/root/practice_data/null_attribution_lag" into table null_attribution_lag;

得到如下结果,如果为null则price为上面的不为null的那个值

+-----------+-------------+--------+------------+
| stock_id  |  date_cal   | price  | price_new  |
+-----------+-------------+--------+------------+
| 300121    | 2020-10-01  | 10.65  | 10.65      |
| 300121    | 2020-10-02  | 11.65  | 11.65      |
| 300121    | 2020-10-03  | 12.15  | 12.15      |
| 300121    | 2020-10-04  | 10.0   | 10.0       |
| 300121    | 2020-10-05  | 10.99  | 10.99      |
| 300121    | 2020-10-06  | NULL   | 10.99      |
| 300121    | 2020-10-07  | NULL   | 10.99      |
| 300121    | 2020-10-08  | 10.01  | 10.01      |
| 300125    | 2020-10-01  | 8.65   | 8.65       |
| 300125    | 2020-10-02  | 9.65   | 9.65       |
| 300125    | 2020-10-03  | 9.11   | 9.11       |
| 300125    | 2020-10-04  | NULL   | 9.11       |
| 300125    | 2020-10-05  | 8.34   | 8.34       |
| 300125    | 2020-10-06  | NULL   | 8.34       |
| 300125    | 2020-10-07  | NULL   | 8.34       |
| 300125    | 2020-10-08  | 10.65  | 10.65      |
+-----------+-------------+--------+------------+
```
```

select stock_id,date_cal,price,max(price) over(partition by stock_id,sum_lable  order by date_cal) as price_new from(
    select stock_id,date_cal,price,
    sum(lable_price) over(partition by stock_id order by date_cal) as sum_lable
    from
         (select 
         stock_id,date_cal,price,if(price is null ,0,1) as lable_price
    from null_attribution_lag) as a1
    ) as a2;

```
```
第二种方法：
通过lag或者lead函数，将null值滞后一项或者提前一项，将符合题意的分组在一起，再通过max函数进行补充数据
放弃：选择通过join的方式

如lag将滞后项与null放为一组

select
   stock_id,date_cal,lable_price,price,if(price is null or lag_price is null ,0,1) lable_01 
   from (
   select
   stock_id,date_cal,price,
   if(price is null ,0 ,1) as lable_price,
   lag(price,1,price) over(partition by stock_id  order by date_cal) as lag_price
   from null_attribution_lag
    ) as a1;


join的方式解决该类问题：

with a as (select
   stock_id,date_cal,price
from null_attribution_lag where price is null)
select * from a join null_attribution_lag on a.date_cal < null_attribution_lag.date_cal
and null_attribution_lag.price is not null
;

```
##292.不用 UNION 操作符实现 UNION 的效果
```

WITH a AS(
SELECT 1 AS id,'aaa' AS v
UNION ALL
SELECT 2 AS id,'bbb' AS v
UNION ALL
SELECT 2 AS id,'bbb' AS v
),b AS (
SELECT 1 AS id,'aaa' AS v
UNION ALL
SELECT 3 AS id,'ccc' AS v
)
select * from  a  union all  select * from  b ;

1	aaa
2	bbb
2	bbb
1	aaa
3	ccc
```
假设在结果集里已经存在 a 表的数据，现在要把 b 表的数据也加进来,使用left join来实现(打标签)
```
WITH a AS(
SELECT 1 AS id,'aaa' AS v
UNION ALL
SELECT 2 AS id,'bbb' AS v
UNION ALL
SELECT 2 AS id,'bbb' AS v
),b AS (
SELECT 1 AS id,'aaa' AS v
UNION ALL
SELECT 3 AS id,'ccc' AS v
),c AS (
SELECT 0 AS flag
UNION ALL
SELECT 1 AS flag
)
select ifnull(a.id,b.id),ifnull(a.v,b.v) from c left join a on c.flag = 1 left join b on c.flag = 0 ;
```
要实现union 的效果直接group by 或者distinct即可
##293.打印一个月的日历
```
select day(curdate()); -- 这个月过了多少天
select date_add(curdate(),interval  - day(curdate()) + 1 DAY ); -- 月初
select last_day(curdate()); -- 月末
```
首先生成1-31天的id,用于生成这个月的所有天数(递归或者辅助表)
```
with  recursive x(id) as (
    select 1 as id
    union all
    select id + 1 from x  where id < 31
    )
select
date_add(first_day , interval id - 1 DAY )
from (
    select date_add(curdate(),interval  - day(curdate()) + 1 DAY ) as first_day
                ) a ,x
where x.id <= day(last_day(curdate()));
```
```
with  recursive x(id) as (
    select 1 as id
    union all
    select id + 1 from x  where id < 31
    )
    SELECT
  MAX(IF(wkday = 0, day_index, '')) AS '一',
  MAX(IF(wkday = 1, day_index, '')) AS '二',
  MAX(IF(wkday = 2, day_index, '')) AS '三',
  MAX(IF(wkday = 3, day_index, '')) AS '四',
  MAX(IF(wkday = 4, day_index, '')) AS '五',
  MAX(IF(wkday = 5, day_index, '')) AS '六',
  MAX(IF(wkday = 6, day_index, '')) AS '日'
FROM(
select
week(day_m,1) as wk ,
     weekday(day_m) as wkday,
     day(day_m) as day_index,
     day_m
from (
select
date_add(first_day , interval id - 1 DAY ) as day_m
from (
    select date_add(curdate(),interval  - day(curdate()) + 1 DAY ) as first_day
                ) a ,x
where x.id <= day(last_day(curdate())) ) b ) c group by wk  ;
```
##294.确定序列里缺失值的范围
先来构造有缺失值的 seq 表，可以用 SQL 派生出这个表。
```
WITH seq AS(
SELECT 1 AS id
UNION ALL SELECT 2
UNION ALL SELECT 3
UNION ALL SELECT 5
UNION ALL SELECT 6
UNION ALL SELECT 7
UNION ALL SELECT 8
UNION ALL SELECT 12
UNION ALL SELECT 13
UNION ALL SELECT 15
UNION ALL SELECT 18
UNION ALL SELECT 19
UNION ALL SELECT 20
)
```
我们观察数据可知，seq 表中目前最大的数是 20，缺失的值有：4、9、10、11、14、16、17。
```

SELECT
  START,
  STOP
FROM
  (SELECT
    s.id + 1 AS START,
    (SELECT
      MIN(id) - 1
    FROM
      seq AS xx
    WHERE xx.id > s.id) AS STOP
  FROM
    seq AS s
    LEFT JOIN seq AS r
      ON s.id = r.id - 1
  WHERE r.id IS NULL) AS t
WHERE STOP IS NOT NULL ;
```
```
select id- 1 ,tag+1  from  (
SELECT id,lag(id,1,id) over () as tag from seq  ) t
where id - tag > 1;
```

##295.打印矩阵 (一)
实现效果
```
A B C D E  F
1 2 3 4 5
6 7 8 9 10
..... 
```
1. 排在同一行的数是因为它们本身除 5 后再向上取整得到的是同一个值；
2. 排在同一列的数是因为它们本身对 5 求余的结果一致；
3. 同一行的数从左到右是递增；同样，同一列的数从上到下也是递增的。
```
WITH recursive t_seq (num) AS
(SELECT
  1 AS num
UNION
ALL
SELECT
  num + 1 AS num
FROM
  t_seq
WHERE num < 25),
x0 AS
(SELECT
  num,
  CEIL(num / 5) AS group_no
FROM
  t_seq),
x1 AS
(SELECT
  group_no AS row_no,
  MAX(IF(num % 5 = 1, num, NULL)) AS A,
  MAX(IF(num % 5 = 2, num, NULL)) AS B,
  MAX(IF(num % 5 = 3, num, NULL)) AS C,
  MAX(IF(num % 5 = 4, num, NULL)) AS D,
  MAX(IF(num % 5 = 0, num, NULL)) AS E
FROM
  x0
GROUP BY group_no)
SELECT A ,B ,C ,D ,E FROM x1 ;
```
##296.打印矩阵 (二)
1. 有一张 5 x 5 的表格，我们要往这张表格中填充 1~25 的数字；
2. 如果是奇数行，则从左到右填充数字；如果是偶数行，就需要按从右到左的顺序填入数字。
3. 先从表格的左上角（即第一行第一列）填入数字 “1”，在第一行第二列填入“2”，直到把第一行填满；
4. 当上一行填满的时候，就开始往下一行填数据。比如，第二行要从右往左依次填入“6”、“7”、“8”、“9”、“10”。
5. 循环反复，直到所有空格都填满数字。
```
WITH recursive t_seq (num) AS
(SELECT
  1 AS num
UNION ALL
SELECT
  num + 1 AS num
FROM
  t_seq
WHERE num < 25),
     x0 AS
(SELECT
  num,
  CEIL(num / 5) AS group_no
FROM
  t_seq),
 x1 AS
(SELECT
  *,
  IF(group_no % 2 = 0, - 1 * num, num) AS ordered,
  row_number () over () AS seq
FROM
  x0
ORDER BY group_no,
  ordered),
 x2 AS
(SELECT
  group_no AS row_no,
  MAX(IF(seq % 5 = 1, num, NULL)) AS A,
  MAX(IF(seq % 5 = 2, num, NULL)) AS B,
  MAX(IF(seq % 5 = 3, num, NULL)) AS C,
  MAX(IF(seq % 5 = 4, num, NULL)) AS D,
  MAX(IF(seq % 5 = 0, num, NULL)) AS E
FROM
  x1
GROUP BY group_no)

SELECT
  A, B, C, D, E
FROM
  x2;
```
注意使用if(-)来重新排序的方法来转换序号的顺序
##297.从字符串中提取数字
```
  id  v       
------  --------
     1  123     
     2  abc     
     3  1d3     
     4  0       
     5  123.0   
     6  0123    
     7  01#123  
     8  0$123   
```
希望得到如下结果
```
    id  v       mix     
------  ------  --------
     1  123     123     
     3  1d3     13      
     4  0       0       
     5  123.0   1230    
     6  0123    0123    
     7  01#123  01123   
     8  0$123   0123
```
```
WITH RECURSIVE chaos (id, v, s, seq) AS 
(SELECT 
  id,
  v,
  SUBSTR(v, 1, 1) AS s,
  1 AS seq 
FROM
  mix 
UNION ALL 
SELECT 
  id,
  v,
  SUBSTR(v, seq + 1, 1),
  seq + 1 
FROM
  chaos 
WHERE seq <= CHAR_LENGTH(v) - 1) 

SELECT 
  id,
  v,
  GROUP_CONCAT(s 
    ORDER BY seq SEPARATOR '') AS mix 
FROM
  chaos 
WHERE s >= '0' 
  AND s <= '9' 
GROUP BY v,
  id 
ORDER BY id 
```
```
SELECT 
  id,
  v,
  GROUP_CONCAT(s 
    ORDER BY seq SEPARATOR '') AS mix 
FROM
  (SELECT 
    mix.id AS id,
    v,
    SUBSTR(v, t20.id, 1) AS s,
    t20.id AS seq 
  FROM
    mix,
    t20 
  WHERE t20.id <= CHAR_LENGTH(v) 
  ORDER BY v,
    t20.id) t 
WHERE s >= '0' 
  AND s <= '9' 
GROUP BY v,
  id 
ORDER BY id 
```
##298.统计奇数
统计ip每行奇数的个数
```
use mydata;
CREATE TABLE T0416(
IP INT,
NUM1 INT,
NUM2 INT,
NUM3 INT
);
INSERT T0416
SELECT 1,3333,4442,221
UNION ALL
SELECT 2,65,24,96;

select * from T0416;
```
```
select * ,NUM3%2 + NUM2%2 + NUM1%2 as '奇数'from T0416;
```
##299.连续字符串
```
CREATE TABLE T0414
(
    val VARCHAR(50)
);
insert into T0414(val) values('A10000003');
insert into T0414(val) values('A10000001');
insert into T0414(val) values('A10000002');
insert into T0414(val) values('A10000011');
insert into T0414(val) values('A10000004');
insert into T0414(val) values('A10000006');
insert into T0414(val) values('A10000009');
insert into T0414(val) values('A10000010');
insert into T0414(val) values('A10000012');

得到如下结果
A10000001-A10000004,A10000006,A10000009-A10000012
```
我写的有点复杂......
```
with x as (
select val,sum(tag) over (order by val) as rang
from  (
select val,num,if (num - lag(num,1,num  ) over () = 1 ,0,1) as tag
from (
select val ,cast(substr(val,2)  as SIGNED ) as num  from  T0414 order by val ) a ) b )

select group_concat(nm) from (
select concat(min(val),'-',max(val)) as nm from x group by rang having  count(1) >1
union  all
select  min(val) from x group by rang having count(1) =1  order by nm ) t11 ;
```
##300.机房状态(列转行)

```
CREATE TABLE T0412A
(
ID INT,
机房 VARCHAR(20)
)
CREATE TABLE T0412B
(
ID INT,
机房ID INT,
主机名称 VARCHAR(20),
主机状态 INT
)
 
INSERT INTO T0412A VALUES (1,'机房A')
INSERT INTO T0412A VALUES (2,'机房B')
INSERT INTO T0412A VALUES (3,'机房C')
 
INSERT INTO T0412B VALUES (1,1,'主机A',1)
INSERT INTO T0412B VALUES (2,1,'主机B',1)
INSERT INTO T0412B VALUES (3,1,'主机C',2)
INSERT INTO T0412B VALUES (4,2,'主机D',0)
INSERT INTO T0412B VALUES (5,2,'主机E',1)

得到如下结果
机房 0 1  2
```` 
```
select
a.机房,ifnull(sum(if(b.主机状态 = 0,1,0)),0) as '0',
       ifnull(sum(if(b.主机状态 = 1,1,0)),0) as '1',
       ifnull(sum(if(b.主机状态 = 2,1,0)),0) as '2'
from T0412A a left join T0412B b on a.ID = b.机房ID
group by a.机房;
```
此处主要回顾一下case when 和if语法,if语法不用说太多,注意一下if(负数)的用法即可还有count(null)...也就这些了,我经常混淆case  when的用法这里简单地梳理一下
1. 在group by中使用case when 是用来分类的,注意在select中也要有相同内容的case when 
2. case when 和 case .. when 无本质的区别,主要是是否复用值的区别,如下
```
CASE 字段 WHEN 值1 THEN 值1 [WHEN 值2 THEN 值2] [ELSE 值] END
CASE WHEN 条件表达式 THEN 值1 [WHEN 条件表达式 [and or] 条件表达式THEN 值2] [ELSE 值] END
```
3. 在select中的case when 和if语法是一致的,如果参与聚合分类判断,则case when 就比if方便,主要也是用来分类的
4. 在用于列转行/行转列的使用在case when 外面来嵌套一层聚合函数,用来分类,这里要注意一下细节

