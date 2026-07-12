##1.Kylin除法运算踩坑
本次踩坑同样适用于presto

Kylin在进行除法运算的时候,如果被除数为0 就会报错
```js
Division undefined
```
此错误仅在使用BigDecimal(bigint)做除法时，且0/0的情况下才会提示。
x/0时，仅提示Division by zero。

解决方法
#####1. 嵌套判断
```
case when sum() = 0 then null  else sum() end 
```
或者在外侧添加where先于除法运算
一般的为了避免发生线上错误,需要测试一下每天组内的数据是否为空
#####2. 转化类型
如果被除数是double就没问题

#####3. 数据类型
一般的0在数仓中统一转化为null,这样group by的时候就不会出现null值,这是最主要的方法


补充:关于kylin接口的治理,迭代过程中造成的接口乱用,如何治理如何整合已经上线的接口

