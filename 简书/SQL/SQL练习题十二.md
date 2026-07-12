##121.银行账户概要
>力扣银行 (LCB) 帮助程序员们完成虚拟支付。我们的银行在表 Transaction 中记录每条交易信息，我们要查询每个用户的当前余额，并检查他们是否已透支（当前额度小于 0）。
写一条 SQL 语句，查询：
user_id 用户 ID
user_name 用户名
credit 完成交易后的余额
credit_limit_breached 检查是否透支 （"Yes" 或 "No"）

```
用户表： Users

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| user_id      | int     |
| user_name    | varchar |
| credit       | int     |
+--------------+---------+
user_id 是这个表的主键。
表中的每一列包含每一个用户当前的额度信息。
 

交易表：Transactions

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| trans_id      | int     |
| paid_by       | int     |
| paid_to       | int     |
| amount        | int     |
| transacted_on | date    |
+---------------+---------+
trans_id 是这个表的主键。
表中的每一列包含银行的交易信息。
ID 为 paid_by 的用户给 ID 为 paid_to 的用户转账。
 

力扣银行 (LCB) 帮助程序员们完成虚拟支付。我们的银行在表 Transaction 中记录每条交易信息，我们要查询每个用户的当前余额，并检查他们是否已透支（当前额度小于 0）。

写一条 SQL 语句，查询：

user_id 用户 ID
user_name 用户名
credit 完成交易后的余额
credit_limit_breached 检查是否透支 （"Yes" 或 "No"）
以任意顺序返回结果表。

 

查询格式见如下示例：

Users 表：
+------------+--------------+-------------+
| user_id    | user_name    | credit      |
+------------+--------------+-------------+
| 1          | Moustafa     | 100         |
| 2          | Jonathan     | 200         |
| 3          | Winston      | 10000       |
| 4          | Luis         | 800         | 
+------------+--------------+-------------+

Transactions 表：
+------------+------------+------------+----------+---------------+
| trans_id   | paid_by    | paid_to    | amount   | transacted_on |
+------------+------------+------------+----------+---------------+
| 1          | 1          | 3          | 400      | 2020-08-01    |
| 2          | 3          | 2          | 500      | 2020-08-02    |
| 3          | 2          | 1          | 200      | 2020-08-03    |
+------------+------------+------------+----------+---------------+

结果表：
+------------+------------+------------+-----------------------+
| user_id    | user_name  | credit     | credit_limit_breached |
+------------+------------+------------+-----------------------+
| 1          | Moustafa   | -100       | Yes                   | 
| 2          | Jonathan   | 500        | No                    |
| 3          | Winston    | 9900       | No                    |
| 4          | Luis       | 800        | No                    |
+------------+------------+------------+-----------------------+
Moustafa 在 "2020-08-01" 支付了 $400 并在 "2020-08-03" 收到了 $200 ，当前额度 (100 -400 +200) = -$100
Jonathan 在 "2020-08-02" 收到了 $500 并在 "2020-08-08" 支付了 $200 ，当前额度 (200 +500 -200) = $500
Winston 在 "2020-08-01" 收到了 $400 并在 "2020-08-03" 支付了 $500 ，当前额度 (10000 +400 -500) = $9900
Luis 未收到任何转账信息，额度 = $800
```
```
--简单题,题意有点绕,实践一下join or的使用,也可以union all
select
t1.user_id,user_name,
max(credit) +
sum(
case when t1.user_id = t2.paid_by then - amount
     when t1.user_id = t2.paid_to then  amount
     else 0 end ) as credit,
if(max(credit) + sum(
case when t1.user_id = t2.paid_by then - amount
     when t1.user_id = t2.paid_to then  amount
     else 0 end) < 0,"Yes","No")  credit_limit_breached    
from
Users t1 
left join Transactions t2
on t1.user_id = t2.paid_by  or  t1.user_id = t2.paid_to 
group by t1.user_id,user_name  
```
##122.按月统计订单数和顾客数
>写一个查询语句来 按月 统计 金额大于 $20 的唯一 订单数 和唯一 顾客数 。
```
表：Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| order_date    | date    |
| customer_id   | int     |
| invoice       | int     |
+---------------+---------+
order_id 是 Orders 表的主键。
这张表包含顾客(customer_id)所下订单的信息。
写一个查询语句来 按月 统计 金额大于 $20 的唯一 订单数 和唯一 顾客数 。

查询结果无排序要求。

查询结果格式如下面例子所示：

Orders
+----------+------------+-------------+------------+
| order_id | order_date | customer_id | invoice    |
+----------+------------+-------------+------------+
| 1        | 2020-09-15 | 1           | 30         |
| 2        | 2020-09-17 | 2           | 90         |
| 3        | 2020-10-06 | 3           | 20         |
| 4        | 2020-10-20 | 3           | 21         |
| 5        | 2020-11-10 | 1           | 10         |
| 6        | 2020-11-21 | 2           | 15         |
| 7        | 2020-12-01 | 4           | 55         |
| 8        | 2020-12-03 | 4           | 77         |
| 9        | 2021-01-07 | 3           | 31         |
| 10       | 2021-01-15 | 2           | 20         |
+----------+------------+-------------+------------+

Result 表：
+---------+-------------+----------------+
| month   | order_count | customer_count |
+---------+-------------+----------------+
| 2020-09 | 2           | 2              |
| 2020-10 | 1           | 1              |
| 2020-12 | 2           | 1              |
| 2021-01 | 1           | 1              |
+---------+-------------+----------------+
在 2020 年 09 月，有 2 份来自 2 位不同顾客的金额大于 $20 的订单。
在 2020 年 10 月，有 2 份来自 1 位顾客的订单，并且只有其中的 1 份订单金额大于 $20 。
在 2020 年 11 月，有 2 份来自 2 位不同顾客的订单，但由于金额都小于 $20 ，所以我们的查询结果中不包含这个月的数据。
在 2020 年 12 月，有 2 份来自 1 位顾客的订单，且 2 份订单金额都大于 $20 。
在 2021 年 01 月，有 2 份来自 2 位不同顾客的订单，但只有其中一份订单金额大于 $20 
```
```
--简单题,一开始我把where条件过滤,写到了select中来去重,发现语法不对?
select
left(order_date,7) month,
count(order_id) order_count,
count(distinct customer_id ) customer_count
from 
Orders
where invoice >20
group by left(order_date,7)
```
##123.仓库经理
>写一个 SQL 查询来报告, 每个仓库的存货量是多少立方英尺.
```
表: Warehouse

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| name         | varchar |
| product_id   | int     |
| units        | int     |
+--------------+---------+
(name, product_id) 是该表主键.
该表的行包含了每个仓库的所有商品信息.
 

表: Products

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
| Width         | int     |
| Length        | int     |
| Height        | int     |
+---------------+---------+
product_id 是该表主键.
该表的行包含了每件商品以英尺为单位的尺寸(宽度, 长度和高度)信息.
 

写一个 SQL 查询来报告, 每个仓库的存货量是多少立方英尺.

仓库名
存货量
返回结果没有顺序要求.

查询结果如下例所示.

 

Warehouse 表:
+------------+--------------+-------------+
| name       | product_id   | units       |
+------------+--------------+-------------+
| LCHouse1   | 1            | 1           |
| LCHouse1   | 2            | 10          |
| LCHouse1   | 3            | 5           |
| LCHouse2   | 1            | 2           |
| LCHouse2   | 2            | 2           |
| LCHouse3   | 4            | 1           |
+------------+--------------+-------------+

Products 表:
+------------+--------------+------------+----------+-----------+
| product_id | product_name | Width      | Length   | Height    |
+------------+--------------+------------+----------+-----------+
| 1          | LC-TV        | 5          | 50       | 40        |
| 2          | LC-KeyChain  | 5          | 5        | 5         |
| 3          | LC-Phone     | 2          | 10       | 10        |
| 4          | LC-T-Shirt   | 4          | 10       | 20        |
+------------+--------------+------------+----------+-----------+

Result 表:
+----------------+------------+
| WAREHOUSE_NAME | VOLUME     | 
+----------------+------------+
| LCHouse1       | 12250      | 
| LCHouse2       | 20250      |
| LCHouse3       | 800        |
+----------------+------------+
Id为1的商品(LC-TV)的存货量为 5x50x40 = 10000
Id为2的商品(LC-KeyChain)的存货量为 5x5x5 = 125 
Id为3的商品(LC-Phone)的存货量为 2x10x10 = 200
Id为4的商品(LC-T-Shirt)的存货量为 4x10x20 = 800
仓库LCHouse1: 1个单位的LC-TV + 10个单位的LC-KeyChain + 5个单位的LC-Phone.
          总存货量为: 1*10000 + 10*125  + 5*200 = 12250 立方英尺
仓库LCHouse2: 2个单位的LC-TV + 2个单位的LC-KeyChain.
          总存货量为: 2*10000 + 2*125 = 20250 立方英尺
仓库LCHouse3: 1个单位的LC-T-Shirt.
          总存货量为: 1*800 = 800 立方英尺
```
```
--简单题
select
name WAREHOUSE_NAME ,sum(units * Width *  Length * Height )  VOLUME 
from 
Warehouse t1
join Products t2 on t1.product_id  =t2.product_id
group by name
```
##124.进店为进行交易过的顾客
>有一些顾客可能光顾了购物中心但没有进行交易。请你编写一个 SQL 查询，来查找这些顾客的 ID ，以及他们只光顾不交易的次数。
```
表：Visits

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| visit_id    | int     |
| customer_id | int     |
+-------------+---------+
visit_id 是该表的主键。
该表包含有关光临过购物中心的顾客的信息。
 

表：Transactions

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| transaction_id | int     |
| visit_id       | int     |
| amount         | int     |
+----------------+---------+
transaction_id 是此表的主键。
此表包含 visit_id 期间进行的交易的信息。
有一些顾客可能光顾了购物中心但没有进行交易。请你编写一个 SQL 查询，来查找这些顾客的 ID ，以及他们只光顾不交易的次数。

返回以任何顺序排序的结果表。

查询结果格式如下例所示：

Visits
+----------+-------------+
| visit_id | customer_id |
+----------+-------------+
| 1        | 23          |
| 2        | 9           |
| 4        | 30          |
| 5        | 54          |
| 6        | 96          |
| 7        | 54          |
| 8        | 54          |
+----------+-------------+

Transactions
+----------------+----------+--------+
| transaction_id | visit_id | amount |
+----------------+----------+--------+
| 2              | 5        | 310    |
| 3              | 5        | 300    |
| 9              | 5        | 200    |
| 12             | 1        | 910    |
| 13             | 2        | 970    |
+----------------+----------+--------+

Result 表：
+-------------+----------------+
| customer_id | count_no_trans |
+-------------+----------------+
| 54          | 2              |
| 30          | 1              |
| 96          | 1              |
+-------------+----------------+
ID = 23 的顾客曾经逛过一次购物中心，并在 ID = 12 的访问期间进行了一笔交易。
ID = 9 的顾客曾经逛过一次购物中心，并在 ID = 13 的访问期间进行了一笔交易。
ID = 30 的顾客曾经去过购物中心，并且没有进行任何交易。
ID = 54 的顾客三度造访了购物中心。在 2 次访问中，他们没有进行任何交易，在 1 次访问中，他们进行了 3 次交易。
ID = 96 的顾客曾经去过购物中心，并且没有进行任何交易。
如我们所见，ID 为 30 和 96 的顾客一次没有进行任何交易就去了购物中心。顾客 54 也两次访问了购物中心并且没有进行任何交易
```
```
--简单题
select
customer_id ,sum(if(amount is null,1,0)) count_no_trans
from
Visits t1
left join Transactions t2 on t1.visit_id = t2.visit_id
group by customer_id
having sum(if(amount is null,1,0)) >= 1
```
```
--直接过滤
select v.customer_id, count(*) count_no_trans
from visits v left join transactions t on v.visit_id = t.visit_id
where t.visit_id  is null
group by v.customer_id
```
```
--子查询
select customer_id 
    ,count(visit_id) as count_no_trans
from Visits
where visit_id not in (select visit_id from Transactions )
group by customer_id 
```
##125.银行账户概要2
>写一个 SQL,  报告余额高于 10000 的所有用户的名字和余额. 账户的余额等于包含该账户的所有交易的总和.
```
表: Users

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| account      | int     |
| name         | varchar |
+--------------+---------+
account 是该表的主键.
表中的每一行包含银行里中每一个用户的账号.
 

表: Transactions

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| trans_id      | int     |
| account       | int     |
| amount        | int     |
| transacted_on | date    |
+---------------+---------+
trans_id 是该表主键.
该表的每一行包含了所有账户的交易改变情况.
如果用户收到了钱, 那么金额是正的; 如果用户转了钱, 那么金额是负的.
所有账户的起始余额为 0.
 

写一个 SQL,  报告余额高于 10000 的所有用户的名字和余额. 账户的余额等于包含该账户的所有交易的总和.

返回结果表单没有顺序要求.

查询结果格式如下例所示.

 

Users table:
+------------+--------------+
| account    | name         |
+------------+--------------+
| 900001     | Alice        |
| 900002     | Bob          |
| 900003     | Charlie      |
+------------+--------------+

Transactions table:
+------------+------------+------------+---------------+
| trans_id   | account    | amount     | transacted_on |
+------------+------------+------------+---------------+
| 1          | 900001     | 7000       |  2020-08-01   |
| 2          | 900001     | 7000       |  2020-09-01   |
| 3          | 900001     | -3000      |  2020-09-02   |
| 4          | 900002     | 1000       |  2020-09-12   |
| 5          | 900003     | 6000       |  2020-08-07   |
| 6          | 900003     | 6000       |  2020-09-07   |
| 7          | 900003     | -4000      |  2020-09-11   |
+------------+------------+------------+---------------+

Result table:
+------------+------------+
| name       | balance    |
+------------+------------+
| Alice      | 11000      |
+------------+------------+
Alice 的余额为(7000 + 7000 - 3000) = 11000.
Bob 的余额为1000.
Charlie 的余额为(6000 + 6000 - 4000) = 8000.
```
```
--简单题
select
name  ,sum(amount) as balance
from 
Users t1 join  Transactions t2 on t1.account = t2.account
group by t1.account ,name  
having sum(amount) > 10000
```
##126.每位顾客最常购买的商品
>写一个 SQL 语句，找到每一个顾客最经常订购的商品。
结果表单应该有每一位至少下过一次单的顾客 customer_id , 他最经常订购的商品的 product_id 和 product_name。
```
表：Customers

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| name          | varchar |
+---------------+---------+
customer_id 是该表主键
该表包含所有顾客的信息
 

表：Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| order_date    | date    |
| customer_id   | int     |
| product_id    | int     |
+---------------+---------+
order_id 是该表主键
该表包含顾客 customer_id 的订单信息
没有顾客会在一天内订购相同的商品 多于一次
 

表：Products

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
| price         | int     |
+---------------+---------+
product_id 是该表主键
该表包含了所有商品的信息
 

写一个 SQL 语句，找到每一个顾客最经常订购的商品。

结果表单应该有每一位至少下过一次单的顾客 customer_id , 他最经常订购的商品的 product_id 和 product_name。

返回结果 没有顺序要求。

查询结果格式如下例所示：

Customers
+-------------+-------+
| customer_id | name  |
+-------------+-------+
| 1           | Alice |
| 2           | Bob   |
| 3           | Tom   |
| 4           | Jerry |
| 5           | John  |
+-------------+-------+

Orders
+----------+------------+-------------+------------+
| order_id | order_date | customer_id | product_id |
+----------+------------+-------------+------------+
| 1        | 2020-07-31 | 1           | 1          |
| 2        | 2020-07-30 | 2           | 2          |
| 3        | 2020-08-29 | 3           | 3          |
| 4        | 2020-07-29 | 4           | 1          |
| 5        | 2020-06-10 | 1           | 2          |
| 6        | 2020-08-01 | 2           | 1          |
| 7        | 2020-08-01 | 3           | 3          |
| 8        | 2020-08-03 | 1           | 2          |
| 9        | 2020-08-07 | 2           | 3          |
| 10       | 2020-07-15 | 1           | 2          |
+----------+------------+-------------+------------+

Products
+------------+--------------+-------+
| product_id | product_name | price |
+------------+--------------+-------+
| 1          | keyboard     | 120   |
| 2          | mouse        | 80    |
| 3          | screen       | 600   |
| 4          | hard disk    | 450   |
+------------+--------------+-------+
Result 表：
+-------------+------------+--------------+
| customer_id | product_id | product_name |
+-------------+------------+--------------+
| 1           | 2          | mouse        |
| 2           | 1          | keyboard     |
| 2           | 2          | mouse        |
| 2           | 3          | screen       |
| 3           | 3          | screen       |
| 4           | 1          | keyboard     |
+-------------+------------+--------------+

Alice (customer 1) 三次订购鼠标, 一次订购键盘, 所以鼠标是 Alice 最经常订购的商品.
Bob (customer 2) 一次订购键盘, 一次订购鼠标, 一次订购显示器, 所以这些都是 Bob 最经常订购的商品.
Tom (customer 3) 只两次订购显示器, 所以显示器是 Tom 最经常订购的商品.
Jerry (customer 4) 只一次订购键盘, 所以键盘是 Jerry 最经常订购的商品.
John (customer 5) 没有订购过商品, 所以我们并没有把 John 包含在结果表中.
```
```
--简单题,窗口写法
select
customer_id,product_id,product_name 
from 
(
select
t1.customer_id,t3.product_id,product_name,
rank() over(partition by t1.customer_id order by count(1) desc) as rn 
from
Customers t1
join Orders t2 on t1.customer_id = t2.customer_id
join Products t3 on t2.product_id = t3.product_id
group by t1.customer_id,t3.product_id,product_name
) t2
where rn =1
```
```
select customer_id, a.product_id, b.product_name from
    (select customer_id, product_id, count(product_id) ct from orders
    group by customer_id, product_id) a, products b
where a.product_id = b.product_id
and (customer_id,ct) in 
    (select customer_id, max(ct) from
        (select customer_id, count(product_id) ct from orders
        group by customer_id, product_id) tmp
    group by customer_id)
```
##127.没有卖出的卖家
>写一个SQL语句, 报告所有在2020年度没有任何卖出的卖家的名字.
返回结果按照 seller_name 升序排列.
```
表: Customer

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| customer_name | varchar |
+---------------+---------+
customer_id 是该表主键.
该表的每行包含网上商城的每一位顾客的信息.
 

表: Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| sale_date     | date    |
| order_cost    | int     |
| customer_id   | int     |
| seller_id     | int     |
+---------------+---------+
order_id 是该表主键.
该表的每行包含网上商城的所有订单的信息.
sale_date 是顾客customer_id和卖家seller_id之间交易的日期.
 

表: Seller

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| seller_id     | int     |
| seller_name   | varchar |
+---------------+---------+
seller_id 是该表主键.
该表的每行包含每一位卖家的信息.
 

写一个SQL语句, 报告所有在2020年度没有任何卖出的卖家的名字.

返回结果按照 seller_name 升序排列.

查询结果格式如下例所示.

 

Customer 表:
+--------------+---------------+
| customer_id  | customer_name |
+--------------+---------------+
| 101          | Alice         |
| 102          | Bob           |
| 103          | Charlie       |
+--------------+---------------+

Orders 表:
+-------------+------------+--------------+-------------+-------------+
| order_id    | sale_date  | order_cost   | customer_id | seller_id   |
+-------------+------------+--------------+-------------+-------------+
| 1           | 2020-03-01 | 1500         | 101         | 1           |
| 2           | 2020-05-25 | 2400         | 102         | 2           |
| 3           | 2019-05-25 | 800          | 101         | 3           |
| 4           | 2020-09-13 | 1000         | 103         | 2           |
| 5           | 2019-02-11 | 700          | 101         | 2           |
+-------------+------------+--------------+-------------+-------------+

Seller 表:
+-------------+-------------+
| seller_id   | seller_name |
+-------------+-------------+
| 1           | Daniel      |
| 2           | Elizabeth   |
| 3           | Frank       |
+-------------+-------------+

Result 表:
+-------------+
| seller_name |
+-------------+
| Frank       |
+-------------+
Daniel在2020年3月卖出1次.
Elizabeth在2020年卖出2次, 在2019年卖出1次.
Frank在2019年卖出1次, 在2020年没有卖出.
```
```
--简单题,考察where 和 on
select
seller_name
from 
Seller
where  seller_id not in 
(
select
seller_id 
from 
Orders
where left(sale_date,4) = "2020" 
)
order by seller_name
```
```
select seller_name
from Seller s
left join Orders o on o.seller_id= s.seller_id
and sale_date like '2020%'
where o.seller_id is null
order by seller_name
```
##128.找到遗失的id(recursive)
>写一个 SQL 语句, 找到所有遗失的顾客id. 遗失的顾客id是指那些不在 Customers 表中, 值却处于 1 和表中最大 customer_id 之间的id.
注意: 最大的 customer_id 值不会超过 100.
返回结果按 ids 升序排列

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
 

