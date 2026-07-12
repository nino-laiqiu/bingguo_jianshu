##181.多条件查找having 与where联合字段☆
```
Table: Product

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| product_id   | int     |
| product_name | varchar |
| unit_price   | int     |
+--------------+---------+
product_id 是这张表的主键
Table: Sales

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
编写一个 SQL 查询，查询购买了 S8 手机却没有购买 iPhone 的买家。注意这里 S8 和 iPhone 是 Product 表中的产品。

查询结果格式如下图表示：

Product table:
+------------+--------------+------------+
| product_id | product_name | unit_price |
+------------+--------------+------------+
| 1          | S8           | 1000       |
| 2          | G4           | 800        |
| 3          | iPhone       | 1400       |
+------------+--------------+------------+

Sales table:
+-----------+------------+----------+------------+----------+-------+
| seller_id | product_id | buyer_id | sale_date  | quantity | price |
+-----------+------------+----------+------------+----------+-------+
| 1         | 1          | 1        | 2019-01-21 | 2        | 2000  |
| 1         | 2          | 2        | 2019-02-17 | 1        | 800   |
| 2         | 1          | 3        | 2019-06-02 | 1        | 800   |
| 3         | 3          | 3        | 2019-05-13 | 2        | 2800  |
+-----------+------------+----------+------------+----------+-------+

Result table:
+-------------+
| buyer_id    |
+-------------+
| 1           |
+-------------+
id 为 1 的买家购买了一部 S8，但是却没有购买 iPhone，而 id 为 3 的买家却同时购买了这 2 部手机。
```
这是错误的写法.....我又写错啦....标注一下
```
select
buyer_id 
from
Sales
where
product_id in (select product_id from Product where product_name ="S8" )  
and 
product_id  in (select product_id from Product where product_name !="iPhone" ) 
group by buyer_id 
```
要写应该这样,这个叫粒度的问题...
```
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
最佳方法
```
select
buyer_id
from 
Sales t1
join Product t2
on t1.product_id = t2.product_id
group by buyer_id
having sum(product_name = 'S8') >0 and sum(product_name = 'iPhone') =0
```
还有注意where联合字段子查询来过滤条件,题目是leedcode产品销售分析3
```
-- where联合字段与子查询
SELECT PRODUCT_ID, YEAR AS FIRST_YEAR, QUANTITY, PRICE
FROM SALES
WHERE (PRODUCT_ID, YEAR) IN (SELECT PRODUCT_ID, MIN(YEAR)
                             FROM SALES
                             GROUP BY 1);
