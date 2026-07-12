1.接口是不能实例化的

![image.png](https://upload-images.jianshu.io/upload_images/9049859-ed4a3d42fe5c51b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0e6266bad1596d76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.为什么抽象类不能实例化却有构造方法

实例化子类对象时首先调用的是抽象类中的构造函数再调用子类中的。

>https://blog.csdn.net/qq_35385687/article/details/90107424?biz_id=102&utm_term=%E6%8A%BD%E8%B1%A1%E5%92%8C%E6%8E%A5%E5%8F%A3&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-90107424&spm=1018.2118.3001.4187


#####3.java对象概述
>继承和实现的区别

单继承多实现


 >  抽象和接口的区别

抽象类是对一组具有相同属性和方法的逻辑上有关系的事物的一种抽象，而接口则是对一组具有相同属性和方法的逻辑上不相关的事物的一种抽象。


>   实例化和初始化的区别

初始化一次加载 实例化 new对象 
问题是子类继承父类,new子类对象  父类初始化

##4.关于BufferedReader/ Writer的读取效率
BufferedReader缓冲区隐藏在Reader对象里面，Reader没有自己的缓冲区，一般用多少读多少不缓存，这样就慢了，如果想要缓存可以自行维护

##5.迭代器原理
迭代器只是一个指针不是容器

java.util.NoSuchElementException原因分析以及解决方法
```
while (iterator.hasNext()) {
       // System.out.println(iterator.next().getName()+iterator.next().getSalary());
        HDFSUNIL.people people1 = iterator.next();
        System.out.println(people1.getName()+people1.getSalary());
    }
```
使用了两次next(),第二次出现了空值

## 6.Iterable  Iterator 的区别

Iterator是迭代器类，而Iterable是接口。 
好多类都实现了Iterable接口，这样对象就可以调用iterator()方法。 
