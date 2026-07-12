##71.即时事物配送2
>写一条 SQL 查询语句获取即时订单在所有用户的首次订单中的比例。保留两位小数。
```
配送表: Delivery

+-----------------------------+---------+
| Column Name                 | Type    |
+-----------------------------+---------+
| delivery_id                 | int     |
| customer_id                 | int     |
| order_date                  | date    |
| customer_pref_delivery_date | date    |
+-----------------------------+---------+
delivery_id 是表的主键。
该表保存着顾客的食物配送信息，顾客在某个日期下了订单，并指定了一个期望的配送日期（和下单日期相同或者在那之后）。
 

如果顾客期望的配送日期和下单日期相同，则该订单称为 「即时订单」，否则称为「计划订单」。

「首次订单」是顾客最早创建的订单。我们保证一个顾客只会有一个「首次订单」。

写一条 SQL 查询语句获取即时订单在所有用户的首次订单中的比例。保留两位小数。

 

查询结果如下所示：

Delivery 表：
+-------------+-------------+------------+-----------------------------+
| delivery_id | customer_id | order_date | customer_pref_delivery_date |
+-------------+-------------+------------+-----------------------------+
| 1           | 1           | 2019-08-01 | 2019-08-02                  |
| 2           | 2           | 2019-08-02 | 2019-08-02                  |
| 3           | 1           | 2019-08-11 | 2019-08-12                  |
| 4           | 3           | 2019-08-24 | 2019-08-24                  |
| 5           | 3           | 2019-08-21 | 2019-08-22                  |
| 6           | 2           | 2019-08-11 | 2019-08-13                  |
| 7           | 4           | 2019-08-09 | 2019-08-09                  |
+-------------+-------------+------------+-----------------------------+

Result 表：
+----------------------+
| immediate_percentage |
+----------------------+
| 50.00                |
+----------------------+
1 号顾客的 1 号订单是首次订单，并且是计划订单。
2 号顾客的 2 号订单是首次订单，并且是即时订单。
3 号顾客的 5 号订单是首次订单，并且是计划订单。
4 号顾客的 7 号订单是首次订单，并且是即时订单。
因此，一半顾客的首次订单是即时的
```
```
--我的写法,考虑可能有的用户没有首次购买时间?,可以优化一下,把获取总的首次购买的总数直接count(1) 而不是用子查询获取

select 
round((count(1) / (select count(distinct customer_id) from Delivery)) *100,2) as immediate_percentage
from 
Delivery
where order_date = customer_pref_delivery_date
and (customer_id,order_date) in 
(
select
customer_id,min(order_date) as da
from
Delivery
group by customer_id
)
```
```
select round(sum(order_date=customer_pref_delivery_date)/count(*)*100,2) immediate_percentage
from(
    select order_date,customer_pref_delivery_date,
           rank() over (partition by customer_id order by order_date) r
    from Delivery
) a
where r=1
```
##72.重新格式化部门表
>编写一个 SQL 查询来重新格式化表，使得新的表中有一个部门 id 列和一些对应 每个月 的收入（revenue）列。
```
部门表 Department：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| revenue       | int     |
| month         | varchar |
+---------------+---------+
(id, month) 是表的联合主键。
这个表格有关于每个部门每月收入的信息。
月份（month）可以取下列值 ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"]。
 

编写一个 SQL 查询来重新格式化表，使得新的表中有一个部门 id 列和一些对应 每个月 的收入（revenue）列。

查询结果格式如下面的示例所示：

Department 表：
+------+---------+-------+
| id   | revenue | month |
+------+---------+-------+
| 1    | 8000    | Jan   |
| 2    | 9000    | Jan   |
| 3    | 10000   | Feb   |
| 1    | 7000    | Feb   |
| 1    | 6000    | Mar   |
+------+---------+-------+

查询得到的结果表：
+------+-------------+-------------+-------------+-----+-------------+
| id   | Jan_Revenue | Feb_Revenue | Mar_Revenue | ... | Dec_Revenue |
+------+-------------+-------------+-------------+-----+-------------+
| 1    | 8000        | 7000        | 6000        | ... | null        |
| 2    | 9000        | null        | null        | ... | null        |
| 3    | null        | 10000       | null        | ... | null        |
+------+-------------+-------------+-------------+-----+-------------+

注意，结果表有 13 列 (1个部门 id 列 + 12个月份的收入列)。
```
```
-- 简单题,考察group by 和聚合函数
select
id,
sum(if(month = "Jan",revenue,null)) as Jan_Revenue,
sum(if(month = "Feb",revenue,null)) as Feb_Revenue,
sum(if(month = "Mar",revenue,null)) as Mar_Revenue,
sum(if(month = "Apr",revenue,null)) as Apr_Revenue,
sum(if(month = "May",revenue,null)) as May_Revenue,
sum(if(month = "Jun",revenue,null)) as Jun_Revenue,
sum(if(month = "Jul",revenue,null)) as Jul_Revenue,
sum(if(month = "Aug",revenue,null)) as Aug_Revenue,
sum(if(month = "Sep",revenue,null)) as Sep_Revenue,
sum(if(month = "Oct",revenue,null)) as Oct_Revenue,
sum(if(month = "Nov",revenue,null)) as Nov_Revenue,
sum(if(month = "Dec",revenue,null)) as Dec_Revenue
from
Department
group by id
```
```
select 
     id
    , sum(case `month` when 'Jan' then revenue else null end) as Jan_Revenue
    , sum(case `month` when 'Feb' then revenue else null end) as Feb_Revenue
    , sum(case `month` when 'Mar' then revenue else null end) as Mar_Revenue
    , sum(case `month` when 'Apr' then revenue else null end) as Apr_Revenue
    , sum(case `month` when 'May' then revenue else null end) as May_Revenue
    , sum(case `month` when 'Jun' then revenue else null end) as Jun_Revenue
    , sum(case `month` when 'Jul' then revenue else null end) as Jul_Revenue
    , sum(case `month` when 'Aug' then revenue else null end) as Aug_Revenue
    , sum(case `month` when 'Sep' then revenue else null end) as Sep_Revenue
    , sum(case `month` when 'Oct' then revenue else null end) as Oct_Revenue
    , sum(case `month` when 'Nov' then revenue else null end) as Nov_Revenue
    , sum(case `month` when 'Dec' then revenue else null end) as Dec_Revenue
from Department group by id
```
##73.每月交易1
>编写一个 sql 查询来查找每个月和每个国家/地区的事务数及其总金额、已批准的事务数及其总金额。
```
Table: Transactions

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| country       | varchar |
| state         | enum    |
| amount        | int     |
| trans_date    | date    |
+---------------+---------+
id 是这个表的主键。
该表包含有关传入事务的信息。
state 列类型为 “[”批准“，”拒绝“] 之一。
 

编写一个 sql 查询来查找每个月和每个国家/地区的事务数及其总金额、已批准的事务数及其总金额。

查询结果格式如下所示：

Transactions table:
+------+---------+----------+--------+------------+
| id   | country | state    | amount | trans_date |
+------+---------+----------+--------+------------+
| 121  | US      | approved | 1000   | 2018-12-18 |
| 122  | US      | declined | 2000   | 2018-12-19 |
| 123  | US      | approved | 2000   | 2019-01-01 |
| 124  | DE      | approved | 2000   | 2019-01-07 |
+------+---------+----------+--------+------------+

Result table:
+----------+---------+-------------+----------------+--------------------+-----------------------+
| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
+----------+---------+-------------+----------------+--------------------+-----------------------+
| 2018-12  | US      | 2           | 1              | 3000               | 1000                  |
| 2019-01  | US      | 1           | 1              | 2000               | 2000                  |
| 2019-01  | DE      | 1           | 1              | 2000               | 2000                  |
+----------+---------+-------------+----------------+--------------------+-----------------------+
```
```
-- 简单题,可以使用date_format或者left函数
select
substr(trans_date,1,7) month , country,count(state) trans_count,
sum(state = "approved") approved_count ,sum(amount) trans_total_amount,
sum(if(state = "approved",amount,0)) approved_total_amount
from
Transactions
group by country,substr(trans_date,1,7)
```
```
select date_format(trans_date,"%Y-%m") month
,country,count(id) trans_count
,count(if(state<>"approved",null,state)) approved_count
,sum(amount) trans_total_amount
,sum(amount*(state<>"declined"))  approved_total_amount
from transactions 
group by date_format(trans_date,"%Y-%m"),country;
```
注意在group by之后使用count(字段=?) 统计的是全部的行,如果要过滤要使用case when  或者 if 来过滤例如:
```
count(if(state<>"approved",null,state)) approved_count
```
##74.锦标赛优胜者
>编写一个 SQL 查询来查找每组中的获胜者。
```
Players 玩家表

+-------------+-------+
| Column Name | Type  |
+-------------+-------+
| player_id   | int   |
| group_id    | int   |
+-------------+-------+
player_id 是此表的主键。
此表的每一行表示每个玩家的组。
Matches 赛事表

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| match_id      | int     |
| first_player  | int     |
| second_player | int     | 
| first_score   | int     |
| second_score  | int     |
+---------------+---------+
match_id 是此表的主键。
每一行是一场比赛的记录，first_player 和 second_player 表示该场比赛的球员 ID。
first_score 和 second_score 分别表示 first_player 和 second_player 的得分。
你可以假设，在每一场比赛中，球员都属于同一组。
 

每组的获胜者是在组内累积得分最高的选手。如果平局，player_id 最小 的选手获胜。

编写一个 SQL 查询来查找每组中的获胜者。

查询结果格式如下所示

Players 表:
+-----------+------------+
| player_id | group_id   |
+-----------+------------+
| 15        | 1          |
| 25        | 1          |
| 30        | 1          |
| 45        | 1          |
| 10        | 2          |
| 35        | 2          |
| 50        | 2          |
| 20        | 3          |
| 40        | 3          |
+-----------+------------+

Matches 表:
+------------+--------------+---------------+-------------+--------------+
| match_id   | first_player | second_player | first_score | second_score |
+------------+--------------+---------------+-------------+--------------+
| 1          | 15           | 45            | 3           | 0            |
| 2          | 30           | 25            | 1           | 2            |
| 3          | 30           | 15            | 2           | 0            |
| 4          | 40           | 20            | 5           | 2            |
| 5          | 35           | 50            | 1           | 1            |
+------------+--------------+---------------+-------------+--------------+

Result 表:
+-----------+------------+
| group_id  | player_id  |
+-----------+------------+ 
| 1         | 15         |
| 2         | 35         |
| 3         | 40         |
+-----------+------------+
```
```
--我的写法先获取每个人的分数,然后在组内比较获取最大值
select
group_id,player_id 
from 
(
select
group_id,player_id,row_number() over( partition by group_id order by score desc ,player_id asc ) as rn 
from 
Players
join 
(
select 
player,sum(score) as score
from 
(
select
first_player as player ,sum(first_score) as score
from
Matches
group by first_player
union all 
select 
second_player ,sum(second_score)
from
Matches
group by second_player
)
group by player
) t2
on Players.player_id = t2.player
) 
where rn =1
```
```
-- 使用first_value可以减少嵌套一层select 
SELECT DISTINCT group_id,  
       FIRST_VALUE(player_id) OVER(PARTITION BY group_id ORDER BY score DESC, player_id) AS player_id
FROM
    (SELECT p.group_id,
            p.player_id,
            SUM(score)AS score
    FROM
        (SELECT first_player AS player_id, first_score AS score
        FROM Matches
        UNION ALL
        SELECT second_player AS player_id, second_score AS score
        FROM Matches) t1
    LEFT JOIN 
        Players p 
    ON t1.player_id = p.player_id
    GROUP BY player_id) t2
```
```
-- 偷懒写法,group by 默认返回第一行
select group_id, player_id
from (
    select players.*, sum(if(player_id = first_player, first_score, second_score)) score
    from players join matches
    on player_id = first_player or player_id = second_player
    group by player_id
    order by score desc, player_id
) tmp
group by group_id
```
##75.最后一个能进入电梯的人(自连接)
>写一条 SQL 查询语句查找最后一个能进入电梯且不超过重量限制的 person_name 。题目确保队列中第一位的人可以进入电梯 。
```
表: Queue

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| person_id   | int     |
| person_name | varchar |
| weight      | int     |
| turn        | int     |
+-------------+---------+
person_id 是这个表的主键。
该表展示了所有等待电梯的人的信息。
表中 person_id 和 turn 列将包含从 1 到 n 的所有数字，其中 n 是表中的行数。
 

电梯最大载重量为 1000。

写一条 SQL 查询语句查找最后一个能进入电梯且不超过重量限制的 person_name 。题目确保队列中第一位的人可以进入电梯 。

查询结果如下所示 :

Queue 表
+-----------+-------------------+--------+------+
| person_id | person_name       | weight | turn |
+-----------+-------------------+--------+------+
| 5         | George Washington | 250    | 1    |
| 3         | John Adams        | 350    | 2    |
| 6         | Thomas Jefferson  | 400    | 3    |
| 2         | Will Johnliams    | 200    | 4    |
| 4         | Thomas Jefferson  | 175    | 5    |
| 1         | James Elephant    | 500    | 6    |
+-----------+-------------------+--------+------+

Result 表
+-------------------+
| person_name       |
+-------------------+
| Thomas Jefferson  |
+-------------------+

为了简化，Queue 表按 turn 列由小到大排序。
上例中 George Washington(id 5), John Adams(id 3) 和 Thomas Jefferson(id 6) 将可以进入电梯,因为他们的体重和为 250 + 350 + 400 = 1000。
Thomas Jefferson(id 6) 是最后一个体重合适并进入电梯的人。
```
```
--我的写法,窗口写法
select 
person_name
from 
(
select
person_name,rank() over(order by weight_rn desc )  as rn 
from 
(
select
person_name,
sum(weight) over(order by turn ) as weight_rn  
from
Queue
) 
where weight_rn <= 1000
) 
where rn =1
```
```
--自连接,where t1.turn >= t2.turn计算直到当前的总的质量
select t1.person_name
from queue t1, queue t2
where t1.turn >= t2.turn
group by t1.turn 
having sum(t2.weight) <= 1000
order by t1.turn desc
limit 1
```
##76.每月交易2(tag)☆
>编写一个 SQL 查询，以查找每个月和每个国家/地区的已批准交易的数量及其总金额、退单的数量及其总金额。
注意：在您的查询中，给定月份和国家，忽略所有为零的行。
```
Transactions 记录表

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| id             | int     |
| country        | varchar |
| state          | enum    |
| amount         | int     |
| trans_date     | date    |
+----------------+---------+
id 是这个表的主键。
该表包含有关传入事务的信息。
状态列是类型为 [approved（已批准）、declined（已拒绝）] 的枚举。
 

Chargebacks 表

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| trans_id       | int     |
| charge_date    | date    |
+----------------+---------+
退单包含有关放置在事务表中的某些事务的传入退单的基本信息。
trans_id 是 transactions 表的 id 列的外键。
每项退单都对应于之前进行的交易，即使未经批准。
 

编写一个 SQL 查询，以查找每个月和每个国家/地区的已批准交易的数量及其总金额、退单的数量及其总金额。

注意：在您的查询中，给定月份和国家，忽略所有为零的行。

查询结果格式如下所示：

Transactions 表：
+------+---------+----------+--------+------------+
| id   | country | state    | amount | trans_date |
+------+---------+----------+--------+------------+
| 101  | US      | approved | 1000   | 2019-05-18 |
| 102  | US      | declined | 2000   | 2019-05-19 |
| 103  | US      | approved | 3000   | 2019-06-10 |
| 104  | US      | declined | 4000   | 2019-06-13 |
| 105  | US      | approved | 5000   | 2019-06-15 |
+------+---------+----------+--------+------------+

Chargebacks 表：
+------------+------------+
| trans_id   | trans_date |
+------------+------------+
| 102        | 2019-05-29 |
| 101        | 2019-06-30 |
| 105        | 2019-09-18 |
+------------+------------+

Result 表：
+----------+---------+----------------+-----------------+-------------------+--------------------+
| month    | country | approved_count | approved_amount | chargeback_count  | chargeback_amount  |
+----------+---------+----------------+-----------------+-------------------+--------------------+
| 2019-05  | US      | 1              | 1000            | 1                 | 2000               |
| 2019-06  | US      | 2              | 8000            | 1                 | 1000               |
| 2019-09  | US      | 0              | 0               | 1                 | 5000               |
+----------+---------+----------------+-----------------+-------------------+------------------
```
```
-- 总少了一个国家,我裂开了.....没有full join难受
select 
t1.dt as month ,t1.country,approved_count,approved_amount,chargeback_count,chargeback_amount
from 
(
select
country,left(trans_date,7) as dt ,ifnull(sum(state ="approved"),0) as approved_count,
ifnull(sum(if(state ="approved",amount,0)),0) as approved_amount
from
Transactions
group by country, left(trans_date,7)
) t1 
left   join 
(
select
country,left(Chargebacks.trans_date,7) as dt ,ifnull(count(1),0) as chargeback_count ,ifnull(sum(amount),0) as chargeback_amount
from 
Transactions
join Chargebacks on Transactions.id = Chargebacks.trans_id and left(Chargebacks.trans_date,7) = left( Transactions.trans_date,7)
group by country,left(Chargebacks.trans_date,7)
) t2
on t1.country = t2.country and t1.dt = t2.dt
```
```
--学习使用tag用法,妙!!!!
select
    date_format(trans_date, '%Y-%m') as month,
    country,
    sum(if(state = 'approved',1,0)) as approved_count,
    sum(if(state = 'approved',amount,0)) as approved_amount,
    sum(if(state = 'chargeback',1,0)) as chargeback_count,
    sum(if(state = 'chargeback',amount,0)) as chargeback_amount
from
    (select
        *
    from
        transactions
    union all
    select
        trans_id as id,
        country,
        'chargeback' as state,
        amount,
        c.trans_date
    from
        chargebacks c left join transactions t 
        on t.id = c.trans_id) as temp
group by 1,2
having approved_count <> 0 or chargeback_count <> 0
```
防止为0
```
having approved_count <> 0 or chargeback_count <> 0
```
##77.查询结果的质量与占比
>编写一组 SQL 来查找每次查询的名称(query_name)、质量(quality) 和 劣质查询百分比(poor_query_percentage)。
质量(quality) 和劣质查询百分比(poor_query_percentage) 都应四舍五入到小数点后两位。

