>总结的挺好的复制一下吧........

# 一.检索记录

## 1.1 从表中检索所有行和列

需要检索所有列，一般是直接用"*"来代表
需要注意如果是查看数据库中的数据可以用 星号来代替，如果是代码中，最好将每个列都列明，方便后面引用列。

代码:

```
select * from emp

```

## 1.2 从表中检索部分行

使用where子句可以指定要保留哪些行。

例如要查看部门编号为10的所有员工:

```
select * from emp where deptno = 10

```

## 1.3 查找满足多个条件的行

使用 where子句以及 OR 和 AND 子句。

例如，如果要查找部门10中所有员工，且有提成的员工

```
select *
  from emp
 where deptno = 20
   and comm is not null

```

## 1.4 从表中检索部分列

指定感兴趣的列。

例如，如果只查看员工的名字、部门号和工资

```
select ename,deptno,sal
  from emp

```

## 1.5 为列取有意义的名称

要改变查询结果列名,可以按这种格式使用AS关键字: 原名 AS 新名(一些数据库可以省略AS)

```
select sal as salary, comm as commission
  from emp

```

## 1.6 在where子句中引用取别名的列

[MySQL在where子句中引用取别名的列](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F107937828)

## 1.7 连接列值

[MySQL 连接列值](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F107950271)

## 1.8 在select语句中使用条件逻辑

使用CASE 表达式 直接在 SELECT 语句中执行条件逻辑。

需求: 要产生一个结果集如果一个员工工资小于2000美金，就返回消息"UNDERPAID"，如果大于等于4000美金，就返回消息"OVERPAID",如果在这两者之间，就返回"OK".

```
select ename, sal,
       case when sal <= 2000 then 'UNDERPAID'
            when sal >= 4000 then 'OVERPAID'
            else 'OK'
        end as status
  from  emp

```

## 1.9 限制返回行数

限制查询中返回的行数，这里不关心顺序，返回任何n行都行。

```
select * from emp limit 5

```

## 1.10 从表中随机返回n行数据

[从表中随机返回n行数据](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108321768)

## 1.11 查找空值

要查找某列值为空的所有行。

要确定值是否为空，必须使用 IS NULL

```
select * from emp where comm is null

```

## 1.12 将空值转为实际值

在一些行中包含空值，需要使用非空值来代替这些空值。

使用coalesce函数用实际的值替换控制,coalesce函数有1个或多个参数，改函数返回第一个非空值。

```
select coalesce(comm,0) from emp

```

## 1.13 按模式搜索

需求: 在部门10和部门20，需要返回名字中有一个 "I" 或者职务(job title)中带有 "ER" 的员工。

使用LIKE运算符和SQL通配符"%"

```
select ename, job
  from emp
 where deptno in (10,20)
   and (ename like '%I' or job like '%ER')
```

<meta charset="utf-8">

# 二.查询结果排序

## 2.1 以指定的次序返回查询结果

需求: 显示部门10中员工名字、职位和工资，并按照工资的升序排列。

使用ORDER BY 子句可以对结果集进行排序。解决方案按SAL升序对行进行排列。
默认情况下，ORDER BY以升序方式排列，因此ASC子句是可选的。DESC表示降序排列。

```
select ename, job, sal
  from emp
 where deptno = 10
 order by sal asc;

-- 等同于
select ename, job, sal
  from emp
 where deptno = 10
 order by 3 asc;

```

## 2.2 按多个字段排序

需求: 在EMP表中，首先按照DEPTNO的升序排序行，然后按照工资的降序排列。

在ORDER BY子句中列出不同的排序列，使用逗号分隔:

```
select empno,deptno,sal,ename,job
  from emp
 order by deptno, sal desc

```

## 2.3 按子串排序

需求: 按字符串的某一部分对查询结果集排序，要从EMP表中返回员工名字和职位，并且按照职位字段的最后两个字符排序。

在ORDER BY子句中使用SUBSTR函数:

```
select ename,job
  from emp
 order by substr(job,length(job)-2)

```

## 2.4 处理排序的空值

需求: 在EMP中根据COMM排序结果。但是，这个字段可以有空值，需要指定是否将空值排在最后。

```
select ename,sal,comm
  from (
select ename,sal,comm,
       case when comm is null then 0 else 1 end as is_null
  from emp
       ) x
 order by is_null,comm desc

```

## 2.5 根据数据项的键排序

[MySQL根据数据项的键排序](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108337872)

# 三.操作多个表

## 3.1 记录集的叠加

需求: 要将来自多个表的数据组织到一起，就想将一个结果集叠加到另一个上面一样。这些表不必有相同的关机子，但是，他们对应累的数据类型应相同。

例如，要显示EMP表部门10中员工的名字和部门编号，以及DEPT表中每个部门的名字和部门编号。

使用集合操作 UNION ALL把多个表中的行组合到一起

```
select ename as ename_and_dname, deptno
  from emp
 where deptno = 10
union all
select '----------', null
union all
select dname,deptno
  from dept

```

## 3.2 组合相关的行

需求: 多个表有一些相同的列，或有些列的值相同，要通过连接这些列得到结果。

例如:要显示部门10中所有员工的名字，以及每个员工所在部门的工作地点，这些数据存储在两个独立的表中

```
select e.ename, d.loc
  from emp e, dept d
 where e.deptno = d.deptno
   and e.deptno = 10;

-- 等同于
select e.ename, d.loc
  from emp e
 inner join dept d
 on e.deptno = d.deptno
 where e.deptno = 10;

```

**讨论**
该解决方案是连接的一种，更准确地说是等值连接,这是内连接(inner join)的一种类型。
连接操作会将来自两个表的行组合到一个表中。等值连接的嵌套条件是相同条件。

从概念上来说，要得到连接的结果集，首先要创建from子句后列出的表的笛卡尔积(所有可能的情况)

```
select e.ename, d.loc,
       e.deptno as emp_deptno,
       d.deptno as dept_deptno
  from emp e, dept d
 where e.deptno = 10

```

笛卡尔积:

```
mysql> select e.ename, d.loc,
    ->        e.deptno as emp_deptno,
    ->        d.deptno as dept_deptno
    ->   from emp e, dept d
    ->  where e.deptno = 10;
+--------+----------+------------+-------------+
| ename  | loc      | emp_deptno | dept_deptno |
+--------+----------+------------+-------------+
| CLARK  | NEW YORK |         10 |          10 |
| KING   | NEW YORK |         10 |          10 |
| MILLER | NEW YORK |         10 |          10 |
| CLARK  | DALLAS   |         10 |          20 |
| KING   | DALLAS   |         10 |          20 |
| MILLER | DALLAS   |         10 |          20 |
| CLARK  | CHICAGO  |         10 |          30 |
| KING   | CHICAGO  |         10 |          30 |
| MILLER | CHICAGO  |         10 |          30 |
| CLARK  | BOSTON   |         10 |          40 |
| KING   | BOSTON   |         10 |          40 |
| MILLER | BOSTON   |         10 |          40 |
+--------+----------+------------+-------------+
12 rows in set (0.01 sec)

```

这将返回表EMP部门10中所有员工，以及DEPT的所有部门。
然后，在WHERE子句的表达式中使用 e.deptno 和 d.deptno 来限制结果集，只返回EMP.DEPTNo和DEPT.DEPTNO相等的那些行。

```
select e.ename, d.loc
  from emp e, dept d
 where e.deptno = d.deptno
   and e.deptno = 10;

```

测试记录:

```
mysql> select e.ename, d.loc
    ->   from emp e, dept d
    ->  where e.deptno = d.deptno
    ->    and e.deptno = 10;
+--------+----------+
| ename  | loc      |
+--------+----------+
| CLARK  | NEW YORK |
| KING   | NEW YORK |
| MILLER | NEW YORK |
+--------+----------+
3 rows in set (0.00 sec)

mysql>

```

## 3.3 在两个表中查找相同的行

需求:
查找两个表中共同行，但有多列可以用来连接这两个表。

```
create view v_test2
as
select ename,job,sal
  from emp
 where job = 'CLERK'

```

