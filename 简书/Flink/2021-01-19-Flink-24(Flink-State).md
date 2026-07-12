
##1.使用Hashmap模拟KeyedState

```
//设置重启策略
public class MyKeyedState01 {

    public static void main(String[] args) throws Exception{

        //创建Flink流计算执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(10000);
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));

        //创建DataStream
        //Source
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        //调用Transformation开始
        //调用Transformation
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = lines.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String line, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] words = line.split(" ");
                for (String word : words) {
                    if("error".equals(word)) {
                        throw new RuntimeException("出现异常了！！！！！");
                    }
                    //new Tuple2<String, Integer>(word, 1)
                    collector.collect(Tuple2.of(word, 1));
                }
            }
        });

        //分组
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndOne.keyBy(t -> t.f0);

        keyed.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {

            //存储中间结果的一个集合
            private HashMap<String, Integer> counter = new HashMap<>();

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> input) throws Exception {
                String word = input.f0;
                Integer count = input.f1;
                //从map中取出历史次数
                Integer historyCount = counter.get(word);
                if(historyCount == null) {
                    historyCount = 0;
                }
                int sum = historyCount + count; //当前输入跟历史次数进行累加
                //更新map中的数据
                counter.put(word, sum);
                //输出结果
                return Tuple2.of(word, sum);
            }
        }).print();




        //启动执行
        env.execute("StreamingWordCount");

    }
}
```
##2.实现一个简单的KeyedState

```
/**
 * 自己每个keyBy之后的SubTask中定义一个hashMap保存中间结果
 * 可以定期的将hashmap中的数据持久好到磁盘
 * 并且subTask出现异常重启，在open方法中可以读取磁盘中的文件，恢复历史状态
 * （区别：每个定时器都是在SubTask中定期执行的）
 */
public class MyKeyedState02 {

    public static void main(String[] args) throws Exception{

        //创建Flink流计算执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        
        //要开启,否则从新开始计数
        env.enableCheckpointing(10000);
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));

        //创建DataStream
        //Source
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        //调用Transformation开始
        //调用Transformation
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = lines.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String line, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] words = line.split(" ");
                for (String word : words) {
                    if("error".equals(word)) {
                        throw new RuntimeException("出现异常了！！！！！");
                    }
                    //new Tuple2<String, Integer>(word, 1)
                    collector.collect(Tuple2.of(word, 1));
                }
            }
        });

        //分组
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndOne.keyBy(t -> t.f0);

        keyed.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {

            private HashMap<String, Integer> counter;

            @Override
            public void open(Configuration parameters) throws Exception {
                //初始化hashMap或恢复历史数据
                //获取当前subTask的编号
                int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
                File ckFile = new File("/Users/xing/Desktop/myck/" + indexOfThisSubtask);
                if(ckFile.exists()) {
                    FileInputStream fileInputStream = new FileInputStream(ckFile);
                    ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
                    counter = (HashMap<String, Integer>) objectInputStream.readObject();
                } else {
                   counter = new HashMap<>();
                }
                //简化：直接在当前的subTask中启动一个定时器
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                       while (true) {
                           try {
                               Thread.sleep(10000);
                               if (!ckFile.exists()) {
                                   ckFile.createNewFile();
                               }
                               //将hashMap中的数据持久化到文件中
                               ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(ckFile));
                               objectOutputStream.writeObject(counter);
                               objectOutputStream.flush();
                               objectOutputStream.close();
                           } catch (Exception e) {
                               e.printStackTrace();
                           }
                       }
                    }
                }).start();



            }

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> input) throws Exception {
                String word = input.f0;
                Integer count = input.f1;
                //从map中取出历史次数
                Integer historyCount = counter.get(word);
                if(historyCount == null) {
                    historyCount = 0;
                }
                int sum = historyCount + count; //当前输入跟历史次数进行累加
                //更新map中的数据
                counter.put(word, sum);
                //输出结果
                return Tuple2.of(word, sum);
            }
        }).print();


        //启动执行
        env.execute("StreamingWordCount");

    }
}
```

##3.KeyedState的简单使用
```
/**
 * KeyBy之后，用来存储K-V类型的状态，叫做KeyedState
 *
 * KeyedState种类有：ValueState<T> （value是一个基本类型、集合类型、自定义类型）
 * MapState<小K,V> （存储的是k，v类型）  （大K -> (小K， 小V)）
 * ListState（Value是一个list集合）
 *
 */
public class KeyedStateDemo01 {

    public static void main(String[] args) throws Exception{

        //创建Flink流计算执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(10000);
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));

        //创建DataStream
        //Source
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        //调用Transformation开始
        //调用Transformation
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = lines.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String line, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] words = line.split(" ");
                for (String word : words) {
                    if("error".equals(word)) {
                        throw new RuntimeException("出现异常了！！！！！");
                    }
                    //new Tuple2<String, Integer>(word, 1)
                    collector.collect(Tuple2.of(word, 1));
                }
            }
        });

        //分组
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndOne.keyBy(t -> t.f0);

        keyed.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {

            private transient ValueState<Integer> counter;

            @Override
            public void open(Configuration parameters) throws Exception {
                //想使用状态，先定义一个状态描述器（State的类型，名称）
                ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>("wc-desc", Integer.class);
                //初始化或恢复历史状态
                counter = getRuntimeContext().getState(stateDescriptor);
            }

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> input) throws Exception {
                //String word = input.f0;
                Integer currentCount = input.f1;
                //从ValueState中取出历史次数
                Integer historyCount = counter.value(); //获取当前key对应的value
                if(historyCount == null) {
                    historyCount = 0;
                }
                Integer total = historyCount + currentCount; //累加
                //跟新状态（内存中）
                counter.update(total);
                input.f1 = total; //累加后的次数
                return input;
            }
        }).print();

        //启动执行
        env.execute("StreamingWordCount");

    }
}

```

