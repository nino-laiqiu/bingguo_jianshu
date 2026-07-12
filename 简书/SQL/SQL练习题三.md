##31.连续空余座位
>你能利用表 cinema ，帮他们写一个查询语句，获取所有空余座位，并将它们按照 seat_id 排序后返回吗？
```
几个朋友来到电影院的售票处，准备预约连续空余座位。
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
--我的写法,可以考虑把row_number() 提取出来with 
select 
seat_id
from 
(
select
rn
from 
(
select 
seat_id,
(seat_id - row_number() over(order by seat_id) ) as rn
from 
cinema
where free =1
)
group by rn
having count(1) >1
) t
join 
(
select 
seat_id,
(seat_id - row_number() over(order by seat_id) ) as rn
from 
cinema
where free =1    
) t1
on t.rn = t1.rn
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
-- 评论区提供了lag偏移函数的使用,注意where的条件,一开始我就写错了
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
--官方的自连接和绝对值函数的使用
select distinct(c1.seat_id) 
from cinema c1 join cinema c2
on    abs(c2.seat_id-c1.seat_id)=1
where c1.free=1 and c2.free=1
order by c1.seat_id
```
##32.销售员(left join与子查询) ☆
>输出所有表 salesperson 中，没有向公司 'RED' 销售任何东西的销售员。
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
--我的写法,这题有坑不简单
select 
distinct salesperson.name as name 
from
salesperson
left join orders
on 
orders.sales_id  = salesperson.sales_id
left join company
on
orders.com_id = company.com_id
group by
salesperson.name
having
sum(if(company.name = "RED", 1, 0))  = 0
```
```
-- 使用子查询
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
##33.树节点
```
给定一个表 tree，id 是树节点的编号， p_id 是它父节点的 id 。

+----+------+
| id | p_id |
+----+------+
| 1  | null |
| 2  | 1    |
| 3  | 1    |
| 4  | 2    |
| 5  | 2    |
+----+------+
树中每个节点属于以下三种类型之一：

叶子：如果这个节点没有任何孩子节点。
根：如果这个节点是整棵树的根，即没有父节点。
内部节点：如果这个节点既不是叶子节点也不是根节点。
 

写一个查询语句，输出所有节点的编号和节点的类型，并将结果按照节点编号排序。上面样例的结果为：

 

+----+------+
| id | Type |
+----+------+
| 1  | Root |
| 2  | Inner|
| 3  | Leaf |
| 4  | Leaf |
| 5  | Leaf |
+----+------+

```

```
-- 我的写法case when 注意null是字符串还是null
select 
distinct 
t.id,
case 
when  t.p_id is null  then 'Root'
when  t1.p_id is not null then 'Inner'
when  t1.p_id is null then 'Leaf'
end as Type
from 
tree t
left join tree t1
on t.id = t1.p_id
```
```
-- 使用子查询
SELECT ID,
       CASE WHEN P_ID IS NULL THEN 'Root'
            WHEN ID IN (SELECT P_ID FROM TREE) THEN 'Inner'
            ELSE 'Leaf' END AS TYPE
FROM TREE;
```
我以前觉得case when就是分类讨论,其实不然,case when 是这样的,先判断第一个,如果不满足,判断第二个,以此类推,而不是分类讨论并列关系

