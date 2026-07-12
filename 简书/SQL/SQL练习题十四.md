##141.苹果和橘子的个数
>编写 SQL 语句，查询每个大箱子中苹果和橘子的个数。如果大箱子中包含小盒子，还应当包含小盒子中苹果和橘子的个数
```
表： Boxes

+--------------+------+
| Column Name  | Type |
+--------------+------+
| box_id       | int  |
| chest_id     | int  |
| apple_count  | int  |
| orange_count | int  |
+--------------+------+
box_id 是该表的主键。
chest_id 是 chests 表的外键。
该表包含大箱子 (box) 中包含的苹果和橘子的个数。每个大箱子中可能包含一个小盒子 (chest) ，小盒子中也包含若干苹果和橘子。
 

表： Chests

+--------------+------+
| Column Name  | Type |
+--------------+------+
| chest_id     | int  |
| apple_count  | int  |
| orange_count | int  |
+--------------+------+
chest_id 是该表的主键。
该表包含小盒子的信息，以及小盒子中包含的苹果和橘子的个数。
 

编写 SQL 语句，查询每个大箱子中苹果和橘子的个数。如果大箱子中包含小盒子，还应当包含小盒子中苹果和橘子的个数。

以任意顺序返回结果表。

查询结果的格式如下示例所示：

 

Boxes 表：
+--------+----------+-------------+--------------+
| box_id | chest_id | apple_count | orange_count |
+--------+----------+-------------+--------------+
| 2      | null     | 6           | 15           |
| 18     | 14       | 4           | 15           |
| 19     | 3        | 8           | 4            |
| 12     | 2        | 19          | 20           |
| 20     | 6        | 12          | 9            |
| 8      | 6        | 9           | 9            |
| 3      | 14       | 16          | 7            |
+--------+----------+-------------+--------------+

Chests 表：
+----------+-------------+--------------+
| chest_id | apple_count | orange_count |
+----------+-------------+--------------+
| 6        | 5           | 6            |
| 14       | 20          | 10           |
| 2        | 8           | 8            |
| 3        | 19          | 4            |
| 16       | 19          | 19           |
+----------+-------------+--------------+

结果表：
+-------------+--------------+
| apple_count | orange_count |
+-------------+--------------+
| 151         | 123          |
+-------------+--------------+
大箱子 2 中有 6 个苹果和 15 个橘子。
大箱子 18 中有 4 + 20 (在小盒子中) = 24 个苹果和 15 + 10 (在小盒子中) = 25 个橘子。
大箱子 19 中有 8 + 19 (在小盒子中) = 27 个苹果和 4 + 4 (在小盒子中) = 8 个橘子。
大箱子 12 中有 19 + 8 (在小盒子中) = 27 个苹果和 20 + 8 (在小盒子中) = 28 个橘子。
大箱子 20 中有 12 + 5 (在小盒子中) = 17 个苹果和 9 + 6 (在小盒子中) = 15 个橘子。
大箱子 8 中有 9 + 5 (在小盒子中) = 14 个苹果和 9 + 6 (在小盒子中) = 15 个橘子。
大箱子 3 中有 16 + 20 (在小盒子中) = 36 个苹果和 7 + 10 (在小盒子中) = 17 个橘子。
苹果的总个数 = 6 + 24 + 27 + 27 + 17 + 14 + 36 = 151
橘子的总个数 = 15 + 25 + 8 + 28 + 15 + 15 + 17 = 123
```
```
简单题,注意null+ ? = null
select
sum(ifnull(t1.apple_count,0) + ifnull(t2.apple_count,0)) apple_count  , 
sum(ifnull(t1.orange_count,0) + ifnull(t2.orange_count,0)) orange_count
from
Boxes t1
left join Chests t2 on t1.chest_id = t2.chest_id
```
##142.求关注者的数量
>写出 SQL 语句，对于每一个用户，返回该用户的关注者数量
```
表： Followers

+-------------+------+
| Column Name | Type |
+-------------+------+
| user_id     | int  |
| follower_id | int  |
+-------------+------+
(user_id, follower_id) 是这个表的主键。
该表包含一个关注关系中关注者和用户的编号，其中关注者关注用户。
写出 SQL 语句，对于每一个用户，返回该用户的关注者数量。

按 user_id 的顺序返回结果表。

查询结果的格式如下示例所示：

 

Followers 表：
+---------+-------------+
| user_id | follower_id |
+---------+-------------+
| 0       | 1           |
| 1       | 0           |
| 2       | 0           |
| 2       | 1           |
+---------+-------------+
结果表：
+---------+----------------+
| user_id | followers_count|
+---------+----------------+
| 0       | 1              |
| 1       | 1              |
| 2       | 2              |
+---------+----------------+
0 的关注者有 {1}
1 的关注者有 {0}
2 的关注者有 {0,1}
```
```
--简单题,直接count(*)即可
select
user_id,count(distinct follower_id ) followers_count
from 
Followers
group by user_id
order by user_id
```
##143.向每个员工报告的员工人数
>编写一个SQL查询来报告id和所有经理的姓名、直接向他们报告的员工人数，以及四舍五入到最接近的整数的报告的平均年龄。
```
Table: Employees

+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| employee_id | int      |
| name        | varchar  |
| reports_to  | int      |
| age         | int      |
+-------------+----------+
employee_id is the primary key for this table.
This table contains information about the employees and the id of the manager they report to. Some employees do not report to anyone (reports_to is null). 
 

For this problem, we will consider a manager an employee who has at least 1 other employee reporting to them.

Write an SQL query to report the ids and the names of all managers, the number of employees who report directly to them, and the average age of the reports rounded to the nearest integer.

Return the result table ordered by employee_id.

The query result format is in the following example:

 

Employees table:
+-------------+---------+------------+-----+
| employee_id | name    | reports_to | age |
+-------------+---------+------------+-----+
| 9           | Hercy   | null       | 43  |
| 6           | Alice   | 9          | 41  |
| 4           | Bob     | 9          | 36  |
| 2           | Winston | null       | 37  |
+-------------+---------+------------+-----+

Result table:
+-------------+-------+---------------+-------------+
| employee_id | name  | reports_count | average_age |
+-------------+-------+---------------+-------------+
| 9           | Hercy | 2             | 39          |
+-------------+-------+---------------+-------------+
Hercy has 2 people report directly to him, Alice and Bob. Their average age is (41+36)/2 = 38.5, which is 39 after rounding it to the nearest integer.
```
```
--简单题
select
t1.employee_id,t1.name,
count(2) as  reports_count,
round(avg(t2.age)) average_age 
from 
Employees t1 
join Employees t2 on t1. employee_id = t2.reports_to
group by t1.employee_id,t1.name
order by  employee_id
```
##144.找出每个员工花费的总时间
>编写一个SQL查询，以分钟为单位计算每个员工每天在办公室花费的总时间
```
Table: Employees

+-------------+------+
| Column Name | Type |
+-------------+------+
| emp_id      | int  |
| event_day   | date |
| in_time     | int  |
| out_time    | int  |
+-------------+------+
(emp_id, event_day, in_time) is the primary key of this table.
The table shows the employees' entries and exits in an office.
event_day is the day at which this event happened, in_time is the minute at which the employee entered the office, and out_time is the minute at which they left the office.
in_time and out_time are between 1 and 1440.
It is guaranteed that no two events on the same day intersect in time, and in_time < out_time.
 

Write an SQL query to calculate the total time in minutes spent by each employee on each day at the office. Note that within one day, an employee can enter and leave more than once. The time spent in the office for a single entry is out_time - in_time.

Return the result table in any order.

The query result format is in the following example:

Employees table:
+--------+------------+---------+----------+
| emp_id | event_day  | in_time | out_time |
+--------+------------+---------+----------+
| 1      | 2020-11-28 | 4       | 32       |
| 1      | 2020-11-28 | 55      | 200      |
| 1      | 2020-12-03 | 1       | 42       |
| 2      | 2020-11-28 | 3       | 33       |
| 2      | 2020-12-09 | 47      | 74       |
+--------+------------+---------+----------+
Result table:
+------------+--------+------------+
| day        | emp_id | total_time |
+------------+--------+------------+
| 2020-11-28 | 1      | 173        |
| 2020-11-28 | 2      | 30         |
| 2020-12-03 | 1      | 41         |
| 2020-12-09 | 2      | 27         |
+------------+--------+------------+
Employee 1 has three events: two on day 2020-11-28 with a total of (32 - 4) + (200 - 55) = 173, and one on day 2020-12-03 with a total of (42 - 1) = 41.
Employee 2 has two events: one on day 2020-11-28 with a total of (33 - 3) = 30, and one on day 2020-12-09 with a total of (74 - 47) = 27.
```
```
--简单题
select
event_day day ,
emp_id ,
sum(out_time - in_time) total_time
from 
Employees
group by 1,2
```
##145.Leetflex禁止帐户
>编写一个SQL查询来查找应该从Leetflex中禁止的帐户的account_id。如果一个帐户在某一时刻从两个不同的IP地址登录，应该被禁止。
```
Table: LogInfo

+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| account_id  | int      |
| ip_address  | int      |
| login       | datetime |
| logout      | datetime |
+-------------+----------+
There is no primary key for this table, and it may contain duplicates.
The table contains information about the login and logout dates of Leetflex accounts. It also contains the IP address from which the account logged in and out.
It is guaranteed that the logout time is after the login time.
 

Write an SQL query to find the account_id of the accounts that should be banned from Leetflex. An account should be banned if it was logged in at some moment from two different IP addresses.

Return the result table in any order.

The query result format is in the following example:

 

LogInfo table:
+------------+------------+---------------------+---------------------+
| account_id | ip_address | login               | logout              |
+------------+------------+---------------------+---------------------+
| 1          | 1          | 2021-02-01 09:00:00 | 2021-02-01 09:30:00 |
| 1          | 2          | 2021-02-01 08:00:00 | 2021-02-01 11:30:00 |
| 2          | 6          | 2021-02-01 20:30:00 | 2021-02-01 22:00:00 |
| 2          | 7          | 2021-02-02 20:30:00 | 2021-02-02 22:00:00 |
| 3          | 9          | 2021-02-01 16:00:00 | 2021-02-01 16:59:59 |
| 3          | 13         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |
| 4          | 10         | 2021-02-01 16:00:00 | 2021-02-01 17:00:00 |
| 4          | 11         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |
+------------+------------+---------------------+---------------------+

Result table:
+------------+
| account_id |
+------------+
| 1          |
| 4          |
+------------+
Account ID 1 --> The account was active from "2021-02-01 09:00:00" to "2021-02-01 09:30:00" with two different IP addresses (1 and 2). It should be banned.
Account ID 2 --> The account was active from two different addresses (6, 7) but in two different times.
Account ID 3 --> The account was active from two different addresses (9, 13) on the same day but they do not intersect at any moment.
Account ID 4 --> The account was active from "2021-02-01 17:00:00" to "2021-02-01 17:00:00" with two different IP addresses (10 and 11). It should be banned.
```
```
--简单题
select
distinct t1.account_id 
from
LogInfo t1
join LogInfo t2 on t1.account_id = t2.account_id and t1.login < t2.login and t1.ip_address <> t2.ip_address
where (t2.logout - t1.login) <  (t1.logout + t2.logout - t1.login - t2.login) or t1.logout = t2.login
```
```
--为什么这个快??
select
distinct l1.account_id
from
loginfo l1 join loginfo l2
on l1.account_id = l2.account_id and l1.ip_address <> l2.ip_address
and l1.login between l2.login and l2.logout
```
##146.可回收低脂产品
>编写一个SQL查询来查找低脂肪和可回收产品的id。
```
Table: Products

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_id  | int     |
| low_fats    | enum    |
| recyclable  | enum    |
+-------------+---------+
product_id is the primary key for this table.
low_fats is an ENUM of type ('Y', 'N') where 'Y' means this product is low fat and 'N' means it is not.
recyclable is an ENUM of types ('Y', 'N') where 'Y' means this product is recyclable and 'N' means it is not.
 

Write an SQL query to find the ids of products that are both low fat and recyclable.

Return the result table in any order.

The query result format is in the following example:

 

Products table:
+-------------+----------+------------+
| product_id  | low_fats | recyclable |
+-------------+----------+------------+
| 0           | Y        | N          |
| 1           | Y        | Y          |
| 2           | N        | Y          |
| 3           | Y        | Y          |
| 4           | N        | N          |
+-------------+----------+------------+
Result table:
+-------------+
| product_id  |
+-------------+
| 1           |
| 3           |
+-------------+
Only products 1 and 3 are both low fat and recyclable.
```

