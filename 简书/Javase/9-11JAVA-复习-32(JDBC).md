JDBC各个类的详解
1.DriverManager:驱动管理对象
功能:注册驱动
        Class.forName("com.mysql.jdbc.Driver")
原因:通过查看原码发现:在com.mysql.jdbc.Driver类中存在静态代码块
        注意mysql5之后的驱动jar包可以省略注册驱动的步骤
        获取数据库连接
       注意 :细节如果连接的是本机mysql服务器,并且默认端口是3306,则URL可以简写为:jdbc:mysql:///数据库
2.Connection:数据库连接对象
功能 获取执行SQL的对象(createstatement preparestatement)
        管理事务(开启 提交 回滚)
3.Statement:执行SQL的对象
(可执行
execute 可执行任意SQL
executeupdate DML DDL
excutequery DQL)
4.ResultSet:结果集对象
事务处理;
![image.png](https://upload-images.jianshu.io/upload_images/9049859-260e07d6f925440e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(在回滚事务时要判断connnection是否为nul)

数据库连接池
![image.png](https://upload-images.jianshu.io/upload_images/9049859-5f1203144911ae2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

JDBC Template
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b33dce28880f3094.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9cdfb30f95ce889f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意事项)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d411558d3a4640ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意一下与queryRunner的区别)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e5754a6f33521922.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意把javabean里的成员变量都设置为引用类型)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-eb7888e690abc97e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一道面试题
![image.png](https://upload-images.jianshu.io/upload_images/9049859-32b4c0f39f403331.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对于第一条  需要强制转换类型
第二条 相当于s1=short(s1 + 1)
