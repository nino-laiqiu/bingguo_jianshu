##51.项目员工3

>写 一个 SQL 查询语句，报告在每一个项目中经验最丰富的雇员是谁。如果出现经验年数相同的情况，请报告所有具有最大经验年数的员工。
```
项目表 Project：

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| project_id  | int     |
| employee_id | int     |
+-------------+---------+
(project_id, employee_id) 是这个表的主键
employee_id 是员工表 Employee 的外键
员工表 Employee：

+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| employee_id      | int     |
| name             | varchar |
| experience_years | int     |
+------------------+---------+
employee_id 是这个表的主键
 
写 一个 SQL 查询语句，报告在每一个项目中经验最丰富的雇员是谁。如果出现经验年数相同的情况，请报告所有具有最大经验年数的员工。

查询结果格式在以下示例中：

Project 表：
+-------------+-------------+
| project_id  | employee_id |
+-------------+-------------+
| 1           | 1           |
| 1           | 2           |
| 1           | 3           |
| 2           | 1           |
| 2           | 4           |
+-------------+-------------+

Employee 表：
+-------------+--------+------------------+
| employee_id | name   | experience_years |
+-------------+--------+------------------+
| 1           | Khaled | 3                |
| 2           | Ali    | 2                |
| 3           | John   | 3                |
| 4           | Doe    | 2                |
+-------------+--------+------------------+

Result 表：
+-------------+---------------+
| project_id  | employee_id   |
+-------------+---------------+
| 1           | 1             |
| 1           | 3             |
| 2           | 1             |
+-------------+---------------+
employee_id 为 1 和 3 的员工在 project_id 为 1 的项目中拥有最丰富的经验。在 project_id 为 2 的项目中，employee_id 为 1 的员工拥有最丰富的经验。

```
```
--我的窗口写法
select 
project_id,t.employee_id
from 
(
select 
t1.employee_id ,
project_id,
dense_rank() over(partition by project_id order by experience_years desc ) as rn 
from
Employee t1
join Project  t2
on t1.employee_id = t2.employee_id
) t
where rn =1
```
```
-- 使用子查询,先查找出每个项目中最大工龄
//max聚合函数只返回一条记录,用where的联合字段获取分组最大值
select p.project_id, p.employee_id
from Project as p inner join Employee as e on p.employee_id = e.employee_id
where (p.project_id, e.experience_years) in (select p.project_id, max(experience_years)
                                            from Project as p inner join Employee as e
                                            on p.employee_id = e.employee_id
                                            group by p.project_id)
```
##52.销售分析1
>编写一个 SQL 查询，查询总销售额最高的销售者，如果有并列的，就都展示出来
```
产品表：Product

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| product_id   | int     |
| product_name | varchar |
| unit_price   | int     |
+--------------+---------+
product_id 是这个表的主键.
销售表：Sales

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| seller_id   | int     |
| product_id  | int     |
| buyer_id    | int     |
| sale_date   | date    |
| quantity    | int     |
| price       | int     |
+------ ------+---------+
这个表没有主键，它可以有重复的行.
product_id 是 Product 表的外键.
 

编写一个 SQL 查询，查询总销售额最高的销售者，如果有并列的，就都展示出来。

查询结果格式如下所示：

Product 表：
+------------+--------------+------------+
| product_id | product_name | unit_price |
+------------+--------------+------------+
| 1          | S8           | 1000       |
| 2          | G4           | 800        |
| 3          | iPhone       | 1400       |
+------------+--------------+------------+

Sales 表：
+-----------+------------+----------+------------+----------+-------+
| seller_id | product_id | buyer_id | sale_date  | quantity | price |
+-----------+------------+----------+------------+----------+-------+
| 1         | 1          | 1        | 2019-01-21 | 2        | 2000  |
| 1         | 2          | 2        | 2019-02-17 | 1        | 800   |
| 2         | 2          | 3        | 2019-06-02 | 1        | 800   |
| 3         | 3          | 4        | 2019-05-13 | 2        | 2800  |
+-----------+------------+----------+------------+----------+-------+

Result 表：
+-------------+
| seller_id   |
+-------------+
| 1           |
| 3           |
+-------------+
Id 为 1 和 3 的销售者，销售总金额都为最高的 2800
```
```
-- 我的写法窗口写法,简单题
select 
seller_id
from 
(
select
seller_id,
dense_rank() over(order by sum(price) desc) as rn 
from
Sales
group by seller_id
) t
where rn =1
```
```
-- all的用法
select
seller_id
from 
Sales
group by 
seller_id
having sum(price) >= all( select sum(price) from Sales group by seller_id order by sum(price) desc )
```
##53.销售分析2(☆)