##34.判断三角形
>假设表 triangle 保存了所有三条线段的长度 x、y、z ，请你帮 Tim 写一个查询语句，来判断每组 x、y、z 是否可以组成一个三角形？
```
--我的写法,简单题
select
x,y,z,
case 
when x+y > z and x+z >y and y+z >x then 'Yes'
else 'No' end  as  triangle
from 
triangle
```
##35.平面上的最近距离(where的写法)
>写一个查询语句找到两点之间的最近距离，保留 2 位小数。
```
-- 我的写法,简单题函数的应用,注意对笛卡尔积的优化
select
round(sqrt(min((pow(t1.x - t2.x, 2) + pow(t1.y - t2.y, 2)))),2) as shortest
from
point_2d t1
join point_2d t2
on t1.x <> t2.x or t1.y <> t2.y
```
```
-- 学习where的这种写法
select round(min(sqrt(power(p1.x-p2.x,2) + power(p1.y-p2.y,2))),2) shortest
from point_2d p1, point_2d p2 
where (p1.x, p1.y) <> (p2.x,p2.y)
```
##36.直线上的最近距离
>写一个查询语句，找到点中最近两个点之间的距离
```
-- 简单题我的写法,可以使用min但是速度是一样的.
select
abs(t1.x - t2.x) as shortest 
from
point t1
join 
point t2
on t1.x <> t2.x
order by shortest
limit 1
```
##37.二级关注者
```
在 facebook 中，表 follow 会有 2 个字段： followee, follower ，分别表示被关注者和关注者。

请写一个 sql 查询语句，对每一个关注者，查询关注他的关注者的数目。

比方说：

+-------------+------------+
| followee    | follower   |
+-------------+------------+
|     A       |     B      |
|     B       |     C      |
|     B       |     D      |
|     D       |     E      |
+-------------+------------+
应该输出：

+-------------+------------+
| follower    | num        |
+-------------+------------+
|     B       |  2         |
|     D       |  1         |
+-------------+------------+
```
```
-- 万万没想到要去重,注意join的条件即可,简单题
select
f1.follower as follower,count(distinct f2.follower) as num 
from
follow f1
join 
follow f2 
on f1.follower = f2.followee
group by f1.follower
      
```
##38.平均工资(☆)
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
--我的写法:窗口写法,读错题意了,读懂题意!!!,题目是求的每个月份的情况
检查了好久要去重.......而且要在外面的那个select来去重?发生了什么
值得注意的一点就是一个月内可能会发多次工资,所以partition的条件应该要是格式化的那个日期
select
distinct
pay_month,
department_id,
case
when dept_avg > com_avg then 'higher'
when dept_avg < com_avg then 'lower'
else 'same'
end as  comparison
from
(
select
//distinct
department_id,
to_char(pay_date,'YYYY-MM')  as pay_month ,
avg(amount) over(partition by pay_date) com_avg,
avg(amount) over(partition by pay_date, department_id) dept_avg
from
salary
join 
employee
on salary.employee_id = employee.employee_id
) t
order by pay_month desc ,department_id
```
```
--group by写法,这里的join条件是日期,总感觉哪里不对
select 
t1.day_month as pay_month,
department_id,
case
when avg_amount1 > avg_amount then 'higher'
when avg_amount1 < avg_amount then 'lower'
else 'same'
end as  comparison
from 
(
select
avg(amount)  as avg_amount,
date_format(pay_date, '%Y-%m') as day_month
from
salary
group by 
date_format(pay_date, '%Y-%m')
) t1
join 
(
select 
avg(amount)  as avg_amount1,
department_id,
date_format(pay_date, '%Y-%m') as day_month
from 
employee
join salary on employee.employee_id = salary.employee_id
group by date_format(pay_date, '%Y-%m') ,department_id
) t2
on t1.day_month = t2.day_month
```
学习使用Oracle中的tochar()来格式化日期还有使用replace和substr也可以格式化日期

##39.学生地理信息报告(☆)

>写一个查询语句实现对大洲（continent）列的 透视表 操作，使得每个学生按照姓名的字母顺序依次排列在对应的大洲下面。输出的标题应依次为美洲（America）、亚洲（Asia）和欧洲（Europe）。数据保证来自美洲的学生不少于来自亚洲或者欧洲的学生。
```
一所美国大学有来自亚洲、欧洲和美洲的学生，他们的地理信息存放在如下 student 表中。

 

| name   | continent |
|--------|-----------|
| Jack   | America   |
| Pascal | Europe    |
| Xi     | Asia      |
| Jane   | America   |
 

写一个查询语句实现对大洲（continent）列的 透视表 操作，使得每个学生按照姓名的字母顺序依次排列在对应的大洲下面。输出的标题应依次为美洲（America）、亚洲（Asia）和欧洲（Europe）。数据保证来自美洲的学生不少于来自亚洲或者欧洲的学生。

对于样例输入，它的对应输出是：


| America | Asia | Europe |
|---------|------|--------|
| Jack    | Xi   | Pascal |
| Jane    |      |        |
 

进阶：如果不能确定哪个大洲的学生数最多，你可以写出一个查询去生成上述学生报告吗？

```
```
-- 写过类似的题,一开始忘记使用聚合函数了,为什么要用序列函数,因为group by 在一个组内不会出现相同的大洲
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

```
-- 分别查询每个大洲在left join
select America,Asia,Europe
from (select row_number() over(order by name) id,name as America from student where continent = 'America') a
left join (select row_number() over(order by name) id,name as Asia from student where continent = 'Asia') b
on a.id=b.id
left join (select row_number() over(order by name) id,name as Europe from student where continent = 'Europe') c
on a.id=c.id

```

##40.只出现一次的最大数字

>你能写一个 SQL 查询语句，找到只出现过一次的数字中，最大的一个数字吗？
```
-- 我的写法
-- 注意null的处理
select
max(num) as num 
from
(
select
num 
from 
my_numbers
group by num 
having count(1) =1
) t
```
```
-- 注意条件查询不会返回null,使用ifnull或者嵌套select
select num from my_numbers group by num having count(*)=1 
order by num desc limit 1
```
