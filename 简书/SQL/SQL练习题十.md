##101.购买了产品A产品B却没有购买产品C的顾客
>请你设计 SQL 查询来报告购买了产品 A 和产品 B 却没有购买产品 C 的顾客的 ID 和姓名（ customer_id 和 customer_name ），我们将基于此结果为他们推荐产品 C 。
您返回的查询结果需要按照 customer_id 排序
```
Customers 表：

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| customer_id         | int     |
| customer_name       | varchar |
+---------------------+---------+
customer_id 是这张表的主键。
customer_name 是顾客的名称。
 

Orders 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| customer_id   | int     |
| product_name  | varchar |
+---------------+---------+
order_id 是这张表的主键。
customer_id 是购买了名为 "product_name" 产品顾客的id。
 

请你设计 SQL 查询来报告购买了产品 A 和产品 B 却没有购买产品 C 的顾客的 ID 和姓名（ customer_id 和 customer_name ），我们将基于此结果为他们推荐产品 C 。
您返回的查询结果需要按照 customer_id 排序。

 

查询结果如下例所示。

Customers table:
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| 1           | Daniel        |
| 2           | Diana         |
| 3           | Elizabeth     |
| 4           | Jhon          |
+-------------+---------------+

Orders table:
+------------+--------------+---------------+
| order_id   | customer_id  | product_name  |
+------------+--------------+---------------+
| 10         |     1        |     A         |
| 20         |     1        |     B         |
| 30         |     1        |     D         |
| 40         |     1        |     C         |
| 50         |     2        |     A         |
| 60         |     3        |     A         |
| 70         |     3        |     B         |
| 80         |     3        |     D         |
| 90         |     4        |     C         |
+------------+--------------+---------------+

Result table:
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| 3           | Elizabeth     |
+-------------+---------------+
只有 customer_id 为 3 的顾客购买了产品 A 和产品 B ，却没有购买产品 C 。
```
```
-- 嘻嘻,终于能实践一下having结合聚合函数来过滤了,简单题
//简化一下having having  sum(product_name='A') > 0 and sum(product_name='B') > 
//0 and sum(product_name='C') = 0
select
t1.customer_id,customer_name
from
Customers t1
join Orders t2 on t1.customer_id = t2.customer_id
group by t1.customer_id ,customer_name
having ( if(sum(product_name = "A") != 0,1,0) + if(sum(product_name = "B") != 0,1,0) + if(sum(product_name = "C") = 0,1,0) ) =3
order by t1.customer_id

```
```
SELECT customer_id,customer_name 
FROM Customers 
WHERE 
customer_id IN(SELECT customer_id FROM Orders WHERE product_name='A') 
AND 
customer_id IN(SELECT customer_id FROM Orders WHERE product_name='B') 
AND 
customer_id NOT IN(SELECT customer_id FROM Orders WHERE product_name='C')
```
##102.排名靠后的旅行者
>写一段 SQL , 报告每个用户的旅行距离.
返回的结果表单,  以 travelled_distance 降序排列, 如果有两个或者更多的用户旅行了相同的距离, 那么再以 name 升序排列.

