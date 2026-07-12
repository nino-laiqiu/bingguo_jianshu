##1.apply 方法 和  update  方法 应用于map

![image.png](https://upload-images.jianshu.io/upload_images/9049859-f6e5f1314cb53fe6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

theArray(0), 取数组的第一个元素的操作会转换成 theArray.apply(0) 操作，这也能解释为什么 Scala 数组取值不用中括号括下标的方式，因为它也是一次方法调用

##2.apply 方法  和   unapply  方法    
* 通常，在一个类的伴生对象中定义apply方法，在生成这个类的对象时，就省去了new关键字
* unapply方法是apply方法的反向操作，apply方法接受构造参数变成对象，而unapply方法接受一个对象从中提取值。

代码:
```
object Test1 {
  def main(args: Array[String]): Unit = {
       val  user = Users("小明",19)
        var result = user match {
        case  Users(name,19) => name
        case  _ =>
      }
      println(result)
  }
}

class   Users(var name:String,var age:Int){}
object  Users{
  def apply(name: String, age: Int): Users = new Users(name, age)
  def unapply(arg: Users): Option[(String, Int)] = {
       if (arg ==null ) None  else  Some(arg.name,arg.age)
  }
}
```

##3.unapplySeq方法(适用的场合)
```
object Test3 {
  def main(args: Array[String]): Unit = {
      //unapplySeq
     var name = "小明  小华  小静"
   var result = name match {
      case Name(f) => println(s"name + $f")
      case _ =>
    }
  }
}
class  Name(val name:Array[String])
object  Name{
  def apply(name: Array[String]): Name = new Name(name)

 // def unapply(arg: Name): Option[Array[String]] = ???

  def  unapplySeq(input:String):Option[Seq[String]]={
         if (input.length==0) None else Some(input.split("\t"))
  }
}
```
**(上面的不适用,理解下面的代码)**
**自动调用unapplyseq方法**
```
object Test3 {
  def main(args: Array[String]): Unit = {
      //unapplySeq
     var name = "小明 小华 小米"
   var result = name match {
      case Name(a,b,_) => println(s"name + $a + $b ")
      case Name(a) => println(s"name + $a  +1 ")
      case _ =>
    }
  }
}
object  Name{
 // def unapply(arg: Name): Option[Array[String]] = ???

  def  unapplySeq(input:String):Option[Seq[String]]={
         if (input.length==0) None else Some(input.split("\\s+"))
  }
}
```

##4.map  flatten   flatMap 区别
**flatten方法理解为打散集合元素,对于元组是无法打散的**
**对于flatmap是集map 和 flatten 方法 先进行map 然后是flatten方法**
```
object Test4 {
  def main(args: Array[String]): Unit = {
    var list = List("hh", "hui", "poi")
    list.map(data => data + "1").foreach(println)
    list.flatten(data => data.toUpperCase).foreach(println)
    // 因为flatMap有flatten的操作所以flatMap的调用对象必须是集合的集合，要满足flatten的要求
    list.flatMap(data => data + "1").foreach(println)
  }
}
```
##5.遍历机制的性能的比较
1.for
2.foreach
3.迭代器
>for循环性能在三者的对比中总体落于下风，而且开销递增幅度较大。以后即使在需要使用索引时我宁愿使用递增变量也不会使用for了。
Iterator的性能在数组以及链表的表现都是最好的，应该是JAVA的设计者优化过了。在响应时间敏感的情况下(例如web响应)，优先考虑。
foreach的性能属于两者之间，写法简单，时间不敏感的情况下我会尽量选用。

##6,groupby的案例
```
  var list = List[Tuple2[Int, Int]]((1, 2), (2, 3), (5, 6), (1, 3), (5, 8))
    list.groupBy(_._1).foreach(println)
    //(5,List((5,6), (5,8)))
    // (1,List((1,2), (1,3)))
    //(2,List((2,3)))
```
##7.三种Wordcount的实现代码
**读取字符串并排序**
```
object Test6 {
  def main(args: Array[String]): Unit = {
    var st = "AAABBBCCCDDRRDDD"
    println(count(st))
  }

  def count(st: String) = {
    val map = mutable.Map[Char, Int]()
    st.map(data =>
      map += (data -> (map.getOrElse(data, 0) + 1))
    )
    map.toList.sortBy(_._1)
  }
}
```
**读取一个数组或者集合并排序,注意使用size 和 reverse**
```
object Test7 {
  def main(args: Array[String]): Unit = {
     val  list = List[String]("asddadafef","DDdsfsf","asddaak")
     count(list)
  }
  
     //统计字母出现的次数 ,如果要统计单词出现的次数只要改变切割的regex就行,对于string来说底层是char
     def count(list: List[String]): Unit ={
          list.flatMap(_.split(""))
         .map(data=>((data,1)))
         .groupBy(data => data._1 )
         .map(data=>(data._1,data._2.size))
         .toList
         .sortBy(_._2)
         .reverse
         .foreach(println)
     }
}
```
**读取文件......老师采用的sum.....**

##8.Scala里面的排序函数的使用
sorted  对一个集合进行自然排序，通过传递隐式的Ordering
sortBy   对一个属性或多个属性进行排序，通过它的类型。
sortWith    基于函数的排序，通过一个comparator函数，实现自定义排序的逻辑。
 
对于map类要转换成list或者seq才能比较

**排序的性能:**
![image.png](https://upload-images.jianshu.io/upload_images/9049859-f02a68ebb70e30c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##9.关于元组的补充事项
![image.png](https://upload-images.jianshu.io/upload_images/9049859-6fa863af1a56f38f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##10改值器 取值器
![image.png](https://upload-images.jianshu.io/upload_images/9049859-4d074c3ed642633a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
