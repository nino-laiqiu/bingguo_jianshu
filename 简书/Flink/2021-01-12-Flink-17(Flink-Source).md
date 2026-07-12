##1.Wordcount案例优化与lambda表达式写法的注意事项

优化,把map((?,1)) 直接在flatmap中完成就减少了一个map算子

lambda表达式写法 要returns(Type) 指明类型
```
public class LambdaStreamingWordCount {

    public static void main(String[] args) throws Exception {

        //LocalStreamEnvironment只能是local模式运行，通常用于本地测试
        //LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment(8);
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> lines = env.socketTextStream(args[0], Integer.parseInt(args[1]));

        //使用Java8的lambda表达式
        SingleOutputStreamOperator<String> words = lines.flatMap((String line, Collector<String> out) ->
                Arrays.stream(line.split(" ")).forEach(out::collect)
        ).returns(Types.STRING); //使用lambda表达式，要用return返回类型信息

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = words.map(w -> Tuple2.of(w, 1)).returns(Types.TUPLE(Types.STRING, Types.INT));

        SingleOutputStreamOperator<Tuple2<String, Integer>> summed = wordAndOne.keyBy(0).sum(1);

        summed.print();

        env.execute();

    }
}
```

##2.Source

**基于Socket网络端口的Source**

>socketTextStream(String hostname, int port, String delimiter, long maxRetry)，这个重载的方法可以指定行分隔符和最大重新连接次数。这两个参数，默认行分隔符是”\n”，最大重新连接次数为0
```
public class SocketSource {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-11
     * Time: 20:52
     */

    //socketTextStream创建的DataStream，不论怎样，并行度永远是1
    public static void main(String[] args) throws Exception {
        //local模式默认的并行度是当前机器的逻辑核的数量
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        System.out.println("执行环境默认分区" + environment.getParallelism());
        //执行环境默认分区8
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        System.out.println("source的并行度" + source.getParallelism());
        //source的并行度1
        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = source.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                for (String word : s.split("\\s+")) {
                    collector.collect(Tuple2.of(word, 1));
                }
            }
        });
        //获取分区
        System.out.println("flatmap的分区" + flatMap.getParallelism());
        //flatmap的分区8
        flatMap.print();
        environment.execute("job");
    }
}
```
**有UI**的
```
public class Test2 {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-11
     * Time: 21:46
     */
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        //自己指定UI端口
        configuration.setInteger("rest.port",8000);
        StreamExecutionEnvironment ui = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);
        DataStreamSource<String> source = ui.socketTextStream("localhost", 8888);
        source.print();
        ui.execute("job");
    }
}
```

**基于集合的Source**
**有限数据流**

```
public class Test2 {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-11
     * Time: 21:46
     */
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        //自己指定UI端口
        configuration.setInteger("rest.port",8000);
        StreamExecutionEnvironment ui = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);
        DataStreamSource<Integer> source = ui.fromCollection(Arrays.asList(1, 2, 3, 4, 5, 6, 7));
        //有限数据流
        source.print();
        ui.execute("job");
    }
}
```

```
public class Test2 {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-11
     * Time: 21:46
     */
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        //自己指定UI端口
        configuration.setInteger("rest.port",8000);
        StreamExecutionEnvironment ui = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);
        DataStreamSource<Integer> source = ui.fromElements(1, 2, 3, 4, 5, 6, 7);
        //有限数据流
        source.print();
        ui.execute("job");
    }
}
```

上面的三种都是单并行的source
>非并行的Source它的并行度只能为1，即用来读取外部数据源的Source只有一个实例，在读取大量数据时效率比较低，通常是用来做一些实验或测试，例如Flink的Socket网络端口读取数据的Source就是一个非并行的Source；并行的Source它的并行度可以是1到多个，即用来读取外部数据源的Source可以有一个到多个实例（在分布式计算中，并行度是影响吞吐量一个非常重要的因素，在计算资源足够的前提下，并行度越大，效率越高）。例如Kafka Source就是并行的Source


下面的是多并行的source