```
表单: Users

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
id 是该表单主键.
name 是用户名字.
 

表单: Rides

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| user_id       | int     |
| distance      | int     |
+---------------+---------+
id 是该表单主键.
user_id 是本次行程的用户的 id, 而该用户此次行程距离为 distance.
 

写一段 SQL , 报告每个用户的旅行距离.

返回的结果表单,  以 travelled_distance 降序排列, 如果有两个或者更多的用户旅行了相同的距离, 那么再以 name 升序排列.

查询结果格式, 如下例所示.

 

Users 表单:
+------+-----------+
| id   | name      |
+------+-----------+
| 1    | Alice     |
| 2    | Bob       |
| 3    | Alex      |
| 4    | Donald    |
| 7    | Lee       |
| 13   | Jonathan  |
| 19   | Elvis     |
+------+-----------+

Rides 表单:
+------+----------+----------+
| id   | user_id  | distance |
+------+----------+----------+
| 1    | 1        | 120      |
| 2    | 2        | 317      |
| 3    | 3        | 222      |
| 4    | 7        | 100      |
| 5    | 13       | 312      |
| 6    | 19       | 50       |
| 7    | 7        | 120      |
| 8    | 19       | 400      |
| 9    | 7        | 230      |
+------+----------+----------+

Result 表单:
+----------+--------------------+
| name     | travelled_distance |
+----------+--------------------+
| Elvis    | 450                |
| Lee      | 450                |
| Bob      | 317                |
| Jonathan | 312                |
| Alex     | 222                |
| Alice    | 120                |
| Donald   | 0                  |
+----------+--------------------+
Elvis 和 Lee 旅行了 450 英里, Elvis 是排名靠前的旅行者, 因为他的名字在字母表上的排序比 Lee 更小.
Bob, Jonathan, Alex 和 Alice 只有一次行程, 我们只按此次行程的全部距离对他们排序.
Donald 没有任何行程, 他的旅行距离为 0.
```
```
-- 简单题
select
max(name) as name ,
ifnull(sum(distance),0) travelled_distance
from 
Users t1
left join Rides t2 on t1.id = t2.user_id
group by t1.id 
order by travelled_distance desc ,name 
```
```
为什么这个不报错?name不是聚合有的哇
select u.name, sum(ifnull(r.distance, 0)) as travelled_distance
from Users as u left join Rides as r on u.id = r.user_id
group by u.id
order by travelled_distance desc, u.name
```
##103.查询成绩处于中游的学生
>成绩处于中游的学生是指至少参加了一次测验, 且得分既不是最高分也不是最低分的学生
写一个 SQL 语句，找出在所有测验中都处于中游的学生 (student_id, student_name)
不要返回从来没有参加过测验的学生。返回结果表按照 student_id 排序

```
表: Student

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| student_id          | int     |
| student_name        | varchar |
+---------------------+---------+
student_id 是该表主键.
student_name 学生名字.
 

表: Exam

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| exam_id       | int     |
| student_id    | int     |
| score         | int     |
+---------------+---------+
(exam_id, student_id) 是该表主键.
学生 student_id 在测验 exam_id 中得分为 score.
 

成绩处于中游的学生是指至少参加了一次测验, 且得分既不是最高分也不是最低分的学生。

写一个 SQL 语句，找出在所有测验中都处于中游的学生 (student_id, student_name)。

不要返回从来没有参加过测验的学生。返回结果表按照 student_id 排序。

查询结果格式如下。

 

Student 表：
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 1           | Daniel        |
| 2           | Jade          |
| 3           | Stella        |
| 4           | Jonathan      |
| 5           | Will          |
+-------------+---------------+

Exam 表：
+------------+--------------+-----------+
| exam_id    | student_id   | score     |
+------------+--------------+-----------+
| 10         |     1        |    70     |
| 10         |     2        |    80     |
| 10         |     3        |    90     |
| 20         |     1        |    80     |
| 30         |     1        |    70     |
| 30         |     3        |    80     |
| 30         |     4        |    90     |
| 40         |     1        |    60     |
| 40         |     2        |    70     |
| 40         |     4        |    80     |
+------------+--------------+-----------+

Result 表：
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 2           | Jade          |
+-------------+---------------+

对于测验 1: 学生 1 和 3 分别获得了最低分和最高分。
对于测验 2: 学生 1 既获得了最高分, 也获得了最低分。
对于测验 3 和 4: 学生 1 和 4 分别获得了最低分和最高分。
学生 2 和 5 没有在任一场测验中获得了最高分或者最低分。
因为学生 5 从来没有参加过任何测验, 所以他被排除于结果表。
由此, 我们仅仅返回学生 2 的信息。
```
```
--执行错误??,发现问题把row_number该为rank.....检查了好久
select
distinct Student.student_id,student_name 
from 
Student join Exam on Student.student_id = Exam.student_id 
where (Student.student_id,student_name) not in (
select
distinct student_id,student_name 
from 
(
select
Student.student_id,student_name,
rank() over(partition by exam_id order by score desc ) as max_rn,
rank() over(partition by exam_id order by score asc ) as min_rn
from
Student
join Exam on Student.student_id = Exam.student_id
) t1
where max_rn = 1 or min_rn = 1 )
order by Student.student_id asc

```
```
with cte as (
select distinct student_id
from (select exam_id,
      student_id,
      rank() over (partition by exam_id order by score desc) as r1,
      rank() over (partition by exam_id order by score asc) as r2
      from exam) as temp
where r1 = 1 or r2 = 1)
select
distinct Student.student_id,student_name 
from 
Student join Exam on Student.student_id = Exam.student_id 
where Student.student_id not in ( select student_id from cte)
order by Student.student_id asc;
```
使用max和min开窗来获取每组的最值情况也可以

