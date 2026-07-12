##11.游戏玩法分析1

```
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
```

```
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |

```

>每行数据记录了一名玩家在退出平台之前，当天使用同一台设备登录平台后打开的游戏的数目（可能是 0 个）,写一条 SQL 查询语句获取每位玩家 第一次登陆平台的日期。

```
-- 我的mysql答案
select
player_id,
min(event_date) as first_login 
from
Activity t
group  by   player_id
order by player_id 
```
```
-- 注意Oracle中的to_char函数
SELECT player_id, MIN(to_char(event_date, 'yyyy-MM-dd')) AS first_login
FROM Activity
GROUP BY player_id
```
考虑不确定的情况
```
select
	player_id,
	to_char(event_date, 'yyyy-mm-dd') as first_login
from
	(select
		player_id,
		event_date,
		row_number() over (partition by player_id order by event_date asc) rn
	from Activity)
where rn = 1
```

##12.游戏玩法分析二(row_number() 和dense_rank() 的效率)

>请编写一个 SQL 查询，描述每一个玩家首次登陆的设备名称

```
-- 我的Oracle答案
with x as (
select
player_id , device_id,
row_number() over(partition by player_id order by event_date) as rn
from
Activity t
) 
select
x.player_id,x.device_id
from 
x
where x.rn =1
```

```
-- 使用子查询
select player_id, device_id
from Activity
where (player_id, event_date) in (select player_id, min(event_date)
                                  from Activity
                                  group by player_id);

```

#####错误的写法:子查询的id没有作用到父查询中
```
select
    player_id, device_id
from 
    Activity
where 
    event_date in 
    (
        select min(event_date) as event_date
        from Activity
        group by player_id
    )
```

##13.游戏玩法分析三(sum窗口函数自连接的实现)

>编写一个 SQL 查询，同时报告每组玩家和日期，以及玩家到目前为止玩了多少游戏。也就是说，在此日期之前玩家所玩的游戏总数。详细情况请查看示例

```
Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 1         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+-----------+------------+---------------------+
| player_id | event_date | games_played_so_far |
+-----------+------------+---------------------+
| 1         | 2016-03-01 | 5                   |
| 1         | 2016-05-02 | 11                  |
| 1         | 2017-06-25 | 12                  |
| 3         | 2016-03-02 | 0                   |
| 3         | 2018-07-03 | 5                   |
+-----------+------------+---------------------+
对于 ID 为 1 的玩家，2016-05-02 共玩了 5+6=11 个游戏，2017-06-25 共玩了 5+6+1=12 个游戏。
对于 ID 为 3 的玩家，2018-07-03 共玩了 0+5=5 个游戏。
请注意，对于每个玩家，我们只关心玩家的登录日期
```

```
-- 我的窗口写法
select 
player_id,to_char(event_date,'yyyy-mm-dd') as event_date,
sum(games_played) over(partition by player_id order by event_date ) as games_played_so_far
from
Activity
```

```
--自连接
SELECT
	a.player_id,
	a.event_date,
	SUM(b.games_played) AS games_played_so_far
FROM
	Activity AS a
	INNER JOIN Activity AS b
	ON a.player_id = b.player_id AND b.event_date <= a.event_date
GROUP BY
	a.player_id, 
	a.event_date
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-cc6644a6c5f3657b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##14.游戏的玩法四

>编写一个 SQL 查询，报告在首次登录的第二天再次登录的玩家的比率，四舍五入到小数点后两位。换句话说，您需要计算从首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数

反思看错题目了,题目只是说首次登录的第二天是否登录了,我使用的是lag函数开窗解决的,然后去重,事实上只要求最小的登录时间即可.....

```
-- 我的写法2
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
-- 使用avg也可
select round(avg(a.event_date is not null), 2) fraction
from 
    (select player_id, min(event_date) as login
    from activity
    group by player_id) p 
