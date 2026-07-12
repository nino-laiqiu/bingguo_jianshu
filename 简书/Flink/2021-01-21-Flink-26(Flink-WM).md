##1.wm
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d3e0c564f3aa2331.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置5秒的时间窗口,设置延迟时间为3s的情形
当时间到达5s后,但是后面还有延迟的数据,延迟时间为3s,换言之实际的水位线为2秒
分析到达数据的情形
2  2-3 水位线不更改
3
6 6-3 实际的水位线为3,三之前的数据全部到达,如果后面还有3之前的数据,则数据丢失(疑惑???,是否会丢失)
7 实际为4
8 8-3 水位线为5 关闭5s的窗口
其中6 7 8在[5 10)的窗口中
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b441f4481c9f3258.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Watermark的核心原理**
Watermark的核心本质可以理解成一个延迟触发机制。  
在 Flink 的窗口处理过程中，如果确定全部数据到达，就可以对 Window 的所有数据做 窗口计算操作（如汇总、分组等），如果数据没有全部到达，则继续等待该窗口中的数据全 部到达才开始处理。这种情况下就需要用到水位线（WaterMarks）机制，它能够衡量数据处 理进度（表达数据到达的完整性），保证事件数据（全部）到达 Flink 系统，或者在乱序及 延迟到达时，也能够像预期一样计算出正确并且连续的结果。当任何 Event 进入到 Flink 系统时，会根据当前最大事件时间产生 Watermarks 时间戳。

那么 Flink 是怎么计算 Watermak 的值呢？
Watermark =进入Flink 的最大的事件时间(mxtEventTime)-指定的延迟时间(t)
那么有 Watermark 的 Window 是怎么触发窗口函数的呢？  
如果有窗口的停止时间等于或者小于 maxEventTime - t(当时的warkmark)，那么这个窗口被触发执行。

其核心处理流程如下图所示。
![触发计算,且不会被丢失](https://upload-images.jianshu.io/upload_images/9049859-9f661e3d99775a51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.wm的传递
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ea89ca2609de1ddd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3679252402b5aa81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>首先，watermark 会以广播的形式在算子之间进行传播。比如说上游的算子，它连接了三个下游的任务，它会把自己当前的收到的 watermark 以广播的形式传到下游。
第二，如果在程序里面收到了一个 Long.MAX_VALUE 这个数值的 watermark，就表示对应的那一条流的一个部分不会再有数据发过来了，它相当于就是一个终止的一个标志。
第三，对于单流而言，这个策略比较好理解，而对于有多个输入的算子，watermark 的计算就有讲究了，一个原则是：单输入取其大，多输入取小。

>举个例子，假设这边蓝色的块代表一个算子的一个任务，然后它有三个输入，分别是 W1、W2、W3，这三个输入可以理解成任何的输入，这三个输入可能是属于同一个流，也可能是属于不同的流。然后在计算 watermark 的时候，对于单个输入而言是取他们的最大值，因为我们都知道 watermark 应该遵循一个单调递增的一个原则。对于多输入，它要统计整个算子任务的 watermark 时，就会取这三个计算出来的 watermark 的最小值。即一个多个输入的任务，它的 watermark 受制于最慢的那条输入流。这一点类似于木桶效应，整个木桶中装的水会就是受制于最矮的那块板。

>watermark 在传播的时候有一个特点是，它的传播是幂等的。多次收到相同的 watermark，甚至收到之前的 watermark 都不会对最后的数值产生影响，因为对于单个输入永远是取最大的，而对于整个任务永远是取一个最小的。

>同时我们可以注意到这种设计其实有一个局限，具体体现在它没有区分你这个输入是一条流多个 partition 还是来自于不同的逻辑上的流的 JOIN。对于同一个流的不同 partition，我们对他做这种强制的时钟同步是没有问题的，因为一开始就是把一条流拆散成不同的部分，但每一个部分之间共享相同的时钟。但是如果算子的任务是在做类似于 JOIN 操作，那么要求你两个输入的时钟强制同步其实没有什么道理的，因为完全有可能是把一条离现在时间很近的数据流和一个离当前时间很远的数据流进行 JOIN，这个时候对于快的那条流，因为它要等慢的那条流，所以说它可能就要在状态中去缓存非常多的数据，这对于整个集群来说是一个很大的性能开销。


