这篇博客用来回顾和梳理一下数据仓库中的常用主题建模架构,面向分析的架构以及集成主题报表,我已经把这些报表上传至GitHub上了,有兴趣的可以看一下
地址:https://github.com/nino-laiqiu/TiTan
我在我部署在GitHub上的hexo博客中记录了一份比较完整的开发流程,有兴趣的可以看一下


##241.ID-Mapping
id-mapping是用来处理多个数据来源用户一致性问题的,可以使用图计算中的连通子图来处理,但是会存在这样的问题,把本来不属于用一个人的属性连接在一起或者把一个人的划分为两个人的维度属性了,可以采用条件过滤来处理,但是这样会造成数据丢失甚至是无法解决问题
图计算不属于SQL练习这个部分,不再写了,我要解决的是匿名用户登录问题,匿名的行为日志消息应该属于哪个用户(账号)的
解决的方案:采用评分的思路,但这种解决方法仍是有问题的,比如需要多次join计算量大,比如说怎么处理一个人多账号的问题
#####生成一张评分表
我们设置这张评分表有如下字段
**设备 账号 分数 最后一次登录时间**
按照设备字段和账号字段group by 每一次会话分数就+100,取这个维度的最后登录时间,于是这就是我们的初始评分表,这张表可能会出现没有账号的行,实际中并没有什么意义,但是还是保留为好
我们取设备 账号维度下以设备为分组,取这个分组下分数最高的行与今天的log数据生成今天的guid
```
      spark.sql(
        """
          |insert into table dw.score partition(dy="2021-02-23")
          |select
          |deviceid,account,cast(count(distinct sessionid ) * 100 as double) as score ,max(timestamp) as  timestamp
          |from db_demo1.app_event_log
          |group by deviceid,account
          |""".stripMargin).show()
```
#####生成GUID
取昨天评分表,每个设备下的最高评分,今天的log数据与其left join(**条件是设备**) 生成guid
**我们的规则是,如果今天的log数据有账号,那么账号就作为guid,如果今天没有账号,但是昨天的数据是有账号的,那么今天的guid就是昨天这个设备下最高评分的账号了,如果都没有,那只能以今天的设备号,作为今天的guid了**(这便是匿名的解决方法)
```
spark.sql(
        """
          |
          |INSERT INTO TABLE dw.log PARTITION(dy='2021-02-23')
          |
          |select
          | regioned.account          ,
          | regioned.appid            ,
          | regioned.appversion       ,
          | regioned.carrier          ,
          | regioned.deviceid         ,
          | regioned.devicetype       ,
          | regioned.eventid          ,
          | regioned.ip               ,
          | regioned.latitude         ,
          | regioned.longitude        ,
          | regioned.nettype          ,
          | regioned.osname           ,
          | regioned.osversion        ,
          | regioned.properties       ,
          | regioned.releasechannel   ,
          | regioned.resolution       ,
          | regioned.sessionid        ,
          | regioned.timestamp        ,
          | regioned.newsession       ,
          | regioned.province   	    ,
          | regioned.city             ,
          | regioned.district         ,
          | regioned.isnew            ,
          | case
          |   when regioned.account is not null then regioned.account
          |   when idbind.account is not null then idbind.account
          |   else regioned.deviceid
          | end as guid
          |
          |from regioned left join bind on idbind.deviceid=regioned.deviceid
          |
          |""".stripMargin)
```
#####生成今天的评分表
步骤1:同步骤生成一张评分表一样,按照设备和账号维度聚合,同样会出现没有账号有设备的行
步骤2.我们把这部分提取出来,因为这些设备中可能有新设备
步骤3.我们把这些数据与昨天的评分表left join 过滤出为右边为null的数据,那么这个设备就是新设备了,我们可以给评分但是并没有什么用
步骤4.**今天的评分表和昨天的评分表full join(条件是设备和账号),我们评分的标准是:如果昨天登录了,今天也登录了那么评分加起来,如果昨天登录了,但是今天没有那么评分衰减,如果昨天没有数据,但是今天有数据,那么就保留其评分与步骤三的数据union all ,就是今天的评分表了,然后不断循环**
```
spark.sql(
        """
          |INSERT INTO TABLE dw.score PARTITION(dy='2021-02-23')
          |
          |select
          |a.deviceid,
          |a.account,
          |a.timestamp,
          |a.score
          |from
          |cur_may_new a
          |left join pre_score b
          |on a.deviceid = b.deviceid
          |where b.deviceid is  null
          |
          |union all
          |
          |select
          |nvl(pre.deviceid,cur.deviceid) as deviceid, -- 设备
          |nvl(pre.account,cur.account) as account, -- 账号
          |nvl(cur.timestamp,pre.timestamp) as timestamp, -- 最近一次访问时间戳
          |case
          |when pre.deviceid is not null and cur.deviceid is not null then pre.score+cur.score
          |when pre.deviceid is not null and cur.deviceid is null then pre.score*0.5
          |when pre.deviceid is  null and cur.deviceid is not  null then cur.score
          |end as score
          |from
          |pre_score pre
          |full join
          |today_score cur
          |on pre.deviceid = cur.deviceid and  pre.account = cur.account
          |""".stripMargin).show()
```
##242.布隆过滤器
布隆过滤器是bitmap的变形式,在大数据领域布隆过滤器能用在各种地方,比如hbase中的LSM树中就有体现,布隆过滤器的原理不在说明,值得注意**布隆过滤器的误差率**的计算公式,以及与去重算法**Hyperloglog**的对比
https://juejin.cn/post/6844903785744056333
下面举一些例子来体现一下布隆过滤器的应用,例如两张大表join怎么优化,如果由于布隆过滤器会覆盖相同组合hash映射的值,所以我们不追求误差率的情况下是可以使用它的,又比如说怎么大量数据去重,又比如说两份大文件,怎么查找出相同的数据(对于重复的只要展示一个即可,不需要展示数量特征)
#####新老用户标识
怎么标识今天的用户是否是老用户还是新用户,在数据量较小的情况在spark的开发环境中,比如我们使用spark的mapjoin(广播变量),还可以使用full join (left join)来实现,没有join上的就是新用户了,在数据量大的情况下我们使用布隆过滤器来优化一下,下面是布隆过滤器的示例
```
 //使用hadoop的布隆过滤器
  def hadoopBloom() {
    val filter = new BloomFilter(1000000, 5, Hash.MURMUR_HASH)
    filter.add(new Key("a".getBytes()))
    filter.add(new Key("b".getBytes()))
    filter.add(new Key("c".getBytes()))
    filter.add(new Key("d".getBytes()))

    val bool = filter.membershipTest(new Key("a".getBytes))
    val bool1 = filter.membershipTest(new Key("dwd".getBytes))
    println(bool)
    println(bool1)
  }
  //使用spark的布隆过滤器
  def sparkBloom(): Unit = {
    val spark = SparkSession.builder().config(new SparkConf().setMaster("local[*]").setAppName(this.getClass.getSimpleName)).getOrCreate()
    import spark.implicits._
    val filter = spark.sparkContext.makeRDD(List("A", "B", "C", "D", "E", "F")).toDF("alphabet").stat.bloomFilter("alphabet", 100000, 0.001)
    val bool = filter.mightContain("A")
    val bool1 = filter.mightContain("R")
    println(bool)
    println(bool1)
  }
```
对于新老用户问题,我们可以把昨天的用户映射到BloomFilter中,并把BloomFilter 给广播出去,使用mapPartitions一个组一个组来比较,如果在布隆过滤器中,那就是老用户,标识newuser = 0 ,否则newuser = 1
##243.拉链表
拉链表就是事实表中的累计快照表,我们使用拉链表的思想能解决时间性质的问题,示例:用户连续登陆问题,来说明一下拉链表的用法与SQL处理,拉链表的处理难点是分类的逻辑比较复杂
#####初始
拉链表的字段如下:
首次登录时间(first_dt) guid 区间开始时间(range_start) 区间结束时间(range_end)
first_dt,guid,range_start,range_end
按照guid聚合,获取今天登陆的用户信息
guid  cur_time
#####分类
昨天的拉链表和今天的登录用户信息 full join (字段guid),join上的是昨天登录了,今天登录了
以及已经封闭区间的了,但是今天确实登录了,这一类要单独处理
没有join上的分为以下几类
1.今天的新用户(guid)
2.昨天登录了,但是今天没有登录,range_end为9999-12-31
3.已经是封闭区间的行,今天也没有登录
#####处理
我们对这几种情况分别处理
没有join上的
**1.昨天登录了pre.range_end = '9999-12-31' 但是今天没有登录 cur.cur_time is null 
first_dt,guid,range_start保持不变,range_end改成昨天的
2.新用户的情形,昨天没有登录 pre.range_end is null ,今天登陆了
first_dt,guid,range_start,range_end分别改为今天,用户,今天,9999-12-31
3.封闭区间,今天没有登录,保持原样不变**
join上的
**1.昨天登录了,今天登录了即join上的,保持原样不变
2.封闭区间(之前登过,但是至少昨天没有登,但是今天登陆了),保持原样行属性不变,对于新增今天的登录这一点我们要单独处理,处理如下
从昨天的数据中获取guid的range_end 不是9999-12-31的guid和首次登录时间
与今天的数据join(字段为 guid) 如果能join上那就是之前登录过的封闭区间,但是至少昨天没有登录,今天登陆了
first_dt,guid,range_start,range_end,分别是之前的初始登录信息,guid,今天,9999-12-31**

