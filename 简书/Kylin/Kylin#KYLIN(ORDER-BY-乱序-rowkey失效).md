需求的描述:
表中有每个城市按月份汇总的流失率,要按照日历选择器的endTime来取月份

报错:`null while executing SQL`,
wiki上的说明:https://issues.apache.org/jira/browse/KYLIN-4731
```
SELECT  sim_cityname,sum(score) as j
from
table
where cityname = '上海'   and travel_month =  substring(@endTime,7,1)  and   ishome = 1
group by sim_cityname
order by j desc
limit 20
```
改进的写法,对rowkey的处理,以及遇到的另一个问题,就是乱序的问题,这个的处理的方法就是嵌套一层
```
select 
row_number() over() as rn ,sim_cityname
from 
(
select
sim_cityname,j
from 
(
SELECT  sim_cityname,travel_month ,sum(score) as j
from
table
where cityname = '上海'     and   ishome = 1
group by sim_cityname, travel_month  
) t1 
where travel_month =  substring(@endTime,7,1)  
) t2 
```

总结:
1. 要对rowkey处理的话,尽量在运算之后进行处理,防止索引失效,处理的步骤是先运算后过滤,而不是先过滤后运算
2. 在使用order by 后仍然乱序,可限制limit,但是不知道枚举数量的情况下就不适用了,这样可使用窗口来排序
3. 嵌套一层select 来避免乱序处理上的问题,事实上是不影响查询速度的

补充:
###  几个sql报错原因 汇总
`Can't create EnumerableAggregate! while executing SQL`
由distinct count引起的错误

`null while executing SQL`
join的左右两个表之间有相互依赖的关系

`missing a new olapcontext while executing SQL`
需要用大表join小表

`Division undefined`
除法中，分母为空，对为空情况进行单独处理(bigint)

`No realization found for OLAPContext`
查询的维度或指标不存在与cube中

`cte内部支持 order by 1,2,3， 不支持 group by 1,2,3`
