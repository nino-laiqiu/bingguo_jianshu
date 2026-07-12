##1.为什么要引入Apache Spark
MapReduce框架的设计具有局限性，**一轮MapReduce任务只能处理一个Map任务和一个Reduce任务，中间结果需要保存到磁盘以供下一轮任务使用**，如果有多轮任务的话，必然需要消耗大量的网络传输和磁盘I/O，而这些都是非常耗时的操作，导致MapReduce构建Cube的性能不佳
Spark是**基于内存的迭代计算框架**，不依赖Hadoop MapReduce的两阶段范式，由RDD组成有向无环图（DAG），RDD的转换操作会生成新的RDD，新的RDD的数据依赖父RDD保存在内存中的数据，这使得对于在重复访问相同数据的场景下，重复访问的次数越多、访问的数据量越大

##2.Spark构建原理
使用分层构建算法构建一个包含4个维度的Cube，第一轮MR任务从源数据聚合得到4维的Cuboid，也就是BaseCuboid，第二轮MR任务由4维的Cuboid聚合得到3维的Cuboid，以此类推，在经过N+1轮MR任务后，所有的Cuboid都被计算出来(0-D是怎么被计算出来的??)

![MapReduce构建四维cube](https://upload-images.jianshu.io/upload_images/9049859-b39ba683425f0f11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第N层N维的Cuboid可以看作一个RDD，那么一个有N个维度的Cube就会生成N+1个RDD，这些RDD是父子关系，N维的RDD由N-1维的RDD生成，并且父RDD是缓存在内存中的RDD.persist(StorageLevel)，这使得Spark构建会比MR构建更加高效，因为**进行MR构建的时候，父层数据是存储在磁盘上的**，为了最大化地利用Spark的内存，父RDD在生成子RDD后需要从内存中释放RDD.unpersist()，而且每一层的RDD都会通过Spark提供的API持久化保存到HDFS上。这样经过N+1层的迭代，就完成了所有Cuboid的计算。

![spark构建四维cube](https://upload-images.jianshu.io/upload_images/9049859-c0bf4541ce257fd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![spark构建cubeDAG](https://upload-images.jianshu.io/upload_images/9049859-0680c18468ebb983.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3.使用Spark构建Cube
#####配置Spark引擎
spark相关配置
```
kylin.engine.spark-conf.spark.master=yarn
kylin.engine.spark-conf.spark.submit.deployMode=cluster
kylin.engine.spark-conf.spark.dynamicAllocation.enabled=true
kylin.engine.spark-conf.spark.dynamicAllocation.minExecutors=1
kylin.engine.spark-conf.spark.dynamicAllocation.maxExecutors=1000
kylin.engine.spark-conf.spark.dynamicAllocation.executorIdleTimeout=300
kylin.engine.spark-conf.spark.yarn.queue=default
kylin.engine.spark-conf.spark.driver.memory=2G
kylin.engine.spark-conf.spark.executor.memory=4G
kylin.engine.spark-conf.spark.yarn.executor.memoryOverhead=1024
kylin.engine.spark-conf.spark.executor.cores=1
kylin.engine.spark-conf.spark.network.timeout=600
kylin.engine.spark-conf.spark.shuffle.service.enabled=true
#kylin.engine.spark-conf.spark.executor.instances=1
kylin.engine.spark-conf.spark.eventLog.enabled=true
kylin.engine.spark-conf.spark.hadoop.dfs.replication=2
kylin.engine.spark-conf.spark.hadoop.mapreduce.output.fileoutputformat.compress=true
kylin.engine.spark-conf.spark.hadoop.mapreduce.output.fileoutputformat.
compress.codec=org.apache.hadoop.io.compress.DefaultCodec
kylin.engine.spark-conf.spark.io.compression.codec=org.apache.spark.
io.SnappyCompressionCodec
kylin.engine.spark-conf.spark.eventLog.dir=hdfs\:///kylin/spark-history
kylin.engine.spark-conf.spark.history.fs.logDirectory=hdfs\:///kylin/spark-history
## uncomment for HDP
#kylin.engine.spark-conf.spark.driver.extraJavaOptions=-Dhdp.version=current
#kylin.engine.spark-conf.spark.yarn.am.extraJavaOptions=-Dhdp.version=current
#kylin.engine.spark-conf.spark.executor.extraJavaOptions=-Dhdp.version=current

```
#####开启Spark动态资源分配
配置Spark动态资源分配需要在Spark的配置文件中添加一些配置项以开启该服务。由于在Kylin中可以通过在kylin.properties中进行配置来直接覆盖Spark中的配置，因此只需要在kylin.properties中进行以下配置,具体配置的值可以根据实际情况进行调整。更多的配置项可以参考Spark相关文档
```
kylin.engine.spark-conf.spark.dynamicAllocation.enabled=true
kylin.engine.spark-conf.spark.executor.instances=0
kylin.engine.spark-conf.spark.dynamicAllocation.minExecutors=1
kylin.engine.spark-conf.spark.dynamicAllocation.maxExecutors=5
kylin.engine.spark-conf.spark.dynamicAllocation.initialExecutors=3
kylin.engine.spark-conf.spark.shuffle.service.enabled=true
````

#####出错处理和问题排查
如果在使用Spark构建引擎的时候遇到构建失败的情况，可以在“$KYLIN_HOEM/logs/kylin.long”文件里找到Kylin提交Spark任务时的完整命令(注意配置信息与一些参数)

```
“Convert Cuboid Data to HFile”
该错误是由于Spark任务在运行时使用的HBase相关的jar包不正确导致的，解决这个问题的方法是复制hbase-hadoop2-compat-*.jar和hbase-hadoop-compat-*.jar到
“$KYLIN_HOME/spark/jars”目录下，这两个jar包可以在HBase的lib目录下找到，然后重新提交运行失败的任务，这个任务最终应该会执行成功。
```

##4.使用Spark SQL创建中间平表
Kylin默认的数据源是Hive，构建Cube的第一步是创建Hive中间平表，也就是把事实表和维度表连接成一张平表，这个步骤是通过调用数据源的接口来完成的。以前使用Hive来执行构建平表的SQL，现在可以用Spark-SQL来执行。
1. 首先要确保以下配置已经存在于hive-site.xml中
```
<property>
 <name>hive.security.authorization.sqlstd.confwhitelist</name>
 <value>mapred.*|hive.*|mapreduce.*|spark.*</value>
</property>
<property>
 <name>hive.security.authorization.sqlstd.confwhitelist.append</name>
 <value>mapred.*|hive.*|mapreduce.*|spark.*</value>
</property>
```
2. 然后把Hive的执行引擎（hive.execution.engine）改成MapReduce
3. hive-site.xml复制到“$SPARK_HOME/conf”目录下，并且确保环境变量“HADOOP_CONF_DIR”已经设置完毕
4. 使用“sbin/start-thriftserver.sh--masterspark：//<sparkmaster-host>：<port>”命令来启动thriftserver，通常端口是7077
5. 通过修改“$KYLIN_HOME/conf/kylin.properties”设置参数
```
kylin.source.hive.enable-sparksql-for-table-ops=true
kylin.source.hive.sparksql-beeline-shell=/path/to/spark-client/bin/beeline
kylin.source.hive.sparksql-beeline-params=-n root -u
'jdbc:hive2://thriftserverip:thriftserverport'
```
6. 配置完成之后，重启Kylin使配置生效，这样Kylin在创建中间平表的时候就会使用Spark SQL来完成。如果想要关闭这个功能，只需要把“kylin.source.hive.enable-sparksql-for-table-ops”重新设为“false”就可以了。
