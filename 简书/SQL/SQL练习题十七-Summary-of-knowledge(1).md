>一些SQL关键字的使用技巧
一些SQL题的一题多解
来源于 leedcode 知乎 牛客等

##171.自连接实现窗口用法
```
Table: Activity
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
（player_id，event_date）是此表的主键。
这张表显示了某些游戏的玩家的活动情况。
每一行是一个玩家的记录，他在某一天使用某个设备注销之前登录并玩了很多游戏（可能是 0 ）。
 
编写一个 SQL 查询，同时报告每组玩家和日期，以及玩家到目前为止玩了多少游戏。也就是说，在此日期之前玩家所玩的游戏总数。详细情况请查看示例。

查询结果格式如下所示：

Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 1         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+-----------+------------+---------------------+
| player_id | event_date | games_played_so_far |
+-----------+------------+---------------------+
| 1         | 2016-03-01 | 5                   |
| 1         | 2016-05-02 | 11                  |
| 1         | 2017-06-25 | 12                  |
| 3         | 2016-03-02 | 0                   |
| 3         | 2018-07-03 | 5                   |
+-----------+------------+---------------------+
对于 ID 为 1 的玩家，2016-05-02 共玩了 5+6=11 个游戏，2017-06-25 共玩了 5+6+1=12 个游戏。
对于 ID 为 3 的玩家，2018-07-03 共玩了 0+5=5 个游戏。
请注意，对于每个玩家，我们只关心玩家的登录日期。
```
```
##自连接
使用自连接也能实现序列函数/分组topN等问题
select
t1.player_id , t1.event_date,sum(t2.games_played) games_played_so_far  
from 
Activity t1 join Activity t2
on t1.player_id = t2. player_id    and t1.event_date >= t2.event_date
group by t1.player_id , t1.event_date
```
```	
示例:排序序列函数自连接实现,来源牛客网23题
select
t1.emp_no ,t1.salary,count(distinct t2.salary) as t_rank 
from
salaries t1 cross join salaries t2 
where t1.salary <= t2.salary
group by t1.emp_no ,t1.salary
order by t_rank
```
```
示例:分组topN自连接实现,来源牛客网74题
select
t1.language_id,t1.score
from
grade t1 join grade t2 on t1.language_id = t2.language_id and t1.score <= t2.score
group by t1.language_id,t1.score
having count(distinct t2.score) <= 2
```
```
示例:sum窗口大小统计,来源leedcode579题
SELECT
  a.Id AS id, a.Month AS month,SUM(b.Salary) AS Salary
FROM
  Employee a, Employee b
WHERE a.Id = b.Id 
AND a.Month >= b.Month
AND a.Month < b.Month+3
AND (a.Id, a.Month) NOT IN (SELECT Id, MAX(Month) FROM Employee GROUP BY Id)
GROUP BY a.Id, a.Month
ORDER BY a.Id, a.Month DESC
```
```
-- 自连接获取第二个 having sum(o1.order_date > o2.order_date) = 1 leedcode1159
//使用sum()统计比当前订单日期更早的订单数，若为1，说明当前订单是对应用户卖
//出的第二单
select user_id seller_id, if(favorite_brand = item_brand, 'yes', 'no') 2nd_item_fav_brand
from users left join (
    select seller_id, item_brand
    from (
        select o1.seller_id, o1.item_id
        from orders o1 join orders o2
        on o1.seller_id = o2.seller_id
        group by o1.order_id
        having sum(o1.order_date > o2.order_date) = 1
    ) o join items i
    on o.item_id = i.item_id
) tmp
on user_id = seller_id
```
##172.正序和逆序
```
Numbers 表保存数字的值及其频率。

+----------+-------------+
|  Number  |  Frequency  |
+----------+-------------|
|  0       |  7          |
|  1       |  1          |
|  2       |  3          |
|  3       |  1          |
+----------+-------------+
在此表中，数字为 0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3，所以中位数是 (0 + 0) / 2 = 0。

+--------+
| median |
+--------|
| 0.0000 |
+--------+
请编写一个查询来查找所有数字的中位数并将结果命名为 median 
```
```
#中位数数学含义,这里的sum求的是number的位置
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
下面一题同样体现了中位数的数学意义,使用两个序列函数正序和逆序就能解决问题,在牛客网的最后一题也是这个思路
```
Employee 表包含所有员工。Employee 表有三列：员工Id，公司名和薪水。

