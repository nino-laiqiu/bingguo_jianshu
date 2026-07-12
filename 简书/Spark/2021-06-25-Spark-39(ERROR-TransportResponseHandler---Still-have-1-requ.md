ERROR TransportResponseHandler: 
Still have 1 requests outstanding when connection from  is closed

我加了如下的参数
```
spark.driver.memory=4g
spark.hadoop.mapreduce.input.fileinputformat.split.maxsize=134217728
spark.dynamicAllocation.initialExecutors=50
spark.yarn.executor.memoryOverhead=2048
spark.dynamicAllocation.maxExecutors=1000
spark.dynamicAllocation.tasksPerExecutorSlot=1
```

运行的程序其实逻辑上比较简单，只是从hive表里读取的数据量很大，差不多60+G，
并且需要将某些hive表读取到dirver节点上，用来获取每个executor上某些数据的映射值，
所以driver设定的资源较大。运行时抛出的异常信息，从网上查了下原因大致是服务器的
并发连接数超过了其承载量，服务器会将其中一些连接Down掉，这也就是说在运行spark程序时，过多的申请资源并发执行。

最近在学习spark调优属实有点难度哈!!!!!
