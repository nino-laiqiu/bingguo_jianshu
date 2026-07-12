##1. 路径分析定义
[神策用户分析模型——路径分析的使用方法](https://www.sensorsdata.cn/blog/20180511/)

漏斗分析是固化了具体的分析过程或者业务环节，然后分析几个大的业务环节的转化；而路径分析，是固化了用户的路径顺序，在每个路径次序中，都包含了各个主要业务环节，因此在每一步中，出现的业务环节很有可能都是类似的。漏斗分析看重的是业务环节之间的留存关系，而路径分析看重的是用户在不同业务环节中的顺序及流失关系。

[路径分析：如何将用户的网站行为轨迹可视化呈现？](https://mp.weixin.qq.com/s/HZvDVo6ytUtDcOHvaZMSsg)
[BI分析系统——路径分析及产品化 )](https://mp.weixin.qq.com/s?__biz=MzI0OTEzNzYyOA==&mid=2650415336&idx=1&sn=c5986c55ecf8057c34c0aabfa5d268dc&chksm=f198905bc6ef194dd144303361ed5a6e2c1bbe4522315e7b5c5d75359e19b3fe79b100600b57&scene=21#wechat_redirect)

##2. 关键路径分析
1. sequenceMatch函数检查是否有事件链满足输入的模式
2. sequenceCount函数则统计满足输入模式的事件链的数量(**如果两个事件发生在同一秒时，是无法准确区分事件的发生先后关系的，所以会存在一定的误差**)
```
sequenceCount(pattern)(timestamp, cond1, cond2, ...)
```
pattern支持3中匹配模式：

(?N)：表示时间序列中的第N个事件，从1开始，最长支持32个条件输入；如，(?1)对应的是cond1

(?t op secs)：插入两个事件之间，表示它们发生时需要满足的时间条件（单位为秒）,支持 >=, >, <, <= 。例如上述SQL中，(?1)(?t<=15)(?2)即表示事件1和2发生的时间间隔在15秒以内，期间可能会发生若干次非指定事件。

.*：表示任意的非指定事件。

```
create table  nonodb.adm_path_tf_demo1
(
    eventTime DateTime,
    uid UInt32,
    event_type String,
    expflag int
)
ENGINE = MergeTree
ORDER BY (eventTime,uid)
SETTINGS index_granularity = 8192;

-- 在10分钟内完成浏览收藏操作.之后下单的人
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:15:00',1,'浏览',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:16:00',1,'浏览',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:16:00',1,'浏览',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:25:00',1,'收藏',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:26:00',1,'收藏',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:26:00',1,'收藏',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:36:00',1,'点击',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:56:00',1,'下单',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:57:00',1,'下单',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:57:00',1,'浏览',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:58:00',1,'下单',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:15:00',2,'浏览',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:25:00',2,'收藏',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:56:00',2,'下单',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:15:00',3,'浏览',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 11:25:00',3,'收藏',1);

insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 12:15:00',1,'浏览',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 12:24:00',1,'收藏',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 12:56:00',1,'下单',1);
insert into nonodb.adm_path_tf_demo1 values ('2022-01-02 12:56:00',2,'下单',1);
```
```
select uid,
      sequenceMatch('(?1)(?t<=600)(?2).*(?3)')(
          eventTime,
          event_type = '浏览',
          event_type = '收藏' and expflag =1 ,
          event_type = '下单'
          ) as is_match
       from nonodb.adm_path_tf_demo1
       where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02'
       and uid in (1,2,3)
       group by uid;

```
??看结果链是不允许重叠的,确实要是允许重叠实现起来是真的非常麻烦啊,业务上有需求的话条件限制喽
```
select uid,
      sequenceCount('(?1)(?t<=600)(?2).*(?3)')(
          eventTime,
          event_type = '浏览',
          event_type = '收藏' and expflag =1 ,
          event_type = '下单'
          ) as is_se
       from nonodb.adm_path_tf_demo1
       where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02'
       and uid in (1,2,3)
       group by uid;
```


##3. 智能路径检测
智能路径分析模型比较复杂，但同时支持的分析需求也会更加复杂，如分析给定期望的路径终点、途经点和最大事件时间间隔，统计出每条路径的用户数，并按照用户数对路径进行倒序排列

???网上给的例子怎么都是一样的,算了吸收一下吧,虽说数组函数非常灵活,但是不怕查询慢吗??具体还得结合业务过滤大部分的数据

#####方案一

**1**
```
-- 用groupArray函数整理成<时间, <事件名, expflag>>的元组
select uid ,groupArray((toUInt32(eventTime),(event_type,expflag)))
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
**2**
```
-- arrayFilter 只保留数组中满足条件的数据
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag))))
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
**3**
```
-- arraySort 对数组中的数据按照指定列进行升序排列
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag)))))
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
**4**
```
--  arrayCompact对事件按照时间维度排序后进行相邻去重
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,arrayCompact(arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag)))))) AS sorted_events
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
**5**
```
-- arrayEnumerate获取下标数组
-- 切割点:过滤出原始行为链中的分界点下标。分界点的条件是路径终点或者时间差大于最大间隔
-- arrayFilter如下返回第一个参数的值,满足x,y,z的值

with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,
       arrayCompact(arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag)))))) AS sorted_events,
       arrayEnumerate(sorted_events) AS event_idxs,
       arrayFilter( (x, y, z) -> z.1 <= end_event_max AND (z.2.1 = '下单' OR y > 600),event_idxs,arrayDifference(sorted_events.1),sorted_events ) AS gap_idxs
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
**6**
```
-- 利用arrayMap和has函数获取下标数组的掩码（由0和1组成的序列），用于最终切分，1表示分界点
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,
       arrayCompact(arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag)))))) AS sorted_events,
       arrayEnumerate(sorted_events) AS event_idxs,
       arrayFilter( (x, y, z) -> z.1 <= end_event_max AND (z.2.1 = '下单' OR y > 600),event_idxs,arrayDifference(sorted_events.1),sorted_events ) AS gap_idxs,
       arrayMap(x -> x + 1, gap_idxs) AS gap_idxs_,  --如果不加1的话上一个事件链的结尾事件会成为下个事件链的开始事件
       arrayMap(x -> if(has(gap_idxs_, x), 1, 0), event_idxs) AS gap_masks  --标记切割点
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
**7**
```
-- 调用arraySplit函数将原始行为链按分界点切分成单次访问的行为链。注意该函数会将分界点作为新链的起始点，所以前面要将分界点的下标加1
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,
       arrayCompact(arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag)))))) AS sorted_events,
       arrayEnumerate(sorted_events) AS event_idxs,
       arrayFilter( (x, y, z) -> z.1 <= end_event_max AND (z.2.1 = '下单' OR y > 600),event_idxs,arrayDifference(sorted_events.1),sorted_events ) AS gap_idxs,
       arrayMap(x -> x + 1, gap_idxs) AS gap_idxs_,
       arrayMap(x -> if(has(gap_idxs_, x), 1, 0), event_idxs) AS gap_masks,
       arraySplit((x, y) -> y, sorted_events, gap_masks) AS split_events  --把用户的访问数据切割成多个事件链
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
**8**
```
-- 调用arrayJoin和arrayCompact函数将事件链的数组打平成多行单列，并去除相邻重复项
select uid,
       arrayJoin(split_events) AS event_chain_,
       arrayCompact(event_chain_.2) AS event_chain  --相邻去重
       from (
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,
       arrayCompact(arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag)))))) AS sorted_events,
       arrayEnumerate(sorted_events) AS event_idxs,
       arrayFilter( (x, y, z) -> z.1 <= end_event_max AND (z.2.1 = '下单' OR y > 600),event_idxs,arrayDifference(sorted_events.1),sorted_events ) AS gap_idxs,
       arrayMap(x -> x + 1, gap_idxs) AS gap_idxs_,
       arrayMap(x -> if(has(gap_idxs_, x), 1, 0), event_idxs) AS gap_masks,
       arraySplit((x, y) -> y, sorted_events, gap_masks) AS split_events  --把用户的访问数据切割成多个事件链
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid ) WHERE event_chain[length(event_chain)].1 = '下单';  -- 事件链最后一个事件必须是目标事件;
```
**9**
```
-- 调用hasAll函数确定是否全部存在指定的途经点。如果要求有任意一个途经点存在即可，就换用hasAny函数。当然，也可以修改WHERE谓词来排除指定的途经点
-- select  hasAll([('浏览',1),('收藏',1),('下单',1)], [('下单',1),('收藏',1)]) ??这能判断,为什么下面的不行???
select uid,
       arrayJoin(split_events) AS event_chain_,
       arrayCompact(event_chain_.2) AS event_chain,
       hasAll(event_chain, [('浏览',1)]) AS has_midway_hit -- 调用hasAll函数确定是否全部存在指定的途经点
       from (
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,
       arrayCompact(arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,expflag)))))) AS sorted_events,
       arrayEnumerate(sorted_events) AS event_idxs,
       arrayFilter( (x, y, z) -> z.1 <= end_event_max AND (z.2.1 = '下单' OR y > 600),event_idxs,arrayDifference(sorted_events.1),sorted_events ) AS gap_idxs,
       arrayMap(x -> x + 1, gap_idxs) AS gap_idxs_,
       arrayMap(x -> if(has(gap_idxs_, x), 1, 0), event_idxs) AS gap_masks,
       arraySplit((x, y) -> y, sorted_events, gap_masks) AS split_events
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid ) WHERE event_chain[length(event_chain)].1 = '下单' AND has_midway_hit = 1;  -- 必须包含途经点;
```
**10**
```
-- 将最终结果整理成可读的字符串
 SELECT
  result_chain,
  uniqCombined(uid) AS user_count
