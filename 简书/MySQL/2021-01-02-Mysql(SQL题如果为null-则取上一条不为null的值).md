![image.png](https://upload-images.jianshu.io/upload_images/9049859-44f5df83a3268631.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 方案一
首先把不为null的数据取出来放在数组中,接着对增加一个字段,如果为null,那么就为0否则为1 ,接着原表开窗(sum()) 取数组中arr[rn-1] 的数据
```
price_date,price 表:price

with x as (
select 
collect_list(price) as arr 
from 
price
where price is not null 
),
y as (
select 
price_date,
price,
sum(label) over(order by price_date) as rn 
from 
(
select
price_date,
price,
if(price is null,0,1) as  label
from
price 
) o 
) 
select 
price_date, arr[rn-1] as price
from 
y 
```
```
+----------+-----+
|price_date|price|
+----------+-----+
|2019-09-25|23.23|
|2019-09-26|32.54|
|2019-09-27|34.55|
|2019-09-28|34.55|
|2019-09-29|34.55|
|2019-09-30|54.22|
|2019-10-01|54.22|
|2019-10-02|54.22|
|2019-10-03|54.22|
|2019-10-04|54.22|
|2019-10-05|54.22|
|2019-10-06|54.22|
|2019-10-07|54.22|
|2019-10-08|33.80|
|2019-10-09|33.83|
```

## 方案二
首先排除为null的,把时间lead上移.如果原表的时间在这个区间,那么就取这个区间的时间
发现join重复数据,采用了过滤为null的值,能否采用其他方法
```
with x as (
select
price_date,price,
lead(price_date,1,price_date) over(order by price_date) as price_date1
from
(
select  -- 查找出price不为null的行
price_date,price
from
pricetable
where price != 'null'
) o
)
select
price_date,price
from
(
select
pricetable.price_date,
x.price
from
pricetable,x
where pricetable.price = 'null' and  (pricetable.price_date between x.price_date and x.price_date1)

union all

select
price_date,price
from
pricetable
where price != 'null'
) t1
order by  price_date
```

