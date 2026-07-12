##1.  学习计划
1. 先过一遍尚硅谷的flink基础视频
2. 过一遍flink数据
3. 过一遍涛哥视频和项目
4. 过一遍flink优化
##2.  每次学习状况
1. 书籍第一章 20220317
2. 简单操作了一下flink 20220322
3. datastream API简单学习 20220403
4. 学习Source 20220404
5. 学习Sink 20220515
6. 学习时间语义和窗口 20220521
7. 学习flink-join 20220525
8. 学习flink-state 20220527
9. 学习精准一次性语义 20220608
10. 学习flink-table 20220611
11. 学习flink-table 20220622

##3. 遇到的问题
水位线问题,如果允许延迟数据到达,但是某个数据延迟非常慢,水位线已经高于这个数据的时间了,则这个数据会丢失吗,如果丢失应该怎么处理,如下的说明
![数据延迟,窗口触发的条件](https://upload-images.jianshu.io/upload_images/9049859-5957302bd3a3041c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

处理迟到数据
双流(基于时间 基于窗口)
自定义窗口函数(分配器  触发器 移除器)
第三章的高可用设置
定义处理槽共享组(共享槽的实际情况)
关于富函数的应用
关于Source和Sink在测试案例中的使用


问题和概念:
[flink taskmanager&slots&并行度&任务链&task分配详解](https://www.cnblogs.com/gentlescholar/p/15044085.html)
task就是spark中的taskset


![概念对比](https://upload-images.jianshu.io/upload_images/9049859-bc128bbc5c13f01d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最近看视频学习,flink中的task和subtask和spark中的概念搞混了,所以摘取一下,时常回顾方式搞混了
