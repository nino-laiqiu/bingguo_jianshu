##21.查询员工的累计薪水(☆)
>Employee 表保存了一年内的薪水信息。
请你编写 SQL 语句，对于每个员工，查询他除最近一个月（即最大月）之外，剩下每个月的近三个月的累计薪水（不足三个月也要计算）。
结果请按 Id 升序，然后按 Month 降序显示

```
| Id | Month | Salary |
|----|-------|--------|
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 1  | 2     | 30     |
| 2  | 2     | 30     |
| 3  | 2     | 40     |
| 1  | 3     | 40     |
| 3  | 3     | 60     |
| 1  | 4     | 60     |
| 3  | 4     | 70     |

-- 输出
| Id | Month | Salary |
|----|-------|--------|
| 1  | 3     | 90     |
| 1  | 2     | 50     |
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 3  | 3     | 100    |
| 3  | 2     | 40     |

```

```
-- lag与窗口的大小
-- 理解求每个月最近三个月 rows between 2 preceding and current row
select 
Id,Month,
sum(Salary) over(partition by Id order by Month rows between 2 preceding and current row ) Salary
from 
(
select
Id,Month,Salary,
lag(Salary,1,0) over(partition by Id order by Month desc) rn 
from
Employee
) 
where rn != 0
order by Id,Month desc
```

```
-- 这也可以非常好的方法,使用两个偏移量函数,求总和,使用一个序列函数来过滤出最大月份的情况,不用窗口就是自连接也可以写
SELECT ID, MONTH, SALARY + IFNULL(L1, 0) + IFNULL(L2, 0) AS SALARY
FROM (SELECT *, LAG(SALARY, 1) OVER(PARTITION BY ID ORDER BY MONTH) AS L1, LAG(SALARY, 2) OVER(PARTITION BY ID ORDER BY MONTH) AS L2, RANK() OVER(PARTITION BY ID ORDER BY MONTH DESC) AS `RANK`
      FROM EMPLOYEE) AS A
WHERE `RANK` > 1
ORDER BY 1, 2 DESC;
```


##22.统计各个专业学生数
>一所大学有 2 个数据表，分别是 student 和 department ，这两个表保存着每个专业的学生数据和院系数据。
写一个查询语句，查询 department 表中每个专业的学生人数 （即使没有学生的专业也需列出）。
将你的查询结果按照学生人数降序排列。 如果有两个或两个以上专业有相同的学生数目，将这些部门按照部门名字的字典序从小到大排列

```
student 表格如下：

| Column Name  | Type      |
|--------------|-----------|
| student_id   | Integer   |
| student_name | String    |
| gender       | Character |
| dept_id      | Integer   |
其中， student_id 是学生的学号， student_name 是学生的姓名， gender 是学生的性别， dept_id 是学生所属专业的专业编号。

department 表格如下：

| Column Name | Type    |
|-------------|---------|
| dept_id     | Integer |
| dept_name   | String  |

```
```
-- 考察的就是count(*) count(表达式) 为null的处理
select 
dept_name,
count(student_name) as student_number
from
(
select 
dept_name,student_name
from 
department 
left join 
student
on department.dept_id = student.dept_id
) t
group by dept_name
order by student_number desc ,dept_name
```
##23.推荐人
```
给定表 customer ，里面保存了所有客户信息和他们的推荐人。

+------+------+-----------+
| id   | name | referee_id|
+------+------+-----------+
|    1 | Will |      NULL |
|    2 | Jane |      NULL |
|    3 | Alex |         2 |
|    4 | Bill |      NULL |
|    5 | Zack |         1 |
|    6 | Mark |         2 |
+------+------+-----------+
写一个查询语句，返回一个编号列表，列表中编号的推荐人的编号都 不是 2。
```
简单题,但是是我的知识的盲区了,官方的答案

