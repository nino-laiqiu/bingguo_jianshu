背景,SQL需要同时在kylin和presto中同时跑通,需求是求取占比,但是某种类型占比稳定在99.9%,如果只是输出小数点后两位,极有可能其他的类型占比都是0%,所以需要展示小数点后5,6位,如何解决?

在写SQL的时候遇到的问题是将decimal/double类型转化为字符串来截取和拼接%,但是会出现小数点前0丢失的问题,一开始的解决方法是 在前面拼接一个 '0'|| ,但是presto就不会出现这个问题,导致结果不一致性

后来使用position函数来解决,恰巧的是presto中也有这个函数,使用的方法如下
```
select
substring(ratio,1,7) || '%'  as ratio
from
(
select case when position('.' in ratio) = 1 then '0'|| ratio else ratio end as ratio
) t1 
```
