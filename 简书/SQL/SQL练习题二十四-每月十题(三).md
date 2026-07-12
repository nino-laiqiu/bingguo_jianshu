##281.获取定长连续子序列
```
create table savior (
     id int , status int
);

insert  into savior values (1,1);
insert  into savior values (2,1);
insert  into savior values (3,0);
insert  into savior values (4,0);
insert  into savior values (5,0);
insert  into savior values (6,1);
insert  into savior values (7,0);
insert  into savior values (8,0);
insert  into savior values (9,0);
insert  into savior values (10,0);
insert  into savior values (11,0);
insert  into savior values (12,1);
insert  into savior values (13,1);
insert  into savior values (14,0);
insert  into savior values (15,0);
```
要求：从 savior 表中获取状态为 0 的 id，并且这些 id 能够组成长度为 3 的连续子序列。
```
3~5     
7~9     
8~10    
9~11   
```

提供一种写法,这种题目在上一篇中有类似的题目,发现最后一行是不符合题意的,怎么有效率过滤掉是难题....想不到确实不好求....
```
select
concat(id-1,'---',id+1)
from (
         select id,
                status,
                max(status) over (order by id rows between 1 preceding and 1 following ) as max_number
         from savior
     ) t1
where  max_number =0 ;
```
方法二:使用自连接也可以求取,答案提供的是row_number获取重复的便签,使用自连接来获取连续的值
```
WITH cte AS 
(SELECT 
  *,
  row_number() over (
ORDER BY id) AS rn 
FROM
  savior 
WHERE STATUS = 0) 
SELECT 
  CONCAT_WS('~', a.id, b.id) AS subseq 
FROM
  cte a 
  INNER JOIN cte b 
    ON a.id + 2 = b.id 
    AND a.rn + 2 = b.rn 
```
方法三.使用偏移量函数来求取
 学习一下max()等窗口函数嵌套if等,到底判断的是什么???例如
```
   select id,
                status,
                max(if(status = 1  , 5,status)) over (order by id rows between 1 preceding and 1 following ) as max_number
         from savior
```
只要max中有一个数是满足的那么结果就是5,!!!!!!
##282.动态规划
>有100天的商品价格,按照买入卖出的顺序,怎么操作能赚得最多的价值,以及是多少
```
create table dtgh (
        data int
);
insert into dtgh values (1);
insert into dtgh values (3);
insert into dtgh values (4);
insert into dtgh values (10);
insert into dtgh values (9);
insert into dtgh values (7);
insert into dtgh values (12);
insert into dtgh values (1);
insert into dtgh values (2);
insert into dtgh values (1);
insert into dtgh values (1);
```
这是一道动态规划题,使用SQL怎么写
```
select
concat(min(data),',',max(data)) ,max(data) - min(data)
from (
         select data,
                sum(`lag`) over (order by rn ) as money
         from (
                  select data,
                         if(data - num < 0, 1, 0) as `lag`,
                         row_number() over ()     as rn
                  from (
                           select data,
                                  lag(data, 1, data + 1) over () as num
                           from dtgh
                       ) t1
              ) t2
     ) t3
group by money
having count(1) > 1 and ( max(data) - min(data) > 0  );
```

