>牛客网的题目都是截图,我懒得截图了直接标明题目,请在牛客网中查找实际序号吧...
下面的题目都是之前出于各种原因在笔记上的题目,再写一遍希望有所收获

##161.获取当前薪水第二多的员工(18)
使用子查询来获取,查找出最大的薪资,然后在剩余的薪资中查找出最大的即是第二大的值
序列函数也可以
```
select 
employees.emp_no,salary,last_name,first_name
from 
employees join salaries on employees.emp_no = salaries.emp_no
where salary = (
select
max(salary)
from 
salaries
where salary < (select max(salary) from   salaries)
) 
```
##162.统计各部门工资条数(22)
我为什么会记录这道题目???这道题目可以考察更多的内容,比如说涨工资的情况等
```
select
t1.dept_no ,t1.dept_name,count(salary) as `sum`
from
departments t1 join  dept_emp t2 on t1.dept_no = t2.dept_no
               join  salaries t3 on t2.emp_no = t3.emp_no
group by t1.dept_no ,t1.dept_name
order by t1.dept_no
```
##163.对所有员工薪水进行排名(23)☆
自连接分数排名,使用窗口也可(rank),学习使用自连接解决窗口问题,比如说sum()开窗,使用having来过滤即可,比如这题的序列排名,还有分组topN问题,使用自连接限制where的条件即可(结合group by 分组获取的条数来判断),面试常问!!!!!!
```
select
t1.emp_no ,t1.salary,count(distinct t2.salary) as t_rank 
from
salaries t1 cross join salaries t2 
where t1.salary <= t2.salary
group by t1.emp_no ,t1.salary
order by t_rank
```
##164.获取员工的薪水比其领导还高的员工相关信息(25)
第一次写的比这个复杂??第二次写还是不够好,看了一下评论区可以采用再join 一张薪水表来实现,可能速度会快一点?然而并不,不知道在大数据中哪个快一点??
```
select
t1.emp_no,t3.emp_no  manager_no , t2.salary  emp_salary , t3.salary  manager_salary
from
dept_emp t1 join salaries t2 on t1.emp_no = t2.emp_no
join 
(
select
dept_manager.dept_no as dept_no,salary,dept_manager.emp_no emp_no
from
dept_manager join salaries on dept_manager.emp_no = salaries.emp_no
) t3 
on t1.dept_no = t3.dept_no
where t2.salary > t3.salary
```
```
select de.emp_no, dm.emp_no as manager_no,
s1.salary as emp_salary, s2.salary as manager_salary
from dept_emp de, dept_manager dm, salaries s1, salaries s2
where de.dept_no = dm.dept_no
and de.emp_no = s1.emp_no
and dm.emp_no = s2.emp_no
and s1.salary > s2.salary
```
##165.找出没有分类的电影id和其他信息(29)
这题我以前写错了....这题考察在left join 条件下 on where 条件的区别,on会返回没有匹配上的行(也就是说始终返回左边的数据),而where会过滤null的值,还有这题也考察了 is null != null的区别,!=null始终会返回true,所以尽量使用is (not) null
```
select
t1.film_id ,title
from 
film t1 left join film_category t2 on t1.film_id = t2.film_id
where category_id is null
```
##166.删除重复emp_no的记录,只保留最小id的记录(42)
这题目在leedcode中也有学习delete语法,牛客网还有其他的DML语法练习题,注意delete在mysql中的语法,注意方法二中的表达
```
delete from titles_test
where id not in (
  select * from (
select
min(id)
from titles_test
group by emp_no ) l ) 
```
```
delete t1  from titles_test t1 , titles_test t2
where t1.emp_no = t2.emp_no and t1.id > t2.id
```
##167.牛客新登录用户的次日成功的留存率(68)
对于这种留存率的问题,就是指标的处理,一般采用的函数有lag/lead,不同窗口大小下的开窗进行比较等,以后专门有一篇博客来示例这种指标值
但是这题....过于简单...就是连续登陆的变形题,序列函数即可...不用窗口更简单

