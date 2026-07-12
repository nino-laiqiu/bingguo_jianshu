##1. Kylin的定义
开源的分布式分析引擎
提供hadoop/spark之上的SQL查询接口以及多维分析(OLAP)
支持超大规模数据,亚秒级巨大hive表

##2. Kylin的简介
1. 标准SQL接口
2. 支持超大数据集的查询(千亿级别)
3. 亚秒级响应
4. 可伸缩性和高吞吐率
5. BI工具集成
ODBC JDBC RESTFUL Zepplin插件

##3. Kylin架构
Kylin拉取来自HIVE/KAFKA中的数据,生成modal(模式),构建的cube存储在HBASE中(OLAP CUBE),查询SQL通过webapp restapi bi jdbc odbc等进行查询

**kylin的架构:**
cube  build engline 处理计算所有离线任务
metadata 对保存在kylin中的元数据进行管理,其中包含最为重要的cube元数据
routing 负责将解析的SQL生成的执行计划转化为cube缓存的查询
query engine 当cube准备就绪后,查询引擎就能获取并解析用户查询,生成执行计划
rest server面向应用程序开发的入口点

##4. CUBE基本概念
**1. 维度和度量**
维度:即分析数据的角度,在统计时可以按照不同维度枚举值,应用聚合函数运算
度量:即被分析(集合)的统计值,也就是聚合运算的结果
**2. CUBE&CUBOID**
CUBOID:一组维度的组合,及其对应度量值的聚合结果
CUBE:所有维度的组合及其度量值聚合结果,及CUBOID的全集

##5. CUBE数据流
对数据离线预计算,计算结果保存至hbase中
在线查询直接获取出聚合结果
体现空间换时间的思想

**CUBE构建**
将数据从数据源拉过来
初始化,构建前准备
计算维度组合及度量聚合值
转化为HFILE(编码压缩)--(可以优化一下)
保存至HBASE中
**CUBE查询**
获取解析转化用户SQL
转化SQL为CUBE查询计划
查询并解码数据
CUBE结果再处理
返回结果

##6. CUBE构建算法
Kylin默认使用MapReduce(spark不稳定)来对数据预处理,一次build结果就是一个segment,build过程涉及多个CUBOID的创建,具体创建算法过程由kylin.cube.algorithm参数来决定,参数设置可选值**auto layer inmem默认值是auto**,即kylin会根据数据分布自动选择最优算法