+-----+------------+--------+
|Id   | Company    | Salary |
+-----+------------+--------+
|1    | A          | 2341   |
|2    | A          | 341    |
|3    | A          | 15     |
|4    | A          | 15314  |
|5    | A          | 451    |
|6    | A          | 513    |
|7    | B          | 15     |
|8    | B          | 13     |
|9    | B          | 1154   |
|10   | B          | 1345   |
|11   | B          | 1221   |
|12   | B          | 234    |
|13   | C          | 2345   |
|14   | C          | 2645   |
|15   | C          | 2645   |
|16   | C          | 2652   |
|17   | C          | 65     |
+-----+------------+--------+
请编写SQL查询来查找每个公司的薪水中位数。挑战点：你是否可以在不使用任何内置的SQL函数的情况下解决此问题。
+-----+------------+--------+
|Id   | Company    | Salary |
+-----+------------+--------+
|5    | A          | 451    |
|6    | A          | 513    |
|12   | B          | 234    |
|9    | B          | 1154   |
|14   | C          | 2645   |
+-----+------------+--------+
```
```
#方法一,注意窗口要指定两个字段
select a.Id,a.Company,a.Salary
from
(select Id,Company,Salary,
cast(row_number()  over(partition by Company order  by salary desc,Id desc ) as signed ) as 'id1',
cast(row_number()  over(partition by Company order  by salary asc,Id asc ) as signed) as 'id2'
from  Employee) a
where abs(a.id1-a.id2) =1 or abs(a.id1-a.id2)=0
```
```
#方法二,中位数的数学含义
select Id,Company,Salary
from
(
select Id,Company,Salary, 
row_number()over(partition by Company order by Salary)as ranking,
count(Id) over(partition by Company)as cnt
from Employee
)a
where ranking>=cnt/2 and ranking<=cnt/2+1
```

##173.窗口大小
```
Employee 表保存了一年内的薪水信息。

请你编写 SQL 语句，对于每个员工，查询他除最近一个月（即最大月）之外，剩下每个月的近三个月的累计薪水（不足三个月也要计算）。

结果请按 Id 升序，然后按 Month 降序显示。

 

示例：
输入：

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
输出：

| Id | Month | Salary |
|----|-------|--------|
| 1  | 3     | 90     |
| 1  | 2     | 50     |
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 3  | 3     | 100    |
| 3  | 2     | 40     |
 

解释：

员工 '1' 除去最近一个月（月份 '4'），有三个月的薪水记录：月份 '3' 薪水为 40，月份 '2' 薪水为 30，月份 '1' 薪水为 20。

所以近 3 个月的薪水累计分别为 (40 + 30 + 20) = 90，(30 + 20) = 50 和 20。

| Id | Month | Salary |
|----|-------|--------|
| 1  | 3     | 90     |
| 1  | 2     | 50     |
| 1  | 1     | 20     |
员工 '2' 除去最近的一个月（月份 '2'）的话，只有月份 '1' 这一个月的薪水记录。

| Id | Month | Salary |
|----|-------|--------|
| 2  | 1     | 20     |
员工 '3' 除去最近一个月（月份 '4'）后有两个月，分别为：月份 '3' 薪水为 60 和 月份 '2' 薪水为 40。所以各月的累计情况如下：

| Id | Month | Salary |
|----|-------|--------|
| 3  | 3     | 100    |
| 3  | 2     | 40     |
```
```
#sum()窗口大小,rank获取最近的一个月where来过滤,使用where联合查询来过滤也可
SELECT Id, Month, Salary
FROM (SELECT Id, Month, SUM(Salary) OVER (PARTITION BY Id ORDER BY Month ROWS 2 PRECEDING) AS Salary, rank() OVER (PARTITION BY Id ORDER BY Month DESC) AS r
      FROM Employee) t
WHERE r > 1
ORDER BY Id, Month DESC;