>编写一个 SQL 查询，查询购买了 S8 手机却没有购买 iPhone 的买家。注意这里 S8 和 iPhone 是 Product 表中的产品。
```
-- 我的写法,过去笨拙..
with x as (
select
buyer_id,product_name
from
Product t1 join  Sales t2 on t1.product_id = t2.product_id
)
select
buyer_id
from
(
select
distinct
buyer_id 
from 
x
where product_name = 'S8'                                                            
union all 
(
select 
distinct
buyer_id 
from 
x
where buyer_id not in (select buyer_id from x where product_name = 'iPhone')
)
) t3
group by  buyer_id having count(1) >1
```
```
--哇!!没想到...
-- sum方法
select
buyer_id
from 
Sales t1
join Product t2
on t1.product_id = t2.product_id
group by buyer_id
having sum(product_name = 'S8') >0 and sum(product_name = 'iPhone') =0
```
```
-- where的嵌套
select distinct buyer_id
from Sales
where buyer_id not in (
    select buyer_id
    from Product p,Sales s
    where p.product_id = s.product_id
    and p.product_name = 'iPhone'
) 
and buyer_id in (
    select buyer_id
    from Product p,Sales s
    where p.product_id = s.product_id
    and p.product_name = 'S8'
)
```
##54.销售分析3
>编写一个SQL查询，报告2019年春季才售出的产品。即仅在2019-01-01至2019-03-31（含）之间出售的商品。
```
-- 我的写法
select 
t2.product_id,product_name
from 
Product t1
join 
(
select
product_id
from 
Sales
group by product_id
having sum( if( sale_date between date_format('2019-01-01','%Y-%m-%d') and date_format('2019-03-31','%Y-%m-%d'),0,1) ) = 0 
) t2
on t1.product_id = t2.product_id
```
```
-- 学习一下METHOD 1,我怎么想不到....
METHOD 1
SELECT P.PRODUCT_ID, PRODUCT_NAME
FROM PRODUCT AS P
INNER JOIN SALES AS S
ON P.PRODUCT_ID = S.PRODUCT_ID
GROUP BY 1
HAVING MIN(SALE_DATE) >= '2019-01-01'
AND MAX(SALE_DATE) <= '2019-03-31';

METHOD 2
-- 说明既要满足存在于条件一,又要满足存在于条件二
SELECT PRODUCT_ID, PRODUCT_NAME
FROM PRODUCT
WHERE PRODUCT_ID IN (SELECT PRODUCT_ID
                     FROM SALES
                     WHERE SALE_DATE BETWEEN '2019-01-01' AND '2019-03-31')
AND PRODUCT_ID NOT IN (SELECT PRODUCT_ID
                       FROM SALES
                       WHERE SALE_DATE < '2019-01-01'
                       OR SALE_DATE > '2019-03-31');

```
上面的三道题目使用not in 把我绕糊涂了...学习一下在having中使用sum ,if 之类的过滤条件的使用,来替换not in where用法
##55.游戏的玩法5(min的窗口的使用)☆
>编写一个 SQL 查询，报告每个安装日期、当天安装游戏的玩家数量和第一天的留存时间。
```
Activity 活动记录表

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
（player_id，event_date）是此表的主键
这张表显示了某些游戏的玩家的活动情况
每一行是一个玩家的记录，他在某一天使用某个设备注销之前登录并玩了很多游戏（可能是 0）
 

我们将玩家的安装日期定义为该玩家的第一个登录日。

我们还将某个日期 X 的第 1 天留存时间定义为安装日期为 X 的玩家的数量，他们在 X 之后的一天重新登录，除以安装日期为 X 的玩家的数量，四舍五入到小数点后两位。

编写一个 SQL 查询，报告每个安装日期、当天安装游戏的玩家数量和第一天的留存时间。

查询结果格式如下所示：

Activity 表：
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-01 | 0            |
| 3         | 4         | 2016-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result 表：
+------------+----------+----------------+
| install_dt | installs | Day1_retention |
+------------+----------+----------------+
| 2016-03-01 | 2        | 0.50           |
| 2017-06-25 | 1        | 0.00           |
+------------+----------+----------------+
玩家 1 和 3 在 2016-03-01 安装了游戏，但只有玩家 1 在 2016-03-02 重新登录，所以 2016-03-01 的第一天留存时间是 1/2=0.50
玩家 2 在 2017-06-25 安装了游戏，但在 2017-06-26 没有重新登录，因此 2017-06-25 的第一天留存时间为 0/1=0.00
```
```
-- 我的写法超时了...优化一下
--我的思路先获取每个用户首次登录的日期作为A
-- 以上表A的字段与原始表join,条件是用户相等,且时间差一天,获取的是留存信息,然后group by 首次登录的日期两张表join获取答案...
select 
x1.event_date as install_dt  ,s1 as installs , nvl(s2/s1,0) as Day1_retention
from
(
select 
event_date,count(1) as s1
from 
(
select
player_id,min(to_char(event_date,'yyyy-mm-dd')) as event_date
from
Activity
group by player_id
) t1
group by event_date ) x1
left join 
(
select 
t2.event_date,count(1) as s2
from 
(
select
player_id,min(to_char(event_date,'yyyy-mm-dd')) as event_date
from
Activity
group by player_id
) t2
join 
Activity 
on t2.player_id = Activity.player_id and t2.event_date = to_char(Activity.event_date -1 ,'yyyy-mm-dd')
group by t2.event_date ) x2
on x1.event_date = x2.event_date
```
```
-- 使用窗口函数,哇没想到.....
select install_dt,count(distinct player_id)installs,
       round(sum(if(datediff(event_date,install_dt)=1,1,0))/count(distinct player_id),2) Day1_retention 
from 
(
    select *,min(event_date) over(partition by player_id) install_dt
    from Activity
)t1   
group by install_dt

```
```
select a1.install_dt,
       count(*) installs,
       round(count(a2.event_date)/count(*),2) Day1_retention
from(
    select player_id,min(event_date) install_dt
    from Activity
    group by player_id
) a1
left join Activity a2
    on a1.player_id = a2.player_id and datediff(a2.event_date,a1.install_dt)=1
group by a1.install_dt
```
```
select
    A1.event_date as install_dt,
    count(*) as installs,
    round(count(A2.event_date) / count(*)*1.0 , 2) as Day1_retention

from Activity A1
left join Activity A2
    on A1.player_id = A2.player_id
    and A1.event_date = DATE_ADD(A2.event_date, INTERVAL -1 day)
where (A1.player_id, A1.event_date) in (select player_id, min(event_date) 
                                        from Activity 
                                        group by player_id)
group by A1.event_date
order by 1
```
##56.小众书籍(on 和 where的区别)
>你需要写一段 SQL 命令，筛选出过去一年中订单总量 少于10本 的 书籍 。
注意：不考虑 上架（available from）距今 不满一个月 的书籍。并且 假设今天是 2019-06-23 。

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| book_id        | int     |
| name           | varchar |
| available_from | date    |
+----------------+---------+
book_id 是这个表的主键。
订单表 Orders：

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| order_id       | int     |
| book_id        | int     |
| quantity       | int     |
| dispatch_date  | date    |
+----------------+---------+
order_id 是这个表的主键。
book_id  是 Books 表的外键。
 

