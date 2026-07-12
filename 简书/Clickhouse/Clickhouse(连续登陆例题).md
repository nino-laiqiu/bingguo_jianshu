#####CH的数据的导入和导出

1 使用集成引擎 
HDFS 
File
MySQL   
KAFKA
2 from  表函数  file  mysql ****  hdfs**** 
3 insert into values
4 cat 本地文件 | clickhouse-client -q 'insert into tb_name  fromat
5 insert into tb_name select from  tb_name ;
6 create table  tmp  engine=Log  as select * from a ;


#####ARRAY JOIN Clause

#####一道例题
```
a,2020-02-05,200
a,2020-02-06,300
a,2020-03-04,400
a,2020-03-05,600
b,2020-02-06,300
b,2020-02-08,200
b,2020-02-09,400
b,2020-02-10,600
c,2020-01-31,200
c,2020-02-01,300
a,2020-02-07,200
a,2020-02-08,400
a,2020-02-10,600
b,2020-02-05,200
a,2020-03-01,200
a,2020-03-02,300
a,2020-03-03,200
c,2020-02-02,200
c,2020-02-03,400
c,2020-02-10,600

-- 将数据导入到CH中 
 create table tb_shop(name String , ctime Date , cost Float64) engine=Log ;
 cat shop.txt  | clickhouse-client  -d 'default' -q 'insert into tb_shop FORMAT CSV'

-- 分析 连续N一天有销售记录的店铺名  4天
```
```

//日期去重(tb_shop)
name` String,
    `ctime` Date,
    `cost` Float64


select distinct ,name,cost ,ctime  from tb_shop;

//编号相减 如果相同则是连续的,取出4
列转行
(select  t1.name,groupArray(t1.ctime)  cte
from
(select distinct  name,cost ,ctime  from tb_shop) t1
group by t1.name)

//编号
select   t2.name ,cte1,cte,rn
from
(select  t1.name,groupArray(t1.ctime)  cte
from
(select distinct  name,cost ,ctime  from tb_shop) t1
group by t1.name) t2
array join t2.cte as  cte1,
arrayEnumerate(cte) as  rn 
order by name,cte1 


//日期相减
select 
name,t3.cte1,t3.cte,t3.rn,
subtractDays(cast(cte1,'Date'),rn) as diff
from
(select   t2.name ,cte1,cte,rn
from
(select  t1.name,groupArray(t1.ctime)  cte
from
(select distinct  name,cost ,ctime  from tb_shop) t1
group by t1.name) t2
array join t2.cte as  cte1,
arrayEnumerate(cte) as  rn 
order by name,cte1 ) t3

//进行计数

select   distinct t5.name
from 
(
select
t4.name,count(1) as con
from
(select 
name,t3.cte1,t3.cte,t3.rn,
subtractDays(cast(cte1,'Date'),rn) as times
from
(select   t2.name ,cte1,cte,rn
from
(select  t1.name,groupArray(t1.ctime)  cte
from
(select distinct  name,cost ,ctime  from tb_shop) t1
group by t1.name) t2
array join t2.cte as  cte1,
arrayEnumerate(cte) as  rn 
order by name,cte1 ) t3) t4
group by t4.name,t4.times
having con>3
) t5
```
```

##连续N一天有销售记录的店铺名  4天
/*name` String,
    `ctime` Date,
    `cost` Float64*/

#首先去重,按照name排序
select distinct *
from tb_shop;


#我要给date标号 但是没有窗口函数 把date转化为数组 使用array join函数获取编号
select t1.name,
       groupArray(t1.ctime) ctime
from (select distinct *
      from tb_shop

      order by name
     ) t1
group by t1.name
order by name;

select t2.name,
       t2.ctime,
       cte,
       cn
from (select t1.name,
             groupArray(t1.ctime) ctime
      from (select distinct *
            from tb_shop
            order by name
           ) t1
      group by t1.name
      order by name) t2 array join  t2.ctime as cte ,arrayEnumerate(t2.ctime) as cn;

#时间相减
select  t3.name,
        t3.ctime,
        t3.cte,
        subtractDays(cast(cte,'Date'),cn) as times
from (
         select t2.name,
                t2.ctime,
                cte,
                cn
         from (select t1.name,
                      groupArray(t1.ctime) ctime
               from (select distinct *
                     from tb_shop
                     order by name
                    ) t1
               group by t1.name
               order by name) t2 array join  t2.ctime as cte ,arrayEnumerate(t2.ctime) as cn
     ) t3;

#分组与计数
select t5.name
from(
select t4.name,count(1) as con
from
(
    select  t3.name,
            t3.ctime,
            t3.cte,
            subtractDays(cast(cte,'Date'),cn) as times
    from (
             select t2.name,
                    t2.ctime,
                    cte,
                    cn
             from (select t1.name,
                          groupArray(t1.ctime) ctime
                   from (select distinct *
                         from tb_shop
                         order by name
                        ) t1
                   group by t1.name
                   order by name) t2 array join  t2.ctime as cte ,arrayEnumerate(t2.ctime) as cn
         ) t3
    )t4
group by t4.name,t4.times
having  con>3)t5;
```