要返回正确的结果，必须按所有必要的列进行连接。或者，如果不想进行连接，也可以使用子查询 in 。

```
select e.ename,e.job,e.sal
  from emp e,v_test2 v
 where e.ename = v.ename
   and e.job = v.job
   and e.sal = v.sal;

select ename,job,sal
  from emp
 where (ename,job,sal) in
 (select ename,job,sal from v_test2);

```

## 3.4 从一个表中找到另一个表中没有的值

需求:
要从一个表(称为源表)中查找在另一目标表中不存在的值。

例如，要从表DEPT中查找在表EMP中不存在的数据的所有部门。
在示例数据库中，DEPTNO值为40的记录在表EMP中不存在。

MySQL可以使用not in

```
select deptno
  from dept
 where deptno not in (select deptno from emp);

```

**不过not in 存在一个问题**
如果not in 后面的结果集中有null的话，前面的查询结果会返回所有的值。

```
select deptno
  from dept 
 where deptno not in (10 ,20, null);

```

测试记录:

```
mysql> select deptno
    ->   from dept
    ->  where deptno not in (10 ,20, null);
Empty set (0.00 sec)

mysql>

```

deptno not in (10,20,null)等同于:
not (deptno = 10 or deptno = 50 or deptno = null)

(false or fasle or null) 最后的结果为null，所以不会有什么输出

**此时可以把not in 改为not exists可以避免上述问题**

```
select deptno
  from dept d
 where not exists (select 1 from emp e where e.deptno = d.deptno );

```

## 3.5 在一个表中查找与其它表不匹配的记录

需求:
对于具有相同关键字的两个表，要在一个表中查找与另外一个表中不匹配的行。
例如: 要查找没有支援的部门

解决方案:
首先返回一个表中的所有行,以及另一个表中与公共列匹配的行或不存在匹配行，然后，仅保留不匹配的行。

```
select d.*
  from dept d
  left join emp e
  on (d.deptno = e.deptno)
 where e.deptno is null;

```

## 3.6 像查询中增加连接而不影响其它连接

emp_bonus表:

```
create table emp_bonus(empno int,received date,type int);

insert into emp_bonus(empno,received,type) values (7369,'2005-03-14',1);
insert into emp_bonus(empno,received,type) values (7900,'2005-03-14',2);
insert into emp_bonus(empno,received,type) values (7788,'2005-03-14',3);

```

问题:
已经有了一个查询可以返回所需要的值，还需要得到其他信息，但当加入这些信息时，发现原始结果集中的数据有丢失。

例如，要返回所有的员工信息、他们工作部门的地点及所获得的奖励。
要求将每个员工说获得的奖励的日期列加入到结果数据中，但是连接到emp_bonus表后，所返回的记录数要比所希望的要少，因为并不是每个员工都有奖励。

```
select e.ename, d.loc
  from emp e, dept d
 where e.deptno = d.deptno;

```

解决方案:
可以使用外连接来获取这些附加的信息，并且原始查询中的数据不会丢失。

```
select e.ename, d.loc,eb.received
  from emp e
 inner join dept d
   on e.deptno = d.deptno
 left join emp_bonus eb
   on e.empno = eb.empno
 order by 2;

```

其实也可以使用标量子查询

```
select e.ename, 
       d.loc,
       (select eb.received from emp_bonus eb where eb.empno = e.empno) as received
  from emp e, dept d
 where e.deptno = d.deptno
 order by 2;

```

## 3.7 检测两个表中是否有相同数据

[检测两个表中是否有相同数据](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108355792)

## 3.8 识别和消除笛卡尔积

需求:
要返回在部门10中每个员工的姓名，以及部门的工作地点

```
select e.ename, d.loc
  from emp e, dept d
 where e.deptno = 10

```

测试记录:

```
mysql> select e.ename, d.loc
    ->   from emp e, dept d
    ->  where e.deptno = 10;
+--------+----------+
| ename  | loc      |
+--------+----------+
| CLARK  | NEW YORK |
| KING   | NEW YORK |
| MILLER | NEW YORK |
| CLARK  | DALLAS   |
| KING   | DALLAS   |
| MILLER | DALLAS   |
| CLARK  | CHICAGO  |
| KING   | CHICAGO  |
| MILLER | CHICAGO  |
| CLARK  | BOSTON   |
| KING   | BOSTON   |
| MILLER | BOSTON   |
+--------+----------+
12 rows in set (0.00 sec)

```

可以看到多了很多不必要的输出，因为忘记写表连接条件了
emp表4条记录，dept表3条记录，笛卡尔积产生的数据就是4*3=12条

其实真正需要的输出是4条

解决方案:
在from子句对表进行连接来返回正确的结果集:

```
select e.ename, d.loc
  from emp e, dept d
 where e.deptno = 10
   and d.deptno = e.deptno;

```

## 3.9 聚集与连接

emp_bonus表:

```
create table emp_bonus(empno int,received date,type int);

insert into emp_bonus(empno,received,type) values (7934,'2005-03-17',1);
insert into emp_bonus(empno,received,type) values (7934,'2005-09-15',2);
insert into emp_bonus(empno,received,type) values (7839,'2005-09-15',3);
insert into emp_bonus(empno,received,type) values (7782,'2005-09-15',1);

```

需求:
要在包含多个表的查询中执行聚集运算，要确保表间连接不能使聚集运算发生错误。

例如，要查找在部门10中所有员工的工资合计和奖金合计。由于有些员工的工资的奖金纪录不止一条，在表EMP表EMP_BONUS之间做连接会导致聚集函数SUM算得的值错误。

解决方案:
当处理聚集与连接混合操作时，一定要小心。如果连接产生重复行，可以有两种方法来避免聚集函数计算错误:

方法1：
只要在调用聚集函数时使用关键字DISTINCT，这样每个值只能参与计算一次

方法2：
在进行连接操作前前先执行聚集操作(在内联视图中)，这样，因为聚集计算已经在进行连接前完成了，所以可以避免聚集函数计算有误，可以规避此类问题。

方法1使用distinct，个人感觉存在问题，如果真有不同员工的工资相同，会少计算了

```
select  deptno,
        sum(distinct sal) as total_sal,
        sum(bonus) as total_bonus
  from  (
select  e.empno,
        e.ename,
        e.sal,
        e.deptno,
        e.sal* case when eb.type = 1 then 0.1 when eb.type =2 then 0.2 else 0.3 end as bonus
  from  emp e, emp_bonus eb
 where  e.empno = eb.empno
   and  e.deptno = 10
) x
 group by deptno;

```

方法2在表连接在聚集运算之前

```
select d.deptno,
       d.total_sal,
       sum(e.sal*case when eb.type = 1 then 0.1 when eb.type =2 then 0.2 else 0.3 end) as bonus
  from emp e,
       emp_bonus eb,
       (
select deptno, sum(sal) as total_sal
  from emp
 where deptno = 10
 group by deptno
       ) d
 where e.deptno = d.deptno
   and e.empno = eb.empno
 group by d.deptno,d.total_sal; 

```

## 3.10 聚集与外连接

emp_bonus表:

```
create table emp_bonus(empno int,received date,type int);

insert into emp_bonus(empno,received,type) values (7934,'2005-03-17',1);
insert into emp_bonus(empno,received,type) values (7934,'2005-09-15',2);

```

方法1使用distinct，个人感觉存在问题，如果真有不同员工的工资相同，会少计算了

```
select  deptno,
        sum(distinct sal) as total_sal,
        sum(bonus) as total_bonus
  from  (
select  e.empno,
        e.ename,
        e.sal,
        e.deptno,
        e.sal* case when eb.type = 1 then 0.1 when eb.type =2 then 0.2 when eb.type =3 then 0.3 end as bonus
  from  emp e
  left join emp_bonus eb
    on  e.empno = eb.empno
  where 1 = 1
   and  e.deptno = 10
) x
 group by deptno;

```

方法2在表连接在聚集运算之前