你需要写一段 SQL 命令，筛选出过去一年中订单总量 少于10本 的 书籍 。

注意：不考虑 上架（available from）距今 不满一个月 的书籍。并且 假设今天是 2019-06-23 。

 

下面是样例输出结果：

Books 表：
+---------+--------------------+----------------+
| book_id | name               | available_from |
+---------+--------------------+----------------+
| 1       | "Kalila And Demna" | 2010-01-01     |
| 2       | "28 Letters"       | 2012-05-12     |
| 3       | "The Hobbit"       | 2019-06-10     |
| 4       | "13 Reasons Why"   | 2019-06-01     |
| 5       | "The Hunger Games" | 2008-09-21     |
+---------+--------------------+----------------+

Orders 表：
+----------+---------+----------+---------------+
| order_id | book_id | quantity | dispatch_date |
+----------+---------+----------+---------------+
| 1        | 1       | 2        | 2018-07-26    |
| 2        | 1       | 1        | 2018-11-05    |
| 3        | 3       | 8        | 2019-06-11    |
| 4        | 4       | 6        | 2019-06-05    |
| 5        | 4       | 5        | 2019-06-20    |
| 6        | 5       | 9        | 2009-02-02    |
| 7        | 5       | 8        | 2010-04-13    |
+----------+---------+----------+---------------+

