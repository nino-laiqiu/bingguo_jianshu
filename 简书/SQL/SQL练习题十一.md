##111.周内的每天销售情况
>写一个SQL语句，报告 周内每天 每个商品类别下订购了多少单位
```
表：Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| customer_id   | int     |
| order_date    | date    | 
| item_id       | varchar |
| quantity      | int     |
+---------------+---------+
(order_id, item_id) 是该表主键
该表包含了订单信息
order_date 是id为 item_id 的商品被id为 customer_id 的消费者订购的日期.
表：Items

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| item_id             | varchar |
| item_name           | varchar |
| item_category       | varchar |
+---------------------+---------+
item_id 是该表主键
item_name 是商品的名字
item_category 是商品的类别
 

你是企业主，想要获得分类商品和周内每天的销售报告。

写一个SQL语句，报告 周内每天 每个商品类别下订购了多少单位。

返回结果表单 按商品类别排序 。

查询结果格式如下例所示：

 

Orders 表：
+------------+--------------+-------------+--------------+-------------+
| order_id   | customer_id  | order_date  | item_id      | quantity    |
+------------+--------------+-------------+--------------+-------------+
| 1          | 1            | 2020-06-01  | 1            | 10          |
| 2          | 1            | 2020-06-08  | 2            | 10          |
| 3          | 2            | 2020-06-02  | 1            | 5           |
| 4          | 3            | 2020-06-03  | 3            | 5           |
| 5          | 4            | 2020-06-04  | 4            | 1           |
| 6          | 4            | 2020-06-05  | 5            | 5           |
| 7          | 5            | 2020-06-05  | 1            | 10          |
| 8          | 5            | 2020-06-14  | 4            | 5           |
| 9          | 5            | 2020-06-21  | 3            | 5           |
+------------+--------------+-------------+--------------+-------------+

Items 表：
+------------+----------------+---------------+
| item_id    | item_name      | item_category |
+------------+----------------+---------------+
| 1          | LC Alg. Book   | Book          |
| 2          | LC DB. Book    | Book          |
| 3          | LC SmarthPhone | Phone         |
| 4          | LC Phone 2020  | Phone         |
| 5          | LC SmartGlass  | Glasses       |
| 6          | LC T-Shirt XL  | T-Shirt       |
+------------+----------------+---------------+

Result 表：
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| Category   | Monday    | Tuesday   | Wednesday | Thursday  | Friday    | Saturday  | Sunday    |
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| Book       | 20        | 5         | 0         | 0         | 10        | 0         | 0         |
| Glasses    | 0         | 0         | 0         | 0         | 5         | 0         | 0         |
| Phone      | 0         | 0         | 5         | 1         | 0         | 0         | 10        |
| T-Shirt    | 0         | 0         | 0         | 0         | 0         | 0         | 0         |
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
在周一(2020-06-01, 2020-06-08)，Book分类(ids: 1, 2)下，总共销售了20个单位(10 + 10)
在周二(2020-06-02)，Book分类(ids: 1, 2)下，总共销售了5个单位
在周三(2020-06-03)，Phone分类(ids: 3, 4)下，总共销售了5个单位
在周四(2020-06-04)，Phone分类(ids: 3, 4)下，总共销售了1个单位
在周五(2020-06-05)，Book分类(ids: 1, 2)下，总共销售了10个单位，Glasses分类(ids: 5)下，总共销售了5个单位
在周六, 没有商品销售
在周天(2020-06-14, 2020-06-21)，Phone分类(ids: 3, 4)下，总共销售了10个单位(5 + 5)
没有销售 T-Shirt 类别的商品
```
```
--简单题,注意怎么获取星期几,以及星期天是星期的第一天
//weekday dayname的使用
select
item_category Category,
ifnull(sum(if(date_format(order_date,"%w")=1,quantity ,0)),0) Monday ,
ifnull(sum(if(date_format(order_date,"%w")=2,quantity ,0)),0) Tuesday  ,
ifnull(sum(if(date_format(order_date,"%w")=3,quantity ,0)),0) Wednesday ,
ifnull(sum(if(date_format(order_date,"%w")=4,quantity ,0)),0) Thursday  ,
ifnull(sum(if(date_format(order_date,"%w")=5,quantity ,0)),0) Friday  ,
ifnull(sum(if(date_format(order_date,"%w")=6,quantity ,0)),0) Saturday  ,
ifnull(sum(if(date_format(order_date,"%w")=0,quantity ,0)),0) Sunday  
from 
Items t1 left join Orders t2 on  t1.item_id = t2.item_id
group by item_category
order by item_category
```
##112.按日期分组销售产品(group_concat)
>编写一个 SQL 查询来查找每个日期、销售的不同产品的数量及其名称。
每个日期的销售产品名称应按词典序排列。
返回按 sell_date 排序的结果表。
```
表 Activities：

+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| sell_date   | date    |
| product     | varchar |
+-------------+---------+
此表没有主键，它可能包含重复项。
此表的每一行都包含产品名称和在市场上销售的日期。
 

编写一个 SQL 查询来查找每个日期、销售的不同产品的数量及其名称。
每个日期的销售产品名称应按词典序排列。
返回按 sell_date 排序的结果表。


查询结果格式如下例所示。

Activities 表：
+------------+-------------+
| sell_date  | product     |
+------------+-------------+
| 2020-05-30 | Headphone   |
| 2020-06-01 | Pencil      |
| 2020-06-02 | Mask        |
| 2020-05-30 | Basketball  |
| 2020-06-01 | Bible       |
| 2020-06-02 | Mask        |
| 2020-05-30 | T-Shirt     |
+------------+-------------+

Result 表：
+------------+----------+------------------------------+
| sell_date  | num_sold | products                     |
+------------+----------+------------------------------+
| 2020-05-30 | 3        | Basketball,Headphone,T-shirt |
| 2020-06-01 | 2        | Bible,Pencil                 |
| 2020-06-02 | 1        | Mask                         |
+------------+----------+------------------------------+
对于2020-05-30，出售的物品是 (Headphone, Basketball, T-shirt)，按词典序排列，并用逗号 ',' 分隔。
对于2020-06-01，出售的物品是 (Pencil, Bible)，按词典序排列，并用逗号分隔。
对于2020-06-02，出售的物品是 (Mask)，只需返回该物品名。
```
```
--难题,没用过group_concat函数,注意取别名类似于hive中的炸裂函数
select
sell_date,count(distinct product) num_sold,
group_concat(distinct product separator ",") products 
from 
(
select
sell_date,product
from
Activities a 
order by sell_date,product
) t1
group by sell_date
order  by sell_date
```
```
--注意也可以在函数中直接排序
select sell_date, count(distinct product) num_sold,
group_concat(distinct product order by product asc separator ',') as products
from activities
group by sell_date
```
##113.上月播放的儿童适宜电影
>写一个 SQL 语句,  报告在 2020 年 6 月份播放的儿童适宜电影的去重电影名
```
表: TVProgram

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| program_date  | date    |
| content_id    | int     |
| channel       | varchar |
+---------------+---------+
(program_date, content_id) 是该表主键.
该表包含电视上的节目信息.
content_id 是电视一些频道上的节目的 id.
 

表: Content

+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| content_id       | varchar |
| title            | varchar |
| Kids_content     | enum    |
| content_type     | varchar |
+------------------+---------+
content_id 是该表主键.
Kids_content 是枚举类型, 取值为('Y', 'N'), 其中: 
'Y' 表示儿童适宜内容, 而'N'表示儿童不宜内容.
content_type 表示内容的类型, 比如电影, 电视剧等.
 

写一个 SQL 语句,  报告在 2020 年 6 月份播放的儿童适宜电影的去重电影名.

返回的结果表单没有顺序要求.

查询结果的格式如下例所示.

 

TVProgram 表:
+--------------------+--------------+-------------+
| program_date       | content_id   | channel     |
+--------------------+--------------+-------------+
| 2020-06-10 08:00   | 1            | LC-Channel  |
| 2020-05-11 12:00   | 2            | LC-Channel  |
| 2020-05-12 12:00   | 3            | LC-Channel  |
| 2020-05-13 14:00   | 4            | Disney Ch   |
| 2020-06-18 14:00   | 4            | Disney Ch   |
| 2020-07-15 16:00   | 5            | Disney Ch   |
+--------------------+--------------+-------------+

Content 表:
+------------+----------------+---------------+---------------+
| content_id | title          | Kids_content  | content_type  |
+------------+----------------+---------------+---------------+
| 1          | Leetcode Movie | N             | Movies        |
| 2          | Alg. for Kids  | Y             | Series        |
| 3          | Database Sols  | N             | Series        |
| 4          | Aladdin        | Y             | Movies        |
| 5          | Cinderella     | Y             | Movies        |
+------------+----------------+---------------+---------------+

Result 表:
+--------------+
| title        |
+--------------+
| Aladdin      |
+--------------+
"Leetcode Movie" 是儿童不宜的电影.
"Alg. for Kids" 不是电影.
"Database Sols" 不是电影
"Alladin" 是电影, 儿童适宜, 并且在 2020 年 6 月份播放.
"Cinderella" 不在 2020 年 6 月份播放
```
```
--简单题,注意使用like要加%的使用
select
distinct title
from
(
select
content_id 
from  
TVProgram t1
where program_date like "2020-06%"
) t1 
join 
(
select
content_id, title 
from  
Content
where  content_type = "Movies"  and Kids_content = "Y"
) t2 on t1.content_id = t2.content_id
```
```
select title from content 
where kids_content = "Y" and content_type = "Movies" 
and content_id in
(select content_id from tvprogram where program_date like "2020-06%")
```
注意查询枚举类型的效率问题
##114.可以放心投资的国家(总体平均值)
>一家电信公司想要投资新的国家. 该公司想要投资的国家是:  该国的平均通话时长要严格地大于全球平均通话时长
写一段 SQL,  找到所有该公司可以投资的国家
```
表 Person:

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| id             | int     |
| name           | varchar |
| phone_number   | varchar |
+----------------+---------+
id 是该表主键.
该表每一行包含一个人的名字和电话号码.
电话号码的格式是:'xxx-yyyyyyy', 其中xxx是国家码(3个字符), yyyyyyy是电话号码(7个字符), x和y都表示数字. 同时, 国家码和电话号码都可以包含前导0.
表 Country:

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| name           | varchar |
| country_code   | varchar |
+----------------+---------+
country_code是该表主键.
该表每一行包含国家名和国家码. country_code的格式是'xxx', x是数字.
 

表 Calls:

+-------------+------+
| Column Name | Type |
+-------------+------+
| caller_id   | int  |
| callee_id   | int  |
| duration    | int  |
+-------------+------+
该表无主键, 可能包含重复行.
每一行包含呼叫方id, 被呼叫方id和以分钟为单位的通话时长. caller_id != callee_id
一家电信公司想要投资新的国家. 该公司想要投资的国家是:  该国的平均通话时长要严格地大于全球平均通话时长.

写一段 SQL,  找到所有该公司可以投资的国家.

返回的结果表没有顺序要求.

查询的结果格式如下例所示.

Person 表:
+----+----------+--------------+
| id | name     | phone_number |
+----+----------+--------------+
| 3  | Jonathan | 051-1234567  |
| 12 | Elvis    | 051-7654321  |
| 1  | Moncef   | 212-1234567  |
| 2  | Maroua   | 212-6523651  |
| 7  | Meir     | 972-1234567  |
| 9  | Rachel   | 972-0011100  |
+----+----------+--------------+

Country 表:
+----------+--------------+
| name     | country_code |
+----------+--------------+
| Peru     | 051          |
| Israel   | 972          |
| Morocco  | 212          |
| Germany  | 049          |
| Ethiopia | 251          |
+----------+--------------+

Calls 表:
+-----------+-----------+----------+
| caller_id | callee_id | duration |
+-----------+-----------+----------+
| 1         | 9         | 33       |
| 2         | 9         | 4        |
| 1         | 2         | 59       |
| 3         | 12        | 102      |
| 3         | 12        | 330      |
| 12        | 3         | 5        |
| 7         | 9         | 13       |
| 7         | 1         | 3        |
| 9         | 7         | 1        |
| 1         | 7         | 7        |
+-----------+-----------+----------+

Result 表:
+----------+
| country  |
+----------+
| Peru     |
+----------+
国家Peru的平均通话时长是 (102 + 102 + 330 + 330 + 5 + 5) / 6 = 145.666667
国家Israel的平均通话时长是 (33 + 4 + 13 + 13 + 3 + 1 + 1 + 7) / 8 = 9.37500
国家Morocco的平均通话时长是 (33 + 4 + 59 + 59 + 3 + 7) / 6 = 27.5000 
全球平均通话时长 = (2 * (33 + 3 + 59 + 102 + 330 + 5 + 13 + 3 + 1 + 7)) / 20 = 55.70000
所以, Peru是唯一的平均通话时长大于全球平均通话时长的国家, 也是唯一的推荐投资的国家.
```
```
--简单题,我写的麻烦,看了下评论区事实上求总体的平均值只需要对表avg即可,不需要union all.....
with x as (
select
name,id 
from
Country t1
left join 
(
select
id,left(phone_number,3) as country_code
from
Person 
) t2 on t1.country_code = t2.country_code ) 

select
distinct name  country 
from 
(
select
name,
avg(duration) over(partition by name ) as avg_1,
avg(duration) over() as avg_2  
from
( 
select
name,duration
from  
Calls
join x on Calls.caller_id = x.id
union all 
(
select
name,duration 
from 
Calls
join x on Calls.callee_id  = x.id    
) 
) s2 
) s3
where avg_1 > avg_2
```
```
with cte as (
select ct.name, avg(cl.duration) as avg_country
from person a 
       join country ct on left(a.phone_number, 3)=ct.country_code
       join calls cl on a.id=cl.caller_id or a.id=cl.callee_id
group by ct.name)
select name as country
from cte
where avg_country > (select avg(duration) from calls)
```
>注意在join中使用  or ....这样就不需要union all了...
##115.消费者下单频率
>写一个 SQL 语句,  报告消费者的 id 和名字, 其中消费者在 2020 年 6 月和 7 月, 每月至少花费了$100.
```
表: Customers

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| name          | varchar |
| country       | varchar |
+---------------+---------+
customer_id 是该表主键.
该表包含公司消费者的信息.
 

表: Product

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| description   | varchar |
| price         | int     |
+---------------+---------+
product_id 是该表主键.
该表包含公司产品的信息.
price 是本产品的花销.
 

表: Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| customer_id   | int     |
| product_id    | int     |
| order_date    | date    |
| quantity      | int     |
+---------------+---------+
order_id 是该表主键.
该表包含消费者下单的信息.
customer_id 是买了数量为"quantity", id为"product_id"产品的消费者的 id.
Order_date 是订单发货的日期, 格式为('YYYY-MM-DD').
 

写一个 SQL 语句,  报告消费者的 id 和名字, 其中消费者在 2020 年 6 月和 7 月, 每月至少花费了$100.

结果表无顺序要求.

查询结果格式如下例所示.

 

Customers
+--------------+-----------+-------------+
| customer_id  | name      | country     |
+--------------+-----------+-------------+
| 1            | Winston   | USA         |
| 2            | Jonathan  | Peru        |
| 3            | Moustafa  | Egypt       |
+--------------+-----------+-------------+

Product
+--------------+-------------+-------------+
| product_id   | description | price       |
+--------------+-------------+-------------+
| 10           | LC Phone    | 300         |
| 20           | LC T-Shirt  | 10          |
| 30           | LC Book     | 45          |
| 40           | LC Keychain | 2           |
+--------------+-------------+-------------+

Orders
+--------------+-------------+-------------+-------------+-----------+
| order_id     | customer_id | product_id  | order_date  | quantity  |
+--------------+-------------+-------------+-------------+-----------+
| 1            | 1           | 10          | 2020-06-10  | 1         |
| 2            | 1           | 20          | 2020-07-01  | 1         |
| 3            | 1           | 30          | 2020-07-08  | 2         |
| 4            | 2           | 10          | 2020-06-15  | 2         |
| 5            | 2           | 40          | 2020-07-01  | 10        |
| 6            | 3           | 20          | 2020-06-24  | 2         |
| 7            | 3           | 30          | 2020-06-25  | 2         |
| 9            | 3           | 30          | 2020-05-08  | 3         |
+--------------+-------------+-------------+-------------+-----------+

Result 表:
+--------------+------------+
| customer_id  | name       |  
+--------------+------------+
| 1            | Winston    |
+--------------+------------+ 
Winston 在2020年6月花费了$300(300 * 1), 在7月花费了$100(10 * 1 + 45 * 2).
Jonathan 在2020年6月花费了$600(300 * 2), 在7月花费了$20(2 * 10).
Moustafa 在2020年6月花费了$110 (10 * 2 + 45 * 2), 在7月花费了$0
```
```
--简单题,在having判断之前先进行过滤是否效率会快一点?发现速度反而慢了.....
select
t1.customer_id,name 
from 
Customers t1 join Orders t2 on t1.customer_id = t2.customer_id
             join Product t3 on t2.product_id  = t3.product_id 
//where  month(order_date) = 6 or month(order_date) = 7
group by t1.customer_id 
having sum(if(left(order_date,7)="2020-06",price * quantity ,0 )) >= 100
       and    
       sum(if(left(order_date,7)="2020-07",price * quantity ,0 )) >= 100  
```
##116.查找拥有有效邮箱的用户(正则匹配)
>写一条 SQL 语句，查询拥有有效邮箱的用户
有效的邮箱包含符合下列条件的前缀名和域名：
前缀名是包含字母（大写或小写）、数字、下划线 '_'、句点 '.' 和/或横杠 '-' 的字符串。前缀名必须以字母开头。
域名是 '@leetcode.com' 