```
查询表 Queries： 

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| query_name  | varchar |
| result      | varchar |
| position    | int     |
| rating      | int     |
+-------------+---------+
此表没有主键，并可能有重复的行。
此表包含了一些从数据库中收集的查询信息。
“位置”（position）列的值为 1 到 500 。
“评分”（rating）列的值为 1 到 5 。评分小于 3 的查询被定义为质量很差的查询。
 

将查询结果的质量 quality 定义为：

各查询结果的评分与其位置之间比率的平均值。

将劣质查询百分比 poor_query_percentage 为：

评分小于 3 的查询结果占全部查询结果的百分比。

编写一组 SQL 来查找每次查询的名称(query_name)、质量(quality) 和 劣质查询百分比(poor_query_percentage)。

质量(quality) 和劣质查询百分比(poor_query_percentage) 都应四舍五入到小数点后两位。

查询结果格式如下所示：

Queries table:
+------------+-------------------+----------+--------+
| query_name | result            | position | rating |
+------------+-------------------+----------+--------+
| Dog        | Golden Retriever  | 1        | 5      |
| Dog        | German Shepherd   | 2        | 5      |
| Dog        | Mule              | 200      | 1      |
| Cat        | Shirazi           | 5        | 2      |
| Cat        | Siamese           | 3        | 3      |
| Cat        | Sphynx            | 7        | 4      |
+------------+-------------------+----------+--------+

Result table:
+------------+---------+-----------------------+
| query_name | quality | poor_query_percentage |
+------------+---------+-----------------------+
| Dog        | 2.50    | 33.33                 |
| Cat        | 0.66    | 33.33                 |
+------------+---------+-----------------------+

Dog 查询结果的质量为 ((5 / 1) + (5 / 2) + (1 / 200)) / 3 = 2.50
Dog 查询结果的劣质查询百分比为 (1 / 3) * 100 = 33.33

Cat 查询结果的质量为 ((2 / 5) + (3 / 3) + (4 / 7)) / 3 = 0.66
Cat 查询结果的劣质查询百分比为 (1 / 3) * 100 = 33.33
```
```
--读懂题意,简单题
select 
query_name,round(sum(rating/position) /count(1),2) as quality,
round(sum(rating < 3) /count(1) *100 ,2)as poor_query_percentage
from
Queries
group by query_name
```
```
SELECT 
    query_name, 
    ROUND(AVG(rating/position), 2) quality,
    ROUND(avg(rating < 3) * 100,2) poor_query_percentage
FROM Queries
GROUP BY query_name
```
##78.查询球队积分
>写出一条SQL语句以查询每个队的 team_id，team_name 和 num_points。结果根据 num_points 降序排序，如果有两队积分相同，那么这两队按 team_id  升序排序