```
select d.deptno,
       d.total_sal,
       sum(e.sal*case when eb.type = 1 then 0.1 when eb.type =2 then 0.2 else 0.3 end) as bonus
  from emp e,
       emp_bonus eb,
       (
select deptno, sum(sal) as total_sal
  from emp
 where deptno = 10
 group by deptno
       ) d
 where e.deptno = d.deptno
   and e.empno = eb.empno
 group by d.deptno,d.total_sal; 

```

## 3.11 从多个表中返回丢失的数据

此处解决方案是full out join全连接

参考之前写的表连接的blog:
[MySQL表连接小结](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F106348552)

## 3.12 在运算和比较时使用null值

需求:
NULL值永远不会等于或不等于任何值，也不等于NULL值自己，但是需要像计算真实值一样计算可为空的返回值。

例如，需要在表EMP中查出所有比"WARD"提成(COMM)低的员工，提成为NULL(空)的员工也应当包括在其中。

解决方案:
使用COALESCE函数将NULL值转换为一个可以用来作为标准值进行比较的真实值:

```
select ename,comm
  from emp
 where coalesce(comm,0) < ( select comm
                             from  emp
                            where  ename = 'WARD'）
```

<meta charset="utf-8">

# 四.插入、更新与删除

## 4.1 插入新纪录

需求:
像表中插入一条新的记录。

例如，要想DEPT表中插入一条新的记录。其中，DEPTNO值为50、DNAME的值为"PROGRAMMING"、LOC的值为"BALTIMORE"

解决方案:
使用VALUES子句的INSERT语句来插入一行:

```
insert into dept (deptno,dname,loc)
 values (50,'PROGRAMMING','BALTIMORE');

```

对于MySQL，可以选择一次插入一行，或者用多个值列表一次插入多行:

```
/* multi row insert */
insert into dept (deptno,dname,loc)
 values (1,'A','B'),
        (2,'C','D');

```

## 4.2 插入默认值

需求:
定义表时可以为某些列定义默认值。现要以默认值插入一行，而无需指定各列的值。

```
create table D (id int default 0,name varchar(20));

```

解决方案:
插入语句可以直接用default或者不指定

```
insert into D (id, name) values (default, 'test1');
insert into D (name) values ('test2');

```

## 4.3 使用NULL代替默认值

需求:
在一个定义了默认值的列插入数据，并且需要不管该列的默认值是什么，都将该列值设为null

```
create table D ( id int default 0, name varchar(10));

```

解决方案:
可以在值列表中明确地指定NULL值

```
insert into d (id, name) values (null, 'test3');

```

## 4.4 从一个表向另外的表中复制行

```
create table DEPT_EAST like dept;

```

需求:
要使用查询从一个表中向另外的表中富支行。该查询可能非常复杂，也可能非常简单，但是最终是需要将查询的结果插入到其它的表中。

例如,要将表dept中的行复制到表DEPT_EAST中。

解决方案:
说使用的方法就是在INSERT语句后面紧跟一个用来产生说要插入的行的查询:

```
insert into dept_east(deptno, dname,loc)
select deptno,dname,loc
  from dept
 where loc in ('NEW YORK','BOSTON');

```

## 4.5 复制表定义

需求:
要创建新表，该表与已有表的列设置相同。

例如，想要创建一个DEPT表的副本，名为DEPT_1,但只是想复杂表结构而不想要复制源表中的记录。

解决方案:
可以使用CTAS命令，也可以使用like命令

```
create table dept_1 as select * from dept where 1 = 0;

create table dept_2 like dept;

```

查看表结构:

```
mysql> show create table dept_1\G
*************************** 1. row ***************************
       Table: dept_1
Create Table: CREATE TABLE `dept_1` (
  `deptno` int NOT NULL,
  `dname` varchar(14) DEFAULT NULL,
  `loc` varchar(13) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql> show create table dept_2\G
*************************** 1. row ***************************
       Table: dept_2
Create Table: CREATE TABLE `dept_2` (
  `deptno` int NOT NULL,
  `dname` varchar(14) DEFAULT NULL,
  `loc` varchar(13) DEFAULT NULL,
  PRIMARY KEY (`deptno`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql>

```

## 4.6 一次性向多个表中插入记录

目前MySQL还不支持此功能

目前Oracle、DB2、Hive支持此功能，但是实现方式不同。

## 4.7 阻止对某几列插入

[阻止对某几列插入](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108375548)

## 4.8 在表中编辑纪录

问题:
要修改表中某写(或全部)行的值。

例如，可能想要将部门20中所有员工的工资增加10%。

解决方案:
使用Update语句来修改数据库表中已有行。

```
update emp
   set sal = sal * 1.1
  where deptno = 20;

```

## 4.9 当相应行存在时更新

```
create table emp_bonus(empno int,received date,type int);

insert into emp_bonus(empno,received,type) values (7369,'2005-03-14',1);
insert into emp_bonus(empno,received,type) values (7900,'2005-03-14',2);
insert into emp_bonus(empno,received,type) values (7788,'2005-03-14',3);

```

问题:
仅当另一个表中相应的行存在时，更新某表中的一些行。

例如，如果表emp_bonus中存在某位员工，则要将该员工的工资增加20%(在表emp中)。

解决方案:
可以在where子句中结合子查询使用

```
update emp
   set sal = sal* 1.2
 where empno in (select empno from emp_bonus);

```

## 4.10 用其它表中的值更新

[用其它表中的值更新](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108397868)

## 4.11 合并记录

目前只有Oracle的merge语句可以解决此类问题。
MySQL暂不支持此类需求。

## 4.12 从表中删除所有记录

```
delete from emp;

```

## 4.13 删除指定记录

```
delete from emp where deptno = 10;

```

## 4.14 删除单个记录

从表中删除单个记录，一般需要找到表的主键列，然后根据主键列进行删除。

```
delete from emp where empno = 7369;

```

## 4.15 删除违反参照完整性的记录

例如，需要删除emp表中deptno在dept表中不存在的记录。

此时可以使用not exists来解决。

```
delete from emp
 where not exists (
   select 1 from dept 
     where dept.deptno = emp.deptno
   );

```

## 4.16 删除重复记录

```
create table test_repeat(id int,name varchar(20),mobile varchar(20));

insert into test_repeat(id ,name, mobile) values (1,'A','123456');
insert into test_repeat(id ,name, mobile) values (2,'B','123456');
insert into test_repeat(id ,name, mobile) values (3,'C','123456');
insert into test_repeat(id ,name, mobile) values (4,'A','123456');
insert into test_repeat(id ,name, mobile) values (5,'A','123456');
insert into test_repeat(id ,name, mobile) values (6,'B','123456');

```

需要删除重复记录，此时可以根据其它列进行分组，然后保留主键id列最大的一条记录。

```
delete from test_repeat
  where id not in 
      ( select max_id from 
      (  select max(id) as max_id from test_repeat group by name,mobile) tmp );

```

测试记录:

```
mysql> select * from test_repeat;
+------+------+--------+
| id   | name | mobile |
+------+------+--------+
|    1 | A    | 123456 |
|    2 | B    | 123456 |
|    3 | C    | 123456 |
|    4 | A    | 123456 |
|    5 | A    | 123456 |
|    6 | B    | 123456 |
+------+------+--------+
6 rows in set (0.00 sec)
mysql> delete from test_repeat
    ->   where id not in
    ->       ( select max_id from
    ->       (  select max(id) as max_id from test_repeat group by name,mobile) tmp );
Query OK, 3 rows affected (0.01 sec)

mysql> select * from test_repeat;
+------+------+--------+
| id   | name | mobile |
+------+------+--------+
|    3 | C    | 123456 |
|    5 | A    | 123456 |
|    6 | B    | 123456 |
+------+------+--------+
3 rows in set (0.00 sec)

mysql>

```

## 4.17 删除从其它表引用的记录

需求:
从一个表中删除被另一个表应用的记录。

考虑下面的dept_accidents表，其中每行代表生产过程中的一次失误，每行中记录了事故发生的部门以及事故类型。

