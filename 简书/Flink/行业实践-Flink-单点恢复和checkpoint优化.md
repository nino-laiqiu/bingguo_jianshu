##1.单点恢复
flink中的checkpoint完成是指所有的flink中的task都完成才算checkpoint完成,所以随着任务数量的增多,checkpoint的完成度是指数型下降的
#####业务背景
![业务背景](https://upload-images.jianshu.io/upload_images/9049859-1e39aee5c9e2baf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####问题和解决思路
![解决思路](https://upload-images.jianshu.io/upload_images/9049859-0ed13e9bf36d1223.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####数据传输机制
![数据传输机制](https://upload-images.jianshu.io/upload_images/9049859-138a61521e4dd39b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####难点
![难点](https://upload-images.jianshu.io/upload_images/9049859-dd838a7d4d8466d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####解决方案
![解决方案](https://upload-images.jianshu.io/upload_images/9049859-eea6492fa61454a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![解决方案](https://upload-images.jianshu.io/upload_images/9049859-530bba3de6bf8c2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![解决方案](https://upload-images.jianshu.io/upload_images/9049859-a2fab25d38418583.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![解决方案](https://upload-images.jianshu.io/upload_images/9049859-196d755814512941.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####业务收益
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0bebd66fc4617985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.checkpoint优化
#####业务背景
![业务背景](https://upload-images.jianshu.io/upload_images/9049859-88eb001d088aa7e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(两阶段提交)
#####问题和解决
![问题和解决](https://upload-images.jianshu.io/upload_images/9049859-639bc93f68b5be91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####checkpoint机制
![checkpoint机制](https://upload-images.jianshu.io/upload_images/9049859-0576ec1b924f10cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![checkpoint机制](https://upload-images.jianshu.io/upload_images/9049859-06fb46201d07ca29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####难点
![难点一](https://upload-images.jianshu.io/upload_images/9049859-3a81b3e24a26f58c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![解决方案](https://upload-images.jianshu.io/upload_images/9049859-1bd5fd74be10ac71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![解决方案](https://upload-images.jianshu.io/upload_images/9049859-393f032e5046d1d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![难点二](https://upload-images.jianshu.io/upload_images/9049859-a0a784b01473fa5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![解决方案](https://upload-images.jianshu.io/upload_images/9049859-156fb2a11948d00e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![解决方案](https://upload-images.jianshu.io/upload_images/9049859-56f58b2a46703fbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####业务收益
![业务收益](https://upload-images.jianshu.io/upload_images/9049859-7543a03f9e70feda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3.其他优化
![其他优化](https://upload-images.jianshu.io/upload_images/9049859-33b080abe9524434.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
