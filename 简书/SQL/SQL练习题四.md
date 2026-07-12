##41.有趣的电影(位运算)

>作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。
```
某城市开了一家新的电影院，吸引了很多人过来看电影。该电影院特别注意用户体验，专门有个 LED显示板做电影推荐，上面公布着影评和相关电影描述。

作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列
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
-- 我的写法,简单题
select 
id,movie,description,rating
from
cinema
where description != 'boring' and (id%2 =1)
order by rating desc
```
```
-- 注意这种写法与位运算
select id, movie, description, rating
from cinema
where description <> 'boring' and id & 1
order by rating desc;
```
##42.换座位
>小美想改变相邻俩学生的座位。
你能不能帮她写一个 SQL query 来输出小美想要的结果呢？
```
小美是一所中学的信息科技老师，她有一张 seat 座位表，平时用来储存学生名字和与他们相对应的座位 id。

其中纵列的 id 是连续递增的

小美想改变相邻俩学生的座位。

你能不能帮她写一个 SQL query 来输出小美想要的结果呢？

示例：

+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Abbot   |
|    2    | Doris   |
|    3    | Emerson |
|    4    | Green   |
|    5    | Jeames  |
+---------+---------+
假如数据输入的是上表，则输出结果如下：

+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Doris   |
|    2    | Abbot   |
|    3    | Green   |
|    4    | Emerson |
|    5    | Jeames  |
+---------+---------+
注意：

如果学生人数是奇数，则不需要改变最后一个同学的座位。
```
```
--我的写法两个偏移量函数的使用
select 
id,
case when mod(id,2) = 1 then lead1  else lag1 end as student
from 
(
select
id,
lag(student,1,student) over(order by id) as lag1,
lead(student,1,student) over(order by id) as lead1
from
seat
) 
```
评论区的一些写法
```
-- 对id进行判断,根据条件如果为奇数且是最后一个则不改变
SELECT (CASE 
            WHEN MOD(id,2) = 1 AND id = (SELECT COUNT(*) FROM seat) THEN id
            WHEN MOD(id,2) = 1 THEN id+1
            ElSE id-1
        END) AS id, student
FROM seat
ORDER BY id;
```
```     
-- 妙,思路是对id-1把偶数转变为奇数,使用位运算,偶数+1 奇数-1
select rank() over(order by (id-1)^1) as id,student from seat
```
##43.更变性别
>给定一个 salary 表，如下所示，有 m = 男性 和 f = 女性 的值。交换所有的 f 和 m 值（例如，将所有 f 值更改为 m，反之亦然）。要求只使用一个更新（Update）语句，并且没有中间的临时表。
注意，您必只能写一个 Update 语句，请不要编写任何 Select 语句。

```
-- 简答题,熟悉一下语法
update salary set sex=if(sex='f','m','f')
```
```
UPDATE salary 

SET 
   sex = CASE sex 
        WHEN "m" THEN "f" 
        ELSE "m" 
    END;
```
```
ALTER TABLE salary 
ADD COLUMN sexadd TEXT
UPDATE salary 
SET sexadd = sex
SET sex = "f"
WHERE sex = "m"
SET sex = "m"
WHERE sexadd = "f";
```
##44.买下所有产品的客户
```
Customer 表：

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| customer_id | int     |
| product_key | int     |
+-------------+---------+
product_key 是 Customer 表的外键。
Product 表：

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_key | int     |
+-------------+---------+
product_key 是这张表的主键。
 

写一条 SQL 查询语句，从 Customer 表中查询购买了 Product 表中所有产品的客户的 id。

示例：

Customer 表：
+-------------+-------------+
| customer_id | product_key |
+-------------+-------------+
| 1           | 5           |
| 2           | 6           |
| 3           | 5           |
| 3           | 6           |
| 1           | 6           |
+-------------+-------------+

Product 表：
+-------------+
| product_key |
+-------------+
| 5           |
| 6           |
+-------------+

Result 表：
+-------------+
| customer_id |
+-------------+
| 1           |
| 3           |
+-------------+
购买了所有产品（5 和 6）的客户的 id 是 1 和 3 
```
```
-- 我的写法注意去重,可以对去重优化一下
select
customer_id
from
(
select
distinct customer_id,product_key
from
Customer
) t
group by customer_id
having count(1) = (select count(1) from Product )
```
```
select
customer_id
from
Customer
group by customer_id
having count(distinct product_key ) = (select count(1) from  Product )
```
##45.合作过至少三次的演员和导演

```
-- 简单题,注意如果使用concat两个字段在这一题中要加","
select 
actor_id,director_id
from
ActorDirector
group by actor_id , director_id
having count(1) >=3
```
##46.产品销售分析1
```
-- 简单题
select 
product_name,year,price
from 
Product t1
join 
Sales t2
on t1. product_id = t2.product_id
```
##47.产品销售分析2
```
--简单题,不用join...
select 
t1.product_id,sum(quantity) as  total_quantity 
from
Product t1
join Sales t2
on t1.product_id = t2.product_id
group by t1.product_id
```
##48.产品销售分析3(where联合字段)
```
-- 简答题
select 
product_id,year as first_year  ,quantity,price 
from 
(
select
product_id,year,quantity,price ,
rank() over(partition by product_id order by year) as rn 
from
Sales 
)  t 
where rn =1
```
```
-- where联合字段与子查询
SELECT PRODUCT_ID, YEAR AS FIRST_YEAR, QUANTITY, PRICE
FROM SALES
WHERE (PRODUCT_ID, YEAR) IN (SELECT PRODUCT_ID, MIN(YEAR)
                             FROM SALES
                             GROUP BY 1);
```
##49.项目员工1
```
项目表 Project： 

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| project_id  | int     |
| employee_id | int     |
+-------------+---------+
主键为 (project_id, employee_id)。
employee_id 是员工表 Employee 表的外键。
员工表 Employee：

+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| employee_id      | int     |
| name             | varchar |
| experience_years | int     |
+------------------+---------+
主键是 employee_id。
 

请写一个 SQL 语句，查询每一个项目中员工的 平均 工作年限，精确到小数点后两位。

查询结果的格式如下：

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
| 3           | John   | 1                |
| 4           | Doe    | 2                |
+-------------+--------+------------------+

Result 表：
+-------------+---------------+
| project_id  | average_years |
+-------------+---------------+
| 1           | 2.00          |
| 2           | 2.50          |
+-------------+---------------+
第一个项目中，员工的平均工作年限是 (3 + 2 + 1) / 3 = 2.00；第二个项目中，员工的平均工作年限是 (3 + 2) / 2 = 2.50

```
```
-- 简单题,注意SQL中的格式化问题
select
project_id,
round(avg(experience_years),2) as average_years
from 
Project t1
join Employee t2
on t1.employee_id =t2.employee_id
group by project_id
```
##50.项目员工2
```
-- 我的写法,简单题,注意可能会有并列第一
select 
project_id
from 
(
select
project_id,dense_rank() over(order by count(1) desc) as rn 
from 
Project
group by project_id
)
where rn =1
```
```
-- 使用子查询,学习使用all的用法
SELECT PROJECT_ID
FROM PROJECT
GROUP BY 1
HAVING COUNT(*) = (SELECT COUNT(*)
                   FROM PROJECT
                   GROUP BY PROJECT_ID
                   ORDER BY 1 DESC
                   LIMIT 1);

METHOD 2
SELECT PROJECT_ID
FROM PROJECT
GROUP BY 1
HAVING COUNT(*) >= ALL (SELECT COUNT(*)
                        FROM PROJECT
                        GROUP BY PROJECT_ID);

```