>MySQL 使用三值逻辑 —— TRUE, FALSE 和 UNKNOWN。任何与 NULL 值进行的比较都会与第三种值 UNKNOWN 做比较。这个“任何值”包括 NULL 本身！这就是为什么 MySQL 提供 IS NULL 和 IS NOT NULL 两种操作来对 NULL 特殊判断。
因此，在 WHERE 语句中我们需要做一个额外的条件判断 `referee_id IS NULL'。

>避免错误的秘诀在于使用 IS NULL 或者 IS NOT NULL 两种操作来对 NULL 值做特殊判断
SELECT name FROM customer WHERE referee_id = NULL OR referee_id <> 2;
```
-- 这样写法的执行的速度大于用 != 2,为什么???
select name
from customer
where referee_id < 2 or referee_id > 2 or referee_id is null
```
##24.2016年的投资(窗口写法)
>写一个查询语句，将 2016 年 (TIV_2016) 所有成功投资的金额加起来，保留 2 位小数
对于一个投保人，他在 2016 年成功投资的条件是：
他在 2015 年的投保额 (TIV_2015) 至少跟一个其他投保人在 2015 年的投保额相同。
他所在的城市必须与其他投保人都不同（也就是说维度和经度不能跟其他任何一个投保人完全相同)
```
输入格式:
表 insurance 格式如下：

| Column Name | Type          |
|-------------|---------------|
| PID         | INTEGER(11)   |
| TIV_2015    | NUMERIC(15,2) |
| TIV_2016    | NUMERIC(15,2) |
| LAT         | NUMERIC(5,2)  |
| LON         | NUMERIC(5,2)  |
PID 字段是投保人的投保编号， TIV_2015 是该投保人在2015年的总投保金额， TIV_2016 是该投保人在2016年的投保金额， LAT 是投保人所在城市的维度， LON 是投保人所在城市的经度。

样例输入

| PID | TIV_2015 | TIV_2016 | LAT | LON |
|-----|----------|----------|-----|-----|
| 1   | 10       | 5        | 10  | 10  |
| 2   | 20       | 20       | 20  | 20  |
| 3   | 10       | 30       | 20  | 20  |
| 4   | 10       | 40       | 40  | 40  |

样例输出

| TIV_2016 |
|----------|
| 45.00    |
```
```
-- 我的写法使用concat函数
select 
round(sum(TIV_2016) , 2) as TIV_2016
from 
insurance
where TIV_2015 in 
(
select 
TIV_2015
from
insurance
group by TIV_2015
having count(TIV_2015) >1
)
and concat(lat, lon) in
(
select
concat(lat, lon)
from
insurance
group by lat , lon
having count(*) = 1
)
```

```
-- 窗口写法
SELECT ROUND(SUM(TIV_2016), 2) AS TIV_2016
FROM (SELECT *, COUNT(*) OVER(PARTITION BY TIV_2015) AS C1, COUNT(*) OVER(PARTITION BY LAT, LON) AS C2
      FROM INSURANCE) AS A
WHERE C1 > 1
AND C2 = 1;

```
##25.订单最多的客户
```
-- 简单题,考虑如果并列第一的话使用where 1
select
customer_number
from
orders
group by customer_number
order by  count(1) desc 
limit 1
```
##26.大的国家(union 和 or 哪个的效率高)

> [https://stackoverflow.com/questions/13750475/sql-performance-union-vs-o](https://stackoverflow.com/questions/13750475/sql-performance-union-vs-or%EF%BC%89)

>为了不影响性能，可以使用union all，然后手动排序去重

```
这里有张 World 表

+-----------------+------------+------------+--------------+---------------+
| name            | continent  | area       | population   | gdp           |
+-----------------+------------+------------+--------------+---------------+
| Afghanistan     | Asia       | 652230     | 25500100     | 20343000      |
| Albania         | Europe     | 28748      | 2831741      | 12960000      |
| Algeria         | Africa     | 2381741    | 37100000     | 188681000     |
| Andorra         | Europe     | 468        | 78115        | 3712000       |
| Angola          | Africa     | 1246700    | 20609294     | 100990000     |
+-----------------+------------+------------+--------------+---------------+
如果一个国家的面积超过 300 万平方公里，或者人口超过 2500 万，那么这个国家就是大国家。

编写一个 SQL 查询，输出表中所有大国家的名称、人口和面积。

例如，根据上表，我们应该输出:

+--------------+-------------+--------------+
| name         | population  | area         |
+--------------+-------------+--------------+
| Afghanistan  | 25500100    | 652230       |
| Algeria      | 37100000    | 2381741      |
+--------------+-------------+--------------+

```
```
SELECT
    name, population, area
FROM
    world
WHERE
    area > 3000000

UNION

SELECT
    name, population, area
