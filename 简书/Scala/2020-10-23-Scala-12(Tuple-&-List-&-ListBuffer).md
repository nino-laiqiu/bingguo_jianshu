#####1.元组的基本介绍
![image.png](https://upload-images.jianshu.io/upload_images/9049859-27991d725ae5a859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.元组的访问与遍历 
```
  var s1 = (1, 2, "hu", true)
    //访问方法1
    val value = s1._1
    //访问方法二
    val value1 = s1.productElement(0)
    //遍历
    for (index<-s1.productIterator){
         println(index)
```

#####3.数组和集合的比较
(1)数组的特点
a.数组本质上就是一段连续的内存空间，用于记录多个类型相同的数据；
b.数组一旦声明完毕，则内存空间固定不变；
c.插入和删除操作不方便，可能会移动大量的元素导致效率太低；
d.支持下标访问，可以实现随机访问；
e.数组中的元素可以是基本数据类型，也可以使用引用数据类型；

(2)集合的特点
a.内存空间可以不连续，数据类型可以不相同；
b.集合的内存空间可以动态地调整；
c.集合的插入删除操作可以不移动大量元素；
d.部分支持下标访问，部分不支持；
e.集合中的元素必须是引用数据类型；

#####4.List不可变集合
![追加数据](https://upload-images.jianshu.io/upload_images/9049859-b64848ce6d1ab8d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2d3a29a0e37b7416.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重要!!!(注意区别)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d2d9afc002115266.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-fb38e4f5aa7253f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(:::左右两边都要为集合)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-fe2d0dbf78d78cff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####5.ListBuffer 可变集合
```
    var s1 = ListBuffer(1, 2, 3, "i", true)
    var s11 = ListBuffer[Any](11,22)
    println(s1)
    //访问
    println(s1(1))//2  从一开始
    //添加1
    var s2 = s1+=12//改变了原来的集合
    println(s1)
    println(s2)
    //添加二
    s1:+3.......出错了........
    println(s1)
    //添加三
    s1.append(1)//
    println(s1)
    //添加集合到集合
    s11++=s1  //s11集合改变
    println(s1)
    println(s11)
    //删除元素 -=
    //删除下标一开始的两个元素
    s1.remove(1,2)
    println(s1) //包含1
    //从最后一个元素开始删除两个元素
    s1.trimEnd(2)
    println(s1)
    //删除下标元素
    s1.remove(1)
    println(s1) //从0开始
```


