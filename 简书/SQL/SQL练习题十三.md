##131.hopper公司查询1
>编写SQL查询以报告2020年每个月的以下统计信息：
截至某月底，当前在Hopper公司工作的驾驶员数量（active_drivers）。
该月接受的乘车次数（accepted_rides）。
返回按month 升序排列的结果表，其中month 是月份的数字（一月是1，二月是2，依此类推）。
```
表: Drivers

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| driver_id   | int     |
| join_date   | date    |
+-------------+---------+
driver_id是该表的主键。
该表的每一行均包含驾驶员的ID以及他们加入Hopper公司的日期。
 

表: Rides

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| ride_id      | int     |
| user_id      | int     |
| requested_at | date    |
+--------------+---------+
ride_id是该表的主键。
该表的每一行均包含行程ID(ride_id)，用户ID(user_id)以及该行程的日期(requested_at)。
该表中可能有一些不被接受的乘车请求。
 

表: AcceptedRides

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| ride_id       | int     |
| driver_id     | int     |
| ride_distance | int     |
| ride_duration | int     |
+---------------+---------+
ride_id是该表的主键。
该表的每一行都包含已接受的行程信息。
表中的行程信息都在“Rides”表中存在。
 

编写SQL查询以报告2020年每个月的以下统计信息：

截至某月底，当前在Hopper公司工作的驾驶员数量（active_drivers）。
该月接受的乘车次数（accepted_rides）。
返回按month 升序排列的结果表，其中month 是月份的数字（一月是1，二月是2，依此类推）。

查询结果格式如下例所示。

表 Drivers:
+-----------+------------+
| driver_id | join_date  |
+-----------+------------+
| 10        | 2019-12-10 |
| 8         | 2020-1-13  |
| 5         | 2020-2-16  |
| 7         | 2020-3-8   |
| 4         | 2020-5-17  |
| 1         | 2020-10-24 |
| 6         | 2021-1-5   |
+-----------+------------+

表 Rides:
+---------+---------+--------------+
| ride_id | user_id | requested_at |
+---------+---------+--------------+
| 6       | 75      | 2019-12-9    |
| 1       | 54      | 2020-2-9     |
| 10      | 63      | 2020-3-4     |
| 19      | 39      | 2020-4-6     |
| 3       | 41      | 2020-6-3     |
| 13      | 52      | 2020-6-22    |
| 7       | 69      | 2020-7-16    |
| 17      | 70      | 2020-8-25    |
| 20      | 81      | 2020-11-2    |
| 5       | 57      | 2020-11-9    |
| 2       | 42      | 2020-12-9    |
| 11      | 68      | 2021-1-11    |
| 15      | 32      | 2021-1-17    |
| 12      | 11      | 2021-1-19    |
| 14      | 18      | 2021-1-27    |
+---------+---------+--------------+

表 AcceptedRides:
+---------+-----------+---------------+---------------+
| ride_id | driver_id | ride_distance | ride_duration |
+---------+-----------+---------------+---------------+
| 10      | 10        | 63            | 38            |
| 13      | 10        | 73            | 96            |
| 7       | 8         | 100           | 28            |
| 17      | 7         | 119           | 68            |
| 20      | 1         | 121           | 92            |
| 5       | 7         | 42            | 101           |
| 2       | 4         | 6             | 38            |
| 11      | 8         | 37            | 43            |
| 15      | 8         | 108           | 82            |
| 12      | 8         | 38            | 34            |
| 14      | 1         | 90            | 74            |
+---------+-----------+---------------+---------------+

结果表:
+-------+----------------+----------------+
| month | active_drivers | accepted_rides |
+-------+----------------+----------------+
| 1     | 2              | 0              |
| 2     | 3              | 0              |
| 3     | 4              | 1              |
| 4     | 4              | 0              |
| 5     | 5              | 0              |
| 6     | 5              | 1              |
| 7     | 5              | 1              |
| 8     | 5              | 1              |
| 9     | 5              | 0              |
| 10    | 6              | 0              |
| 11    | 6              | 2              |
| 12    | 6              | 1              |
+-------+----------------+----------------+

截至1月底->两个活跃的驾驶员（10,8），没有被接受的行程。
截至2月底->三个活跃的驾驶员（10,8,5），没有被接受的行程。
截至3月底->四个活跃的驾驶员（10,8,5,7），一个被接受的行程（10）。
截至4月底->四个活跃的驾驶员（10,8,5,7），没有被接受的行程。
截至5月底->五个活跃的驾驶员（10,8,5,7,4），没有被接受的行程。
截至6月底->五个活跃的驾驶员（10,8,5,7,4），一个被接受的行程（13）。
截至7月底->五个活跃的驾驶员（10,8,5,7,4），一个被接受的行程（7）。
截至8月底->五个活跃的驾驶员（10,8,5,7,4），一位接受的行程（17）。
截至9月底->五个活跃的驾驶员（10,8,5,7,4），没有被接受的行程。
截至10月底->六个活跃的驾驶员（10,8,5,7,4,1），没有被接受的行程。
截至11月底->六个活跃的驾驶员（10,8,5,7,4,1），两个被接受的行程（20,5）。
截至12月底->六个活跃的驾驶员（10,8,5,7,4,1），一个被接受的行程（2）。
```
```
--有点难度,分析一下与我遇到的错误
//我一直没读懂第三列要求什么,直到看了答案,二:在求第三列的时候我要时间限制在<= 2020,事实上应该是=2020

第一步:循环求1-12
with recursive x as (
    select 1 as month
    union all 
    select month +1  from  x where month <=11
)

第二步:由题意我们要把2019年的驾驶员,归纳到1月份,事实上是要group 结合if就可以了,不需要union all 在求得每个月的新增驾驶员之后开窗获取sum() 累计的驾驶员数量
select
x.month,
ifnull(sum(driver_id_count) over(order by month),0) as  active_drivers,
from 
x left join 
(
select
if(year(join_date) < "2020",1,month(join_date))  month,
count(driver_id) driver_id_count
from 
Drivers
where  year(join_date) <= "2020"
group by if(year(join_date) < "2020",1,month(join_date))
) t1 on x.month = t1.month

第三步:和第二步一个思路,按照月份求取符合的id,注意去重,事实上,这里的题意不怎么明确,join  on 的条件是month 

综上有:下面的SQL

with recursive x as (
    select 1 as month
    union all 
    select month +1  from  x where month <=11

)
select
x.month,
ifnull(sum(driver_id_count) over(order by month),0) as  active_drivers,
ifnull(accepted_rides,0) accepted_rides
from 
x left join 
(
select
if(year(join_date) < "2020",1,month(join_date))  month,
count(driver_id) driver_id_count
from 
Drivers
where  year(join_date) <= "2020"
group by if(year(join_date) < "2020",1,month(join_date))
) t1 on x.month = t1.month
left join 
(
select
if(year(requested_at) < "2020",1,month(requested_at))  month, 
count(distinct ride_id) accepted_rides
from 
Rides
where year(requested_at) = "2020" and ride_id in (select ride_id  from AcceptedRides)
group by if(year(requested_at) < "2020",1,month(requested_at)) 
) t2
on x.month = t2.month
order by month
```
##132.hopper公司查询2
>编写一个SQL查询，报告2020年每个月工作驱动程序的百分比(working_percentage)，注意，如果一个月内可用驱动程序的数量为零，我们认为working_percentage为0,返回按月升序排序的结果表
```
Table: Drivers

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| driver_id   | int     |
| join_date   | date    |
+-------------+---------+
driver_id is the primary key for this table.
Each row of this table contains the driver's ID and the date they joined the Hopper company.
 

Table: Rides

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| ride_id      | int     |
| user_id      | int     |
| requested_at | date    |
+--------------+---------+
ride_id is the primary key for this table.
Each row of this table contains the ID of a ride, the user's ID that requested it, and the day they requested it.
There may be some ride requests in this table that were not accepted.
 

Table: AcceptedRides

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| ride_id       | int     |
| driver_id     | int     |
| ride_distance | int     |
| ride_duration | int     |
+---------------+---------+
ride_id is the primary key for this table.
Each row of this table contains some information about an accepted ride.
It is guaranteed that each accepted ride exists in the Rides table.
 

Write an SQL query to report the percentage of working drivers (working_percentage) for each month of 2020 where:


Note that if the number of available drivers during a month is zero, we consider the working_percentage to be 0.

Return the result table ordered by month in ascending order, where month is the month's number (January is 1, February is 2, etc.). Round working_percentage to the nearest 2 decimal places.

The query result format is in the following example.

 

Drivers table:
+-----------+------------+
| driver_id | join_date  |
+-----------+------------+
| 10        | 2019-12-10 |
| 8         | 2020-1-13  |
| 5         | 2020-2-16  |
| 7         | 2020-3-8   |
| 4         | 2020-5-17  |
| 1         | 2020-10-24 |
| 6         | 2021-1-5   |
+-----------+------------+

Rides table:
+---------+---------+--------------+
| ride_id | user_id | requested_at |
+---------+---------+--------------+
| 6       | 75      | 2019-12-9    |
| 1       | 54      | 2020-2-9     |
| 10      | 63      | 2020-3-4     |
| 19      | 39      | 2020-4-6     |
| 3       | 41      | 2020-6-3     |
| 13      | 52      | 2020-6-22    |
| 7       | 69      | 2020-7-16    |
| 17      | 70      | 2020-8-25    |
| 20      | 81      | 2020-11-2    |
| 5       | 57      | 2020-11-9    |
| 2       | 42      | 2020-12-9    |
| 11      | 68      | 2021-1-11    |
| 15      | 32      | 2021-1-17    |
| 12      | 11      | 2021-1-19    |
| 14      | 18      | 2021-1-27    |
+---------+---------+--------------+

AcceptedRides table:
+---------+-----------+---------------+---------------+
| ride_id | driver_id | ride_distance | ride_duration |
+---------+-----------+---------------+---------------+
| 10      | 10        | 63            | 38            |
| 13      | 10        | 73            | 96            |
| 7       | 8         | 100           | 28            |
| 17      | 7         | 119           | 68            |
| 20      | 1         | 121           | 92            |
| 5       | 7         | 42            | 101           |
| 2       | 4         | 6             | 38            |
| 11      | 8         | 37            | 43            |
| 15      | 8         | 108           | 82            |
| 12      | 8         | 38            | 34            |
| 14      | 1         | 90            | 74            |
+---------+-----------+---------------+---------------+

Result table:
+-------+--------------------+
| month | working_percentage |
+-------+--------------------+
| 1     | 0.00               |
| 2     | 0.00               |
| 3     | 25.00              |
| 4     | 0.00               |
| 5     | 0.00               |
| 6     | 20.00              |
| 7     | 20.00              |
| 8     | 20.00              |
| 9     | 0.00               |
| 10    | 0.00               |
| 11    | 33.33              |
| 12    | 16.67              |
+-------+--------------------+

By the end of January --> two active drivers (10, 8) and no accepted rides. The percentage is 0%.
By the end of February --> three active drivers (10, 8, 5) and no accepted rides. The percentage is 0%.
By the end of March --> four active drivers (10, 8, 5, 7) and one accepted ride by driver (10). The percentage is (1 / 4) * 100 = 25%.
By the end of April --> four active drivers (10, 8, 5, 7) and no accepted rides. The percentage is 0%.
By the end of May --> five active drivers (10, 8, 5, 7, 4) and no accepted rides. The percentage is 0%.
By the end of June --> five active drivers (10, 8, 5, 7, 4) and one accepted ride by driver (10). The percentage is (1 / 5) * 100 = 20%.
By the end of July --> five active drivers (10, 8, 5, 7, 4) and one accepted ride by driver (8). The percentage is (1 / 5) * 100 = 20%.
By the end of August --> five active drivers (10, 8, 5, 7, 4) and one accepted ride by driver (7). The percentage is (1 / 5) * 100 = 20%.
By the end of Septemeber --> five active drivers (10, 8, 5, 7, 4) and no accepted rides. The percentage is 0%.
By the end of October --> six active drivers (10, 8, 5, 7, 4, 1) and no accepted rides. The percentage is 0%.
By the end of November --> six active drivers (10, 8, 5, 7, 4, 1) and two accepted rides by two different drivers (1, 7). The percentage is (2 / 6) * 100 = 33.33%.
By the end of December --> six active drivers (10, 8, 5, 7, 4, 1) and one accepted ride by driver (4). The percentage is (1 / 6) * 100 = 16.67%.
```
```
--没有翻译难受,事实上就是第一题,区别就是在AcceptedRides表获取每个月的驾驶员
WITH
    recursive full_month AS 
    (
    SELECT 1 AS month
    union
    SELECT month+1 AS month FROM full_month WHERE month <= 11
    ),

    month_drivers AS
    (SELECT count(driver_id) AS nums,if(year(join_date)<'2020',1,month(join_date)) month
    FROM Drivers WHERE year(join_date) <= '2020' GROUP BY month),

    month_active AS
    (SELECT month(requested_at) month, count(distinct driver_id) AS counts
    FROM Rides r left join AcceptedRides a on r.ride_id = a.ride_id
    WHERE year(requested_at) = '2020'
    GROUP BY month)

SELECT month,
    if(driver_num=0,0,round(actice_num/driver_num*100,2)) AS working_percentage 
FROM
    (
    SELECT 
        t1.month, 
        ifnull(sum(t2.nums) over(order by month),0) AS driver_num,
        ifnull(t3.counts,0) AS actice_num
    FROM
        full_month t1 left join month_drivers t2 on t1.month = t2.month left join month_active t3 on t3.month = t1.month
    ) AS t
GROUP BY month
ORDER BY month
```
##133.hopper公司查询3
>编写一个SQL查询来计算从2020年1月- 3月到2020年10月- 12月的每个3个月窗口的平均ride_distance和平均ride_duration。将average ride_distance和average ride_duration四舍五入到小数点后两位。
```
Table: Drivers

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| driver_id   | int     |
| join_date   | date    |
+-------------+---------+
driver_id is the primary key for this table.
Each row of this table contains the driver's ID and the date they joined the Hopper company.
 

Table: Rides

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| ride_id      | int     |
| user_id      | int     |
| requested_at | date    |
+--------------+---------+
ride_id is the primary key for this table.
Each row of this table contains the ID of a ride, the user's ID that requested it, and the day they requested it.
There may be some ride requests in this table that were not accepted.
 

Table: AcceptedRides

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| ride_id       | int     |
| driver_id     | int     |
| ride_distance | int     |
| ride_duration | int     |
+---------------+---------+
ride_id is the primary key for this table.
Each row of this table contains some information about an accepted ride.
It is guaranteed that each accepted ride exists in the Rides table.
 

Write an SQL query to compute the average_ride_distance and average_ride_duration of every 3-month window starting from January - March 2020 to October - December 2020. Round average_ride_distance and average_ride_duration to the nearest two decimal places.

The average_ride_distance is calculated by summing up the total ride_distance values from the three months and dividing it by 3. The average_ride_duration is calculated in a similar way.

Return the result table ordered by month in ascending order, where month is the starting month's number (January is 1, February is 2, etc.).

The query result format is in the following example.

 

Drivers table:
+-----------+------------+
| driver_id | join_date  |
+-----------+------------+
| 10        | 2019-12-10 |
| 8         | 2020-1-13  |
| 5         | 2020-2-16  |
| 7         | 2020-3-8   |
| 4         | 2020-5-17  |
| 1         | 2020-10-24 |
| 6         | 2021-1-5   |
+-----------+------------+

Rides table:
+---------+---------+--------------+
| ride_id | user_id | requested_at |
+---------+---------+--------------+
| 6       | 75      | 2019-12-9    |
| 1       | 54      | 2020-2-9     |
| 10      | 63      | 2020-3-4     |
| 19      | 39      | 2020-4-6     |
| 3       | 41      | 2020-6-3     |
| 13      | 52      | 2020-6-22    |
| 7       | 69      | 2020-7-16    |
| 17      | 70      | 2020-8-25    |
| 20      | 81      | 2020-11-2    |
| 5       | 57      | 2020-11-9    |
| 2       | 42      | 2020-12-9    |
| 11      | 68      | 2021-1-11    |
| 15      | 32      | 2021-1-17    |
| 12      | 11      | 2021-1-19    |
| 14      | 18      | 2021-1-27    |
+---------+---------+--------------+

AcceptedRides table:
+---------+-----------+---------------+---------------+
| ride_id | driver_id | ride_distance | ride_duration |
+---------+-----------+---------------+---------------+
| 10      | 10        | 63            | 38            |
| 13      | 10        | 73            | 96            |
| 7       | 8         | 100           | 28            |
| 17      | 7         | 119           | 68            |
| 20      | 1         | 121           | 92            |
| 5       | 7         | 42            | 101           |
| 2       | 4         | 6             | 38            |
| 11      | 8         | 37            | 43            |
| 15      | 8         | 108           | 82            |
| 12      | 8         | 38            | 34            |
| 14      | 1         | 90            | 74            |
+---------+-----------+---------------+---------------+

Result table:
+-------+-----------------------+-----------------------+
| month | average_ride_distance | average_ride_duration |
+-------+-----------------------+-----------------------+
| 1     | 21.00                 | 12.67                 |
| 2     | 21.00                 | 12.67                 |
| 3     | 21.00                 | 12.67                 |
| 4     | 24.33                 | 32.00                 |
| 5     | 57.67                 | 41.33                 |
| 6     | 97.33                 | 64.00                 |
| 7     | 73.00                 | 32.00                 |
| 8     | 39.67                 | 22.67                 |
| 9     | 54.33                 | 64.33                 |
| 10    | 56.33                 | 77.00                 |
+-------+-----------------------+-----------------------+

By the end of January --> average_ride_distance = (0+0+63)/3=21, average_ride_duration = (0+0+38)/3=12.67
By the end of February --> average_ride_distance = (0+63+0)/3=21, average_ride_duration = (0+38+0)/3=12.67
By the end of March --> average_ride_distance = (63+0+0)/3=21, average_ride_duration = (38+0+0)/3=12.67
By the end of April --> average_ride_distance = (0+0+73)/3=24.33, average_ride_duration = (0+0+96)/3=32.00
By the end of May --> average_ride_distance = (0+73+100)/3=57.67, average_ride_duration = (0+96+28)/3=41.33
By the end of June --> average_ride_distance = (73+100+119)/3=97.33, average_ride_duration = (96+28+68)/3=64.00
By the end of July --> average_ride_distance = (100+119+0)/3=73.00, average_ride_duration = (28+68+0)/3=32.00
By the end of August --> average_ride_distance = (119+0+0)/3=39.67, average_ride_duration = (68+0+0)/3=22.67
By the end of Septemeber --> average_ride_distance = (0+0+163)/3=54.33, average_ride_duration = (0+0+193)/3=64.33
By the end of October --> average_ride_distance = (0+163+6)/3=56.33, average_ride_duration = (0+193+38)/3=77.00
```
```
WITH RECURSIVE cte (month) AS
(
  SELECT 1
  UNION ALL
  SELECT month + 1 FROM cte WHERE month < 10
)

SELECT
    cte.month,
    ROUND(SUM(IFNULL(ride_distance,0))/3,2) average_ride_distance,
    ROUND(SUM(IFNULL(ride_duration,0))/3,2) average_ride_duration
FROM 
    cte
LEFT JOIN
    Rides r
ON MONTH(r.requested_at) BETWEEN cte.month AND cte.month + 2 AND r.requested_at BETWEEN '2020-01-01' AND '2020-12-31'
LEFT JOIN
    AcceptedRides a
ON r.ride_id = a.ride_id
GROUP BY
    cte.month
ORDER BY
    month
```
```
WITH RECURSIVE cte(n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1 FROM cte WHERE n < 12
)

SELECT month, average_ride_distance, average_ride_duration
FROM (
SELECT cte.n AS `month`,
       ROUND(AVG(IFNULL(ride_distance, 0)) OVER(ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING), 2) AS average_ride_distance,
       ROUND(AVG(IFNULL(ride_duration, 0)) OVER(ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING), 2) AS average_ride_duration
FROM cte 
LEFT JOIN (
SELECT MONTH(requested_at) AS `month`,
       SUM(ride_distance) AS ride_distance,
       SUM(ride_duration) AS ride_duration
FROM AcceptedRides A 
JOIN Rides R USING(ride_id)
WHERE YEAR(requested_at) = 2020
GROUP BY MONTH(requested_at)) T ON cte.n = T.month) T2 
WHERE month <= 10
```
##134.每台机器的平均运行时间
>现在有一个工厂网站由几台机器运行，每台机器上运行着相同数量的进程. 请写出一条SQL计算每台机器各自完成一个进程任务的平均耗时.
完成一个进程任务的时间指进程的'end' 时间戳 减去 'start' 时间戳. 平均耗时通过计算每台机器上所有进程任务的总耗费时间除以机器上的总进程数量获得.
结果表必须包含machine_id（机器ID） 和对应的 average time（平均耗时） 别名 processing_time, 且四舍五入保留3位小数.
```
表: Activity

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| machine_id     | int     |
| process_id     | int     |
| activity_type  | enum    |
| timestamp      | float   |
+----------------+---------+
该表展示了一家工厂网站的用户活动.
(machine_id, process_id, activity_type) 是当前表的主键.
machine_id 是一台机器的ID号.
process_id 是运行在各机器上的进程ID号.
activity_type 是枚举类型 ('start', 'end').
timestamp 是浮点类型,代表当前时间(以秒为单位).
'start' 代表该进程在这台机器上的开始运行时间戳 , 'end' 代表该进程在这台机器上的终止运行时间戳.
同一台机器，同一个进程都有一对开始时间戳和结束时间戳，而且开始时间戳永远在结束时间戳前面.
 

现在有一个工厂网站由几台机器运行，每台机器上运行着相同数量的进程. 请写出一条SQL计算每台机器各自完成一个进程任务的平均耗时.

完成一个进程任务的时间指进程的'end' 时间戳 减去 'start' 时间戳. 平均耗时通过计算每台机器上所有进程任务的总耗费时间除以机器上的总进程数量获得.

结果表必须包含machine_id（机器ID） 和对应的 average time（平均耗时） 别名 processing_time, 且四舍五入保留3位小数.

具体参考例子如下:

 

Activity table:
+------------+------------+---------------+-----------+
| machine_id | process_id | activity_type | timestamp |
+------------+------------+---------------+-----------+
| 0          | 0          | start         | 0.712     |
| 0          | 0          | end           | 1.520     |
| 0          | 1          | start         | 3.140     |
| 0          | 1          | end           | 4.120     |
| 1          | 0          | start         | 0.550     |
| 1          | 0          | end           | 1.550     |
| 1          | 1          | start         | 0.430     |
| 1          | 1          | end           | 1.420     |
| 2          | 0          | start         | 4.100     |
| 2          | 0          | end           | 4.512     |
| 2          | 1          | start         | 2.500     |
| 2          | 1          | end           | 5.000     |
+------------+------------+---------------+-----------+

Result table:
+------------+-----------------+
| machine_id | processing_time |
+------------+-----------------+
| 0          | 0.894           |
| 1          | 0.995           |
| 2          | 1.456           |
+------------+-----------------+

一共有3台机器,每台机器运行着两个进程.
机器 0 的平均耗时: ((1.520 - 0.712) + (4.120 - 3.140)) / 2 = 0.894
机器 1 的平均耗时: ((1.550 - 0.550) + (1.420 - 0.430)) / 2 = 0.995
机器 2 的平均耗时: ((4.512 - 4.100) + (5.000 - 2.500)) / 2 = 1.456
```
```
--简单题,终于是中文的了.....
select
machine_id,
round(sum(if(activity_type = "end",timestamp,-timestamp)) / count(distinct process_id ) ,3) processing_time
from 
Activity
group by 
machine_id
```
##135.修复表中的名字
>编写一个 SQL 查询来修复名字，使得只有第一个字符是大写的，其余都是小写的。
```表： Users

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| user_id        | int     |
| name           | varchar |
+----------------+---------+
user_id 是该表的主键。
该表包含用户的 ID 和名字。名字仅由小写和大写字符组成。
 

编写一个 SQL 查询来修复名字，使得只有第一个字符是大写的，其余都是小写的。

返回按 user_id 排序的结果表。

查询结果格式示例如下：

Users table:
+---------+-------+
| user_id | name  |
+---------+-------+
| 1       | aLice |
| 2       | bOB   |
+---------+-------+

Result table:
+---------+-------+
| user_id | name  |
+---------+-------+
| 1       | Alice |
| 2       | Bob   |
+---------+-------+
```
```
--简单题
select
user_id,
concat(upper(left(name,1)),lower(substr(name,2))) as name
from 
Users
order by 
user_id
```
```
--length函数
select user_id,
concat(upper(substr(name,1,1)),lower(substr(name,2,length(name)-1))) as name

from users
order by user_id
```
##136.发票中的产品金额
>要求写一个SQL查询，返回全部发票中每个产品的产品名称、总应缴款项、总已支付金额、总已取消金额、总已退款金额
查询结果按 product_name排序
```
Product 表：

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_id  | int     |
| name        | varchar |
+-------------+---------+
product_id 是这张表的主键
表中含有产品 id 、产品名称。产品名称都是小写的英文字母，产品名称都是唯一的
 

Invoice 表：

+-------------+------+
| Column Name | Type |
+-------------+------+
| invoice_id  | int  |
| product_id  | int  |
| rest        | int  |
| paid        | int  |
| canceled    | int  |
| refunded    | int  |
+-------------+------+
invoice_id 发票 id ，是这张表的主键
product_id 产品 id
rest 应缴款项
paid 已支付金额
canceled 已取消金额
refunded 已退款金额
 

要求写一个SQL查询，返回全部发票中每个产品的产品名称、总应缴款项、总已支付金额、总已取消金额、总已退款金额

查询结果按 product_name排序

示例：

Product 表：
+------------+-------+
| product_id | name  |
+------------+-------+
| 0          | ham   |
| 1          | bacon |
+------------+-------+
Invoice table:
+------------+------------+------+------+----------+----------+
| invoice_id | product_id | rest | paid | canceled | refunded |
+------------+------------+------+------+----------+----------+
| 23         | 0          | 2    | 0    | 5        | 0        |
| 12         | 0          | 0    | 4    | 0        | 3        |
| 1          | 1          | 1    | 1    | 0        | 1        |
| 2          | 1          | 1    | 0    | 1        | 1        |
| 3          | 1          | 0    | 1    | 1        | 1        |
| 4          | 1          | 1    | 1    | 1        | 0        |
+------------+------------+------+------+----------+----------+
Result 表：
+-------+------+------+----------+----------+
| name  | rest | paid | canceled | refunded |
+-------+------+------+----------+----------+
| bacon | 3    | 3    | 3        | 3        |
| ham   | 2    | 4    | 5        | 3        |
+-------+------+------+----------+----------+
- bacon 的总应缴款项为 1 + 1 + 0 + 1 = 3
- bacon 的总已支付金额为 1 + 0 + 1 + 1 = 3
- bacon 的总已取消金额为 0 + 1 + 1 + 1 = 3
- bacon 的总已退款金额为 1 + 1 + 1 + 0 = 3
- ham 的总应缴款项为 2 + 0 = 2
- ham 的总已支付金额为 0 + 4 = 4
- ham 的总已取消金额为 5 + 0 = 5
- ham 的总已退款金额为 0 + 3 = 3
```
```
select
name,
sum( rest)   rest ,
sum(paid) paid,
sum(canceled) canceled,
sum(refunded ) refunded 
from 
Product t1  join Invoice t2 on t1. product_id =t2. product_id
group by t1. product_id ,name 
order by name 
```
##137.无效的推文
>写一条 SQL 语句，查询所有无效推文的编号（ID）。当推文内容中的字符数严格大于 15 时，该推文是无效的。
```
表：Tweets

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| tweet_id       | int     |
| content        | varchar |
+----------------+---------+
tweet_id 是这个表的主键。
这个表包含某社交媒体 App 中所有的推文。
 

写一条 SQL 语句，查询所有无效推文的编号（ID）。当推文内容中的字符数严格大于 15 时，该推文是无效的。

以任意顺序返回结果表。

查询结果格式如下示例所示：

 

Tweets 表：
+----------+----------------------------------+
| tweet_id | content                          |
+----------+----------------------------------+
| 1        | Vote for Biden                   |
| 2        | Let us make America great again! |
+----------+----------------------------------+

结果表：
+----------+
| tweet_id |
+----------+
| 2        |
+----------+
推文 1 的长度 length = 14。该推文是有效的。
推文 2 的长度 length = 32。该推文是无效的。
```
```
--简单题
select
tweet_id 
from
Tweets
where length(trim(content )) >15
```
>1、char_length(str)
（1）计算单位：字符
（2）不管汉字还是数字或者是字母都算是一个字符
2、length(str)
>（1）计算单位：字节
>（2）utf8编码：一个汉字三个字节，一个数字或字母一个字节。
（3）gbk编码：一个汉字两个字节，一个数字或字母一个字节。
//注意一下上面的区别

