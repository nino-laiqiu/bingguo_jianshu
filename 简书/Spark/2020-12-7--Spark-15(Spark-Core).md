##1.sort的采样分析
##2.cache/persist是lazy的但不是转换算子/行动算子
一个stage的分区数取决去最后一个RDD的分区数量的含义是指示不是一个stage分区总和
##3.sortby自定义排序
方法一:pojo类,把属性写在类名的后面
```
class Boy(val name:String,val age :Int,val salary:Double) extends Comparable[Boy] with Serializable {
  override def compareTo(that: Boy): Int = {
         if (this.name != that.name ){
                this.age - that.age
         }
         else {
                - java.lang.Double.compare(this.salary,that.salary)
         }
  }
  override def toString = s"Boy($name, $age, $salary)"
}
```
```
import org.apache.spark.{SparkConf, SparkContext}
//实现排序功能
object Test1 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("sorted").setMaster("local[*]"))
    val rdd = sc.makeRDD(List("小明,19,130.0", "小华,20,780.0", "小妮,18,230.9"))
    rdd.map(data => {
      val txt = data.split(",")
      val name =txt(0)
      val age = txt(1).toInt
      val salary = txt(2).toDouble
      new Boy(name,age,salary)
    }).sortBy(boy => boy).collect().foreach(println)
    sc.stop()
  }
}
```
**方法二:pojo类,把属性写在类里面,使用fastjson要提供set方法(注意事项)**
```
class Users extends Serializable with Ordered[Users] {
  @BeanProperty var name: String = _
  @BeanProperty var age: Int = _
  @BeanProperty var salary: Double = _

  override def toString = s"User($name, $age, $salary)"

  override def compare(that: Users): Int = {
        if (this.name == that.name){
           java.lang.Double.compare(this.salary,that.salary)
        }
     else {
           -this.age -that.age
        }
  }
}
```
```
object Test5 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("sort").setMaster("local[*]"))
    val json = List("{\"name\":\"小妮\",\"age\":\"18\",\"salary\":18}","{\"name\":\"小化\",\"age\":\"22\",\"salary\":12}")
    val jsontxt = sc.makeRDD(json)
    jsontxt.map(data => {
      val users = JSON.parseObject(data, classOf[Users])
      users
    }).sortBy(u => u).collect().foreach(println)
    sc.stop()
  }
}
```
**fastjson是通过反射获取类的,类中要提供对应格式的set方法,最好将属性定义在构造器中,如果没有构造器(关于构造器中的属性:构造器中定义少的属性,辅助构造去才是定义较多的属性),则使用@BeanProperty添加getget方法,最好使用版本高的fastjson**
**pojo类scala自动生成的getset方法并不是真正的方法.要自己添加,最好使用样例类,对于样例类,是多例的,而不是单例的,其本质上是调用的是apply方法,样例类中的属性是val默认修饰的,要采用var,必须自己添加**

**方法三:样例类:使用隐式转换(没写完 .隐式转换的流程不清晰,再学一遍)**
```
object Test6 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("sort").setMaster("local[*]"))
    val json = List("{\"name\":\"小妮\",\"age\":\"18\",\"salary\":18}", "{\"name\":\"小化\",\"age\":\"22\",\"salary\":12}")
    val jsontxt = sc.makeRDD(json)
    val mapjson = jsontxt.map(data => {
      val employee = JSON.parseObject(data, classOf[Employee])
      employee
    })
    import  ObjectContext.order
    mapjson.sortBy(e => e).collect().foreach(println)
    sc.stop()
  }
}
```
```
object ObjectContext {

  implicit object order extends Ordering[Employee] {
    override def compare(x: Employee, y: Employee): Int = {
      -java.lang.Double.compare(x.salary, y.salary)
    }
  }
}
```
```
case class Employee(name :String,age:Int,salary:Double){

}
```
**方法四:使用元组的的排序规则**
```
object Test4 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("sort").setMaster("local[*]"))
    val json = List("{\"name\":\"小妮\",\"age\":\"18\",\"salary\":18}","{\"name\":\"小化\",\"age\":\"22\",\"salary\":12}")
    val jsontxt = sc.makeRDD(json)
    jsontxt.map(data => {
      val nObject = JSON.parseObject(data)
      val name = nObject.getString("name")
      val age = nObject.getInteger("age").toInt
      val salary = nObject.getDouble("salary")
      (salary,(age,name))
    }).sortBy(t => t).collect().foreach(println)
    sc.stop()
  }
}
```

##4.序列化问题
1.使用object,就不需要new 一个实例(会出现线程安全的问题)
2.new 一个实例(每个task持有一个实例)
3.不在driver端初始化,就没有加载类,相当于工具类,在extutor端初始化
4.在算子内new 每来一条数据就new 一个实例效率低下
5.使用mapPartitions是一个迭代器new 一个实例,效率较高且不需要考虑序列化的问题
(重点学习1和2的适用场所:1适用于只读2适用于存在改的场景)
(使用1每个excutor共用一个地址,driver一个地址,如果是class的场景,每个task共用一个地址值)
(多线程问题与线程不安全类的解决简单解决方案):
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e9bafc14a84637dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
方法一:加锁(读写锁)
方法二:(一个extutor一个task)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c51646f65feb440d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##5.广播变量
![广播变量](https://upload-images.jianshu.io/upload_images/9049859-3a353ff1e56504a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