#####逐层构建算法
逐层构建算法,首先会计算出基础CUBOID(FULL DATA),然后按照维度数逐层减少聚合计算不同的CUBOID
**每一轮的计算都是一个MapReduce任务,且串行执行**
一个N维的CUBE,至少需要N次MapReduce Job
分层计算,算法代码清晰易维护调试,且运行稳定,但是CUBE维度较多时,存在多个MR任务,效率低,慢,增加集群的压力
![逐层构建算法](https://upload-images.jianshu.io/upload_images/9049859-20e618ddbfcfca0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####快速构建算法
**对mapper所分配的数据块,将它计算成一个完整的小CUBE段(包含所有CUBOID);每个mapper将计算完的cube段输出给reduceer做合并,生成大CUBE**
优点是快,mapper会利用内存做预计算,算出所有的组合,一轮MapReduce完成所有层次的计算,减少hadoop任务的调配,但是缺点是耗内存以及不够稳定
![快速构建算法](https://upload-images.jianshu.io/upload_images/9049859-f2cd455af6b27d8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##7. CUBE构建过程
#####一般步骤
1. 创建hive中间表
2. 重分发中间表
3. 提取事实表唯一列
4. 构建维度字典
5. 保存CUBOID统计数据,创建HTBLE
6. 构建基础CUBOID层级构建,构建N维CUBOID或者构建CUBE-快速构建算法
7. CUBOID数据转化HFILE
8. HFILE数据导入HBase
9. 更新CUBE元信息
10. 清理资源(中间表)
了解一下名词:HTBLE HFILE

#####1. 创建HIVE中间表
从指定的hive表中提取数据,对于多表连接,这一部中会提前连接到一张临时表中
**构建cube指定与hive相同的分区列,从而可以避免从hive提取数据进行全表扫描**

#####2. 重分发中间表(避免数据倾斜)
在第一步之后,hive在HDFS上的目录里生成数据文件,但是可能有大有小甚至是有空文件,通过增加重分发步骤,避免后续MR任务数据倾斜
1. **根据临时中间表的总行数,默认按照以百万行每个文件的配置,设置每个文件输出个数如果资源充足,可以减少每个文件的行数,提高计算并行度:
kylin.job.mapreduce.mappper.input,rows**

2. **kylin默认通过:INSERT OVERWRITE TABLE .... DISTRBUTE BY RAND() 将数据发送到指定数量的reducer上,输出大小相近的文件,当存在超高基维度时,可以使用DISTRBUTE BY   该维度**
#####3.提取事实表唯一列(bitmap)
1. 对维度列,通过MR任务去重提取的唯一值
2. 统计cube数据,估算每个CUBOID的行数

#####4. 构建维度字典
根据前一步提取的维度列唯一值,kylin在内存中构建维度字典
当为**超高基维度**时,可能会报出类似字典不支持高基数的异常,对于超高基列"fixed_length" , "integer"等编码方式

#####5. 保存CUBOID统计数据,创建HTable表
![kylin元数据信息](https://upload-images.jianshu.io/upload_images/9049859-1987b116f41b2953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####6. 构建基础CUBOID-逐层构建算法
用hive的中间临时表构建基础的CUBOID,是逐层构建cube算法的第一轮MR计算
Mapper的数目与重分发的文件数目相等
**此步骤任务中,reduce的数量根据cube统计数目估计的:默认每500MB使用一个reducer输出,若reducer的数量减少,可以在配置文件中减少reduce输入数据量,从而增加输出文件个数,提高下一轮MR任务的并行度**
**kylin.job.mareduce.default.reduce.input.mb=200**

#####7. 构建N维CUBOID
此步骤是在逐层构建cube的过程,每一步以前一步的输出作为输入,然后去掉一个维度聚合成一个字CUBOUD
**优化点:在设计rowkey序列时,基数较小的维度,尽量放在靠后的位置**
#####8. 构建CUBE-快速构建算法
这个步骤使用一个新的算法来构建CUBE,"逐片"构建(内存构建),他会使用一轮MR来计算所有的CUBOUD,优点是更快.缺点是更耗内存
**关于他的优化点是:在conf/kylin_job_inmem_xml,专门为这一步提高mapper内存,默认3G
#####9. CUBOID数据转化为HFILE
这一步启动一个MapReduce任务来将CUBOID文件(序列文件格式)转换为Hbase的HFILE文件格式
**Kylin通过cube统计数据计算Hbase的region的数目,默认情况下每5G数据对应一个region,region的数量越多,reducer越多,因此可以在配置文件中减少一个region的数据量,从而增加reducer个数,增加这一步的并行度**
#####10. HFILE数据导入Hbase表中
使用Hbase-API来将HFILE导入到region server中
#####11.更新CUBE文件
在导入数据到Hbase后,kylin在元数据中将对应cube segment标注为ready
#####12. 清理资源(中间表)
将使用到的中间临时表从hive表中删除

##8. CUBE查询过程
1. parser:将SQL转化为AST
2. validation:基于metadata进一步校验SQL合法性
3. optimizer:根据优化规则优化生成的logicplan
4. kylin适配:将逻辑执行计划转化为kylin物理执行计划
5. query:基于生成的plan读取cube数据并计算
##9. CUBE设计优化
#####说明
假设有10个维度,在没有优化的情况下,会存在2^10种cuboid
假设有20个维度,在没有优化的情况下,则有1048576中cuboud
维度越多,造成预计算的数据量越大,对计算的数据量越大,对计算和存储都会造成非常大的压力,甚至是无法计算

#####膨胀率
膨胀率= cube数据量/原始数据量
通过膨胀率可以粗略的判断cube是否需要优化,一般维持在0% - 1000%之间
膨胀率高一般有如下原因:
 1. cube中的维度较多,没有进行较高的剪枝优化
 2. cube中存在超高基维度
 3.  存在比较占用空间的度量,比如说count distinct,每一行都要保存一个寄存器

#####检查CUBOID
**kylin.sh org.apache.kylin.enaine.mr.common.CubeStatsReader CubeName**
该命令会显示所有构建好的cuboid的状态,包括cuboid组合,行数,大小, shrink,.其中shrink代表和父cuboid的相似度,相似度接近100%,说明这个Cuboid虽然比它的父亲Cuboid少了一个维度,但是并没有比它的父亲Cuboid少很多行数据。换而言之,即使没有这个Cuboid,在查询时使用它的父亲Cuboid,也不会有太大的代价,就可以对这个Cuboid进行剪枝操作。

#####如何优化
**聚合组**
1. 聚合组可以将维度根据业务需求分组,组合关系紧密的维度,隔离关系弱的维度,从而减少cuboids个数。
2. 多个聚合组内的维度可以部分重合,每个聚合组都会根据定义的维度关系都会贡献出cuboids.
3. 所有聚合组贡献的cuboids的并集,就是cube需要构建的cuboid的全集。多个聚合组之间存在cuboid重复时, kylin不会重复构建。

**强制维度**
1. 如果一个维度被定义为强制维度,那么这个分组产生的所有Cuboid中每一个Cuboid都会包含该维度。
2. 每个分组中都可以有0个、1个或多个强制维度,每多一个强制维度,总的cuboids减少一半。
3. 如果业务查询逻辑查询时,一定包含某个维度,则可以在该分组中把该维度设置为强制维度。

**层级维度**
1. 层级维度可以将多个维度按照指定的顺序,只以层级的顺序组合。
2. 多个层级维度组合之间,不能有重复的维度。

![层级维度优化示例](https://upload-images.jianshu.io/upload_images/9049859-c89d35e33e69e941.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**联合维度**
1. 联合维度将多个维度捆绑,被捆绑的所有维度,**在cuboid中要么都出现,要么都不出现。**
2. 如果根据这个分组的业务逻辑,多个维度在查询中总是同时出现,则可以在该分组中把这些维度设置为联合维度。
![联合维度优化示例](https://upload-images.jianshu.io/upload_images/9049859-1a22d900e2032e62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##10. CUBE构建优化
需求: Cube构建优化指优化cube的build过程,加快构建速度和减少构建的数据量。

**1)设置与Hive表相同的分区列
2)提高build过程MapReduce任务的并发度。
3)修改超高基维度的字典编码方式。
4)提高逐层构建算法的构建并行度。
5)Rowkey优化,超高基维度尽量放在前面。
6)增加Cuboid转化为Hfile的并行度。**
##11. CUBE查询优化
**1).提高Cube查询并发粒度当Segment中某个Cuboid的大小超出一定阈值时,系统会将该cuboid的数据分片到多个分区中,实现Cuboid数据并行读取,提高Cube查询速度。
2).减少Cube体积。通过前面提到的cube剪枝优化,减少cuboid数量,减少cube体积。
3).Segment手动删除。每一次Build之后,都会在HDFS上形成一个Segment,并且聚合结果加载到Hbase中后, Segment也不会被删除,会被保留下来供Segment合并时使用。如果确定某些Segment不会参与合并,可以手动删除HDFS上的Segment.**
##12. CUBE构建与反馈
![CUBE构建流程](https://upload-images.jianshu.io/upload_images/9049859-1a3a8194ae6f43f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于历史数据回补的流程:(分区设置??)
替换开始结束日期,回补历史数据,然后每日full build (历史回补结束日期与每日任务的开始日期无交叉)

对kylin调研的结果
16个维度,4个度量,2年的数据,存储量为150G
查询效率,查询近一个月的数据,1,2秒,亚秒级响应
cube构建优化复杂,但是复用性高
cube监控可以调用api,对开发人员友好

>难点
>1.kylin构建维度字典
>2.kylin中cube查询过程kylin适配过程
>3.执行计划中的bitmap相关优化的策略
>4.cube设计的优化(如何优化)
>5.对于历史数据回补分区的设置问题
>6.cube构建实践中常用维度优化措施,维度优化手段,例如强制维度,层级维度,联合维度,以及这些优化方法是否冲突
>7.关于kylin分区计算结果的保留问题
>8.cube维度的相关维度,怎么重新构建但是保留复用性
>9.kylin中的去重算法和top_N


