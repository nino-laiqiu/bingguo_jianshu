>这里的题目都是之前博客篇hive实践里面的题目,来源于简书 csdn 公众号 ,其中的题目都是常规题,较好的巩固了case when 语法  高级聚合函数  行转列/列转行  窗口函数/窗口大小

hive远程连接步骤:要开启hdfs和hive的元数据管理,后台运行
hive --service metastore &
hiveserver2  & 
##251.集合数据类型
建表
```
create table test(
    MobilePlatform string,
    APP array<string>,
    name map<string,int>,
    address  struct<street:string, city:string>
)
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';
#语句的解释
1.行列分隔符
2.MAP STRUCT 和 ARRAY 的分隔符(数据分割符号)
3.MAP中的key与value的分隔符
4.行分隔符
```
数据如下
```
apple,jianshu_weibo,xiaoming:19_xiaohua:17,1000A_zhongguo
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
```
查询结果与查询方法
```
apple	["jianshu","weibo"]	{"xiaoming":19,"xiaohua":17}	{"street":"1000A","city":"zhongguo"}
songsong	["bingbing","lili"]	{"xiao song":18,"xiaoxiao song":19}	{"street":"hui long guan","city":"beijing"}
```
```
#array的查询方法使用下标
select APP[1] from listtest;

#map的查询方法使用,查询指定K
select name['xiaoxiao song']  from listtest;

#struct的查询方法,使用属性.K
select  address.street from listtest;
```
##252.窗口语法
>over关键字用来指定函数执行的窗口范围，若后面括号中什么都不写，则意味着窗口包含满足WHERE条件的所有行，窗口函数基于所有行进行计算；如果不为空，则支持以下4中语法来设置窗口。
①window_name：给窗口指定一个别名。如果SQL中涉及的窗口较多，采用别名可以看起来更清晰易读；
②PARTITION BY 子句：窗口按照哪些字段进行分组，窗口函数在不同的分组上分别执行；
③ORDER BY子句：按照哪些字段进行排序，窗口函数将按照排序后的记录顺序进行编号；
④FRAME子句：FRAME是当前分区的一个子集，子句用来定义子集的规则，通常用来作为滑动窗口使用
```
#例如
select name,orderdate,cost,
sum(cost) over() as fullagg, --所有行相加
sum(cost) over(partition by name) as fullaggbyname, --按name分组，组内数据相加
sum(cost) over(partition by name order by orderdate) as fabno, --按name分组，组内数据累加 
sum(cost) over(partition by name order by orderdate rows between unbounded preceding and current row) as mw1   --和fabno一样,由最前面的起点到当前行的聚合 
sum(cost) over(partition by name order by orderdate rows between 1 preceding and current row) as mw2,   --当前行和前面一行做聚合 
sum(cost) over(partition by name order by orderdate rows between 1 preceding and 1 following) as mw3,   --当前行和前边一行及后面一行 
sum(cost) over(partition by name order by orderdate rows between current row and unbounded following) as mw4  --当前行及后面所有行 
over (order by score range between 2 preceding and 2 following) 窗口范围为当前行的数据幅度减2加2后的范围内的数据求和。
from order; 
```
>控制窗口大小的使用
PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点
##253.窗口的简单使用
#####TopN案例
```
select 
*
from 
(
select 
*,
row_number() over(partition by subject order by score desc) rmp
from score
) t
where t.rmp<=3;
```
#####连续登陆案例
```
1,2020-01-01
1,2020-01-02
1,2020-01-03
1,2020-01-04
1,2020-01-07
1,2020-01-08
9,2020-01-08
7,2020-01-15
3,2020-01-09
4,2020-01-12
1,2020-01-09
2,2020-02-09
2,2020-02-10
2,2020-02-11
2,2020-02-12
2,2020-02-14
2,2020-02-15
4,2020-01-11
4,2020-01-13
4,2020-01-15

#建表与SQL
create table sigin1(
userid int,
sigindate string
)row format delimited
fields terminated by ",";
load data local inpath '/root/user1.txt' into table sigin1;
```
```
select
userid
from (
         select userid,
                date_sub(sigindate , rn) as num_rn
         from (
                  select *, row_number() over (partition by userid order by sigindate) as rn from sigin1
              ) t1
     ) t2
group by userid,num_rn
having count(1)  >= 3 ;
```
#####练习
```
#测试数据
jack,2017-01-01,10
tony,2017-01-02,15
jack,2017-02-03,23
tony,2017-01-04,29
jack,2017-01-05,46
jack,2017-04-06,42
tony,2017-01-07,50
jack,2017-01-08,55
mart,2017-04-08,62
mart,2017-04-09,68
neil,2017-05-10,12
mart,2017-04-11,75
neil,2017-06-12,80
mart,2017-04-13,94

#建表与需求
create table business
(
name string, 
orderdate string,
cost int
)ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

#加载数据
load  data local inpath "/root/business.txt" into table business;
```
#####1、查询在2017年4月份购买过的顾客及总人数(不去重)
```
select
collect_set(name) as name,count(1) as count_p
from business
where substr(orderdate,1,7) = "2017-04";
```
#####2、查询顾客的月购买总额,查询月购买明细
```
???直接group by 就行了,这里是练习一下窗口
select
name,total_amount
from
(
select
*,
row_number() over (partition by name ,substr(orderdate,1,7)) as rank
from
(
select
*,
sum(cost) over(partition by name,substr(orderdate,1,7) ) total_amount
from
business) t1 )t2
where rank =1;
```
#####3、查询顾客的购买明细及到目前为止每个顾客购买总金额
```
select *,
sum(cost) over (partition by name order by orderdate rows between unbounded preceding  and current row )
from business;
```
#####4、查询顾客上次的购买时间
```
select
*,
lag(orderdate,1,0) over (partition by name order by orderdate) as Thelastimetobuy
from business;
```
#####5、查询前20%时间的订单信息
```
select
name,orderdate,cost
from
(
select
*,
ntile(5) over (order by  orderdate) as portion
from business) t1
where portion =1;
```
##254.行转列,列转行
>CONCAT(STRING A ,STRINFG B):返回字符串的连接后的结果
CONCAT_WS(separator, str1, str2,…):这是一个特殊的CONCAT(),第一个参数是分隔符
COLLECT_SET():函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段
CONCAT_WS(SEPARATOR ,COLLECT_SET(column)) ===>GROUP_CONCAT（）函数