```
--非窗口写法
select e.student_id, s.student_name
from
    (select exam_id, max(score) s1, min(score) s2
    from exam group by exam_id) t, exam e, student s 
where t.exam_id = e.exam_id and e.student_id = s.student_id
group by e.student_id
having count(*) = sum(e.score>s2 and e.score<s1)
order by e.student_id;
```
##104.净现值查询
>写一个 SQL, 找到 Queries 表中每一次查询的净现值
```
表: NPV

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| year          | int     |
| npv           | int     |
+---------------+---------+
(id, year) 是该表主键.
该表有每一笔存货的年份, id 和对应净现值的信息.
 

表: Queries

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| year          | int     |
+---------------+---------+
(id, year) 是该表主键.
该表有每一次查询所对应存货的 id 和年份的信息.
 

写一个 SQL, 找到 Queries 表中每一次查询的净现值.

结果表没有顺序要求.

查询结果的格式如下所示:

NPV 表:
+------+--------+--------+
| id   | year   | npv    |
+------+--------+--------+
| 1    | 2018   | 100    |
| 7    | 2020   | 30     |
| 13   | 2019   | 40     |
| 1    | 2019   | 113    |
| 2    | 2008   | 121    |
| 3    | 2009   | 12     |
| 11   | 2020   | 99     |
| 7    | 2019   | 0      |
+------+--------+--------+

Queries 表:
+------+--------+
| id   | year   |
+------+--------+
| 1    | 2019   |
| 2    | 2008   |
| 3    | 2009   |
| 7    | 2018   |
| 7    | 2019   |
| 7    | 2020   |
| 13   | 2019   |
+------+--------+

结果表:
+------+--------+--------+
| id   | year   | npv    |
+------+--------+--------+
| 1    | 2019   | 113    |
| 2    | 2008   | 121    |
| 3    | 2009   | 12     |
| 7    | 2018   | 0      |
| 7    | 2019   | 0      |
| 7    | 2020   | 30     |
| 13   | 2019   | 40     |
+------+--------+--------+

(7, 2018)的净现值不在 NPV 表中, 我们把它看作是 0.
所有其它查询的净现值都能在 NPV 表中找到.
```
```
--简单题
select 
t1.id,t1.year,ifnull(npv,0) as npv 
from 
Queries t1
left join NPV t2 on t1.id =t2.id and t1.year = t2.year
```
##105.制作会话柱状图
>你想知道用户在你的 app 上的访问时长情况。因此决定统计访问时长区间分别为 "[0-5>", "[5-10>", "[10-15>" 和 "15 or more" （单位：分钟）的会话数量，并以此绘制柱状图
写一个SQL查询来报告（访问时长区间，会话总数）。结果可用任何顺序呈现
```
表：Sessions

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| session_id          | int     |
| duration            | int     |
+---------------------+---------+
session_id 是该表主键
duration 是用户访问应用的时间, 以秒为单位
 

你想知道用户在你的 app 上的访问时长情况。因此决定统计访问时长区间分别为 "[0-5>", "[5-10>", "[10-15>" 和 "15 or more" （单位：分钟）的会话数量，并以此绘制柱状图。

写一个SQL查询来报告（访问时长区间，会话总数）。结果可用任何顺序呈现。

 

下方为查询的输出格式：

Sessions 表：
+-------------+---------------+
| session_id  | duration      |
+-------------+---------------+
| 1           | 30            |
| 2           | 199           |
| 3           | 299           |
| 4           | 580           |
| 5           | 1000          |
+-------------+---------------+

Result 表：
+--------------+--------------+
| bin          | total        |
+--------------+--------------+
| [0-5>        | 3            |
| [5-10>       | 1            |
| [10-15>      | 0            |
| 15 or more   | 1            |
+--------------+--------------+

对于 session_id 1，2 和 3 ，它们的访问时间大于等于 0 分钟且小于 5 分钟。
对于 session_id 4，它的访问时间大于等于 5 分钟且小于 10 分钟。
没有会话的访问时间大于等于 10 分钟且小于 15 分钟。
对于 session_id 5, 它的访问时间大于等于 15 分钟。

```
```
--我的写法,怎么改进一下
select
"[0-5>" as bin ,sum(if(duration/60 >=0 and duration/60 <5,1,0)) as total 
from
Sessions
union all 
(
select
"[5-10>"  ,sum(if(duration/60 >=5 and duration/60 <10,1,0) )
from
Sessions
)

union all 
(
select
"[10-15>"  ,sum(if(duration/60 >=10 and duration/60 <15,1,0) )
from
Sessions
)
union all 
(
select
"15 or more"  ,sum(if(duration/60 >=15 ,1,0) )
from
Sessions
)
```
```
#解法1
select derived2.bin,ifnull(derived1.total,0) as total 
from
#返回每个区间的计数
(
	select 
		case
			when duration/60<5 then '[0-5>'
			when duration/60<10 then '[5-10>'
			when duration/60<15 then '[10-15>'
			else '15 or more'
		end bin,
		count(*) total
	from Sessions
	group by bin

) as derived1
# 构建区间表
right join 
(
	select '[0-5>' as bin 
			union
	select '[5-10>' as bin 
			union
	select '[10-15>' as bin 
			union
	select '15 or more' as bin
     
)derived2

on derived2.bin = derived1.bin



#解法2
select '[0-5>' bin, count(*) as total from Sessions where floor(duration/(60*5))=0
union
select '[5-10>' bin, count(*) as total from Sessions where floor(duration/(60*5))=1 
union
select '[10-15>' bin, count(*) as total from Sessions where floor(duration/(60*5))=2
union
select '15 or more' bin, count(*) as total from Sessions where floor(duration/(60*5))>=3
order by bin



#解法3
select '[0-5>' BIN, sum(if(duration<300,1,0)) TOTAL from Sessions 
union 
select '[5-10>' bin, sum(if(300<=duration and duration<600,1,0)) total from Sessions
union 
select '[10-15>' bin, sum(if(600<=duration and duration<900,1,0)) total from Sessions 
union 
select '15 or more' bin, sum(if(900<=duration,1,0)) total from Sessions 
```
##106.计算布尔表达式的值(等量代换)☆
>写一个 SQL 查询,  以计算表 Expressions 中的布尔表达式
```
表 Variables:

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| name          | varchar |
| value         | int     |
+---------------+---------+
name 是该表主键.
该表包含了存储的变量及其对应的值.
 

表 Expressions:

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| left_operand  | varchar |
| operator      | enum    |
| right_operand | varchar |
+---------------+---------+
(left_operand, operator, right_operand) 是该表主键.
该表包含了需要计算的布尔表达式.
operator 是枚举类型, 取值于('<', '>', '=')
left_operand 和 right_operand 的值保证存在于 Variables 表单中.
 

写一个 SQL 查询,  以计算表 Expressions 中的布尔表达式.

返回的结果表没有顺序要求.

查询结果格式如下例所示.

Variables 表:
+------+-------+
| name | value |
+------+-------+
| x    | 66    |
| y    | 77    |
+------+-------+

Expressions 表:
+--------------+----------+---------------+
| left_operand | operator | right_operand |
+--------------+----------+---------------+
| x            | >        | y             |
| x            | <        | y             |
| x            | =        | y             |
| y            | >        | x             |
| y            | <        | x             |
| x            | =        | x             |
+--------------+----------+---------------+

Result 表:
+--------------+----------+---------------+-------+
| left_operand | operator | right_operand | value |
+--------------+----------+---------------+-------+
| x            | >        | y             | false |
| x            | <        | y             | true  |
| x            | =        | y             | false |
| y            | >        | x             | true  |
| y            | <        | x             | false |
| x            | =        | x             | true  |
+--------------+----------+---------------+-------+
如上所示, 你需要通过使用 Variables 表来找到 Expressions 表中的每一个布尔表达式的值
```
```
--难题,我的解决方法就是casewhen 判断怎么优化一下?
select e.*,
    case
        when operator = '=' and v1.value = v2.value then 'true'
        when operator = '>' and v1.value > v2.value then 'true'
        when operator = '<' and v1.value < v2.value then 'true'
        else 'false'
    end value
from Expressions e
left join Variables v1
on e.left_operand = v1.name
left join Variables v2
on e.right_operand = v2.name

```
学习一招---等量代换用join来实现
##107.苹果与橘子
>写一个 SQL 查询, 报告每一天 苹果 和 桔子 销售的数目的差异.
返回的结果表, 按照格式为 ('YYYY-MM-DD') 的 sale_date 排序.
```
表: Sales

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| sale_date     | date    |
| fruit         | enum    | 
| sold_num      | int     | 
+---------------+---------+
(sale_date,fruit) 是该表主键.
该表包含了每一天中"苹果" 和 "桔子"的销售情况.
 

写一个 SQL 查询, 报告每一天 苹果 和 桔子 销售的数目的差异.

返回的结果表, 按照格式为 ('YYYY-MM-DD') 的 sale_date 排序.

查询结果表如下例所示:

 

Sales 表:
+------------+------------+-------------+
| sale_date  | fruit      | sold_num    |
+------------+------------+-------------+
| 2020-05-01 | apples     | 10          |
| 2020-05-01 | oranges    | 8           |
| 2020-05-02 | apples     | 15          |
| 2020-05-02 | oranges    | 15          |
| 2020-05-03 | apples     | 20          |
| 2020-05-03 | oranges    | 0           |
| 2020-05-04 | apples     | 15          |
| 2020-05-04 | oranges    | 16          |
+------------+------------+-------------+

Result 表:
+------------+--------------+
| sale_date  | diff         |
+------------+--------------+
| 2020-05-01 | 2            |
| 2020-05-02 | 0            |
| 2020-05-03 | 20           |
| 2020-05-04 | -1           |
+------------+--------------+

在 2020-05-01, 卖了 10 个苹果 和 8 个桔子 (差异为 10 - 8 = 2).
在 2020-05-02, 卖了 15 个苹果 和 15 个桔子 (差异为 15 - 15 = 0).
在 2020-05-03, 卖了 20 个苹果 和 0 个桔子 (差异为 20 - 0 = 20).
在 2020-05-04, 卖了 15 个苹果 和 16 个桔子 (差异为 15 - 16 = -1).
```
```
--简单题
select
sale_date,
sum(if(fruit ="apples",sold_num,0)) - sum(if(fruit ="oranges",sold_num,0)) diff
from
Sales
group by sale_date
```
```
--学习一下,忘记了...
SELECT sale_date, SUM(case when fruit='apples' then sold_num else -sold_num end) as diff
FROM Sales
GROUP BY sale_date
ORDER BY sale_date
```
##108.活跃用户