##138.每天的领导和合伙人
>写一条 SQL 语句，使得对于每一个 date_id 和 make_name，返回不同的 lead_id 以及不同的 partner_id 的数量。
```
表：DailySales

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| date_id     | date    |
| make_name   | varchar |
| lead_id     | int     |
| partner_id  | int     |
+-------------+---------+
该表没有主键。
该表包含日期、产品的名称，以及售给的领导和合伙人的编号。
名称只包含小写英文字母。
 

写一条 SQL 语句，使得对于每一个 date_id 和 make_name，返回不同的 lead_id 以及不同的 partner_id 的数量。

按任意顺序返回结果表。

查询结果格式如下示例所示：

 

DailySales 表：
+-----------+-----------+---------+------------+
| date_id   | make_name | lead_id | partner_id |
+-----------+-----------+---------+------------+
| 2020-12-8 | toyota    | 0       | 1          |
| 2020-12-8 | toyota    | 1       | 0          |
| 2020-12-8 | toyota    | 1       | 2          |
| 2020-12-7 | toyota    | 0       | 2          |
| 2020-12-7 | toyota    | 0       | 1          |
| 2020-12-8 | honda     | 1       | 2          |
| 2020-12-8 | honda     | 2       | 1          |
| 2020-12-7 | honda     | 0       | 1          |
| 2020-12-7 | honda     | 1       | 2          |
| 2020-12-7 | honda     | 2       | 1          |
+-----------+-----------+---------+------------+
结果表：
+-----------+-----------+--------------+-----------------+
| date_id   | make_name | unique_leads | unique_partners |
+-----------+-----------+--------------+-----------------+
| 2020-12-8 | toyota    | 2            | 3               |
| 2020-12-7 | toyota    | 1            | 2               |
| 2020-12-8 | honda     | 2            | 2               |
| 2020-12-7 | honda     | 3            | 2               |
+-----------+-----------+--------------+-----------------+
在 2020-12-8，丰田（toyota）有领导者 = [0, 1] 和合伙人 = [0, 1, 2] ，同时本田（honda）有领导者 = [1, 2] 和合伙人 = [1, 2]。
在 2020-12-7，丰田（toyota）有领导者 = [0] 和合伙人 = [1, 2] ，同时本田（honda）有领导者 = [0, 1, 2] 和合伙人 = [1, 2]。
```
```
--简单题
select
date_id,make_name,
count(distinct lead_id) unique_leads ,
count(distinct partner_id )  unique_partners
from 
DailySales
group by  date_id , make_name
```
##139.两人之间的通话次数
>编写 SQL 语句，查询每一对用户 (person1, person2) 之间的通话次数和通话总时长，其中 person1 < person2 。
```
表： Calls

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| from_id     | int     |
| to_id       | int     |
| duration    | int     |
+-------------+---------+
该表没有主键，可能存在重复项。
该表包含 from_id 与 to_id 间的一次电话的时长。
from_id != to_id
 

编写 SQL 语句，查询每一对用户 (person1, person2) 之间的通话次数和通话总时长，其中 person1 < person2 。

以任意顺序返回结果表。

查询结果格式如下示例所示：

 

Calls 表：
+---------+-------+----------+
| from_id | to_id | duration |
+---------+-------+----------+
| 1       | 2     | 59       |
| 2       | 1     | 11       |
| 1       | 3     | 20       |
| 3       | 4     | 100      |
| 3       | 4     | 200      |
| 3       | 4     | 200      |
| 4       | 3     | 499      |
+---------+-------+----------+

结果表：
+---------+---------+------------+----------------+
| person1 | person2 | call_count | total_duration |
+---------+---------+------------+----------------+
| 1       | 2       | 2          | 70             |
| 1       | 3       | 1          | 20             |
| 3       | 4       | 4          | 999            |
+---------+---------+------------+----------------+
用户 1 和 2 打过 2 次电话，总时长为 70 (59 + 11)
用户 1 和 3 打过 1 次电话，总时长为 20
用户 3 和 4 打过 4 次电话，总时长为 999 (100 + 200 + 200 + 499)
```
```
--简单题
select
person1,person2,
count(1) as  call_count,
sum(duration) total_duration
from 
(
select
if(from_id<to_id,from_id,to_id)  person1,
if(from_id>to_id,from_id,to_id)  person2,
duration
from
Calls 
) t1
group by 1,2
```
##140.访问日期之间最大的空档期
>假设今天的日期是 '2021-1-1' 。
编写 SQL 语句，对于每个 user_id ，求出每次访问及其下一个访问（若该次访问是最后一次，则为今天）之间最大的空档期天数 window 。
返回结果表，按用户编号 user_id 排序
```
表： UserVisits

+-------------+------+
| Column Name | Type |
+-------------+------+
| user_id     | int  |
| visit_date  | date |
+-------------+------+
该表没有主键。
该表包含用户访问某特定零售商的日期日志。
 

假设今天的日期是 '2021-1-1' 。

编写 SQL 语句，对于每个 user_id ，求出每次访问及其下一个访问（若该次访问是最后一次，则为今天）之间最大的空档期天数 window 。

返回结果表，按用户编号 user_id 排序。

查询格式如下示例所示：

 

UserVisits 表：
+---------+------------+
| user_id | visit_date |
+---------+------------+
| 1       | 2020-11-28 |
| 1       | 2020-10-20 |
| 1       | 2020-12-3  |
| 2       | 2020-10-5  |
| 2       | 2020-12-9  |
| 3       | 2020-11-11 |
+---------+------------+
结果表：
+---------+---------------+
| user_id | biggest_window|
+---------+---------------+
| 1       | 39            |
| 2       | 65            |
| 3       | 51            |
+---------+---------------+
对于第一个用户，问题中的空档期在以下日期之间：
    - 2020-10-20 至 2020-11-28 ，共计 39 天。
    - 2020-11-28 至 2020-12-3 ，共计 5 天。
    - 2020-12-3 至 2021-1-1 ，共计 29 天。
由此得出，最大的空档期为 39 天。
对于第二个用户，问题中的空档期在以下日期之间：
    - 2020-10-5 至 2020-12-9 ，共计 65 天。
    - 2020-12-9 至 2021-1-1 ，共计 23 天。
由此得出，最大的空档期为 65 天。
对于第三个用户，问题中的唯一空档期在 2020-11-11 至 2021-1-1 之间，共计 51 天。

```
```
--简单题,好久没碰到偏移量的题目了....
select
user_id, max(rn) biggest_window
from 
(
select
user_id,
datediff(
lead(visit_date,1,"2021-01-01") over(partition by user_id order by visit_date ) ,visit_date) rn
from
UserVisits
) t1
group by user_id
order by user_id
```