写一个 SQL 语句, 找到所有遗失的顾客id. 遗失的顾客id是指那些不在 Customers 表中, 值却处于 1 和表中最大 customer_id 之间的id.

注意: 最大的 customer_id 值不会超过 100.

返回结果按 ids 升序排列

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
>https://dev.mysql.com/doc/refman/8.0/en/with.html#common-table-expressions-recursive
mysql8.0 循环写法

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
##129.三人国家代表队
>有一个国家只有三所学校，这个国家的每一个学生只会注册一所学校。
这个国家正在参加一个竞赛，他们希望从这三所学校中各选出一个学生来组建一支三人的代表队。
例如：
member_A是从 SchoolA中选出的
member_B是从 SchoolB中选出的
member_C是从 SchoolC中选出的
被选中的学生具有不同的名字和ID（没有任何两个学生拥有相同的名字、没有任何两个学生拥有相同的ID）
使用上述条件，编写SQL查询语句来找到所有可能的三人国家代表队组合。

```
表: SchoolA

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
student_id 是表的主键
表中的每一行包含了学校A中每一个学生的名字和ID
所有student_name在表中都是独一无二的
 

表: SchoolB

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
student_id 是表的主键
表中的每一行包含了学校B中每一个学生的名字和ID
所有student_name在表中都是独一无二的
 

表: SchoolC

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
student_id 是表的主键
表中的每一行包含了学校C中每一个学生的名字和ID
所有student_name在表中都是独一无二的
 

有一个国家只有三所学校，这个国家的每一个学生只会注册一所学校。

这个国家正在参加一个竞赛，他们希望从这三所学校中各选出一个学生来组建一支三人的代表队。

例如：

member_A是从 SchoolA中选出的
member_B是从 SchoolB中选出的
member_C是从 SchoolC中选出的
被选中的学生具有不同的名字和ID（没有任何两个学生拥有相同的名字、没有任何两个学生拥有相同的ID）
使用上述条件，编写SQL查询语句来找到所有可能的三人国家代表队组合。

查询结果接受任何顺序。

 

查询结果格式样例：

SchoolA table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
+------------+--------------+

SchoolB table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 3          | Tom          |
+------------+--------------+

SchoolC table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 3          | Tom          |
| 2          | Jerry        |
| 10         | Alice        |
+------------+--------------+

预期结果:
+----------+----------+----------+
| member_A | member_B | member_C |
+----------+----------+----------+
| Alice    | Tom      | Jerry    |
| Bob      | Tom      | Alice    |
+----------+----------+----------+

让我们看看有哪些可能的组合：
- (Alice, Tom, Tom) --> 不适用，因为member_B（Tom）和member_C（Tom）有相同的名字和ID
- (Alice, Tom, Jerry) --> 可能的组合
- (Alice, Tom, Alice) --> 不适用，因为member_A和member_C有相同的名字
- (Bob, Tom, Tom) --> 不适用，因为member_B和member_C有相同的名字和ID
- (Bob, Tom, Jerry) --> 不适用，因为member_A和member_C有相同的ID
- (Bob, Tom, Alice) --> 可能的组合.
```
```
--简单题,注意不要写错,检查了好久
select a.student_name as member_A,b.student_name as member_B,c.student_name as member_C 
from SchoolA a,SchoolB b,SchoolC c
where a.student_id <> b.student_id 
and a.student_id <> c.student_id 
and b.student_id <> c.student_id 
and a.student_name <> b.student_name 
and a.student_name <> c.student_name 
and b.student_name <> c.student_name
```
##130.各赛事的用户注册率
>写一条 SQL 语句，查询各赛事的用户注册百分率，保留两位小数。
```
用户表： Users

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| user_id     | int     |
| user_name   | varchar |
+-------------+---------+
user_id 是该表的主键。
该表中的每行包括用户 ID 和用户名。
 

注册表： Register

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| contest_id  | int     |
| user_id     | int     |
+-------------+---------+
(contest_id, user_id) 是该表的主键。
该表中的每行包含用户的 ID 和他们注册的赛事。
 

写一条 SQL 语句，查询各赛事的用户注册百分率，保留两位小数。

返回的结果表按 percentage 的降序排序，若相同则按 contest_id 的升序排序。

查询结果如下示例所示：

 

Users 表：
+---------+-----------+
| user_id | user_name |
+---------+-----------+
| 6       | Alice     |
| 2       | Bob       |
| 7       | Alex      |
+---------+-----------+

Register 表：
+------------+---------+
| contest_id | user_id |
+------------+---------+
| 215        | 6       |
| 209        | 2       |
| 208        | 2       |
| 210        | 6       |
| 208        | 6       |
| 209        | 7       |
| 209        | 6       |
| 215        | 7       |
| 208        | 7       |
| 210        | 2       |
| 207        | 2       |
| 210        | 7       |
+------------+---------+

结果表：
+------------+------------+
| contest_id | percentage |
+------------+------------+
| 208        | 100.0      |
| 209        | 100.0      |
| 210        | 100.0      |
| 215        | 66.67      |
| 207        | 33.33      |
+------------+------------+
所有用户都注册了 208、209 和 210 赛事，因此这些赛事的注册率为 100% ，我们按 contest_id 的降序排序加入结果表中。
Alice 和 Alex 注册了 215 赛事，注册率为 ((2/3) * 100) = 66.67%
Bob 注册了 207 赛事，注册率为 ((1/3) * 100) = 33.33%
```
```
with x as (
select count(1) as t  from Users
)
select
contest_id,round(count(distinct user_id ) / (select x.t from x)  * 100,2) percentage
from 
Register 
group by  contest_id
order by percentage desc ,contest_id 
```
