>Dstream是什么
偏移量源码分析

##1.Dstream
Sparkstreaming 是一个微批次,准实时的流式计算框架
Dstream相当于一个不断产生RDD的工厂,离线相比,增强在driver,不断产生RDD
```
A DStream internally is characterized by a few basic properties:
 - A list of other DStreams that the DStream depends on
 - A time interval at which the DStream generates an RDD
 - A function that is used to generate an RDD after each time interval
```
每一个Dstream都维护一个hashmap(time,rdd)用来存储rdd,但是这样会有一个问题,如果程序一直运行,hashmap就会越来越大,所以spark采用一旦执行 完这次,就会清空这次rdd的描述信息
```
例如:dstream中的map方法底层是MappedDStream
def map[U: ClassTag](mapFunc: T => U): DStream[U] = ssc.withScope {
    new MappedDStream(this, context.sparkContext.clean(mapFunc))
  }
```
```
rdd task会调用computer方法
override def compute(validTime: Time): Option[RDD[U]] = {
    parent.getOrCompute(validTime).map(_.map[U](mapFunc))
  }
```
```
 private[streaming] var generatedRDDs = new HashMap[Time, RDD[T]]()
```

![image.png](https://upload-images.jianshu.io/upload_images/9049859-42f0f1d4e3f92508.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##偏移量
获取偏移量,第一手的rdd是特殊的rdd,kafkardd,只有kafkardd才有偏移量(在driver端获取)