#lag函数的使用
SELECT ID, MONTH, SALARY + IFNULL(L1, 0) + IFNULL(L2, 0) AS SALARY
FROM (SELECT *, LAG(SALARY, 1) OVER(PARTITION BY ID ORDER BY MONTH) AS L1, LAG(SALARY, 2) OVER(PARTITION BY ID ORDER BY MONTH) AS L2, RANK() OVER(PARTITION BY ID ORDER BY MONTH DESC) AS `RANK`
      FROM EMPLOYEE) AS A
WHERE `RANK` > 1
ORDER BY 1, 2 DESC;
```
```
#自连接的使用
SELECT
  a.Id AS id, a.Month AS month,SUM(b.Salary) AS Salary
FROM
  Employee a, Employee b
WHERE a.Id = b.Id 
AND a.Month >= b.Month
AND a.Month < b.Month+3
AND (a.Id, a.Month) NOT IN (SELECT Id, MAX(Month) FROM Employee GROUP BY Id)
GROUP BY a.Id, a.Month
ORDER BY a.Id, a.Month DESC
```
##174.不同分组
```
写一个查询语句，将 2016 年 (TIV_2016) 所有成功投资的金额加起来，保留 2 位小数。

对于一个投保人，他在 2016 年成功投资的条件是：

他在 2015 年的投保额 (TIV_2015) 至少跟一个其他投保人在 2015 年的投保额相同。
他所在的城市必须与其他投保人都不同（也就是说维度和经度不能跟其他任何一个投保人完全相同）。
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
解释

就如最后一个投保人，第一个投保人同时满足两个条件：
1. 他在 2015 年的投保金额 TIV_2015 为 '10' ，与第三个和第四个投保人在 2015 年的投保金额相同。
2. 他所在城市的经纬度是独一无二的。
第二个投保人两个条件都不满足。他在 2015 年的投资 TIV_2015 与其他任何投保人都不相同。
且他所在城市的经纬度与第三个投保人相同。基于同样的原因，第三个投保人投资失败。
所以返回的结果是第一个投保人和最后一个投保人的 TIV_2016 之和，结果是 45 。
```
```
#不同分组,使用子查询,可能用到的函数concat()
select
round(sum(TIV_2016),2)  TIV_2016
from 
(
select
TIV_2016,
count(TIV_2015) over(partition by TIV_2015 ) as rn,
count(TIV_2015) over(partition by LAT,LON) as rn_1
from 
insurance 
) t1 
where rn >1 and rn_1 =1
```
```
#比较清晰的思路对需要的条件进行子查询
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

##175.偏移量函数
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
 

编写一个 SQL 查询以找出每行的人数大于或等于 100 且 id 连续的三行或更多行记录。

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
--分类讨论
select distinct a.* from stadium a,stadium b,stadium c
where a.people>=100 and b.people>=100 and c.people>=100
and (
     (a.id = b.id-1 and b.id = c.id -1) or
     (a.id = b.id-1 and a.id = c.id +1) or
     (a.id = b.id+1 and b.id = c.id +1)
) order by a.id
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
```
select id,visit_date,people from
(select id,visit_date,people,count(*)over(partition by k1) k2  录数
from 
(select id,visit_date,people,id-row_number()over(order by visit_date) k1 from stadium 
where people>=100
) a    
) b
where k2>=3
```
窗口大小那题也可以使用偏移量函数来实现
这一题也可以使用偏移量函数来实现
```
几个朋友来到电影院的售票处，准备预约连续空余座位。

你能利用表 cinema ，帮他们写一个查询语句，获取所有空余座位，并将它们按照 seat_id 排序后返回吗？

| seat_id | free |
|---------|------|
| 1       | 1    |
| 2       | 0    |
| 3       | 1    |
| 4       | 1    |
| 5       | 1    |
 

对于如上样例，你的查询语句应该返回如下结果。

 

| seat_id |
|---------|
| 3       |
| 4       |
| 5       |
注意：

seat_id 字段是一个自增的整数，free 字段是布尔类型（'1' 表示空余， '0' 表示已被占据）。
连续空余座位的定义是大于等于 2 个连续空余的座位。
```
```
select 
seat_id
from 
(
select
seat_id,free,
lag(free,1,0) over(order by seat_id) as pre,
lead(free,1,0) over(order by seat_id) as nex
from
cinema
)t
where free =1 and  (pre =1 or nex =1)
order by seat_id
```
```
with sub as (
    select seat_id,seat_id - rank() over(order by seat_id) as rk from cinema
    where free = 1
)
select seat_id
from sub
where rk in (select rk from sub group by rk having count(1) > 1)
```
```
select distinct(c1.seat_id) 
from cinema c1 join cinema c2
on    abs(c2.seat_id-c1.seat_id)=1
where c1.free=1 and c2.free=1
order by c1.seat_id
```
##176.union all的用法
```
在 Facebook 或者 Twitter 这样的社交应用中，人们经常会发好友申请也会收到其他人的好友申请。

 

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
 

解释：

编号为 '3' 的人是编号为 '1'，'2' 和 '4' 的好友，所以他总共有 3 个好友，比其他人都多。

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

```
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