```
用户表： Users

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| name          | varchar |
| mail          | varchar | 
+---------------+---------+
user_id （用户 ID）是该表的主键。
这个表包含用户在某网站上注册的信息。有些邮箱是无效的。
 

写一条 SQL 语句，查询拥有有效邮箱的用户。

有效的邮箱包含符合下列条件的前缀名和域名：

前缀名是包含字母（大写或小写）、数字、下划线 '_'、句点 '.' 和/或横杠 '-' 的字符串。前缀名必须以字母开头。
域名是 '@leetcode.com' 。
按任意顺序返回结果表。

 

查询格式如下所示：

Users
+---------+-----------+-------------------------+
| user_id | name      | mail                    |
+---------+-----------+-------------------------+
| 1       | Winston   | winston@leetcode.com    |
| 2       | Jonathan  | jonathanisgreat         |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |
| 5       | Marwan    | quarz#2020@leetcode.com |
| 6       | David     | david69@gmail.com       |
| 7       | Shapiro   | .shapo@leetcode.com     |
+---------+-----------+-------------------------+

结果表：
+---------+-----------+-------------------------+
| user_id | name      | mail                    |
+---------+-----------+-------------------------+
| 1       | Winston   | winston@leetcode.com    |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |
+---------+-----------+-------------------------+
2 号用户的邮箱没有域名。
5 号用户的邮箱包含非法字符 #。
6 号用户的邮箱的域名不是 leetcode。
7 号用户的邮箱以句点（.）开头。
```
```
--正则匹配,非常讨厌老忘记符号的含义....
select *
from users 
where mail regexp '^[a-zA-Z]+[a-zA-Z0-9_\\./\\-]*@leetcode\\.com$'
```

