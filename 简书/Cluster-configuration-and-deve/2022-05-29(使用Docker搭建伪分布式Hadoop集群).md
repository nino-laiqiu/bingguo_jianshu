#####1. 环境准备
1. 一台服务器,2C、4G、80G（足够了）
2. git [Linux安装Git-两种方式详细教程）](https://www.cnblogs.com/fuzongle/p/12830572.html)
3. 安装Docker [linux上安装Docker](https://cloud.tencent.com/developer/article/1605163),指定yum源
```
 sudo systemctl status docker
```


#####2. 拉取镜像
1. 把当前用户加入到Docker组中
2. 拉取镜像
3. 创建hadoop网络
4.  start-container.sh中开放9000,9090端口
docker network ls
5. 起docker,hadoop,查看是否起启来了
 docker ps
jps


#####3. 8088,50070无法访问的解决方案
一般的步骤:看一下防火墙的问题,如果是虚拟机看一下V8是不是关掉了,配置一下`/hdfs-site.xml ` 和` core-site.xml`,改一下hosts映射关系....
参考一下:
[阿里云服务器hadoop端口9000,50070访问不了的问题](https://blog.csdn.net/k393393/article/details/91869501?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-91869501-blog-81044923.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-91869501-blog-81044923.pc_relevant_paycolumn_v3&utm_relevant_index=1)
[无法访问问题解决汇总](https://blog.csdn.net/zxz547388910/article/details/86468925)

不知道是不是问题的关键,要设置一下内网

....我的问题不在这里,结果2小时的查问题,发现服务器要设置安全组....改一下,TCP通过端口,无语了.....

#####4. 其他问题
1. 问题:怎么关闭hadoop,几次修问题我都reboot....哈哈哈
2. 怎么升级hadoop版本,这个github都是2.7版本,要改一下镜像的
3. 内网,公网,我看datanode的网也不太一样,这是一个问题.....

#####5. 一些命令
复习一些shell和docker命令
1. 防火墙查看状态,关闭和开启 [linux 防火墙](https://blog.csdn.net/qq_38071435/article/details/82312562)
2. docker ps   / jps
3. vi 操作
4. netstat  [netstat命令](https://zhuanlan.zhihu.com/p/367635200)

[基于Docker搭建Hadoop集群之升级版 | 寒雁Talk (kiwenlau.com)](https://kiwenlau.com/2016/06/12/160612-hadoop-cluster-docker-update/)




