##91.列出指定时间段所有下单的产品
>写一个 SQL 语句，要求获取在 2020 年 2 月份下单的数量不少于 100 的产品的名字和数目
```
表: Products

+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| product_id       | int     |
| product_name     | varchar |
| product_category | varchar |
+------------------+---------+
product_id 是该表主键。
该表包含该公司产品的数据。
表: Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| order_date    | date    |
| unit          | int     |
+---------------+---------+
该表无主键，可能包含重复行。
product_id 是表单 Products 的外键。
unit 是在日期 order_date 内下单产品的数目。
 

写一个 SQL 语句，要求获取在 2020 年 2 月份下单的数量不少于 100 的产品的名字和数目。

返回结果表单的顺序无要求。

 

查询结果的格式如下：

Products 表:
+-------------+-----------------------+------------------+
| product_id  | product_name          | product_category |
+-------------+-----------------------+------------------+
| 1           | Leetcode Solutions    | Book             |
| 2           | Jewels of Stringology | Book             |
| 3           | HP                    | Laptop           |
| 4           | Lenovo                | Laptop           |
| 5           | Leetcode Kit          | T-shirt          |
+-------------+-----------------------+------------------+

Orders 表:
+--------------+--------------+----------+
| product_id   | order_date   | unit     |
+--------------+--------------+----------+
| 1            | 2020-02-05   | 60       |
| 1            | 2020-02-10   | 70       |
| 2            | 2020-01-18   | 30       |
| 2            | 2020-02-11   | 80       |
| 3            | 2020-02-17   | 2        |
| 3            | 2020-02-24   | 3        |
| 4            | 2020-03-01   | 20       |
| 4            | 2020-03-04   | 30       |
| 4            | 2020-03-04   | 60       |
| 5            | 2020-02-25   | 50       |
| 5            | 2020-02-27   | 50       |
| 5            | 2020-03-01   | 50       |
+--------------+--------------+----------+

Result 表:
+--------------------+---------+
| product_name       | unit    |
+--------------------+---------+
| Leetcode Solutions | 130     |
| Leetcode Kit       | 100     |
+--------------------+---------+

2020 年 2 月份下单 product_id = 1 的产品的数目总和为 (60 + 70) = 130 
2020 年 2 月份下单 product_id = 2 的产品的数目总和为 80 
2020 年 2 月份下单 product_id = 3 的产品的数目总和为 (2 + 3) = 5 
2020 年 2 月份 product_id = 4 的产品并没有下单
2020 年 2 月份下单 product_id = 5 的产品的数目总和为 (50 + 50) = 100 
```
```
-- 简单题,使用like来获取年份效率高
select
product_name,sum(unit) as unit
from
Products t1
join Orders t2
on t1.product_id = t2.product_id and left(order_date,7) = "2020-02"
group by product_name
having sum(unit) >= 100
```
```
#总结一下获取年份的方法
(1) order_date between '2020-02-01' and '2020-02-29'  
(2) order_date like '2020-02%'  
(3) DATE_FORMAT(order_date, "%Y-%m") = "2020-02"   
(4) LEFT(order_date, 7) 或 substr(1, 7)
```
##92.每次访问的交易次数(构造0-max)☆
>银行想要得到银行客户在一次访问时的交易次数和相应的在一次访问时该交易次数的客户数量的图表
写一条 SQL 查询多少客户访问了银行但没有进行任何交易，多少客户访问了银行进行了一次交易等等
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
![这个例子的图表](https://upload-images.jianshu.io/upload_images/9049859-1458ffc55f163f3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
-- 难题没想到使用序列函数和0 union all 构造0-max
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

##93.电影评分
>请你编写一组 SQL 查询：
查找评论电影数量最多的用户名。
如果出现平局，返回字典序较小的用户名。
查找在 2020 年 2 月 平均评分最高 的电影名称。
如果出现平局，返回字典序较小的电影名称。


```
表：Movies

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| movie_id      | int     |
| title         | varchar |
+---------------+---------+
movie_id 是这个表的主键。
title 是电影的名字。
表：Users

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| name          | varchar |
+---------------+---------+
user_id 是表的主键。
表：Movie_Rating

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| movie_id      | int     |
| user_id       | int     |
| rating        | int     |
| created_at    | date    |
+---------------+---------+
(movie_id, user_id) 是这个表的主键。
这个表包含用户在其评论中对电影的评分 rating 。
created_at 是用户的点评日期。 
 

请你编写一组 SQL 查询：

查找评论电影数量最多的用户名。
如果出现平局，返回字典序较小的用户名。

查找在 2020 年 2 月 平均评分最高 的电影名称。
如果出现平局，返回字典序较小的电影名称。

查询分两行返回，查询结果格式如下例所示：

Movies 表：
+-------------+--------------+
| movie_id    |  title       |
+-------------+--------------+
| 1           | Avengers     |
| 2           | Frozen 2     |
| 3           | Joker        |
+-------------+--------------+

Users 表：
+-------------+--------------+
| user_id     |  name        |
+-------------+--------------+
| 1           | Daniel       |
| 2           | Monica       |
| 3           | Maria        |
| 4           | James        |
+-------------+--------------+

Movie_Rating 表：
+-------------+--------------+--------------+-------------+
| movie_id    | user_id      | rating       | created_at  |
+-------------+--------------+--------------+-------------+
| 1           | 1            | 3            | 2020-01-12  |
| 1           | 2            | 4            | 2020-02-11  |
| 1           | 3            | 2            | 2020-02-12  |
| 1           | 4            | 1            | 2020-01-01  |
| 2           | 1            | 5            | 2020-02-17  | 
| 2           | 2            | 2            | 2020-02-01  | 
| 2           | 3            | 2            | 2020-03-01  |
| 3           | 1            | 3            | 2020-02-22  | 
| 3           | 2            | 4            | 2020-02-25  | 
+-------------+--------------+--------------+-------------+

Result 表：
+--------------+
| results      |
+--------------+
| Daniel       |
| Frozen 2     |
+--------------+

Daniel 和 Monica 都点评了 3 部电影（"Avengers", "Frozen 2" 和 "Joker"） 但是 Daniel 字典序比较小。
Frozen 2 和 Joker 在 2 月的评分都是 3.5，但是 Frozen 2 的字典序比较小。
```
```
--简单题,为什么union all 上下要打括号?
(
select
name results 
from
Users t1
join Movie_Rating t2
on t1.user_id = t2.user_id
group by name
order by count(distinct movie_id) desc,name
limit 1 ) 
union all
(
select 
title 
from 
Movies t3
join Movie_Rating t4
on t3.movie_id = t4.movie_id  
where left(created_at ,7) = "2020-02"
group by title 
order by avg(rating ) desc, title 
limit 1 ) 
```
##94院系无效的学生
>写一条 SQL 语句以查询那些所在院系不存在的学生的 id 和姓名
```
院系表: Departments

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
id 是该表的主键
该表包含一所大学每个院系的 id 信息
 

学生表: Students

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
| department_id | int     |
+---------------+---------+
id 是该表的主键
该表包含一所大学每个学生的 id 和他/她就读的院系信息
 

写一条 SQL 语句以查询那些所在院系不存在的学生的 id 和姓名

可以以任何顺序返回结果

下面是返回结果格式的例子

Departments 表:
+------+--------------------------+
| id   | name                     |
+------+--------------------------+
| 1    | Electrical Engineering   |
| 7    | Computer Engineering     |
| 13   | Bussiness Administration |
+------+--------------------------+

Students 表:
+------+----------+---------------+
| id   | name     | department_id |
+------+----------+---------------+
| 23   | Alice    | 1             |
| 1    | Bob      | 7             |
| 5    | Jennifer | 13            |
| 2    | John     | 14            |
| 4    | Jasmine  | 77            |
| 3    | Steve    | 74            |
| 6    | Luis     | 1             |
| 8    | Jonathan | 7             |
| 7    | Daiana   | 33            |
| 11   | Madelynn | 1             |
+------+----------+---------------+

结果表:
+------+----------+
| id   | name     |
+------+----------+
| 2    | John     |
| 7    | Daiana   |
| 4    | Jasmine  |
| 3    | Steve    |
+------+----------+

John, Daiana, Steve 和 Jasmine 所在的院系分别是 14, 33, 74 和 77， 其中 14, 33, 74 和 77 并不存在于院系表
```
```
--简单题
select
s.id,s.name
from 
Students s left join Departments d on s.department_id = d.id 
where d.id is null
```
```
select id, name
from Students
where department_id not in (select id from Departments)
```
>join之间的差别
>https://leetcode-cn.com/problems/nth-highest-salary/solution/mysql-zi-ding-yi-bian-liang-by-luanz/

##95.活动参与者
>写一条 SQL 查询那些既没有最多，也没有最少参与者的活动的名字
```
表: Friends

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
| activity      | varchar |
+---------------+---------+
id 是朋友的 id 和该表的主键
name 是朋友的名字
activity 是朋友参加的活动的名字
表: Activities

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
id 是该表的主键
name 是活动的名字
 

写一条 SQL 查询那些既没有最多，也没有最少参与者的活动的名字

可以以任何顺序返回结果，Activities 表的每项活动的参与者都来自 Friends 表

注意：名称相同 id 不同的参与者算作两个人

下面是查询结果格式的例子：

Friends 表:
+------+--------------+---------------+
| id   | name         | activity      |
+------+--------------+---------------+
| 1    | Jonathan D.  | Eating        |
| 2    | Jade W.      | Singing       |
| 3    | Victor J.    | Singing       |
| 4    | Elvis Q.     | Eating        |
| 5    | Daniel A.    | Eating        |
| 6    | Bob B.       | Horse Riding  |
+------+--------------+---------------+

Activities 表:
+------+--------------+
| id   | name         |
+------+--------------+
| 1    | Eating       |
| 2    | Singing      |
| 3    | Horse Riding |
+------+--------------+

Result 表:
+--------------+
| activity     |
+--------------+
| Singing      |
+--------------+

Eating 活动有三个人参加, 是最多人参加的活动 (Jonathan D. , Elvis Q. and Daniel A.)
Horse Riding 活动有一个人参加, 是最少人参加的活动 (Bob B.)
Singing 活动有两个人参加 (Victor J. and Jade W.)
```
```
-- 简单题,窗口
select
name as activity 
from 
Activities
where name not in 
(
select
activity 
from 
(
select
activity,
dense_rank() over(order by count(1) desc) as max_count,
dense_rank() over(order by count(1) asc) as min_count
from
Friends f 
group by 
activity
) t1
where max_count =1 or min_count = 1
)
```
```
with a as 
(select activity, count(*) amount
from friends
group by activity)

select activity
from friends
group by activity
having count(*) > (select min(amount) from a) 
and count(*) < (select max(amount) from a)
```
##96.顾客的可信联系人数量
>为每张发票 invoice_id 编写一个SQL查询以查找以下内容：
customer_name：与发票相关的顾客名称。
price：发票的价格。
contacts_cnt：该顾客的联系人数量。
trusted_contacts_cnt：可信联系人的数量：既是该顾客的联系人又是商店顾客的联系人数量（即：可信联系人的电子邮件存在于客户表中）。

```
顾客表：Customers

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| customer_name | varchar |
| email         | varchar |
+---------------+---------+
customer_id 是这张表的主键。
此表的每一行包含了某在线商店顾客的姓名和电子邮件。
 

联系方式表：Contacts

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | id      |
| contact_name  | varchar |
| contact_email | varchar |
+---------------+---------+
(user_id, contact_email) 是这张表的主键。
此表的每一行表示编号为 user_id 的顾客的某位联系人的姓名和电子邮件。
此表包含每位顾客的联系人信息，但顾客的联系人不一定存在于顾客表中。
 

发票表：Invoices

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| invoice_id   | int     |
| price        | int     |
| user_id      | int     |
+--------------+---------+
invoice_id 是这张表的主键。
此表的每一行分别表示编号为 user_id 的顾客拥有有一张编号为 invoice_id、价格为 price 的发票。
 

为每张发票 invoice_id 编写一个SQL查询以查找以下内容：

customer_name：与发票相关的顾客名称。
price：发票的价格。
contacts_cnt：该顾客的联系人数量。
trusted_contacts_cnt：可信联系人的数量：既是该顾客的联系人又是商店顾客的联系人数量（即：可信联系人的电子邮件存在于客户表中）。
将查询的结果按照 invoice_id 排序。

查询结果的格式如下例所示：

Customers table:
+-------------+---------------+--------------------+
| customer_id | customer_name | email              |
+-------------+---------------+--------------------+
| 1           | Alice         | alice@leetcode.com |
| 2           | Bob           | bob@leetcode.com   |
| 13          | John          | john@leetcode.com  |
| 6           | Alex          | alex@leetcode.com  |
+-------------+---------------+--------------------+
Contacts table:
+-------------+--------------+--------------------+
| user_id     | contact_name | contact_email      |
+-------------+--------------+--------------------+
| 1           | Bob          | bob@leetcode.com   |
| 1           | John         | john@leetcode.com  |
| 1           | Jal          | jal@leetcode.com   |
| 2           | Omar         | omar@leetcode.com  |
| 2           | Meir         | meir@leetcode.com  |
| 6           | Alice        | alice@leetcode.com |
+-------------+--------------+--------------------+
Invoices table:
+------------+-------+---------+
| invoice_id | price | user_id |
+------------+-------+---------+
| 77         | 100   | 1       |
| 88         | 200   | 1       |
| 99         | 300   | 2       |
| 66         | 400   | 2       |
| 55         | 500   | 13      |
| 44         | 60    | 6       |
+------------+-------+---------+
Result table:
+------------+---------------+-------+--------------+----------------------+
| invoice_id | customer_name | price | contacts_cnt | trusted_contacts_cnt |
+------------+---------------+-------+--------------+----------------------+
| 44         | Alex          | 60    | 1            | 1                    |
| 55         | John          | 500   | 0            | 0                    |
| 66         | Bob           | 400   | 2            | 0                    |
| 77         | Alice         | 100   | 3            | 2                    |
| 88         | Alice         | 200   | 3            | 2                    |
| 99         | Bob           | 300   | 2            | 0                    |
+------------+---------------+-------+--------------+----------------------+
Alice 有三位联系人，其中两位(Bob 和 John)是可信联系人
Bob 有两位联系人, 他们中的任何一位都不是可信联系人
Alex 只有一位联系人(Alice)，并是一位可信联系人
John 没有任何联系人
```
```
--简单题,对if子查询查询优化一下,采用自连接
select
invoice_id,customer_name,price,contacts_cnt,trusted_contacts_cnt 
from 
(
select
invoice_id,customer_name,price,customer_id
from
Invoices
join Customers on Invoices.user_id = Customers.customer_id
) s1
join 
(
select
customer_id,ifnull(count(contact_name),0) contacts_cnt ,
ifnull(sum(if(contact_name in (select customer_name from Customers) ,1,0)),0) trusted_contacts_cnt
from
Customers c1 
left join Contacts c2 on c1.customer_id = c2.user_id
group by customer_id
) s2
on s1.customer_id =s2.customer_id
order by invoice_id
```
```
select
    i.invoice_id,
    cut.customer_name customer_name,
    i.price,
    count(co.contact_email) contacts_cnt,
    count(cu.email) trusted_contacts_cnt
from Invoices i
left join Customers cut on i.user_id = cut.customer_id
left join Contacts co on i.user_id = co.user_id
left join Customers cu on co.contact_email = cu.email
group by i.invoice_id,i.user_id
order by i.invoice_id
```
##97.获取最近第二次的活动(组内count)
>写一条SQL查询展示每一位用户 最近第二次 的活动
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
-- 简单题,优化一下
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
##98.使用唯一标识码替换员工id
>写一段SQL查询来展示每位用户的 唯一标识码（unique ID ）；如果某位员工没有唯一标识码，使用 null 填充即可。
```
Employees 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
id 是这张表的主键。
这张表的每一行分别代表了某公司其中一位员工的名字和 ID 。
 

EmployeeUNI 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| unique_id     | int     |
+---------------+---------+
(id, unique_id) 是这张表的主键。
这张表的每一行包含了该公司某位员工的 ID 和他的唯一标识码（unique ID）。
 

写一段SQL查询来展示每位用户的 唯一标识码（unique ID ）；如果某位员工没有唯一标识码，使用 null 填充即可。

你可以以 任意 顺序返回结果表。

查询结果的格式如下例所示：

Employees table:
+----+----------+
| id | name     |
+----+----------+
| 1  | Alice    |
| 7  | Bob      |
| 11 | Meir     |
| 90 | Winston  |
| 3  | Jonathan |
+----+----------+

EmployeeUNI table:
+----+-----------+
| id | unique_id |
+----+-----------+
| 3  | 1         |
| 11 | 2         |
| 90 | 3         |
+----+-----------+

EmployeeUNI table:
+-----------+----------+
| unique_id | name     |
+-----------+----------+
| null      | Alice    |
| null      | Bob      |
| 2         | Meir     |
| 3         | Winston  |
| 1         | Jonathan |
+-----------+----------+

Alice and Bob 没有唯一标识码, 因此我们使用 null 替代。
Meir 的唯一标识码是 2 。
Winston 的唯一标识码是 3 。
Jonathan 唯一标识码是 1 
```
```
--简单题
select
 unique_id,name
from
Employees
left join EmployeeUNI on EmployeeUNI.id = Employees.id
```
##99.按年度列出销售总额
>编写一段 SQL 查询每个产品每年的总销售额，并包含 product_id, product_name 以及 report_year 等信息。
销售年份的日期介于 2018 年到 2020 年之间。你返回的结果需要按 product_id 和 report_year 排序。
```
Product 表：

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
+---------------+---------+
product_id 是这张表的主键。
product_name 是产品的名称。
 

Sales 表：

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| product_id          | int     |
| period_start        | date    |
| period_end          | date    |
| average_daily_sales | int     |
+---------------------+---------+
product_id 是这张表的主键。
period_start 和 period_end 是该产品销售期的起始日期和结束日期，且这两个日期包含在销售期内。
average_daily_sales 列存储销售期内该产品的日平均销售额。
 

编写一段 SQL 查询每个产品每年的总销售额，并包含 product_id, product_name 以及 report_year 等信息。

销售年份的日期介于 2018 年到 2020 年之间。你返回的结果需要按 product_id 和 report_year 排序。

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
LC T-shirt 在 2018-12-01 至 2020-01-01 期间销售，该产品在2018年、2019年、2020年的销售时间分别是31天、365天、1天，2018年、2019年、2020年的销售总额分别是31*10=310、365*10=3650、1*10=10。
LC Keychain 在 2019-12-01 至 2020-01-31 期间销售，该产品在2019年、2020年的销售时间分别是：31天、31天，2019年、2020年的销售总额分别是31*1=31、31*1=31。
```
```
--简单题,怎么优化一下吧负数给在里面那层就过滤掉
select 
x1.product_id,product_name,report_year,total_amount
from 
(
select
product_id,
"2020" as report_year ,
(datediff(period_end,"2020-01-01") + 1 ) * average_daily_sales as total_amount
from
Sales
where  period_end >= "2020-01-01" 
union all 
(
select 
product_id,
"2019" as report_year,
(datediff(
if(datediff(period_end,"2019-12-31")>=0,"2019-12-31",period_end),
if(datediff(period_start,"2019-01-01")<=0,"2019-01-01",period_start)
)  +1 ) * average_daily_sales 
from
Sales )
union all 
(
select 
product_id,
"2018" as report_year,
(datediff(
if(datediff(period_end,"2018-12-31")>=0,"2018-12-31",period_end),
if(datediff(period_start,"2018-01-01")<=0,"2018-01-01",period_start)
)  +1 ) * average_daily_sales 
from
Sales ) ) x1 
join Product on x1.product_id = Product.product_id
where total_amount>=0
order by product_id,report_year 
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
```
WITH report_year_list AS (
    SELECT 2018 AS report_year
    UNION
    SELECT 2019 AS report_year
    UNION 
    SELECT 2020 AS report_year
),

product_report_year AS (
    SELECT s.product_id, s.period_start, s.period_end, s.average_daily_sales, r.report_year
    FROM Sales s JOIN report_year_list r
    ON YEAR(s.period_start) <= r.report_year AND r.report_year <= YEAR(s.period_end)    
)

SELECT
    CAST(p.product_id AS CHAR) AS product_id,
    p.product_name,
    CAST(pr.report_year AS CHAR) AS report_year,
    CASE
        WHEN YEAR(period_start) = 2018 AND YEAR(period_end) = 2018 THEN (1+DATEDIFF(period_end, period_start))*average_daily_sales
        WHEN YEAR(period_start) = 2018 AND YEAR(period_end) = 2019 AND report_year = 2018 THEN (1+DATEDIFF("2018-12-31", period_start))*average_daily_sales
        WHEN YEAR(period_start) = 2018 AND YEAR(period_end) = 2019 AND report_year = 2019 THEN (1+DATEDIFF(period_end, "2019-01-01"))*average_daily_sales
        WHEN YEAR(period_start) = 2018 AND YEAR(period_end) = 2020 AND report_year = 2018 THEN (1+DATEDIFF("2018-12-31", period_start))*average_daily_sales
        WHEN YEAR(period_start) = 2018 AND YEAR(period_end) = 2020 AND report_year = 2019 THEN (1+DATEDIFF("2019-12-31", "2019-01-01"))*average_daily_sales
        WHEN YEAR(period_start) = 2018 AND YEAR(period_end) = 2020 AND report_year = 2020 THEN (1+DATEDIFF(period_end, "2020-01-01"))*average_daily_sales
        WHEN YEAR(period_start) = 2019 AND YEAR(period_end) = 2019 THEN (1+DATEDIFF(period_end, period_start))*average_daily_sales
        WHEN YEAR(period_start) = 2019 AND YEAR(period_end) = 2020 AND report_year = 2019 THEN (1+DATEDIFF("2019-12-31", period_start))*average_daily_sales
        WHEN YEAR(period_start) = 2019 AND YEAR(period_end) = 2020 AND report_year = 2020 THEN (1+DATEDIFF(period_end, "2020-01-01"))*average_daily_sales
        WHEN YEAR(period_start) = 2020 AND YEAR(period_end) = 2020 THEN (1+DATEDIFF(period_end, period_start))*average_daily_sales
    END AS total_amount
FROM product_report_year pr INNER JOIN Product p
ON pr.product_id = p.product_id
ORDER BY CAST(p.product_id AS CHAR), CAST(pr.report_year AS CHAR)
```
##100.股票的资本损益
>编写一个SQL查询来报告每支股票的资本损益
```
Stocks 表：

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
