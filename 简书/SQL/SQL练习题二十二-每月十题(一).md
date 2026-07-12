>该主题告一段落,市面上的SQL题大致我这个主题基本都囊括了,入门绰绰有余,剩余的,这个主题,我争取每个月更新一篇,主要是文本处理,窗口处理,多表处理,多条件处理,另类写法SQL/HQL,其中的题目来源来自面试题 公众号 社区


##261.窗口函数打标签

```
create table miaoying (
     id int,data int
);
insert into miaoying values (1,3),(2.,null),(3,null),(4,null),(5,5),(6,null),(7,null),(8,6),(9,null);

select *
from miaoying;
```
有列id,数据(data),id为主键,规则如下,data中有null的处理规则:如果null行离上面的近就取上面的不为空的数据,如果离下面的近,就取下面的不为空的数据,如果奇数的情况的处理则归属上面,请写出SQL
两种方法一个是array第二种是拉链表的思想,拉链表有两种方法,方法一是直接sum相加,第二种是对null和非null打标签(注意尝试不同的标签打法)

提供一种写法
```
with y  as (select x.id,x.data,x.rn ,lead(data,1,data)  over (order by id) as lead_n from (
                                                                 select * ,sum(data) over(order by id) as rn  from miaoying

                                                                    ) x  where data is not null )
select t2.id,
       case when row_n <= round(count_n/2) then y.data else y.lead_n end as data
from
     (
select * ,
       row_number() over (partition by t1.rn order by t1.id) as row_n,
       count(t1.rn) over (partition by t1.rn) as count_n
from (
select * ,sum(data) over(order by id) as rn  from miaoying )t1 where  t1.data is null ) t2
join y on t2.rn = y.rn
union  all
select id,data from miaoying where  data is not null
```
对于如何求中位数,可以使用正序和逆序或者where条件来判断

##262.分组与窗口
```
create table mioaying1 (
    id int,shoe varchar(1)
);
insert into mioaying1 values (1,'白'),(2,'白'),(3,'白'),(4,'红'),(5,'黑'),(6,'绿'),(7,'绿');
```
有id,鞋子的颜色,求第三列
```
1	白 1
2	白 2
3	白 3
4	红 1
5	黑 1
6	绿 1
7	绿 2
```
提供一种方法,偏移打标签
```
select id,shoe,row_number() over (partition by rn order by id) as number
from (
         select id, shoe, sum(log) over (order by id) as rn
         from (
                  select id, shoe, case when shoe != log then 1 else 0 end as log
                  from (
                           select *, lag(shoe, 1, 1) over () as log
                           from mioaying1
                       ) t1
              ) t2
     ) t3;
```
//注意在打标签0/1是要尝试多种打法,可能答案就出来了
##263.条件过滤
```
create table a21210226 (
    a int ,b int
);

insert  into a21210226 values (1,2),(1,5),(2,1),(2,3),(2,5),(3,4),(3,2),(3,5);
```
a->b是一对多的关系,找出b中同时存在3 5 的 a

```
--子查询
select  distinct  a
from a21210226
where  a in  (select a from a21210226 where b =3 )
and a in  (select a from a21210226 where b =5 );
```
```
--having的过滤,好像不太对,在求取之前要去重一下
select a
from a21210226
group by a
having  sum(if(b in (3,5) ,1,0)) =2;
```
```
-- join连接
select t1.a
from a21210226  t1 join  a21210226 t2 on  t1.a = t2.a and t1.b =3 and t2.b =5 ;
```
```
-- having过滤方法二,也得先去重,防止同一个A有多个相同的B
select a
from (
select  a,b from a21210226 where b =3 or b =5 ) t1
group by a
having count(1) >= 2;
```
##264.打标签
```
CREATE TABLE b20210226 
(TIME INT,
 NAME VARCHAR(5),
 COMPLETE VARCHAR(5)
)
  
INSERT INTO b20210226 VALUES(1,'1','0');
INSERT INTO b20210226 VALUES(2,'1','1');
INSERT INTO b20210226 VALUES(3,'1','1');
INSERT INTO b20210226 VALUES(4,'1','1');
INSERT INTO b20210226 VALUES(5,'1','0');
INSERT INTO b20210226 VALUES(6,'1','0');
INSERT INTO b20210226 VALUES(7,'1','1');
INSERT INTO b20210226 VALUES(8,'1','1');
INSERT INTO b20210226 VALUES(2,'3','1');
INSERT INTO b20210226 VALUES(3,'3','1');
INSERT INTO b20210226 VALUES(4,'3','1');
INSERT INTO b20210226 VALUES(5,'3','0');
INSERT INTO b20210226 VALUES(6,'3','1');
INSERT INTO b20210226 VALUES(7,'3','1');
INSERT INTO b20210226 VALUES(8,'3','0');
```
需要统计每组（NAME）连续时间（TIME）内的连续完成数（COMPLETE），其中有某一时间的完成数为0就重新计算
```
select
TIME,NAME,COMPLETE ,row_number() over (partition by NAME,rn_1 order by  TIME) as re
from (
         select TIME,
                NAME,
                COMPLETE,
                sum(tag) over (order by rn ) rn_1
         from (
                  select *, if(COMPLETE = '0', 1, 0) as tag, row_number() over (order by NAME) as rn
                  from b20210226
              ) t1
     ) t2
where  COMPLETE =1

union  all

select *,'0'
from b20210226
where  COMPLETE ='0'
order by NAME,re;
```

