参考：[基于StarRocks的用户画像解决方案](https://www.bilibili.com/video/BV13T4y1o7K8?spm_id_from=333.999.0.0)

##0. 演进背景
1. 用户运营1.0时代
运营重心偏重于商业诉求，对用户关注相对较少。
2. 精细运营2.0时代
精细化运营是流量价值最大化，从用户角度来说就是按需投放。
3. 单对单营运3.0时代
单对单运营最突出特点表现在实时性，之前的运营模式往往是通过对用户历史数据的分析来进行一系列运营操作，单对单运营能做到千人千面。

##1. 数据应用体系层级
![数据应用体系层级](https://upload-images.jianshu.io/upload_images/9049859-b2aef9922bf646b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 基础平台搭建
基础设施建设，ETL工程，DW建设，统一SDK，lakeHouse
2. OLAP报表
广告主报表，OLAP多维报表，BI报表，实时大屏
3. 产品运营与分析
Adhoc交互分析，自助BI，订单可视化分析
4. 精细化运营
数据挖掘，个性化推荐...用户行为分析，用户画像
5. 战略决策
通过下面4层提供的一些数据应用产物，组合而成的数据产品和服务，从各个运营视角和分析维度为组织的战略目标服务，方便决策者用数据驱动业务，精准做出正确决断。

##2. 用户画像-业务特点
#####业务应用
精准营销：广告投放,个性推荐,弹窗推送
群体分析
风险预警
效果分析
渠道分析
#####业务难点
数据量庞大，检索方式灵活
组合标签计算，开发复杂度高
精准去重计算，资源消耗巨大
聚合标签集合，查询并发度高

##3. 标签类型
用户画像建模最重要的是对用户「打标签」，常见的3种分类：
#####统计类标签
基础标签类型，用户的性别、年龄、城市、星座、职业等等基础属性，可以做分布统计，
也包括如活跃时长、注册用户数、访问次数、消费金额等按照某些基础维度统计出的指标
#####规则类标签
基于确定的规则及用户行为产生。
规则，通常是需要对基础维度添加前置修饰词来限定统计的口径；或者需要提前做数据调研，如通过RFM客户价值法来科学的制定规则，从而打标签。
例如：时间是基础维度，天是最小粒度，而最近30天交易次数 ≥ 2就是个有修饰词限定的规则类标签
#####机器学习挖掘类标签
该类标签通过机器学习挖掘产生，用于对用户的某些属性或某些行为进行预测判断
例如：基于客户的行为推断客户行为上的性别
互联网公司花样百出的标签


##4. 用户标签体系
![用户标签体系](https://upload-images.jianshu.io/upload_images/9049859-cd5c4905778b867e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##5. 用户属性维度
![用户属性维度](https://upload-images.jianshu.io/upload_images/9049859-48287b9c31482f2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##6. 风险控制维度
![风险控制维度](https://upload-images.jianshu.io/upload_images/9049859-97fdfcedf37b71b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##7. 社交属性维度
![社交属性维度](https://upload-images.jianshu.io/upload_images/9049859-c248e83614e70241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##8. 画像标签开发
![标签开发](https://upload-images.jianshu.io/upload_images/9049859-c438cef6b17c77a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##9. 组合标签计算-客群圈选场景
![组合标签计算-客群圈选场景](https://upload-images.jianshu.io/upload_images/9049859-5fa639c2ead67538.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##10. 组合标签计算-传统方案
![传统方案](https://upload-images.jianshu.io/upload_images/9049859-5b0db0a6f3a5102d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####传统方案痛点
1. 应用角度：
筛选客群得分别在多个索引搜索后，再做聚合，比较麻烦
2. 技术角度：
架构较重，维护复杂
Sql能力差(join和聚合等)，开发成本大，
定制开发，扩展不灵活

#####基于ES+Hbase组合标签方案
![基于ES+Hbase组合标签方案](https://upload-images.jianshu.io/upload_images/9049859-e3e8a53c4a1811b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![基于ES+Hbase组合标签方案](https://upload-images.jianshu.io/upload_images/9049859-8a02705e65e0b516.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##11. 客群圈选
#####bitmap_union + 物化视图
例子：在明细表中找出有过click购物车和view收藏页行为的user列表
```
create table t1(
event_day date,
event_time datetime,
uid int,
action string,
page string,
product_code string,
from_days int)
DUPLICATE KEY(`event_day`,`event_time`, `uid`)
PARTITION BY RANGE(event_day)
(
PARTITION p20210401 VALUES LESS THAN ('2021-04-01'),
PARTITION p20210501 VALUES LESS THAN ('2021-05-01')
)
DISTRIBUTED BY HASH(`uid`) BUCKETS 3;
```

```
mysql> select * from t1;
+-------------+----------------------------+--------------+--------+-----------+---------------+--------------+
| event_day | event_time | uid | action | page | product_code | from_days |
+-------------+----------------------------+--------------+--------+-----------+---------------+--------------+
| 2021-04-03 | 2021-04-03 10:01:30 | 274649163 | click | 购物车 | MDS | 1 |
| 2021-04-03 | 2021-04-03 10:04:30 | 274649163 | view | 收藏页 | MDS | 4 |
| 2021-04-03 | 2021-04-03 10:03:30 | 274649164 | click | 购物车 | MDS | 2 |
| 2021-04-03 | 2021-04-03 10:06:30 | 274649165 | click | 购物车 | MMS | 8 |
| 2021-04-03 | 2021-04-03 10:08:30 | 274649164 | view | 收藏页 | MDS | 3 |
| 2021-04-03 | 2021-04-03 10:09:30 | 274649165 | view | 购物车 | MDS | 10 |
+------------+---------------------------+---------------+-------+------------+--------------+---------------+

```

物化视图：
```
CREATE MATERIALIZED VIEW user_profile_view AS
SELECT event_day ,action, page , bitmap_union(to_bitmap(uid)) b_uid
FROM t1
GROUP BY event_day , action, page;

```

圈选：
```
WITH tbl_c AS (
SELECT bitmap_union(to_bitmap(uid)) b_uid
FROM t1
WHERE event_day = '2021-04-03'
AND action='click' and page= '购物车'
GROUP BY action, page
) , tbl_v AS(
SELECT bitmap_union(to_bitmap(uid)) b_uid
FROM t1
WHERE event_day = '2021-04-03'
AND action='view' and page='收藏页'
GROUP BY action, page
) ,tbl_u AS(
SELECT b_uid from tbl_c
UNION ALL
SELECT b_uid from tbl_v
) 
SELECT bitmap_count(bitmap_intersect(b_uid)) uid_ct,  bitmap_to_string(bitmap_intersect(b_uid)) uid_list
```
#####bitmap to array、unnest
#####bitmap_and(not)

bitmap函数参考;
[bitmap_intersect ](https://docs.starrocks.com/zh-cn/main/sql-reference/sql-functions/bitmap-functions/bitmap_intersect)
[BITMAP函数](https://cloud.baidu.com/doc/DORIS/s/3kmealsht#bitmap_intersect)

宽表合并方案？