##117.患某疾病的患者
>写一条 SQL 语句，查询患有 I 类糖尿病的患者 ID （patient_id）、患者姓名（patient_name）以及其患有的所有疾病代码（conditions）。I 类糖尿病的代码总是包含前缀 DIAB1 。

```
患者信息表： Patients

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| patient_id   | int     |
| patient_name | varchar |
| conditions   | varchar |
+--------------+---------+
patient_id （患者 ID）是该表的主键。
'conditions' （疾病）包含 0 个或以上的疾病代码，以空格分隔。
这个表包含医院中患者的信息。
 

写一条 SQL 语句，查询患有 I 类糖尿病的患者 ID （patient_id）、患者姓名（patient_name）以及其患有的所有疾病代码（conditions）。I 类糖尿病的代码总是包含前缀 DIAB1 。

按任意顺序返回结果表。

查询结果格式如下示例所示：

 

Patients
+------------+--------------+--------------+
| patient_id | patient_name | conditions   |
+------------+--------------+--------------+
| 1          | Daniel       | YFEV COUGH   |
| 2          | Alice        |              |
| 3          | Bob          | DIAB100 MYOP |
| 4          | George       | ACNE DIAB100 |
| 5          | Alain        | DIAB201      |
+------------+--------------+--------------+

结果表：
+------------+--------------+--------------+
| patient_id | patient_name | conditions   |
+------------+--------------+--------------+
| 3          | Bob          | DIAB100 MYOP |
| 4          | George       | ACNE DIAB100 | 
+------------+--------------+--------------+
Bob 和 George 都患有代码以 DIAB1 开头的疾病
```
```
--like用法,有一个通过不了限制一下....
SELECT 
    patient_id,
    patient_name,
    conditions
FROM Patients
WHERE conditions LIKE '%DIAB1%' AND conditions NOT LIKE 'SADIAB100'
```
```
--正确的写法
select * from Patients where conditions  REGEXP '^DIAB1|\\sDIAB1'
```
```
select * from patients where conditions like "DIAB1%" or conditions like "% DIAB1%"
```
##118.最近的三笔订单
>写一个 SQL 语句，找到每个用户的最近三笔订单。如果用户的订单少于 3 笔，则返回他的全部订单。
```
表：Customers

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| name          | varchar |
+---------------+---------+
customer_id 是该表主键
该表包含消费者的信息
 

表：Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| order_date    | date    |
| customer_id   | int     |
| cost          | int     |
+---------------+---------+
order_id 是该表主键
该表包含id为customer_id的消费者的订单信息
每一个消费者 每天一笔订单
 

写一个 SQL 语句，找到每个用户的最近三笔订单。如果用户的订单少于 3 笔，则返回他的全部订单。

返回的结果按照 customer_name 升序排列。如果排名有相同，则继续按照 customer_id 升序排列。如果排名还有相同，则继续按照 order_date 降序排列。

查询结果格式如下例所示：

Customers
+-------------+-----------+
| customer_id | name      |
+-------------+-----------+
| 1           | Winston   |
| 2           | Jonathan  |
| 3           | Annabelle |
| 4           | Marwan    |
| 5           | Khaled    |
+-------------+-----------+

Orders
+----------+------------+-------------+------+
| order_id | order_date | customer_id | cost |
+----------+------------+-------------+------+
| 1        | 2020-07-31 | 1           | 30   |
| 2        | 2020-07-30 | 2           | 40   |
| 3        | 2020-07-31 | 3           | 70   |
| 4        | 2020-07-29 | 4           | 100  |
| 5        | 2020-06-10 | 1           | 1010 |
| 6        | 2020-08-01 | 2           | 102  |
| 7        | 2020-08-01 | 3           | 111  |
| 8        | 2020-08-03 | 1           | 99   |
| 9        | 2020-08-07 | 2           | 32   |
| 10       | 2020-07-15 | 1           | 2    |
+----------+------------+-------------+------+

Result table：
+---------------+-------------+----------+------------+
| customer_name | customer_id | order_id | order_date |
+---------------+-------------+----------+------------+
| Annabelle     | 3           | 7        | 2020-08-01 |
| Annabelle     | 3           | 3        | 2020-07-31 |
| Jonathan      | 2           | 9        | 2020-08-07 |
| Jonathan      | 2           | 6        | 2020-08-01 |
| Jonathan      | 2           | 2        | 2020-07-30 |
| Marwan        | 4           | 4        | 2020-07-29 |
| Winston       | 1           | 8        | 2020-08-03 |
| Winston       | 1           | 1        | 2020-07-31 |
| Winston       | 1           | 10       | 2020-07-15 |
+---------------+-------------+----------+------------+
Winston 有 4 笔订单, 排除了 "2020-06-10" 的订单, 因为它是最老的订单。
Annabelle 只有 2 笔订单, 全部返回。
Jonathan 恰好有 3 笔订单。
Marwan 只有 1 笔订单。
结果表我们按照 customer_name 升序排列，customer_id 升序排列，order_date 降序排列。
```
```
--简单题,窗口写法
--注意不能以name为分组,因为名字会重复...要以id.....
select 
name customer_name,
customer_id,
order_id,
order_date
from 
(
select
name,
t1.customer_id,
order_id,
order_date,
dense_rank() over(partition by t1.customer_id order by order_date desc ) as rn 
from Customers t1 join Orders t2 on t1.customer_id = t2.customer_id
) t1
where rn <=3
order by customer_name ,customer_id,order_date desc 
```
```
SELECT c.name customer_name,o2.customer_id,o2.order_id,o2.order_date
FROM orders o1,orders o2,Customers c
WHERE o1.customer_id=o2.customer_id AND c.customer_id=o2.customer_id
GROUP BY c.name,o2.order_id,o2.customer_id,o2.order_date
HAVING SUM(o1.order_date>=o2.order_date)<=3
ORDER BY c.name,o2.customer_id,o2.order_date desc
```
##119.产品名称格式修复(内容处理函数)
>写一个 SQL 语句报告每个月的销售情况：
product_name 是小写字母且不包含前后空格
sale_date 格式为 ('YYYY-MM') 
total 是产品在本月销售的次数