**注意一下left semi join 和 join 的区别**
```
   spark.sql(
     """
       |select
       |  nvl(pre.first_dt, cur.cur_time) as first_dt,
       |  -- 如果之前存在那就取之前,之前不存在那就是新用户了
       |  nvl(pre.guid, cur.guid) as guid,
       |  nvl(pre.range_start, cur.cur_time) as range_start,
       |  case when pre.range_end = '9999-12-31'
       |  and cur.cur_time is null then pre.pre_time --  昨天登录,今天没有登录
       |  when pre.range_end is null then cur.cur_time -- 新用户(join不上的)
       |  else pre.range_end -- 封闭区间保持原样
       |  end as range_end
       |from
       |  (
       |    select
       |      first_dt,
       |      guid,
       |      range_start,
       |      range_end,
       |      dy as pre_time
       |    from
       |      dw.user_act_range
       |    where
       |      dy = '2020-12-10'
       |  ) pre full
       |  join (
       |    select
       |      guid,
       |      max(dy) as cur_time
       |    from
       |      dw.traffic_aggr_session
       |    where
       |      dy = '2020-12-11'
       |    group by
       |      guid
       |  ) cur on pre.guid = cur.guid
       |union all
       |  -- 从会话层获取今日登陆的用户
       |  -- range_end封闭且今日登陆的情况
       |select
       |  first_dt as first_dt,
       |  o1.guid as guid,
       |  '2020-12-11' as range_start,
       |  '9999-12-31' as range_end
       |from
       |  (
       |    select
       |      guid,
       |      first_dt
       |    from
       |      dw.user_act_range
       |    where
       |      dy = '2020-12-10'
       |    group by
       |      guid,
       |      first_dt
       |    having
       |      max(range_end) != '9999-12-31'
       |  ) o1 -- 从会话层取出今天登陆的所有用户
       |  left semi
       |  join (
       |    select
       |      guid
       |    from
       |      dw.traffic_aggr_session
       |    where
       |      dy = '2020-12-11'
       |    group by
       |      guid
       |  ) o2 on o1.guid = o2.guid
       |""".stripMargin).show(100,false)
```
##244.bitmap思想
如上面的那种处理会产生大量的冗余数据,我们可以定时清除冗余数据,但是bitmap思想建模处理,可以避免这种数据冗余,bitmap表在用户登录问题上,最多只有(有多少账号/用户就有多少行),如果我们想要保留所有的历史数据,我们可以对bitmap表进行压缩,bitmap压缩中咆哮位图算法就是很好的压缩算法,这里我们采用把bit-01显示转化为long类型的数来压缩存储
bitmap生成表报的思路:使用**like函数,replace函数,split()函数以及正则表达式**即可解决用户登陆问题(示例),比如like获取用户连续登陆,replace把0替换为空格,获取登陆天数等,split切分获取访问间隔等