```
单并行的底层都是setParallelism(1)
return addSource(function, "Collection Source", typeInfo, 
Boundedness.BOUNDED).setParallelism(1);
```
```
//多并行
RichParallelSourceFunction<T>
```

```
public class Test2 {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-11
     * Time: 21:46
     */
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        //自己指定UI端口
        configuration.setInteger("rest.port",8000);
        StreamExecutionEnvironment ui = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);
        DataStreamSource<Long> source = ui.fromParallelCollection(new NumberSequenceIterator(1L, 10L), // 生成数组的range
                Long.class);//输出数据的类型)
        source.print();
        ui.execute("job");
    }
}
```
**readFile**
> readFile(FileInputFormat inputFormat, String filePath) 方法可以指定读取文件的FileInputFormat 格式，其中一个重载的方法readFile(FileInputFormat inputFormat, String filePath, FileProcessingMode watchType, long interval) 可以指定FileProcessingMode，它有两个枚举类型分别是PROCESS_ONCE和PROCESS_CONTINUOUSLY模式，PROCESS_ONCE模式Source只读取文件中的数据一次，读取完成后，程序退出。PROCESS_CONTINUOUSLY模式Source会一直监听指定的文件，如果使用该模式，需要指定检测该文件是否发生变化的时间间隔，但是使用这种模式，文件的内容发生变化后，会将以前的内容和新的内容全部都读取出来，进而造成数据重复读取,哪个文件发生了改变就重新读取哪个文件。

```
public class Test3 {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-11
     * Time: 22:34
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        String path = "file:///C:\\Users\\hp\\IdeaProjects\\Flink_Java\\src\\main\\resources";
        DataStreamSource<String> readFile = environment.readFile(new TextInputFormat(null), path, FileProcessingMode.PROCESS_CONTINUOUSLY, 200);
        readFile.print();
        environment.execute("job");

    }
}
```

**readTextFile**

>readTextFile(String filePath) 可以从指定的目录或文件读取数据，默认使用的是TextInputFormat格式读取数据，还有一个重载的方法readTextFile(String filePath, String charsetName)可以传入读取文件指定的字符集，默认是UTF-8编码。该方法是一个有限的数据源，数据读完后，程序就会退出，不能一直运行。该方法底层调用的是readFile方法，FileProcessingMode为PROCESS_ONCE

```
public class Test4 {
/**
 * Created with IntelliJ IDEA.
 * Description: 
 * User: 
 * Date: 2021-01-11
 * Time: 22:55
 */
public static void main(String[] args) throws Exception {
    StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
    String path = "file:///C:\\Users\\hp\\IdeaProjects\\Flink_Java\\src\\main\\resources";
    DataStreamSource<String> streamSource = environment.readTextFile(path);
    streamSource.print();
    environment.execute("job");
}
}
```
**第三方Connector Source**

>在现实生产环境中，为了保证flink可以高效地读取数据源中的数据，通常是跟一些分布式消息中件结合使用，例如Apache Kafka。Kafka的特点是分布式、多副本、高可用、高吞吐、可以记录偏移量等。Flink和Kafka整合可以高效的读取数据，并且可以保证Exactly Once（精确一次性语义）

//添加依赖
```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.12</artifactId>
    <version>1.12.0</version>
</dependency>
```

```
public class Test5 {
/**
 * Created with IntelliJ IDEA.
 * Description: 
 * User: 
 * Date: 2021-01-11
 * Time: 23:03
 */
public static void main(String[] args) throws Exception {
    Configuration configuration = new Configuration();
    StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);

    //设置Kafka相关参数
    Properties properties = new Properties();//设置Kafka的地址和端口
    properties.setProperty("bootstrap.servers", "linux03:9092,linux04:9092,linux05:9092");
    //读取偏移量策略：如果没有记录偏移量，就从头读，如果记录过偏移量，就接着读
    properties.setProperty("auto.offset.reset", "earliest");
    //设置消费者组ID
    properties.setProperty("group.id", "g1");
    //没有开启checkpoint，让flink提交偏移量的消费者定期自动提交偏移量
    properties.setProperty("enable.auto.commit", "true");
    //创建FlinkKafkaConsumer并传入相关参数
    FlinkKafkaConsumer<String> kafkaConsumer = new FlinkKafkaConsumer<>(
            "first", //要读取数据的Topic名称
            new SimpleStringSchema(), //读取文件的反序列化Schema
            properties //传入Kafka的参数
    );
    //使用addSource添加kafkaConsumer

    DataStreamSource<String> lines = env.addSource(kafkaConsumer);

    lines.print();

    env.execute();
}
}
```