```
Table: Teams

+---------------+----------+
| Column Name   | Type     |
+---------------+----------+
| team_id       | int      |
| team_name     | varchar  |
+---------------+----------+
此表的主键是 team_id，表中的每一行都代表一支独立足球队。
Table: Matches

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| match_id      | int     |
| host_team     | int     |
| guest_team    | int     | 
| host_goals    | int     |
| guest_goals   | int     |
+---------------+---------+
此表的主键是 match_id，表中的每一行都代表一场已结束的比赛，比赛的主客队分别由它们自己的 id 表示，他们的进球由 host_goals 和 guest_goals 分别表示。
 

积分规则如下：

赢一场得三分；
平一场得一分；
输一场不得分。
写出一条SQL语句以查询每个队的 team_id，team_name 和 num_points。结果根据 num_points 降序排序，如果有两队积分相同，那么这两队按 team_id  升序排序。

查询结果格式如下：

Teams table:
+-----------+--------------+
| team_id   | team_name    |
+-----------+--------------+
| 10        | Leetcode FC  |
| 20        | NewYork FC   |
| 30        | Atlanta FC   |
| 40        | Chicago FC   |
| 50        | Toronto FC   |
+-----------+--------------+

Matches table:
+------------+--------------+---------------+-------------+--------------+
| match_id   | host_team    | guest_team    | host_goals  | guest_goals  |
+------------+--------------+---------------+-------------+--------------+
| 1          | 10           | 20            | 3           | 0            |
| 2          | 30           | 10            | 2           | 2            |
| 3          | 10           | 50            | 5           | 1            |
| 4          | 20           | 30            | 1           | 0            |
| 5          | 50           | 30            | 1           | 0            |
+------------+--------------+---------------+-------------+--------------+

Result table:
+------------+--------------+---------------+
| team_id    | team_name    | num_points    |
+------------+--------------+---------------+
| 10         | Leetcode FC  | 7             |
| 20         | NewYork FC   | 3             |
| 50         | Toronto FC   | 3             |
| 30         | Atlanta FC   | 1             |
| 40         | Chicago FC   | 0             |
+------------+--------------+---------------+
```
```
--我的写法,读懂题意
select
Teams.team_id team_id,team_name, ifnull(num_points,0) as num_points 
from 
Teams left join 
(
select 
team_id,sum(points) as num_points 
from 
(
select 
host_team as team_id,sum(case when host_goals > guest_goals  then 3 when host_goals < guest_goals then 0 else 1 end ) as points 
from
Matches
group by host_team
union all 
select 
guest_team  ,sum(case when host_goals > guest_goals  then 0 when host_goals < guest_goals then 3 else 1 end ) 
from
Matches
group by guest_team 
) t1
group by  team_id
) t2 
on Teams.team_id = t2.team_id
order by num_points desc  ,team_id 
```
```
--评论区写法
select t.team_id,t.team_name,
        ifnull(sum(case when m.host_goals>m.guest_goals and t.team_id=m.host_team then 3 
                        when m.host_goals=m.guest_goals and t.team_id=m.host_team then 1
                        when m.host_goals<m.guest_goals and t.team_id=m.host_team then 0
                        when m.host_goals>m.guest_goals and t.team_id=m.guest_team then 0
                        when m.host_goals=m.guest_goals and t.team_id=m.guest_team then 1
                        when m.host_goals<m.guest_goals and t.team_id=m.guest_team then 3
                   end),0) as num_points
from Teams t,Matches m
group by t.team_id
order by num_points desc,t.team_id asc
```
##79.报告系统状态的连续日期
>编写一个 SQL 查询 2019-01-01 到 2019-12-31 期间任务连续同状态 period_state 的起止日期（start_date 和 end_date）。即如果任务失败了，就是失败状态的起止日期，如果任务成功了，就是成功状态的起止日期。
```
Table: Failed

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| fail_date    | date    |
+--------------+---------+
该表主键为 fail_date。
该表包含失败任务的天数.
Table: Succeeded

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| success_date | date    |
+--------------+---------+
该表主键为 success_date。
该表包含成功任务的天数.
 

系统 每天 运行一个任务。每个任务都独立于先前的任务。任务的状态可以是失败或是成功。

编写一个 SQL 查询 2019-01-01 到 2019-12-31 期间任务连续同状态 period_state 的起止日期（start_date 和 end_date）。即如果任务失败了，就是失败状态的起止日期，如果任务成功了，就是成功状态的起止日期。

最后结果按照起始日期 start_date 排序

查询结果样例如下所示:

Failed table:
+-------------------+
| fail_date         |
+-------------------+
| 2018-12-28        |
| 2018-12-29        |
| 2019-01-04        |
| 2019-01-05        |
+-------------------+

Succeeded table:
+-------------------+
| success_date      |
+-------------------+
| 2018-12-30        |
| 2018-12-31        |
| 2019-01-01        |
| 2019-01-02        |
| 2019-01-03        |
| 2019-01-06        |
+-------------------+


Result table:
+--------------+--------------+--------------+
| period_state | start_date   | end_date     |
+--------------+--------------+--------------+
| succeeded    | 2019-01-01   | 2019-01-03   |
| failed       | 2019-01-04   | 2019-01-05   |
| succeeded    | 2019-01-06   | 2019-01-06   |
+--------------+--------------+--------------+

结果忽略了 2018 年的记录，因为我们只关心从 2019-01-01 到 2019-12-31 的记录
从 2019-01-01 到 2019-01-03 所有任务成功，系统状态为 "succeeded"。
从 2019-01-04 到 2019-01-05 所有任务失败，系统状态为 "failed"。
从 2019-01-06 到 2019-01-06 所有任务成功，系统状态为 "succeeded"。
```

