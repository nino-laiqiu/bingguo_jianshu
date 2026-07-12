##1.SQL的问题
1. 集合函数开窗,在kylin中over()需要聚合后的值,不可以直接取over(sum(字段)),应该分步骤求取
2. SQL中查询中where条件不可以直接过滤measure为Null的值,因为这些measure是根据各个维度聚合的值,不group by 是查不到值的
3. SQL查询条件where的限制类型是不会自动转化的

##2.采坑
1. 当某列数据都大于0时,查询sum返回负数,原因是在hive中列的声明是smallint或者int,但是在kylin中的sum求和的值可能大于类型的最大值,在这种情况下返回的就是负数,解决的方法是在hive中把数据的类型更改为bigint等类型,同步到kylin中,修改hive中字段的类型

2. cube在构建时发生类型转化异常,java.lang.classcastexception...具体的原因是字段的历史数据为int类型,但是当前为bigint类型,解决的方法是回刷历史数据全部转化为bigint类型,重新构建cube

3. kylin在使用abs绝对值排序的乱序问题使用case when 度量来解决

4. order by 之后要加limit不然kylin可会乱序

##3.cube的相关问题
1. cube中build数据时间周期为**左闭右开**
2. cube中度量measure为MAX时,那么在kylinSQL查询除了自身的count以及measure之外的其他聚合函数都无法使用,因为没有预聚合

##4. 简单优化
建议将强制维度放在rowkeys的开头
where条件作用很大的,建议将基数大的维度放在基数下的维度前面

