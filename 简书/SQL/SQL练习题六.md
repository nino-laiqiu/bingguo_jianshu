##61.用户购买平台(☆)

>写一段 SQL 来查找每天 仅 使用手机端用户、仅 使用桌面端用户和 同时 使用桌面端和手机端的用户人数和总支出金额。

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
在 2019-07-01, 用户1 同时 使用桌面端和手机端购买, 用户2 仅 使用了手机端购买，而用户3 仅 使用了桌面端购买。
在 2019-07-02, 用户2 仅 使用了手机端购买, 用户3 仅 使用了桌面端购买，且没有用户 同时 使用桌面端和手机端购买。
```
```
-- 有点难度,难点一:按时间和id分组以及打标签,难点二:构建一张表来left join
思路分析
1.获取每一天的id的类型和acount
select 
spend_date,user_id,
if(count(distinct platform) =1,platform,'both') as platform,
sum(amount) as amount
from
Spending
group by spend_date, user_id

2.如果直接group by  spend_date, platform 就无法出现0,0的情况,考虑使用join来实现
构建表:
select distinct spend_date, "desktop" platform from Spending
union
select distinct spend_date, "mobile" platform from Spending
union
select distinct spend_date, "both" platform from Spending

综合:
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
--摘评论区写法
select
    spend_date, platform,
    ifnull(sum(total_am),0) total_amount,
    ifnull(sum(total_u),0) total_users
from
(
    select p.spend_date, p.platform, t.total_am, t.total_u
    from
    (
        select distinct spend_date, "desktop" platform from Spending
        union
        select distinct spend_date, "mobile" platform from Spending
        union
        select distinct spend_date, "both" platform from Spending
    ) p
    left join
    (
        select spend_date, 
            if(count(distinct platform)=1, platform, 'both') plat,
            sum(amount) total_am,
            count(distinct user_id) total_u
        from Spending
        group by spend_date, user_id
    ) t
    on p.platform = t.plat and p.spend_date = t.spend_date
) temp
group by spend_date, platform
```
是否要对最外面的select 中的统计人数去重,不需要但是怎么和leedcode的执行答案有出入
##62.报告的记录2(avg的应用)
>编写一段 SQL 来查找：在被报告为垃圾广告的帖子中，被移除的帖子的每日平均占比，四舍五入到小数点后 2 位
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
-- *100 位置不同影响结果....评论区提供了求平均值的方法比较好
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
##63.查询近三十天活跃用户数
```
-- 简单题,我的写法,注意这里的近三十不是小于等于
select
activity_date as day ,count(distinct user_id) as active_users
from
Activity
where datediff("2019-07-27",activity_date ) < 30 and activity_date <= "2019-07-27"
group by activity_date
```
##64.过去三十天的用户活动2

>编写SQL查询以查找截至2019年7月27日（含）的30天内每个用户的平均会话数，四舍五入到小数点后两位。我们只统计那些会话期间用户至少进行一项活动的有效会话。
```
Table: Activity

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| session_id    | int     |
| activity_date | date    |
| activity_type | enum    |
+---------------+---------+
该表没有主键，它可能有重复的行。
activity_type 列是 ENUM（“ open_session”，“ end_session”，“ scroll_down”，“ send_message”）中的某一类型。
该表显示了社交媒体网站的用户活动。
请注意，每个会话完全属于一个用户。
 

编写SQL查询以查找截至2019年7月27日（含）的30天内每个用户的平均会话数，四舍五入到小数点后两位。我们只统计那些会话期间用户至少进行一项活动的有效会话。

 

查询结果格式如下例所示：

Activity table:
+---------+------------+---------------+---------------+
| user_id | session_id | activity_date | activity_type |
+---------+------------+---------------+---------------+
| 1       | 1          | 2019-07-20    | open_session  |
| 1       | 1          | 2019-07-20    | scroll_down   |
| 1       | 1          | 2019-07-20    | end_session   |
| 2       | 4          | 2019-07-20    | open_session  |
| 2       | 4          | 2019-07-21    | send_message  |
| 2       | 4          | 2019-07-21    | end_session   |
| 3       | 2          | 2019-07-21    | open_session  |
| 3       | 2          | 2019-07-21    | send_message  |
| 3       | 2          | 2019-07-21    | end_session   |
| 3       | 5          | 2019-07-21    | open_session  |
| 3       | 5          | 2019-07-21    | scroll_down   |
| 3       | 5          | 2019-07-21    | end_session   |
| 4       | 3          | 2019-06-25    | open_session  |
| 4       | 3          | 2019-06-25    | end_session   |
+---------+------------+---------------+---------------+