```
--终于跑对了,Oracle真麻烦
select
'succeeded' as period_state ,to_char(min(success_date),'yyyy-mm-dd') as start_date,
to_char(max(success_date),'yyyy-mm-dd') as end_date
from 
(
select
success_date,
(success_date - row_number() over(order by success_date) ) rn 
from
Succeeded
where to_char(success_date,'yyyy-mm-dd') >= '2019-01-01'  and to_char(success_date,'yyyy-mm-dd') <=  '2019-12-31'
)
group by rn 
union all 
select
'failed',to_char(min(fail_date),'yyyy-mm-dd'),to_char(max(fail_date),'yyyy-mm-dd')
from 
(
select
fail_date,
( fail_date   - row_number() over(order by  fail_date  ) ) rn 
from
Failed
where to_char(fail_date,'yyyy-mm-dd') between '2019-01-01'  and  '2019-12-31'
) 
group by rn 
order by start_date
```
```
-- 整体求,注意不要排序简单的连续登陆案例变形题
select state as period_state,min(dt) as start_date,max(dt) as end_date 
from
(
    select *,subdate(dt,rank() over(partition by state order by dt)) as dif 
    from
    (
        select 'failed' as state,fail_date as dt from failed where fail_date between '2019-01-01' and '2019-12-31'
    union all
        select 'succeeded' as state,success_date as dt from succeeded where success_date between '2019-01-01' and '2019-12-31'
     )t1 
)t2
group by state,dif
order by dt
```
##80.每个帖子的评论数
>编写 SQL 语句以查找每个帖子的评论数。
结果表应包含帖子的 post_id 和对应的评论数 number_of_comments 并且按 post_id 升序排列。
Submissions 可能包含重复的评论。您应该计算每个帖子的唯一评论数。
Submissions 可能包含重复的帖子。您应该将它们视为一个帖子。

