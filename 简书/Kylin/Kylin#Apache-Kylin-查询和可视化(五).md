##1. Web GUI
#####查询
Insight页面提供一个SQL输入框，点击提交即可查询结果
#####显示结果
对于查询，默认会以表格的形式显示结果，如果需要以图表形式显示，可以点击表格右上角的“Visualization”按钮进行切换
若以图形化显示结果，前端图形化支持折线图（Line）、柱状图（Bar）、饼图（Pie）三种类型

##2. REST API
#####查询认证
Kylin查询请求对应的URL为“http://<hostname>:<port>/kylin/api/query”，HTTP的请求方式为POST。
#####查询请求参数
查询API的Body部分要求发送一个JSON对象，下面对请求对象的各个属性逐一进行说明。
1. sql：必填，字符串类型，请求的SQL。
2. offset：可选，整型，查询默认从第一行返回结果，可以设置该参数决定返回数据从哪一行开始往后返回。
3. limit：可选，整型，加上limit参数后会从offset开始返回对应的行数，返回的数据行数小于limit，以实际行数为准。
4. project：必填，字符串类型，设置为自己要查询的项目。
![示例](https://upload-images.jianshu.io/upload_images/9049859-99a1e140cb7443b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####查询返回结果
查询结果返回的也是一个JSON对象，下面给出返回对象中每一个属性的解释。
1. columnMetas：每个列的元数据信息。
2. results：返回的结果集。
3. cube：这个查询对应使用的CUBE。
4. affectedRowCount：这个查询关系到总行数。
5. isException：这个查询返回是否异常。
6. exceptionMessage：如果查询返回异常，则给出对应的内容。
7. duration：查询消耗时间，单位为毫秒。
8. totalScanCount：Scan的总行数。
9. totalScanBytes：Scan的总字节数。
10. hitExceptionCache：是否击中异常缓存。
11. storageCacheUsed：是否使用存储缓存。
12. traceUrl：跟踪的URL。
13. pushDown：是否使用查询下压。·partial：这个查询结果是否为部分结果，这取决于请求参数中的“acceptPartial”为“true”还是“false”。
![示例](https://upload-images.jianshu.io/upload_images/9049859-b3ef8eaf5c6010d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3. ODBC
##4. JDBC
#####获得驱动包
在默认发布的二进制包中，对应LIB目录下有名称为kylin-jdbc-{version}-SNAPSHOT.jar的jar包，这就是Apache Kylin的JDBC驱动包。
#####认证
创建JDBC连接时，有“user”“password”“ssl”三个属性需要填写，下面分别对每个属性进行说明：
1. user：Kylin用户的名称。
2. password：Kylin用户的密码。
3. ssl：其值默认为false，如果为true，对其的所有访问都将基于HTTPS。
#####URL格式
JDBC访问Kylin对应的URL格式为“jdbc：kylin：//<hostname>：<port>/<kylin_project_name>”。URL中需要填写端口信息，如果JDBC连接属性对应的“ssl”设置为“true”，那端口对应Kylin服务器的HTTPS端口一般为“443”；此外，Apache Kylin的缺省HTTP服务端口是“7070”；
#####获取元数据信息
Kylin JDBC驱动支持获取元数据信息，我们可以基于SQL的一些过滤表达式（如“％”）列出Catalog、Schema、表和列信息

##5. Tableau集成
Tableau支持两种连接方式，分别为“Live”和“Extract”。“Extract”模式会把全部数据加载到系统内存，查询的时候直接从内存中获取数据，是非常不适合大数据处理的一种方式，因为大数据无法全部驻留在内存中。“Live”模式会实时发送请求到服务器进行查询，配合Apache Kylin亚秒级的查询速度，能很好地实现交互式的大数据可视化分析。
##6. Zeppelin集成
##7. Superset集成
##8. QlikView集成
##9. Redash集成
##10. MicroStrategy集成