```
表：Sales

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| sale_id      | int     |
| product_name | varchar |
| sale_date    | date    |
+--------------+---------+
sale_id 是该表主键
该表的每一行包含了产品的名称及其销售日期
因为在 2000 年该表是手工填写的，product_name 可能包含前后空格，而且包含大小写。

写一个 SQL 语句报告每个月的销售情况：

product_name 是小写字母且不包含前后空格
sale_date 格式为 ('YYYY-MM') 
total 是产品在本月销售的次数
返回结果以 product_name 升序 排列，如果有排名相同，再以 sale_date 升序 排列。

查询结果格式如下所示：

Sales 表：
+------------+------------------+--------------+
| sale_id    | product_name     | sale_date    |
+------------+------------------+--------------+
| 1          |      LCPHONE     | 2000-01-16   |
| 2          |    LCPhone       | 2000-01-17   |
| 3          |     LcPhOnE      | 2000-02-18   |
| 4          |      LCKeyCHAiN  | 2000-02-19   |
| 5          |   LCKeyChain     | 2000-02-28   |
| 6          | Matryoshka       | 2000-03-31   | 
+------------+------------------+--------------+

Result 表：
+--------------+--------------+----------+
| product_name | sale_date    | total    |
+--------------+--------------+----------+
| lcphone      | 2000-01      | 2        |
| lckeychain   | 2000-02      | 2        | 
| lcphone      | 2000-02      | 1        | 
| matryoshka   | 2000-03      | 1        | 
+--------------+--------------+----------+

1 月份，卖了 2 个 LcPhones，请注意产品名称是小写的，中间可能包含空格
2 月份，卖了 2 个 LCKeychains 和 1 个 LCPhone
3 月份，卖了 1 个 matryoshka
```
```
--简单题,注意对数据的处理函数的学习,是我的薄弱点,
--replace 可用trim函数来代替
select
replace(lower(product_name),' ','') product_name ,left(sale_date,7) sale_date ,
count(1) total
from 
Sales
group by replace(lower(product_name),' ','') ,left(sale_date,7)
order by product_name,sale_date
```