#####bitmap表
获取用户的登录信息,按照guid聚合,考虑存储sum来节省空间
```
select
guid, 
sum(pow(2,datediff('2020-11-18',dt)))
from 
db_demo1.dou
group by guid
```

#####更新方法一(substr)
两种更新方法的区别在于位运算无需为原来的bitmap中的sum数据进行翻转,直接对最后一位进行操作,二方法一是需要翻转的(当然不翻转也行),对第一位操作
补全为一个月的登录情况
```
reverse(lpad(cast(bin(bitmap) as string), 31, '0'))
```
与今天guid的登录情况full join (字段guid)
1.今天没有登录
2.今天登陆了
3.新的guid

**注意conv/bin/lpad/rpad等函数的应用**
```
with a as (
  select
    -- 会话视图层
    guid
  from
    dw.traffic_aggr_session
  where
    dy = '2020-12-17'
  group by
    guid
),
b as (
  select
    -- bitmap表
    guid,
    reverse(lpad(cast(bin(bitmap) as string), 31, '0')) as bitstr
  from
    dw.bitmp_30d
  where
    dy = '2020-12-16'
)
insert into
  table dw.bitmp_30d partition(dt = '2020-12-17')
select
  nvl(a.guid, b.guid) as guid,
  conv(
    -- conv函数把二进制转为10进制
    reverse(
      -- 反转把最近登陆反转到结尾,离今天越近数值越小
      case when a.guid is not null
      and b.guid is not null then concat('1', substr(bitstr, 1, 30)) when a.guid is null
      and b.guid is not null then concat('0', substr(bitstr, 1, 30)) when b.guid is null then rpad('1', 31, '0') end
    ),
    2,
    10
  ) as bitmap
from
  a full
  join b on a.guid = b.guid
```
##### 更新方法二(位运算)
**利用&的特性,1&1 =1 1&0 = 0**