FROM (
select uid,
       arrayJoin(split_events) AS event_chain_,
       arrayCompact(event_chain_.2) AS event_chain,
       hasAll(event_chain, [('浏览','1')]) AS has_midway_hit,
       arrayStringConcat(arrayMap( x -> concat(x.1, '#', toString(x.2)),event_chain ), ' -> ') AS result_chain
       from (
with  (select toUInt32(maxIf(eventTime, event_type = '下单')) AS end_event_max from nonodb.adm_path_tf_demo1) as end_event_max
select uid ,
       arrayCompact(arraySort(x -> (x.1),arrayFilter( x -> x.1 <= end_event_max,groupArray((toUInt32(eventTime),(event_type,toString(expflag))))))) AS sorted_events,
       arrayEnumerate(sorted_events) AS event_idxs,
       arrayFilter( (x, y, z) -> z.1 <= end_event_max AND (z.2.1 = '下单' OR y > 600),event_idxs,arrayDifference(sorted_events.1),sorted_events ) AS gap_idxs,
       arrayMap(x -> x + 1, gap_idxs) AS gap_idxs_,
       arrayMap(x -> if(has(gap_idxs_, x), 1, 0), event_idxs) AS gap_masks,
       arraySplit((x, y) -> y, sorted_events, gap_masks) AS split_events
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid ) WHERE event_chain[length(event_chain)].1 = '下单'
                AND has_midway_hit = 1
     ) group by result_chain order by user_count desc limit  10 ;
