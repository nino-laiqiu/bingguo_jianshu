##1.startNewChain

```
public class Test3 {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-17
     * Time: 22:04
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Tuple2<String, Integer>> map = source.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String s) throws Exception {
                return Tuple2.of(s, 1);
            }
        }).setParallelism(1);
        SingleOutputStreamOperator<Tuple2<String, Integer>> filter = map.filter(x -> x.f0.startsWith("a")).startNewChain();
        KeyedStream<Tuple2<String, Integer>, Tuple> keyBy = filter.keyBy(0);
        keyBy.sum(1).print().setParallelism(1);
        environment.execute("job");
    }
}
```

![image.png](https://upload-images.jianshu.io/upload_images/9049859-d263ed6a9d18bdfd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##2.disableChaining
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3825e0fbaef2b49c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##3.join
```
public class Join {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-18
     * Time: 21:22
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        DataStreamSource<String> source1 = environment.socketTextStream("localhost", 9999);

        //添加水位线
        SingleOutputStreamOperator<String> watermarks = source.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<String>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(String element) {
                return Long.parseLong(element.split(",")[0]);
            }
        });
        SingleOutputStreamOperator<String> watermarks1 = source1.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<String>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(String element) {
                return Long.parseLong(element.split(",")[0]);
            }
        });

        SingleOutputStreamOperator<Tuple3<Long, String, Integer>> map = watermarks.map(new MapFunction<String, Tuple3<Long, String, Integer>>() {
            @Override
            public Tuple3<Long, String, Integer> map(String s) throws Exception {
                String[] split = s.split(",");
                return Tuple3.of(Long.parseLong(split[0]), split[1], Integer.parseInt(split[2]));
            }
        });

        SingleOutputStreamOperator<Tuple3<Long, String, Integer>> map1 = watermarks1.map(new MapFunction<String, Tuple3<Long, String, Integer>>() {
            @Override
            public Tuple3<Long, String, Integer> map(String s) throws Exception {
                String[] split = s.split(",");
                return Tuple3.of(Long.parseLong(split[0]), split[1], Integer.parseInt(split[2]));
            }
        });
        map.join(map)
                //条件
                .where(new KeySelector<Tuple3<Long, String, Integer>, String>() {
                    @Override
                    public String getKey(Tuple3<Long, String, Integer> longStringIntegerTuple3) throws Exception {
                        return longStringIntegerTuple3.f1;
                    }
                    //条件
                }).equalTo(new KeySelector<Tuple3<Long, String, Integer>, String>() {
            @Override
            public String getKey(Tuple3<Long, String, Integer> longStringIntegerTuple3) throws Exception {
                return longStringIntegerTuple3.f1;
            }
            //窗口
        }).window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .apply(new JoinFunction<Tuple3<Long, String, Integer>, Tuple3<Long, String, Integer>, Tuple6<Long, String, Integer, Long, String, Integer>>() {
                    @Override
                    public Tuple6<Long, String, Integer, Long, String, Integer> join(Tuple3<Long, String, Integer> l1, Tuple3<Long, String, Integer> l2) throws Exception {
                        return Tuple6.of(l1.f0, l1.f1, l2.f2, l2.f0, l2.f1, l2.f2);
                    }
                }).print();

        environment.execute("joinjob");

    }
}
```
##4.LeftJoin/RightJoin

```
public class LeftJoin {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-18
     * Time: 22:58
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        DataStreamSource<String> source1 = environment.socketTextStream("localhost", 9999);


        SingleOutputStreamOperator<String> watermarks = source.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<String>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(String element) {
                return Long.parseLong(element.split(",")[0]);
            }
        });

        SingleOutputStreamOperator<String> watermarks1 = source1.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<String>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(String element) {
                return Long.parseLong(element.split(",")[0]);
            }
        });

        SingleOutputStreamOperator<Tuple3<Long, String, Integer>> map = watermarks.map(new MapFunction<String, Tuple3<Long, String, Integer>>() {
            @Override
            public Tuple3<Long, String, Integer> map(String s) throws Exception {
                String[] split = s.split(",");
                return Tuple3.of(Long.parseLong(split[0]), split[1], Integer.parseInt(split[2]));
            }
        });

        SingleOutputStreamOperator<Tuple3<Long, String, Integer>> map1 = watermarks1.map(new MapFunction<String, Tuple3<Long, String, Integer>>() {
            @Override
            public Tuple3<Long, String, Integer> map(String s) throws Exception {
                String[] split = s.split(",");
                return Tuple3.of(Long.parseLong(split[0]), split[1], Integer.parseInt(split[2]));
            }
        });

        map.coGroup(map1)
                .where(new KeySelector<Tuple3<Long, String, Integer>, String>() {
                    @Override
                    public String getKey(Tuple3<Long, String, Integer> longStringIntegerTuple3) throws Exception {
                        return longStringIntegerTuple3.f1;
                    }
                })
                .equalTo(new KeySelector<Tuple3<Long, String, Integer>, String>() {
                    @Override
                    public String getKey(Tuple3<Long, String, Integer> longStringIntegerTuple3) throws Exception {
                        return longStringIntegerTuple3.f1;
                    }
                })
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .apply(new CoGroupFunction<Tuple3<Long, String, Integer>, Tuple3<Long, String, Integer>, Tuple6<Long, String, Integer, Long, String, Integer>>() {
                    @Override
                    public void coGroup(Iterable<Tuple3<Long, String, Integer>> left, Iterable<Tuple3<Long, String, Integer>> right, Collector<Tuple6<Long, String, Integer, Long, String, Integer>> collector) throws Exception {
                        //实现左外连接
                        //设置一个标签,只要右边的为空就设置为false
                        //如果是右外连接则改变顺序即可
                        for (Tuple3<Long, String, Integer> first : left) {
                            boolean flag = true;

                            for (Tuple3<Long, String, Integer> second : right) {
                                collector.collect(Tuple6.of(first.f0, first.f1, first.f2, second.f0, second.f1, second.f2));
                                flag = false;
                            }
                            if (flag) {
                                collector.collect(Tuple6.of(first.f0, first.f1, first.f2, null, null, null));
                            }
                        }
                    }
                })
                .print().setParallelism(1);
        environment.execute("leftjoin");

    }
}

```

##5.intervalJoin
```
public class MyProcessJoinFunction extends ProcessJoinFunction<
        Tuple3<Long, String, String>, //第一个数据流（左流）输入的数据类型
        Tuple3<Long, String, String>, //第二个数据流（右流）输入的数据类型
        Tuple6<Long, String, String, Long, String, String>> { //join后输出的数据类型
    @Override
    public void processElement(Tuple3<Long, String, String> left, //左流输入的一条数据
                               Tuple3<Long, String, String> right, //右流输入的一条数据
                               Context ctx, //上下文信息，可以获取各个流的timestamp和侧流输出的output
                               //用来输出join上的数据的Collector
                               Collector<Tuple6<Long, String, String, Long, String, String>> out)
            throws Exception {
        //将join上的数据添加到Collector中输出
        out.collect(Tuple6.of(left.f0, left.f1, left.f2, right.f0, right.f1, right.f2));
    }
}
```

```
public class IntervalJoin {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //1000,A,1
        DataStreamSource<String> leftLines = env.socketTextStream("localhost", 8888);
        //2000,A,2
        DataStreamSource<String> rightLines = env.socketTextStream("localhost", 9999);

        //提取第一个流中数据的EventTime
        DataStream<String> leftWaterMarkStream = leftLines
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<String>(Time.seconds(0)) {
                    @Override
                    public long extractTimestamp(String line) {
                        return Long.parseLong(line.split(",")[0]);
                    }
                });
        //提取第二个流中数据的EventTime
        DataStream<String> rightWaterMarkStream = rightLines
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<String>(Time.seconds(0)) {
                    @Override
                    public long extractTimestamp(String line) {
                        return Long.parseLong(line.split(",")[0]);
                    }
                });
        //对第一个流整理成tuple3
        DataStream<Tuple3<Long, String, String>> leftStream = leftWaterMarkStream.map(
                new MapFunction<String, Tuple3<Long, String, String>>() {
                    @Override
                    public Tuple3<Long, String, String> map(String value) throws Exception {
                        String[] fields = value.split(",");
                        return Tuple3.of(Long.parseLong(fields[0]), fields[1], fields[2]);
                    }
                }
        );
        //对第二个流整理成tuple3
        DataStream<Tuple3<Long, String, String>> rightStream = rightWaterMarkStream.map(
                new MapFunction<String, Tuple3<Long, String, String>>() {
                    @Override
                    public Tuple3<Long, String, String> map(String value) throws Exception {
                        String[] fields = value.split(",");
                        return Tuple3.of(Long.parseLong(fields[0]), fields[1], fields[2]);
                    }
                }
        );
        DataStream<Tuple6<Long, String, String, Long, String, String>> joinedStream = leftStream
                .keyBy(t -> t.f1) //指定第一个流分组KeySelector
                .intervalJoin(rightStream.keyBy(t -> t.f1)) //调用intervalJoin方法并指定第二个流的分组KeySelector
                .between(Time.seconds(-1), Time.seconds(1)) //设置join的时间区间范围为当前数据时间±1秒
                .upperBoundExclusive() //默认join时间范围为前后都包括的闭区间，现在设置为前闭后开区间
                .process(new MyProcessJoinFunction()); //调用process方法中传入自定义的MyProcessJoinFunction
        joinedStream.print(); //调用print sink 输出结果
        env.execute("IntervalJoinDemo");
    }
}
```
##6.任务槽和资源
>每个TaskManager是一个JVM进程，并且可以在单独的线程中执行一个或多个子任务。为了控制worker接受多少个任务，worker具有所谓的任务槽【至少有一个】。
每个任务槽代表TaskManager的资源的固定子集。例如，具有三个插槽的TaskManager会将其托管内存的1/3专用于每个插槽。分配资源意味着子任务不会与其他作业的子任务竞争托管内存，而是具有一定数量的保留托管内存。请注意，此处没有CPU隔离。当前插槽仅将任务的托管内存分开。
通过调整任务槽的数量，用户可以定义子任务如何相互隔离。每个TaskManager具有一个插槽，这意味着每个任务组都在单独的JVM上运行。具有多个插槽意味着更多子任务共享同一个JVM。同一JVM中的任务共享TCP连接【通过多路复用】和心跳消息。他们还可以共享数据集和数据结构，从而减少每个任务的开销。图解如下：

![image.png](https://upload-images.jianshu.io/upload_images/9049859-e7f694fdfea25593.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>默认情况下，Flink允许子任务共享插槽，即使它们是不同任务的子任务也是如此，只要它们来自同一任务即可。结果是一个插槽可以容纳整个作业流水线。允许此插槽共享有两个主要好处：
Flink集群所需的任务槽与作业中使用的最高并行度恰好一样多。无需计算一个程序总共包含多少个任务。
更容易获得更好的资源利用率。如果没有插槽共享，则非密集型source/map子任务将阻塞与资源密集型窗口子任务一样多的资源。通过插槽共享，可以充分利用插槽资源，同时确保沉重的子任务在TaskManager之间公平分配

![image.png](https://upload-images.jianshu.io/upload_images/9049859-3e5a82495eadec47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