```
select product_id from Products where low_fats ="Y" and recyclable ="Y"
```
##147.行程和用户
>写一段 SQL 语句查出 "2013-10-01" 至 "2013-10-03" 期间非禁止用户（乘客和司机都必须未被禁止）的取消率。非禁止用户即 Banned 为 No 的用户，禁止用户即 Banned 为 Yes 的用户。
取消率 的计算方式如下：(被司机或乘客取消的非禁止用户生成的订单数量) / (非禁止用户生成的订单总数)。
返回结果表中的数据可以按任意顺序组织。其中取消率 Cancellation Rate 需要四舍五入保留 两位小数 
```
表：Trips
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| Id          | int      |
| Client_Id   | int      |
| Driver_Id   | int      |
| City_Id     | int      |
| Status      | enum     |
| Request_at  | date     |     
+-------------+----------+
Id 是这张表的主键。
这张表中存所有出租车的行程信息。每段行程有唯一 Id ，其中 Client_Id 和 Driver_Id 是 Users 表中 Users_Id 的外键。
Status 是一个表示行程状态的枚举类型，枚举成员为(‘completed’, ‘cancelled_by_driver’, ‘cancelled_by_client’) 。
 

表：Users

+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| Users_Id    | int      |
| Banned      | enum     |
| Role        | enum     |
+-------------+----------+
Users_Id 是这张表的主键。
这张表中存所有用户，每个用户都有一个唯一的 Users_Id ，Role 是一个表示用户身份的枚举类型，枚举成员为 (‘client’, ‘driver’, ‘partner’) 。
Banned 是一个表示用户是否被禁止的枚举类型，枚举成员为 (‘Yes’, ‘No’) 。
 

写一段 SQL 语句查出 "2013-10-01" 至 "2013-10-03" 期间非禁止用户（乘客和司机都必须未被禁止）的取消率。非禁止用户即 Banned 为 No 的用户，禁止用户即 Banned 为 Yes 的用户。

取消率 的计算方式如下：(被司机或乘客取消的非禁止用户生成的订单数量) / (非禁止用户生成的订单总数)。

返回结果表中的数据可以按任意顺序组织。其中取消率 Cancellation Rate 需要四舍五入保留 两位小数 。

 

查询结果格式如下例所示：

Trips 表：
+----+-----------+-----------+---------+---------------------+------------+
| Id | Client_Id | Driver_Id | City_Id | Status              | Request_at |
+----+-----------+-----------+---------+---------------------+------------+
| 1  | 1         | 10        | 1       | completed           | 2013-10-01 |
| 2  | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 |
| 3  | 3         | 12        | 6       | completed           | 2013-10-01 |
| 4  | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 |
| 5  | 1         | 10        | 1       | completed           | 2013-10-02 |
| 6  | 2         | 11        | 6       | completed           | 2013-10-02 |
| 7  | 3         | 12        | 6       | completed           | 2013-10-02 |
| 8  | 2         | 12        | 12      | completed           | 2013-10-03 |
| 9  | 3         | 10        | 12      | completed           | 2013-10-03 |
| 10 | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 |
+----+-----------+-----------+---------+---------------------+------------+

Users 表：
+----------+--------+--------+
| Users_Id | Banned | Role   |
+----------+--------+--------+
| 1        | No     | client |
| 2        | Yes    | client |
| 3        | No     | client |
| 4        | No     | client |
| 10       | No     | driver |
| 11       | No     | driver |
| 12       | No     | driver |
| 13       | No     | driver |
+----------+--------+--------+

Result 表：
+------------+-------------------+
| Day        | Cancellation Rate |
+------------+-------------------+
| 2013-10-01 | 0.33              |
| 2013-10-02 | 0.00              |
| 2013-10-03 | 0.50              |
+------------+-------------------+

2013-10-01：
  - 共有 4 条请求，其中 2 条取消。
  - 然而，Id=2 的请求是由禁止用户（User_Id=2）发出的，所以计算时应当忽略它。
  - 因此，总共有 3 条非禁止请求参与计算，其中 1 条取消。
  - 取消率为 (1 / 3) = 0.33
2013-10-02：
  - 共有 3 条请求，其中 0 条取消。
  - 然而，Id=6 的请求是由禁止用户发出的，所以计算时应当忽略它。
  - 因此，总共有 2 条非禁止请求参与计算，其中 0 条取消。
  - 取消率为 (0 / 2) = 0.00
2013-10-03：
  - 共有 3 条请求，其中 1 条取消
  - 然而，Id=8 的请求是由禁止用户发出的，所以计算时应当忽略它
  - 因此，总共有 2 条非禁止请求参与计算，其中 1 条取消。
  - 取消率为 (1 / 2) = 0.50
```
```
--简单题
select request_at as Day,
round(sum(if(status != 'completed',1,0))/count(1),2) as 'Cancellation Rate' 
from Trips
where 
request_at between "2013-10-01" and "2013-10-03"
and
client_id not in (select users_id from users where banned = "Yes")
group by request_at;
```