```
create table dept_accidents
( deptno    int,
  accident_name varchar(20) );

insert into dept_accidents values (10, 'BROKEN FOOT');
insert into dept_accidents values (10, 'FLESH WOUND');
insert into dept_accidents values (20, 'FIRE');
insert into dept_accidents values (20, 'FIRE');
insert into dept_accidents values (20, 'FLOOD');
insert into dept_accidents values (30, 'BRUISED GLUTE');

```

要从表EMP中删除所在部门已经发生了三次以上的事务的所有员工的记录。

解决方案:
使用子查询和聚集函数count来查找出发生三次事故以上的部门，然后将这些部门的员工全部删除。

```
delete from emp
 where deptno in ( select deptno 
                     from dept_accidents
                    group by deptno
                   having count(*) >= 3 );
```


<meta charset="utf-8">

# 五.元数据查询

[元数据查询](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108440653)

包含5.1到5.4章节

## 5.1 列出模式的表

```
select table_name
from information_schema.tables
where table_schema = 'ZQS';

```

## 5.2 列出表的列

```
select column_name,data_type,ordinal_position
from information_schema.columns
where table_schema = 'ZQS'
and   table_name = 'EMP';

```

## 5.3 列出表的索引列

```
show index from emp\G

```

## 5.4 列出表的约束

```
select  a.table_name,
        a.constraint_name,
        b.column_name,
        a.constraint_type
from information_schema.table_constraints a,
     information_schema.key_column_usage b
where a.table_name = 'EMP'
and   a.table_schema  = 'ZQS'
and   a.table_name = b.table_name
and   a.table_schema = b.table_schema
and   a.constraint_name = b.constraint_name;

```

## 5.5 列出没有相应索引的外键

可以使用 SHOW INDEX 命令来检查索引信息，例如索引名，在索引中的列和在索引中这些列的顺序。
另外，可以查询INFORMATION_SCHEMA.KEY_COLUMN_USAGE列出指定表的外键。

在MySQL 5中，外键号称可以自动索引，但事实上是可以解除索引的。

要检测外键索引是否被解除，可以对该表执行 SHOW INDEX命令，并且将结果跟INFORMATION_SCHEMA.KEY_COLUMN_USAGE.COLUMN_NAME中同一个表的结果相比，如果KEY_COLUMN_USAGE中有COLUMN_NAME,但SHOW INDEX的结果中没有，则该列没有被索引。

## 5.6 使用SQL来生成SQL

[使用SQL来生成SQL](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108592750)

# 六.使用字符串

## 6.1 遍历字符串

[MySQL 遍历字符串](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108614833)

## 6.2 字符串文字中包含引号

[字符串中包含引号](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108614890)

## 6.3 计算字符在字符串中出现的次数

[计算字符在字符串中出现的次数](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108635521)

## 6.4 从字符串中删除不需要的字符

[从字符串中删除不需要的字符](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108660999)

## 6.5 将字符串和数字数据分离

此解决方案需要使用到translate，MySQL暂时不支持

## 6.6 判断字符串是不是数字字符类型

[判断字符串是不是数字字符类型](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108703782)

## 6.7 提取姓名大写首字母缩写

[提取姓名大写首字母缩写](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F108995845)

## 6.8 按字符串中的部分内容排序

前面章节已经涉及
order by substr(col_name,length(col_name) - 2)

## 6.9 按字符串中的数值排序

此解决方案需要使用到translate，MySQL暂时不支持

## 6.10 根据表中的行创建一个分隔列表

[根据表中的行创建一个分隔列表](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109021640)

## 6.11 将分隔数据转换为多值IN列表

[将分隔数据转换为多值IN列表](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109026972)

## 6.12 按字母顺序排列其中的各个字符

[按字母顺序排列其中的各个字符](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109050042)

## 6.13 判别可以作为数值的字符串

[提取字符和数字里面的数字](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109072332)

## 6.14 提取第n个分隔的子串

需求:
从字符串中提取出一个指定的、由分隔符隔开的子字符串。

例如，去除每行中第二个姓名。

```
create view V as
select 'mo,larry,curly' as name
union all
select 'tina,gina,jaunita,regina,leena' as name;

drop table if exists t10;

create table t10(id int);

insert into t10 values (1),(2),(3),(4),(5),(6),(7),(8),(9),(10);

```

解决方案:
解决此问题的关键就是将每个姓名作为单独的行返回，并保留每个姓名在列表中的顺序。

```
select v.name,
       t10.id,
       substring_index(v.name,',',t10.id) as middle_name,
       substring_index(substring_index(v.name,',',t10.id),',',-1)  as new_name
  from V
 inner join t10
 on t10.id <= length(v.name) - length(replace(v.name,',','')) + 1 
 order by v.name,t10.id;

```

测试记录:

```
mysql> select v.name,
    ->        t10.id,
    ->        substring_index(v.name,',',t10.id) as middle_name,
    ->        substring_index(substring_index(v.name,',',t10.id),',',-1)  as new_name
    ->   from V
    ->  inner join t10
    ->  on t10.id <= length(v.name) - length(replace(v.name,',','')) + 1
    ->  order by v.name,t10.id;
+--------------------------------+------+--------------------------------+----------+
| name                           | id   | middle_name                    | new_name |
+--------------------------------+------+--------------------------------+----------+
| mo,larry,curly                 |    1 | mo                             | mo       |
| mo,larry,curly                 |    2 | mo,larry                       | larry    |
| mo,larry,curly                 |    3 | mo,larry,curly                 | curly    |
| tina,gina,jaunita,regina,leena |    1 | tina                           | tina     |
| tina,gina,jaunita,regina,leena |    2 | tina,gina                      | gina     |
| tina,gina,jaunita,regina,leena |    3 | tina,gina,jaunita              | jaunita  |
| tina,gina,jaunita,regina,leena |    4 | tina,gina,jaunita,regina       | regina   |
| tina,gina,jaunita,regina,leena |    5 | tina,gina,jaunita,regina,leena | leena    |
+--------------------------------+------+--------------------------------+----------+
8 rows in set (0.00 sec)

```

## 6.15 分解IP地址

[分解IP地址](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109183381)

# 七.使用数字

## 7.1 计算平均值

```
select avg(sal) from emp;

```

## 7.2 求某列的最小/最大值

```
select min(sal) as min_sal,max(sal) as max_sal from emp;

```

## 7.3 对某列的值求和

```
select sum(sal) from emp ;

select deptno,sum(sal) from emp group by deptno;

```

## 7.4 求一个表的行数

```
select count(*) from emp;

select deptno,count(*) from emp group by deptno;

```

## 7.5 求某列值的个数

[求某列值的个数](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109199498)

## 7.6 生成累计和

[生成累计和](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109218929)

## 7.7 生成累计乘积

[生成累计乘积](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109241500)

## 7.8 计算累积差

[MySQL 生成累计差](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109285329)

## 7.9 计算模式

需求:
查找某个列中值的模式(数学中的模式概念就是对于给定的数据集出现最频繁的元素)。

例如，查找deptno 20中工资的模式。

解决方案:

```
select sal
 from 
  (
select sal,
       dense_rank() over (order by cnt desc) as rnk
  from (
select sal,count(*) as cnt
  from emp
 where deptno = 20
 group by sal
       ) x
  ) y
 where rnk = 1

```

## 7.10 计算中间值

[计算中间值](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109309863)

## 7.11 求总和的百分比

```
select (sum(
         case when deptno = 10 then sal end)/ sum(sal)
       )*100 as pct
  from emp;

```

## 7.12 对可空列做聚集

对空值做聚集的时候，最好将空值转为0，不然会影响到平均值的计算。

```
select avg(coalesce(comm,0)) as avg_comm from emp where deptno = 30;

```

测试记录:

```
mysql> select ename,comm from emp where deptno = 30;
+--------+---------+
| ename  | comm    |
+--------+---------+
| ALLEN  |  700.00 |
| WARD   |  500.00 |
| MARTIN | 1400.00 |
| BLAKE  |    NULL |
| TURNER |    0.00 |
| JAMES  |    NULL |
+--------+---------+
6 rows in set (0.00 sec)

mysql> select avg(comm) from emp where deptno = 30;
+------------+
| avg(comm)  |
+------------+
| 650.000000 |
+------------+
1 row in set (0.00 sec)

mysql> select avg(coalesce(comm,0)) as avg_comm from emp where deptno = 30;
+------------+
| avg_comm   |
+------------+
| 433.333333 |
+------------+
1 row in set (0.00 sec)

mysql>

```

