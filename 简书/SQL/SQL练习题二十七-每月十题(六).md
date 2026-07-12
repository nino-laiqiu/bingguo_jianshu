##311. 打标签合并
```
山西	交城	100	平安:40
山西	交城	100	非平安:60

合并成如下
山西	交城	100	平安:40  非平安:60
```
```
create table a0628 (
    Secondaryinstitutions varchar(4) ,
    Tertiaryinstitutions   varchar(4),
    num int,
    sources varchar(10)
);
insert into a0628 values ('山西','交城',100,'平安:40'),('山西','交城',100,'非平安:60');
```
```
select
Secondaryinstitutions, Tertiaryinstitutions, num,
       MAX(CASE  WHEN tag = '平安' then sources end) as '平安',
       MAX(CASE  WHEN tag = '非平安' then sources end) as '非平安'
from
     (
select
*,if ( left(sources,2)  = '平安' ,'平安' ,'非平安' ) as tag
from a0628 )  t1
group by Secondaryinstitutions,Tertiaryinstitutions,num;
```
##312. explode()和posexplode()
参考文章
https://blog.csdn.net/dzysunshine/article/details/101110467?utm_medium=distribute.pc_relevant.none-task-blog-2~

#####1.-- explode的使用
select explode(split(concat_ws(',','1','2','3','4','5','6','7','8','9'),',')) ;
```
col
1 
2 
3 
4 
5 
6 
7 
8 
9 
```
关于explode的用法其实质上就是行转列,把一行转为多行
关于concat_ws的用法是第一个字段是分隔符,后split进行切分
#####2. --  LATERAL VIEW POSEXPLODE的使用
```
WITH a AS (
        SELECT '2021-04-20' AS depart_time, '2021-04-10' AS arrive_time
        UNION ALL
        SELECT '2021-03-11', '2021-03-09'
    )
SELECT tf.*
FROM a
    LATERAL VIEW POSEXPLODE
 (SPLIT(SPACE(DATEDIFF(TO_DATE(a.depart_time), TO_DATE(a.arrive_time)) - 1), ' ')) tf 
 ```

#####3. -- LATERAL VIEW explode的使用(explode(array (or map))
1. 示例一
```
WITH x AS (
        SELECT 'a' AS id, '12_14_15' AS age
        UNION ALL
        SELECT 'b', '12_14_15'
    )
SELECT age_single
FROM x
    LATERAL VIEW explode(split(x.age, '_')) temp AS age_single
```
2. 示例二,这里不使用with来展示,是由于这边的spark引擎不支持在with中使用array类型
```
SELECT col
FROM (
    SELECT 'UU', array('A', 'B', 'C') AS arr
) t
    LATERAL VIEW explode(t.arr) tf AS col
```
3.示例三
```
select t1.*,sp  from  (select 1 ) as t1    
lateral view explode(split(concat_ws(',','1','2','3','4','5','6','7','8','9'),',')) t as sp
```
这里本来是使用with的,但是发现可能是版本问题导致,所以我就使用嵌套一层select来处理了
4. 说明
lateral view explode() as temp 相当于生成了一张虚拟表,后与原表进行笛卡尔积
对于这两个UDTF函数的难点在于理解虚拟表的逻辑,以及在处理中使用正则表达式来获取正确的逻辑

参考文档,这里是SQL复杂处理的核心函数,掌握了这些就非常nice
https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF

