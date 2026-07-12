##null对数据处理的影响

建表:
```
create  table  tb_test(
    name varchar(10),
    age  varchar(10)
);
insert  into  tb_test  values ('小明','12') , ('小题','13'),('宵夜',null);

create  table  tb_test1(
    name varchar(10),
    age  varchar(10)
);
insert  into  tb_test1  values ('小湖','12') , ('小小','13'),('小',null);
```



```
# count(字段) 忽略null值
select  count(age) from tb_test;
select  count(1) from tb_test;
```



!=会过滤值为null的数据,慎用!= null
SQL实际使用 is NULL 和 is not NULL判断字段为空，注意为空不代表为”(空字符串)或为0。
而NULL = NULL和NULL <> NULL其实返回的都是 FALSE，任何值和NULL做运算的结果都是false。
```
select  * from tb_test1 where age != null;
select  * from  tb_test1 where  age is not  null;
```


join时 and和where的区别在null上,and是在join右表时判断,会过滤出为null的值.但是左表中未null的任然会显示

where 是在join之后进行判断,过滤出join之后某字段为空的值
```
select  *
from tb_test left join tb_test1 on tb_test.age = tb_test1.age and  tb_test1.age is not  null;

select  *
from tb_test left join tb_test1 on tb_test.age = tb_test1.age where  tb_test1.age is not  null;
```