```
##182.指标问题
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
每一行表示一个玩家的记录，在某一天使用某个设备注销之前，登录并玩了很多游戏（可能是 0）
 

玩家的 安装日期 定义为该玩家的第一个登录日。

玩家的 第一天留存率 定义为：假定安装日期为 X 的玩家的数量为 N ，其中在 X 之后的某一天重新登录的玩家数量为 M ，M/N 就是第一天留存率，四舍五入到小数点后两位。

编写一个 SQL 查询，报告所有安装日期、当天安装游戏的玩家数量和玩家的第一天留存率。

 

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
玩家 1 和 3 在 2016-03-01 安装了游戏，但只有玩家 1 在 2016-03-02 重新登录，所以 2016-03-01 的第一天留存率是 1/2=0.50
玩家 2 在 2017-06-25 安装了游戏，但在 2017-06-26 没有重新登录，因此 2017-06-25 的第一天留存率为 0/1=0.00
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
--注意min/max窗口在求指标时的使用
下面的思路就是先获取每个用户第一次登陆的时间,然后与原始表进行比较时间是否存在时间+1的情况
```
select first_date as install_dt,
count(distinct player_id) as installs,
round(sum(if(datediff(event_date,first_date)=1,1,0))/count(distinct player_id),2)
as Day1_retention
from
(
    select player_id,event_date,min(event_date) over(partition by player_id) as first_date
    from activity
)tmp
group by first_date
```
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
每一行是一个玩家的记录，他在某一天使用某个设备注销之前登录并玩了很多游戏（可能是 0）。
 

编写一个 SQL 查询，报告在首次登录的第二天再次登录的玩家的比率，四舍五入到小数点后两位。换句话说，您需要计算从首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数。

查询结果格式如下所示：

Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+-----------+
| fraction  |
+-----------+
| 0.33      |
+-----------+
只有 ID 为 1 的玩家在第一天登录后才重新登录，所以答案是 1/3 = 0.33
```
```
select 
round((count(b.player_id) / count(a.player_id) ),2) as   fraction 
from 
(
select 
player_id,min(event_date) as event_date
from
Activity
group by player_id
) a 
left join Activity b 
on a.player_id = b.player_id and datediff(b.event_date,a.event_date) =1
```
```
select
round(sum(if(datediff(event_date,rn)=1,1,0)) /  count(distinct player_id) ,2)  fraction
from 
(
select
player_id,event_date,min(event_date) over(partition by player_id) rn
from 
Activity 
) t1
```
```
select
round(sum(if(datediff(event_date,rn)=-1,1,0)) /  count(distinct player_id) ,2)  fraction 
from 
(
select
player_id,event_date,
lead(event_date) over(partition by player_id order by event_date asc) rn,
row_number() over(partition by player_id order by event_date) rn_1
from 
Activity
) t1
where rn_1 =1
```
##183.构建辅助表
```
支出表: Spending

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| user_id     | int     |
| spend_date  | date    |
| platform    | enum    | 
| amount      | int     |
+-------------+---------+
这张表记录了用户在一个在线购物网站的支出历史，该在线购物平台同时拥有桌面端（'desktop'）和手机端（'mobile'）的应用程序。
这张表的主键是 (user_id, spend_date, platform)。
平台列 platform 是一种 ENUM ，类型为（'desktop', 'mobile'）。
 

写一段 SQL 来查找每天 仅 使用手机端用户、仅 使用桌面端用户和 同时 使用桌面端和手机端的用户人数和总支出金额。

查询结果格式如下例所示：

Spending table:
+---------+------------+----------+--------+
| user_id | spend_date | platform | amount |
+---------+------------+----------+--------+
| 1       | 2019-07-01 | mobile   | 100    |
| 1       | 2019-07-01 | desktop  | 100    |
| 2       | 2019-07-01 | mobile   | 100    |
| 2       | 2019-07-02 | mobile   | 100    |
| 3       | 2019-07-01 | desktop  | 100    |
| 3       | 2019-07-02 | desktop  | 100    |
+---------+------------+----------+--------+

Result table:
+------------+----------+--------------+-------------+
| spend_date | platform | total_amount | total_users |
+------------+----------+--------------+-------------+
| 2019-07-01 | desktop  | 100          | 1           |
| 2019-07-01 | mobile   | 100          | 1           |
| 2019-07-01 | both     | 200          | 1           |
| 2019-07-02 | desktop  | 100          | 1           |
| 2019-07-02 | mobile   | 100          | 1           |
| 2019-07-02 | both     | 0            | 0           |
+------------+----------+--------------+-------------+ 
在 2019-07-01, 用户1 同时 使用桌面端和手机端购买, 用户2 仅 使用了手机端购买，而用户3 仅 使用了桌面端购买
在 2019-07-02, 用户2 仅 使用了手机端购买, 用户3 仅 使用了桌面端购买，且没有用户 同时 使用桌面端和手机端购买
```
```
-- 先按用户维度集合,打一个标签,然后对这个标签聚合
select 
t1.spend_date,t1.platform,
ifnull(sum(amount),0)  as total_amount,
ifnull(count(distinct user_id),0) as total_users
from
(
select distinct spend_date, "desktop" platform from Spending
union
select distinct spend_date, "mobile" platform from Spending
union
select distinct spend_date, "both" platform from Spending
) t1
left join
( 
select 
spend_date,user_id,
if(count(distinct platform) =1,platform,"both") as platform,
sum(amount) as amount
from
Spending
group by spend_date, user_id
) t2
on t1.spend_date = t2.spend_date and t1.platform = t2.platform
group by t1.spend_date,t1.platform
```
```
Product 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
+---------------+---------+
product_id 是这张表的主键。
product_name 是产品的名称。
 

Sales 表：

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| product_id          | int     |
| period_start        | date    |
| period_end          | date    |
| average_daily_sales | int     |
+---------------------+---------+
product_id 是这张表的主键。
period_start 和 period_end 是该产品销售期的起始日期和结束日期，且这两个日期包含在销售期内。
average_daily_sales 列存储销售期内该产品的日平均销售额。
 

编写一段 SQL 查询每个产品每年的总销售额，并包含 product_id, product_name 以及 report_year 等信息。

销售年份的日期介于 2018 年到 2020 年之间。你返回的结果需要按 product_id 和 report_year 排序。

查询结果格式如下例所示：

Product table:
+------------+--------------+
| product_id | product_name |
+------------+--------------+
| 1          | LC Phone     |
| 2          | LC T-Shirt   |
| 3          | LC Keychain  |
+------------+--------------+

Sales table:
+------------+--------------+-------------+---------------------+
| product_id | period_start | period_end  | average_daily_sales |
+------------+--------------+-------------+---------------------+
| 1          | 2019-01-25   | 2019-02-28  | 100                 |
| 2          | 2018-12-01   | 2020-01-01  | 10                  |
| 3          | 2019-12-01   | 2020-01-31  | 1                   |
+------------+--------------+-------------+---------------------+

Result table:
+------------+--------------+-------------+--------------+
| product_id | product_name | report_year | total_amount |
+------------+--------------+-------------+--------------+
| 1          | LC Phone     |    2019     | 3500         |
| 2          | LC T-Shirt   |    2018     | 310          |
| 2          | LC T-Shirt   |    2019     | 3650         |
| 2          | LC T-Shirt   |    2020     | 10           |
| 3          | LC Keychain  |    2019     | 31           |
| 3          | LC Keychain  |    2020     | 31           |
+------------+--------------+-------------+--------------+
LC Phone 在 2019-01-25 至 2019-02-28 期间销售，该产品销售时间总计35天。销售总额 35*100 = 3500。
LC T-shirt 在 2018-12-01 至 2020-01-01 期间销售，该产品在2018年、2019年、2020年的销售时间分别是31天、365天、1天，2018年、2019年、2020年的销售总额分别是31*10=310、365*10=3650、1*10=10。
LC Keychain 在 2019-12-01 至 2020-01-31 期间销售，该产品在2019年、2020年的销售时间分别是：31天、31天，2019年、2020年的销售总额分别是31*1=31、31*1=31。
```
```
SELECT
    CAST(p.product_id AS CHAR) product_id,
    p.product_name,
    report_year,
    (DATEDIFF(IF(end_dt>period_end,period_end,end_dt),IF(start_dt>period_start,start_dt,period_start)) + 1) * average_daily_sales total_amount
FROM
    Product p
CROSS JOIN
    (
    SELECT '2018' report_year,'2018-01-01' start_dt,'2018-12-31' end_dt
    UNION ALL
    SELECT '2019' report_year,'2019-01-01' start_dt,'2019-12-31' end_dt
    UNION ALL
    SELECT '2020' report_year,'2020-01-01' start_dt,'2020-12-31' end_dt
    ) y
INNER JOIN
    Sales s
ON p.product_id = s.product_id
WHERE
    period_start<=end_dt AND period_end>=start_dt
ORDER BY
    product_id,
    report_year
```
##184.avg口径
```
动作表： Actions

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| post_id       | int     |
| action_date   | date    |
| action        | enum    |
| extra         | varchar |
+---------------+---------+
这张表没有主键，并有可能存在重复的行。
action 列的类型是 ENUM，可能的值为 ('view', 'like', 'reaction', 'comment', 'report', 'share')。
extra 列拥有一些可选信息，例如：报告理由（a reason for report）或反应类型（a type of reaction）等。
移除表： Removals

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| post_id       | int     |
| remove_date   | date    | 
+---------------+---------+
这张表的主键是 post_id。
这张表的每一行表示一个被移除的帖子，原因可能是由于被举报或被管理员审查。
 

编写一段 SQL 来查找：在被报告为垃圾广告的帖子中，被移除的帖子的每日平均占比，四舍五入到小数点后 2 位。

查询结果的格式如下：

Actions table:
+---------+---------+-------------+--------+--------+
| user_id | post_id | action_date | action | extra  |
+---------+---------+-------------+--------+--------+
| 1       | 1       | 2019-07-01  | view   | null   |
| 1       | 1       | 2019-07-01  | like   | null   |
| 1       | 1       | 2019-07-01  | share  | null   |
| 2       | 2       | 2019-07-04  | view   | null   |
| 2       | 2       | 2019-07-04  | report | spam   |
| 3       | 4       | 2019-07-04  | view   | null   |
| 3       | 4       | 2019-07-04  | report | spam   |
| 4       | 3       | 2019-07-02  | view   | null   |
| 4       | 3       | 2019-07-02  | report | spam   |
| 5       | 2       | 2019-07-03  | view   | null   |
| 5       | 2       | 2019-07-03  | report | racism |
| 5       | 5       | 2019-07-03  | view   | null   |
| 5       | 5       | 2019-07-03  | report | racism |
+---------+---------+-------------+--------+--------+

Removals table:
+---------+-------------+
| post_id | remove_date |
+---------+-------------+
| 2       | 2019-07-20  |
| 3       | 2019-07-18  |
+---------+-------------+

Result table:
+-----------------------+
| average_daily_percent |
+-----------------------+
| 75.00                 |
+-----------------------+
2019-07-04 的垃圾广告移除率是 50%，因为有两张帖子被报告为垃圾广告，但只有一个得到移除。
2019-07-02 的垃圾广告移除率是 100%，因为有一张帖子被举报为垃圾广告并得到移除。
其余几天没有收到垃圾广告的举报，因此平均值为：(50 + 100) / 2 = 75%
注意，输出仅需要一个平均值即可，我们并不关注移除操作的日期。
```
```
select 
round(avg(s1),2)   as average_daily_percent
from 
(
select
action_date,
count(distinct Removals.post_id ) / count(distinct Actions.post_id) * 100 as s1
from
Actions 
left join 
Removals on Actions.post_id = Removals.post_id
where extra = "spam"
group by action_date
) t1
```
两种口径
```
SELECT
    ROUND(COUNT(DISTINCT R.post_id) / COUNT(DISTINCT A.post_id) * 100,2) average_daily_percent
FROM 
    Actions A
LEFT JOIN
    Removals R
ON A.post_id = R.post_id
WHERE
    extra='spam'
```
##185.tag标签
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
+----------+---------+----------------+-----------------+-------------------+--------------------+
```
不是要统计另一张表的退单情况,打一个标签与原始表union all 
```
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
##186.构造0-max
```
表: Visits

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| visit_date    | date    |
+---------------+---------+
(user_id, visit_date) 是该表的主键
该表的每行表示 user_id 在 visit_date 访问了银行
 

表: Transactions

+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| user_id          | int     |
| transaction_date | date    |
| amount           | int     |
+------------------+---------+
该表没有主键，所以可能有重复行
该表的每一行表示 user_id 在 transaction_date 完成了一笔 amount 数额的交易
可以保证用户 (user) 在 transaction_date 访问了银行 (也就是说 Visits 表包含 (user_id, transaction_date) 行)
 

银行想要得到银行客户在一次访问时的交易次数和相应的在一次访问时该交易次数的客户数量的图表

写一条 SQL 查询多少客户访问了银行但没有进行任何交易，多少客户访问了银行进行了一次交易等等

结果包含两列：

transactions_count： 客户在一次访问中的交易次数
visits_count： 在 transactions_count 交易次数下相应的一次访问时的客户数量
transactions_count 的值从 0 到所有用户一次访问中的 max(transactions_count) 

按 transactions_count 排序

下面是查询结果格式的例子：

Visits 表:
+---------+------------+
| user_id | visit_date |
+---------+------------+
| 1       | 2020-01-01 |
| 2       | 2020-01-02 |
| 12      | 2020-01-01 |
| 19      | 2020-01-03 |
| 1       | 2020-01-02 |
| 2       | 2020-01-03 |
| 1       | 2020-01-04 |
| 7       | 2020-01-11 |
| 9       | 2020-01-25 |
| 8       | 2020-01-28 |
+---------+------------+
Transactions 表:
+---------+------------------+--------+
| user_id | transaction_date | amount |
+---------+------------------+--------+
| 1       | 2020-01-02       | 120    |
| 2       | 2020-01-03       | 22     |
| 7       | 2020-01-11       | 232    |
| 1       | 2020-01-04       | 7      |
| 9       | 2020-01-25       | 33     |
| 9       | 2020-01-25       | 66     |
| 8       | 2020-01-28       | 1      |
| 9       | 2020-01-25       | 99     |
+---------+------------------+--------+
结果表:
+--------------------+--------------+
| transactions_count | visits_count |
+--------------------+--------------+
| 0                  | 4            |
| 1                  | 5            |
| 2                  | 0            |
| 3                  | 1            |
+--------------------+--------------+
* 对于 transactions_count = 0, visits 中 (1, "2020-01-01"), (2, "2020-01-02"), (12, "2020-01-01") 和 (19, "2020-01-03") 没有进行交易，所以 visits_count = 4 。
* 对于 transactions_count = 1, visits 中 (2, "2020-01-03"), (7, "2020-01-11"), (8, "2020-01-28"), (1, "2020-01-02") 和 (1, "2020-01-04") 进行了一次交易，所以 visits_count = 5 。
* 对于 transactions_count = 2, 没有客户访问银行进行了两次交易，所以 visits_count = 0 。
* 对于 transactions_count = 3, visits 中 (9, "2020-01-25") 进行了三次交易，所以 visits_count = 1 。
* 对于 transactions_count >= 4, 没有客户访问银行进行了超过3次交易，所以我们停止在 transactions_count = 3 。
```
是一次不是第一次,两表连接,group by 用户和时间,获取时间维度下,用户每次交易次数,这个对交易次数group by,获取这个交易次数下游多少用户,所以这个报表是对时间sum的,这个是难题......以后再写一遍
```
with temp1 as (
    select transactions_count,count(user_id) visits_count
    from (
        select v.user_id,count(t.user_id) transactions_count
        from Visits v
        left join Transactions t
        on v.user_id = t.user_id and v.visit_date = transaction_date
        group by v.user_id,v.visit_date
    ) a
    group by transactions_count
)

select temp2.transactions_count,ifnull(temp1.visits_count,0) visits_count
from (
    select 0 transactions_count
    union
    select row_number() over (order by transaction_date) transactions_count
    from Transactions
) temp2
left join temp1
on temp2.transactions_count = temp1.transactions_count
where temp2.transactions_count <= (
    select max(transactions_count)
    from temp1
)
```
##187.等量代换
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
 