##313. 同比环比
如上,按月份聚合统计不同业务的同比/环比情况
这个SQL就是之前conf上写的SQL,这里复制一下,具体的看那篇博客即可
```
select
lag,j_lag,
MAX(case when substring(j_month,6,7) = '01' then relative  end ) as m1,
MAX(case when substring(j_month,6,7) = '02' then relative  end ) as m2,
MAX(case when substring(j_month,6,7) = '03' then relative  end ) as m3,
MAX(case when substring(j_month,6,7) = '04' then relative  end ) as m4,
MAX(case when substring(j_month,6,7) = '05' then relative  end ) as m5,
MAX(case when substring(j_month,6,7) = '06' then relative  end ) as m6,
MAX(case when substring(j_month,6,7) = '07' then relative  end ) as m7,
MAX(case when substring(j_month,6,7) = '08' then relative  end ) as m8,
MAX(case when substring(j_month,6,7) = '09' then relative  end ) as m9,
MAX(case when substring(j_month,6,7) = '10' then relative  end ) as m10,
MAX(case when substring(j_month,6,7) = '11' then relative  end ) as m11,
MAX(case when substring(j_month,6,7) = '12' then relative  end ) as m12
from 
(
SELECT lag, '环比' AS j_lag, j_month
    , (order_price - lead_price_relative) * 1.0 / lead_price_relative AS relative
FROM (
    SELECT lag, j_month, order_price
        , lead(order_price, 1, NULL) OVER (PARTITION BY lag ORDER BY j_month DESC) AS lead_price_relative
    FROM (
        SELECT concat(lag_country, bu) AS lag, j_month, order_price
        FROM (
            SELECT bu
                , CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END AS lag_country, substring(d, 0, 7) AS j_month
                , round(SUM(order_price), 0) AS order_price
            FROM table
            WHERE d >= '2019-01-01'
                AND bu IN ('机票', '酒店', '度假')
                AND d <> '4000-01-01'
            GROUP BY bu, substring(d, 0, 7), CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END
        ) a
    ) b
    WHERE lag IS NOT NULL
) c
UNION ALL
(
SELECT lag, '同比' , j_month
    , (order_price - lead_price_basis) * 1.0 / lead_price_basis 
FROM (
    SELECT lag, j_month, order_price
        , lead(order_price, 1, NULL) OVER (PARTITION BY lag ORDER BY substring(j_month, 6, 7) DESC,substring(j_month, 0, 4) desc) AS lead_price_basis
    FROM (
        SELECT concat(lag_country, bu) AS lag, j_month, order_price
        FROM (
            SELECT bu
                , CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END AS lag_country, substring(d, 0, 7) AS j_month
                , round(SUM(order_price), 0) AS order_price
            FROM table
            WHERE d >= '2019-01-01'
                AND bu IN ('机票', '酒店', '度假')
                AND d <> '4000-01-01'
            GROUP BY bu, substring(d, 0, 7), CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END
        ) a
    ) b
    WHERE lag IS NOT NULL
) c
)
) 
WHERE substring(j_month, 0, 4) = '2021'
GROUP by lag,j_lag
order by substring(lag,3,4),substring(lag,0,2),j_lag
```
关于这道SQL主要用了如下知识点:标签,lead偏移,order by复合排序
##314. 列转行
上边同比环比的超级简化版,简单复习一下列转行,注意max(case when的时候不要使用else null 可能会出问题)
对于行转列,事实上就是对数据结果的处理,而关于explode的用法其实质上就是行转列,把一行转为多行.
```
2020	1	9.1
2020	2	9.2
2020	3	9.3
2020	4	9.4
2021	1	8.1
2021	2	8.2
2021	3	8.3
2021	4	8.4

将如上转化为
year    M1   M2   M3   M4 
2020	9.1	9.2	9.3	9.4
2021	8.1	8.2	8.3	8.4

````
```
create  table if not exists  b0628(
    year varchar(4), month varchar(1) ,amount double
);

INSERT INTO b0628 VALUES
('2020','1',9.1),
('2020','2',9.2),
('2020','3',9.3),
('2020','4',9.4),
('2021','1',8.1),
('2021','2',8.2),
('2021','3',8.3),
('2021','4',8.4);
```
```
select
year,
        max(case  when month = 1 then amount end ) as M1,
        max(case  when month = 2 then amount end ) as M2,
        max(case  when month = 3 then amount end ) as M3,
        max(case  when month = 4 then amount end ) as M4
