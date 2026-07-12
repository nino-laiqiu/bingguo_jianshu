简单的描述一下这个需求:就是求取部分在总体中的占比,SQL如下,这是我一开始写的SQL
```sql
select 
'标签' as type , cast (persons * 1.0 /lag_1 as Decimal(4,2)  ) as  ra
from 
(
select
persons,lead(persons,1,null) over(persons asc) as lag_1
from (
select sum(person_num) as persons 
from 
tabe_部分
where d >= '2021-01-09' and d <= '2021-01-09'
and ....
group by booking

union all 

select sum(person_num) 
from 
tabe_总体
where d >= '2021-01-09' and d <= '2021-01-09'
and ....
group by booking ) a  ) b 
where lag_1 is not null 
```
上面的SQL犯了好几个错误

1. kylin不支持union all 一个字段的情况,应该打一个标签或者group by的那个维度,至于是什么原因导致的还没有查找出来
2. lead中不支持 0 和null,所以不要传三个参数,传两个参数或者本身即可
3. union all 的下部分,注意最好使用一些括号括起来,有些情况下,会导致错误

正确的写法
```sql
select 
'标签' as type , cast (persons * 1.0 /lag_1 as Decimal(4,2)  ) as  ra
from 
(
select
persons,lead(persons,1) over(persons asc) as lag_1
from (
select 'tag2',sum(person_num) as persons 
from 
tabe_部分
where d >= '2021-01-09' and d <= '2021-01-09'
and ....
group by booking

union all 
(
select 'tag2',sum(person_num) 
from 
tabe_总体
where d >= '2021-01-09' and d <= '2021-01-09'
and ....
group by booking  ) ) a  ) b 
where lag_1 is not null 

```