学习一下,我本来的写法是对为0的单独处理,事实上在求出结果之后if来判断一下,即可,我怎么没想到......
```
select TIME,NAME,COMPLETE,if(COMPLETE = '0',0,re)
from (
         select TIME,
                NAME,
                COMPLETE,
                row_number() over (partition by NAME,COMPLETE,rn) as re
         from (
                  select *, TIME - row_number() over (partition by NAME,COMPLETE order by TIME) as rn
                  from b20210226
              ) t1
     ) t2
order by NAME,TIME;
```
改进一下
```
select
TIME, NAME, COMPLETE, if(COMPLETE ='0',0,re)
from (
select
TIME,NAME,COMPLETE ,row_number() over (partition by NAME,COMPLETE,rn_1 order by  TIME) as re
from (
         select TIME,
                NAME,
                COMPLETE,
                sum(tag) over (order by rn ) rn_1
         from (
                  select *, if(COMPLETE = '0', 1, 0) as tag, row_number() over (order by NAME) as rn
                  from b20210226
              ) t1
     ) t2
    ) t3
order by NAME,TIME;
```
##265.文本处理

```
CREATE TABLE c20210226 (地址 VARCHAR(100));

INSERT INTO c20210226 VALUES('北京市东城区273号201房');
INSERT INTO c20210226 VALUES('广州市天河区11号1311房');
INSERT INTO c20210226 VALUES('深圳市福田区992号121房');
```
把数字替换为*
.....有问题,怎么变成一行了,不知道怎么用正则写??

```
SELECT regexp_replace(地址,'[0-9]','*')  FROM c20210226 ;
```
可以使用变量来解决
##266.left join 
```
CREATE TABLE T0222B
(WAREHOUSE VARCHAR(10),
 ITEM VARCHAR(10),
 QTY INT
);

INSERT INTO T0222B VALUES ('A','P001',50);
INSERT INTO T0222B VALUES ('B','P001',30);

CREATE TABLE T0222C
(WAREHOUSE VARCHAR(10),
 ITEM VARCHAR(10),
 QTY INT
);

INSERT INTO T0222C VALUES ('A','P001',10);
INSERT INTO T0222C VALUES ('A','P002',20);
INSERT INTO T0222C VALUES ('C','P001',15);
INSERT INTO T0222C VALUES ('C','P003',10);
```
说明:
表1.仓库,产品,期初余额
表2.仓库,产品,发出金额
表3.仓库,产品,收入

求:仓库,产品的详情和期末余额

```
select
t1.WAREHOUSE as '仓库',t1.ITEM as '产品',ifnull(t1.QTY,0) as '期初' , ifnull(t2.QTY,0) as '发出' ,ifnull(t3.QTY,0) as '收入' ,
       (ifnull(t1.QTY,0) - ifnull(t2.QTY,0) + ifnull(t3.QTY,0) ) as '结存'
from T0222A t1 left join T0222B  t2 on t1.WAREHOUSE = t2.WAREHOUSE and t1.ITEM = t2.ITEM
left join T0222C t3  on   t1.WAREHOUSE = t3.WAREHOUSE and t1.ITEM = t3.ITEM

union all

select
WAREHOUSE,ITEM,0,0,QTY,QTY
from T0222C
where (WAREHOUSE,ITEM) not in (select WAREHOUSE,ITEM from T0222A);
```
有full join 就用这个避免使用union all 连接子查询
##267.循环
定义如下两个变量
DECLARE @date_start datetime
DECLARE @date_end datetime 
set  @date_start = '2021-02-20 01:00:00'    
set  @date_end = '2021-02-20 10:00:00'
希望求解出两个变量直接每小时的时间分布

学习一下时间相减函数
```
select  TIMESTAMPDIFF(hour ,'2021-02-20 01:00:00','2021-02-20 10:00:00' ) ;

select datediff('2021-02-02','2021-01-09');

date_add('2021-02-20 01:00:00, interval 1  hour )
```

