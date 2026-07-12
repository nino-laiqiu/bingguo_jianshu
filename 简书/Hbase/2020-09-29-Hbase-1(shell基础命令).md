1.DDL命令
alter, alter_async, alter_status, clone_table_schema, create, describe, disable, disable_all, drop, drop_all, enable, enable_all, exists, get_table, is_disabled, is_enabled, list, list_regions, locate_region, show_filters

#####(1)list  列出Hbase中存在的所有表
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9be169f63c63dc15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> list_regions 'tab_4'(列出表的regions信息)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-e25aaceef6f042c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>locate_region  查看某行数据在哪个region中

![image.png](https://upload-images.jianshu.io/upload_images/9049859-c0619e79d58e12aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####(2)create 创建表

>create "tab_3","num1","num2"


![image.png](https://upload-images.jianshu.io/upload_images/9049859-46af302ce94a91ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>create "tb_b" , {NAME => "cf1" , TTL=>60 , VERSIONS=>3} ,{NAME=>"cf2"}

NAME:列族名
TTL:数据储存时间
VERSIONS:存储几个数据副本
{}几个列族
![image.png](https://upload-images.jianshu.io/upload_images/9049859-71b552d17f3e5a4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##alter 修改列簇(column family)模式

#####(3)增加一个列簇
> alter 'tab_3',NAME=>'num3',VERSIONS=>4  增加一个列簇f1,每个region会多一个store

![image.png](https://upload-images.jianshu.io/upload_images/9049859-e081ba03a991589d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####(3)删除一个列簇

>alter 'tab_3', NAME => 'num3',METHOD => 'delete'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-f574c7deefb32547.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>alter 'tab_3', 'delete' => 'num2'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-356e3f49d2720b15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###(4)启用与删除
先禁用后删除
disable 禁用表
disable_all 禁用多张表
drop   删除表 
drop_all   删除多张表
enable
enable_all  启用多张表
exists 是否存在
is_disabled
is_enabled


##2.NAMESPACE命令

  alter_namespace,               修改名称空间的属性

  create_namespace,               创建

  describe_namespace,           查看名称空间的属性

  drop_namespace,                  删除名称空间 drop_namespace  "Linux15"

  list_namespace,                  列出系统中所有的名称空间

  list_namespace_tables          列出指定名称空间下所有的表

##3.DML命令

#####(1)append

>append 'tab_2','10001','status:name','bbb'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-b3184bfc27f3588a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>count 'tab_2'(统计多少行---行的定义)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-42ed3a99a5d0e211.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##(2)scan
>scan 'tab_2'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-0b35cdfc83660ec6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>scan 'tab_2' , {STARTROW=>'10001' , LIMIT=>3 , COLUMN=>'status2'} 

![image.png](https://upload-images.jianshu.io/upload_images/9049859-8e1c7c8594fdc9aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-6a6389f4108c63d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(重要!!!! 查看所有数据)

> scan 'tab_1' ,{RAM=> true ,VERSIONS=>10}
(注意删除不是真正的删除只是打一个标签,如果要增添数据的时间戳小于被删除数据的时间戳,则不能put进去 时间戳!!!!重要)

##(3)get 获取数据

获取整行数据 

>get 'tab_2','10001'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-3ea89c686497816e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


获取行的某个列族  

get 'tab_2','10001','status'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-6d28354e617068bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



获取行的多个列族

>get 'tab_2','10001','status','status2'


![image.png](https://upload-images.jianshu.io/upload_images/9049859-70c2edf8b11fb46a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


获取行的某个单元格  

>get 'tab_2','10001','status:name'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-641f03b0817875db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


获取行的多个列族中的多个单元格   

> get 'tab_2','10001','status:name','status:sex','status2:age'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-3faa77a1840764db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-125844b31dc6d37c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####(4)delete 删除数据
只能删除某行的某个列族的属性  只能删除单元格

> delete 'tab_2','10001','status:name'

![image.png](https://upload-images.jianshu.io/upload_images/9049859-71a68ccf7878ecd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####(5)deleteall

>deleteall tab_2  ,10001  删除整行

>deleteall tab_2  ,10001,status:name  删除单元格

#####(6)get_splits 获取表的切割点

>get_splits 'tab_4'

![get_splits 'tab_4'](https://upload-images.jianshu.io/upload_images/9049859-07abe30ff9db5a8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>get 'tab_1','rk10001',{COLUMN=>'info:name',VERSIONS=>3}

![image.png](https://upload-images.jianshu.io/upload_images/9049859-75808c26fc648e0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意这里的get 版本不管用要视创建表时的版本号,如下(修改版本数)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a69c16d1ad51c827.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>list_regions 'tab_4'查看一张表的所有的region信息

![list_regions 'tab_4'](https://upload-images.jianshu.io/upload_images/9049859-de98a6ccde67e323.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

预分region表(以什么标准划分)
>create "tb_s" , "cf" , SPLITS => ['4','7']