from b0628 group by year order by year ;
```
##315. LAG&LEAD
偏移量函数的简单应用,写到这里我想起来了,sum等开窗函数可以限制窗口的大小,好久没用了

```
数据
id  NUM
1	5
2	11
3	0
4	-2
5	2
6	9
7	1
8	-4
9	-7
```
要求当NUM中的数同时大于上下两行数据,返回是,当num中的数据小于上下两行中的任意一行,返回否
```
CREATE TABLE T0611 (
ID INT,
Num INT
);

INSERT INTO T0611 VALUES(1,5);
INSERT INTO T0611 VALUES(2,11);
INSERT INTO T0611 VALUES(3,0);
INSERT INTO T0611 VALUES(4,-2);
INSERT INTO T0611 VALUES(5,2);
INSERT INTO T0611 VALUES(6,9);
INSERT INTO T0611 VALUES(7,1);
INSERT INTO T0611 VALUES(8,-4);
INSERT INTO T0611 VALUES(9,-7);
```
```
select
ID,Num,if(Num>lag_1 and Num>lead_1,'是','否') as result
from (
         select id,
                num,
                lag(Num, 1) over (order by id ) as lag_1,
                lead(Num, 1) over (order by ID) as lead_1
         from T0611
     ) t1
order by  id ;
```
##316. 中位数
表中保存了数字的值以及其个数,求取中位数,在此表中,数字为0,0,0,0,0,0,0,1,2,2,2.3,所以中位数为(0+0)/2,这道题目在leedcode上有,记得之前写过一遍
```
0	7
1	1
2	3
3	1
```
```
create  table if not exists c0629(
 Number int,Frequency INT
);
insert into  c0629 values (0,7),(1,1),(2,3),(3,1);
```

请编写一个查询来查找所有数字的中位数并将结果命名为 median 。注意：什么是中位数？当一串数字是奇数个时，例如8,3,5,1,4。我们按顺序排列后为：1,3,4,5,8。那么4就是中位数
当一串数字为偶数个时，例如8,3,5,1,4,2。我们按顺序排列后为：1,2,3,4,5,8。那么(3+4)/2=3.5就是中位数。
```
select
    avg(n.Number) as median
from
    Numbers as n
where
    n.Frequency >= abs(
        (select sum(Frequency) from Numbers where Number <= n.Number) - 
        (select sum(Frequency) from Numbers where Number >= n.Number)
    )
