
##331. 提取JSON中的value值
1. get_json_object
语法：get_json_object(json_string, ‘$.key’),注意事项：这个函数每次只能返回一个数据项

```
select 
get_json_object('{"company":"阿里","age":18}','$.ompany'),
get_json_object('{"company":"拼多多","age":18}','$.age');
```

2. json_tuple
语法：json_tuple(json_string, k1, k2 …)，解析多个字段不加 `$` [hive：函数：json_tuple处理json数据](https://blog.csdn.net/weixin_38750084/article/details/104443016)

```
如下：
hive> select * from json_test;
select * from json_test
OK
1	{"name":"孙先生","carrer":"大数据开发工程师","dream":["开个便利店","去外面逛一逛","看本好书"],"friend":{"friend_1":"MM","friend_2":"NN","friend_3":"BB","friend_4":"VV"}}
2	{"name":"唐女士","carrer":"退休农民","dream":["儿子听话","带孙子"],"friend":{"friend_1":"CC"}}
```

```
-- 提取二级格式下的数据(如好友1)
select good_friend_1
from temp_db.json_test
lateral view json_tuple(str,'friend') dd as good_friend
lateral view json_tuple(good_friend,'friend_1') tb as good_friend_1;
查询结果：
MM
CC
 
-- 提取标签中所有的内容(没有的标签，返回null)
select good_friend_1,good_friend_2,good_friend_3
from temp_db.json_test
lateral view json_tuple(str,'friend') dd as good_friend
lateral view json_tuple(good_friend,'friend_1','friend_2','friend_3') tb as good_friend_1,good_friend_2,good_friend_3;
 
查询结果：
MM	NN	BB
CC	NULL	NULL
```

```
--dreaming string格式转数组（string不可以直接转数组的，即使string的数据符合数组的格式）
select regexp_replace(regexp_replace(regexp_replace(dreaming, '\\[', ''),'\\]',''),'\\"','')
from temp_db.json_test
lateral view  json_tuple(str,'dream') dd as dreaming;
查询结果：
开个便利店,去外面逛一逛,看本好书
儿子听话,带孙子
```
3. 嵌套子查询解析json数组

```
[{“website”:“baidu.com”,“name”:“百度”},{“website”:“google.com”,“name”:“谷歌”}]

解析为

website	name
baidu.com	百度
google.com	谷歌
```

```
SELECT explode(split(
    regexp_replace(
        regexp_replace(
            '[
                {"website":"baidu.com","name":"百度"},
                {"website":"google.com","name":"谷歌"}
            ]', 
            '\\[|\\]' , ''), 将json数组两边的中括号去掉
            
              '\\}\\,\\{' , '\\}\\;\\{'), 将json数组元素之间的逗号换成分号
                
                 '\\;') 以分号作为分隔符(split函数以分号作为分隔)
          ); 


嵌套一层json_tuple
select json_tuple(json, 'website', 'name') 
from (
select explode(split(regexp_replace(regexp_replace('[{"website":"baidu.com","name":"百度"},{"website":"google.com","name":"谷歌"}]', '\\[|\\]',''),'\\}\\,\\{','\\}\\;\\{'),'\\;')) 
as json) t;
```

4. 使用 lateral view 解析json数组
lateral view通常和UDTF一起出现，为了解决UDTF不允许在select存在多个字段的问题
```
goods_id	json_str
1,2,3	[{“source”:“7fresh”,“monthSales”:4900,“userCount”:1900,“score”:“9.9”},{“source”:“jd”,“monthSales”:2090,“userCount”:78981,“score”:“9.8”},{“source”:“jdmart”,“monthSales”:6987,“userCount”:1600,“score”:“9.0”}]
```

```
select good_id,get_json_object(sale_json,'$.monthSales') as monthSales
from tableName 
LATERAL VIEW explode(split(goods_id,','))goods as good_id 
LATERAL VIEW explode(split(regexp_replace(regexp_replace(json_str , '\\[|\\]',''),'\\}\\,\\{','\\}\\;\\{'),'\\;')) sales as sale_json;
```

##332. 提取JSON中 key值
```
[{“website”:“baidu.com”,“name”:“百度”},{“website”:“google.com”,“name”:“谷歌”}]
```
```
select rn
      ,val as key
from(
    select pos+1 as rn
          ,val
    from(
        select regexp_replace(translate('[{"website":"baidu.com","name":"百度"},{"website":"google.com","name":"谷歌"}]'
                          ,'[]{}""','') ,'\,','\:') as str
    ) t1 lateral view posexplode(split(str,':')) t2 as pos,val
) m 
where rn%2=1 
```
提供了一种方法，主要的思路还是正则和炸裂的使用

```
website:baidu.com:name:百度:website:google.com:name:谷歌
```

这里有个难点是怎么获取奇数值，作者使用了posexplode函数，第一个值是索引，第二是值是value值，我在想其实不用吧  ,--> :


##333. 遍历字符串
1. 'a,b,c,d,e,f'
```
select posexplode(split('a,b,c,d,e,f',','))
```
2. "abcdef"

先根据字符串长度生成对应的索引
```
select posexplode(split(space(length("abcdef")-1),' '))
```
针对生成的索引值按行进行展开
```
select 'abcdef' as str
      ,pos --索引
      ,substr("abcdef",pos+1,1)
      ,substr("abcdef",0,pos+1)
from(
select posexplode(split(space(length("abcdef")-1),' '))
) t
```

（1）利用posexplode()函数生成索引
（2）已知某个长度值，根据长度值生成索引的方法。space()函数及posexplode()函数
（3）通过索引值及substr()函数获取当前索引处的字符串值。


##334. 删除字符串中多余的字符  translate(input, from, to)


input：输入字符串【集是要被替换的字符串】
from：需要匹配的字符【即需要被替换的字符】，这里一定要注意是字符不是字符串
to ：用哪些字符来替换被匹配到的字符

1. 如果 from 字符串长度=to的字符串长度，如translate(“abcdef-abcdef”,“abcdef”,“123456”);替换不是说把"abcdef"替换成"123456"，而是把a替换成1，把b替换成2，把c替换成3，把d替换成4，e替换成5，f替换成6。
2. 如果 from 字符串长度>to的字符串长度 ，例如TRANSLATE(‘abcdef-abcdef’,‘adbc’,‘123’) 意思是把 a替换为1，b替换为2，c替换为3，d替换为空，即删除掉
3. 如果 from里有重复字符 比如abca，1231，重复的字符a对应to的替换不会起作用
4. from长度<to的长度，不报错但是to里面长的字符没有意义。

```
select 
TRANSLATE('abcdef-abcdef','abcd','1234'),--1234ef-1234ef
TRANSLATE('abcdef-abcdef','abcd','123'), --123ef-123ef
TRANSLATE ('abcdaabbaaabbb','aa','12'),--1bcd11bb111bbb
TRANSLATE ('abcdaabbaaabbb','a','123')--1bcd11bb111bbb
```

##335. 分离字符串中的字符和数字 repeat()

1. 转换为指定特征字符或特征数字
```
select data
      ,translate(lower(data),'abcdefghijklmnopqrstuvwxyz',repeat('*',26))
      ,translate(lower(data),'0123456789',repeat('$',10))
from books
```
2. 删除指定的特征字符
```
select data
      ,cast(regexp_replace(translate(lower(data),'abcdefghijklmnopqrstuvwxyz',repeat('*',26)),'\\*','') as bigint) as book_money
      ,regexp_replace(translate(lower(data),'0123456789',repeat('$',10)),'\\$','')  as book_name
from books	
```
##336. 某个字符是否在字符串中 
1. 用like正则匹配
2. 用locate()函数判断
3. 用regexp正则匹配。注意此时regxp()里面的正则为regexp(’.* 1.*’)


##337. HIVE日期函数大全
```
【hive 日期函数 大全】Hive常用日期函数整理
 
注意：1)  hive 没有 to_char函数  2) HIVE 日期函数只识别 年-月-日 不能识别 年-月 ，所以处理月份的时候需要特殊处理
 
1)hive 字符创拼接:
CONCAT(string A, string B…)
SELECT CONCAT('2019','05','11');
2) 字符截取
select substr(add_months(from_unixtime((unix_timestamp('2015-09','yyyy-MM')),'yyyy-MM-dd'),-1),0,7);  
SELECT SUBSTR(from_unixtime((unix_timestamp('201509','yyyyMM')),'yyyyMMdd'),0,6);
SELECT SUBSTR(from_unixtime((unix_timestamp('201509','yyyyMM')),'yyyy-MM-dd'),0,7);
 
hive (felix)> SELECT from_unixtime(unix_timestamp(cast('201509' as string),'yyyyMM'),'yyyy-MM-dd') ;
2015-09-01
Time taken: 0.06 seconds, Fetched: 1 row(s)
hive (felix)> SELECT from_unixtime(unix_timestamp(cast('201509' as string),'yyyyMM'),'yyyy-MM') ;
2015-09
 
select from_unixtime(unix_timestamp(add_months(from_unixtime((unix_timestamp('201509','yyyyMM')),'yyyy-MM-dd'),-1),'yyyy-MM-dd'),'YYYYMM') time_list
 
 
3) 时间区间获取
 
4) 中英文日期转换
hive (default)> select from_unixtime((unix_timestamp('201509','yyyyMM')),'MMM-YY');
OK
Sep-15
Time taken: 0.06 seconds, Fetched: 1 row(s)
 
hive (default)> SELECT from_unixtime(unix_timestamp('Sep-19','MMM-YY'),'yyyyMM');
OK
201812
---英文日期大写
hive (default)> select upper(from_unixtime((unix_timestamp('201509','yyyyMM')),'MMM-YY'));
OK
SEP-15
Time taken: 0.062 seconds, Fetched: 1 row(s)
 
 
 
1.unix_timestamp()
返回当前时区的unix时间戳
返回类型：bigint
hive (default)> SELECT UNIX_TIMESTAMP();
 
2.from_unixtime(bigint unixtime[,string format])
时间戳转日期函数
返回类型：string
hive (default)> select from_unixtime(unix_timestamp(),'yyyyMMdd') ;
20160614
 
3.unix_timestamp(string date)
返回指定日期格式的的时间戳
返回类型：bigint
注意：如果后面只有date参数，date的形式必须为'yyyy-MM-dd HH:mm:ss'的形式。
hive (default)> select unix_timestamp('2020-05-01');
NULL
hive (default)> select unix_timestamp('2020-05-01 00:00:00');
1464710400
 
4.unix_timestamp(string date,string pattern)
返回指定日期格式的时间戳
返回类型:bigint
hive (default)> select unix_timestamp('2020-05-01','yyyy-MM-dd');
1449331200
 
5.to_date(string date)
返回时间字段中的日期部分 必须是yyyy-MM-dd格式
返回类型:string
 
说明: 返回日期时间字段中的日期部分。只能识别到 “年-月-日” 级别的时间，无法识别 “年-月” 级别的时间。
--1) 注意 使用年月，to_date 日期是识别不了的
hive (default)> select to_date('2016-09');  --有问题，结果为null
OK
NULL 
Time taken: 0.067 seconds, Fetched: 1 row(s)
 
hive (default)> select add_months('2020-05-01',-1);
OK
2020-04-01
 
hive (default)> select to_date('2020-05-01 00:00:00') ;
2020-05-01
 
hive (default)> select to_date('2020-05-01');
2020-05-01
 
 
6.year(string date)
返回时间字段中的年
返回类型:int
hive (default)> select year('2020-05-01 00:00:00') ;
2016
hive (default)> select year('2020-05-01') ;
2016
 
7.month(string date)
返回时间字段中的月
返回类型:int
hive (default)> select month('2020-05-01') ;
6
hive (default)> select month('2020-05-01');
6
 
8.day(string date)
返回时间字段中的天
返回类型:int
hive (default)> select day('2020-05-01') ;
1
 
9、day：返回日期中的天
 
select day('2015-04-13 11:32:12');
输出：13
 
10、hour：返回日期中的小时
 
select hour('2015-04-13 11:32:12');
输出：11
 
11、minute：返回日期中的分钟
 
select minute('2015-04-13 11:32:12');
输出：32
 
12、second：返回日期中的秒
 
select second('2015-04-13 11:32:56');
输出：56
 
 
13.weekofyear(string date)
返回时间字段是本年的第多少周
返回类型:int
hive (default)> select weekofyear('2020-05-01') ;
22
 
14.datediff(string enddate,string begindate)
返回enddate与begindate之间的时间差的天数
返回类型:int
hive (default)> select datediff('2020-05-01','2016-05-01') ;
31
--DEMO 2 
hive (default)> SELECT datediff('2020-05-01 00:00:00','2016-05-01 23:00:00');
OK
31
 
15.date_add(string date,int days)
返回date增加days天后的日期
返回类型:string
hive (default)> select date_add('2020-05-01',15) ;
2020-05-16
 
16.date_sub(string date,int days)
返回date减少days天后的日期
返回类型:string
hive (default)> select date_sub('2020-05-01',15) ;
2016-05-17
 
17：Hive中取最近30天数据
datediff(CURRENT_TIMESTAMP ,gmt_create)<=30 
 
18、Hive中 两个日期相差多少小时
select (unix_timestamp('2018-05-25 12:03:55') - unix_timestamp('2018-05-25 11:03:55'))/3600
输出：1
19、Hive中 两个日期相差多少分钟
select (unix_timestamp('2018-05-25 12:03:55') - unix_timestamp('2018-05-25 11:03:55'))/60
输出：60
 
20、hive 计算某一个日期属于星期几，如2018-05-20 是星期日
SELECT IF(pmod(datediff('2018-05-20', '1920-01-01') - 3, 7)='0', 7, pmod(datediff('2018-05-20', '1920-01-01') - 3, 7)) 
输出：7
 
 
21、hive返回上个月第一天和最后一天
--上个月第一天
select trunc(add_months('2020-05-08',-1),'MM');  --只能识别 年-月-日 
select trunc(add_months(CURRENT_TIMESTAMP,-1),'MM')
 
hive (default)> select trunc(add_months('2020-05',-1),'MM'); 
OK
NULL
Time taken: 0.067 seconds, Fetched: 1 row(s)
hive (default)> select trunc(add_months('2020-05-08',-1),'MM'); 
OK
2020-04-01
Time taken: 0.079 seconds, Fetched: 1 row(s)
hive (default)> 
 
 
 
select concat(substr(add_months(from_unixtime(unix_timestamp(),'yyyy-MM-dd'),-1),1,7),'-01'); 
```

##338. 时间序列--补全数据
```
date_id   a   b    c
2014     AB  12    bc
2015         23    
2016               d
2017     BC 

如何使用最新数据补全表格

date_id   a   b    c
2014     AB  12    bc
2015     AB  23    bc
2016     AB  23    d
2017     BC  23    d
```

```
select 
  date_id, 
  first_value(a) over(partition by aa order by date_id) as a,
  first_value(b) over(partition by bb order by date_id) as b,
  first_value(c) over(partition by cc order by date_id) as c
from
(
  select 
    date_id,
    a,
    b,
    c,
    count(a) over(order by date_id) as aa,
    count(b) over(order by date_id) as bb,
    count(c) over(order by date_id) as cc
  from t20
)tmp1;
```

##339. like，not like，rlike， regexp的区别和使用详解
1. Rlike功能和like功能大致一样，like是后面只支持简单表达式匹配（_%）,而rlike则支持标准正则表达式语法。所以如果正则表达式使用熟练的话，建议使用rlike，功能更加强大。所有的like匹配都可以被替换成rlike。反之，则不行。但是注意：like是从头逐一字符匹配的，是全部匹配，但是rlike则不是,可以从任意部位匹配，而且不是全部匹配。
2. NOT A LIKE B是LIKE的结果否定，如果like匹配结果时true，则not…like的匹配结果时false，反之也是结果也是相对。实际中也可以使用 A NOT LIKE B，也是LIKE的否定，与 NOT A LIKE B一样。当然前提要排除出现null问题，null值这个奇葩除外，null的结果都是null值。
3. 同理NOT RLIKE 的使用，也是NOT A RLIKE B是对RLIKE的否定。当然前提要排除出现null问题，null值这个奇葩除外，null的结果都是null值。

##340. 统计字符串中字符的个数
1. 用regexp_replace()函数将要计算的字符替换为’'
2. 用字符串的总长度减去替换字符后的串长度，得到要计算的字符总长度
```
select (length("HELLO HELLO")) - (length(regexp_replace("HELLO HELLO","LL",'')))
```
