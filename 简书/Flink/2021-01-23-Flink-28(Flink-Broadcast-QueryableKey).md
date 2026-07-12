##1.BroadcastState
广播变量的使用,关联维度
```
public class BrocastStateDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //mysql维度表 -> canal -> kafka -> flinkKafkaSource
        //INSERT,1,浏览
        //INSERT,2,参与
        //INSERT,3,消费
        //UPDATE,3,退出
        //DELETE,3,删除
        DataStreamSource<String> dicDataStream = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<Tuple3<String, String, String>> dicTupleStream = dicDataStream.map(new MapFunction<String, Tuple3<String, String, String>>() {
            @Override
            public Tuple3<String, String, String> map(String value) throws Exception {
                String[] fields = value.split(",");
                return Tuple3.of(fields[0], fields[1], fields[2]);
            }
        });

        MapStateDescriptor<String, String> broadcastMapStateDescriptor = new MapStateDescriptor<>("dic-state", String.class, String.class);

        //返回一个广播的流
        BroadcastStream<Tuple3<String, String, String>> broadcastStream = dicTupleStream.broadcast(broadcastMapStateDescriptor);

        //user1,A,1
        DataStreamSource<String> actDataStream = env.socketTextStream("localhost", 9999);

        //对数据进行整理
        SingleOutputStreamOperator<Tuple3<String, String, String>> tpDataStream = actDataStream.map(new MapFunction<String, Tuple3<String, String, String>>() {
            @Override
            public Tuple3<String, String, String> map(String line) throws Exception {
                String[] fields = line.split(",");
                return Tuple3.of(fields[0], fields[1], fields[2]);
            }
        });

        KeyedStream<Tuple3<String, String, String>, Tuple2<String,String>> keyedStream = tpDataStream.keyBy(t -> Tuple2.of(t.f1, t.f2), TypeInformation.of(new TypeHint<Tuple2<String,String>>() {}));

        keyedStream
                .connect(broadcastStream)
                .process(new MyBroadcastProcessFunc(broadcastMapStateDescriptor))
                .print();

        env.execute();
    }


    private static class MyBroadcastProcessFunc extends KeyedBroadcastProcessFunction<Tuple2<String, String>, Tuple3<String, String, String>, Tuple3<String, String, String>, Tuple4<String, String, Long, Long>> {

        private MapStateDescriptor<String, String> broadcastMapStateDescriptor;

        public MyBroadcastProcessFunc(){}

        public MyBroadcastProcessFunc(MapStateDescriptor<String, String> broadcastMapStateDescriptor) {
            this.broadcastMapStateDescriptor = broadcastMapStateDescriptor;
        }

        private transient ValueState<Long> actCountState;
        private transient ValueState<HashSet<String>> userDisState;

        @Override
        public void open(Configuration parameters) throws Exception {
            ValueStateDescriptor<Long> stateDescriptor1 = new ValueStateDescriptor<>("ac-count", Long.class);
            actCountState = getRuntimeContext().getState(stateDescriptor1);
            ValueStateDescriptor<HashSet<String>> stateDescriptor2 = new ValueStateDescriptor<HashSet<String>>("dis-ac-count", TypeInformation.of(new TypeHint<HashSet<String>>(){}));
            userDisState = getRuntimeContext().getState(stateDescriptor2);
        }

        /**
         * 处理输入的每一条活动的数据（事实）就是你按照活动ID和活动分类key的 keyedStream
         * 例如 user1,A,1
         * @param value
         * @param ctx
         * @param out
         * @throws Exception
         */
        @Override
        public void processElement(Tuple3<String, String, String> value, ReadOnlyContext ctx, Collector<Tuple4<String, String, Long, Long>> out) throws Exception {

            //使用ReadOnlyContext获取ReadOnlyBroadcastState
            //1,参与
            ReadOnlyBroadcastState<String, String> broadcastState = ctx.getBroadcastState(broadcastMapStateDescriptor);

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
            String eventId = value.f2;
            String eventName = broadcastState.get(eventId);

            out.collect(Tuple4.of(value.f1, eventName, totalCount, (long) disUserSet.size()));
        }

        /**
         * 处理输入的每一条维度数据，BroadcastStream<Tuple3<String, String, String>>
         * 例如：INSERT,1,浏览
         * @param value
         * @param ctx
         * @param out
         * @throws Exception
         */
        @Override
        public void processBroadcastElement(Tuple3<String, String, String> value, Context ctx, Collector<Tuple4<String, String, Long, Long>> out) throws Exception {
            //INSERT、UPDATE、DELETE
            String type = value.f0;
            String eventId = value.f1;
            String eventName = value.f2;
            //获取广播状态
            BroadcastState<String, String> broadcastState = ctx.getBroadcastState(broadcastMapStateDescriptor);
            //将广播的状态存起来或更新、删除
            if("DELETE".equals(type)) {
                broadcastState.remove(eventId);
            } else {
                broadcastState.put(eventId, eventName);
            }
        }
    }
}

```

