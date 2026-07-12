>leedcode高频题,半年前写过再写一遍.....
##151.组合两张表 
>编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：
```
表1: Person

+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId 是上表主键
表2: Address

+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId 是上表主键
 

编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

 

FirstName, LastName, City, State

```     
```
--简单题
select
FirstName,LastName, City, State 
from Person t1 left join  Address t2 
on t1.PersonId  = t2. PersonId   
```
##152.第二高的薪水(自连接)
>编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary） 。
```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
例如上述 Employee 表，SQL查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null。

+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```
一开始我是这ifnull(?,null) 发现为空,where条件返回空,如果是聚合函数则不存在就会返回null
```
select
ifnull(max(Salary),null)  SecondHighestSalary
from 
(
select
Salary,
dense_rank() over(order by Salary desc ) rn 
from
Employee
) t1
where rn =2
```
其他方法找到最大的子查询取max最大的但是小于max
```
select max(Salary) SecondHighestSalary
from employee
where
salary<(select max(salary) from employee)
```
limit分页
```
select (select distinct salary from Employee order by salary desc limit 1,1) as SecondHighestSalary 
```
自连接
```
select
(
    select e1.Salary
    from Employee e1,Employee e2
    where e1.Salary<=e2.Salary
    group by e1.Salary
    having count(distinct e2.Salary)=2
) as SecondHighestSalary
```
##153.分数排名
>编写一个 SQL 查询来实现分数排名
```
编写一个 SQL 查询来实现分数排名。

如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。

+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：

+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
重要提示：对于 MySQL 解决方案，如果要转义用作列名的保留字，可以在关键字之前和之后使用撇号。例如 `Rank`
```
```
--简单题
select
Score,
dense_rank() over(order by Score desc  ) `Rank`
from 
Scores
```

