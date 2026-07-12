启动客户端   远程连接启动 见集群配置

hive架构:

![image.png](https://upload-images.jianshu.io/upload_images/9049859-6401f3b822079085.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##交互命令:
> hive -e "select * from tb_user3;" --不进入hive直接查询

>hive -f 文件名(里面是SQL语句)

##1.将本地文件导入Hive及将hdfs文件导入Hive
**(本地)**
>create table student(
  id int, 
  name string)
 ROW FORMAT DELIMITED FIELDS  TERMINATED
 BY '\t';

>load data local inpath '/opt/module/datas/student.txt' into table student;

**(hdfs) 传入到hive就会转移**
>create table if  not exists tb_user3(
uid int ,
name string ,
age int ,
sal double
) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ","--按 , 切割
;

>load data inpath  '/data/user'  into table tb_user3

##insert 方式



##2.DDL语法

#####(1).创建库

> create database dp_database;(默认路径)

#####(2).查找与详情(支持模糊查询)

> show databases  like 'tb*';
>desc database  extended 'tb';(详情)

>查看表结构  desc tb_product ;

>查看表的详细信息   desc formatted tb_product ;

#####(3).建表语句
![image.png](https://upload-images.jianshu.io/upload_images/9049859-51e624bcc2b2be6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![表的展示](https://upload-images.jianshu.io/upload_images/9049859-da98ceaedce4a254.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意事项:查询由元数据表和hdfs构成,只要其中一个存在,下一次创建就默认指定)

**另一种建表语句:**
> create table test.stuinfo3 like test.stuinfo;(和另一张表结构一致)

**未被external修饰的是内部表（managed table），被external修饰的为外（external table)**内部表由hive直接管理当删除表时,元数据和hdfs上储存的数据也会被删除
(外部表和内部表的转换)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-acb1f4226abf48e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**清除数据**
![image.png](https://upload-images.jianshu.io/upload_images/9049859-cfbdb17a9d5f8f72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**查询结果写入本地**

>insert overwrite directory "地址"  查询语句....;


##(4).分区表:

**静态分区(分区字段可作为where查询字段)**
```
create table datatable(
ID int,
date1 string,
num int
)
partitioned  by (dy string)
row format delimited fields terminated by "," ;


load data local inpath "/root/people/1.txt"  into table datatable   partition(dy="2020-06-18");
load data local inpath "/root/people/2.txt"  into table datatable   partition(dy="2020-06-19");
--手动分区,将多路径(txt)导入到同一张表,按路径来存储
```

**(二级分区)**
```
create table datatables(
id int,
date string,
num int
)
partitioned by (dy string,dt string)
row format delimited fields terminated by ",";

load data local inpath "/root/people/a.txt" into table   datatables partition(dy="2020-07",dt="07-12");
load data local inpath "/root/people/a.txt" into table   datatables partition(dy="2020-07",dt="07-13");
load data local inpath "/root/people/a.txt" into table   datatables partition(dy="2020-08",dt="07-12");
load data local inpath "/root/people/a.txt" into table   datatables partition(dy="2020-08",dt="07-13"); 
```

**(动态分区)**
```

create table tb_user(
uid string ,
name  string ,
age int ,
gender string ,
address string 
)
row format delimited fields terminated by " ";
```
```
load  data local  inpath "/root/tb_user.txt"  into table  tb_user;
```
```
create table tb_user1(
uid string,
name string,
gender string,
address string
)
partitioned  by (addr string)
row format delimited fields  terminated by  " " ;
```

>set hive.exec.dynamic.partition=true ;

>set hive.exec.dynamic.partition.mode=nonstrick;  可以从普通表中导入数据

```
//动态导入
insert into tb_user1  partition(addr)  select uid,name,gender,address,address from tb_user;
```

**增加与删除分区**

>alter table tablename add partition(分区name = "")

>alter table tablename drop partition(分区name = "")

>show partitions  tablename;(查询表有多少分区)