```
#数据
孙尚香,白羊座,A
司马懿,射手座,A
吕布,白羊座,B
貂蝉,白羊座,A
许褚,射手座,A

#需求
射手座,A    司马懿|许褚
白羊座,A    孙尚香|貂蝉
白羊座,B    吕布

#建表和SQL
create table person_info(
name string,
constellation string,
blood_type string)
row format delimited fields terminated by ",";
load data local inpath "/root/f.txt" into table person_info;
```
```
select
//concat(constellation,",",blood_type),
concat_ws(",",constellation,blood_type) as list,
concat_ws("|",collect_set(name)) as name
from
person_info
group by  constellation,blood_type;
```

>EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。
LATERAL VIEW:用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合
split(str , 分隔符):返回一个数组
```
#SQL
#数据
《疑犯追踪》 悬疑,动作,科幻,剧情
《Lie to me》	悬疑,警匪,动作,心理,剧情
《战狼2》 战争,动作,灾难

#需求
《疑犯追踪》      悬疑
《疑犯追踪》      动作
《疑犯追踪》      科幻
《疑犯追踪》      剧情
《Lie to me》   悬疑
《Lie to me》   警匪
《Lie to me》   动作
《Lie to me》   心理
《Lie to me》   剧情
《战狼2》        战争
《战狼2》        动作
《战狼2》        灾难

#建表与SQL
create table movie_info(
movie string,
category array<string>)
row format delimited fields terminated by "|"
collection items terminated by ",";

load data local inpath "/root/g.txt" into table movie_info;
```
```
select
movie
result
from movie_info
lateral view
explode(category) result as result
```