Result table:
+---------------------------+ 
| average_sessions_per_user |
+---------------------------+ 
| 1.33                      |
+---------------------------+ 
User 1 和 2 在过去30天内各自进行了1次会话，而用户3进行了2次会话，因此平均值为（1 +1 + 2）/ 3 = 1.33。
```
```
--我的写法,麻烦了,看了评论区事实上直接求即可,就是上一题对avg的正确描述
select 
ifnull(round(sum(s1) /count(user_id),2),0) as  average_sessions_per_user
from 
(
select 
user_id,count(distinct session_id ) as s1
from 
Activity where datediff("2019-07-27",activity_date) < 30
and  
session_id 
in (
select
session_id
from
Activity
where datediff("2019-07-27",activity_date) < 30
group by session_id
having count(distinct activity_type) >= 1
) 
group by user_id
) t1
```
```
select ifnull(round(count(distinct session_id)/count(distinct user_id),2),0) 
as average_sessions_per_user
from Activity
where activity_date between '2019-06-28' and '2019-07-27'
```
##65.文章浏览1
```
-- 简单题
select 
distinct author_id as id
from
Views
where author_id = viewer_id
order by id 
```
##66.文章浏览2

>编写一条 SQL 查询来找出在同一天阅读至少两篇文章的人，结果按照 id 升序排序。
```
Table: Views

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| article_id    | int     |
| author_id     | int     |
| viewer_id     | int     |
| view_date     | date    |
+---------------+---------+
此表无主键，因此可能会存在重复行。此表的每一行都表示某人在某天浏览了某位作者的某篇文章。 请注意，同一人的 author_id 和 viewer_id 是相同的。
 

编写一条 SQL 查询来找出在同一天阅读至少两篇文章的人，结果按照 id 升序排序。

查询结果的格式如下：

Views table:
+------------+-----------+-----------+------------+
| article_id | author_id | viewer_id | view_date  |
+------------+-----------+-----------+------------+
| 1          | 3         | 5         | 2019-08-01 |
| 3          | 4         | 5         | 2019-08-01 |
| 1          | 3         | 6         | 2019-08-02 |
| 2          | 7         | 7         | 2019-08-01 |
| 2          | 7         | 6         | 2019-08-02 |
| 4          | 7         | 1         | 2019-07-22 |
| 3          | 4         | 4         | 2019-07-21 |
| 3          | 4         | 4         | 2019-07-21 |
+------------+-----------+-----------+------------+

Result table:
+------+
| id   |
+------+
| 5    |
| 6    |
+------+
```
```
-- 读题,跑了好几次,发现问题题目要求是读的文章,所以去重的字段应该是文章而不是读者
select  distinct viewer_id id
from views
group by viewer_id, view_date
having count(distinct article_id) > 1
order by id 
```
##67.市场分析1

>请写出一条SQL语句以查询每个用户的注册日期和在 2019 年作为买家的订单总数。
```
Table: Users

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| user_id        | int     |
| join_date      | date    |
| favorite_brand | varchar |
+----------------+---------+
此表主键是 user_id，表中描述了购物网站的用户信息，用户可以在此网站上进行商品买卖。
Table: Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| order_date    | date    |
| item_id       | int     |
| buyer_id      | int     |
| seller_id     | int     |
+---------------+---------+
此表主键是 order_id，外键是 item_id 和（buyer_id，seller_id）。
Table: Item

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| item_id       | int     |
| item_brand    | varchar |
+---------------+---------+
此表主键是 item_id。
 
请写出一条SQL语句以查询每个用户的注册日期和在 2019 年作为买家的订单总数。

查询结果格式如下：

Users table:
+---------+------------+----------------+
| user_id | join_date  | favorite_brand |
+---------+------------+----------------+
| 1       | 2018-01-01 | Lenovo         |
| 2       | 2018-02-09 | Samsung        |
| 3       | 2018-01-19 | LG             |
| 4       | 2018-05-21 | HP             |
+---------+------------+----------------+

Orders table:
+----------+------------+---------+----------+-----------+
| order_id | order_date | item_id | buyer_id | seller_id |
+----------+------------+---------+----------+-----------+
| 1        | 2019-08-01 | 4       | 1        | 2         |
| 2        | 2018-08-02 | 2       | 1        | 3         |
| 3        | 2019-08-03 | 3       | 2        | 3         |
| 4        | 2018-08-04 | 1       | 4        | 2         |
| 5        | 2018-08-04 | 1       | 3        | 4         |
| 6        | 2019-08-05 | 2       | 2        | 4         |
+----------+------------+---------+----------+-----------+

