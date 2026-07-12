>要点:
知道数据源分别为集合,文本 分区数的内涵以及读取时的偏移量的计算公式与内涵

##1.MapReduce的中间结果保留在哪
通过设置:
```
<property>
<name>keep.failed.task.files</name>
<value>true</value>
</property>
<property>
<name>keep.task.files.pattern</name>
<value>*</value>
</property>
这样所有中间临时文件都会被保存，map临时文件位于{hadoop.tmp.dir}/mapred/local/tasktracker/
```

##2.RDD论文


##3.Spark中术语
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2aa2e39f4ef7e489.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##4.Spark文件数据源如何切片,与分区有什么关系
```
	//我们得到了总的字节数，totalSize
		//这里我们定义一个理想化的切片目标大小，总的大小/我们得到的defaultParallelism
		//大概意思是想保证每个片的大小能够保证均分
        long goalSize = totalSize / (long)(numSplits == 0 ? 1 : numSplits);
        //这里是定义了一个最小的切片字节大小
        long minSize = Math.max(job.getLong("mapreduce.input.fileinputformat.split.minsize", 1L), this.minSplitSize);
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-1fcb6d55a69fda93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(同一个文本除以goalsize 剩余的字节如果占比超过0.1就占据一个切片数量)

##5.并行度和分区的关系
并行度可以自己设置,分区数可设置最小分区数.但这并不是真正的分区数,真正的分区数与数据块大小有关

##6.spark读取文本的偏移量
(注意spark读取文件是照hadoop的一行一行读取,不会重复读取)
(偏移量范围的计算就是goalsize的大小)
![偏移量](https://upload-images.jianshu.io/upload_images/9049859-5382e4dab8935d39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


