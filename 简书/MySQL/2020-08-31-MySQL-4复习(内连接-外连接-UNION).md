##1.图
![image.png](https://upload-images.jianshu.io/upload_images/9049859-40b003cb6d956e6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对该图进行一个简单的代码说明

A,B 表 设为 tb_abc tb_pq
代码如下
```
create table tb_abc(
    A int,
    B int,
    C int
);
create table tb_pq(
                       P int,
                       Q int

);
insert into tb_abc values (3,2,1),(4,2,2),(4,3,3),(2,4,4);
insert into tb_pq values (2,1),(4,2),(3,3);
```
###左连接
```
select *
from tb_abc
left join tb_pq on tb_abc.C=tb_pq.Q
   /* and tb_pq.P is  null*/
 ;
```
![左连接的结果](https://upload-images.jianshu.io/upload_images/9049859-1c84fbbc1f85eff9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以左为标准,值得注意的是左连接可能也会有笛卡尔积的出现,tb_abc如果有m行,查询的结果可能大于等于m行 如下
![结果](https://upload-images.jianshu.io/upload_images/9049859-7402b25152060500.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###右连接
![结果](https://upload-images.jianshu.io/upload_images/9049859-25a26b8d511d92cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对a1 和 b2 的说明:
left join 只获取 a1 b2 非公有数据
代码如下:
```
select *
from tb_abc
left join tb_pq on tb_abc.C=tb_pq.Q
where tb_pq.P is  null
;
```
![结果](https://upload-images.jianshu.io/upload_images/9049859-821b40639435466c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ```
select *
from tb_abc
      right   join tb_pq on tb_abc.C=tb_pq.Q
where C is  null;
```

###内连接
**获取两个表的公有数据,注意与左连接和右链接区别 左连接是以左表标准,依次匹配
没有的字段以null显示**
```
select *
from tb_abc
         join tb_pq on tb_abc.C=tb_pq.Q
;
```
![结果](https://upload-images.jianshu.io/upload_images/9049859-2a9813d610f2762a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**使用join 时系统自动选取数据行段少的为主表**


**注意在左右连接时添加and:
and判断条件为假时，左右连接的主方都不会为null，而从方会被置为null**
代码如下:
```
select *
from tb_abc
       left  join tb_pq on tb_abc.C=tb_pq.Q
    and tb_pq.P is /*not*/ null
;

select *
from tb_abc
          join tb_pq on tb_abc.C=tb_pq.Q
and tb_pq.P is /* not*/ null;
```
显示的结果分别为:
![1](https://upload-images.jianshu.io/upload_images/9049859-9bd5f7634de49f18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![2.空](https://upload-images.jianshu.io/upload_images/9049859-a5c0600bfad3b458.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

mysql是没有全连接的
对4 和 4a 的说明
```
select *
from tb_abc
cross join tb_pq ;
select  *
from tb_abc
join tb_pq ;
select  *
from tb_abc
inner join tb_pq ;
select  *
from tb_abc
/*full*/  join  tb_pq on tb_abc.C=tb_pq.Q;
```

对于MySQL 是没有full join的
代码如下:
```
/*全连接*/
select *
from tb_abc
left join  tb_pq on tb_abc.C=tb_pq.Q
union
select  *
from  tb_abc
 right join  tb_pq on tb_abc.C=tb_pq.Q
````
![结果](https://upload-images.jianshu.io/upload_images/9049859-e4a21b0c36a21dfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**join on 和 join where 的区别**
join on:on后边写条件，以后边的条件为准生成一个临时表存储数据。
join where ：不会生成中间表

问题:
1.如果多条件查询,on后面加多条件 执行函数是and 那么这个and 是在生成m*n表之前判断的还是之后判断的
2.多个left join on使用时的顺序问题
3.查询两表,有共同字段 jion on 和 where 有什么区别


![要点](https://upload-images.jianshu.io/upload_images/9049859-268c1204a8e08ecf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

网课Join连接摘要:
199语法
![199](https://upload-images.jianshu.io/upload_images/9049859-266ecba9331e6c6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![顺序](https://upload-images.jianshu.io/upload_images/9049859-2eb8db7ce6971bac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)