Items table:
+---------+------------+
| item_id | item_brand |
+---------+------------+
| 1       | Samsung    |
| 2       | Lenovo     |
| 3       | LG         |
| 4       | HP         |
+---------+------------+

Result table:
+-----------+------------+----------------+
| buyer_id  | join_date  | orders_in_2019 |
+-----------+------------+----------------+
| 1         | 2018-01-01 | 1              |
| 2         | 2018-02-09 | 2              |
| 3         | 2018-01-19 | 0              |
| 4         | 2018-05-21 | 0              |
+-----------+------------+----------------+
```
```
--我的写法,简单题
select 
user_id as buyer_id ,join_date ,ifnull(orders_in_2019,0) as orders_in_2019
from 
Users
left join 
( 
select
buyer_id,count(1) as orders_in_2019
from
Orders
where year(order_date) = "2019"
group by buyer_id
) t2
on Users. user_id = t2.buyer_id
```
```
--注意错误的写法,where条件会把行给过滤掉,所以要条件提取到on上
select  u.user_id  ,u.join_date ,ifnull(count(o.order_id),0) orders_in_2019 
from  Users u 
left join Orders o 
on u.user_id=o.buyer_id 
where year(o.order_date)=2019
group by u.user_id
```
##68.市场分析2

>写一个 SQL 查询确定每一个用户按日期顺序卖出的第二件商品的品牌是否是他们最喜爱的品牌。如果一个用户卖出少于两件商品，查询的结果是 no 。
```
表: Users

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| user_id        | int     |
| join_date      | date    |
| favorite_brand | varchar |
+----------------+---------+
user_id 是该表的主键
表中包含一位在线购物网站用户的个人信息，用户可以在该网站出售和购买商品。
表: Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| order_date    | date    |
| item_id       | int     |
| buyer_id      | int     |
| seller_id     | int     |
+---------------+---------+
order_id 是该表的主键
item_id 是 Items 表的外键
buyer_id 和 seller_id 是 Users 表的外键
表: Items

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| item_id       | int     |
| item_brand    | varchar |
+---------------+---------+
item_id 是该表的主键
 

写一个 SQL 查询确定每一个用户按日期顺序卖出的第二件商品的品牌是否是他们最喜爱的品牌。如果一个用户卖出少于两件商品，查询的结果是 no 。

题目保证没有一个用户在一天中卖出超过一件商品

下面是查询结果格式的例子：

Users table:
+---------+------------+----------------+
| user_id | join_date  | favorite_brand |
+---------+------------+----------------+
| 1       | 2019-01-01 | Lenovo         |
| 2       | 2019-02-09 | Samsung        |
| 3       | 2019-01-19 | LG             |
| 4       | 2019-05-21 | HP             |
+---------+------------+----------------+

Orders table:
+----------+------------+---------+----------+-----------+
| order_id | order_date | item_id | buyer_id | seller_id |
+----------+------------+---------+----------+-----------+
| 1        | 2019-08-01 | 4       | 1        | 2         |
| 2        | 2019-08-02 | 2       | 1        | 3         |
| 3        | 2019-08-03 | 3       | 2        | 3         |
| 4        | 2019-08-04 | 1       | 4        | 2         |
| 5        | 2019-08-04 | 1       | 3        | 4         |
| 6        | 2019-08-05 | 2       | 2        | 4         |
+----------+------------+---------+----------+-----------+

Items table:
+---------+------------+
| item_id | item_brand |
+---------+------------+
| 1       | Samsung    |
| 2       | Lenovo     |
| 3       | LG         |
| 4       | HP         |
+---------+------------+

Result table:
+-----------+--------------------+
| seller_id | 2nd_item_fav_brand |
+-----------+--------------------+
| 1         | no                 |
| 2         | yes                |
| 3         | yes                |
| 4         | no                 |
+-----------+--------------------+

