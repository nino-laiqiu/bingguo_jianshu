
**遇到的问题:**
机器故障不可避免,flink单机故障自动恢复能力不足
机器quota不均衡导致资源碎片
磁盘 内存 CPU等异常难以100%检测到
作业依赖的agent没有托管到yarn,发生故障仍然会调度

![异常节点的处理逻辑](https://upload-images.jianshu.io/upload_images/9049859-c38f5ef8ffcbe8c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**遇到的问题**
作业启动时间不稳定
作业启动时间靠经验和用户的反馈
作业要求实时性高,要求重启时间要尽可能短

![flink启动流程](https://upload-images.jianshu.io/upload_images/9049859-3ee8aa7ad352ee13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![慢节点机制](https://upload-images.jianshu.io/upload_images/9049859-7926b7864b21dd24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![mode](https://upload-images.jianshu.io/upload_images/9049859-4c9b011bb384319c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调度器
![image.png](https://upload-images.jianshu.io/upload_images/9049859-5de74d5c154955ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重启
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a755b8dd5b3815a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
