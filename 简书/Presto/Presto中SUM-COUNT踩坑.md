##1.使用除法\，结果值为0
原因：在presto中：两个value相除，至少有一个为浮点数才能返回正确结果
解决方案：转为浮点型
（1）select sum(case when storecode = '15' then 1 else 0 end)*1.00 / count(1) from orders;
（2）cast(value AS type)

select * from table where date=20210101
在hive中正常执行，presto中会报错：operator equal(varchar, bigint) are not registered
原因：Presto不支持隐式转换，要求什么格式的参数，就一定得是什么格式的参数
改为select * from table where date=‘20210101’

##2.其他:
在kylin中跑sum()/count() 和在presto中跑相同的sum/count结果不一致,原因分析:发现在kylin中count字段是不忽略null值的,应该怎么解决kylin中的这个问题????(KYLIN中没有avg函数,真是不够用)
目前的方法有:
重新构建一个cube(忽略null值的)
更改avg的口径问题(不可取....)
更改cube中null字段的数据类型