```
表 Submissions 结构如下：

+---------------+----------+
| 列名           | 类型     |
+---------------+----------+
| sub_id        | int      |
| parent_id     | int      |
+---------------+----------+
上表没有主键, 所以可能会出现重复的行。
每行可以是一个帖子或对该帖子的评论。
如果是帖子的话，parent_id 就是 null。
对于评论来说，parent_id 就是表中对应帖子的 sub_id。
 

编写 SQL 语句以查找每个帖子的评论数。

结果表应包含帖子的 post_id 和对应的评论数 number_of_comments 并且按 post_id 升序排列。

Submissions 可能包含重复的评论。您应该计算每个帖子的唯一评论数。

Submissions 可能包含重复的帖子。您应该将它们视为一个帖子。

查询结果格式如下例所示：

Submissions table:
+---------+------------+
| sub_id  | parent_id  |
+---------+------------+
| 1       | Null       |
| 2       | Null       |
| 1       | Null       |
| 12      | Null       |
| 3       | 1          |
| 5       | 2          |
| 3       | 1          |
| 4       | 1          |
| 9       | 1          |
| 10      | 2          |
| 6       | 7          |
+---------+------------+

结果表：
+---------+--------------------+
| post_id | number_of_comments |
+---------+--------------------+
| 1       | 3                  |
| 2       | 2                  |
| 12      | 0                  |
+---------+--------------------+

表中 ID 为 1 的帖子有 ID 为 3、4 和 9 的三个评论。表中 ID 为 3 的评论重复出现了，所以我们只对它进行了一次计数。
表中 ID 为 2 的帖子有 ID 为 5 和 10 的两个评论。
ID 为 12 的帖子在表中没有评论。
表中 ID 为 6 的评论是对 ID 为 7 的已删除帖子的评论，因此我们将其忽略。
```
```
-- 简单题
select
a.sub_id as post_id ,ifnull(count(distinct b.sub_id ),0) as  number_of_comments
from 
(
select
distinct sub_id as sub_id
from
Submissions
where parent_id is null
) a 
left join 
Submissions b
on a.sub_id = b.parent_id
group by a.sub_id
```
```
select s1.sub_id post_id, count(distinct s2.sub_id) number_of_comments
from 
submissions s1 left join submissions s2 on s1.sub_id = s2.parent_id
where s1.parent_id is null
group by s1.sub_id
order by 1
```
一个基础题left join on 和 left join where 的区别,left join on 无论是否满足都会返回左边的数据
>https://www.cnblogs.com/leochenliang/p/7364665.html