```
```
select avg(number) as median
from
(select Number, frequency,
        sum(frequency) over(order by number asc) as total,
        sum(frequency) over(order by number desc) as total1
from Numbers
order by number asc)as a
where total>=(select sum(frequency) from Numbers)/2
and total1>=(select sum(frequency) from Numbers)/2
```
可考虑使用其他方法,使用中位数的定义从位置逻辑和值和上来理解
##317. 使用sparksql解析json类型的字符串 
#####-- 方法一,使用spark自带的函数get_json_object解析,但是效率比较低
SELECT get_json_object(label_value_text, '$.BUS')
FROM (
    SELECT '{"BUS":1,"CAR":1,"DIY":1,"FLT":1,"HTL":1,"PKG":1,"TRN":1,"TTD":1}' AS label_value_text
) t1
LIMIT 10
#####-- 方法二,from_json的用法
select a.k 
from (
select from_json('{"k": "xiecheng", "v": 1.0}','k STRING, v STRING',map("","")) as a
) t1 
#####-- 方法三,json_tuple的使用方法,多项获取,一次解析获取全部的json数据
SELECT json_tuple(label_value_text, 'BUS','CAR')
FROM (
    SELECT '{"BUS":1,"CAR":0,"DIY":1,"FLT":1,"HTL":1,"PKG":1,"TRN":1,"TTD":1}' AS label_value_text
) t1
#####-- 方法四,自主开发的自定义函数,其实很简单,就是使用spark写json处理逻辑,打包上传即可,难点是怎么高效处理(对嵌套的处理,使用的简洁性,但是一般公司都开发了直接用就好了.......)

参考文档:
https://blog.csdn.net/lsr40/article/details/79399166
https://blog.csdn.net/lsr40/article/details/103020021
https://cloud.tencent.com/developer/article/1451308
https://cloud.tencent.com/developer/article/1032532
https://blog.csdn.net/weixin_38750084/article/details/93498986?utm_medium=distribute.pc_relevant.none-task-blog-2~
##318. 怪异的排序
```
1	张三	1
1	李四	2
2	张三	1
2	李四	2
3	张三	1
3	李四	2
4	王五	1
4	赵柳	2
5	王五	1
5	赵柳	2
6	王五	1
6	赵柳	2
7	麻七	1
7	赖八	2
8	麻七	1
8	赖八	2
9	麻七	1
9	赖八	2
```
转变为如下
```
1	张三	1
2	张三	1
3	张三	1
1	李四	2
2	李四	2
3	李四	2
4	王五	1
5	王五	1
6	王五	1
4	赵柳	2
5	赵柳	2
6	赵柳	2
7	麻七	1
8	麻七	1
9	麻七	1
7	赖八	2
8	赖八	2
9	赖八	2
```
```
create table weird_order
(
机台号 int,
姓名 char(20),
班组 int
);
INSERT INTO `weird_order` (`机台号`, `姓名`, `班组`)
VALUES
  (1, '张三', 1),
  (1, '李四', 2),
  (2, '张三', 1),
  (2, '李四', 2),
  (3, '张三', 1),
  (3, '李四', 2),
  (4, '王五', 1),
  (4, '赵柳', 2),
  (5, '王五', 1),
  (5, '赵柳', 2),
  (6, '王五', 1),
  (6, '赵柳', 2),
  (7, '麻七', 1),
  (7, '赖八', 2),
  (8, '麻七', 1),
  (8, '赖八', 2),
  (9, '麻七', 1),
  (9, '赖八', 2);
```
这里的难点就是要对姓名排序,使用row_number来获取编号的最小值,就是姓名的新序号
```
SELECT
  t.机台号,
  t.姓名,
  t.班组
FROM
  weird_order t
JOIN
    (SELECT 姓名, MIN(原始序号) AS 新序号
    FROM (SELECT *, row_number () over () AS '原始序号'
      FROM
        weird_order) t
    GROUP BY 姓名) tt
    ON tt.姓名 = t.姓名
ORDER BY 新序号,
  班组,
  机台号;
```
##319. 打印成绩单
有两个表：Students 和 Grades，Students 记录了学生的分数，Grades 存储了分数和绩点的对应关系。
Students 包含了三个字段：ID、Name（姓名）、Marks（分数）。
Grades 有三个字段：Grade（绩点）、Min_Mark（最小分值）、Max_Mark（最大分值）
根据这两张表，生成一份学习成绩单，这份成绩单要包含这三个字段：Name、Grade、Mark 。

成绩单需要满足以下几个要求：
绩点低于 8 的学生不显示名字，使用 NULL 代替。
成绩单都得先按照绩点降序排序，对于绩点相同的记录，如果绩点 >= 8，就再按照姓名的字母顺序排序；如果绩点 < 8 ，就再按照分数升序排序。
```
SELECT
  IF(grade >= 8, name, NULL) AS name,
  grade,
  marks
FROM
  Students
  INNER JOIN Grades
    ON marks BETWEEN min_mark
    AND max_mark
ORDER BY grade DESC,
  IF(grade >= 8, name, marks)
```
留一个思考题，如果把排序的条件“如果绩点 < 8 ，就再按照分数升序排序”中的“升序排序”改成“降序排序”，你会怎么做？(直接加一个负号?)
##320. 完全包含
求左边完全包含右边的
```
SQL数据库开发	数据库
北京	中国
新加坡城	新加坡
```
```
CREATE TABLE T0608
(
A VARCHAR(100),
B VARCHAR(100)
);

INSERT INTO T0608 VALUES
('SQL数据库开发','数据库'),
('北京','中国'),
('新加坡城','新加坡');
```
```
select  *   from T0608 where  A like  concat('%',B,'%');
```
