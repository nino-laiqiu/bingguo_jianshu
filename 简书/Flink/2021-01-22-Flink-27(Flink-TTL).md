##1.TTL
> 我们都知道，状态是不断累积的，占用的空间也就不断增大，对于作业中定义了超长的时间窗口，或者在动态表上应用了无限范围的 GROUP BY 语句，以及执行了没有时间窗口限制的双流 JOIN 等等操作，就容易出现OOM, Flink引入了TTL特性，作用就是对作业中的state进行清理

>TTL 的更新策略（默认是 OnCreateAndWrite）：
StateTtlConfig.UpdateType.OnCreateAndWrite - 仅在创建和写入时更新
StateTtlConfig.UpdateType.OnReadAndWrite - 读取时也更新
数据在过期但还未被清理时的可见性配置如下（默认为 NeverReturnExpired):
StateTtlConfig.StateVisibility.NeverReturnExpired - 不返回过期数据
StateTtlConfig.StateVisibility.ReturnExpiredIfNotCleanedUp - 会返回过期但未清理的数据
NeverReturnExpired 情况下，过期数据就像不存在一样，不管是否被物理删除。这对于不能访问过期数据的场景下非常有用，比如敏感数据。 ReturnExpiredIfNotCleanedUp 在数据被物理删除前都会返回。

>https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/dev/stream/state/state.html#%E7%8A%B6%E6%80%81%E6%9C%89%E6%95%88%E6%9C%9F-ttl

```
//设置ValueState的存活时间
public class KeyedStateTTLDemo {

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
                //定义一个状态TTLCOnfig
                StateTtlConfig ttlConfig = StateTtlConfig.newBuilder(Time.seconds(10))
                        .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
                        .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
                        .build();

                //想使用状态，先定义一个状态描述器（State的类型，名称）
                ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>("wc-desc", Integer.class);
                //关联状态描述器
                stateDescriptor.enableTimeToLive(ttlConfig);
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


**案例:去重人数**
//如果人数过多,状态太大则采用TTL或者布隆过滤器

```
用户ID,活动ID,事件类型(1，代表浏览；2代表参与)
user1,A,1
user1,A,1
user1,A,2
user2,A,1
user2,A,2
user3,A,2
user1,B,1
user1,B,2
user2,B,1
user3,A,1
user3,A,1
user3,B,1
user4,A,1
user4,A,1

题目：统计各个活动，各种事件的人数和次数
````

```

public class ActivityCount {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);
        
        //对数据进行整理
        SingleOutputStreamOperator<Tuple3<String, String, String>> tpDataStream = lines.map(new MapFunction<String, Tuple3<String, String, String>>() {
            @Override
            public Tuple3<String, String, String> map(String line) throws Exception {
                String[] fields = line.split(",");
                return Tuple3.of(fields[0], fields[1], fields[2]);
            }
        });
        
        KeyedStream<Tuple3<String, String, String>, Tuple2<String, String>> keyed = tpDataStream.keyBy(t -> Tuple2.of(t.f1, t.f2), TypeInformation.of(new TypeHint<Tuple2<String, String>>() {}));


        SingleOutputStreamOperator<Tuple4<String, String, Long, Long>> result = keyed.process(new ActivityCountFunction());

        result.print();

        env.execute();
    }
}
```
```
/**
 * 统计次数和人数的
 * 人数要去重
 */
public class ActivityCountFunction extends KeyedProcessFunction<Tuple2<String, String>, Tuple3<String, String, String>, Tuple4<String, String, Long, Long>> {

    private transient ValueState<Long> actCountState;
    private transient ValueState<HashSet<String>> userDisState;

    @Override
    public void open(Configuration parameters) throws Exception {
        ValueStateDescriptor<Long> stateDescriptor1 = new ValueStateDescriptor<>("ac-count", Long.class);
        actCountState = getRuntimeContext().getState(stateDescriptor1);
        ValueStateDescriptor<HashSet<String>> stateDescriptor2 = new ValueStateDescriptor<HashSet<String>>("dis-ac-count", TypeInformation.of(new TypeHint<HashSet<String>>(){}));
        userDisState = getRuntimeContext().getState(stateDescriptor2);
    }
    @Override
    public void processElement(Tuple3<String, String, String> value, Context ctx, Collector<Tuple4<String, String, Long, Long>> out) throws Exception {

        //计算次数
        Long historyCount = actCountState.value();
        if(historyCount == null) {
            historyCount = 0L;
        }
        Long totalCount = historyCount + 1;
        actCountState.update(totalCount);
        //计算人数
        HashSet<String> disUserSet = userDisState.value();
        if (disUserSet == null) {
            disUserSet = new HashSet<>();
        }
        disUserSet.add(value.f0);
        userDisState.update(disUserSet);
        //输出结果
        out.collect(Tuple4.of(value.f1, value.f2, totalCount, (long) disUserSet.size()));
    }
}
```