## 7.13 计算不包含最大值和最小值的均值

[计算不包含最大值和最小值的均值](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109357670)

## 7.14 把字母数字串转换为数值

MySQL目前不支持translate函数，暂时无解决方案。

## 7.15 更改累积和中的值

问题:
根据另一列中的值修改累计和中的值。

假设一个场景，要显示信用卡账号的事务处理历史遗迹每次事务处理之后的当前余额。

```
create view v (id,amt,trx)
as
select 1, 100 , '存款'
union all
select 2, 100 , '存款'
union all
select 3, 50 , '取款'
union all
select 4, 100 , '存款'
union all
select 5, 200 , '取款'
union all
select 6, 50 , '取款'

```

解决方案:
使用标量子查询创建累计和，并使用CASE表达式判断事务处理的类型

```
select  id,
        amt,
        trx,
        sum(new_amt) over (order by id)  as balance
  from 
 (
select  case when v.trx = '取款' then -amt
             when v.trx = '存款' then amt
        else null end as new_amt,
        id,
        amt,
        trx
  from v
) tmp1;

```

测试记录:

```
mysql> select  id,
    ->         amt,
    ->         trx,
    ->         sum(new_amt) over (order by id)  as balance
    ->   from
    ->  (
    -> select  case when v.trx = '取款' then -amt
    ->              when v.trx = '存款' then amt
    ->         else null end as new_amt,
    ->         id,
    ->         amt,
    ->         trx
    ->   from v
    -> ) tmp1;
+----+-----+--------+---------+
| id | amt | trx    | balance |
+----+-----+--------+---------+
|  1 | 100 | 存款   |     100 |
|  2 | 100 | 存款   |     200 |
|  3 |  50 | 取款   |     150 |
|  4 | 100 | 存款   |     250 |
|  5 | 200 | 取款   |      50 |
|  6 |  50 | 取款   |       0 |
+----+-----+--------+---------+
6 rows in set (0.00 sec)
```
<meta charset="utf-8">

# 八.日期运算

## 8.1 加减日、月、年

```
select hiredate - interval 5 day     as d_minus_5d,
       hiredate + interval 5 day     as d_plus_5D,
       hiredate - interval 5 month   as d_minus_5M,
       hiredate + interval 5 month   as d_plus_5M,
       hiredate - interval 5 year    as d_minus_5Y,
       hiredate + interval 5 year    as d_plus_5Y
  from emp
 where deptno = 10;

```

## 8.2 计算两个日期之间的天数

```
select ward_hd,allen_hd,datediff(ward_hd,allen_hd)
  from (
select hiredate as ward_hd
  from emp
 where ename = 'WARD'
       ) x,
       (
select hiredate as allen_hd
  from emp 
 where ename = 'ALLEN'
       ) y

```

## 8.3 确定两个日期之间工作日数目

weekday 返回数值的星期数
0 = Monday, 1 = Tuesday, … 6 = Sunday
根据这个剔除，然后count(*) 求总数即可

## 8.4 确定两个日期之间的月份数或年数

```
select mnth, mnth/12
  from (
select (year(max_hd) - year(min_hd)) *12 +
        (month(max_hd) - month(min_hd)) as mnth
  from (
select min(hiredate) as min_hd, max(hiredate) as max_hd
  from emp
       ) x
       ) y;

```

## 8.5 确定两个日期之间秒、分、小时数

```
select datediff(ward_hd,allen_hd)*24 hr,
       datediff(ward_hd,allen_hd)*24*60 min,
       datediff(ward_hd,allen_hd)*24*60*60 sec
  from (
select max(case when ename = 'WARD'
                then hiredate
           end) as ward_hd,
       max(case when ename = 'ALLEN'
                then hiredate
           end) as allen_hd
  from emp
       ) x

```

## 8.6 计算一年中周内各日期次数

生成包含一年的所有日期

使用DATE_FORMAT函数，确定每个日期为星期几，然后计算周内各日期的次数。

```
with recursive c(n) as
(
 select 1 
 union all
 select n + 1  from c where n < 400
),
tmp1 as
(
select '2021-01-01' as dt 
),
tmp2 as
(
select adddate(dt,n-1) dt
  from tmp1
inner join c
)
select date_format(dt,'%a'),
       count(*)
  from tmp2
where dt < '2022-01-01'
group by date_format(dt,'%a')

```

## 8.7 确定当前记录和下一条记录之间相差的天数

问题:
求两个日期之间相差的天数。

例如，对于deptno 10中的每个员工，确定聘用他们的日期及聘用下一个员工的日期之间相差的天数。

解决方案：
这个问题的解决方案是找到当前员工聘用的最早hiredate，再来求差。

```
select x.*,
       datediff(x.next_hd,x.hiredate) diff
  from (
select e.deptno, e.ename, e.hiredate,
       (select min(d.hiredate) from emp d
         where d.hiredate > e.hiredate) next_hd
  from emp e
 where e.deptno = 10
       ) x;

```

# 九.日期操作

## 9.1 确定一年是否为闰年

[计算今年是否为闰年](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109381861)

## 9.2 确定一年内的天数

[计算本年的天数](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109449470)

## 9.3 从日期中提取时间的各部分

```
select date_format(current_timestamp,'%k') hr,
       date_format(current_timestamp,'%i') min,
       date_format(current_timestamp,'%s') sec,
       date_format(current_timestamp,'%d') dy,
       date_format(current_timestamp,'%m') mon,
       date_format(current_timestamp,'%Y') yr;

```

## 9.4 确定某个月的第一天和最后一天

[计算本月的第一天及最后一天](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109471312)

## 9.5 确定一年内属于周内某一天的所有日期

```
with recursive c(n) as
(
 select 1 
 union all
 select n + 1  from c where n < 400
),
tmp1 as
(
select '2021-01-01' as dt 
),
tmp2 as
(
select adddate(dt,n-1) dt
  from tmp1
inner join c
)
select dt,
       date_format(dt,'%a') week
  from tmp2
where dt < '2022-01-01'
  and date_format(dt,'%a') = 'Fri'
order by dt;

```

## 9.6 确定某月内第一个和最后一个"周内某天"的日期

[计算本月的第一个和最后一个周一](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109489942)

## 9.7 创建日历

[打印本月日历](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109495960)

## 9.8 列出一年中每个季度的开始日期和结束日期

```
select quarter(adddate(dy,-1)) qtr,
       date_add(dy,interval -3 month) q_start,
       adddate(dy,-1) q_end
  from (
select date_add(dy,interval (3*id) month) dy
  from (
select id,
       adddate(current_date,-dayofyear(current_date) + 1 ) dy
  from t10
 where id <= 4
       ) x
       ) y;

```

测试记录:

```
mysql> select quarter(adddate(dy,-1)) qtr,
    ->        date_add(dy,interval -3 month) q_start,
    ->        adddate(dy,-1) q_end
    ->   from (
    -> select date_add(dy,interval (3*id) month) dy
    ->   from (
    -> select id,
    ->        adddate(current_date,-dayofyear(current_date) + 1 ) dy
    ->   from t10
    ->  where id <= 4
    ->        ) x
    ->        ) y;
+------+------------+------------+
| qtr  | q_start    | q_end      |
+------+------------+------------+
|    1 | 2021-01-01 | 2021-03-31 |
|    2 | 2021-04-01 | 2021-06-30 |
|    3 | 2021-07-01 | 2021-09-30 |
|    4 | 2021-10-01 | 2021-12-31 |
+------+------------+------------+
4 rows in set (0.00 sec)

```

## 9.9 确定某个给定季度的开始日期和结束日期

下面这个sql输出还是存在一点问题

