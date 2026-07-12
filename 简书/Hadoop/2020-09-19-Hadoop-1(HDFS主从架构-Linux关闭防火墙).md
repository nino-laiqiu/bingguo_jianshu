##1.基本框架

##2.HDFS(Hadoop distribute flie system)
(1).数据存储在集群的位置
(2).不知道数据存储的位置.人为的失误----在机器中记录数据存储的位置(元数据)!!!!
     怎么找 优化 格式问题
(3).假设集群中某个节点宕机----数据存储副本(规定为存储三个副本在不同机器上)
(4).HDFS要储存的数据块过大不大容易存储,考虑网络不稳定等----以128M的数据块分布存储不同的物理硬盘中
(5).给用户提供一个虚拟目录(方便用户查找)

##3.HDFS主从架构
在HDFS中的文件不能随机修改内容,一次存储多次读取
集群中有BP-POLL-BLOOK文件夹用来存储数据块
namenode(主)
>记录元数据
>接收客户端请求,返回元数据给客户端,分配给Datanode任务
>为用户提供一个虚拟目录
>管理集群中的所有datanode

datanode(存储真实的数据,出问题向主节点汇报)---分布数据
>注册和汇报,获取一个唯一的标识,识别集群版本
>汇报自己的存储能力
>汇报自己储存数据块的信息,储存用户的数据块
![image.png](https://upload-images.jianshu.io/upload_images/9049859-efd49df5de175799.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##4.MapReduce工作原理
Map
Reduce
MapReduce

##5.Linux关闭防火墙
1:查看防火状态
>systemctl status firewalld

>service  iptables status

2.暂时关闭防火墙
>systemctl stop firewalld

>service  iptables stop

3.永久关闭防火墙
>systemctl disable firewalld

>chkconfig iptables off

4.重启防火墙

>systemctl enable firewalld

>service iptables restart  