##2.QueryableKeyedState

>https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/dev/stream/state/queryable_state.html

```
public class QueryStateClientDemo {
    public static void main(String[] args) throws Exception {
        QueryableStateClient client = new QueryableStateClient("localhost", 9069);
        //初始化状态数据或恢复历史状态数据
        ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>(
                "wc-state", //指定状态描述器的名称
                Integer.class //存储数据的类型
        );
        CompletableFuture<ValueState<Integer>> resultFuture = client.getKvState(
                JobID.fromHexString("23ba9de14e4135f6eb91d62af859c51e"), //job的ID
                "my-query-name", //可查询的state的名称
                "spark", //查询的key
                BasicTypeInfo.STRING_TYPE_INFO,
                stateDescriptor);
        resultFuture.thenAccept(response -> {
            try {
                Integer res = response.value();
                System.out.println(res);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        Thread.sleep(5000);
    }
}
```

```
public class QueryableKeyedStateDemo {

    public static class MyQueryStateRichMapFunction extends
            RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>> {
        //使用flink的ValueState保存单词对应的次数
        private transient ValueState<Integer> countState;

        @Override
        public void open(Configuration parameters) throws Exception {
            //初始化状态数据或恢复历史状态数据
            ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>(
                    "wc-state", //指定状态描述器的名称
                    Integer.class //存储数据的类型
            );
            stateDescriptor.setQueryable("my-query-name"); //设置状态可以查询，并指定状态查询名称
            //获取getRuntimeContext并根据状态描述器取出对应的State
            countState = getRuntimeContext().getState(stateDescriptor);
        }

        @Override
        public Tuple2<String, Integer> map(Tuple2<String, Integer> tp) throws Exception {
            Integer current = tp.f1;  //获取当前输入的单词的次数
            Integer counts = countState.value(); //获取历史的次数
            if (counts == null) { //历史次数是否为空
                counts = 0; //如果历史床头为空，初始值设置为0
            }
            int total = current + counts; //将当前的次数和历史次数进行累加
            countState.update(total); //更新状态
            tp.f1 = total; //将累加后的次数放入到tp
            return tp; //返回tp
        }
    }

    public static void main(String[] args) throws Exception {
        ParameterTool params = ParameterTool.fromArgs(args);
        Configuration config = params.getConfiguration();
        //启用Queryable State服务相关参赛
        config.setInteger("rest.port", 8081);
        config.setBoolean(QueryableStateOptions.ENABLE_QUERYABLE_STATE_PROXY_SERVER, true);
        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(config);

        //开启checkpoint【检查点，可以将任务计算的中间结果（状态数据保存起来）】
        env.enableCheckpointing(30000);
        //设置重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, Time.seconds(5)));
        //从指定的Socket地址和端口创建DataStream
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);
        //将单词和1组合，放入到Tuple2中
        DataStream<Tuple2<String, Integer>> wordAndOne = lines.map(
                new MapFunction<String, Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> map(String word) throws Exception {
                        return Tuple2.of(word, 1);
                    }
                }
        );
        //按照单词进行分组
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndOne.keyBy(t -> t.f0);
        //分完组，相同的key会进到相同的组，每一个组都维护自己的状态数据
        DataStream<Tuple2<String, Integer>> result = keyed.map(new MyQueryStateRichMapFunction());
        result.print();
        env.execute();
    }
}

```