```
select date_add(adddate(q_start,-day(q_start)+1) , interval -2 month) q_start,
       q_end
  from 
  (
select date_format(q_end,'%Y-%m-%d') as q_start,
       q_end 
  from (
select last_day(
       str_to_date(
          concat(
          substr(yrq,1,4), mod(yrq,10)*3), '%Y%m')) as q_end
  from (
select 20211 as yrq union all
select 20212 as yrq union all
select 20213 as yrq union all
select 20214 as yrq 
       ) x
       ) y
       ) z

```

测试记录:

```
mysql> select date_add(
    ->        adddate(q_end,-day(q_end)+1),
    ->                interval -2 month) q_start,
    ->        q_end
    ->   from (
    -> select last_day(
    ->        str_to_date(
    ->           concat(
    ->           substr(yrq,1,4), mod(yrq,10)*3), '%Y%m')) q_end
    ->   from (
    -> select 20211 as yrq union all
    -> select 20212 as yrq union all
    -> select 20213 as yrq union all
    -> select 20214 as yrq
    ->        ) x
    ->        ) y
    ->       ;
+---------+------------+
| q_start | q_end      |
+---------+------------+
| NULL    | 2021-03-31 |
| NULL    | 2021-06-30 |
| NULL    | 2021-09-30 |
| NULL    | 2021-12-31 |
+---------+------------+
4 rows in set, 4 warnings (0.00 sec)

mysql> select date_add(adddate(q_start,-day(q_start)+1) , interval -2 month) q_start,
    ->        q_end
    ->   from
    ->   (
    -> select date_format(q_end,'%Y-%m-%d') as q_start,
    ->        q_end
    ->   from (
    -> select last_day(
    ->        str_to_date(
    ->           concat(
    ->           substr(yrq,1,4), mod(yrq,10)*3), '%Y%m')) as q_end
    ->   from (
    -> select 20211 as yrq union all
    -> select 20212 as yrq union all
    -> select 20213 as yrq union all
    -> select 20214 as yrq
    ->        ) x
    ->        ) y
    ->        ) z;
+------------+------------+
| q_start    | q_end      |
+------------+------------+
| 2021-01-01 | 2021-03-31 |
| 2021-04-01 | 2021-06-30 |
| 2021-07-01 | 2021-09-30 |
| 2021-10-01 | 2021-12-31 |
+------------+------------+
4 rows in set (0.00 sec)

mysql>

```

## 9.10 填充丢失的日期

需求:
为给定范围内的每个日期(每个月、周或年)生成一行信息。这样的行集通常用于生成综合报告。

例如，计算没年内每个月聘用的员工数。

解决方案:
这里的敲门是为每个月返回一行信息，即使并未聘用任何员工(此时该数为0)。
由于1980年1987年间并不是每个月都聘用过员工，因此必须生成这些月份，然后把按HIREDATE(聘用日期)与表EMP进行外连接(对HIREDATE截取月份，以便它能与生成的月份相匹配)。

```
with recursive c(n) as
(
select 1 as n
union all
select n + 1 from c
 where n < 400
),
tmp1 as
(
select min(hiredate) min_hiredate,max(hiredate) max_hiredate
  from emp
),
tmp2 as
(
select min_hiredate,
       adddate(min_hiredate,-dayofyear(min_hiredate) + 1) new_min_hiredate,
       max_hiredate,
       date_add(adddate(max_hiredate,-dayofyear(max_hiredate) + 1),interval 12 month) new_max_hiredate
  from tmp1
),
tmp3 as
(
select date_add(new_min_hiredate, interval c.n - 1 month)  new_min_hiredate2
  from tmp2
inner join c
where date_add(new_min_hiredate, interval c.n - 1 month) < new_max_hiredate
)
select date_format(new_min_hiredate2,'%Y-%m') hired_mth,
       count(e.hiredate) as hired_nums
  from tmp3
 left join emp e
 on date_format(new_min_hiredate2,'%Y-%m') = date_format(e.hiredate,'%Y-%m')
 group by date_format(new_min_hiredate2,'%Y-%m')
 order by date_format(new_min_hiredate2,'%Y-%m')

```

测试记录:

```
mysql> with recursive c(n) as
    -> (
    -> select 1 as n
    -> union all
    -> select n + 1 from c
    ->  where n < 400
    -> ),
    -> tmp1 as
    -> (
    -> select min(hiredate) min_hiredate,max(hiredate) max_hiredate
    ->   from emp
    -> ),
    -> tmp2 as
    -> (
    -> select min_hiredate,
    ->        adddate(min_hiredate,-dayofyear(min_hiredate) + 1) new_min_hiredate,
    ->        max_hiredate,
    ->        date_add(adddate(max_hiredate,-dayofyear(max_hiredate) + 1),interval 12 month) new_max_hiredate
    ->   from tmp1
    -> ),
    -> tmp3 as
    -> (
    -> select date_add(new_min_hiredate, interval c.n - 1 month)  new_min_hiredate2
    ->   from tmp2
    -> inner join c
    -> where date_add(new_min_hiredate, interval c.n - 1 month) < new_max_hiredate
    -> )
    -> select date_format(new_min_hiredate2,'%Y-%m') hired_mth,
    ->        count(e.hiredate) as hired_nums
    ->   from tmp3
    ->  left join emp e
    ->  on date_format(new_min_hiredate2,'%Y-%m') = date_format(e.hiredate,'%Y-%m')
    ->  group by date_format(new_min_hiredate2,'%Y-%m')
    ->  order by date_format(new_min_hiredate2,'%Y-%m');
+-----------+------------+
| hired_mth | hired_nums |
+-----------+------------+
| 1980-01   |          0 |
| 1980-02   |          0 |
| 1980-03   |          0 |
| 1980-04   |          0 |
| 1980-05   |          0 |
| 1980-06   |          0 |
| 1980-07   |          0 |
| 1980-08   |          0 |
| 1980-09   |          0 |
| 1980-10   |          0 |
| 1980-11   |          0 |
| 1980-12   |          1 |
| 1981-01   |          0 |
| 1981-02   |          2 |
| 1981-03   |          0 |
| 1981-04   |          1 |
| 1981-05   |          1 |
| 1981-06   |          1 |
| 1981-07   |          0 |
| 1981-08   |          0 |
| 1981-09   |          2 |
| 1981-10   |          0 |
| 1981-11   |          1 |
| 1981-12   |          2 |
| 1982-01   |          1 |
| 1982-02   |          0 |
| 1982-03   |          0 |
| 1982-04   |          0 |
| 1982-05   |          0 |
| 1982-06   |          0 |
| 1982-07   |          0 |
| 1982-08   |          0 |
| 1982-09   |          0 |
| 1982-10   |          0 |
| 1982-11   |          0 |
| 1982-12   |          0 |
| 1983-01   |          0 |
| 1983-02   |          0 |
| 1983-03   |          0 |
| 1983-04   |          0 |
| 1983-05   |          0 |
| 1983-06   |          0 |
| 1983-07   |          0 |
| 1983-08   |          0 |
| 1983-09   |          0 |
| 1983-10   |          0 |
| 1983-11   |          0 |
| 1983-12   |          0 |
| 1984-01   |          0 |
| 1984-02   |          0 |
| 1984-03   |          0 |
| 1984-04   |          0 |
| 1984-05   |          0 |
| 1984-06   |          0 |
| 1984-07   |          0 |
| 1984-08   |          0 |
| 1984-09   |          0 |
| 1984-10   |          0 |
| 1984-11   |          0 |
| 1984-12   |          0 |
| 1985-01   |          0 |
| 1985-02   |          0 |
| 1985-03   |          0 |
| 1985-04   |          0 |
| 1985-05   |          0 |
| 1985-06   |          0 |
| 1985-07   |          0 |
| 1985-08   |          0 |
| 1985-09   |          0 |
| 1985-10   |          0 |
| 1985-11   |          0 |
| 1985-12   |          0 |
| 1986-01   |          0 |
| 1986-02   |          0 |
| 1986-03   |          0 |
| 1986-04   |          0 |
| 1986-05   |          0 |
| 1986-06   |          0 |
| 1986-07   |          0 |
| 1986-08   |          0 |
| 1986-09   |          0 |
| 1986-10   |          0 |
| 1986-11   |          0 |
| 1986-12   |          0 |
| 1987-01   |          0 |
| 1987-02   |          0 |
| 1987-03   |          0 |
| 1987-04   |          0 |
| 1987-05   |          0 |
| 1987-06   |          2 |
| 1987-07   |          0 |
| 1987-08   |          0 |
| 1987-09   |          0 |
| 1987-10   |          0 |
| 1987-11   |          0 |
| 1987-12   |          0 |
+-----------+------------+
96 rows in set (0.00 sec)

mysql>

```