T+1日，用户没活跃，则如下更新：
select bin(1073741823 & cast(conv('111111111111111111111111111111',2,10)*2 as int));

T+1日，用户活跃，则如下更新：
select bin(1073741823 & cast(conv('111111111111111111111111111111',2,10)*2+1 as int));
取前三十天的位,把今天的位设置为1,距今天越近,越小,即1073741823是经过处理后的bitmap登录信息

select bin(concat( substr(lpad(bin(1),31,0),2,31) , '1') & cast(conv('111111111111111111111111111111',2,10)*2 as int));

##245.漏斗模型
漏斗模型分析的处理,在SQL是比较难处理的,这里提供一种巧妙的方法:正则表达式,关于漏斗模型在Clickhouse中有自带的函数,非常方便,有兴趣的可以学习一下Clickhouse的文档,这里不再示例 https://clickhouse.tech/docs/en/

**注意对regexp_extract/ regexp_replace/sort_array/collect_list/ concat_ws的使用与学习**
```
create table dw.funnel_statistic_1d
( 
  funnel_name  string,-- 漏斗的业务步骤
  guid         string,-- 用户
  comp_step    int -- 用户完成这个步骤多少步
)
partitioned by (dy string)
stored as parquet 
tblproperties("parquet.compress" = "snappy")
;

insert into table dw.funnel_statistic_1d partition(dy='2020-12-12')
select
  '浏览分享添加' as funnel_name,
  guid,
  comp_step
from
  (
    select
      guid as guid,
      -- sort_array()返回的是一个数组  regexp_extract() 只能对字符串操作
      case when regexp_extract(
        concat_ws(
          ',',
          sort_array(
            collect_list(
              concat_ws('_', cast(`timestamp` as string), eventId)
            )
          )
        ),
        '.*?(pageView).*?(share).*?(addCart).*?',
        3
      ) = 'addCart' then 3 when regexp_extract(
        concat_ws(
          ',',
          sort_array(
            collect_list(
              concat_ws('_', cast(`timestamp` as string), eventId)
            )
          )
        ),
        '.*?(pageView).*?(share).*?',
        2
      ) = 'share' then 2 when regexp_extract(
        concat_ws(
          ',',
          sort_array(
            collect_list(
              concat_ws('_', cast(`timestamp` as string), eventId)
            )
          )
        ),
        '.*?(pageView).*?',
        1
      ) = 'pageView' then 1 else 0 end as comp_step
    from
      dw.enent_app_detail
    where
      dy = '2020-12-12'
      and -- 漏斗步骤(
        (
          eventId = 'pageView'
          and properties ['pageId'] = '877'
        )
        or (
          eventId = 'share'
          and properties ['pageId'] = '791'
        )
        or (
          eventId = 'addCart'
          and properties ['pageId'] = '72'
           )
         group by guid
     )o
where
  comp_step > 0;

-- 报表案例:统计完成步骤的人数

select 
count(if(comp_step>=1,1,null)) as step_1,--完成第一步的人
count(if(comp_step>=2,1,null)) as step_2,
count(if(comp_step>=3,1,null)) as step_3,
from 
dw.funnel_statistic_1d 
where dy='2020-12-12';
```
##246.宽表
在数据仓库架构层DWS层,采取的是大宽表集成的主题架构,这样的架构是清晰的,避免了多表join的操作,提高的性能,减少了使用难度
所谓宽表就是维度退化把维度属性增添至事实表中,并且事实表中的粒度也是不同的,不同的粒度由同一个自然键相连着,不同的粒度的事实信息采用union all 来连接,分别求取
```
-- 源表.订单主要信息表  订单签收人信息表
-- order order_desc
-- 重要的处理事项:是否要去重,where(分区字段),
-- 可以考虑把temp单独地做成一张表,这样方便计算
with temp as (
select
  a.order_id   ,-- 订单ID
  a.order_date ,-- 订单日期
  a.user_id    , -- 用户ID
  a.order_money , -- 订单金额(应付金额)
  a.order.status , -- 订单状态(6:退货,7:拒收)
  a.pay_type , -- 订单支付类型
  b.area_name ,-- 收货人地址
  b.address ,-- 手工地址
  b.coupen_money -- 代金券金额
from 
  order a join order_desc b on a.user_id = b.user_id
)

-- 模块一
select 
  user_id , -- 用户
  min(order_date) , -- 首单
  max(order_date) , -- 末单日期
  datediff('2020-12-12',min(order_date)) , -- 首单距今时间
  datediff('2020-12-12',max(order_date)) , -- 末单距今时间
  count(if(datediff('2020-12-12',order_date) <=30,1,null)), -- 最近三十天购买次数
  sum(if(datediff('2020-12-12',order_date) <=30,order_money,0)), -- 最近三十天购买的金额
  count(if(datediff('2020-12-12',order_date) <=60,1,null)), -- 最近六十天购买次数
  sum(if(datediff('2020-12-12',order_date) <=60,order_money,0)), -- 最近六十天购买的金额
  count(if(datediff('2020-12-12',order_date) <=90,1,null)), -- 最近九十天购买次数
  sum(if(datediff('2020-12-12',order_date) <=90,order_money,0)), -- 最近九十天购买的金额
  min(order_money) , -- 最小的金额
  max(order_money) , -- 最大的金额
  count(if(order_status != '6' and order_status != '7',1,null)) , -- 累计消费次数(不含推拒),注意是6.7是字符串
  sum(if(order_status != '6' and order_status != '7',order_money,0)) , -- 累计代金券金额(不含推拒)
  avg(order_money) , -- 平均订单金额
  avg(if(datediff('2020-12-12',order_date) <=90,order_money,null)) -- 最近九十天的平均的订单的金额           
from
  temp
group by user_id -- 用户画像标签计算


-- 模块二:常用地址
select 
  user_id, 
  common_address -- 常用地址
from 
(
select 
  user_id  ,-- 用户
  concat_ws(' ',address,area_name ) as common_address  ,-- 常用地址(可能为null)
  row_number() over(partition by user_id order by count(1) desc rows between unbounded preceding  and current row) as rn
from
  temp 
group by user_id,concat_ws(' ',address,area_name )
) o 
where o.rn = 1

-- 模块三:常用支付方式
select
  user_id, -- 用户ID
  pay_type,-- 常用支付方式
from 
(
select 
  user_id,
  pay_type,
  row_number() over(partition by user_id order by num_pay desc rows between unbounded preceding  and current row ) as rn
from 
(
select 
  user_id,
  pay_type,
  count(1) as num_pay
from
  temp
group by user_id,pay_type
) o 
) o1
where rn = 1 

-- 模块四:购物车
select 
  user_id , -- 用户ID
  count(1),      --最近30天加购次数
  sum(number) ,-- 最近30天加购商品件数
  sum(if(submit_time is not null ,number ,0)) -- 最近30天提交次数
from
  cart -- 购物车信息表
where datediff('2020-12-12',add_time) <= 30 -- 先过滤
group  by user_id

-- 模块五:可能有的用户订单购物车某一部分没有,考虑是否这两张表的uid作为数据链接的基础
with temp5 as (
select  user_id from order 
union 
select  user_id from cart
)

-- 整合五个模块即可求出
```
##247.增量与全量
增量,全量导入数据以及如何更新维度表,事实表的更新,以及增添属性应该如何处理在之前的文章里已经写过了,不再写了
##248.归因分析
归因分析,是一类**循环问题**,我们用SQL来处理是复杂的,难度较高的,所以这里我提供在spark开发环境下的如何处理循环问题,对于归因分析本身就有多种的分析方法:
末次触点分析,首次触点分析,末次非直接点击归因,线性归因分析,U型归因分析等等,这里我们简单地示例一下时间衰减归因分析在spark中是如何处理的,至于其他的分析方法在我的hexo博客中已经有了简单的示例了,这里不再写了
https://nino-laiqiu.github.io/2020/12/10/Data-System-Warehouse/
```
 val decay = (event_list: mutable.WrappedArray[String], attribute: mutable.WrappedArray[String]) => {
    var tuple: (List[String], List[String]) = event_list.toList.map(_.split("_")(1)).span(_ != "e6")
    var buffer = ListBuffer[(String, Double)]()
    while (tuple._2.nonEmpty) {
      if (tuple._1.nonEmpty) {
        val list = tuple._1.toSet.toList
        val seq = for (i <- list.indices) yield Math.pow(0.9, i)
        val tuples1: immutable.Seq[(String, Double)] = for (j <- list.indices) yield (list(j), seq(j) / seq.sum)
        for (elem <- tuples1) {
          buffer += elem
        }
      }
      tuple = tuple._2.tail.span(_ != "e6")
    }
    buffer
  }
```
##249.报表
报表的开发基于DWS层的集成主题建模架构的,这里简单的示例一些主题报表SQL的写法
#####用户访问间隔
SQL使用偏移量函数,注意对时间范围的规整
```
//用户访问间隔分析
//guid,first_dt,rng_start,rng_end
object UserAccessInterval {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName(this.getClass.getSimpleName).master("local[*]").getOrCreate()
    val schema = new StructType()
      .add("guid", DataTypes.LongType)
      .add("first_dt", DataTypes.StringType)
      .add("rng_start", DataTypes.StringType)
      .add("rng_end", DataTypes.StringType)
    val frame = spark.read.schema(schema).csv("file:///C:\\Users\\hp\\IdeaProjects\\TiTan\\dataware\\src\\main\\resources\\UserAccessInterval.csv")
    frame.createTempView("tmp")
    /*1.过滤出rng_end时间与今天相比小于30的数据
    2.把rng_end为9999-12-31的日期改成2020-03-14
    3.如果first_dt与今天相比大于30天则更改为今天的日期-30,否则不变*/
    val frame1 = spark.sql(
      """
        |select
        |  guid,
        |  if(datediff('2020-03-14',rng_start)>=30,date_sub('2020-03-14',30),rng_start) as rng_start,
        |  if(rng_end = '9999-12-31','2020-03-14',rng_end) as rng_end
        |from
        |  tmp
        |where datediff('2020-03-14',rng_end) <=30
        |""".stripMargin)

    //统计登陆间隔
    //按用户来分组
    val guidrdd = frame1.rdd.map(row => {
      val guid = row.getAs[Long]("guid")
      val rng_start = row.getAs[String]("rng_start")
      val rng_end = row.getAs[String]("rng_end")
      (guid, (rng_start, rng_end))
    }).groupByKey()

    val value = guidrdd.flatMap(data => {
      val guid = data._1
      val sorted = data._2.toList.sortBy(_._2)
      //统计间隔为0的次数
      val list: List[(Long, Int, Int)] = for (elem <- sorted) yield (guid, 0, datediff(elem._1, elem._2))

      //统计间隔为N的次数(要排序),用后一个的rng_start- 前一个的 rng_end
      val list1 = for (i <- 0 until sorted.length - 1) yield (guid, datediff(sorted(i)._2, sorted(i + 1)._1), 1)
      list ++ list1
    })
    import spark.implicits._
    import org.apache.spark.sql.functions._
    value.toDF("guid", "interval", "times")
      .groupBy("guid", "interval").agg(sum("times") as "times")
      .show(100, false)
  }

  //时间相减
  def datediff(date1: String, date2: String): Int = {
    val sdf = new SimpleDateFormat("yyyy-MM-dd")
    val s1: Date = sdf.parse(date2)
    val s2: Date = sdf.parse(date1)
    ((s1.getTime - s2.getTime) / (24 * 60 * 60 * 1000)).toInt
  }
}
```

