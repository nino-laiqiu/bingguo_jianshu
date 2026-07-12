#####1.namenode格式化注意事项
![image.png](https://upload-images.jianshu.io/upload_images/9049859-645f940547cff571.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.配置历史服务器

> vi mapred-site.xml

```
<!-- 历史服务器端地址 -->
	<property>
	<name>mapreduce.jobhistory.address</name>
	<value>linux03:10020</value>
	</property>
	<!-- 历史服务器web端地址 -->
	<property>
	    <name>mapreduce.jobhistory.webapp.address</name>
	    <value>linux03:19888</value>
	</property>
```

>mr-jobhistory-daemon.sh start historyserver  启动历史服务器

#####3.配置日志聚集

>  vi  yarn-site.xml

```
<!-- 日志聚集功能使能 -->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>
<!-- 日志保留时间设置7天 -->
<property>
<name>yarn.log-aggregation.retain-seconds</name>
<value>604800</value>
</property>
```

#####4.scp  -r  和 rsync - rvl
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7793477ff819e7d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####5.shell工具
![image.png](https://upload-images.jianshu.io/upload_images/9049859-37047791a2640622.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####6.yarn的启动
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3a42ca99d275d045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####7.时间同步