##148.第N高的薪水
>编写一个 SQL 查询，获取 Employee 表中第 n 高的薪水（Salary）。
```
编写一个 SQL 查询，获取 Employee 表中第 n 高的薪水（Salary）。

+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
例如上述 Employee 表，n = 2 时，应返回第二高的薪水 200。如果不存在第 n 高的薪水，那么查询应返回 null。

+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
+------------------------+
```
```
--简单题
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      select distinct Salary
      from(
      select Salary ,dense_rank()over(order by Salary desc ) as r 
      from Employee) t
      where r=N
  );
END
```

补充两道SQL题,凑整数
##149.如果为null,则取上一个不为null的值
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a7e7a9def8fb3a56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>方案一
首先把不为null的数据取出来放在数组中,接着对增加一个字段,如果为null,那么就为0否则为1 ,接着原表开窗(sum()) 取数组中arr[rn-1] 的数据
```
price_date,price 表:price

with x as (
select 
collect_list(price) as arr 
from 
price
where price is not null 
),
y as (
select 
price_date,
price,
sum(label) over(order by price_date) as rn 
from 
(
select
price_date,
price,
if(price is null,0,1) as  label
from
price 
) o 
) 
select 
price_date, arr[rn-1] as price
from 
y 
```
```
+----------+-----+
|price_date|price|
+----------+-----+
|2019-09-25|23.23|
|2019-09-26|32.54|
|2019-09-27|34.55|
|2019-09-28|34.55|
|2019-09-29|34.55|
|2019-09-30|54.22|
|2019-10-01|54.22|
|2019-10-02|54.22|
|2019-10-03|54.22|
|2019-10-04|54.22|
|2019-10-05|54.22|
|2019-10-06|54.22|
|2019-10-07|54.22|
|2019-10-08|33.80|
|2019-10-09|33.83|
```

