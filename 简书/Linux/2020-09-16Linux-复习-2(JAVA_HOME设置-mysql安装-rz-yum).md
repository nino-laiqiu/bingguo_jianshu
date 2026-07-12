###1.tail功能(监控)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c88c7904546b1153.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###2.压缩命令
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a1f61acc793f4b3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###3.JAVA_HOME配置
>上传linux版本java
>解压到指定的目录中,/opt/
>配置 vi /etc/profile(G 最后一行 o 下一行插入)
(例如设置)
>export     JAVA_HOME=/opt/java/jdk1.8.0_141
>export     PATH=$PATH:$JAVA_HOME/bin
>source /etc/profile
(在任意位置输入)
>在任意的位置输入 java -version(完成)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-1eb0437a3d60530b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###4.查找命令
![image.png](https://upload-images.jianshu.io/upload_images/9049859-4b6e60af87c60930.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-98d5b7da1f1ef553.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意一下)
###5.管道命令
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9a490f81545a5c81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


xshell连接虚拟机
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e01832d743f27205.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(有用 但是我的错误是网关和电脑的IP冲突了)

关于安装mysql遇到的一个问题
![image.png](https://upload-images.jianshu.io/upload_images/9049859-003403e0c205b220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于安装mysql的步骤
>https://blog.csdn.net/Bb15070047748/article/details/106245223?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param

Tomcat的安装和运行

Tomcat的下载与安装
cd /usr/local                                 进入/usr/local目录
mkdir tomcat                                  新建Tomcat目
opy安装包到tomcat目录，并解压
cp /usr/apache-tomcat-8.0.43.tar.gz /usr/local/tomcat     复制安装包到tomcat目录
tar -zxvf apache-tomcat-8.0.43.tar.gz                     解压安装包
cd /usr/local/tomcat/bin                                   进入bin目录
./startup.sh                                         执行startup.sh文件启动Tomcat
./shutdown.sh                                              关闭Tomcat服务

##重要的命令
###yum
![image.png](https://upload-images.jianshu.io/upload_images/9049859-77a51dbac4006b44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###rz
yum list | grep  rz  查找rz
yum -y install  	lrzsz.x86_64   安装rz
rz 上传文件到linux  ---位置是rz命令的执行处

###sz
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9e15ab878e8f6144.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###pstree
安装 
>yum -y install psmisc
执行
![image.png](https://upload-images.jianshu.io/upload_images/9049859-02ee983f527be04d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
