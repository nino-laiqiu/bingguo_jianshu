##271.九九乘法表
要求:打印九九乘法表
```
with recursive t(n) as (
      select  1
      union  all
      select  n + 1 from t where  n < 9
)

select
group_concat(n order  by n separator ' ')
from (
         select left_n,
                concat_ws('=', concat_ws('*', left_n, right_n), result) as n
         from (
                  select t1.n        as left_n,
                         t2.n        as right_n,
                         t1.n * t2.n as result
                  from t t1
                           join t t2 on t1.n >= t2.n
              ) s1
     ) s2
group by  left_n;
```
在hive中使用sort_array来排序

提供另一种方法
```
with recursive t(n) as (
      select  1
      union  all
      select  n + 1 from t where  n < 9
)

select
max(if(right_n =1,result,'')),
       max(if(right_n =2,result,'')),
       max(if(right_n =3,result,'')),
       max(if(right_n =4,result,'')),
       max(if(right_n =5,result,'')),
       max(if(right_n =6,result,'')),
       max(if(right_n =7,result,'')),
       max(if(right_n =8,result,'')),
       max(if(right_n =9,result,''))
from (
         select t1.n as                                   left_n,
                t2.n as                                   right_n,
                concat(t2.n, '*', t1.n, '=', t1.n * t2.n) result
         from t t1
                  join t t2 on t1.n >= t2.n
     ) s1
group by left_n;
```
关于如何生成1-9的自然数,有方法row_number直接生成,变量生成,mysql的递归,直接union all 来生成

##272.递归表达式和斐波那契数列

>递归表达式的作用
生成斐波那契数列
补全两个日期之间的缺失日期；
树形查询

生成斐波那契数列
```
with  recursive  t(id,first,last) as (
      select 1 as id ,
             1 as first,
             1 as last
             union  all
     select  id +1 ,
             if(id< 2 ,1 ,first + last),
             first

     from t
     where  id < 10
)

select first  from  t;
```
##273.5X5方格棋盘难题
在5X5的方格棋盘中（如图），每行、列、斜线（斜线不仅仅包括对角线）最多可以放两个球，如何摆放才能放置最多的球，这样的摆法总共有几种？输出所有的摆法。
要求：用一句SQL实现

>http://www.itpub.net/thread-1407072-1-1.html
有兴趣的学习一下,这是第一届的题目

##274.行转列问题
现有一表数据如下
```
设备  加油量    加油日期
A1     10       2012-10-1

A2     2        2012-10-2

A3     3        2012-10-1
```
现在想得到10月份每天每台设备的加油汇总表，如下
```
A1	10	10	10	10	10	10
A2	0	3	3	3	3	3
A3	0	0	5	9	9	9
      10	13	18	22	22	22

```
```
create table if not exists cheer(
       equipment  varchar(2),
       fuel_charge  double,
      fuel_datetime  date
);

insert into cheer values ('A1',10,'2021-10-1');
insert into cheer values ('A2',3,'2021-10-2');
insert into cheer values ('A3',5,'2021-10-3');
insert into cheer values ('A3',4,'2021-10-4');
```
也不是什么正规的行转列,提供一种写法
```
with x as (
    select equipment,
           sum(if(fuel_datetime <= '2021-10-01', charge, 0)) as '1',
           sum(if(fuel_datetime <= '2021-10-02', charge, 0)) as '2',
           sum(if(fuel_datetime <= '2021-10-03', charge, 0)) as '3',
           sum(if(fuel_datetime <= '2021-10-04', charge, 0)) as '4',
           sum(if(fuel_datetime <= '2021-10-04', charge, 0)) as '5',
           sum(if(fuel_datetime <= '2021-10-04', charge, 0)) as '6'
    from (
             select equipment,
                    fuel_datetime,
                    sum(fuel_charge) as charge
             from cheer
             where fuel_datetime like '2021-10%'
             group by equipment, fuel_datetime
         ) t1
    group by equipment
)
select  * from x
union  all
select
group_concat(equipment),sum(x.`1`),sum(x.`2`),sum(x.`3`),sum(x.`4`),sum(x.`5`),sum(x.`6`)
from x ;
```