#####复购率分析
```
-- 复购率分析
-- 需要两张表 订单主要信息表 order 订单商品详情表 order_goods
-- 订单 用户id 商品 品类
with x as (
  select
    t1.order_id,
    t1.use_id,
    t2.good_id,
    t2.type_id
  from
    (
      select
        order_id,
        use_id
      from
        order
      where
        dy = '2020-12-12'
    ) t1
    join order_goods t2 on t1.order_id = t2.order_goods
)
select
  '2020-12-12' as dy,
  --今日时间
  type_id,
  -- 品类
  count(use_id) -- 购买该品类多少人,已经对人进行了去重了,上一步group by use_id
  count(if(user_cnt >= 2, 1, null)) as time_2,
  -- 统计购买次数大于2次的人的个数
  count(if(user_cnt >= 3, 1, null)) as time_3,
from
  (
    select
      -- 一个用户购买多少该品类
      type_id,
      use_id,
      count (distinct order_id) as user_cnt 
      -- 为什么要对order_id 去重因为一个品类一条数据,一个用户一次可能购买多个品类
    from
      x
    group by
      use_id,
      type_id
  ) o
group by
  o.type_id


-- 复杂报表分析
-- 表报分析
-- 店铺 地区 月份 月销售金额 同地区同月份所有店铺销售金额 该店铺到该月的累计金额
-- 数据
shop  province    provincetable
shop month sale   shoptable


with y as (
  with x as (
    select
      a.shop,
      max(b.province) as province,
      a.month,
      sum(a.sale) as sale_cnt
    from
      shoptable a
      join provincetable b on a.shop = b.shop
    group by
      a.shop,
      a.month
  )
  select
    -- 增加字段累计金额
    shop,
    province,
    month,
    sale_cnt,
    sum(sale_cnt) over(
      partition by shop
      order by
        month rows between unbounded preceding
        and current row
    ) as shop_account_cnt
  from
    x
) -- 按地区和月份进行分组,求同地区同月份下所有店铺的销售金额
select
  y.shop,
  y.province,
  y.sale_cnt,
  z.province_sale,
  y.shop_account_cnt
from
  (
    select
      province,
      month,
      sum(s.sale) as province_sale
    from
      shoptable s
      join provincetable p on s.shop = p.shop
    group by
      s.month,
      p.province
  ) z
  join y on z.province = y.province
  and z.month = y.month
```
#####用户偏好
```
-- 用户购物偏好标签
-- 关联三张表
-- 订单表 order 订单商品详情表 orders_good 商品描述表 goods
-- 做一张中间表存储三张表关联的结果

create table tag(
user_id string ,
first_cat_name  string ,
second_cat_name string ,
third_cat_name string,
brand_id_name  string,
)
stored as parquet tblproperties("parquet.compress" = "snappy")

insert  into  tag 
select 
  order.user_id, -- 用户ID
  goods.first_cat_name, -- 一类标签
  goods.second_cat_name,
  goods.third_cat_name,
  goods.brand_id_name -- 商品标签
from 
  order 
join orders_good on order.order_id = orders_good.order_id
join goods on  orders_good.goods_id = goods.sku_id

with tmp1 as (
select 
  user_id,
  first_cat_name
from 
(
select 
  user_id,
  first_cat_name,
  row_number() over(partition by user_id order by count(1)) as rn-- 标签数
from
  tag
group by user_id,first_cat_name
)
where rn = 1
),
tmp2 as (
select 
  user_id,
  second_cat_name
from 
(
select 
  user_id,
  second_cat_name,
  row_number() over(partition by user_id order by count(1)) as rn 
from
  tag
group by user_id,second_cat_name
)
where rn = 1
),
tmp3 as (
select 
  user_id,
  third_cat_name
from 
(
select 
  user_id,
  third_cat_name,
  row_number() over(partition by user_id order by count(1)) as rn 
from
  tag
group by user_id,third_cat_name
)
where rn = 1
),
tmp4 as (
select 
  user_id,
  brand_id_name
from 
(
select 
  user_id,
  brand_id_name,
  row_number() over(partition by user_id order by count(1)) as rn 
from
  tag
group by user_id,brand_id_name
)
where rn = 1
)

select 
  tmp1.user_id, -- 用户ID
  tmp1.first_cat_name as common_first_cat, -- 最常购买的一类标签
  tmp2.second_cat_name as common_second_cat,
  tmp3.third_cat_name as common_third_cat, -- 最常购买的三类标签
  tmp4.brand_id_name as common_brand_id -- 最常购买的标签
from 
  tmp 
join  tmp2 on tmp1.user_id = tmp2.user_id
join  tmp3 on tmp1.user_id = tmp3.user_id
join  tmp4 on tmp1.user_id = tmp4.user_id
```

##250.etl处理
etl的处理包含数据的导入导出(全量/增量),数据的清洗与标准化,数据的治理,etl元数据的治理,我的关注点在数据的治理和分类
这是一个大的模块,以后再写
