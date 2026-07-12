求取top_20 city人次的均价,使用presto
```

select  city,sum(price)/sum(person_num) as av_price
from
table_test 
where ..... and ....
group by city
order by sum(person_num) desc 
limit 20 

```
使用这种写法,presto无法跑通,使用其他写法,例如

```
select  city,sum(price)/sum(person_num) as av_price
from
table_test
where ..... and ....
group by city
order by av_price desc 
limit 20 
```
使用均价来替换....貌似更改了需求,人次多不代表均价大或者小

使用嵌套一层来处理
```
select city, prices/person_nums as av_price
from 
(
select  city,sum(price) as prices, sum(person_num) as person_nums
from
table_test
where ..... and ....
group by city
order by  sum(person_num) desc 
limit 20 
) t1 
```
报错是因为排序了,如果不排序直接取top或者使用窗口写

但是不一定排序了,就会报错,如果在select中有单独的sum度量,且order by 这个度量就不会报错,例如在求取top_20占比案例中就不会报错
```
select  city, cast(person_nums * 1.0 /sum(person_nums) over() as Decimal(9,4))as ro
from
(
select  city,sum(person_num) as person_nums
from
table_test
where ..... and ....
group by city
order by  sum(person_num) desc 
limit 20 
) t1 
```
