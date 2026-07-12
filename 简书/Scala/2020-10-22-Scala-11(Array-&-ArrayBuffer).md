#####1.基本介绍
![image.png](https://upload-images.jianshu.io/upload_images/9049859-44be436336d106f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####2.定长数组的声明

```
  //1.[Int] 是泛型 可以是[Any]  (10)是容量
  def main(args: Array[String]): Unit = {
    var s1 = new Array[Int](4)
    s1(1) = 1
    s1(2) = 3
    s1(3) = 3
    val arr1: Array[Int] = new Array[Int](3)
    arr1(0) = 6
    arr1(1) = 6
    //遍历方法1  注意这种遍历方法 我一开始写的是  println(arr1[i])......没有理解
    for (i <- arr1) {
      println(i)
    }
    //直接赋值
    var s2 = Array(1, 2, 3, 4, 5, 6)
    //遍历方法2
    for (index <- 0 until s2.length) {
      println(s2(index))
    }
  }
}
```

#####3.变长数组的声明
```
  var s1 = ArrayBuffer(1, 2, 3, 4, 5, 6)
    //改
    s1(2)=22
    //查
    for (i <- s1) {
      println(i)
    }
    //增
    s1.append(1, 2, 3)
    for (i <- s1) {
      println(i)
    }
    //删,索引
    s1.remove(1)
```

#####4.Array 和 ArrayBuffer 的转换
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3fe0ecab2ffc8e7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####5.二维数组
```
 var s1 = Array.ofDim[Int](3, 4)
    //赋值
    s1(1)(1) = 100
    //遍历方法一
    for (index <- s1) {
      for (index2 <- index) {
        print(index2 + "\t")
      }
      println()
    }
   println("-----")
    //第二种遍历方法
    for (i <- 0 until s1.length) {
      for (j <- 0 until s1(i).length) {
        print(s1(i)(j) + "\t")
      }
      println()
    }
  }
```
#####6.scala数组转java的list(注意补充知识点)....好像执行不了....
![image.png](https://upload-images.jianshu.io/upload_images/9049859-326c97d3ee50c464.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####7.java集合转scala数组Buffer