##283.层级查询(一)
在工作中都遇到过存在层次关系的数据表，典型的例子诸如菜单表（多级菜单）、用户表（拥有上下级关系）、商品类目表（多级类目）
```
 empno  ename      mgr  
------  ------  --------
  7369  SMITH       7902
  7499  ALLEN       7698
  7521  WARD        7698
  7566  JONES       7839
  7654  MARTIN      7698
  7698  BLAKE       7839
  7782  CLARK       7839
  7788  SCOTT       7566
  7839  KING      (NULL)
  7844  TURNER      7698
  7876  ADAMS       7788
  7900  JAMES       7698
  7902  FORD        7566
  7934  MILLER      7782
```
其中，mgr 为 NULL 表明该员工没有上级领导。
我们要把每个员工的所有上级领导都找出来，实现的效果如下：
```
 empno  ename   path                  
------  ------  ----------------------
  7369  SMITH   ->FORD->JONES->KING   
  7499  ALLEN   ->BLAKE->KING         
  7521  WARD    ->BLAKE->KING         
  7566  JONES   ->KING                
  7654  MARTIN  ->BLAKE->KING         
  7698  BLAKE   ->KING                
  7782  CLARK   ->KING                
  7788  SCOTT   ->JONES->KING         
  7839  KING                          
  7844  TURNER  ->BLAKE->KING         
  7876  ADAMS   ->SCOTT->JONES->KING  
  7900  JAMES   ->BLAKE->KING         
  7902  FORD    ->JONES->KING         
  7934  MILLER  ->CLARK->KING    
```
对于编号为 7369 的 SMITH，他的直属领导的编号是 7902，姓名叫做 FORD；FORD 的直属领导叫做 JONES，编号为 7566；编号为 7566 的直属领导是编号为 7839 的 KING，而 KING 没有直属领导。因此，SMITH 的上级领导的关系链构成：->FORD->JONES->KING 
```
WITH RECURSIVE leader_path(empno, ename, mgr, path) AS 
(SELECT 
  empno,
  ename,
  mgr,
  CAST('' AS CHAR(100)) AS path 
FROM
  emp 
UNION ALL 
SELECT 
  a.empno,
  a.ename,
  b.mgr,
  CONCAT(
    a.path,
    IFNULL(CONCAT('->', b.ename), '')
  ) 
FROM
  leader_path a 
  LEFT JOIN emp b 
    ON a.mgr = b.empno 
WHERE b.empno IS NOT NULL) 
SELECT 
  empno,
  ename,
  path 
FROM
  leader_path 
WHERE mgr IS NULL 
ORDER BY 1 
```
>在递归中一定要加入终止条件，本 SQL 的终止条件是 WHERE b.empno IS NOT NULL；
遇到字符串拼接需要提前设置该字段的长度，对应到 SQL 中的操作是 CAST('' AS CHAR(100))；
递归会生成中间结果，我们要把中间结果过滤掉，WHERE mgr IS NULL 就是只获取最终的结果。
##284.层级查询(二)
在mysql中实现层次查询的两种方式。层级查询(一)举的示例是获取从叶子点到根节点的路径，层级查询(二)要实现的是从根节点找到所有叶子节点。
```
WITH RECURSIVE leader_path (empno, ename, mgr, lv) AS 
(SELECT 
  empno,
  ename,
  mgr,
  1 AS lv
FROM
  emp WHERE mgr IS NULL
UNION ALL 
SELECT 
  b.empno,
  b.ename,
  b.mgr,
  lv + 1
FROM
  leader_path a 
  INNER JOIN emp b 
    ON a.empno = b.mgr ) 
SELECT 
  empno,
  ename,
  lv 
FROM
  leader_path 
ORDER BY 1 
```
结果如下:
```
 empno  ename       lv  
------  ------  --------
  7369  SMITH          4
  7499  ALLEN          3
  7521  WARD           3
  7566  JONES          2
  7654  MARTIN         3
  7698  BLAKE          2
  7782  CLARK          2
  7788  SCOTT          3
  7839  KING           1
  7844  TURNER         3
  7876  ADAMS          4
  7900  JAMES          3
  7902  FORD           3
  7934  MILLER         3
```
##285.获取一行中多个字段的最大值
```
    id      v1      v2      v3  
------  ------  ------  --------
     1     100      80       102
     2       2     -20        -1
     3     999      12       111
     4    1234    2222      -123
     5     871     888       666
     6    -210       9      1024
     7       0      -1         0
     8       2       2         2
```
查询结果
```
    id   v_max  
------  --------
     1       102
     2         2
     3       999
     4      2222
     5       888
     6      1024
     7         0
     8         2
```
方法一:GREATEST()函数
```
SELECT 
  id,
  GREATEST(v1, v2, v3) AS v_max 
FROM
  chaos
```
方法二:嵌套的 IF 语句
```
v12 = IF(v1 > v2, v1, v2)
v_max = IF(v12 > v3, v12, v3)
```
即:
```
--有点复杂.....
SELECT 
  id,
  IF(
    IF(v1 > v2, v1, v2) > v3,
    IF(v1 > v2, v1, v2),
    v3
  ) AS v_max 
FROM
  chaos
```
方法三:使用union all 扁平化
```
WITH chaos_union AS 
(SELECT 
  id,
  v1 AS v 
FROM
  chaos 
UNION ALL 
SELECT 
  id,
  v2 AS v 
FROM
  chaos 
UNION ALL 
SELECT 
  id,
  v3 AS v 
FROM
  chaos)
SELECT 
  id,
  MAX(v) AS v_max 
FROM
  chaos_union 
GROUP BY id 
```
##286.分位函数
>hive中求中位数,我一般是用正序和逆序来解决,这里提供hive中的分位函数来求取中位数

>https://blog.csdn.net/Jarry_cm/article/details/82185576?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161631858216780265426998%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161631858216780265426998&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~

percentile：percentile(col, p) col是要计算的列（值必须为int类型），p的取值为0-1，若为0.2，那么就是2分位数，依次类推。
percentile_approx：percentile_approx(col, p)。列为数值类型都可以。
percentile_approx还有一种形式percentile_approx(col, p，B)，参数B控制内存消耗的近似精度，B越大，结果的精度越高。默认值为10000。当col字段中的distinct值的个数小于B时，结果就为准确的百分位数