写一个 SQL 查询,  以计算表 Expressions 中的布尔表达式.

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
##188.窗口函数关联
```
表: UserActivity

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| username      | varchar |
| activity      | varchar |
| startDate     | Date    |
| endDate       | Date    |
+---------------+---------+
该表不包含主键
该表包含每个用户在一段时间内进行的活动的信息
名为 username 的用户在 startDate 到 endDate 日内有一次活动
 

写一条SQL查询展示每一位用户 最近第二次 的活动

如果用户仅有一次活动，返回该活动

一个用户不能同时进行超过一项活动，以 任意 顺序返回结果

下面是查询结果格式的例子：

UserActivity 表:
+------------+--------------+-------------+-------------+
| username   | activity     | startDate   | endDate     |
+------------+--------------+-------------+-------------+
| Alice      | Travel       | 2020-02-12  | 2020-02-20  |
| Alice      | Dancing      | 2020-02-21  | 2020-02-23  |
| Alice      | Travel       | 2020-02-24  | 2020-02-28  |
| Bob        | Travel       | 2020-02-11  | 2020-02-18  |
+------------+--------------+-------------+-------------+

Result 表:
+------------+--------------+-------------+-------------+
| username   | activity     | startDate   | endDate     |
+------------+--------------+-------------+-------------+
| Alice      | Dancing      | 2020-02-21  | 2020-02-23  |
| Bob        | Travel       | 2020-02-11  | 2020-02-18  |
+------------+--------------+-------------+-------------+

Alice 最近一次的活动是从 2020-02-24 到 2020-02-28 的旅行, 在此之前的 2020-02-21 到 2020-02-23 她进行了舞蹈
Bob 只有一条记录，我们就取这条记录
```
```
--没想到,采用一个组内全局的count来获取
select username,activity,startDate,endDate
from(
    select *,
        row_number() over (partition by username order by startDate desc) r,
        count(*) over (partition by username) c
    from UserActivity
) a
where c = 1 or r = 2
```
```
select
username, activity,to_char(startDate,'yyyy-mm-dd') startDate,to_char(endDate,'yyyy-mm-dd') endDate 
from 
(
select
username,activity,startDate,endDate,
row_number() over(partition by username order by startDate  ) as rn1  
from 
(
select
username,activity,startDate,endDate,
row_number() over(partition by username order by startDate desc ) as rn 
from
UserActivity ) t1 
where rn = 1 or rn = 2  ) t2
where rn1 = 1
```
##189.在if中使用负数
```
Stocks 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| stock_name    | varchar |
| operation     | enum    |
| operation_day | int     |
| price         | int     |
+---------------+---------+
(stock_name, day) 是这张表的主键
operation 列使用的是一种枚举类型，包括：('Sell','Buy')
此表的每一行代表了名为 stock_name 的某支股票在 operation_day 这一天的操作价格。
保证股票的每次'Sell'操作前，都有相应的'Buy'操作。
 

编写一个SQL查询来报告每支股票的资本损益。

股票的资本损益是一次或多次买卖股票后的全部收益或损失。

以任意顺序返回结果即可。

SQL查询结果的格式如下例所示：

Stocks 表:
+---------------+-----------+---------------+--------+
| stock_name    | operation | operation_day | price  |
+---------------+-----------+---------------+--------+
| Leetcode      | Buy       | 1             | 1000   |
| Corona Masks  | Buy       | 2             | 10     |
| Leetcode      | Sell      | 5             | 9000   |
| Handbags      | Buy       | 17            | 30000  |
| Corona Masks  | Sell      | 3             | 1010   |
| Corona Masks  | Buy       | 4             | 1000   |
| Corona Masks  | Sell      | 5             | 500    |
| Corona Masks  | Buy       | 6             | 1000   |
| Handbags      | Sell      | 29            | 7000   |
| Corona Masks  | Sell      | 10            | 10000  |
+---------------+-----------+---------------+--------+

Result 表:
+---------------+-------------------+
| stock_name    | capital_gain_loss |
+---------------+-------------------+
| Corona Masks  | 9500              |
| Leetcode      | 8000              |
| Handbags      | -23000            |
+---------------+-------------------+
Leetcode 股票在第一天以1000美元的价格买入，在第五天以9000美元的价格卖出。资本收益=9000-1000=8000美元。
Handbags 股票在第17天以30000美元的价格买入，在第29天以7000美元的价格卖出。资本损失=7000-30000=-23000美元。
Corona Masks 股票在第1天以10美元的价格买入，在第3天以1010美元的价格卖出。在第4天以1000美元的价格再次购买，在第5天以500美元的价格出售。最后，它在第6天以1000美元的价格被买走，在第10天以10000美元的价格被卖掉。资本损益是每次（’Buy'->'Sell'）操作资本收益或损失的和=（1010-10）+（500-1000）+（10000-1000）=1000-500+9000=9500美元。
```