>写一个 SQL 查询,  找到活跃用户的 id 和 name.
活跃用户是指那些至少连续 5 天登录账户的用户.
```
表 Accounts:

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
id 是该表主键.
该表包含账户 id 和账户的用户名.
 

表 Logins:

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| login_date    | date    |
+---------------+---------+
该表无主键, 可能包含重复项.
该表包含登录用户的账户 id 和登录日期. 用户也许一天内登录多次.
 

写一个 SQL 查询,  找到活跃用户的 id 和 name.

活跃用户是指那些至少连续 5 天登录账户的用户.

返回的结果表按照 id 排序.

结果表格式如下例所示:

Accounts 表:
+----+----------+
| id | name     |
+----+----------+
| 1  | Winston  |
| 7  | Jonathan |
+----+----------+

Logins 表:
+----+------------+
| id | login_date |
+----+------------+
| 7  | 2020-05-30 |
| 1  | 2020-05-30 |
| 7  | 2020-05-31 |
| 7  | 2020-06-01 |
| 7  | 2020-06-02 |
| 7  | 2020-06-02 |
| 7  | 2020-06-03 |
| 1  | 2020-06-07 |
| 7  | 2020-06-10 |
+----+------------+

Result 表:
+----+----------+
| id | name     |
+----+----------+
| 7  | Jonathan |
+----+----------+
id = 1 的用户 Winston 仅仅在不同的 2 天内登录了 2 次, 所以, Winston 不是活跃用户
id = 7 的用户 Jonathon 在不同的 6 天内登录了 7 次, , 6 天中有 5 天是连续的, 所以, Jonathan 是活跃用户
```
```
--简答题
select
distinct t3.id,name 
from
(
select
id 
from 
(
select
id,
date_sub(login_date,INTERVAL dense_rank() over(partition by id order by login_date ) DAY ) x1
from
(
select
distinct id,login_date 
from 
Logins
) t1
) t2
group by id,x1
having count(1) >= 5
) t3
join Accounts on t3.id = Accounts.id 
order by t3.id 
```
```
select distinct a.id, a.name
from Accounts a
join
(select * ,lead(s.login_date,4) over(partition by s.id order by s.login_date asc) as last_date
 from (select distinct id, login_date from Logins) s
)t
on a.id = t.id
where datediff(t.login_date, t.last_date) = -4
order by a.id
```
##109.矩形面积
>写一个 SQL 语句, 报告由表中任意两点可以形成的所有可能的矩形
```
表: Points

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| x_value       | int     |
| y_value       | int     |
+---------------+---------+
id 是该表主键.
每个点都表示为二维空间 (x_value, y_value).
写一个 SQL 语句, 报告由表中任意两点可以形成的所有可能的矩形. 

结果表中的每一行包含三列 (p1, p2, area) 如下:

p1 和 p2 是矩形两个对角的 id 且 p1 < p2.
矩形的面积由列 area 表示. 
请按照面积大小降序排列，如果面积相同的话, 则按照 p1 和 p2 升序对结果表排序

Points 表:
+----------+-------------+-------------+
| id       | x_value     | y_value     |
+----------+-------------+-------------+
| 1        | 2           | 8           |
| 2        | 4           | 7           |
| 3        | 2           | 10          |
+----------+-------------+-------------+

Result 表:
+----------+-------------+-------------+
| p1       | p2          | area        |
+----------+-------------+-------------+
| 2        | 3           | 6           |
| 1        | 2           | 2           |
+----------+-------------+-------------+

p1 应该小于 p2 并且面积大于 0.
p1 = 1 且 p2 = 2 时, 面积等于 |2-4| * |8-7| = 2.
p1 = 2 且 p2 = 3 时, 面积等于 |4-2| * |7-10| = 6.
p1 = 1 且 p2 = 3 时, 是不可能为矩形的, 因为面积等于 0.
```
```
--理解错题意了写了半天,id是对角id?
select
distinct  p1,p2,area 
from 
(
select
if(abs(t1.x_value - t2.x_value)>=abs(t1.y_value - t2.y_value),abs(t1.y_value - t2.y_value),abs(t1.x_value - t2.x_value)) p1, 
if(abs(t1.x_value - t2.x_value)<=abs(t1.y_value - t2.y_value),abs(t1.y_value - t2.y_value),abs(t1.x_value - t2.x_value)) p2,
abs(t1.x_value - t2.x_value) * abs(t1.y_value - t2.y_value) as area 
from
Points t1
cross join Points t2
) s1
where area > 0
order by area desc,p1 desc,p2 desc 
```
```
--简单题,优化一下直接 where p1.id<p2.id 判断
select
distinct  p1,p2,area 
from 
(
select
if(t1.id>=t2.id,t2.id,t1.id) as p1,
if(t1.id<=t2.id,t2.id,t1.id) as p2,
abs(t1.x_value - t2.x_value) * abs(t1.y_value - t2.y_value) as area 
from
Points t1
cross join Points t2
) s1
where area > 0
order by area desc,p1 ,p2
```
##110.计算税后工资
>写一条查询 SQL 来查找每个员工的税后工资
每个公司的税率计算依照以下规则
如果这个公司员工最高工资不到 1000 ，税率为 0%
如果这个公司员工最高工资在 1000 到 10000 之间，税率为 24%
如果这个公司员工最高工资大于 10000 ，税率为 49%

