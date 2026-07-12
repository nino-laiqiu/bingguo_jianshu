业务要求按照某类型的同比的绝对值来排序

之前在kylin中遇到的一个问题是order by 排序后不加limit会导致的乱序的问题
```
select  to_city_name , sum(person)
from  table 
where d = '2021-01-01' and to_prov_name = '安徽'
group by to_city_name
order by sum(person) desc 
limit 10 
```
上面的是正确的写法,要限制limit,另外关于这条SQL的注意事项还有如果person为int类型,会导致sum的超过int的最大值,所以要求load 到kylin中sum度量的不能是int,得超过int

这里的绝对值排序,之前的写法是
```
select  .... lag
from 
(
select  (person_sum -pre_sum )/per_sum  as lag
from  table
where 的= '2021-01-01' and zone >= 1
) t1 
order by abs(lag) desc 
```
结果导致乱序,后来使用case when 对 lag进行正负的判断
```
case when lag >= 0 then lag else -lag end as lag_1
```
