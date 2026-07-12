```sql
SELECT TIMESTAMPADD(DAY, -7, CURRENT_DATE) 
```

这里以两个需求为例子来说明日期函数的用法

##1. 未来14天的总量
一般的写法就是在第一层select中进行限制,但是这样写的结果是时间rowkey的失效,导致全表扫描,如果要group by的维度枚举少的化,其实也是可以使用的,但是枚举过多的话,就不行了,这里的折中的方案就是嵌套一层,但是也没有从根本上解决这个问题,最好的方法就是由后端来限制时间来解决
```sql
select 
name,sum(persons) as persons
from 
(
select name,d,sum(person_num) as persons
from table
where .... ,d >= '2021-01-01'
group by name,d
) t1 
where d >= CURRENT_DATE AND d <= TIMESTAMPADD(DAY, 14, CURRENT_DATE) 
group by name 
```

##2. 过去14天的趋势
在之前的文章中写过这个需求的SQL的,最开始的方法也是同上面的写法,但是出现了一个问题就是用户输入的天数小于14天,就无法展示14天的趋势,后来改进了一下在第一层嵌套中直接指定时间范围为 d大于等于一个初始值,使用group by d 来解决的,最后最终的解决方法还是借助后端来解决这个问题的

归根结底kylin在分区表的时间特性上的支持不是很好,可以考虑在底层来优化一下数据类型的问题
