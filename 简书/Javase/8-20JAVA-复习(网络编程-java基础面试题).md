1.JDK/JRE/JVM 三者之间的联系与区别
JDK：开发者提供的开发工具箱，是给程序开发者用的。它包括完整的JRE(Java Runtime Environment)，Java运行环境，还包含了其他供开发者使用的工具包。
JRE：(Java Runtime Environment) JVM运行时所须的包依赖的环境都在JRE中。
JVM：当我们运行一个程序时，JVM负责将字节码转换为特定机器代码，JVM提供了内存管理、垃圾回收和安全机制等。这种独立于硬件和操作系统，正是Java程序可以一次编写多处执行的原因。
JDK>JRE>JVM

2.成员变量与局部变量的区别
从语法形式上看：
成员变量是属于类的，而局部变量是在方法中定义的变量或是方法的参数；成员变量可以被public、private、static等修饰符所修饰，而局部变量不能被访问控制修饰符及static所修饰。但是，成员变量和局部变量都能被final所修饰。
从变量在内存中的存储方式来看：
如果成员变量是使用的 static修饰的，那么这个成员变量是属于类的，如果没有使用 static修饰，这个成员变量是属于实例(对象)的。而对象存在与堆内存中、局部变量则存在于栈内存中。从变量在内存中的生存时间上看：
成员变量是对象的一部分。它随着对象的创建而存在，而局部变量随着方法的调用而自动消失。成员变量如果没有被赋初值，则会自动以类型的默认值而赋值（一种情况例外：被final修饰的成员变量也必须显示的赋值），而局部变量则不会自动赋值。

3.父子关系的构造方法的执行顺序
父类有无参构造方法，子类才可以写无参构造方法；父类有含参构造方法，子类才可以写含参构造方法。
构造方法不能被继承、重写。
当进行无参构造时，先调用父类无参构造方法，然后调用子类无参构造方法；当进行有参构造时，先调用父类有参构造，然后调用子类有参构造。

4.为什么在静态成员方法中不能使用super(类 和 对象)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e0d7c03516aab897.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.String和StringBuffer、StringBuidler
每次对String类型进行改变的时候，都会生成一个新的String对象，然后将地址引用指向新的String对象。
操作少量的数据：String
单线程在字符串缓冲区下操作大量数据：StringBuidler
多线程在字符串缓冲区下操作大量数据：StringBuffer

6.自动装箱与拆箱
 装箱：将基本类型用它们对应的引用类型包装起来；
 拆箱：将包装类型转换为对应的基本数据类型；

7.HashCode、equals与==
hashcode()的作用是获取哈希码
hashCode() 所使用的杂凑算法也许刚好会让多个对象传回相同的杂凑值。越糟糕的杂凑算法越容易碰撞，但这也与数据值域分布的特性有关（所谓碰撞也就是指的是不同的对象得到相同的hashCode）。在HashSet中，在作对比的时候，同样的hashCode有多个对象，它会使用equals()来判断是否真的相同。也就是说hashCode只是用来缩小查找成本的。
### ==与euqals
**==：**它的作用是判断两个对象的地址是不是相等。即：判断两个对象是不是同一个对象。（基本数据类型"=="比较的是值，引用数据类型"=="比较的是内存地址）
**equals()：**它的作用也是判断两个对象是否相等。但它一般有两种使用情况：
类没有重写equals()方法。则通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。类重写了equals()方法。一般，我们都通过重写equals()方法来比较两个对象的内容是否相等；如果内容相等，则返回true（即，认为这两个对象相等）
String中的equals方法是被重写过的，因为Object类的equals方法是比较的对象的内存地址，而String的equals方法比较的是对象的值。
	当创建String类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重写创建一个String对象。

8.关于final关键字的一些总结
final关键字主要用在三个地方：变量、方法、类。
对于一个final变量：
如果是基本数据类型的变量，则其数组一旦初始化以后就不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
当用final修饰一个类时，表面这个类不能被继承。final类中的所有成员方法都会被隐式的指定为final方法。使用final修饰方法的原因：
把方法锁定，以防任何继承类修改它的含义效率：在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用final方法进行这些优化了）。类中所有的private方法都隐式地指定为final。

9.值传递与引用传递
![image.png](https://upload-images.jianshu.io/upload_images/9049859-aa1d622dfa420c34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-580602e8548a624e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###摘自:
>链接：https://juejin.im/post/6844904030687215629
>链接：https://juejin.im/post/6844903971144859661

网络编程
udp
![image.png](https://upload-images.jianshu.io/upload_images/9049859-956a8cbc6cf61c9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






