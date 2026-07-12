>Wordcount案例  数据源为集合时如何分区

##1.spark jar包上传到Linux并执行

spark-submit --master spark://linux03:7077 --executor-memory 1g --total-executor-cores 4 --class cn.tongyongtao.Day1.Test1   /root/demo.jar hdfs://linux03:8020/mydata/words.txt   hdfs://linux03:8020/mydata/hh.txt

对于参数的说明
--master 指定masterd地址和端口，协议为spark://，端口是RPC的通信端口
--executor-memory 指定每一个executor的使用的内存大小
--total-executor-cores指定整个application总共使用了cores
--class 指定程序的main方法全类名(全类名 reference)
jar包路径 args0 args1


##2.wordcount案例Scala Java Python实现
```
object Test1 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("")
    val sc = new SparkContext(conf)
    val data = sc.textFile(args(0))
      data.flatMap(_.split(" "))
      .map((_,1))
      .reduceByKey(_+_)
      .sortBy(_._2,false)
      .saveAsTextFile(args(1))
      sc.stop()
  }
}
```
```
package Day1;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import scala.Tuple2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;

public class Test1 {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("RDD");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> lines = sc.textFile(args[0]);
        //首先继续flatmap操作
        JavaRDD<String> flatMap = lines.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String s) throws Exception {
                return new ArrayList<String>(Arrays.asList(s.split(" "))).iterator();
            }
        });
        //这里为什么使用mapToPair  而不是 map
        JavaPairRDD<String, Integer> mapToPair = flatMap.mapToPair(new PairFunction<String, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(String s) throws Exception {
                return Tuple2.apply(s, 1);

            }
        });
        //对值继续sum
        JavaPairRDD<String, Integer> reduceByKey = mapToPair.reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer v1, Integer v2) throws Exception {
                return v1 + v2;
            }
        });
        //排序,但是只能安key来,反转
        JavaPairRDD<Integer, String> mapToPair1 = reduceByKey.mapToPair(new PairFunction<Tuple2<String, Integer>, Integer, String>() {
            @Override
            public Tuple2<Integer, String> call(Tuple2<String, Integer> stringIntegerTuple2) throws Exception {
                return stringIntegerTuple2.swap();


            }
        });
        //排序
        JavaPairRDD<Integer, String> sortByKey = mapToPair1.sortByKey(false);
        //再次反转
        JavaPairRDD<String, Integer> mapToPair2 = sortByKey.mapToPair(new PairFunction<Tuple2<Integer, String>, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(Tuple2<Integer, String> integerStringTuple2) throws Exception {
                return integerStringTuple2.swap();
            }
        });

        mapToPair2.saveAsTextFile(args[1]);
        sc.stop();
    }
}
```

#33.并行度和分区
![image.png](https://upload-images.jianshu.io/upload_images/9049859-35b83e34f7b940e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分区数与数据不成比例如何?
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2a66bca27d19129f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-42791ac891c3c0d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

