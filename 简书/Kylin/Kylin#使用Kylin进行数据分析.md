##1. Kylin 进行留存分析
先取出触发事件类型的所有用户(去重),在这个步骤之后有另一个步骤我们需要评估这俩个阶段之间用户流失原因,按照这个逻辑来去重取出下一个步骤的用户,我们发现这样的spark写法要先子查询然后在join,效果不好,如下使用spark-sql来计算留存率
```
SELECT count(distinct first_day.USER_ID) FROM
 (select distinct USER_ID as USER_ID from access_log where DT = '20190101') as first_day 
   INNER JOIN
 (select distinct USER_ID as USER_ID from access_log where DT = '20190102') as second_day 
 ON first_day.USER_ID = second_day.USER_ID
```
比较用户俩天触发同一事件的用户数,用于分析该事件
```
intersect_count(columnToCount, columnToFilter, filterValueList)
```
`columnToCount` 要做计数的列，也就是bitmap存储的列，这里就是“user_id“
`columnToFilter` 做交集计算的列，如果是把多个不同日期的bitmap来做交集，那么这列就是日期
`filterValueList` 做交集计算的bitmap的key值，需要是一个数组
**实例**
```
select intersect_count(user_id, dt, array['20190101', '20190102'])
from access_log
where dt in ('20190101', '20190102')
```
**多天的比较**
```
select city, version,
intersect_count(user_id, dt, array['20161014']) as first_day_uv,
intersect_count(user_id, dt, array['20161015']) as second_day_uv,
intersect_count(user_id, dt, array['20161016']) as third_day_uv,
intersect_count(user_id, dt, array['20161014', '20161015']) as retention_oneday,
intersect_count(user_id, dt, array['20161014', '20161016']) as retention_twoday
from access_log
where dt in ('2016104', '20161015', '20161016')
group by city, version
order by city, version
```

##2.Kylin 进行漏斗分析
```
select 
intersect_count(user_id, page, array['index.html']) as first_step_uv,
intersect_count(user_id, page, array['search.html']) as second_step_uv,
intersect_count(user_id, page, array['detail.html']) as third_step_uv,
intersect_count(user_id, page, array['index.html', 'search.html']) as retention_one_two,
intersect_count(user_id, page, array['search.html','detail.html']) as retention_two_three
from access_log
where dt in ('2016104', '20161015', '20161016')
```
##3.Kylin 多维度滑动的留存分析
虽然 intersect_count 交集函数只接受一个维度值的变化，但我们可以巧妙利用 where 做其它维度的筛选，最后的结果交给 SQL 执行器来计算
```
select 
intersect_count(user_id, dt, array[‘20190101’])，#第一天的UV
intersect_count(user_id, dt, array[‘20190101’, ‘20190102’]) #第一天和第二天交集
from access_log
where (dt='20190101’ and page=‘detail.html’) #筛选第一天&访问明细页的用户
or 
(dt=‘20190102’ and page=‘payment.html’) #筛选第二天&访问付款页的用户
```
使用说明,array中的数据是必须满足的
#####array中先或再与操作表达分析
```
select 
intersect_count(user_id, tag_value, array['男', '90后', '10-20万']) 
from user_profile
where (tag_type='性别' and tag_value='男') or (tag_type='年龄' and tag_value='90后') or (tag_type='收入' and tag_value='10-20万')
```
##4.Kylin 用户明细分析
intersect_value函数的使用
```
select 
intersect_value(user_id, tag_value, array['男', '90后', '10-20万']) 
from user_profile
where (tag_type='性别' and tag_value='男') or (tag_type='年龄' and tag_value='90后') or (tag_type='收入' and tag_value='10-20万')
```
##5.去重
收集了一些有关去重的博客
https://blog.bcmeng.com/post/kylin-distinct-count-global-dict.html
https://hexiaoqiao.github.io/blog/2017/01/18/exact-count-optimization-of-apache-kylin/
https://hexiaoqiao.github.io/blog/2016/11/27/exact-count-and-global-dictionary-of-apache-kylin/
https://blog.csdn.net/weixin_39074599/article/details/89971320
https://blog.csdn.net/weixin_39074599/article/details/89971868
##6.Top-N近似预计算
Top－N的优点：因为它只保留Top的记录，会让Cube空间大幅度减少，而查询性能大大提升。在一个典型的例子里，改用Top-N后，Cube的大小减少了90%，而查询时间则只有以前的10%不到。
缺点是它可能是近似的结果（当50倍空间也无法容纳所有基数的时候）。如果业务场景需要绝对精确的话，它可能不适合。
Top－N误差率由很多因素决定的：
1）数据的分布：数据分布越陡，误差越小。
2）算法使用的空间：如果对精度要求高的话，可以选择用更多的空间换取更精准的准确率 。在实际使用中，可以做一些比较以了解误差情况。

**space saving 算法**(可以学习一下)
https://blog.csdn.net/nazeniwaresakini/article/details/109113529
http://sites.nlsde.buaa.edu.cn/~yxtong/bd2016/07_Streaming_1027.pdf
https://zhengzangw.com/notes/advanced-algorithm/
https://www.cs.dartmouth.edu/~ac/Teach/CS49-Fall11/Notes/lecnotes.pdf

##7.百分位数预计算



>内容来源于https://blog.csdn.net/weixin_39074599

