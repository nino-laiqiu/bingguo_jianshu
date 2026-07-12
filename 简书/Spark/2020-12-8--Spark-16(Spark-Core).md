##1.广播变量的复习
广播数据必须要在driver端准备好(无论以何种方式),以BT的方式把数据广播给executor(每个executor并不拥有全部的数据)
```
object Test2 {
  def main(args: Array[String]): Unit = {
    val sc = new SparkContext(new SparkConf().setAppName("iptest").setMaster("local[*]"))
    val rdd = sc.textFile("src/main/resources/ip.txt", 4)
    val iptxt: Array[(Long, Long, String, String)] = rdd.map(data => {
      val txt = data.split("[|]")
      val start = txt(2).toLong
      val stop = txt(3).toLong
      val province = txt(6)
      val city = txt(7)
      (start, stop, province, city)
    }).collect()
    //广播变量,这是一个阻塞方法,没有广播完,下面的代码无法执行
    val ipBT: Broadcast[Array[(Long, Long, String, String)]] = sc.broadcast(iptxt)
    val IAP: RDD[(String, Int)] = sc.textFile("src/main/resources/ipaccess.log").map(data => {
      val logtxt = data.split("[|]")
      //转换编码
      val l = IpUtils.ip2Long(logtxt(1))
      //在广播数据中查找,知识点二分查找
      val executorindex = IpUtils.binarySearch(ipBT.value, l)
      var province: String = null
      if (executorindex != -1) {
        //说明查找到了数据
        province = ipBT.value(executorindex)._3
      }
      //wordcount案例
      (province, 1)
    })
    IAP.map {
      case (x, y) => (x, y + 10)
    }.reduceByKey(_ + _).collect().foreach(println)
    sc.stop()
  }
}
```
```
object IpUtils {
  def ip2Long(ip: String): Long = {
    val fragments = ip.split("[.]")
    var ipNum = 0L
    for (i <- 0 until fragments.length) {
      ipNum = fragments(i).toLong | ipNum << 8L
    }
    ipNum
  }
  def binarySearch(lines: Array[(Long, Long, String, String)], ip: Long): Int = {
    var low = 0 //起始
    var high = lines.length - 1 //结束
    while (low <= high) {
      val middle = (low + high) / 2
      if ((ip >= lines(middle)._1) && (ip <= lines(middle)._2))
        return middle
      if (ip < lines(middle)._1)
        high = middle - 1
      else {
        low = middle + 1
      }
    }
    -1 //没有找到
  }
}
```
##2.理解driver端 executor 端
##3.累加器
##4.反面教材:嵌套RDD
task是driver端生成的传递给executor的.
里面的RDD是在executor调用的,没有准备工作
##5.spark on yarn 
spark启动 shell命令的学习
sortBy采样,如下图灰色复用是只有在sortBy之前shuffle才会复用
![image.png](https://upload-images.jianshu.io/upload_images/9049859-880085d6233e138f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

spark on yarn的流程叙述
spark on yarn(client) 和  spark on yarn(cluster)  的区别
##6.driver  和  submit概念
##7.磁盘的归并操作