```
Salaries 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| company_id    | int     |
| employee_id   | int     |
| employee_name | varchar |
| salary        | int     |
+---------------+---------+
(company_id, employee_id) 是这个表的主键
这个表包括员工的company id, id, name 和 salary 
 

写一条查询 SQL 来查找每个员工的税后工资

每个公司的税率计算依照以下规则

如果这个公司员工最高工资不到 1000 ，税率为 0%
如果这个公司员工最高工资在 1000 到 10000 之间，税率为 24%
如果这个公司员工最高工资大于 10000 ，税率为 49%
按任意顺序返回结果，税后工资结果取整

 

结果表格式如下例所示：

Salaries 表：
+------------+-------------+---------------+--------+
| company_id | employee_id | employee_name | salary |
+------------+-------------+---------------+--------+
| 1          | 1           | Tony          | 2000   |
| 1          | 2           | Pronub        | 21300  |
| 1          | 3           | Tyrrox        | 10800  |
| 2          | 1           | Pam           | 300    |
| 2          | 7           | Bassem        | 450    |
| 2          | 9           | Hermione      | 700    |
| 3          | 7           | Bocaben       | 100    |
| 3          | 2           | Ognjen        | 2200   |
| 3          | 13          | Nyancat       | 3300   |
| 3          | 15          | Morninngcat   | 7777   |
+------------+-------------+---------------+--------+

Result 表：
+------------+-------------+---------------+--------+
| company_id | employee_id | employee_name | salary |
+------------+-------------+---------------+--------+
| 1          | 1           | Tony          | 1020   |
| 1          | 2           | Pronub        | 10863  |
| 1          | 3           | Tyrrox        | 5508   |
| 2          | 1           | Pam           | 300    |
| 2          | 7           | Bassem        | 450    |
| 2          | 9           | Hermione      | 700    |
| 3          | 7           | Bocaben       | 76     |
| 3          | 2           | Ognjen        | 1672   |
| 3          | 13          | Nyancat       | 2508   |
| 3          | 15          | Morninngcat   | 5911   |
+------------+-------------+---------------+--------+
对于公司 1 ，最高工资是 21300 ，其每个员工的税率为 49%
对于公司 2 ，最高工资是 700 ，其每个员工税率为 0%
对于公司 3 ，最高工资是 7777 ，其每个员工税率是 24%
税后工资计算 = 工资 - ( 税率 / 100）*工资
对于上述案例，Morninngcat 的税后工资 = 7777 - 7777 * ( 24 / 100) = 7777 - 1866.48 = 5910.52 ，取整为 5911
```
```
--简单题,可以一步完成在case when 中使用窗口函数
select 
company_id,
employee_id,
employee_name,
round(
case 
when max_salary <1000 then salary 
when max_salary > 10000 then salary - salary * ( 49 / 100) 
else salary - salary * ( 24 / 100)  end )  as salary
from 
(
select
company_id,
employee_id,
employee_name,
salary,
max(salary) over(partition by company_id) as max_salary
from
Salaries
) temp 
```
```
with tax as
(select company_id, case when max(salary) < 1000 then 0 
                        when max(salary) between 1000 and 10000 then 0.24
                        else 0.49 end as tax_rate
from salaries
group by company_id)

select s.company_id, s.employee_id, s.employee_name, round(s.salary * (1-tax_rate),0) as salary
from salaries s join tax t on s.company_id = t.company_id
```

