Hive是通过利用或扩展Hadoop的组件功能来运行的,常见的抽象有InputFormat,OutputFormat, Mapper和Reducer,还包含一些自己的抽象接口,例如SerializerDeserializer(SerDe)、用户自定义函数(UDF)和StorageHandlers.

Hive中提供了多个语法来使用streaming,包括: MAPO,REDUCE)和TRANSFORM().....Streaming也不怎么用吧,直接用UDF不好吗??

>使用Transform（Python）执行效率低的根本原因在于Python是直接向操作系统申请资源，而不是向YARN的ResourceManager申请资源，故而导致节点的资源无法高效组织和被利用

参考和踩坑:https://blog.csdn.net/purisuit_knowledge/article/details/81565295

##1. 恒等变换
![恒等变换示例](https://upload-images.jianshu.io/upload_images/9049859-0f3d90a03a35ef31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. 改变类型
![改变类型示例](https://upload-images.jianshu.io/upload_images/9049859-2080b35efa9ca140.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3. 投影变换
![投影变换](https://upload-images.jianshu.io/upload_images/9049859-365c2936cf302b81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4. 操作变换
![操作变换示例](https://upload-images.jianshu.io/upload_images/9049859-006d866f8c2fd851.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##5. 使用分布式内存
##6. 一行转多行
##7. 使用streaming进行聚合计算
##8. CLUSTER BY. DISTRIBUTE BY. SORT BY
##9. cogroup

??job是怎么分配的
