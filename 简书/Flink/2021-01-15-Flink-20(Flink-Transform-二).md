
##1.union
>该方法可以将两个或者多个数据类型一致的DataStream合并成一个DataStream。DataStream<T> union(DataStream<T>… streams)可以看出DataStream的union方法的参数为可变参数，即可以合并两个或多个数据类型一致的DataStream

```
public class Union {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 14:59
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        DataStreamSource<String> source1 = environment.socketTextStream("localhost", 9999);
        DataStream<String> union = source.union(source1);
        union.print();

        environment.execute("job");

    }
}
```
```
public class Union {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 14:59
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<Integer> source = environment.fromElements(1, 3, 5, 7, 9);
        DataStreamSource<Integer> source1 = environment.fromElements(2, 4, 6, 8, 10);
        DataStream<Integer> union = source.union(source1);
        union.print();
        environment.execute("job");

    }
}
```
##2.connect
>两个数据类型一样也可以类型不一样DataStream连接成一个新的ConnectedStreams。需要注意的是，connect方法与union方法不同，虽然调用connect方法将两个流连接成一个新的ConnectedStreams，但是里面的两个流依然是相互独立的，这个方法最大的好处是可以让两个流共享State状态

>new CoMapFunction<String, Integer, String>()，需要输入三个泛型：第一个String类型代表第一个DataStream输入的数据类型；第二个Integer类型代表第二个DataStream输入的数据类型；第三个String类型代返回DataStream的数据类型。这个匿名类要重写两个方法，一个是map1方法，是对第一个流进行map的处理逻辑。另一个是map2方法，是对二个流进行map的处理逻辑，这两个方法都必须返回String类型。最终返回一个新的DataStream中的数据是String
```
public class Connect {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 15:33
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.fromElements("spark", "flink", "java");
        DataStreamSource<Integer> source1 = environment.fromElements(1, 2, 3, 4, 5);
        ConnectedStreams<String, Integer> connect = source.connect(source1);
        SingleOutputStreamOperator<String> map = connect.map(new CoMapFunction<String, Integer, String>() {
            @Override
            public String map1(String value) throws Exception {
                return value.toUpperCase();
            }
            @Override
            public String map2(Integer value) throws Exception {
                return value * 2 + "";
            }
        });
        map.print();
        environment.execute("job");
    }
}
```
>new CoFlatMapFunction<String, String, String>()，需要指定三个泛型：第一个String类型代表第一个DataStream输入的数据类型；第二个String类型代表第二个DataStream输入的数据类型；第三个String类型代返回DataStream的数据类型。这个匿名类要重写两个方法，一个是flatMap1方法，是对第一个流进行flatMap的处理逻辑。另一个是flatMap2方法，是对二个流进行flatMap的处理逻辑，这两个方法都必须返回String都要通过各自的Collector收集，最终返回一个新的DataStream中的数据是String
```
public class Connect {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 15:33
     */
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.fromElements("spark,flink,scala");
        DataStreamSource<String> source1 = environment.fromElements("1,2,3");
        ConnectedStreams<String, String> connect = source.connect(source1);
        SingleOutputStreamOperator<String> flatMap = connect.flatMap(new CoFlatMapFunction<String, String, String>() {
            @Override
            public void flatMap1(String value, Collector<String> out) throws Exception {
                for (String s : value.split(",")) {
                    out.collect(s);
                }
            }

            @Override
            public void flatMap2(String value, Collector<String> out) throws Exception {
                for (String s : value.split(",")) {
                    out.collect(s);
                }
            }
        });
        flatMap.print();
        environment.execute("job");

    }
}
```
##3.iterate
```
public class Iterate {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 16:10
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Long> map = source.map(new MapFunction<String, Long>() {
            @Override
            public Long map(String s) throws Exception {
                return Long.parseLong(s);
            }
        });
        IterativeStream<Long> iterate = map.iterate();
        //对Nums进行迭代
        SingleOutputStreamOperator<Long> operator = iterate.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long aLong) throws Exception {
                System.out.println("触发");
                return aLong -= 1;
            }
        });

        //只要满足value > 0的条件，就会形成一个回路，重新的迭代
        SingleOutputStreamOperator<Long> filter = operator.filter(new FilterFunction<Long>() {
            @Override
            public boolean filter(Long aLong) throws Exception {
                return aLong > 0;
            }
        });

        iterate.closeWith(filter);

        SingleOutputStreamOperator<Long> filter1 = operator.filter(new FilterFunction<Long>() {
            @Override
            public boolean filter(Long aLong) throws Exception {
                return aLong < 0;
            }
        });

        filter.print();
        environment.execute("kob");


    }
}
```
##4.project
```
//使用fromElements生成数据，数据为Tuple3类型，分别代表姓名、性别、年龄
DataStreamSource<Tuple3<String, String, Integer>> users = env.fromElements(
        Tuple3.of(“佩奇”, “女”, 5), Tuple3.of(“乔治”, “男”, 3)
);
//投影方法只可以用于类型为Tuple的DataStream，返回的数据只要姓名和年龄
DataStream<Tuple> result = users.project(0, 2);
```