```
--简单题
select
stock_name,
sum(if(operation = "Buy",-price,price)) capital_gain_loss
from
Stocks
group by  stock_name
```
```
--这题是指标分析那题
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
##190.循环
```
Table: Tasks

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| task_id        | int     |
| subtasks_count | int     |
+----------------+---------+
task_id is the primary key for this table.
Each row in this table indicates that task_id was divided into subtasks_count subtasks labelled from 1 to subtasks_count.
It is guaranteed that 2 <= subtasks_count <= 20.
Table: Executed

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| task_id       | int     |
| subtask_id    | int     |
+---------------+---------+
(task_id, subtask_id) is the primary key for this table.
Each row in this table indicates that for the task task_id, the subtask with ID subtask_id was executed successfully.
It is guaranteed that subtask_id <= subtasks_count for each task_id.
Write an SQL query to report the IDs of the missing subtasks for each task_id.

Return the result table in any order.

The query result format is in the following example:

Tasks table:
+---------+----------------+
| task_id | subtasks_count |
+---------+----------------+
| 1       | 3              |
| 2       | 2              |
| 3       | 4              |
+---------+----------------+

Executed table:
+---------+------------+
| task_id | subtask_id |
+---------+------------+
| 1       | 2          |
| 3       | 1          |
| 3       | 2          |
| 3       | 3          |
| 3       | 4          |
+---------+------------+