##177.sum(),count() 与if结合的注意事项

```
描述

给定 3 个表： salesperson， company， orders。
输出所有表 salesperson 中，没有向公司 'RED' 销售任何东西的销售员。

示例：
输入

表： salesperson

+----------+------+--------+-----------------+-----------+
| sales_id | name | salary | commission_rate | hire_date |
+----------+------+--------+-----------------+-----------+
|   1      | John | 100000 |     6           | 4/1/2006  |
|   2      | Amy  | 120000 |     5           | 5/1/2010  |
|   3      | Mark | 65000  |     12          | 12/25/2008|
|   4      | Pam  | 25000  |     25          | 1/1/2005  |
|   5      | Alex | 50000  |     10          | 2/3/2007  |
+----------+------+--------+-----------------+-----------+
表 salesperson 存储了所有销售员的信息。每个销售员都有一个销售员编号 sales_id 和他的名字 name 。

表： company

+---------+--------+------------+
| com_id  |  name  |    city    |
+---------+--------+------------+
|   1     |  RED   |   Boston   |
|   2     | ORANGE |   New York |
|   3     | YELLOW |   Boston   |
|   4     | GREEN  |   Austin   |
+---------+--------+------------+
表 company 存储了所有公司的信息。每个公司都有一个公司编号 com_id 和它的名字 name 。

表： orders

+----------+------------+---------+----------+--------+
| order_id | order_date | com_id  | sales_id | amount |
+----------+------------+---------+----------+--------+
| 1        |   1/1/2014 |    3    |    4     | 100000 |
| 2        |   2/1/2014 |    4    |    5     | 5000   |
| 3        |   3/1/2014 |    1    |    1     | 50000  |
| 4        |   4/1/2014 |    1    |    4     | 25000  |
+----------+----------+---------+----------+--------+
表 orders 存储了所有的销售数据，包括销售员编号 sales_id 和公司编号 com_id 。

输出

+------+
| name | 
+------+
| Amy  | 
| Mark | 
| Alex |
+------+
解释

根据表 orders 中的订单 '3' 和 '4' ，容易看出只有 'John' 和 'Pam' 两个销售员曾经向公司 'RED' 销售过。

所以我们需要输出表 salesperson 中所有其他人的名字。
```

```
select 
name 
from 
salesperson
where sales_id  not in 
(
select
sales_id
from
company
join orders
on company.com_id = orders.com_id
where company.name = "RED"
)
```

```
select 
name 
from 
salesperson
where sales_id  not in 
(
select
sales_id
from
company
join orders
on company.com_id = orders.com_id
where company.name = "RED"
)
```
另一个重要的知识点:辨析left join 中条件过滤 on和where的区别,这个注意事项和!=null重要