```

***bug的解决,hasAll这块判断***
```
 select  hasAll([('浏览',1),('收藏',1),('下单',1)], [('下单',1),('收藏',1)]
```
虽然上面的是可以的,但是例子中SQL太长了,好像没有转换的问题,导致无法匹配上,最好都转成string类型的....
#####方案二
不设置途经点，且仅以用户最后一次到达目标事件作为参考
```
-- 这个有个问题3号没有下单的记录...过滤一下
select uid,
       arrayMap((x, y) -> (x, y),groupArray(event_type),groupArray(eventTime)),
       arrayWithConstant(length(groupArray(eventTime)),maxIf(eventTime, event_type = '下单'))
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
```
-- 找到目标节点前1小时内的所有事件
select uid,
       arrayFilter((x,y) -> y - x.2 > 3600,arrayMap((x, y) -> (x, y),groupArray(event_type),groupArray(eventTime)),
       arrayWithConstant(length(groupArray(eventTime)),maxIf(eventTime, event_type = '下单')))
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
```
-- 进行排序和映射
select uid,
       arrayMap(b -> b.1,arraySort(y -> y.2,arrayFilter((x,y) -> y - x.2 > 3600,arrayMap((x, y) -> (x, y),groupArray(event_type),groupArray(eventTime)),
       arrayWithConstant(length(groupArray(eventTime)),maxIf(eventTime, event_type = '下单')))))
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
```
-- 相邻事件去重
select uid,
       arrayCompact(arrayMap(b -> b.1,arraySort(y -> y.2,arrayFilter((x,y) -> y - x.2 > 3600,arrayMap((x, y) -> (x, y),groupArray(event_type),groupArray(eventTime)),
       arrayWithConstant(length(groupArray(eventTime)),maxIf(eventTime, event_type = '下单'))))))
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid;
```
```
-- 拼接路径 arrayStringConcat
select result,uniqCombined(uid) AS user_count
       from (
select uid,
       arrayStringConcat(arrayCompact(arrayMap(b -> b.1,arraySort(y -> y.2,arrayFilter((x,y) -> y - x.2 > 3600,arrayMap((x, y) -> (x, y),groupArray(event_type),groupArray(eventTime)),
       arrayWithConstant(length(groupArray(eventTime)),maxIf(eventTime, event_type = '下单')))))),'->') result
from nonodb.adm_path_tf_demo1
where toDate(eventTime) >= '2022-01-02'  and toDate(eventTime) <= '2022-01-02' and uid in (1,2,3)
group by uid )where result <> ''  group by result;
```


#####上面用到的几个高阶函数
**1. arrayCompact对数组中的数据进行相邻去重，用户重复操作的事件只记录一次(页面去重)**
```
SELECT arrayCompact([1, 2, 3, 3, 1, 1, 1, 4]) AS data

[1,2,3,1,4]
```
**2. arraySort  对数组中的数据按照指定列进行升序排列；降序排列参考arrayReverseSort**
```
 SELECT arraySort(x -> (x.1), [(1, 'a'), (4, 'd'), (2, 'b'), (3, 'c')]) AS data

[(1,'a'),(2,'b'),(3,'c'),(4,'d')]
```
**3. arrayEnumerate  取数组的下标掩码序列**
```
SELECT arrayEnumerate([1, 2, 3, 3, 1, 1, 4, 2]) AS data

[1,2,3,4,5,6,7,8]
```
**4. arrayMap 对数组中的每一列进行处理，并返回长度相同的新数组**
```
SELECT arrayMap(x -> concat(toString(x.1), ':', x.2), [(1, 'a'), (4, 'a'), (3, 'a'), (2, 'c')]) AS data

['1:a','4:a','3:a','2:c']
```
**5. arrayStringConcat将数组元素按照给定分隔符进行拼接，返回拼接后的字符串**
```
SELECT arrayStringConcat( ['a','b','c'] , '-');

a-b-c
```
**6. arraySplit 按照规则对数组进行分割(遇到下标为1时进行分割，分割点为下一个 数组的起始点；注意，首项为1还是0不影响结果)**
```
SELECT arraySplit((x, y) -> y, ['a', 'b', 'c', 'd', 'e'], [1, 1, 1, 0, 0]) AS data

[['a'],['b'],['c','d','e']]
```
**7. arrayDifference参数必须是数值类型；计算数组中相邻数字的差值，第一个值为0**
```
SELECT arrayDifference([3, 1, 1, 4, 2]) AS data

[0,-2,0,3,-2]
```
**8. arrayFilter 只保留数组中满足条件的数据**
```
SELECT arrayFilter((x,y) -> (x > 2 and y >3), [12, 3, 4, 1, 0],[12, 3, 4, 1, 0]) AS data

[12,4]

SELECT arrayFilter(x -> (x > 2), [12, 3, 4, 1, 0]) AS data

[12,3,4]
```

##4. VIVO方案

[VIVO路径分析模型](https://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247490504&idx=1&sn=9827b136fa5cfc81467cb1b795f7bc41&chksm=ebd86b5adcafe24c450237a7ef2ac09c0efb5c8212cb6755e1703d46329dc321f17d32d9617d&scene=178&cur_album_id=1500549131247910916#rd)


**通常用户在需要进行路径分析的场景时关注的主要问题：**
1. 按转换率从高至低排列在APP内用户的主要路径是什么？
2. 用户在离开预想的路径后，实际走向是什么？
3. 不同特征的用户行为路径有什么差异？

#####1. 一些概念
1. Session和Session Time
不同于WEB应用中的Session，在数据分析中的Session会话，是指在指定的时间段内在网站上发生的一系列互动。本模型中的Session Time的含义是，当两个行为间隔时间超过Session Time，我们便认为这两个行为不属于同一条路径。
2. 桑基图
![桑基图](https://upload-images.jianshu.io/upload_images/9049859-b42dadce4b1268dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####2. 技术关键点
1. Session切分
其实这里我们可以从前端那拿到会话id,这样就不用切分了
2. 相邻页面去重
3. 滑动数组
4. 转化率计算
页面转化率,路径转化率
5. 邻接表和剪枝




##5. Clickhouse问题解决
#####1. dataGrip连接clickhouse时，时间字段显示差八小时问题
![改一下的](https://upload-images.jianshu.io/upload_images/9049859-c64e38d151cea829.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2. 远程连接clickhouse
[ClickHouse安装和使用 ](https://cloud.tencent.com/developer/article/1910211)

注意关闭防火墙
```
systemctl status firewalld 

systemctl stop firewalld.service

systemctl disable firewalld.service
```
#####3. 遇到的问题
[clickhouse单节点报错 Code: 210\. DB::NetException: Connection refused (localhost:9000)](https://blog.csdn.net/qq_35515661/article/details/114896300)
```
<-- <listen_host>0.0.0.0</listen_host> -->
```
这个注释千万不能去掉....暂时不知道怎么处理..卸载重装吧

[clickhouse卸载重装](https://blog.csdn.net/zjx_z/article/details/95449050)