FROM
    world
WHERE
    population > 25000000
```
##27.超过5名学生的课
```
-- 注意去重即可
select
class
from
courses
group by class
having count(DISTINCT student) >=5
```
##28.好友申请1-总体通过率
```
-- 注意ifnull的用法以及理解题意
select
round(
ifnull(
(select count(*) from (select distinct requester_id, accepter_id from RequestAccepted) as A)
/
(select count(*) from (select distinct sender_id, send_to_id from FriendRequest) as B),
0)
, 2) as accept_rate;
```
##29.图书馆的人流量

>编写一个 SQL 查询以找出每行的人数大于或等于 100 且 id 连续的三行或更多行记录。

```
表：Stadium
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| visit_date    | date    |
| people        | int     |
+---------------+---------+
visit_date 是表的主键
每日人流量信息被记录在这三列信息中：序号 (id)、日期 (visit_date)、 人流量 (people)
每天只有一行记录，日期随着 id 的增加而增加
 
返回按 visit_date 升序排列的结果表。

查询结果格式如下所示。

Stadium table:
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+

Result table:
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+
id 为 5、6、7、8 的四行 id 连续，并且每行都有 >= 100 的人数记录。
请注意，即使第 7 行和第 8 行的 visit_date 不是连续的，输出也应当包含第 8 行，因为我们只需要考虑 id 连续的记录。
不输出 id 为 2 和 3 的行，因为至少需要三条 id 连续的记录。
```
```
-- 以前写过一遍 我在纠结select 和 having的执行顺序问题,怎么和hive的不一样嫩
//可直接子查询过滤出count(1) >3 的:where rk in (
//select rk from t1 group by rk having count(1) >= 3);
select 
id,
to_char(visit_date,'yyyy-mm-dd') as visit_date,
people
from 
(
select
id,
visit_date,
people,
count(1) over(partition by rn ) as rm1
from
(
select
id,
visit_date,
people,
(id - row_number() over(order by id)) as rn  
from
Stadium
where 
people >= 100
) t
) t1
where rm1 >=3
order by id 
```
```
select distinct t2.*    
from(
    select *, 
    lead(people,1)over(order by visit_date  ) as p2,
    lead(people,2)over(order by visit_date ) as p3
    from Stadium 
)t, Stadium t2
where t.people >=100 and p2>=100 and p3>=100 and 
t2.id>=t.id and t2.id-2<=t.id
```
##30.好友申请2-谁有最多的好友(注意union all 用法)

```
表 request_accepted 存储了所有好友申请通过的数据记录，其中， requester_id 和 accepter_id 都是用户的编号。

 

| requester_id | accepter_id | accept_date|
|--------------|-------------|------------|
| 1            | 2           | 2016_06-03 |
| 1            | 3           | 2016-06-08 |
| 2            | 3           | 2016-06-08 |
| 3            | 4           | 2016-06-09 |
写一个查询语句，求出谁拥有最多的好友和他拥有的好友数目。对于上面的样例数据，结果为：

| id | num |
|----|-----|
| 3  | 3   |
注意：

保证拥有最多好友数目的只有 1 个人。
好友申请只会被接受一次，所以不会有 requester_id 和 accepter_id 值都相同的重复记录。

```
```
-- 开窗写不知道哪里错了,用spark检验一下,没检查出来重新了一遍,估计是符号错了
select
id,num
from
(
select 
id,sum(num) as num,
row_number(order by sum(num)  desc) as  rn
from 
(
select 
requester_id as id,
count(1) as num
from
request_accepted
group by requester_id
union all 
select 
accepter_id as id,
count(1) as num
from
request_accepted
group by accepter_id
) 
group by id
) t
where rn =1 
```
```
-- 正确的写法
select
id,nu as num
from
(
select
id,sum(num) as nu,
row_number() over(order by sum(num) desc) as rn
from
(
select requester_id as id, count(1) as num from request_accepted group by requester_id
union all
select  accepter_id as id1, count(1) as num1 from request_accepted group by accepter_id
) t1
group by id
) t2
where rn =1
```

```
-- 使用union all 连接
select id, count(*) num
from 
    (select requester_id id from request_accepted 
    union all
    select accepter_id id from request_accepted) tmp 
group by id
order by count(*) desc
limit 1;
```
