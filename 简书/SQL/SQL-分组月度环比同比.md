有一个需求是按月份统计环比和同比值,每次都取一遍非常麻烦,同时也容易出错,这里我把要取的数据报表化,这里提供一个模板,这个SQL写了我好久,经过对比数据发现,这样写是没什么问题的,这里提供一些注意事项

1. 同比环比的定义
月份同比计算  (2021-01 - 2020-01) / 2020-01
月份环比计算  (2021-02 - 2021-01) / 2021-01

2. 在计算同比的时候要注意order by要对月份和年份都要排序,原因是lead 中的order by只指定了一个排序规则,这里都要指定,否则会乱序,这个检查了我好久的时间

3. 同比和环比的SQL思路
同比,lead按照业务线分组,之后lead上推度量值,注意要排序使得相同月份,不同年份的数据在一起
环比,这里也是使用lead来上推,这个没什么注意事项

开始的写法,我这里把同比和环比放在一行了,这种写法的缺点就是不够直观
```
select 
lag,j_month,basis,relative
FROM
(
SELECT lag, j_month, order_price
    , (order_price - lead_price_basis) * 1.0 / lead_price_basis AS basis
    , relative,
FROM (
    SELECT lag, j_month, order_price, relative
        , lead(order_price, 1, NULL) OVER (PARTITION BY lag ORDER BY substring(j_month, 6, 7) DESC,substring(j_month, 0, 4) desc) AS lead_price_basis
    FROM (
        SELECT lag, j_month, order_price
            , (order_price - lead_price_relative) * 1.0 / lead_price_relative AS relative
        FROM (
            SELECT lag, j_month, order_price
                , lead(order_price, 1, NULL) OVER (PARTITION BY lag ORDER BY j_month DESC) AS lead_price_relative
            FROM (
                SELECT concat(lag_country, bu) AS lag, j_month, order_price
                FROM (
                    SELECT bu
                        , CASE 
                            WHEN to_country_id = 1 THEN '国内'
                            WHEN to_country_id <> 1 THEN '国外'
                        END AS lag_country, substring(d, 0, 7) AS j_month
                        , round(SUM(order_price), 0) AS order_price
                    FROM table
                    WHERE d >= '2019-01-01'
                        AND bu IN ('机票', '酒店', '度假')
                        AND d <> '4000-01-01'
                    GROUP BY bu, substring(d, 0, 7), CASE 
                            WHEN to_country_id = 1 THEN '国内'
                            WHEN to_country_id <> 1 THEN '国外'
                        END
                ) a
                ORDER BY lag, j_month DESC
            ) b
            WHERE lag IS NOT NULL
        ) c
    ) d
) e
) f 
where  substring(j_month,0,4) = '2021'
```

后来使用union all 把同比和环比分开显示,然后使用行转列,就得到了下面的效果图,个人认为是一种非常好的报表展示
![展示效果](https://upload-images.jianshu.io/upload_images/9049859-b5a3a89eaf8bf118.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
select
lag,j_lag,
MAX(case when substring(j_month,6,7) = '01' then relative  end ) as m1,
MAX(case when substring(j_month,6,7) = '02' then relative  end ) as m2,
MAX(case when substring(j_month,6,7) = '03' then relative  end ) as m3,
MAX(case when substring(j_month,6,7) = '04' then relative  end ) as m4,
MAX(case when substring(j_month,6,7) = '05' then relative  end ) as m5,
MAX(case when substring(j_month,6,7) = '06' then relative  end ) as m6,
MAX(case when substring(j_month,6,7) = '07' then relative  end ) as m7,
MAX(case when substring(j_month,6,7) = '08' then relative  end ) as m8,
MAX(case when substring(j_month,6,7) = '09' then relative  end ) as m9,
MAX(case when substring(j_month,6,7) = '10' then relative  end ) as m10,
MAX(case when substring(j_month,6,7) = '11' then relative  end ) as m11,
MAX(case when substring(j_month,6,7) = '12' then relative  end ) as m12
from 
(
SELECT lag, '环比' AS j_lag, j_month
    , (order_price - lead_price_relative) * 1.0 / lead_price_relative AS relative
FROM (
    SELECT lag, j_month, order_price
        , lead(order_price, 1, NULL) OVER (PARTITION BY lag ORDER BY j_month DESC) AS lead_price_relative
    FROM (
        SELECT concat(lag_country, bu) AS lag, j_month, order_price
        FROM (
            SELECT bu
                , CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END AS lag_country, substring(d, 0, 7) AS j_month
                , round(SUM(order_price), 0) AS order_price
            FROM table
            WHERE d >= '2019-01-01'
                AND bu IN ('机票', '酒店', '度假')
                AND d <> '4000-01-01'
            GROUP BY bu, substring(d, 0, 7), CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END
        ) a
    ) b
    WHERE lag IS NOT NULL
) c
UNION ALL
(
SELECT lag, '同比' , j_month
    , (order_price - lead_price_basis) * 1.0 / lead_price_basis 
FROM (
    SELECT lag, j_month, order_price
        , lead(order_price, 1, NULL) OVER (PARTITION BY lag ORDER BY substring(j_month, 6, 7) DESC,substring(j_month, 0, 4) desc) AS lead_price_basis
    FROM (
        SELECT concat(lag_country, bu) AS lag, j_month, order_price
        FROM (
            SELECT bu
                , CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END AS lag_country, substring(d, 0, 7) AS j_month
                , round(SUM(order_price), 0) AS order_price
            FROM table
            WHERE d >= '2019-01-01'
                AND bu IN ('机票', '酒店', '度假')
                AND d <> '4000-01-01'
            GROUP BY bu, substring(d, 0, 7), CASE 
                    WHEN to_country_id = 1 THEN '国内'
                    WHEN to_country_id <> 1 THEN '国外'
                END
        ) a
    ) b
    WHERE lag IS NOT NULL
) c
)
) 
WHERE substring(j_month, 0, 4) = '2021'
GROUP by lag,j_lag
order by substring(lag,3,4),substring(lag,0,2),j_lag
```
同时我们要对多月份进行汇总计算同比值,如果在这些报表上直接添加汇总是非常麻烦的,需求也是易变的,这里要对每一年的sum(度量)进行报表化,分业务类型,这个报表是简单的,直接分别统计不同年份,直接 union all ,然后使用行转列转变格式即可,还是非常简单的,我们基于这张表来计算多月份的同比就比较简单了,就不用每次麻烦取数了,这张报表展示如下
![报表的展示](https://upload-images.jianshu.io/upload_images/9049859-c9e7d9499e22d163.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完成这个需求,我发现SQL的设计同SQL的优化一样重要,如果这个SQL得出的结果不够直观还要二次处理非常低效


