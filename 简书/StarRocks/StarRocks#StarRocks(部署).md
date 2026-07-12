1. 安装之前先使用命令检查CPU是否支持，有信息输出则支持，没信息输出则不支持建议更换机器
2. 下载tar包,并重命名
3. 部署FE，修改配置文件，添加jvm参数，建议-Xmx参数设置到16G以上
4. 创建元数据目录
5. 分发，启动FE节点
6. 启动mysql客户端，访问FE，查看FE状况
7. 查看FE状态
```
mysql> SHOW PROC '/frontends'\G
```

8. 添加其他FE节点，角色也分为FOLLOWER,OBSERVER
9. 启动hadoop102,hadoop103 FE节点，第一次启动需指定--helper参数，后续再启动无需指定此参数
10. 全部启动完毕后，再使用mysql客户端查看FE的状况,alive全显示true则无问题
11. 部署BE，用户可以使用命令直接将BE添加到集群中，一般至少布置3个BE，每个BE实例添加步骤相同
```
mkdir -p storage
```
12. 使用mysql客户端添加hadoop101对应be节点
13. 添加完毕后，启动hadoop101 BE节点
```
start_be.sh --daemon
```
14. 查看BE状况,也是同样alive为true是正常运行
```
SHOW PROC '/backends'\G
```

15. 查看BE状况,也是同样alive为true是正常运行
16. 同样步骤在hadoop102,hadoop103部署BE，并添加节点
17. 部署Broker,此角色主要用于后续Broker load使用,启动安装目录的Broker服务，需要与hdfs交互
```
start_broker.sh --daemon
```
18. 使用mysql添加对应节点
```
mysql> ALTER SYSTEM ADD BROKER broker1 "hadoop101:8000";
mysql> ALTER SYSTEM ADD BROKER broker2 "hadoop102:8000";
mysql> ALTER SYSTEM ADD BROKER broker3 "hadoop103:8000";
```
19. 查看状态
```
SHOW PROC "/brokers"\G
```

[手动部署](https://docs.starrocks.com/zh-cn/main/quick_start/Deploy)
[扩容缩容](https://docs.starrocks.com/zh-cn/main/administration/Scale_up_down)
[数据流和控制流](https://docs.starrocks.com/zh-cn/main/quick_start/Data_flow_and_control_flow)
[FE开发环境搭建（拓展篇）](https://blog.csdn.net/ult_me/article/details/121582881?spm=1001.2014.3001.5501)
[StarRocks部署--集群部署](https://blog.csdn.net/ult_me/article/details/121716779?spm=1001.2014.3001.5501)