##255.case when(if)语法
>CASE 字段 WHEN 值1 THEN 值1 [WHEN 值2 THEN 值2] [ELSE 值] END
CASE WHEN 条件表达式 THEN 值1 [WHEN 条件表达式 [and or] 条件表达式THEN 值2] [ELSE 值] END
##256.case when的简单使用
```
#数据与需求
(用户工资组成表,基本工资,基金,提成)
1,2000,3000,1500,1
2,5000,500,1000,2
3,1500,1000,3000,2
4,3000,6000,8000,3
5,1500,2000,1800,1
6,2500,1000,1900,1
(部门表)
1,销售
2,技术
3,行政
(员工信息表)
1,zs,M,28
2,ww,F,36
3,zl,F,48
4,pp,M,44
5,wb,M,32
6,TQ,F,32

create table gz(
uid int,
jb int,
jj int,
tc int,
deptno int
)
row format delimited fields terminated by ",";
load data local inpath "/root/gz.txt" into table gz;

create table bm(
deptno string ,
name string
)
row format delimited fields terminated by ",";
load data local inpath "/root/bm.txt" into table bm;

create table yg(
uid int,
name string,
gender string,
age int
)
row format delimited fields terminated by ",";
load data local inpath "/root/yg.txt" into table yg;

1.求出公司中每个员工的姓名 和 三类收入中最高的那种收入的类型(greatest()函数)
with x as (
select
uid, greatest(jb, jj, tc)as `max`,
//case  when   greatest(jb, jj, tc) = jb then 'jb'  when greatest(jb, jj, tc) = jj then 'jj' when  greatest(jb, jj, tc) = tc then 'tc' else '_' end  as  gz_category,
case   greatest(jb, jj, tc)  when jb then 'jb' when  jj then 'jj' when tc then 'tc' else '_' end  as gz_category1
from gz )
select  yg.uid,max,gz_category1
from  yg  join  x on yg.uid = x.uid;

2.求出公司中每个岗位的薪资总和
with x as (
select
deptno,
sum(jj+tc+jb)  sum_gz
from
gz
group by deptno)
select
bm.name,x.sum_gz
from bm
join  x on x.deptno = bm.deptno;

3.求出公司中不同性别、不同年龄阶段（20-30,31-40,41-50）的员工薪资总和
with x as
(
select
uid,
gender,
case when age>20 and age <=30 then '20-30' when age>30 and age<=40 then '31-40' when age > 41 and age <=50 then '41-50' else '_' end  as stage
from yg )
select
gender,stage,sum(jb+jj+tc) as `max`
from  gz  join  x  on gz.uid = x.uid
group by  x.gender,x.stage
order by stage desc ;
```
##257.一道面试题
```
#数据与需求
现有这么一批数据，现要求出：  
每个用户截止到每月为止的最大单月访问次数和累计到该月的总访问次数  
三个字段的意思：  
用户名，月份，访问次数 
A,2015-01,5
A,2015-01,15
B,2015-01,5
A,2015-01,8
B,2015-01,25
A,2015-01,5
A,2015-02,4
A,2015-02,6
B,2015-02,10
B,2015-02,5
A,2015-03,16
A,2015-03,22
B,2015-03,23
B,2015-03,10
B,2015-03,11
create table if not exists infos(name string, date string, ftime int) 
row format delimited fields terminated by ",";
load data local inpath "/root/j.txt" into table infos;

#sql(窗口写法)
select
name,mydate,
max(t1.ints) over (partition by name order by mydate),
t1.ints,
sum(ints) over (partition by name order by mydate rows  between  unbounded preceding and  current row )
from
(
select
name,mydate,sum(ints) as ints
from infos1
group by  name,mydate) t1;

sql(考虑使用自连接实现)
//核心where datea <= dateb
select nameb, dateb, visitb,
max(visita) as max_visit,
sum(visita) as sum_visit
from (
select
a.name as namea,
a.mydate as datea,
a.visit as visita,
b.name as nameb,
b.mydate as dateb,
b.visit as visitb
from (select name,mydate,sum(ints) as visit from infos1 group by name,mydate) a join
(select name,mydate,sum(ints) as visit from infos1 group by name,mydate)  b
on a.name = b.name
 ) t1
where datea <= dateb
group by nameb, dateb, visitb;
```
##258.分区和分桶


