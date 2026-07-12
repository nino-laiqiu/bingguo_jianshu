获取时间等同于hive的date_add和date_sub
```
SELECT  TIMESTAMPADD(DAY, -7, CURRENT_TIMESTAMP) 
```
```
SELECT TIMESTAMPADD(DAY, -7, CURRENT_DATE) 
```
```
SELECT  TIMESTAMPADD(DAY, -7, date '2021-01-01') 
```
```
SELECT   (YEAR(TIMESTAMPADD(DAY, -7, CURRENT_DATE)) * 10000 + MONTH(TIMESTAMPADD(DAY, -7, CURRENT_DATE)) * 100 + DAYOFMONTH(TIMESTAMPADD(DAY, -7, CURRENT_DATE)) )
```
这个函数其他计算框架都没有,如果想在多个计算框架中都能跑通,建议使用开窗函数排序来获取时间差

时间相差函数
```
TIMESTAMPDIFF(timeUnit, datetime, datetime2)
```
presto中的日期函数
```
presto
date_add('day',-14,cast('2021-01-01' as date)
date_add('day',-14, date'2021-01-01' )
```
求日期间隔函数
```
date_diff('day',cast('2019-04-24' as TIMESTAMP),cast('2019-04-26' as TIMESTAMP))
```
注意与hive的区别
```
presto中 date_diff('day',date1,date2)【后-前】
hive,mysql中 datediff(date1,date2) 【前-后】
```


hive中的常用日期函数和格式化函数
```
SELECT  from_unixtime(UNIX_TIMESTAMP(),'yyyy-MM-dd')
```
hive的常用函数参考如下
>https://blog.csdn.net/u010711495/article/details/110131586?ops_request_misc=%25257B%252522request%25255Fid%252522%25253A%252522160975865616780263058513%252522%25252C%252522scm%252522%25253A%25252220140713.130102334.pc%25255Fblog.%252522%25257D&request_id=160975865616780263058513&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~
