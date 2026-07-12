Hive具有一个可选的组件叫做HiveServer或者HiveThrift,其允许通过指定端口访问Hive. Thrift是一个软件框架,其用于跨语言的服务开发。访问Hive的最常用的方式就是通过CLI进行访问。不过, CLI的设计使其不便于通过编程的方式进行访问。CLI是胖客户端,其需要本地具有所有的Hive组件,包括配置,同时还需要一个Hadoop客户端及其配置。同时,其可作为HDFS客户端、MapReduce客户端以及JDBC客户端(用于访问元数据库)进行使用。即使客户端安装是成功的,但是配置所有的网络访问还是会比较困难的,特别是对于不同网段或跨数据中心的情况。

##1. 启动Thrift Server
后台启动 `bin/hive --service hiveserver &`
检查HiveServer是否启动成功的最快捷方法就是使用netstat命令查看10, 000端口是否打开并监听连接`netstat -nl I grep 10000`

##2. 获取执行计划
##3. 元数据存储方法
