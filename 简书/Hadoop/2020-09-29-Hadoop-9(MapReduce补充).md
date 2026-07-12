######数据压缩

![image.png](https://upload-images.jianshu.io/upload_images/9049859-707a70730c3f45c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.MapReduce跑得慢的原因

![image.png](https://upload-images.jianshu.io/upload_images/9049859-6f9abd45d7feb518.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.优化
![image.png](https://upload-images.jianshu.io/upload_images/9049859-943e99194a5ccaad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-75e56969aa29dbfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.shuffle机制

![image.png](https://upload-images.jianshu.io/upload_images/9049859-b9348bcb1971f56a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.hadoop集群启动了哪些进程

![image.png](https://upload-images.jianshu.io/upload_images/9049859-74a5ffca35d7f86d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.hadoop的配置文件及作用

core.site.xml:fs.  defaultFS:hdfs://cluster1(域名) 默认的hdfs路径
hadoop.en.sh:  设置jdk路径
hdfs.site.xml:  设置备份文件块数 节点目录 本地系统路径
mapred.site.xml:   yarn指定运行在yarn上

6.hadoop的几个默认端口
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3b9c8b83c4d60318.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7.MapReduce术语
read---map---collect---溢出---combine---   (map)
copy---merge---sort---reduce---   (reduce)

8.常见算法
单词计数
数据去重
排序
Top K
选择
投影
分组
多表连接(没练习过......)
单表关联