Result 表：
+-----------+--------------------+
| book_id   | name               |
+-----------+--------------------+
| 1         | "Kalila And Demna" |
| 2         | "28 Letters"       |
| 5         | "The Hunger Games" |
+-----------+--------------------+
```
```
-- 这题有毒,提交了5,6次
select b.book_id,b.name
from
books b left join orders o
on 
b.book_id=o.book_id
where
b.available_from<'2019-05-23'
group by b.book_id
having ifnull(sum(if(o.dispatch_date<'2018-06-23',0,quantity)),0)<10
order by b.book_id
```

>注意条件中on 和 where的区别
1、 on条件是在生成临时表时使用的条件，它不管on中的条件是否为真，都会返回左边表中的记录。
2、where条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有left join的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。
书籍表 Books：
例如:
```
select b.book_id, b.name
from books b left join orders o on b.book_id = o.book_id and  o.dispatch_date >= '2018-06-23'
where b.available_from < '2019-05-23'
group by b.book_id
having ifnull(sum(o.quantity),0) < 10
```
##57.每日新用户统计
>编写一个 SQL 查询，以查询从今天起最多 90 天内，每个日期该日期首次登录的用户数。假设今天是 2019-06-30
```
Traffic 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| activity      | enum    |
| activity_date | date    |
+---------------+---------+
该表没有主键，它可能有重复的行。
activity 列是 ENUM 类型，可能取 ('login', 'logout', 'jobs', 'groups', 'homepage') 几个值之一。

编写一个 SQL 查询，以查询从今天起最多 90 天内，每个日期该日期首次登录的用户数。假设今天是 2019-06-30.

查询结果格式如下例所示：

Traffic 表：
+---------+----------+---------------+
| user_id | activity | activity_date |
+---------+----------+---------------+
| 1       | login    | 2019-05-01    |
| 1       | homepage | 2019-05-01    |
| 1       | logout   | 2019-05-01    |
| 2       | login    | 2019-06-21    |
| 2       | logout   | 2019-06-21    |
| 3       | login    | 2019-01-01    |
| 3       | jobs     | 2019-01-01    |
| 3       | logout   | 2019-01-01    |
| 4       | login    | 2019-06-21    |
| 4       | groups   | 2019-06-21    |
| 4       | logout   | 2019-06-21    |
| 5       | login    | 2019-03-01    |
| 5       | logout   | 2019-03-01    |
| 5       | login    | 2019-06-21    |
| 5       | logout   | 2019-06-21    |
+---------+----------+---------------+

Result 表：
+------------+-------------+
| login_date | user_count  |
+------------+-------------+
| 2019-05-01 | 1           |
| 2019-06-21 | 2           |
+------------+-------------+
请注意，我们只关心用户数非零的日期.
ID 为 5 的用户第一次登陆于 2019-03-01，因此他不算在 2019-06-21 的的统计内。
```
```
--我的写法为什么不能直接-90天,?简单题
select 
login_date,count(1) as user_count
from 
(
select
 user_id,min(activity_date) as login_date
from
Traffic
where activity ='login'
group by 
user_id
) t1
where datediff("2019-06-30",login_date) <= 90
group by login_date
```
mysql中的date_sub('2019-06-30', interval 90 day)用法

##58.每位学生的最高成绩

>编写一个 SQL 查询，查询每位学生获得的最高成绩和它所对应的科目，若科目成绩并列，取 course_id 最小的一门。查询结果需按 student_id 增序进行排序。
```
表：Enrollments

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| course_id     | int     |
| grade         | int     |
+---------------+---------+
(student_id, course_id) 是该表的主键。


编写一个 SQL 查询，查询每位学生获得的最高成绩和它所对应的科目，若科目成绩并列，取 course_id 最小的一门。查询结果需按 student_id 增序进行排序。

查询结果格式如下所示：

Enrollments 表：
+------------+-------------------+
| student_id | course_id | grade |
+------------+-----------+-------+
| 2          | 2         | 95    |
| 2          | 3         | 95    |
| 1          | 1         | 90    |
| 1          | 2         | 99    |
| 3          | 1         | 80    |
| 3          | 2         | 75    |
| 3          | 3         | 82    |
+------------+-----------+-------+

Result 表：
+------------+-------------------+
| student_id | course_id | grade |
+------------+-----------+-------+
| 1          | 2         | 99    |
| 2          | 2         | 95    |
| 3          | 3         | 82    |
+------------+-----------+-------+

```
```
--我的写法,窗口简单题
select 
student_id,course_id,grade
from 
(
select 
student_id,course_id,grade,
row_number() over(partition by student_id order by grade desc ,course_id asc) as rn 
from 
Enrollments
) t1
where rn =1
order by student_id
```
```
select student_id,min(course_id) as course_id,grade
from Enrollments
where (student_id,grade) in 
    (select student_id,max(grade) from Enrollments group by student_id)