发现这种写法不符合题意使用打标签的方法来统计
```
select
equipment,
           sum(if(fuel_datetime = '2021-10-01', charge, 0)) as '1',
           sum(if(fuel_datetime = '2021-10-02', charge, 0)) as '2',
           sum(if(fuel_datetime = '2021-10-03', charge, 0)) as '3',
           sum(if(fuel_datetime = '2021-10-04', charge, 0)) as '4',
           sum(if(fuel_datetime = '2021-10-05', charge, 0)) as '5',
           sum(if(fuel_datetime = '2021-10-06', charge, 0)) as '6',
           sum(charge) as '1-31'
from (
         select 1 as 'type', cheer.equipment, cheer.fuel_charge as charge , cheer.fuel_datetime
         from cheer where fuel_datetime like '2021-10%'

         union all

         select 2 as 'type', '日统计' , sum(fuel_charge), fuel_datetime
         from cheer
         where
         fuel_datetime like '2021-10%'
         group by fuel_datetime
     ) t1
group by type,equipment;
```
##275.混合排序
有点难度....
![示例](https://upload-images.jianshu.io/upload_images/9049859-dcc9c04f82aa081b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

name 是店铺名称，名称中带有“-”表示分店，score 是销售额。出题人希望能依据城市、销售额查看各个店铺的销售数据，并且当存在分店时，分店能紧挨在总店后面按照 id 排序
```
create table order_data (
     id int ,
     city varchar(1),
     name  varchar(4),
     score int
);

insert into order_data values (1,'a','A',100);
insert into order_data values (2,'a','A-1',80);
insert into order_data values (3,'b','C',70);
insert into order_data values (4,'a','A-2',90);
insert into order_data values (5,'b','D',85);
insert into order_data values (6,'b','B',75);
insert into order_data values (7,'b','E',30);
insert into order_data values (8,'b','B-1',50);
insert into order_data values (9,'a','F',95);
insert into order_data values (10,'b','G',65);
```
非常巧妙,提供一种方法
```
with  x as (
     select
     id, city, name, score,
           if( instr(name,'-'),substr(name,1,1),name  ) as base_name
     from order_data
)

select
x.id,x.city,x.name,
x.score
from order_data join x  on  order_data.name  = x.base_name
order by x.city,order_data.score desc,x.id;
```
##276.获取状态一致的分组
简单题
星星点灯是一家水果店，它提供了外卖水果拼盘的服务。水果店能够提供四种水果拼盘：水果魔方、海星欧蕾、猫头鹰、草莓雪山，下表反应了某一时刻店内的水果的准备情况。
```
create  table  if not exists fruit_data (
       id int,
       platter  varchar(6),
       fruit    varchar(6),
       ready    int
);
insert into fruit_data values (1,'水果魔方','猕猴桃',1);
insert into fruit_data values (2,'水果魔方','香蕉',1);
insert into fruit_data values (3,'水果魔方','菠萝',1);
insert into fruit_data values (4,'水果魔方','芒果',1);
insert into fruit_data values (5,'水果魔方','哈密瓜',1);
insert into fruit_data values (6,'水果魔方','猕猴桃',1);
insert into fruit_data values (7,'海星欧蕾','草莓',0);
insert into fruit_data values (8,'海星欧蕾','橙子',1);
insert into fruit_data values (9,'猫头鹰 ','猕猴桃',1);
insert into fruit_data values (10,'猫头鹰 ','小橙子',1);
insert into fruit_data values (11,'猫头鹰 ','橘子',1);
```
上面这些数据存在 platters 表中，platter 是拼盘的名称，fruit 是拼盘要用到的水果，ready 表示水果是否准备好了。当有客户订水果拼盘时，只有拼盘要用到的所有水果都准备好了才能制作。
现在，我们要写 SQL 找出可以立即制作的水果拼盘的名称。
```
select
fruit_data.platter
from fruit_data
group by platter
having sum(if(ready=1,0,1)) =0 ;

select
fruit_data.platter
from fruit_data
group by platter
having sum(if(ready=1,1,0)) = count(*);


select distinct platter
from fruit_data
where platter not in (
select
platter
from fruit_data
where ready =0 ) ;
```
##277.mysql中的更新语法
更新语法没怎么用过,学习一下
```
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
    SET assignment_list
    [WHERE where_condition]
    [ORDER BY ...]
    [LIMIT row_count]
```
你会发现有limit 和 order by 语法,实践一下
```
create  table  update_test (
     id int ,
     test0 int ,
     test1 int ,
     test2 int
);

insert into update_test values (1,1,1,1);
insert into update_test values (2,1,1,1);
insert into update_test values (3,1,1,1);
insert into update_test values (4,1,1,1);
insert into update_test values (5,1,1,1);
```

```
update  update_test
set test0 =2
limit 2;

update  update_test
set test1 =2
order by  id desc
limit 2;

update  update_test
set
    test1 =10 * test0,
    test2 = test1
limit 2;
```
不过，需要注意的是，如果更新的行的原来的值和要更新的值一致，那么 MySQL 并不会真正执行更新操作，但仍会计入受 LIMIT 子句影响的行数。

注意update和子查询的结合的注意事项
>ERROR 1093 (HY000): You can't specify target table 'xxx' for update in FROM clause
##278.最短路径
从出发地到目的地有多条路线可走，要求使用算法找出最短路径。
如果使用的是 SQL ，怎么解决这类问题？
```
sp      ep      distance  
------  ------  ----------
a       b                5
a       c                1
b       c                2
b       d                1
c       d                4
c       e                8
d       e                3
d       f                6
```
```
create table max_data (
     sp varchar(1),
     ep varchar(1),
     distance int
);
insert into max_data values ('a','b',5);
insert into max_data values ('a','c',1);
insert into max_data values ('b','c',2);
insert into max_data values ('b','d',1);
insert into max_data values ('c','d',4);
insert into max_data values ('c','e',8);
insert into max_data values ('d','e',3);
insert into max_data values ('d','f',6);

```
递归的思路:
先获取与a的直接间接(目前有 a -> b、a -> c 这两条路线。),与原表join,条件是 t 表的ep = 原表的sp,获取原表的ep

在 SQL 中加入了条件 INSTR(t.path, b.ep) <= 0 , 主要是防止有环路而出现死循环
```
with  recursive  t(sp,ep,distance,path) as (
      select * ,cast(concat(sp,'-->',ep) as CHAR(100)) as path   from max_data where sp = 'a'

      union  all
      select
      t.sp ,t1.ep,t1.distance + t.distance ,cast(concat(t.path ,'-->',t1.ep) as CHAR(100)) as path
      from t
          join max_data t1 on  t.ep = t1.sp and instr(t.path , t1.ep) <= 0
)

select sp,ep,distance,path
from  (
select  * ,row_number() over (partition by ep  order by  distance) as rn
from t ) t1
where rn =1;
```
##279.背包问题
这是一道简化的背包问题：有一背包能容纳 50kg 的物品，现有 9 种物品（它们的重量分别是 5kg、8kg、20kg、35kg、41kg、2kg、15kg、10kg、9kg），要刚好能装满背包，有多少种物品组合？
```
create  table if not exists  knapsack (
     id int ,
     weight int
);

insert into  knapsack values (1,5);
insert into  knapsack values (2,8);
insert into  knapsack values (3,20);
insert into  knapsack values (4,35);
insert into  knapsack values (5,41);
insert into  knapsack values (6,2);
insert into  knapsack values (7,15);
insert into  knapsack values (8,10);
insert into  knapsack values (9,9);
```
提供一种写法,递归来做
```
with recursive  t(first_id,count_num,id_path,num_path,next_id) as (
       select
       id as fistst_id,
       weight as count_num,
       cast(id as char(100)) as id_path,
       cast( weight as char(100)) as num_path ,
       id as next_id
       from knapsack

    union  all

    select
    t.first_id,
    t.count_num + t1.weight,
    cast( concat(t.id_path ,'--',t1.id) as char(100))  ,
    cast( concat(t.num_path,'--',t1.weight) as char(100)) ,
    t1.id
    from knapsack t1 ,t
    where t.next_id < t1.id and t.count_num + t1.weight <= 50
)
select  * from t  where count_num = 50 ;
```
做了三道递归,发现自连接有妙用啊,学习一下
##280.分析大盘走势
下表（stock）记录了某指数过去一段时间的收盘价，我们要从这张表中找出收盘价持续上涨的日期

希望得到的结果
```
2020-11-24----2020-11-26	3362	3408
2020-11-27----2020-11-30	3391	3451

```
```
create  table  grail (
    deal_date date,
    price   double
);

insert  into grail values ('2020-11-20',3424);
insert  into grail values ('2020-11-23',3402);
insert  into grail values ('2020-11-24',3362);
insert  into grail values ('2020-11-25',3369);
insert  into grail values ('2020-11-26',3408);
insert  into grail values ('2020-11-27',3391);
insert  into grail values ('2020-11-30',3451);
insert  into grail values ('2020-12-01',3449);
insert  into grail values ('2020-12-02',3441);
```
```
select
concat(min(deal_date),'----',max(deal_date)),
       min(price),
       max(price)
from (
         select deal_date,
                price,
                sum(lag_p) over (order by deal_date) as lag_s
         from (
                  select *,
                         if(price - lag(price, 1, price) over (order by deal_date) <= 0, 1, 0) as lag_p
                  from grail
              ) t1
     ) t1
group by  lag_s
having count(1) >= 2;
```
和前面的穿鞋子那题一个套路,打标签,注意lag/lead拉取比较时的变化
另一种方法是求取所有的时间组合然后进行比较