>目前这种方式无法保证Exactly Once，Flink的Source消费完数据后，将偏移量定期的写入到Kafka的一个特殊的topic中，这个topic就是__consumer_offset，这种方式虽然可以记录偏移量，但是无法保证Exactly Once.

**自定义Source**
>Flink的DataStream API可以让开发者根据实际需要，灵活的自定义Source，本质上就是定义一个类，实现SourceFunction这个接口，实现run方法和cancel方法。run方法中实现的就是获取数据的逻辑，然后调用SourceContext的collect方法，将获取的数据收集起来，这样就返回了一个新的DataStreamSource，但是如果只实现这个接口，该Source只能是一个非并行的Source。在生产环境，通常是希望Source可以并行的读取数据，这样读取数据的速度才更快，所以最好的方式是实现ParallelSourceFunction接口或继承RichParallelSourceFunction这个抽象类，同样实现实现run方法和cancel方法，这样该Source就是一个可以并行的Source了。其实所有的Source底层都是调用该的方法
```
public class MyParallelSource extends RichParallelSourceFunction<String> {
    private int i = 1; //定义一个int类型的变量，从1开始
    private boolean flag = true; //定义一个flag标标志
    //run方法就是用来读取外部的数据或产生数据的逻辑
    @Override
    public void run(SourceContext<String> ctx) throws Exception {
        //满足while循环的条件，就将数据通过SourceContext收集起来
        while (i <= 10 && flag) {
            Thread.sleep(1000); //为避免太快，睡眠1秒
            ctx.collect(“data：” + i++); //将数据通过SourceContext收集起来
        }
    }
    //cancel方法就是让Source停止
    @Override
    public void cancel() {
        //将flag设置成false，即停止Source
        flag = false;
    }
}
```
```
public class CustomSource03 {

    public static void main(String[] args) throws Exception {

        Configuration configuration = new Configuration();
        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);
        env.setParallelism(3); //设置Env的并行度

        DataStreamSource<String> streamSource = env.addSource(new MySource3());

        System.out.println("这个自定义的MySource2的并行度：" + streamSource.getParallelism());

        streamSource.print();

        env.execute();

    }


    //继承RichParallelSourceFunction抽象类的Source, 是多并行的Source
    public static class MySource3 extends RichParallelSourceFunction<String> {

        public MySource3() {
            System.out.println("constructor invoked");
        }
        //1.调用MySource3构造方法
        //2.调用open方法，调用一次
        //3.调用run方法
        //4.调用cancel方法停止
        //5.调用close方法释放资源

        private boolean flag = true;

        @Override
        public void open(Configuration parameters) throws Exception {
            int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
            System.out.println("subTask: " + indexOfThisSubtask + " open method invoked");
        }

        @Override
        public void close() throws Exception {
            int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
            System.out.println("subTask: " + indexOfThisSubtask + " close method invoked");
        }

        @Override
        public void run(SourceContext<String> ctx) throws Exception {
            int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
            System.out.println("subTask: " + indexOfThisSubtask + " run method invoked");
            while (flag) {
                ctx.collect("subTask: " + indexOfThisSubtask + " " + UUID.randomUUID().toString());
                Thread.sleep(2000);
            }
        }

        @Override
        public void cancel() {
            int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
            System.out.println("subTask: " + indexOfThisSubtask + " cancel method invoked");
            flag = false;
        }
    }
}
```
