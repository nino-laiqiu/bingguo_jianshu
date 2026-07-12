
##1.上传解压
rz命令
tar -zxf  文件名 -C 目录位置
压缩完成注意要删除hadoop中etc中的配置文件doc 可以加快速度

##2. 配置JDK文件 

>/opt/hadoop/hadoop-3.1.1/etc/hadoop/hadoop-env.sh 

>  export  JAVA_HOME=/opt/java/jdk1.8.0_261(最下一行加入)

##3.Hadoop的文件系统配置文件是hdfs-site.xml
>vi /opt/hadoop/hadoop-3.1.1/etc/hadoop/hdfs-site.xml 


  **(在<configuration>中间插入如下代码)**
  **(配置两个namenode)**

```
<!-- 集群的namenode的位置  datanode能通过这个地址注册-->
	<property>
	     <name>dfs.namenode.rpc-address</name>
		 <value>linux01:8020</value>
	</property>
	 <!-- namenode存储元数据的位置 -->
	<property>
	     <name>dfs.namenode.name.dir</name>
		 <value>/opt/hdpdata/name</value>
	</property>
	 <!-- datanode存储数据的位置 -->
	<property>
	     <name>dfs.datanode.data.dir</name>
		 <value>/opt/hdpdata/data</value>
	</property>
	 <!-- secondary namenode机器的位置-->
	<property>
	<name>dfs.namenode.secondary.http-address</name>
		<value>linux02:50090</value>
	</property>
```

##4.分发到集群的每台机器

>scp -r hadoop-3.1.1  linux02(这是虚拟机的名字):$PWD 

##5.初始化(在Hadoop的bin目录下)

> ./hadoop  namenode -format 

![image.png](https://upload-images.jianshu.io/upload_images/9049859-96553a96079a9b95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##6.启动
单节点的启动(单个启动namenode)

> ./hadoop-daemon.sh   start  namenode 

![image.png](https://upload-images.jianshu.io/upload_images/9049859-85ee8a1a7ec556e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

datanode的启动

>/opt/hadoop/hadoop-3.1.1/sbin/hadoop-daemon.sh start datanode

##6.1设置脚本启动所有节点(ssh无密配置要配置包含自身)

在Hadoop配置文件目录下添加域名映射信息

![image.png](https://upload-images.jianshu.io/upload_images/9049859-f5400a0c449ce309.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>vi workers  (只需要在namenode设置)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-87f587516559b7d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在启动和停止的脚本中添加如下配置(执行如下代码添加如下5行代码,如果有则减少一些)

>vi sbin/start-dfs.sh  

> vi sbin/stop-dfs.sh

![image.png](https://upload-images.jianshu.io/upload_images/9049859-159d9cf934e3e80f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root 

```

##6.2如果想水平扩展一台机器需要的步骤
1.设置ssh免密登录
2.删除namenode的目录文件信息

> hdpdata/data

3.域名映射配置(一般不设置 直接IP地址)

##7.配置系统环境变量

>vi /etc/profile

(在最下面添加如下)

```
export  JAVA_HOME=/opt/apps/jdk1.8.0_141
export  TOMCAT_HOME=/opt/apps/apache-tomcat-7.0.47
export  HADOOP_HOME=/opt/apps/hadoop-3.1.1
export  PATH=$PATH:$JAVA_HOME/bin:$TOMCAT_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
>source /etc/profile (启动服务)


##8.一键启动|停止

>start-dfs.sh  一键启动HDFS集群

>stop-dfs.sh  一键停止HDFS集群



##9.修改  etc/hadoop/core-site.xml  让默认操作的文件 系统是HDFS分布式文件系统

>vi  etc/hadoop/core-site.xml

```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://linux01:8020</value>
  </property>
```
(注意此处所有的机子都要设置为namenode的地址)
##yarn的安装

>vi /opt/hadoop/hadoop-3.1.1/etc/hadoop/yarn-site.xml

在configuration>内添加(可修改hostname)
```
<!--  resource,manager主节点所在机器 -->
<property>
<name>yarn.resourcemanager.hostname</name>
<value>linux01</value>
</property>

<!--  为mr程序提供shuffle服务 -->
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

<!--  一台NodeManager的总可用内存资源 -->
<property>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>4096</value>
</property>
<!--  一台NodeManager的总可用（逻辑）cpu核数 -->
<property>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>4</value>
</property>

<!--  是否检查容器的虚拟内存使用超标情况 -->
<property>
  <name>yarn.nodemanager.vmem-check-enabled</name>
  <value>false</value>
</property>

<!--  容器的虚拟内存使用上限：与物理内存的比率 --> 
<property>
  <name>yarn.nodemanager.vmem-pmem-ratio</name>
  <value>2.1</value>
</property>
```

分发yarn.site.xml(注意要在这个文件的目录分发 分发对象也要有这个路径和目录,文件)

>scp -r yarn.site.xml  linux04:$PWD

设置集群一键启动和停止(建立在works的基础上)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-454f25eccb538b9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(/etc/hadoop/works)

>vi  /opt/hadoop/hadoop-3.1.1/sbin/start-yarn.sh 
>vi /opt/hadoop/hadoop-3.1.1/sbin/stop-yarn.sh 