#####分区
```
#案例
#数据user.txt
u001 ZSS 23 M beijing
u002 YMM 33 F nanjing
u003 LSS 43 M beijing
u004 ZY 23 F beijing
u005 ZM 23 M beijing
u006 CL 23 M dongjing
u007 LX 23 F beijing
u008 YZ 23 M beijing
u009 YM 23 F nanjing
u010 XM 23 M beijing
u011 XD 23 F beijing
u012 LH 23 M dongjing

#建普通表语句
create  table  if not exists  tb_user(
uid string ,
name  string ,
age int ,
gender string ,
address string
)
row format delimited fields  terminated by  " " ;

##加载数据
load data local  inpath  "/root/user.txt"  into table  tb_user ;

##创建目标表
create  table  if not exists  tb_p_user(
uid string ,
name  string ,
age int ,
gender string ,
address string
)
partitioned  by (addr string)
row format delimited fields  terminated by  " " ;

#开启动态分区设置
set hive.exec.dynamic.partition=true ;
set hive.exec.dynamic.partition.mode= nonstrict;  可以从普通表中导入数据

#插入数据(这里是全部插入,也可以插入普通表的部分字段)
普通表5个字段
分区表 5个主字段 1 个分区字段
插入数据的时候字段个数类型一致  最后一个字段就是分区字段 
insert into tb_p_user partition(addr) select uid , name , age , gender , address,address  from  tb_user ;
```
#####分桶
>分区针对的是数据的存储路径；分桶针对的是数据文件,
分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区，特别是之前所提到过的要确定合适的划分大小这个疑虑
分桶对数据的处理比分区更加细粒度化；
分桶和分区两者不干扰，可以把分区表进一步分桶；

```
create table stu_buck(id int, name string)
clustered by(id)
into 4 buckets
row format delimited fields terminated by '\t';

set hive.enforce.bucketing = true;

load data local inpath '/root/e.txt' into table
stu_buck;
```
##259.小文件处理方法
```
set hive.merge.mapfiles = true ##在 map only 的任务结束时合并小文件
set hive.merge.mapredfiles = false ## true 时在 MapReduce 的任务结束时合并小文件
set hive.merge.size.per.task = 256*1000*1000 ##合并文件的大小
set mapred.max.split.size=256000000; ##每个 Map 最大分割大小
set mapred.min.split.size.per.node=1; ##一个节点上 split 的最少值
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; ##执行 Map 前进行小文件合并
```
```
#每个Map最大输入大小(这个值决定了合并后文件的数量)
set mapred.max.split.size=256000000;

#一个节点上split的至少的大小(这个值决定了多个DataNode上的文件是否需要合并)
set mapred.min.split.size.per.node=100000000;

#一个交换机下split的至少的大小(这个值决定了多个交换机上的文件是否需要合并)
set mapred.min.split.size.per.rack=100000000;

#执行Map前进行小文件合并
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;

#===设置map输出和reduce输出进行合并的相关参数：

#设置map端输出进行合并，默认为true
set hive.merge.mapfiles = true

#设置reduce端输出进行合并，默认为false
set hive.merge.mapredfiles = true

#设置合并文件的大小
set hive.merge.size.per.task = 256*1000*1000

#当输出文件的平均大小小于该值时，启动一个独立的MapReduce任务进行文件merge。
set hive.merge.smallfiles.avgsize=16000000
```
```
#用来控制归档是否可用
set hive.archive.enabled=true;
#通知Hive在创建归档时是否可以设置父目录
set hive.archive.har.parentdir.settable=true;
#控制需要归档文件的大小
set har.partfile.size=1099511627776;

#使用以下命令进行归档
ALTER TABLE srcpart ARCHIVE PARTITION(ds='2020-04-08', hr='12');

#对已归档的分区恢复为原文件
ALTER TABLE srcpart UNARCHIVE PARTITION(ds='2020-04-08', hr='12');

#注意，归档的分区不能够INSERT OVERWRITE，必须先unarchive
```
##260.补充事项

>substr(a,b)：从字符串a中，第b位开始取，取右边所有的字符
substr(a,b,c)：从字符串a中，第b为开始取，取c个字符,b可为负数从后面数

>str_to_map(‘a:1,b:2,c:3’);字符串转map
select size(str_to_map(‘a:1,b:2,c:3’));返回map的元素个数
map_keys(str_to_map(‘a:1,b:2’));返回key

>unix_timestamp() 日期转换为当前时间戳
from_unixtime(t1,yyyy-MM-dd HH:mm:ss)时间戳转变为日期格式
from_unixtime(unix_timestamp(date_created),yyyy-MM-dd HH:mm:ss)来规范时间的格式
