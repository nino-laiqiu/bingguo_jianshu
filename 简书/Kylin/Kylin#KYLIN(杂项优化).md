##### join的使用
只能按照构建model的连接条件来写SQL，并且还有一个要求就是必须是事实表在前，维度表在后否则报错


#####cube的存储
维度字典
![维度字典](https://upload-images.jianshu.io/upload_images/9049859-16009b8cbada40b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


映射为hbase的key-value结构 

![映射](https://upload-images.jianshu.io/upload_images/9049859-a74726d30e741346.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 衍生维度

![衍生维度](https://upload-images.jianshu.io/upload_images/9049859-9fc6e2bf98e8aa09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

维度表的N个维度组合成的cuboid个数会从2的N次方降为2,预计算的速度会加快，但是查询的时候要在线计算


#####ROWKEY的设计原则
1.  查询优化
被用作过滤条件的维度放在前面
![查询优化](https://upload-images.jianshu.io/upload_images/9049859-6432d4ea1decb52d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.  构建优化
基数大的维度放在基数小的维度前面，kylin自动选择的按照cuboid小的，要遵循机制
![构建优化](https://upload-images.jianshu.io/upload_images/9049859-a1f612eee6522f2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