##4.MapState简单使用

```
/**
 * KeyBy之后，用来存储K-V类型的状态，叫做KeyedState
 *
 * KeyedState种类有：ValueState<T> （value是一个基本类型、集合类型、自定义类型）
 * MapState<小K,V> （存储的是k，v类型）  （大K -> (小K， 小V)）
 * ListState（Value是一个list集合）
 *
 */
public class MapStateDemo01 {

    public static void main(String[] args) throws Exception{

        //创建Flink流计算执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(10000);
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));

        //创建DataStream
        //Source
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        //调用Transformation开始
        //调用Transformation
        SingleOutputStreamOperator<Tuple3<String, String, Double>> tpDataStream = lines.map(new MapFunction<String, Tuple3<String, String, Double>>() {
            @Override
            public Tuple3<String, String, Double> map(String value) throws Exception {
                String[] fields = value.split(",");
                return Tuple3.of(fields[0], fields[1], Double.parseDouble(fields[2]));
            }
        });

        KeyedStream<Tuple3<String, String, Double>, String> keyedStream = tpDataStream.keyBy(t -> t.f0);

        SingleOutputStreamOperator<Tuple3<String, String, Double>> result = keyedStream.process(new KeyedProcessFunction<String, Tuple3<String, String, Double>, Tuple3<String, String, Double>>() {

            private transient MapState<String, Double> mapState;

            @Override
            public void open(Configuration parameters) throws Exception {
                //定义一个状态描述器
                MapStateDescriptor<String, Double> stateDescriptor = new MapStateDescriptor<String, Double>("kv-state", String.class, Double.class);
                //初始化或恢复历史状态
                mapState = getRuntimeContext().getMapState(stateDescriptor);
            }

            @Override
            public void processElement(Tuple3<String, String, Double> value, Context ctx, Collector<Tuple3<String, String, Double>> out) throws Exception {
                String city = value.f1;
                Double money = value.f2;
                Double historyMoney = mapState.get(city);
                if (historyMoney == null) {
                    historyMoney = 0.0;
                }
                Double totalMoney = historyMoney + money; //累加
                //更新到state中
                mapState.put(city, totalMoney);
                //输出
                value.f2 = totalMoney;
                out.collect(value);
            }
        });

        result.print();

        //启动执行
        env.execute("StreamingWordCount");

    }
}

```

##5.ListState简单使用

