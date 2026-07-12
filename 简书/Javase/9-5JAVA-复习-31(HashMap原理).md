一道面试题:哈希表底层算法
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b8a4719270182454.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
面试题(比较equals比较的是key值,hashcode值相同则比较key的内容......这个面试题非常重要)
**允许hashcode相同的值存在,放在链表或者红黑树中,但是不允许key重复情况,符合事实..**
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7b4d1899e71a0c48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c224b43659002811.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图解
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b2a83dccd02c2a9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意一点
![image.png](https://upload-images.jianshu.io/upload_images/9049859-eb8b5bb5cf00f14e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##为什么HashMap数组扩容长度是2 的n次方幂?
![问题](https://upload-images.jianshu.io/upload_images/9049859-8055cf2114b96a83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![例子](https://upload-images.jianshu.io/upload_images/9049859-c6214da6ae1c1244.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![例子二](https://upload-images.jianshu.io/upload_images/9049859-aef20a665d5c5280.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建集合是指定容量不为2的n次幂
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9e0f58339e3f6b7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-22f13bc27e6f5cbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

加载因子
![image.png](https://upload-images.jianshu.io/upload_images/9049859-15a582f34ebfc22d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-6c985ff4d36287a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一道面试题
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0ba09c1b0494d633.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**注意一下(转换为红黑树的条件)**
![image.png](https://upload-images.jianshu.io/upload_images/9049859-bb4186215fe549ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


链表的连接
![image.png](https://upload-images.jianshu.io/upload_images/9049859-da4f1f2d5cafb38c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

扩容机制
![image.png](https://upload-images.jianshu.io/upload_images/9049859-8e90948eaddb2588.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-be46162a1b845695.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

HashMap的四种遍历方式
1.通过 keyset values方法
2.通过set的迭代器功能,迭代器可以换作for循环(此处有四种不同的写法)
3.通过get(key),此方法不推荐使用,两次调用了迭代器
5.通过map接口默认方法
![image.png](https://upload-images.jianshu.io/upload_images/9049859-62b045eaeeca3be4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

HashMap的初始化问题描述
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e262ccf590b272e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

问题:
扩容的性能问题与初始容量的设置
数组的索引和hash冲突
底层红黑树与链表的转化条件
散列表是懒加载机制,在第一次put()方法添加元素时创建
寻址算法

<复习一下>hashmap动力原点视频
![image.png](https://upload-images.jianshu.io/upload_images/9049859-637c052c8a3f3f9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-78c9b365e78898c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3a1168c5667b8a49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-47cdfbe4742de2cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(同一个链表的元素的hash值是相同的,而key的equals比较的必然不相同)
(注意一下不重写的后果)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2d680543b7b6cafe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