提供一种写法,可能跨年的时候不适用
```
with   recursive x(number) as (
        select 0
        union  all
        select  number + 1 from x where  number <  (select  TIMESTAMPDIFF(hour ,'2021-02-20 01:00:00','2021-02-20 10:00:00' ))
    )

select
date_add(t1.times, interval number hour )
from  x cross join ( select '2021-02-20 01:00:00' as times ) t1 ;
```
##268.指标

有一张充值表，先需要根据财务的需求，根据充值日期、有效天数和充值金额分摊到2020年最后一天，即2020年12月31日。
```
CREATE TABLE T0218
(
订单号 VARCHAR(10),
充值日期 DATE,
充值金额 double,
充值产品 VARCHAR(100),
有效天数 INT
)
 
INSERT INTO T0218 VALUES('1001','2020-07-01',500.00,'初一数学提高班',90)
INSERT INTO T0218 VALUES('1002','2020-08-04',1000.00,'成人英语口语突破班',30)
INSERT INTO T0218 VALUES('1003','2020-09-10',2000.00,'初三数学提高班',240)
INSERT INTO T0218 VALUES('1004','2020-11-15',3000.00,'高三语文作文提高班',360)
INSERT INTO T0218 VALUES('1005','2020-12-20',2000.00,'高一物理精讲班',60)
```
在上表的基础上增加两列分摊金额和剩余金额。分摊金额时，包括充值日期和2020年12月31日这两天，即包括头尾日期。

例如2020-09-10这天充值了2000元，从2020-09-10到2020-12-31日这一天总共有113天，实际有效期为240天，那么到2020-12-31日这一天，需要分摊这2000元的金额计算方式为：2000/240*113=941.6629。如果有效天数小于到2020-12-31日这天的天数，那么就全部分摊
```
select *,
       if(datediff('2020-12-31',t1.充值日期 ) >= t1.有效天数 , round(t1.充值金额,4) ,round(t1.充值金额 * datediff('2020-12-31',t1.充值日期 ) / t1.有效天数,4) ) as '分摊金额',
       if(datediff('2020-12-31',t1.充值日期 ) >= t1.有效天数 , 0.0000 ,round(t1.充值金额 -  t1.充值金额 * datediff('2020-12-31',t1.充值日期 ) / t1.有效天数,4)) as '剩余金额'
from T0218 t1;
```
//优化一下,把if中的计算步骤放在里select,在外面嵌套一层select来求取

##269.group by 
```
CREATE TABLE T0205A (
A VARCHAR(20),
B VARCHAR(20)
)

INSERT INTO T0205A VALUES('跑步','张三');
INSERT INTO T0205A VALUES('游泳','张三');
INSERT INTO T0205A VALUES('跳远','李四');
INSERT INTO T0205A VALUES('跳高','王五');

CREATE TABLE T0205B (
A VARCHAR(20),
B VARCHAR(20),
C VARCHAR(10)
)

INSERT INTO T0205B VALUES('跑步','张三','胜');
INSERT INTO T0205B VALUES('游泳','张三','胜');
INSERT INTO T0205B VALUES('跳高','王五','胜');
```
anum表示每个人参加的项目数，bnum表示每个人在各自项目中胜利的次数
```
       anum   bnum
张三	2	2
王五	1	1
李四	1	0

```
```
select
t1.B,
       count(1) as anum,
       count(if(t2.C='胜',1,null)) as bnum
from T0205A t1  left join T0205B t2  on  t1.A = t2.A and t1.B = t2.B
group by  t1.B;
```
##270.上下比较

```
CREATE TABLE T0204 (
ID INT,
Num INT
);

INSERT INTO T0204 VALUES(1,5);
INSERT INTO T0204 VALUES(2,11);
INSERT INTO T0204 VALUES(3,0);
INSERT INTO T0204 VALUES(4,-2);
INSERT INTO T0204 VALUES(5,2);
INSERT INTO T0204 VALUES(6,9);
INSERT INTO T0204 VALUES(7,1);
INSERT INTO T0204 VALUES(8,-4);
INSERT INTO T0204 VALUES(9,-7);
```

当Num中的数据同时大于上下两行数据，返回是
当Num中的数据小于上下两行数据中的任何一行，返回否

例如：11大于5,11大于0，所以返回是
5小于11所以返回否

注意学习窗口中控制大小的用法

```
select *,
case when max(Num) over ( order by id  rows between 1 preceding  and  1 following ) <= Num  then '是' else '否'end
from T0204;
```
方法二可以使用偏移量函数的上移和下移
```
select *,
case when lag(Num) over (order by ID) < Num AND LEAD(Num) over (order by ID) < Num then '是' else '否'end
from T0204;
```

