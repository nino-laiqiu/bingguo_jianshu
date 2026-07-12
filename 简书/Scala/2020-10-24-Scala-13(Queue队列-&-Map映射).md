##1.Queue
创建 增删改查
```
    var s1 = new mutable.Queue[Any]()
    //添加元素
    s1 += 1 //加入元素
    s1 ++= List(1, 2, 3,4) //打散加入 队列往后加入
    //s1 += List(9, 90) //加入一个list集合
    println(s1)
    //取出元素,本身会变,头元素
    val value = s1.dequeue()
    println(value + "\t" + s1)//1	Queue(1, 2, 3, 4)
    //入队尾
    val value1 = s1.enqueue(22)//Queue(1, 2, 3, 4, 22)	Queue(1, 2, 3, 4, 22)
    println(value1 + "\t" + s1)
    //取出元素
    println(s1.head)//1
    println(s1.last)//最后一个元素22
    println(s1.tail.tail)//队尾元素  Queue(3, 4, 22)
  }
}
```
##2.Map
对于map集合默认是不可变的,要声明包,对于不可变的集合是有序的
#####map的创建 & map的取值
```
//默认的不可变集合
    var s1 = Map(1 -> 11, 2 -> 22, 3 -> 33)
    //可变集合
    var s2 = mutable.Map("1" -> 1, "helloworld" -> "你好")
    //创建一个空的map
    var s3 = new mutable.HashMap[String, String]
    //对偶元组
    var s4 = mutable.Map((1, 2), (2, 3))
    println(s1)
    println(s2)
    println(s3)
    println(s4)

    //取值的四种方法
    var s5 = mutable.Map("1" -> 1, "helloworld" -> "你好")
    //1map(key).如果不存在会报错
    println(s5("1"))
    //2判断方法,contains
    if(s5.contains("hello")){
          println("存在"+ s5("hello"))
    }
    else {
         println("......没有这个key")
    }
    //3.map.get(key).get  第一个get取出的是一个some,如果第一个取出的值为空则会报错
    println(s5.get("1").get)
    //4.getorelse方法,适合的场景是要准确取值 不存在会返回默认值
    println(s5.getOrElse("helloworld","...."))
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-22009982b331fa8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####map的修改 增加 删除元素
![image.png](https://upload-images.jianshu.io/upload_images/9049859-068bf812c44ad008.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-4013452877660862.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-554f73b7cf074893.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####map的遍历
![image.png](https://upload-images.jianshu.io/upload_images/9049859-74b40a2dcdb06f42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码:
```
  //对map的修改 增加 删除 和 遍历
    var map1 = mutable.Map(1 -> 11, 2 -> 22, 3 -> 33, 4 -> 44)
    //修改
    map1(1) = 22 //没有则增加,有则更新
    //增加
    map1 += (1 -> 222, 8 -> 88) //可以一次增加一个 可以一次增加多个
    //删除
    map1 -= (1) //(key),可以填写多个key
    println(map1)
    //四种遍历方法
    for ((k, v) <- map1) {
      println(k + "\t" + v)
    }

    for (tumpl <- map1) {
      println(tumpl._1 + "\t" + tumpl._2)
      println(tumpl)
    }

    for (k <- map1.keys) {
      println(map1(k))
      println(k)
    }
    for (v <- map1.values) {
      println(v)
    }
```


补充:自己实现 ++=  *=  +()方法