left join activity a 
on p.player_id=a.player_id and datediff(a.event_date, p.login)=1
```

##15.中位数

```
+-----+------------+--------+
|Id   | Company    | Salary |
+-----+------------+--------+
|1    | A          | 2341   |
|2    | A          | 341    |
|3    | A          | 15     |
|4    | A          | 15314  |
|5    | A          | 451    |
|6    | A          | 513    |
|7    | B          | 15     |
|8    | B          | 13     |
|9    | B          | 1154   |
|10   | B          | 1345   |
|11   | B          | 1221   |
|12   | B          | 234    |
|13   | C          | 2345   |
|14   | C          | 2645   |
|15   | C          | 2645   |
|16   | C          | 2652   |
|17   | C          | 65     |
+-----+------------+--------+
```
>请编写SQL查询来查找每个公司的薪水中位数
```
select Id,Company,Salary
from
(
select Id,Company,Salary, 
row_number()over(partition by Company order by Salary)as ranking,
count(Id) over(partition by Company)as cnt
from Employee
)a
where ranking>=cnt/2 and ranking<=cnt/2+1
```
使用两个序列函数,发现问题,用spark检验一下,问题出在C是单数
```
+---+-------+------+------+-------+
|Id |Company|Salary|asc_rn|desc_rn|
+---+-------+------+------+-------+
|12 |B      |234   |6     |1      |
|7  |B      |15    |5     |2      |
|10 |B      |1345  |4     |3      |
|8  |B      |13    |3     |4      |
|11 |B      |1221  |2     |5      |
|9  |B      |1154  |1     |6      |
|17 |C      |65    |5     |1      |
|16 |C      |2652  |4     |2      |
|14 |C      |2645  |2     |3      |
|15 |C      |2645  |3     |4      |
|13 |C      |2345  |1     |5      |
|6  |A      |513   |6     |1      |
|5  |A      |451   |5     |2      |
|2  |A      |341   |4     |3      |
|1  |A      |2341  |3     |4      |
|4  |A      |15314 |2     |5      |
|3  |A      |15    |1     |6      |
+---+-------+------+------+-------+
```
正确的写法是添加ID字段的排序,由于薪资的在题目中是重复的使用两个序列函数只比较一个字段的化,会乱序,因为一个字段值相同就会比较下一个字段,而默认的比较方法是降序的
```
with x as(
select 
Id,Company,Salary,
row_number() over(partition by Company order by Salary asc ,Id asc ) as asc_rn,
row_number() over(partition by Company order by Salary desc ,Id desc) as desc_rn,
count(1) over(partition by Company ) as  cnt 
from
Employee e
)
select 
Id,Company,Salary
from 
x
where (mod(cnt,2) = 1 and asc_rn = desc_rn ) or (mod(cnt,2)=0 and abs(asc_rn-desc_rn) =1)
```
这道题目还未完成还有补充的空间:例如使用自连接怎么优化它

##16.至少有5名直接下属的经理


```
Employee 表包含所有员工和他们的经理。每个员工都有一个 Id，并且还有一列是经理的 Id。
+------+----------+-----------+----------+
|Id    |Name 	  |Department |ManagerId |
+------+----------+-----------+----------+
|101   |John 	  |A 	      |null      |
|102   |Dan 	  |A 	      |101       |
|103   |James 	  |A 	      |101       |
|104   |Amy 	  |A 	      |101       |
|105   |Anne 	  |A 	      |101       |
|106   |Ron 	  |B 	      |101       |
+------+----------+-----------+----------+
```
>给定 Employee 表，请编写一个SQL查询来查找至少有5名直接下属的经理。对于上表，您的SQL查询应该返回：
```
+-------+
| Name  |
+-------+
| John  |
+-------+
```

```
-- 我的写法,我习惯使用with,在大数据领域with 不一定全部在内存中可配置参数到磁盘
with x as (
select
e1.Name as Employee_name ,e2.Name as Manager_name
from
Employee e1
join Employee e2
on e1.ManagerId = e2.Id
)
select 
Manager_name as Name
from 
x
group by Manager_name
having count(1) >=5
```
其他解法,类似可以我的解法,一步完成,这道题目的坑在于null的处理使用join内连接

##17.给定数字的频率查询中位数(两个序列函数)

```
Numbers 表保存数字的值及其频率。
+----------+-------------+
|  Number  |  Frequency  |
+----------+-------------|
|  0       |  7          |
|  1       |  1          |
|  2       |  3          |
|  3       |  1          |
+----------+-------------+

```

```
在此表中，数字为 0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3，所以中位数是 (0 + 0) / 2 = 0
+--------+
| median |
+--------|
| 0.0000 |
+--------+
```

```
--我的解法,还是上面的方法中位数的两个序列函数排序思想
select avg(number) as median
from
(select Number, frequency,
        sum(frequency) over(order by number asc) as total,
        sum(frequency) over(order by number desc) as total1
from Numbers
order by number asc)as a
where total>=(select sum(frequency) from Numbers)/2
and total1>=(select sum(frequency) from Numbers)/2
```

```
select
    avg(n.Number) as median