##287.打标签(1)
>下面两题是携程数仓的面试题
```
create table if not exists  xiechen  (
     time_day  date,product varchar(4) ,profit int
);

insert into xiechen values ('2021-01-01','A',2);

insert into xiechen values ('2021-01-02','A',3);
insert into xiechen values ('2021-01-03','A',4);
insert into xiechen values ('2021-01-04','A',5);
insert into xiechen values ('2021-01-01','B',2);
insert into xiechen values ('2021-01-02','B',4);
insert into xiechen values ('2021-01-03','B',3);
insert into xiechen values ('2021-01-04','B',4);
insert into xiechen values ('2021-01-01','B',5);
```
求每组收益连续增加3天以上的天数
```
select concat(min(time_day),'--',max(time_day)) , max(product)
from (
         select time_day,
                product,
                profit,
                sum(lag_rn) over (partition by product order by time_day) as rn
         from (
                  select time_day,
                         product,
                         profit,
                         if(profit - lag(profit, 1, profit) over (partition by product) <= 0, 1, 0) as lag_rn
                  from xiechen) t1
     )  t2
  group by product,rn
  having  count(rn) >= 3;
```
##288.打标签(2)
题目:在第一题的基础上,求日期是连续的且收益是连续3天递增的
解答:
在第一题的基础上在group by 后的having后增加:having  count(rn) >= 3 and datediff(max(time_day),min(time_day)) + 1 = count(rn);
只要datediff(max(time_day),min(time_day)) 间隔数等于总的行数就是答案
```
select concat(min(time_day),'--',max(time_day)) , max(product)
from (
         select time_day,
                product,
                profit,
                sum(lag_rn) over (partition by product order by time_day) as rn
         from (
                  select time_day,
                         product,
                         profit,
                         if(profit - lag(profit, 1, profit) over (partition by product) <= 0, 1, 0) as lag_rn
                  from xiechen) t1
     )  t2
  group by product,rn
  having  count(rn) >= 3 and datediff(max(time_day),min(time_day)) + 1 = count(rn);
```
##289.魔力猫盒面试题
```
主粮,渴望,0.45
主粮,哈根纽翠斯,0.15
主粮,爱肯纳,0.40
罐头,仪亲,0.05
罐头,麦克劳德医生,0.8
罐头,happy100,0.15
零食,仪亲,0.24
零食,益智选,0.33
零食,mikbone,0.20
零食,巅峰,0.23
```

要求：求取amount_percent从大到小，累计和大于等于75%,的情况，且以大于等于75%为界限
```
select cat_name,brand_name,amount_percent from(
   select
   cat_name,brand_name,amount_percent,calculate_percent,
   lag(calculate_percent,1,0) over(partition by cat_name order by calculate_percent) as lag_persent
   from
        (
         select
         cat_name,brand_name,amount_percent,
         sum(amount_percent) over(partition by cat_name order by amount_percent desc) as calculate_percent    
         from group_precent_top_n) as a1
      ) as a2
where (calculate_percent >= 0.75 and lag_persent < 0.75) or
(calculate_percent < 0.75 and lag_persent < 0.75)
;
```
其实不必这么写,这道题考查是沟通需求,过滤出特殊的数据行或者改写需求
提供其他方法:
1. 使用hive中的桶函数分20个桶
2. 如上面的解答方法使用sum窗口来求
3. 使用row_number() 和count() 窗口来求
4. 使用自连接来求

##290.简单用户画像
有表sale:order_id 订单id,唯一字段   user_id 用户id  product_id 订单id create_time 订单的创建时间 ,product_num 订单数
有表user_info user_id 用户id sex age

求:使用一条SQL生成完整的用户画像包含,用户id sex age d7order_num  d14_order_num.后面两个字段分别为近七天的订单数,近14天的订单数

使用sum(if)来写
```
select t1.user_id ,
       sex,
       age,
       sum(if(datediff('2021-3-21',create_time)<=7,product_num,0)) as d7order_num ,
       sum(if(datediff('2021-3-21',create_time)<=14,product_num,0)) as d14order_num
from user_info t1 left join sale t2 on t1.user_id = t2.user_id
group by t1.user_id, sex, age;
```
使用join来写
```
select t1.user_id,sex,age,t2.d14order_num,t3.d7order_num
from user_info t1 left join (
      select  user_id ,sum(product_num) as d14order_num
             from sale
             where datediff('2021-3-21',create_time)<=14
             group by user_id
    ) t2 on t1.user_id = t2.user_id
left join (
        select  user_id ,sum(product_num) as d7order_num
              from sale
              where datediff('2021-3-21',create_time)<= 7
              group by user_id
    ) t3 on t1.user_id = t3.user_id;
```