Result table:
+---------+------------+
| task_id | subtask_id |
+---------+------------+
| 1       | 1          |
| 1       | 3          |
| 2       | 1          |
| 2       | 2          |
+---------+------------+
Task 1 was divided into 3 subtasks (1, 2, 3). Only subtask 2 was excuted successfully, so we include (1, 1) and (1, 3) in the answer.
Task 2 was divided into 2 subtasks (1, 2). No subtask was excuted successfully, so we include (2, 1) and (2, 2) in the answer.
Task 3 was divided into 4 subtasks (1, 2, 3, 4). All of the subtasks were excuted succes.
```
```
WITH RECURSIVE t(task_id, subtask_id) AS
(
    select distinct task_id, 1 from tasks
    UNION ALL
    SELECT t.task_id, subtask_id + 1 from t join tasks on t.task_id = tasks.task_id where subtask_id < subtasks_count
)
select
task_id, subtask_id 
from 
t 
where (task_id, subtask_id) not in (select task_id, subtask_id from Executed )
```
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
--我又写麻烦了.直接使用year和month即可
with recursive mon_time as 
(
    select 1 as month
    union all 
    select month + 1 from mon_time where month <= 11
)

select m.month,
sum(ifnull(d.num_driver,0)) over(order by m.month) as active_drivers,
ifnull(num_rides,0) as accepted_rides

from mon_time m 
left join 
(
    select if(year(join_date)<'2020',1,month(join_date)) as month_driver,
    count(driver_id) as num_driver
    from drivers
    where year(join_date)<='2020'
    group by if(year(join_date)<'2020',1,month(join_date))
)d  on m.month = month_driver
left join
(
    select month(requested_at) as month_rides,
    count(ride_id) as num_rides
    from rides 
    where ride_id in (select ride_id from acceptedrides)
    and year(requested_at) = '2020'
    group by month(requested_at)
)r on m.month = month_rides
group by m.month
order by m.month
```
```
表: Customers

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| customer_name | varchar |
+---------------+---------+
customer_id 是该表主键.
该表第一行包含了顾客的名字和id.
 

写一个 SQL 语句, 找到所有遗失的顾客id. 遗失的顾客id是指那些不在 Customers 表中, 值却处于 1 和表中最大 customer_id 之间的id.

注意: 最大的 customer_id 值不会超过 100.

返回结果按 ids 升序排列

查询结果格式如下例所示.

 

Customers 表:
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| 1           | Alice         |
| 4           | Bob           |
| 5           | Charlie       |
+-------------+---------------+

Result 表:
+-----+
| ids |
+-----+
| 2   |
| 3   |
+-----+
表中最大的customer_id是5, 所以在范围[1,5]内, ID2和3从表中遗失.
```
```
with recursive x as (
select 1 as n 
union all 
select n + 1 from  x where n <100
)

select
n as ids  
from 
x
where  n <= (select max(customer_id) from customers) and n not in (select customer_id from customers)
```
还有一些用法,找不到题目来说明了
1.join时 使用or来代替union all 
2.left join 同一张表join来避免子查询
3.采用一个组内全局的count来获取,top问题
4.联合去重distinct 
5.窗口大小 窗口函数结合使用的使用技巧题
6.窗口order by排序字段为count等聚合函数
7.SQL中的正则用法,SQL中的文本处理函数
.....
