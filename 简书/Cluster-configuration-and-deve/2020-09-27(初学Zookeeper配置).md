#####1.上传压缩

>/opt/zookeeper/zookeeper-3.4.6

##2.修改配置文件
```
(zkData)在ZOOKEEPER_HOME目录下创建
>cd /opt/zookeeper/zookeeper-3.4.6/conf
>mv  zoo_sample.cfg   zoo.cfg
>vi zoo.cfg
```
dataDir=/opt/zookeeper/zookeeper-3.4.6/zkData
server.3=linux03:2888:3888
server.4=linux04:2888:3888
server.5=linux05:2888:3888

#####3.创建zkData目录 并写入myid文件

>mkdir /opt/zookeeper/zookeeper-3.4.6/zkData

>echo 3 >  /opt/zookeeper/zookeeper-3.4.6/zkData/myid

4. 2.4 分发

scp -r zookeeper-3.4.6/  linux04:$PWD

scp -r zookeeper-3.4.6/  linux05:$PWD

##5. 修改linux03 linux04 机器的myid的值

>echo 4 >  /opt/zookeeper/zookeeper-3.4.6/zkData/myid

>echo 5>  /opt/zookeeper/zookeeper-3.4.6/zkData/myid

##6.可以不配置环境,没有一键启动功能,写脚本启动

> /opt/zookeeper/zookeeper-3.4.6/bin/zkServer.sh   start 

(每个机器都要启动)
否则报错
Error contacting service. It is probably not running  

(查看集群状态)
>/opt/zookeeper/zookeeper-3.4.6/bin/bin/zkServer.sh status 

#####登录客户端

>bin/zkCli.sh    
>bin/zkCli.sh    -server  linux03:2181
>bin/zkCli.sh    -server  linux03:2181,linux04:2181,linux05:2181

##集群启动脚本
```
do
ssh linux0${i} "source /etc/profile;/opt/zookeeper/zookeeper-3.4.6/bin/zkServer.sh $1"
done
 
sleep 2
 
if [ $1 == start ]
then
for i in {3..5}
do
ssh linux0${i} "source /etc/profile;/opt/zookeeper/zookeeper-3.4.6/bin/zkServer.sh status "
done
fi
```
> sh  zk.sh start