##178.窗口不同分组
```
给如下两个表，写一个查询语句，求出在每一个工资发放日，每个部门的平均工资与公司的平均工资的比较结果 （高 / 低 / 相同）。

 

表： salary

| id | employee_id | amount | pay_date   |
|----|-------------|--------|------------|
| 1  | 1           | 9000   | 2017-03-31 |
| 2  | 2           | 6000   | 2017-03-31 |
| 3  | 3           | 10000  | 2017-03-31 |
| 4  | 1           | 7000   | 2017-02-28 |
| 5  | 2           | 6000   | 2017-02-28 |
| 6  | 3           | 8000   | 2017-02-28 |
 

employee_id 字段是表 employee 中 employee_id 字段的外键。

 

| employee_id | department_id |
|-------------|---------------|
| 1           | 1             |
| 2           | 2             |
| 3           | 2             |
 

对于如上样例数据，结果为：

 

| pay_month | department_id | comparison  |
|-----------|---------------|-------------|
| 2017-03   | 1             | higher      |
| 2017-03   | 2             | lower       |
| 2017-02   | 1             | same        |
| 2017-02   | 2             | same        |

```
```
select
    pay_month,
    department_id,
    case
        when dept_avg > com_avg then 'higher'
        when dept_avg < com_avg then 'lower'
        else 'same'
    end comparison
from (
    select
        distinct
        pay_month,
        department_id,
        avg(amount) over(partition by pay_month) com_avg,
        avg(amount) over(partition by pay_month, department_id) dept_avg
    from (
        select
            date_format(s.pay_date, '%Y-%m') pay_month,
            e.department_id,
            s.amount
        from salary s 
        left join employee e on s.employee_id = e.employee_id
    ) t
) t1
```
方法二:分别求公司和部门的join比较
```
select a.pay_month,b.department_id,
    case when b.d_avg_salarly>a.enter_avg_salary then 'higher' 
        when b.d_avg_salarly=a.enter_avg_salary then 'same'
        else 'lower' end as comparison  
from(
select date_format(pay_date,'%Y-%m') as pay_month ,avg(amount) as enter_avg_salary
from salary
group by date_format(pay_date,'%Y-%m')
) a
join 
( 
select date_format(s.pay_date,'%Y-%m') as pay_month 
        ,e.department_id
        ,avg(s.amount) as d_avg_salarly
from salary s
join employee e on s.employee_id = e.employee_id 
group by date_format(s.pay_date,'%Y-%m'),e.department_id ) b 
on a.pay_month = b.pay_month
```
##179.行转列(序列函数实现)
```
一所美国大学有来自亚洲、欧洲和美洲的学生，他们的地理信息存放在如下 student 表中。

 

| name   | continent |
|--------|-----------|
| Jack   | America   |
| Pascal | Europe    |
| Xi     | Asia      |
| Jane   | America   |
 

写一个查询语句实现对大洲（continent）列的 透视表 操作，使得每个学生按照姓名的字母顺序依次排列在对应的大洲下面。输出的标题应依次为美洲（America）、亚洲（Asia）和欧洲（Europe）。

 

对于样例输入，它的对应输出是：

 

| America | Asia | Europe |
|---------|------|--------|
| Jack    | Xi   | Pascal |
| Jane    |      |        |
 

进阶：如果不能确定哪个大洲的学生数最多，你可以写出一个查询去生成上述学生报告吗？

```
```
--分组相同州的不会出现在同一个分组中
select
     max(case when continent='America' then name else null end) as America
    ,max(case when continent='Asia' then name else null end) as Asia
    ,max(case when continent='Europe' then name else null end) as Europe
from(
    select row_number() over(partition by continent order by name) as rn,name,continent from student
) t
group by rn
order by rn 
```
注意事项:leedcode经常有人提...group by 之后必须用聚合函数，不用的话会返回第一个
```
select America,Asia,Europe 
from(
    select row_number() over(order by name) as rn,name as America from student
    where continent='America'
) a
left join(
    select row_number() over(order by name) as rn,name as Asia from student
    where continent='Asia'
) b on a.rn=b.rn
left join(
    select row_number() over(order by name) as rn,name as Europe from student
    where continent='Europe'
) c on a.rn=c.rn
```
##180.位运算
```
某城市开了一家新的电影院，吸引了很多人过来看电影。该电影院特别注意用户体验，专门有个 LED显示板做电影推荐，上面公布着影评和相关电影描述。

作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。

 

例如，下表 cinema:

+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
对于上面的例子，则正确的输出是为：

+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+

```
```
奇数&1 就是1
SELECT
	id,
	movie,
	description,
	rating
FROM
	cinema
WHERE
	id & 1
AND description <> 'boring'
ORDER BY
	rating DESC
```
