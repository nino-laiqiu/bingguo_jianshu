>https://blog.csdn.net/BeiisBei/article/details/103566584

##1.控制窗口的大小

**PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点**

例如:
```
select name,orderdate,cost,
sum(cost) over() as fullagg, --所有行相加
sum(cost) over(partition by name) as fullaggbyname, --按name分组，组内数据相加
sum(cost) over(partition by name order by orderdate) as fabno, --按name分组，组内数据累加 
sum(cost) over(partition by name order by orderdate rows between unbounded preceding and current row) as mw1   --和fabno一样,由最前面的起点到当前行的聚合 
sum(cost) over(partition by name order by orderdate rows between 1 preceding and current row) as mw2,   --当前行和前面一行做聚合 
sum(cost) over(partition by name order by orderdate rows between 1 preceding and 1 following) as mw3,   --当前行和前边一行及后面一行 
sum(cost) over(partition by name order by orderdate rows between current row and unbounded following) as mw4  --当前行及后面所有行 
from order; 
```

#####first_value 和 last_value

first_value取分组内排序后，截止到当前行，第一个值
last_value取分组内排序后，截止到当前行，最后一个值

![image.png](https://upload-images.jianshu.io/upload_images/9049859-04530b3c0f18dbe3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