```
--学习一下自连接,面试常问
select s1.Score,count(distinct(s2.score)) `Rank`
from
Scores s1,Scores s2
where
s1.score<=s2.score
group by s1.Id
order by `Rank`
```
##154.连续出现的数字
>编写一个 SQL 查询，查找所有至少连续出现三次的数字。
```
表：Logs

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| num         | varchar |
+-------------+---------+
id 是这个表的主键。
 

编写一个 SQL 查询，查找所有至少连续出现三次的数字。

返回的结果表中的数据可以按 任意顺序 排列。

 

查询结果格式如下面的例子所示：

 

Logs 表：
+----+-----+
| Id | Num |
+----+-----+
| 1  | 1   |
| 2  | 1   |
| 3  | 1   |
| 4  | 2   |
| 5  | 1   |
| 6  | 2   |
| 7  | 2   |
+----+-----+

Result 表：
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
1 是唯一连续出现至少三次的数字。
```
```
--不需要判断id的偏移量,因为样本id是连续的
select
distinct Num ConsecutiveNums
from
(
select
Id,
Num,
lag(Num,1) over() as lag_num,
lag(Id,1) over() as lag_id,
lead(Num,1) over() as lead_num,
lead(Id,1) over() as lead_id
from 
Logs
) t1
where Id = lag_id + 1 and Id = lead_id-1 and Num = lag_num and Num =lead_num
```
```
--一个巧妙的方法
SELECT DISTINCT Num AS ConsecutiveNums FROM Logs 
WHERE (Id+1, Num) IN (SELECT * FROM Logs)
AND (Id+2, Num) IN (SELECT * FROM Logs)
```
类似于连续登陆案例
```
select distinct Num as ConsecutiveNums
from (select Num,abs(Id+1-(dense_rank() over(partition by Num  order by Id)) ) as temp 
        from Logs ) as t 
group by Num,temp
having count(temp )>=3
```
##155.超过经理收入的员工
>给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。
```
Employee 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。

+----------+
| Employee |
+----------+
| Joe      |
+----------+
```
```
--简单题
select 
t1.Name Employee
from 
Employee t1 
join Employee t2 on t1.ManagerId = t2.Id 
where t1.Salary > t2.Salary
```
使用子查询,首先获取是经理的员工,然后在比较收入
##156查找重复的邮箱
>编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。
```
编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。

示例：

+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
根据以上输入，你的查询应返回以下结果：

+---------+
| Email   |
+---------+
| a@b.com |
```
```
select 
Email
from 
Person
group by Email
having count(1) >1
```
##157.从不订购的客户
>某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户
```
某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

Customers 表：

+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders 表：

+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
例如给定上述表格，你的查询应返回：

+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```
```
简单题
select 
t1.Name Customers
from 
Customers t1 left join Orders t2 on t1.Id = t2.CustomerId
where t2.CustomerId is null 
```
```
子查询
SELECT Name 'Customers'
FROM Customers
WHERE Id NOT IN(
    SELECT CustomerId 
    FROM Orders
)
```
##158.部门工资最高的员工
>编写一个 SQL 查询，找出每个部门工资最高的员工。对于上述表，您的 SQL 查询应返回以下行（行的顺序无关紧要）
```
Employee 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Jim   | 90000  | 1            |
| 3  | Henry | 80000  | 2            |
| 4  | Sam   | 60000  | 2            |
| 5  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
Department 表包含公司所有部门的信息。

+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
编写一个 SQL 查询，找出每个部门工资最高的员工。对于上述表，您的 SQL 查询应返回以下行（行的顺序无关紧要）。

+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
解释：

Max 和 Jim 在 IT 部门的工资都是最高的，Henry 在销售部的工资最高。
```
```
--简单题
select
Department,Employee,Salary 
from 
(
select
t1.Name Department, t2.Name Employee,Salary,
rank() over(partition by t1.Name order by Salary desc ) rn 
from Department t1 join Employee t2 on t1.Id = t2.DepartmentId
) s1 
where rn =1
```
```
子查询
select 
    d.Name as Department,
    e.Name as Employee,
    e.Salary 
from 
    Employee e,Department d 
where
    e.DepartmentId=d.id 
    and
    (e.Salary,e.DepartmentId) in (select max(Salary),DepartmentId from Employee group by DepartmentId);

```
##159.工资前三高的员工
>编写一个 SQL 查询，找出每个部门获得前三高工资的所有员工。例如，根据上述给定的表，查询结果应返回：
```
Employee 表包含所有员工信息，每个员工有其对应的工号 Id，姓名 Name，工资 Salary 和部门编号 DepartmentId 。

+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |
+----+-------+--------+--------------+
Department 表包含公司所有部门的信息。

+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
编写一个 SQL 查询，找出每个部门获得前三高工资的所有员工。例如，根据上述给定的表，查询结果应返回：

+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
解释：

IT 部门中，Max 获得了最高的工资，Randy 和 Joe 都拿到了第二高的工资，Will 的工资排第三。销售部门（Sales）只有两名员工，Henry 的工资最高，Sam 的工资排第二。
```
```
简单题窗口
select Department, Employee, Salary
from (
    select d.Name as Department, e.Name as Employee, e.Salary as Salary, 
dense_rank() over ( partition by DepartmentId order by Salary desc) as rk
    from Employee as e, Department as d
    where e.DepartmentId = d.Id
) m
where rk <= 3;
```
```
select d.Name as Department,e.Name as Employee,e.Salary as Salary
from Employee as e left join Department as d 
on e.DepartmentId = d.Id
where e.Id in
(
    select e1.Id
    from Employee as e1 left join Employee as e2
    on e1.DepartmentId = e2.DepartmentId and e1.Salary < e2.Salary
    group by e1.Id
    having count(distinct e2.Salary) <= 2
)
and e.DepartmentId in (select Id from Department)
order by d.Id asc,e.Salary desc
```
##170.删除重复的邮箱
```
编写一个 SQL 查询，来删除 Person 表中所有重复的电子邮箱，重复的邮箱里只保留 Id 最小 的那个。

+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Id 是这个表的主键。
例如，在运行你的查询语句之后，上面的 Person 表应返回以下几行:

+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
 

提示：

执行 SQL 之后，输出是整个 Person 表。
使用 delete 语句。
```
```
delete p1 from (person as p1 left join person as p2 on p1.email = p2.email) where p1.id > p2.id ;

```
```
DELETE from Person 
Where Id not in (
    Select Id 
    From(
    Select MIN(Id) as id
    From Person 
    Group by Email
   ) t
)
```