id 为 1 的用户的查询结果是 no，因为他什么也没有卖出
id为 2 和 3 的用户的查询结果是 yes，因为他们卖出的第二件商品的品牌是他们自己最喜爱的品牌
id为 4 的用户的查询结果是 no，因为他卖出的第二件商品的品牌不是他最喜爱的品牌
```
```
--我的写法,简单题注意'' 和"" 是使用
select 
user_id as seller_id,
case when favorite_brand = t3.item_brand then 'yes' else 'no' end as  "2nd_item_fav_brand"
from 
Users
left join 
(
select 
seller_id,item_brand
from 
(
select 
item_brand,seller_id,
row_number() over(partition by seller_id order by order_date ) as rn
from 
Orders
join Items on Orders.item_id = Items.item_id
) t1
where rn =2
) t3
on Users.user_id = t3.seller_id
order by seller_id
```
```
-- 自连接获取第二个 having sum(o1.order_date > o2.order_date) = 1 学习一下
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
##69.指定日期的产品价格
>写一段 SQL来查找在 2019-08-16 时全部产品的价格，假设所有产品在修改前的价格都是 10。
```
产品数据表: Products

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| new_price     | int     |
| change_date   | date    |
+---------------+---------+
这张表的主键是 (product_id, change_date)。
这张表的每一行分别记录了 某产品 在某个日期 更改后 的新价格。
 

写一段 SQL来查找在 2019-08-16 时全部产品的价格，假设所有产品在修改前的价格都是 10。

查询结果格式如下例所示：

Products table:
+------------+-----------+-------------+
| product_id | new_price | change_date |
+------------+-----------+-------------+
| 1          | 20        | 2019-08-14  |
| 2          | 50        | 2019-08-14  |
| 1          | 30        | 2019-08-15  |
| 1          | 35        | 2019-08-16  |
| 2          | 65        | 2019-08-17  |
| 3          | 20        | 2019-08-18  |
+------------+-----------+-------------+

Result table:
+------------+-------+
| product_id | price |
+------------+-------+
| 2          | 50    |
| 1          | 35    |
| 3          | 10    |
+------------+-------+
```
```
-- 我的答案
select
t1.product_id,ifnull(t3.new_price,10) as price
from 
(
select
distinct product_id
from
Products
) t1
left join 
(
select
product_id,new_price
from 
Products
where (product_id,change_date) in 
(
select 
product_id,max(change_date)
from 
Products
where 
change_date <= '2019-08-16'
group by product_id
) 
) t3 
on t1.product_id = t3.product_id
```
```
select distinct a.product_id,ifnull(b.new_price,10) as price 
from Products a left join 
(select*,row_number()over(partition by product_id order by change_date desc) as rk 
from Products where change_date<='2019-08-16' )b
on a.product_id=b.product_id and rk=1
```
distinct 联合去重的问题概念有点模糊?,说明上面的不去重的结果是每个id有多个相同的price,去重获取唯一一个,如果值不同使用,对id去重,且price也不相同,那么去重的结果是全部的值,要想获取全部的id值只能子查询获取

```
-- 方案三,无需去重但是为什么比去重的跑得慢....
(select product_id as product_id,10 as price 
from Products 
group by product_id
having min(change_date)>'2019-08-16')
union all 
(select product_id,new_price as price 
from Products 
where (product_id,change_date) in
(select product_id,max(change_date) as max_date
from Products
where change_date<='2019-08-16'
group by product_id))
```
##70.即时食物配送1
>写一条 SQL 查询语句获取即时订单所占的百分比， 保留两位小数
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

写一条 SQL 查询语句获取即时订单所占的百分比， 保留两位小数。

查询结果如下所示：

Delivery 表:
+-------------+-------------+------------+-----------------------------+
| delivery_id | customer_id | order_date | customer_pref_delivery_date |
+-------------+-------------+------------+-----------------------------+
| 1           | 1           | 2019-08-01 | 2019-08-02                  |
| 2           | 5           | 2019-08-02 | 2019-08-02                  |
| 3           | 1           | 2019-08-11 | 2019-08-11                  |
| 4           | 3           | 2019-08-24 | 2019-08-26                  |
| 5           | 4           | 2019-08-21 | 2019-08-22                  |
| 6           | 2           | 2019-08-11 | 2019-08-13                  |
+-------------+-------------+------------+-----------------------------+

Result 表:
+----------------------+
| immediate_percentage |
+----------------------+
| 33.33                |
+----------------------+
2 和 3 号订单为即时订单，其他的为计划订单
```
```
select
round( count(if(order_date=customer_pref_delivery_date,1,null)) / count(1) *100,2) as immediate_percentage
from
Delivery
```
```
select
round( sum(if(order_date=customer_pref_delivery_date,1,0)) / count(1) *100,2) as immediate_percentage
from
Delivery
```
效率问题,上两条SQL的效率
```
--avg
select
round(avg(order_date=customer_pref_delivery_date)*100,2) immediate_percentage 
from
Delivery
```
```
--自连接
SELECT ROUND(COUNT(d2.customer_pref_delivery_date)/COUNT(d1.delivery_id)*100,2)
AS immediate_percentage
FROM Delivery d1 LEFT JOIN Delivery d2
ON d1.delivery_id= d2.delivery_id 
AND d1.order_date= d2.customer_pref_delivery_date;
```
