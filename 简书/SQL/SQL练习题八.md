##81.平均售价
>编写SQL查询以查找每种产品的平均售价。
average_price 应该四舍五入到小数点后两位。
```
Table: Prices

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| start_date    | date    |
| end_date      | date    |
| price         | int     |
+---------------+---------+
(product_id，start_date，end_date) 是 Prices 表的主键。
Prices 表的每一行表示的是某个产品在一段时期内的价格。
每个产品的对应时间段是不会重叠的，这也意味着同一个产品的价格时段不会出现交叉。
 

Table: UnitsSold

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| purchase_date | date    |
| units         | int     |
+---------------+---------+
UnitsSold 表没有主键，它可能包含重复项。
UnitsSold 表的每一行表示的是每种产品的出售日期，单位和产品 id。
 

编写SQL查询以查找每种产品的平均售价。
average_price 应该四舍五入到小数点后两位。
查询结果格式如下例所示：

Prices table:
+------------+------------+------------+--------+
| product_id | start_date | end_date   | price  |
+------------+------------+------------+--------+
| 1          | 2019-02-17 | 2019-02-28 | 5      |
| 1          | 2019-03-01 | 2019-03-22 | 20     |
| 2          | 2019-02-01 | 2019-02-20 | 15     |
| 2          | 2019-02-21 | 2019-03-31 | 30     |
+------------+------------+------------+--------+
 
UnitsSold table:
+------------+---------------+-------+
| product_id | purchase_date | units |
+------------+---------------+-------+
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |
| 2          | 2019-03-22    | 30    |
+------------+---------------+-------+

Result table:
+------------+---------------+
| product_id | average_price |
+------------+---------------+
| 1          | 6.96          |
| 2          | 16.96         |
+------------+---------------+
平均售价 = 产品总价 / 销售的产品数量。
产品 1 的平均售价 = ((100 * 5)+(15 * 20) )/ 115 = 6.96
产品 2 的平均售价 = ((200 * 15)+(30 * 30) )/ 230 = 16.96
```
```
--简单题,我的写法
select
t1.product_id product_id,ifnull(round(sum(price*units) / sum(units),2),0) average_price
from
Prices t1
left join
UnitsSold t2
on t1.product_id = t2.product_id and purchase_date between start_date and end_date
group by t1.product_id
```
##82.页面推荐
>写一段 SQL  向user_id = 1 的用户，推荐其朋友们喜欢的页面。不要推荐该用户已经喜欢的页面。
```
朋友关系列表： Friendship

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user1_id      | int     |
| user2_id      | int     |
+---------------+---------+
这张表的主键是 (user1_id, user2_id)。
这张表的每一行代表着 user1_id 和 user2_id 之间存在着朋友关系。
 

喜欢列表： Likes

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| user_id     | int     |
| page_id     | int     |
+-------------+---------+
这张表的主键是 (user_id, page_id)。
这张表的每一行代表着 user_id 喜欢 page_id。
 

写一段 SQL  向user_id = 1 的用户，推荐其朋友们喜欢的页面。不要推荐该用户已经喜欢的页面。

你返回的结果中不应当包含重复项。

返回结果的格式如下例所示：

Friendship table:
+----------+----------+
| user1_id | user2_id |
+----------+----------+
| 1        | 2        |
| 1        | 3        |
| 1        | 4        |
| 2        | 3        |
| 2        | 4        |
| 2        | 5        |
| 6        | 1        |
+----------+----------+
 
Likes table:
+---------+---------+
| user_id | page_id |
+---------+---------+
| 1       | 88      |
| 2       | 23      |
| 3       | 24      |
| 4       | 56      |
| 5       | 11      |
| 6       | 33      |
| 2       | 77      |
| 3       | 77      |
| 6       | 88      |
+---------+---------+

Result table:
+------------------+
| recommended_page |
+------------------+
| 23               |
| 24               |
| 56               |
| 33               |
| 77               |
+------------------+
用户1 同 用户2, 3, 4, 6 是朋友关系。
推荐页面为： 页面23 来自于 用户2, 页面24 来自于 用户3, 页面56 来自于 用户3 以及 页面33 来自于 用户6。
页面77 同时被 用户2 和 用户3 推荐。
页面88 没有被推荐，因为 用户1 已经喜欢了它。

```
```
--简单题我的写法,使用union all  代替 union 提高速度,在外层来去重
select
distinct page_id as recommended_page
from 
Likes
join 
(
select
user2_id as user1_f
from
Friendship
where user1_id = 1
union 
select
user1_id
from
Friendship
where user2_id = 1
) t1 
on Likes.user_id = t1.user1_f
where page_id not in (select page_id from Likes where user_id =1 )
```
##83.向公司CEO汇报的所有人
>用 SQL 查询出所有直接或间接向公司 CEO 汇报工作的职工的 employee_id 。
```
员工表：Employees

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| employee_id   | int     |
| employee_name | varchar |
| manager_id    | int     |
+---------------+---------+
employee_id 是这个表的主键。
这个表中每一行中，employee_id 表示职工的 ID，employee_name 表示职工的名字，manager_id 表示该职工汇报工作的直线经理。
这个公司 CEO 是 employee_id = 1 的人。
 

用 SQL 查询出所有直接或间接向公司 CEO 汇报工作的职工的 employee_id 。

由于公司规模较小，经理之间的间接关系不超过 3 个经理。

可以以任何顺序返回的结果，不需要去重。

查询结果示例如下：

Employees table:
+-------------+---------------+------------+
| employee_id | employee_name | manager_id |
+-------------+---------------+------------+
| 1           | Boss          | 1          |
| 3           | Alice         | 3          |
| 2           | Bob           | 1          |
| 4           | Daniel        | 2          |
| 7           | Luis          | 4          |
| 8           | Jhon          | 3          |
| 9           | Angela        | 8          |
| 77          | Robert        | 1          |
+-------------+---------------+------------+

Result table:
+-------------+
| employee_id |
+-------------+
| 2           |
| 77          |
| 4           |
| 7           |
+-------------+

公司 CEO 的 employee_id 是 1.
employee_id 是 2 和 77 的职员直接汇报给公司 CEO。
employee_id 是 4 的职员间接汇报给公司 CEO 4 --> 2 --> 1 。
employee_id 是 7 的职员间接汇报给公司 CEO 7 --> 4 --> 2 --> 1 。
employee_id 是 3, 8 ，9 的职员不会直接或间接的汇报给公司 CEO。
```
```
--简单题,我的写法读懂题意,注意or要打括号
select
a.employee_id as employee_id
from
Employees a
left join Employees b on a.manager_id = b.employee_id
left join Employees c on b.manager_id = c.employee_id
left join Employees d on c.manager_id = d.employee_id
where (b.manager_id =1 or c.manager_id =1 or d.manager_id =1 ) and a.employee_id != 1
```
```
-- 考虑如果第二个就是1的话与第三张表连接一定是1-1所以成立
select distinct
    E1.employee_id
from Employees E1, Employees E2, Employees E3
where E1.manager_id = E2.employee_id
and E2.manager_id = E3.employee_id
and E1.employee_id != 1 
and E3.manager_id = 1;
```
##84.学生参加各科的测试次数
>要求写一段 SQL 语句，查询出每个学生参加每一门科目测试的次数，结果按 student_id 和 subject_name 排序。
```
学生表: Students

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
主键为 student_id（学生ID），该表内的每一行都记录有学校一名学生的信息。
 

科目表: Subjects

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| subject_name | varchar |
+--------------+---------+
主键为 subject_name（科目名称），每一行记录学校的一门科目名称。
 

考试表: Examinations

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| student_id   | int     |
| subject_name | varchar |
+--------------+---------+
这张表压根没有主键，可能会有重复行。
学生表里的一个学生修读科目表里的每一门科目，而这张考试表的每一行记录就表示学生表里的某个学生参加了一次科目表里某门科目的测试。
 

要求写一段 SQL 语句，查询出每个学生参加每一门科目测试的次数，结果按 student_id 和 subject_name 排序。

查询结构格式如下所示：

Students table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
Subjects table:
+--------------+
| subject_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+
Examinations table:
+------------+--------------+
| student_id | subject_name |
+------------+--------------+
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |
+------------+--------------+
Result table:
+------------+--------------+--------------+----------------+
| student_id | student_name | subject_name | attended_exams |
+------------+--------------+--------------+----------------+
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |
+------------+--------------+--------------+----------------+
结果表需包含所有学生和所有科目（即便测试次数为0）：
Alice 参加了 3 次数学测试, 2 次物理测试，以及 1 次编程测试；
Bob 参加了 1 次数学测试, 1 次编程测试，没有参加物理测试；
Alex 啥测试都没参加；
John  参加了数学、物理、编程测试各 1 次
```
```
--有点坑,简单题要借助第三张表来实现
-- 直接count(t2.subject_name)  attended_exams 不用ifnull
select
t1.student_id,max(t1.student_name) student_name ,t1.subject_name,
ifnull(sum(if(t2.subject_name is not null,1,0)),0)  attended_exams
from 
(
select
student_id,student_name,subject_name
from 
Students
cross join Subjects
) t1 
left join 
Examinations t2 on t1.student_id = t2.student_id and t1.subject_name = t2.subject_name
group by t1.student_id,t1.subject_name
order by t1.student_id,subject_name
```
##85.找到连续区间的开始和结束数字
>后来一些 ID 从 Logs 表中删除。编写一个 SQL 查询得到 Logs 表中的连续区间的开始数字和结束数字。
```
表：Logs

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| log_id        | int     |
+---------------+---------+
id 是上表的主键。
上表的每一行包含日志表中的一个 ID。
 

后来一些 ID 从 Logs 表中删除。编写一个 SQL 查询得到 Logs 表中的连续区间的开始数字和结束数字。

将查询表按照 start_id 排序。

查询结果格式如下面的例子：

Logs 表：
+------------+
| log_id     |
+------------+
| 1          |
| 2          |
| 3          |
| 7          |
| 8          |
| 10         |
+------------+

结果表：
+------------+--------------+
| start_id   | end_id       |
+------------+--------------+
| 1          | 3            |
| 7          | 8            |
| 10         | 10           |
+------------+--------------+
结果表应包含 Logs 表中的所有区间。
从 1 到 3 在表中。
从 4 到 6 不在表中。
从 7 到 8 在表中。
9 不在表中。
10 在表中。

```
```
--简单题,窗口写法
select
min(log_id) start_id  ,max(log_id) end_id    
from 
(
select
log_id,
(log_id - row_number() over(order by log_id) ) as rn 
from
Logs
) t1
group by rn 
order by start_id
```
```
select a.log_id as START_ID ,min(b.log_id) as END_ID from 
(select log_id from logs where log_id-1 not in (select * from logs)) a,
(select log_id from logs where log_id+1 not in (select * from logs)) b
where b.log_id>=a.log_id
group by a.log_id
```
##86.不同国家的天气类型
>写一段 SQL 来找到表中每个国家在 2019 年 11 月的天气类型
天气类型的定义如下：当 weather_state 的平均值小于或等于15返回 Cold，当 weather_state 的平均值大于或等于 25 返回 Hot，否则返回 Warm。
```
国家表：Countries

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| country_id    | int     |
| country_name  | varchar |
+---------------+---------+
country_id 是这张表的主键。
该表的每行有 country_id 和 country_name 两列。
 

天气表：Weather

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| country_id    | int     |
| weather_state | varchar |
| day           | date    |
+---------------+---------+
(country_id, day) 是该表的复合主键。
该表的每一行记录了某个国家某一天的天气情况。
 

写一段 SQL 来找到表中每个国家在 2019 年 11 月的天气类型。

天气类型的定义如下：当 weather_state 的平均值小于或等于15返回 Cold，当 weather_state 的平均值大于或等于 25 返回 Hot，否则返回 Warm。

你可以以任意顺序返回你的查询结果。

查询结果格式如下所示：

Countries table:
+------------+--------------+
| country_id | country_name |
+------------+--------------+
| 2          | USA          |
| 3          | Australia    |
| 7          | Peru         |
| 5          | China        |
| 8          | Morocco      |
| 9          | Spain        |
+------------+--------------+
Weather table:
+------------+---------------+------------+
| country_id | weather_state | day        |
+------------+---------------+------------+
| 2          | 15            | 2019-11-01 |
| 2          | 12            | 2019-10-28 |
| 2          | 12            | 2019-10-27 |
| 3          | -2            | 2019-11-10 |
| 3          | 0             | 2019-11-11 |
| 3          | 3             | 2019-11-12 |
| 5          | 16            | 2019-11-07 |
| 5          | 18            | 2019-11-09 |
| 5          | 21            | 2019-11-23 |
| 7          | 25            | 2019-11-28 |
| 7          | 22            | 2019-12-01 |
| 7          | 20            | 2019-12-02 |
| 8          | 25            | 2019-11-05 |
| 8          | 27            | 2019-11-15 |
| 8          | 31            | 2019-11-25 |
| 9          | 7             | 2019-10-23 |
| 9          | 3             | 2019-12-23 |
+------------+---------------+------------+
Result table:
+--------------+--------------+
| country_name | weather_type |
+--------------+--------------+
| USA          | Cold         |
| Austraila    | Cold         |
| Peru         | Hot          |
| China        | Warm         |
| Morocco      | Hot          |
+--------------+--------------+
USA 11 月的平均 weather_state 为 (15) / 1 = 15 所以天气类型为 Cold。
Australia 11 月的平均 weather_state 为 (-2 + 0 + 3) / 3 = 0.333 所以天气类型为 Cold。
Peru 11 月的平均 weather_state 为 (25) / 1 = 25 所以天气类型为 Hot。
China 11 月的平均 weather_state 为 (16 + 18 + 21) / 3 = 18.333 所以天气类型为 Warm。
Morocco 11 月的平均 weather_state 为 (25 + 27 + 31) / 3 = 27.667 所以天气类型为 Hot。
我们并不知道 Spain 在 11 月的 weather_state 情况所以无需将他包含在结果中。
```
```
--简单题,mysql使用left老忘记,date_format(w.day,'%Y-%m')='2019-11'也行
select 
country_name,case when s1 <= 15 then 'Cold' when s1>= 25 then 'Hot' else 'Warm' end  weather_type
from 
Countries join 
(
select 
country_id,avg(weather_state) as s1
from 
Weather
where to_char(day,'yyyy-mm') = '2019-11'
group by country_id,to_char(day,'yyyy-mm')
) t1
on Countries.country_id = t1.country_id
```
##87.团队人数
>编写一个 SQL 查询，以求得每个员工所在团队的总人数
```
员工表：Employee

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| employee_id   | int     |
| team_id       | int     |
+---------------+---------+
employee_id 字段是这张表的主键，表中的每一行都包含每个员工的 ID 和他们所属的团队。
编写一个 SQL 查询，以求得每个员工所在团队的总人数。

查询结果中的顺序无特定要求。

查询结果格式示例如下：

Employee Table:
+-------------+------------+
| employee_id | team_id    |
+-------------+------------+
|     1       |     8      |
|     2       |     8      |
|     3       |     8      |
|     4       |     7      |
|     5       |     9      |
|     6       |     9      |
+-------------+------------+
Result table:
+-------------+------------+
| employee_id | team_size  |
+-------------+------------+
|     1       |     3      |
|     2       |     3      |
|     3       |     3      |
|     4       |     1      |
|     5       |     2      |
|     6       |     2      |
+-------------+------------+
ID 为 1、2、3 的员工是 team_id 为 8 的团队的成员，
ID 为 4 的员工是 team_id 为 7 的团队的成员，
ID 为 5、6 的员工是 team_id 为 9 的团队的成员。
```
```
-- 简单题,我的写法
select
employee_id,team_size
from
Employee t1 
join 
(
select
team_id,count(employee_id) team_size
from
Employee
group by team_id
) t2 
on t1.team_id = t2.team_id
```
```
select employee_id, count(*) over(partition by team_id) as team_size
from employee
```
##88.不同性别每日分数统计
>写一条SQL语句查询每种性别在每一天的总分，并按性别和日期对查询结果排序
```
表: Scores

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| player_name   | varchar |
| gender        | varchar |
| day           | date    |
| score_points  | int     |
+---------------+---------+
(gender, day)是该表的主键
一场比赛是在女队和男队之间举行的
该表的每一行表示一个名叫 (player_name) 性别为 (gender) 的参赛者在某一天获得了 (score_points) 的分数
如果参赛者是女性，那么 gender 列为 'F'，如果参赛者是男性，那么 gender 列为 'M'
 

写一条SQL语句查询每种性别在每一天的总分，并按性别和日期对查询结果排序

下面是查询结果格式的例子：

Scores表:
+-------------+--------+------------+--------------+
| player_name | gender | day        | score_points |
+-------------+--------+------------+--------------+
| Aron        | F      | 2020-01-01 | 17           |
| Alice       | F      | 2020-01-07 | 23           |
| Bajrang     | M      | 2020-01-07 | 7            |
| Khali       | M      | 2019-12-25 | 11           |
| Slaman      | M      | 2019-12-30 | 13           |
| Joe         | M      | 2019-12-31 | 3            |
| Jose        | M      | 2019-12-18 | 2            |
| Priya       | F      | 2019-12-31 | 23           |
| Priyanka    | F      | 2019-12-30 | 17           |
+-------------+--------+------------+--------------+
结果表:
+--------+------------+-------+
| gender | day        | total |
+--------+------------+-------+
| F      | 2019-12-30 | 17    |
| F      | 2019-12-31 | 40    |
| F      | 2020-01-01 | 57    |
| F      | 2020-01-07 | 80    |
| M      | 2019-12-18 | 2     |
| M      | 2019-12-25 | 13    |
| M      | 2019-12-30 | 26    |
| M      | 2019-12-31 | 29    |
| M      | 2020-01-07 | 36    |
+--------+------------+-------+
女性队伍:
第一天是 2019-12-30，Priyanka 获得 17 分，队伍的总分是 17 分
第二天是 2019-12-31, Priya 获得 23 分，队伍的总分是 40 分
第三天是 2020-01-01, Aron 获得 17 分，队伍的总分是 57 分
第四天是 2020-01-07, Alice 获得 23 分，队伍的总分是 80 分
男性队伍：
第一天是 2019-12-18, Jose 获得 2 分，队伍的总分是 2 分
第二天是 2019-12-25, Khali 获得 11 分，队伍的总分是 13 分
第三天是 2019-12-30, Slaman 获得 13 分，队伍的总分是 26 分
第四天是 2019-12-31, Joe 获得 3 分，队伍的总分是 29 分
第五天是 2020-01-07, Bajrang 获得 7 分，队伍的总分是 36 分
```
```
--简单题窗口写法
select 
gender,day,sum(score_points) over(partition by gender order by day) as total
from 
(
select
gender,to_char(day,'yyyy-mm-dd') as day ,sum(score_points) score_points
from 
Scores
group by  gender,day
) t1
```
```
SELECT
    S1.gender,
    S1.day,
    SUM(S2.score_points) total
FROM
    Scores S1
LEFT JOIN
    Scores S2
ON S1.gender = S2.gender AND S1.day >= S2.day
GROUP BY
    S1.gender,
    S1.day
ORDER BY
    S1.gender,
    S1.day
```
##89.餐厅营业额变化增长(窗口函数)
>写一条 SQL 查询计算以 7 天（某日期 + 该日期前的 6 天）为一个时间段的顾客消费平均值
```
表: Customer

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| name          | varchar |
| visited_on    | date    |
| amount        | int     |
+---------------+---------+
(customer_id, visited_on) 是该表的主键
该表包含一家餐馆的顾客交易数据
visited_on 表示 (customer_id) 的顾客在 visited_on 那天访问了餐馆
amount 是一个顾客某一天的消费总额
 

你是餐馆的老板，现在你想分析一下可能的营业额变化增长（每天至少有一位顾客）

写一条 SQL 查询计算以 7 天（某日期 + 该日期前的 6 天）为一个时间段的顾客消费平均值

查询结果格式的例子如下：

查询结果按 visited_on 排序
average_amount 要 保留两位小数，日期数据的格式为 ('YYYY-MM-DD')
 

Customer 表:
+-------------+--------------+--------------+-------------+
| customer_id | name         | visited_on   | amount      |
+-------------+--------------+--------------+-------------+
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 
+-------------+--------------+--------------+-------------+

结果表:
+--------------+--------------+----------------+
| visited_on   | amount       | average_amount |
+--------------+--------------+----------------+
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |
+--------------+--------------+----------------+

第一个七天消费平均值从 2019-01-01 到 2019-01-07 是 (100 + 110 + 120 + 130 + 110 + 140 + 150)/7 = 122.86
第二个七天消费平均值从 2019-01-02 到 2019-01-08 是 (110 + 120 + 130 + 110 + 140 + 150 + 80)/7 = 120
第三个七天消费平均值从 2019-01-03 到 2019-01-09 是 (120 + 130 + 110 + 140 + 150 + 80 + 110)/7 = 120
第四个七天消费平均值从 2019-01-04 到 2019-01-10 是 (130 + 110 + 140 + 150 + 80 + 110 + 130 + 150)/7 = 142.8
```
```
--简单题,窗口写法我写的比较麻烦,直接过滤就行了
--例如 having datediff(max(b.visited_on),min(b.visited_on)) = 6
select
t1.visited_on,round(s1,2) as amount,round(s2,2) as average_amount
from 
(
select
distinct to_char(visited_on ,'yyyy-mm-dd') visited_on
from 
Customer
where to_char(visited_on - 6,'yyyy-mm-dd') 
in (select to_char(visited_on ,'yyyy-mm-dd') from  Customer ) ) t1
join 
(
select
to_char(visited_on ,'yyyy-mm-dd') visited,
sum(sum(amount)) over(order by visited_on rows between 6 preceding and current row) s1 ,
avg(sum(amount)) over(order by visited_on rows between 6 preceding and current row) s2
from 
Customer
group by visited_on ) t2
on t1.visited_on = t2.visited
```
```
--用序列函数来过滤.....超时了....
select
visited_on,round(s1,2) as amount,round(s2,2) as average_amount
from 
(
select
to_char(visited_on ,'yyyy-mm-dd') visited_on,
sum(sum(amount)) over(order by visited_on rows between 6 preceding and current row) s1 ,
avg(sum(amount)) over(order by visited_on rows between 6 preceding and current row) s2 ,
rank() over(order by visited_on) s3
from 
Customer
group by visited_on ) t2
where s3>=7
```
```
--评论区的自连接写法
SELECT
	a.visited_on,
	sum( b.amount ) AS amount,
	round(sum( b.amount ) / 7, 2 ) AS average_amount 
FROM
	(SELECT DISTINCT visited_on FROM customer) a 
JOIN 
	customer b 
ON 
	datediff( a.visited_on, b.visited_on ) BETWEEN 0 AND 6 
WHERE
	a.visited_on >= (SELECT min(visited_on) FROM customer) + 6 
GROUP BY
	a.visited_on
```
主要学习要点就是怎么过滤出有前七天时间的日期(效率)
学习一下窗口大小 还有datediff()与between and 的连用
##90.广告效果
>写一条SQL语句来查询每一条广告的 ctr
```
表: Ads

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| ad_id         | int     |
| user_id       | int     |
| action        | enum    |
+---------------+---------+
(ad_id, user_id) 是该表的主键
该表的每一行包含一条广告的 ID(ad_id)，用户的 ID(user_id) 和用户对广告采取的行为 (action)
action 列是一个枚举类型 ('Clicked', 'Viewed', 'Ignored') 。
 

一家公司正在运营这些广告并想计算每条广告的效果。

广告效果用点击通过率（Click-Through Rate：CTR）来衡量，公式如下:



写一条SQL语句来查询每一条广告的 ctr ，

 ctr 要保留两位小数。结果需要按 ctr 降序、按 ad_id 升序 进行排序。

 

查询结果示例如下：

Ads 表:
+-------+---------+---------+
| ad_id | user_id | action  |
+-------+---------+---------+
| 1     | 1       | Clicked |
| 2     | 2       | Clicked |
| 3     | 3       | Viewed  |
| 5     | 5       | Ignored |
| 1     | 7       | Ignored |
| 2     | 7       | Viewed  |
| 3     | 5       | Clicked |
| 1     | 4       | Viewed  |
| 2     | 11      | Viewed  |
| 1     | 2       | Clicked |
+-------+---------+---------+
结果表:
+-------+-------+
| ad_id | ctr   |
+-------+-------+
| 1     | 66.67 |
| 3     | 50.00 |
| 2     | 33.33 |
| 5     | 0.00  |
+-------+-------+
对于 ad_id = 1, ctr = (2/(2+1)) * 100 = 66.67
对于 ad_id = 2, ctr = (1/(1+2)) * 100 = 33.33
对于 ad_id = 3, ctr = (1/(1+1)) * 100 = 50.00
对于 ad_id = 5, ctr = 0.00, 注意 ad_id = 5 没有被点击 (Clicked) 或查看 (Viewed) 过
注意我们不关心 action 为 Ingnored 的广告
结果按 ctr（降序），ad_id（升序）排序
```
```
--简单题注意考察对null的处理
select
ad_id,ifnull(round(sum(action="Clicked") /(sum(action <> "Ignored")) *100,2),0.00) ctr
from 
Ads
group by ad_id
order by ctr desc ,ad_id 
```
