#####1. pull镜像
```
docker pull zookeeper
docker pull wurstmeister/kafka
```

#####2. 创建通信网络。zookeeper和kafka之间的通信
```
docker network create kafka_zk_net
```
```
查看网络
docker network ls
docker network inspect kafka_zk_net
```
#####3. 创建容器
```
docker run --net=kafka_zk_net --name zookeeper -p 21810:2181 -d docker.io/zookeeper
```
```
docker run --net=kafka_zk_net --name kafka -p 9093:9092 \
--link zookeeper \
-e KAFKA_BROKER_ID=4 \
-e KAFKA_ZOOKEEPER_CONNECT=172.18.0.2:2181 \
-e KAFKA_ADVERTISED_HOST_NAME=43.142.80.86 \
-e KAFKA_ADVERTISED_PORT=9092 \
-d wurstmeister/kafka
```
KAFKA_ADVERTISED_HOST_NAME参数需要设置为宿主机地址(或服务器外网地址)
KAFKA_ZOOKEEPER_CONNECT参数设置zookeeper容器内部地址和端口,查看指令为docker inspect kafka_zk_net
```
  "a3300202308bec58530baaa31247f1870def61f07208e72b18ab10f0bb44c5d9": {
                "Name": "zookeeper",
                "EndpointID": "ae59143a1679b7534df41b9a0aa44164c26a6841a04893a3053f14c73cbe2df9",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
```


#####4. 启动生产者和消费者
```
docker exec -it  kafka bash
cd opt/kafka_2.13-2.8.1/bin/

./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic topic1  生产者
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic1 --from-beginning  消费者
```

#####5. bug
**1. 启动consumer发现: Error while fetching metadata with correlation id**

修复步骤:
vi /opt/kafka/kafka/config/server.properties

添加:
```
listeners=PLAINTEXT://localhost:9092

advertised.listeners=PLAINTEXT://localhost:9092
```
[kafka listeners和advertised配置](https://www.cnblogs.com/xuliang666/p/11871389.html)

发现:Docker：bash: vi: command not found
```
apt-get update
apt-get install vim
```
停止kafka容器和重启
```
docker stop kafka
docker restart kafka
docker restart kafka  
```

![成功截图](https://upload-images.jianshu.io/upload_images/9049859-da01341a0ac7b09e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ps:如果你修改容器文件导致无法启动容器可以参考:
[Docker容器无法启动,里面的配置文件如何修改 ](https://zhuanlan.zhihu.com/p/159426055)

非内网连接,必须构建时指定端口和网址...测试了一下,修改是没用的,应该zookeeper已经存了元数据了..
```
    docker run --net=kafka_zk_net --name kafka -p 9093:9092 \
        --link zookeeper \
        -e KAFKA_BROKER_ID=4 \
        -e KAFKA_ZOOKEEPER_CONNECT=172.18.0.2:2181 \
        -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://43.142.80.86:9093 \
        -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
        -d wurstmeister/kafka*/
```

