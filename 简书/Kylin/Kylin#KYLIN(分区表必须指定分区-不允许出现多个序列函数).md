##分区表必须指定分区
对某张hive表构建cube,因为它只有一个历史分区,所以我就没有指定分区,导致报错,因为是分区表,如果不指定分区,kylin就不知道如何构建cube

##不允许出现多个序列函数
在求取某类型同期排名和当期排名的时候出现的问题
```
SELECT rw, rn, a.master_hotel_name
FROM (
    SELECT master_hotel_name, ROW_NUMBER() OVER () AS rw
    FROM (
        SELECT master_hotel_name, SUM(arrival_room_nights) AS room_nights
        FROM table
        WHERE ........
            AND master_hotel_name IS NOT NULL
        GROUP BY master_hotel_name
        ORDER BY SUM(arrival_room_nights) DESC
        LIMIT 10
    ) t
) a
   left  JOIN (
        SELECT master_hotel_name, ROW_NUMBER() OVER (ORDER BY room_nights DESC) AS rn
        FROM (
            SELECT master_hotel_name, SUM(arrival_room_nights) AS room_nights
            FROM table
            WHERE .....
            GROUP BY master_hotel_name
        ) b
    ) c
    ON a.master_hotel_name = c.master_hotel_name
```

错误的写法,在两个子句中都使用的序列函数
```
SELECT row_number() over() as rw, rn, a.master_hotel_name
FROM (
        SELECT master_hotel_name,SUM(arrival_room_nights) as room_nights
        FROM table
        WHERE  .......
        GROUP BY master_hotel_name
        ORDER BY SUM(arrival_room_nights) DESC
        LIMIT 10
) a
   left  JOIN (
        SELECT master_hotel_name, ROW_NUMBER() OVER (ORDER BY room_nights DESC) AS rn
        FROM (
            SELECT master_hotel_name, SUM(arrival_room_nights) AS room_nights
            FROM table
            WHERE  ......
            GROUP BY master_hotel_name
        ) b
    ) c
    ON a.master_hotel_name = c.master_hotel_name
```
改进后的写法,把第一个序列函数提取到嵌套的外层来解决,当然还有其他的解法,比如上次分享的使用union all 来结合偏移量函数来解决,至于正确的写法还没有产出
