##配置Apache Kylin
#####配置文件分类
在Apache Kylin安装目录的“conf/”目录中包含了对ApacheKylin进行配置的所有配置文件。读者可以修改配置文件中的参数，以达到环境适配、性能调优等目的。
1. kylin.properties
该文件是Apache Kylin服务使用的全局配置文件，和ApacheKylin有关的配置项都在此文件中。
2. kylin_hive_conf.xml
该文件包含了Apache Hive任务的配置项。在构建Cube的第一步通过Hive生成中间表时，会根据该文件的设置调整Hive的配置参数。
3. kylin_job_conf_inmem.xml
该文件包含了MapReduce任务的配置项。当Cube构建算法是FastCubing时，会根据该文件的设置来调整构建任务中的MapReduce参数。
4. kylin_job_conf.xml
该文件包含了MapReduce任务的配置项。当kylin_job_conf_inmem.xml不存在，或者Cube构建算法是LayerCubing时，用来调整构建任务中的MapReduce参数
#####基本配置参数
kylin.properties在Apache Kylin的配置文件中占有重要位置
```
kylin.metadata.url
kylin.hdfs.working.dir
kylin.server.mode
kylin.source.hive.database-for-flat-table
kylin.storage.hbase.compression-codec
kylin.security.profile
kylin.web.timezone
kylin.source.hive.client
kylin.storage.hbase.coprocessor-local-jar
kylin.engine.mr.job-jar
kylin.env
```
##多级配置重写
$KYLIN_HOME/conf/下的部分配置项可以在Web UI中重写。配置重写有两个作用域，分别是项目级别和Cube级别。项目级别的配置继承于全局配置文件；Cube级别的配置继承于项目级别配置文件。配置的覆盖优先级关系是：Cube级别配置项>项目级别配置项>配置文件。

能被配置重写的配置文件主要是$KYLIN_HOME/conf/文件夹下的kylin.properties、kylin_hive_conf.xml、kylin_job_conf.xml和kylin_job_conf_inmem.xml。
#####项目级别配置重写
在Web UI界面点击“Manage Project”命令，选中某个项目，点击“Edit”→“Project Config”→“+Property”按钮，进行项目级别的配置重写
![项目级别的配置重写](https://upload-images.jianshu.io/upload_images/9049859-9a9f443214cebf12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####Cube级别配置重写
在设计Cube（Cube Designer的Configuration Overwrites的这一步骤可以添加配置项，进行Cube级别的配置重写
![Cube级别配置重写](https://upload-images.jianshu.io/upload_images/9049859-f6b12f160b9a7882.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以下参数可以在Cube级别重写：
```
·kylin.cube.size-estimate*
·kylin.cube.algorithm*
·kylin.cube.aggrgroup*
·kylin.metadata.dimension-encoding-max-length
·kylin.cube.max-building-segments
·kylin.cube.is-automerge-enabled·kylin.job.allow-empty-segment
·kylin.job.sampling-percentage
·kylin.source.hive.redistribute-flat-table
·kylin.engine.spark*
·kylin.query.skip-empty-segments
```
#####MapReduce任务配置重写
Kylin支持在项目和Cube级别重写kylin_job_conf.xml和kylin_job_conf_inmem.xml中的参数，以键值对的性质，按照如下格式替换
```
kylin.engine.mr.config-override.<key> = <value>
```
如果用户希望Cube的构建任务使用不同的YARN resource queue，可以进行如下设置
```
kylin.engine.mr.config-override.mapreduce.job.queuename={queueName}
```
#####Hive任务配置重写
Kylin支持在项目和Cube级别重写kylin_hive_conf.xml中的参数，以键值对的性质，按照如下格式替换
```
kylin.source.hive.config-override.<key> = <value>
```
如果用户希望Hive使用不同的YARN resource queue，可以进行如下设置：
```
kylin.source.hive.config-override.mapreduce.job.queuename={queueName}
```
#####Spark任务配置重写
Kylin支持在项目和Cube级别重写kylin.properties中的“Spark”参数，以键值对的性质，按照如下格式替换：
```
kylin.engine.spark-conf.<key> = <value>
```
如果用户希望Spark使用不同的YARN resource queue，可以进行如下设置：
```
kylin.engine.spark-conf.spark.yarn.queue={queueName}
```
##常见问题与解决方案
#####在Fast Cubing构建Cube时，Mapper抛出“OutOfMemoryException”异常。
造成这种问题的原因，一方面有可能是Mapper的内存设置过少，可以调整kylin_job_conf.xml或kylin_job_conf_inmem.xml中对Mapper内存的设置，参数是“mapreduce.map.memory.mb”和“mapreduce.map.java.opts”。另一方面有可能是一个Mapper处理了过多数据，可以减小kylin_hive_conf.xml中的“dfs.block.size”参数，使Hive输出更多小文件，从而增加Mapper数量，并减少单个Mapper处理的数据量。
#####如何在Cube中增加维度和度量？
一旦Cube被构建完成，它的结构将不能被修改。用户可以通过克隆该Cube，在新的Cube中增加维度和度量，并在新的Cube构建完成后，禁用或删除原来的Cube，使得查询能够使用新的Cube进行回答。如果用户能接受新的维度在历史数据中不存在，则可以在原来的Cube的构建结束后开始构建新的Cube，并基于原来的Cube和新的Cube创建一个混合（Hybrid）模型。
#####构建Cube时报错“Too high cardinality is notsuitable for dictionary”。
Kylin使用字典编码方式来编码/解码维度的值；通常一个维度的基数都是小于百万的，所以字典编码方式非常好用。正如字典需要被持久化、加载进内存，如果一个维度的基数非常高，内存占用会非常大，所以Kylin加了一层检查。如果用户看到这类报错，建议先找到哪些维度是超高基维度，并重新对Cube进行设计(如：是否需要将超高基的列设置为维度)，如果用户必须保留该超高基列作为一个维度，有如下解决方法：
1. 修改编码方式，如修改为“fixed_length”或“integer”。
2. 增大“$KYLIN_HOME/conf/kylin.properties”中的“kylin.dictionary.max.cardinality”的值。
#####当某列中的数据都大于0时，查询SUM()返回负值。
如果在Hive中将列声明为integer，则SQL引擎(Calcite)将使用列的数据类型作为“SUM()”的数据类型，而此字段上的聚合值可能超出整数范围。在这种情况下，查询的返回结果可能是负值。解决方法如下：
在Hive中将该列的数据类型更改为BIGINT，然后将表同步到Kylin（对应的Cube不需要刷新）。
#####HDFS上的工作目录中文件超过了300G，可以手动删除吗？
HDFS上的工作目录中的数据包括了中间数据（将被垃圾清理所清除）和Cuboid数据（不会被垃圾清理所清除），Cuboid数据将为之后的Segment合并而保留。所以如果用户确认这些Segment在之后不会被合并，可以将Cuboid数据移动到其他路径甚至删除。另外，请注意：HDFS工作目录下的“resources”子目录中会存放一些大的元数据，如字典文件和维表的快照，这些文件不能被删除
