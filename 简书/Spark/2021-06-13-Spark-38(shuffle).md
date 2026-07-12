
#####1. shuffle算子是否一定触发shuffle?
![一个实例](https://upload-images.jianshu.io/upload_images/9049859-8f4fa7fd10d92aa7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![执行计划](https://upload-images.jianshu.io/upload_images/9049859-089de14e61abc756.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

中间就涉及到shuffle 过程，前一个stage 的 ShuffleMapTask 进行 shuffle write， 把数据存储在 blockManager 上面， 并且把数据位置元信息上报到 driver 的 mapOutTrack 组件中， 下一个 stage 根据数据位置元信息， 进行 shuffle read， 拉取上个stage 的输出数据。

#####2.BypassMergeSortShuffleWriter
UnsafeShuffleWriter需要Serializer支持relocation，Serializer支持relocation：原始数据首先被序列化处理，并且再也不需要反序列，在其对应的元数据被排序后，需要Serializer支持relocation，在指定位置读取对应数据

BypassMergeSortShuffleWriter和Hash Shuffle中的HashShuffleWriter实现基本一致， 唯一的区别在于，map端的多个输出文件会被汇总为一个文件。 所有分区的数据会合并为同一个文件，会生成一个索引文件，是为了索引到每个分区的起始地址，可以随机 access 某个partition的所有数据。

![实现细节](https://upload-images.jianshu.io/upload_images/9049859-9044fd39630d8940.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是需要注意的是，这种方式不宜有太多分区，因为过程中会并发打开所有分区对应的临时文件，会对文件系统造成很大的压力。
具体实现就是给每个分区分配一个临时文件，对每个 record的key 使用分区器（模式是hash，如果用户自定义就使用自定义的分区器）找到对应分区的输出文件句柄，直接写入文件，没有在内存中使用 buffer。 最后copyStream方法把所有的临时分区文件拷贝到最终的输出文件中，并且记录每个分区的文件起始写入位置，把这些位置数据写入索引文件中。
#####3.SortShuffleWriter
SortShuffleWriter 中的处理步骤
1. 使用 PartitionedAppendOnlyMap 或者 PartitionedPairBuffer 在内存中进行排序，  排序的 K 是（partitionId， hash（key）） 这样一个元组。
2. 如果超过内存 limit， 我 spill 到一个文件中，这个文件中元素也是有序的，首先是按照 partitionId的排序，如果 partitionId 相同， 再根据 hash（key）进行比较排序
3. 如果需要输出全局有序的文件的时候，就需要对之前所有的输出文件 和 当前内存中的数据结构中的数据进行  merge sort， 进行全局排序

#####4.UnsafeShuffleWriter
UnsafeShuffleWriter 里面维护着一个 ShuffleExternalSorter， 用来做外部排序，  外部排序就是要先部分排序数据并把数据输出到磁盘，然后最后再进行merge 全局排序， 既然这里也是外部排序，跟 SortShuffleWriter 有什么区别呢， 这里只根据 record 的 partition id 先在内存 ShuffleInMemorySorter 中进行排序， 排好序的数据经过序列化压缩输出到换一个临时文件的一段，并且记录每个分区段的seek位置，方便后续可以单独读取每个分区的数据，读取流经过解压反序列化，就可以正常读取了。

整个过程就是不断地在 ShuffleInMemorySorter 插入数据，如果没有内存就申请内存，如果申请不到内存就 spill 到文件中，最终合并成一个 依据 partition id 全局有序 的大文件。

待续,还没搞完........