方案二
>首先排除为null的,把时间lead上移.如果原表的时间在这个区间,那么就取这个区间的时间
发现join重复数据,采用了过滤为null的值,能否采用其他方法
```
with x as (
select
price_date,price,
lead(price_date,1,price_date) over(order by price_date) as price_date1
from
(
select  -- 查找出price不为null的行
price_date,price
from
pricetable
where price != 'null'
) o
)
select
price_date,price
from
(
select
pricetable.price_date,
x.price
from
pricetable,x
where pricetable.price = 'null' and  (pricetable.price_date between x.price_date and x.price_date1)

union all

select
price_date,price
from
pricetable
where price != 'null'
) t1
order by  price_date
```

##150.指标分析
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b536bbe3ad17bdb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
create table  TEST(
     DEP_CODE varchar(10),
     ITEM_CODE varchar(10),
     ITEM_VALUE int
);
insert into TEST values ('单位一','A',100);
insert into TEST values ('单位二','A',200);
insert into TEST values ('单位一','B',300);
insert into TEST values ('单位二','B',300);
insert into TEST values ('单位一','C',350);
insert into TEST values ('单位二','C',300);

select
    DEP_CODE,
    ITEM_CODE,
    ITEM_VALUE
from
    TEST
UNION ALL

select
    DEP_CODE,
    'D'  as ITEM_CODE,
    sum(if(ITEM_CODE = 'C',-ITEM_VALUE,ITEM_VALUE)) as  ITEM_VALUE
from
    TEST
group by  DEP_CODE
```