##5. split/select
>split是将一个DataStream中的数据流打上不同的标签，逻辑的拆分成多个不同类型的流，返回一个新的SplitStream，本质上还是一个数据流，只不过是将流中的数据打上了不同的标签

>select是将一个关联上多个标签的SplitStream中的数据流根据一个或多个标签名称进行筛选，返回一个新的DataStream。select方法传入的是一个字符串类型的可变参数，即可以选取一个标签名称的数据流，也可以选取多个标签名称的数据流

```
DataStreamSource<Integer> numbers = env.fromElements(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
//将数据打上标签，拆分成奇数和偶数
SplitStream<Integer> splited = numbers.split(new OutputSelector<Integer>() {
    @Override
    public Iterable<String> select(Integer value) {
        List<String> out = new ArrayList<String>();
        if (value % 2 == 0) {
            out.add(“even”); //将该条数据打上even的标签
        } else {
            out.add(“odd”); //将该条数据打上odd的标签
        }
        return out; //返回带有标签的集合
    }
});
```
```
//挑选出偶数的数据流
DataStream<Integer> even = splited.select(“even”);
//挑选出奇数的数据流
DataStream<Integer> odd = splited.select(“odd”);
//挑选出奇数、偶数全部的数据流
DataStream<Integer> all = splited.select(“even”,”odd”);
```

##6.Rebalance/shuffle
```
//轮询
public class Rebalance {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 16:52
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<String> map = source.map(new RichMapFunction<String, String>() {
            @Override
            public String map(String s) throws Exception {
                int subtask = getRuntimeContext().getIndexOfThisSubtask();
                return s + "->" + subtask;
            }
            //设置并行度
        }).setParallelism(1);
        DataStream<String> rebalance = map.rebalance();
        DataStreamSink<String> addSink = rebalance.addSink(new RichSinkFunction<String>() {
            @Override
            public void invoke(String value, Context context) throws Exception {
                int subtask = getRuntimeContext().getIndexOfThisSubtask();
                System.out.println(value + "->" + subtask);
            }
        });
        environment.execute("job");

    }
}
```
//shuffle不同于轮询,是没什么规律的
![image.png](https://upload-images.jianshu.io/upload_images/9049859-436efaab3a92519e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##7.broadcast
广播复制把上游的数据广播到下游的所有数据
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b86b88580b7b6f2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##8.partitionCustom
```
public class CustomPartitioningDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());

        //Source是一个非并行的Source
        //并行度是1
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        //并行度2
        SingleOutputStreamOperator<Tuple2<String, Integer>> mapped = lines.map(new RichMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
                return Tuple2.of(value, indexOfThisSubtask);
            }
        });//.setParallelism(2);

        //按照指定的规则进行分区
        DataStream<Tuple2<String, Integer>> partitioned = mapped.partitionCustom(new Partitioner<String>() {

            @Override
            public int partition(String key, int numPartitions) {
                //System.out.println("key: " + key  + " ,下游task的并行度：" + numPartitions);
                int res = 0;
                if("spark".equals(key)) {
                    res = 1;
                } else if ("flink".equals(key)){
                    res = 2;
                } else if("hadoop".equals(key)) {
                    res = 3;
                }
                return res;
            }
            //分区的Key
        }, tp -> tp.f0);

        partitioned.addSink(new RichSinkFunction<Tuple2<String, Integer>>() {

            @Override
            public void invoke(Tuple2<String, Integer> value, Context context) throws Exception {

                int index = getRuntimeContext().getIndexOfThisSubtask();
                System.out.println(value.f0 + " , 上游 " + value.f1 + " -> 下游 " + index);
            }
        });

        env.execute();


    }
}
```
