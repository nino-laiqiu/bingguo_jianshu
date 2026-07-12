1.DOM
![image.png](https://upload-images.jianshu.io/upload_images/9049859-072d989c195ca192.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2430d271bd31b436.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-028ad07bc57503c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.事件的简单学习
(耦合度高)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ae10ea5784e0a90d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-348775dd4af19307.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
电灯开关
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0e1eb39cf98800fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.BOM
(5个对象)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-156e967f187e9a33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
概念
![image.png](https://upload-images.jianshu.io/upload_images/9049859-45c44caafc6ce9bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###window
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c93387eb6f95a963.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-4ff6836b36aa91ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
定时器(一次定时器 循环定时器)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-f4721379f9f6f483.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-90bfb63c3c45e00c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-8660e2748ad3f99a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
案例:轮播图
![image.png](https://upload-images.jianshu.io/upload_images/9049859-5dceb3717ab14a7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-1a56994d8e49ae1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(document的来源自window)

###Lacation
点击事件
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c2ab95669f7fc618.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<!--设置表单标签-->
<input type="button" id="btn1" value="刷新">
<input type="button" id="btn" value="某个网址">
<script>
    //获取按钮
    var btn1 = document.getElementById("btn1");
    //获取按钮
    var btn = document.getElementById("btn");
    //执行刷新
    btn1.onclick = function () {
        location.reload();
    };
    //获取href 返回完整的URL
    let href = location.href;
    alert(href);

    //跳转某个网址
    btn.onclick=function () {
           location.href= "https://www.bilibili.com";
    }
</script>
</body>
</html>
```
概述
![image.png](https://upload-images.jianshu.io/upload_images/9049859-064c4c78189c66b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