```
select
round(count(distinct t1.user_id ) /(select count(distinct user_id) from login ) ,3)  p
from 
(
select
user_id,min(date) as date
from 
login
group by user_id
) t1 
join login on t1.user_id = login.user_id
where datediff(login.date,t1.date) =1
```
评论区:查询最小的天数+1 where联合查询看是否存在这一天

```
select 
round(count(distinct user_id)*1.0/(select count(distinct user_id) from login) ,3)
from login
where (user_id,date)
in (select user_id,DATE_ADD(min(date),INTERVAL 1 DAY) from login group by user_id);
```
##168.统计牛客每个日期登录新用户个数(69)(窗口写法)

注意sum/count与if一起用的时候要注意0的使用count也会+1所以这里是sum,如果要使用count则应该把0改为null然后在外面嵌套一个ifnull
```
select
date,
sum(if(rn = 1,1,0)) new 
from 
(
select 
date,rank() over(partition by user_id order by date) rn 
from 
login
) t1
group by date
```
也可以使用老方法区每个人最早登录的时间然后按照时间group by ,接着获取全部的时间,不能join上的就为0
```
select a.all_date as date, ifnull(count(b.user_id),0) as new
from
    (select distinct(date) as all_date
    from login) as a
left join
    (select user_id, min(date) as reg_date
    from login
    group by user_id) as b 
on a.all_date = b.reg_date
group by a.all_date
```
##169.每个日期新用户的次日留存率(70)
使用union 来代替一次join连接

```
(
select
t1.date ,round(count(t2.user_id) / count(t1.user_id) ,3)  p
from 
(
select
user_id,min(date) as date
from 
login
group by user_id
) t1 
left join login t2 on datediff(t1.date,t2.date) = -1  and t1.user_id = t2.user_id
group by t1.date
)
union 
select date,0.000 p from login 
where date not in (select min(date)
from login
group by user_id)
order by date
```

```
(
select
t1.date ,round(count(t2.user_id) / count(t1.user_id) ,3)  p
from 
(
select
user_id,min(date) as date
from 
login
group by user_id
) t1 
left join login t2 on DATE_SUB(t1.date,INTERVAL - 1 DAY) = t2.date  and t1.user_id = t2.user_id
group by t1.date
)
union 
select date,0.000 p from login 
where date not in (select min(date)
from login
group by user_id)
order by date
```
##170.请你找出每个岗位分数排名前2的用户(74)
分组topN的问题,使用自连接having过滤来获取topN,二是使用窗口来求
```
select
grade.id,name,s1.score
from 
grade
join 
(
select
t1.language_id,t1.score
from
grade t1 join grade t2 on t1.language_id = t2.language_id and t1.score <= t2.score
group by t1.language_id,t1.score
having count(distinct t2.score) <= 2
) s1 
on grade.language_id = s1.language_id and grade.score = s1.score 
join 
language 
on grade.language_id = language.id
order by name,score desc 
```
```
select gl.id,gl.name,gl.score
from (select g.id,l.name,g.score,
dense_rank() over(partition by g.language_id order by g.score desc) as ran
from grade g  join language l
on g.language_id = l.id) gl
where gl.ran < 3
order by gl.name
```
##171.中位数位置的范围(75)
第76同这一题都是求中位数,leedcode中也有类似的题目,使用两个序列函数来求,值得注意的是如果排两个,要排序的字段有重复的,按照字典顺序顺序和逆序会乱序,要指定第二个人字段desc/asc
这道题目之前写复杂了,直接借助于round的四舍五入来求即可..不需要使用case when 来分类讨论
```
select
job,round(count(score)/2) start ,round((count(score) +1) /2) end
from
grade
group by job
order by job
```
##一些函数的使用
```
limit的用法 limit 1,2 从位移1截取取2个
```
```
length( '10,A,B') - length(REPLACE( '10,A,B',',',''));
```
```
--负数的使用,从后面开始查询
select 
first_name
from
employees
order by  substr(first_name,-2,2) 
```
```
--group_concat函数的使用大致等同于concat和collect_list的结合
select 
dept_no,
group_concat(emp_no)
from
dept_emp
group  by dept_no
```
```
--limit的分页功能
select *
from employees
limit 5, 5;
``