from
    Numbers as n
where
    n.Frequency >= abs(
        (select sum(Frequency) from Numbers where Number <= n.Number) - 
        (select sum(Frequency) from Numbers where Number >= n.Number)
    )
```
```
-- 理解起来有点难度...
SELECT 
AVG(Number)median 
FROM
(SELECT n1.Number FROM Numbers n1 JOIN Numbers n2 ON n1.Number>=n2.Number 
 GROUP BY 
 n1.Number 
 HAVING 
 SUM(n2.Frequency)>=(SELECT SUM(Frequency) FROM Numbers)/2 
 AND 
 SUM(n2.Frequency)-AVG(n1.Frequency)<=(SELECT SUM(Frequency) FROM Numbers)/2
)s
```

##18.当选者
```
+-----+---------+
| id  | Name    |
+-----+---------+
| 1   | A       |
| 2   | B       |
| 3   | C       |
| 4   | D       |
| 5   | E       |
+-----+---------+  
```
```
+-----+--------------+
| id  | CandidateId  |
+-----+--------------+
| 1   |     2        |
| 2   |     4        |
| 3   |     3        |
| 4   |     2        |
| 5   |     5        |
+-----+--------------+
id 是自动递增的主键，
CandidateId 是 Candidate 表中的 id.
```
>请编写 sql 语句来找到当选者的名字，上面的例子将返回当选者 B.
```
+------+
| Name |
+------+
| B    |
+------+

```
```
--我的解答,我忘记用limit函数了.....我这个答案不知道哪里有问题,只好在写一个了...
with x as (
select 
CandidateId
from 
(
select 
CandidateId,row_number(order by num1 desc) as rn
from 
(
select 
CandidateId, count(1) as num1
from 
Vote
group CandidateId
) t1
) t2
where rn =1
)

select 
Name
from 
Candidate y
where y.id = x.CandidateId

```
```
select name from Candidate where id =(
select CandidateId
from Vote
group by CandidateId 
order by count(*) desc
limit 1)
```
##19.员工奖金
```
select
name,bonus
from
Employee t
left join  Bonus t1
on t.empId  = t1.empId
where bonus <1000 or bonus is null
```
注意where条件后要加 is null 懒得写了,577题,简单题

##20查询回答率最高的问题

>从 survey_log 表中获得回答率最高的问题，survey_log 表包含这些列：id, action, question_id, answer_id, q_num, timestamp。
id 表示用户 id；action 有以下几种值："show"，"answer"，"skip"；当 action 值为 "answer" 时 answer_id 非空，而 action 值为 "show" 或者 "skip" 时 answer_id 为空；q_num 表示当前会话中问题的编号。
请编写 SQL 查询来找到具有最高回答率的问题。
回答率最高的含义是：同一问题编号中回答数占显示数的比例最高
```
输入：
+------+-----------+--------------+------------+-----------+------------+
| id   | action    | question_id  | answer_id  | q_num     | timestamp  |
+------+-----------+--------------+------------+-----------+------------+
| 5    | show      | 285          | null       | 1         | 123        |
| 5    | answer    | 285          | 124124     | 1         | 124        |
| 5    | show      | 369          | null       | 2         | 125        |
| 5    | skip      | 369          | null       | 2         | 126        |
+------+-----------+--------------+------------+-----------+------------+
输出：
+-------------+
| survey_log  |
+-------------+
|    285      |
+-------------+
解释：
问题 285 的回答率为 1/1，而问题 369 回答率为 0/1，因此输出 285 。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/get-highest-answer-rate-question
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```
```
-- 我的答案
select question_id as survey_log
from survey_log
group by question_id
order by sum(if(action = 'answer', 1, 0)) / sum(if(action = 'show', 1, 0)) desc
limit 1
```
```
select question_id as survey_log
from (
  select
      question_id,
      sum(if(action = 'answer', 1, 0)) as AnswerCnt,
      sum(if(action = 'show', 1, 0)) as ShowCnt
  from
      survey_log
  group by question_id
) as tbl
order by (AnswerCnt / ShowCnt) desc
limit 1
```
题目的意思不是很明确,主要考察sum和if结合使用,简单题