##120.每件商品的最新订单
>写一个SQL 语句, 找到每件商品的最新订单(可能有多个)
返回的结果以 product_name 升序排列, 如果有排序相同, 再以 product_id 升序排列. 如果还有排序相同, 再以 order_id 升序排列
```
表: Customers

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| name          | varchar |
+---------------+---------+
customer_id 是该表主键.
该表包含消费者的信息.
 

表: Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| order_date    | date    |
| customer_id   | int     |
| product_id    | int     |
+---------------+---------+
order_id 是该表主键.
该表包含消费者customer_id产生的订单.
不会有商品被相同的用户在一天内下单超过一次.
 

表: Products

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
| price         | int     |
+---------------+---------+
product_id 是该表主键.
该表包含所有商品的信息.
 

写一个SQL 语句, 找到每件商品的最新订单(可能有多个).

返回的结果以 product_name 升序排列, 如果有排序相同, 再以 product_id 升序排列. 如果还有排序相同, 再以 order_id 升序排列.

查询结果格式如下例所示:

Customers
+-------------+-----------+
| customer_id | name      |
+-------------+-----------+
| 1           | Winston   |
| 2           | Jonathan  |
| 3           | Annabelle |
| 4           | Marwan    |
| 5           | Khaled    |
+-------------+-----------+

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
| 7        | 2020-08-01 | 3           | 1          |
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

Result
+--------------+------------+----------+------------+
| product_name | product_id | order_id | order_date |
+--------------+------------+----------+------------+
| keyboard     | 1          | 6        | 2020-08-01 |
| keyboard     | 1          | 7        | 2020-08-01 |
| mouse        | 2          | 8        | 2020-08-03 |
| screen       | 3          | 3        | 2020-08-29 |
+--------------+------------+----------+------------+
keyboard 的最新订单在2020-08-01, 在这天有两次下单
mouse 的最新订单在2020-08-03, 在这天只有一次下单
screen 的最新订单在2020-08-29, 在这天只有一次下单
hard disk 没有被下单, 我们不把它包含在结果表中
```
```
--简单题,窗口写法
select
product_name,
product_id,
order_id,
order_date 
from 
(
select
product_name,
t3.product_id,
order_id,
order_date,
dense_rank() over(partition by t3.product_id order by order_date desc ) as rn 
from
Customers t1
join Orders t2 on t1.customer_id = t2.customer_id
join Products t3 on t2.product_id = t3.product_id
) t4
where rn =1
order by product_name,product_id,order_id
```
```
SELECT
	p.product_name AS product_name,
	o2.product_id,
	o2.order_id,
	o2.order_date
FROM
	Orders o1,
	Orders o2,
	Customers c,
	Products p
WHERE
	o1.product_id = o2.product_id
AND o1.order_date >= o2.order_date
AND o2.customer_id = c.customer_id
AND o2.product_id = p.product_id
GROUP BY
	p.product_name,
	o2.product_id,
	o2.order_id,
	o2.order_date
HAVING
	COUNT(DISTINCT o1.order_date) = 1
```