## 9.11 按照给定的时间单位进行查找

需求:
查找与给定月份、星期几或其它时间单位相匹配的日期。
例如，找到2月份和12月份聘用的所有员工，或者查找星期二聘用的所有员工。

解决方案:
用MySQL自带的函数即可。

```
select ename
  from emp
 where month(hiredate) in (2,12) or weekday(hiredate) = 1;

```

测试记录:

```
mysql> select ename
    ->   from emp
    ->  where month(hiredate) in (2,12) or weekday(hiredate) = 1;
+--------+
| ename  |
+--------+
| SMITH  |
| ALLEN  |
| WARD   |
| CLARK  |
| KING   |
| TURNER |
| JAMES  |
| FORD   |
+--------+
8 rows in set (0.00 sec)

```

## 9.12 使用日期的特殊部分比较记录

需求:
查找聘用日期月份和周内日期都相同的员工。

例如，如果在1983年3月10日星期一聘用了某个员工，而在2001年3月2日星期一聘用了另一个员工，那么二者的聘用日都在星期一，而且月份名一致，则可以认为他们相匹配。

解决方案:
因为要把一个员工的hiredate与另一个员工的hiredate相比较，所以需要对表emp进行自连接，这样，就可以对hiredate的每种组合进行比较，然后从每个hiredate中提取它是星期几及月份名，并进行比较。

```
select concat(e1.ename,' was hired on the same month and weekday as ',e2.ename) as str
  from emp e1
 inner join emp e2
 on month(e1.hiredate) = month(e2.hiredate)
and weekday(e1.hiredate) = weekday(e2.hiredate)
and e1.empno <  e2.empno

```

测试记录:

```
mysql> select concat(e1.ename,' was hired on the same month and weekday as ',e2.ename) as str
    ->   from emp e1
    ->  inner join emp e2
    ->  on month(e1.hiredate) = month(e2.hiredate)
    -> and weekday(e1.hiredate) = weekday(e2.hiredate)
    -> and e1.empno <  e2.empno;
+--------------------------------------------------------+
| str                                                    |
+--------------------------------------------------------+
| SCOTT was hired on the same month and weekday as ADAMS |
| JAMES was hired on the same month and weekday as FORD  |
+--------------------------------------------------------+
2 rows in set (0.00 sec)

```

## 9.13 识别重叠的日期范围

```

create table emp_project(empno int, ename varchar(20), proj_id int, proj_start date, proj_end date);

insert into emp_project values (7782,'CLARK',1,'2005-06-16','2005-06-18');
insert into emp_project values (7782,'CLARK',4,'2005-06-19','2005-06-24');
insert into emp_project values (7782,'CLARK',7,'2005-06-22','2005-06-25');
insert into emp_project values (7782,'CALRK',10,'2005-06-25','2005-06-28');
insert into emp_project values (7782,'CLARK',13,'2005-06-28','2005-07-02');
insert into emp_project values (7839,'KING',2,'2005-06-17','2005-06-21');
insert into emp_project values (7839,'KING',8,'2005-06-23','2005-06-25');
insert into emp_project values (7839,'KING',14,'2005-06-29','2005-06-30');
insert into emp_project values (7839,'KING',11,'2005-06-26','2005-06-27');
insert into emp_project values (7839,'KING',5,'2005-06-20','2005-06-24');
insert into emp_project values (7934,'MILLER',3,'2005-06-18','2005-06-22');
insert into emp_project values (7934,'MILLER',12,'2005-06-27','2005-06-28');
insert into emp_project values (7934,'MILLER',15,'2005-06-30','2005-07-03');
insert into emp_project values (7934,'MILLER',9,'2005-06-24','2005-06-27');
insert into emp_project values (7934,'MILLER',6,'2005-06-21','2005-06-23');

```

需求:
查找员工在老工程结束之前就开始新工程的所有实例。

观察一下员工KING的结果，会浮现KING在完成PROJ_ID 5之前开始了PROJ_ID 8，而且在完成PROJ_ID之前又开始了PROJ_ID5.

解决方案:
这里的关键是,找打PROJ_START(新工程的开始日期)出现另一个工程的PROJ_START之后PROJ_END日期之前的那些行。
首先，应该把同一个员工的每个工程与另一个工程相比较。
对EMP_PROJECT按员工进行自连接，得到每个员工任意两个工程的所有组合。
要找到重叠的情况，只需找到一个PROJ_ID的PROJ_START介于另一个PROJ_ID的PROJ_START和PROJ_END之间的所有行即可。

```
select a.empno, a.ename,
       concat('project ',b.proj_id,
               ' overlaps project ',a.proj_id) as msg
  from emp_project a,
       emp_project b
 where a.empno = b.empno
   and b.proj_start >= a.proj_start
   and b.proj_start <= a.proj_end
   and a.proj_id != b.proj_id;

```

测试记录:

```
mysql> select a.empno, a.ename,
    ->        concat('project ',b.proj_id,
    ->                ' overlaps project ',a.proj_id) as msg
    ->   from emp_project a,
    ->        emp_project b
    ->  where a.empno = b.empno
    ->    and b.proj_start >= a.proj_start
    ->    and b.proj_start <= a.proj_end
    ->    and a.proj_id != b.proj_id;
+-------+--------+--------------------------------+
| empno | ename  | msg                            |
+-------+--------+--------------------------------+
|  7782 | CLARK  | project 7 overlaps project 4   |
|  7782 | CLARK  | project 10 overlaps project 7  |
|  7782 | CALRK  | project 13 overlaps project 10 |
|  7839 | KING   | project 8 overlaps project 5   |
|  7839 | KING   | project 5 overlaps project 2   |
|  7934 | MILLER | project 12 overlaps project 9  |
|  7934 | MILLER | project 6 overlaps project 3   |
+-------+--------+--------------------------------+
7 rows in set (0.00 sec)

mysql>
```

<meta charset="utf-8">

<article class="_2rhmJa">

# 十.范围处理

## 10.1 定位连续值范围

[定位连续值范围](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109503303)

## 10.2 查找同一组或分区中行之前的差

[查找同一组或分区中行的差](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109503356)

## 10.3 定义连续值范围的开始和结束点

[定义连续值范围的开始点和结束点](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109525116)

## 10.4 补充范围内丢失的值

问题:
返回1980年起始的十年间每年聘用的员工数，但有些奶粉并没有聘用员工。

解决方案:
这个解决方案的技巧是，对为聘用任何员工的年份返回0。

```
with recursive c(n) as
(
select 1 as n
union all
select n + 1 from c where n < 10
),
tmp1 as
(
select year(min(hiredate)) min_hiredate
  from emp
),
tmp2 as
(
select min_hiredate + n as new_hired
  from tmp1
 inner join c
 )
 select new_hired,
        count(emp.hiredate)
   from tmp2
 left join emp 
   on tmp2.new_hired = year(hiredate)
 group by new_hired
 order by new_hired

```

测试记录:

```
mysql> with recursive c(n) as
    -> (
    -> select 1 as n
    -> union all
    -> select n + 1 from c where n < 10
    -> ),
    -> tmp1 as
    -> (
    -> select year(min(hiredate)) min_hiredate
    ->   from emp
    -> ),
    -> tmp2 as
    -> (
    -> select min_hiredate + n as new_hired
    ->   from tmp1
    ->  inner join c
    ->  )
    ->  select new_hired,
    ->         count(emp.hiredate)
    ->    from tmp2
    ->  left join emp
    ->    on tmp2.new_hired = year(hiredate)
    ->  group by new_hired
    ->  order by new_hired;
+-----------+---------------------+
| new_hired | count(emp.hiredate) |
+-----------+---------------------+
|      1981 |                  10 |
|      1982 |                   1 |
|      1983 |                   0 |
|      1984 |                   0 |
|      1985 |                   0 |
|      1986 |                   0 |
|      1987 |                   2 |
|      1988 |                   0 |
|      1989 |                   0 |
|      1990 |                   0 |
+-----------+---------------------+
10 rows in set (0.00 sec)

```

## 10.5 生成连续数字

[生成连续数字](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109113113)

# 十一.高级查找

## 11.1 给结果集分页

[按结果集分页](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109525157)

## 11.2 跳过表中的n行

[跳过表中的n行](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109569366)

## 11.3 在外连接中用OR逻辑

[在外连接中用 or 逻辑](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109569429)

## 11.4 确定哪些行是彼此互换的

[确定那些行是彼此互换的](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109601897)

## 11.5 选择前n个记录

[选择前n个记录](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109601926)

## 11.6 找到包含最大值和最小值的记录

[找到包含最大值和最小值的记录](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109613638)

## 11.7 存取"未来"行

[存取“未来”行](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109613697)

## 11.8 轮换行值

[轮换行值](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109636330)

## 11.9 给结果集分等级

[给结果集分等级](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109636366)

## 11.10 抑制重复

[抑制重复](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109670015)

## 11.11 找到骑士值

[找到骑士值](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F109670104)

## 11.12 生成简单的预测

[生成简单的预测](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F110126163)

# 十二.报表和数据仓库运算

## 12.1 将结果集转为一行

[将结果集转置为一行](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F110184847)

## 12.2 把结果集转置为多行

[把结果集转置为多行](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F110231043)

## 12.3 反向转置结果集

[反向转置结果集](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F110386260)

## 12.4 将结果集反向转置为一列

[将结果集反向转为一列](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113246568)

## 12.5 抑制结果集中的重复值

[抑制结果集中的重复值](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113307778)

## 12.6 转置结果集以利于跨行计算

[转置结果集以利于跨行计算](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113369692)

## 12.7 创建固定大小的数据桶

[创建固定大小的桶](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113524185)

## 12.8 创建预定数目的桶

[创建预定数目的桶](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113540800)

## 12.9 创建横向直方图

[创建横向直方图](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113589048)

## 12.10 创建纵向直方图

[创建纵向直方图](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113589279)

## 12.11 返回未包含在group by中的列

[返回未包含在group by中的列](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113644853)

## 12.12 计算简单的小计

[计算简单的小计](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113645053)

## 12.13 计算所有表达式组合的小计

[计算所有表达式组合的小计](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113680352)

## 12.14 判别非小计的行

[分组语句](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F106466884)

根据grouping来判断是否是小计的行

## 12.15 使用case表达式给行做标记

[使用CASE表达式给行做标记](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113680651)

## 12.16 创建稀疏矩阵

需求:
创建一个稀疏的矩阵。

例如这个sql:

```
select  case deptno when 10 then ename end as d10,
        case deptno when 20 then ename end as d20,
        case deptno when 30 then ename end as d30,
        case job when 'CLERK' then ename end as clerks,
        case job when 'MANAGER' then ename end as mgrs,
        case job when 'PRESIDENT' then ename end as prez,
        case job when 'ALALYST' then ename end as anals,
        case job when 'SALESMAN' then ename end as sales
  from  emp;

```

生成了一个稀疏的矩阵

解决方案:
把deptno和job行变换为列，只需使用case表达式，判断由这些行返回的可能性。这就是对它的所有解释。
另外，如果想使报表变'稠密',并去除一些null行，就需要找到分组的依据。

例如，使用串口函数row_number over,为每个deptno中的每个员工分等级，然后使用聚集函数max去掉一些null

```
mysql> select  case deptno when 10 then ename end as d10,
    ->         case deptno when 20 then ename end as d20,
    ->         case deptno when 30 then ename end as d30,
    ->         case job when 'CLERK' then ename end as clerks,
    ->         case job when 'MANAGER' then ename end as mgrs,
    ->         case job when 'PRESIDENT' then ename end as prez,
    ->         case job when 'ALALYST' then ename end as anals,
    ->         case job when 'SALESMAN' then ename end as sales
    ->   from  emp;
+--------+-------+--------+--------+-------+------+-------+--------+
| d10    | d20   | d30    | clerks | mgrs  | prez | anals | sales  |
+--------+-------+--------+--------+-------+------+-------+--------+
| NULL   | SMITH | NULL   | SMITH  | NULL  | NULL | NULL  | NULL   |
| NULL   | NULL  | ALLEN  | NULL   | NULL  | NULL | NULL  | ALLEN  |
| NULL   | NULL  | WARD   | NULL   | NULL  | NULL | NULL  | WARD   |
| NULL   | JONES | NULL   | NULL   | JONES | NULL | NULL  | NULL   |
| NULL   | NULL  | MARTIN | NULL   | NULL  | NULL | NULL  | MARTIN |
| NULL   | NULL  | BLAKE  | NULL   | BLAKE | NULL | NULL  | NULL   |
| CLARK  | NULL  | NULL   | NULL   | CLARK | NULL | NULL  | NULL   |
| NULL   | SCOTT | NULL   | NULL   | NULL  | NULL | NULL  | NULL   |
| KING   | NULL  | NULL   | NULL   | NULL  | KING | NULL  | NULL   |
| NULL   | NULL  | TURNER | NULL   | NULL  | NULL | NULL  | TURNER |
| NULL   | ADAMS | NULL   | ADAMS  | NULL  | NULL | NULL  | NULL   |
| NULL   | NULL  | JAMES  | JAMES  | NULL  | NULL | NULL  | NULL   |
| NULL   | FORD  | NULL   | NULL   | NULL  | NULL | NULL  | NULL   |
| MILLER | NULL  | NULL   | MILLER | NULL  | NULL | NULL  | NULL   |
+--------+-------+--------+--------+-------+------+-------+--------+
14 rows in set (0.00 sec)

```

解决方案:

```
select  max(case deptno when 10 then ename end) as d10,
        max(case deptno when 20 then ename end) as d20,
        max(case deptno when 30 then ename end) as d30,
        max(case job when 'CLERK' then ename end) as clerks,
        max(case job when 'MANAGER' then ename end) as mgrs,
        max(case job when 'PRESIDENT' then ename end) as prez,
        max(case job when 'ANALYST' then ename end) as anals,
        max(case job when 'SALESMAN' then ename end) as sales
  from  ( 
  select deptno, job, ename,
         row_number() over (partition by deptno order by empno) rn
    from emp
   ) x
 group by rn
  ;

```

## 12.17 按时间按单位给行分组

[按时间单位进行分组](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113935397)

## 12.18 对不同组/分区同时实现聚集

[对不同组/分区进行聚集](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113985289)

## 12.19 对移动范围内的值进行聚集

[对移动范围内的值进行聚集](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F113985417)

## 12.20 转置带小计的结果集

[转置带小计的结果集](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114021285)

# 十三.分层查询

## 13.1 表示父子关系

[表示 父-子关系](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114021411)

## 13.2 表示父-子-祖关系

[表示 子-父-祖父关系](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114078449)

## 13.3 创建表的分层视图

[创建表的分层视图](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114078584)

## 13.4 为给定父行找到所有子行

[为给定父行找到所有子行](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114128041)

## 13.5 确定哪些是叶节点、分支节点及根节点

[确定哪些是叶节点、分子节点、根节点](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114128175)

# 十四.若干另类目标

## 14.4 从不固定位置提取字符串元素

[从不固定位置提取字符串元素](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114261191)

## 14.8 转置已分等级的结果集

[转置已分等级的结果集](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114261240)

## 14.9 给两次转置的结果集增加列头

[给两次转置的结果集增加列头](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fu010520724%2Farticle%2Fdetails%2F114310170)