```
/**
 * KeyBy之后，用来存储K-V类型的状态，叫做KeyedState
 *
 * ListState（Value是一个list集合）
 *
 */
public class ListStateDemo01 {

    public static void main(String[] args) throws Exception{

        //创建Flink流计算执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(10000);
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));

        //创建DataStream
        //Source
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        //调用Transformation开始
        //调用Transformation
        SingleOutputStreamOperator<Tuple2<String, String>> tpDataStream = lines.map(new MapFunction<String, Tuple2<String, String>>() {
            @Override
            public Tuple2<String, String> map(String value) throws Exception {
                String[] fields = value.split(",");
                return Tuple2.of(fields[0], fields[1]);
            }
        });

        KeyedStream<Tuple2<String, String>, String> keyedStream = tpDataStream.keyBy(t -> t.f0);

        keyedStream.process(new KeyedProcessFunction<String, Tuple2<String, String>, Tuple2<String, List<String>>>() {

            private transient ListState<String> listState;

            @Override
            public void open(Configuration parameters) throws Exception {
                //定义一个状态描述器
                ListStateDescriptor<String> stateDescriptor = new ListStateDescriptor<>("lst-state", String.class);
                //初始化状态或恢复状态
                listState = getRuntimeContext().getListState(stateDescriptor);
            }

            @Override
            public void processElement(Tuple2<String, String> value, Context ctx, Collector<Tuple2<String, List<String>>> out) throws Exception {
                String action = value.f1;
                listState.add(action);
                Iterable<String> iterator = listState.get();
                ArrayList<String> events = new ArrayList<>();
                for (String name : iterator) {
                    events.add(name);
                }
                out.collect(Tuple2.of(value.f0, events));
            }
        }).print();

        //启动执行
        env.execute("StreamingWordCount");

    }
}
```
注意TypeHint嵌套类型的使用
```
/**
 * KeyBy之后，用来存储K-V类型的状态，叫做KeyedState
 *
 * ListState（Value是一个list集合）
 *
 * 不是以ListState，而是使用ValueState，实现ListState的功能
 *
 * ValueState<List<String>>
 *
 */
public class ListStateDemo02 {

    public static void main(String[] args) throws Exception{

        //创建Flink流计算执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(10000);
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));

        //创建DataStream
        //Source
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        //调用Transformation开始
        //调用Transformation
        SingleOutputStreamOperator<Tuple2<String, String>> tpDataStream = lines.map(new MapFunction<String, Tuple2<String, String>>() {
            @Override
            public Tuple2<String, String> map(String value) throws Exception {
                String[] fields = value.split(",");
                return Tuple2.of(fields[0], fields[1]);
            }
        });

        KeyedStream<Tuple2<String, String>, String> keyedStream = tpDataStream.keyBy(t -> t.f0);

        keyedStream.process(new KeyedProcessFunction<String, Tuple2<String, String>, Tuple2<String, List<String>>>() {

            private transient ValueState<List<String>> listState;

            @Override
            public void open(Configuration parameters) throws Exception {
                //定义一个状态描述器
                ValueStateDescriptor<List<String>> listStateDescriptor = new ValueStateDescriptor<>("lst-state", TypeInformation.of(new TypeHint<List<String>>() {}));
                listState = getRuntimeContext().getState(listStateDescriptor);
            }

            @Override
            public void processElement(Tuple2<String, String> value, Context ctx, Collector<Tuple2<String, List<String>>> out) throws Exception {
                String action = value.f1;
                List<String> lst = listState.value();
                if(lst == null) {
                    lst = new ArrayList<String>();
                }
                lst.add(action);
                //更新状态
                listState.update(lst);
                out.collect(Tuple2.of(value.f0, lst));
            }
        }).print();

        //启动执行
        env.execute("StreamingWordCount");

    }
}
```

##6.AtLeastOnceSource

![image.png](https://upload-images.jianshu.io/upload_images/9049859-022a5d1ebab13b17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-78ca5949414b9c06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
public class MyAtLeastOnceSource extends RichParallelSourceFunction<String> implements CheckpointedFunction {


    private String path;

    public MyAtLeastOnceSource(String path) {
        this.path = path;
    }

    private boolean flag = true;
    private Long offset = 0L;
    private transient ListState<Long> listState;
    /*
     * @param context
     * @throws Exception
     * 初始化状态或恢复状态执行一次，在run方法执行之前执行一次
     */
    @Override
    public void initializeState(FunctionInitializationContext context) throws Exception {
        ListStateDescriptor<Long> stateDescriptor = new ListStateDescriptor<>("offset-state", Long.class);
        listState = context.getOperatorStateStore().getListState(stateDescriptor);
        //当前的状态是否已经恢复了
        if(context.isRestored()) {
            //从ListState中恢复偏移量
            Iterable<Long> iterable = listState.get();
            for (Long l : iterable) {
                    offset =+ l;
            }
        }
    }

    @Override
    public void run(SourceContext<String> ctx) throws Exception {
        int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
        RandomAccessFile randomAccessFile = new RandomAccessFile(path + "/" + indexOfThisSubtask + ".txt", "r");
        randomAccessFile.seek(offset); //从指定的位置读取数据
        while (flag) {
            String line = randomAccessFile.readLine();
            if(line != null) {
                line = new String(line.getBytes(Charsets.ISO_8859_1), Charsets.UTF_8);
                synchronized (ctx.getCheckpointLock()) {
                    offset = randomAccessFile.getFilePointer();
                    ctx.collect(indexOfThisSubtask + ".txt : " + line);
                }
            } else {
                Thread.sleep(1000);
            }
        }
    }

    /**
     *
     * @param context
     * @throws Exception
     * 在checkpoint时会支持一次，该方法会周期性的调用
     */
    @Override
    public void snapshotState(FunctionSnapshotContext context) throws Exception {
        //定期的更新OperatorState
        listState.clear();
        listState.add(offset);
    }


    @Override
    public void cancel() {
        flag = false;
    }
}
```
```
public class MyAtLeastOnceSourceDemo {

    public static void main(String[] args) throws Exception{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //可以定期将状态保存到StateBackend（对状态做快照）
        env.enableCheckpointing(30000);
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 5000));


        DataStreamSource<String> lines1 = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<String> errorData = lines1.map(new MapFunction<String, String>() {
            @Override
            public String map(String value) throws Exception {
                if (value.startsWith("error")) {
                    int i = 10 / 0;
                }
                return value;
            }
        });

        DataStreamSource<String> lines2 = env.addSource(new MyAtLeastOnceSource("/Users/xing/Desktop/data"));

        DataStream<String> union = errorData.union(lines2);

        union.print();

        env.execute();


    }
}

```