group by student_id, grade

```
##59.报告的记录
>编写一条SQL，查询每种 报告理由（report reason）在昨天的不同报告数量（post_id）。假设今天是 2019-07-05。
```
动作表：Actions

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| post_id       | int     |
| action_date   | date    | 
| action        | enum    |
| extra         | varchar |
+---------------+---------+
此表没有主键，所以可能会有重复的行。
action 字段是 ENUM 类型的，包含:('view', 'like', 'reaction', 'comment', 'report', 'share')
extra 字段是可选的信息（可能为 null），其中的信息例如有：1.报告理由(a reason for report) 2.反应类型(a type of reaction)
 

编写一条SQL，查询每种 报告理由（report reason）在昨天的不同报告数量（post_id）。假设今天是 2019-07-05。

查询及结果的格式示例：

Actions table:
+---------+---------+-------------+--------+--------+
| user_id | post_id | action_date | action | extra  |
+---------+---------+-------------+--------+--------+
| 1       | 1       | 2019-07-01  | view   | null   |
| 1       | 1       | 2019-07-01  | like   | null   |
| 1       | 1       | 2019-07-01  | share  | null   |
| 2       | 4       | 2019-07-04  | view   | null   |
| 2       | 4       | 2019-07-04  | report | spam   |
| 3       | 4       | 2019-07-04  | view   | null   |
| 3       | 4       | 2019-07-04  | report | spam   |
| 4       | 3       | 2019-07-02  | view   | null   |
| 4       | 3       | 2019-07-02  | report | spam   |
| 5       | 2       | 2019-07-04  | view   | null   |
| 5       | 2       | 2019-07-04  | report | racism |
| 5       | 5       | 2019-07-04  | view   | null   |
| 5       | 5       | 2019-07-04  | report | racism |
+---------+---------+-------------+--------+--------+

Result table:
+---------------+--------------+
| report_reason | report_count |
+---------------+--------------+
| spam          | 1            |
| racism        | 2            |
+---------------+--------------+ 
注意，我们只关心报告数量非零的结果

```
```
--我的答案
select 
extra report_reason,count(distinct if(post_id !=0,post_id,null) ) as report_count
from 
Actions
where  action_date ='2019-07-04' and action = 'report' 
group by extra
having report_reason is not null
```

**注意count(distinct 字段的用法)count和null(空值）
 count(?) ，即计算某个字段的时候，并不计算空值**
```
1	0
1	2
1	2

```
```
-- 1
select count(distinct if(ag != 0,ag,null))
from tb_aaa
group by id; 

-- 2
select count(distinct ag )
from tb_aaa
group by id;
```
##60.查询活跃业务
>写一段 SQL 来查询所有活跃的业务
如果一个业务的某个事件类型的发生次数大于此事件类型在所有业务中的平均发生次数，并且该业务至少有两个这样的事件类型，那么该业务就可被看做是活跃业务
```
事件表：Events

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| business_id   | int     |
| event_type    | varchar |
| occurences    | int     | 
+---------------+---------+
此表的主键是 (business_id, event_type)。
表中的每一行记录了某种类型的事件在某些业务中多次发生的信息。
 

写一段 SQL 来查询所有活跃的业务。

如果一个业务的某个事件类型的发生次数大于此事件类型在所有业务中的平均发生次数，并且该业务至少有两个这样的事件类型，那么该业务就可被看做是活跃业务。

查询结果格式如下所示：

Events table:
+-------------+------------+------------+
| business_id | event_type | occurences |
+-------------+------------+------------+
| 1           | reviews    | 7          |
| 3           | reviews    | 3          |
| 1           | ads        | 11         |
| 2           | ads        | 7          |
| 3           | ads        | 6          |
| 1           | page views | 3          |
| 2           | page views | 12         |
+-------------+------------+------------+

结果表
+-------------+
| business_id |
+-------------+
| 1           |
+-------------+ 
'reviews'、 'ads' 和 'page views' 的总平均发生次数分别是 (7+3)/2=5, (11+7+6)/3=8, (3+12)/2=7.5。
id 为 1 的业务有 7 个 'reviews' 事件（大于 5）和 11 个 'ads' 事件（大于 8），所以它是活跃业务。
```
```
--终于执行对了,简单题,leedcode的Oracle有毒,好像不支持if...?
select 
business_id
from 
(
select
business_id,event_type,occurences,
avg(occurences) over(partition by event_type )  as rn
from
Events
) t1
where occurences > rn
group by business_id
having count(distinct event_type) >=2
```
