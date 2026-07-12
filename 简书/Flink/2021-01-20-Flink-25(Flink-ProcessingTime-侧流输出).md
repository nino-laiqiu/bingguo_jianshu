## 1.ProcessingTime
```
public class ProcessingTimeTimerDemo {

    public static void main(String[] args) throws Exception{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //spark,1
        //spark,2
        //hadoop,1
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndCount = lines.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String line) throws Exception {
                String[] fields = line.split(",");
                return Tuple2.of(fields[0], Integer.parseInt(fields[1]));
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndCount.keyBy(t -> t.f0);

        keyed.process(new KeyedProcessFunction<String, Tuple2<String, Integer>, Tuple2<String, Integer>>() {

            @Override
            public void processElement(Tuple2<String, Integer> value, Context ctx, Collector<Tuple2<String, Integer>> out) throws Exception {
                //获取当前的ProcessingTime
                long currentProcessingTime = ctx.timerService().currentProcessingTime();

                System.out.println("当前时间：" + currentProcessingTime + "，定时器触发的时间：" + (currentProcessingTime + 30000));

                //将当前的ProcessingTime + 30 秒，注册一个定时器
                ctx.timerService().registerProcessingTimeTimer(currentProcessingTime + 30000);


            }

            //当闹钟到了指定的时间，就执行onTimer方法
            @Override
            public void onTimer(long timestamp, OnTimerContext ctx, Collector<Tuple2<String, Integer>> out) throws Exception {
                System.out.println("定时器执行了：" + timestamp);
            }
        }).print();

        env.execute();

    }
}
```

```
public class ProcessingTimeTimer {

    public static void main(String[] args) throws Exception{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //spark,1
        //spark,2
        //hadoop,1
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndCount = lines.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String line) throws Exception {
                String[] fields = line.split(",");
                return Tuple2.of(fields[0], Integer.parseInt(fields[1]));
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndCount.keyBy(t -> t.f0);

        keyed.process(new KeyedProcessFunction<String, Tuple2<String, Integer>, Tuple2<String, Integer>>() {

            private transient ValueState<Integer> counter;

            @Override
            public void open(Configuration parameters) throws Exception {
                ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>("wc-state", Integer.class);
                counter = getRuntimeContext().getState(stateDescriptor);
            }

            @Override
            public void processElement(Tuple2<String, Integer> value, Context ctx, Collector<Tuple2<String, Integer>> out) throws Exception {
                //获取当前的ProcessingTime

                long currentProcessingTime = ctx.timerService().currentProcessingTime();
                long fireTime = currentProcessingTime - currentProcessingTime % 60000 + 60000;
                //下一分钟
                //如果注册相同数据的TimeTimer，后面的会将前面的覆盖，即相同的timeTimer只会触发一次
                ctx.timerService().registerProcessingTimeTimer(fireTime);

                Integer currentCount = value.f1;
                Integer historyCount = counter.value();
                if(historyCount == null) {
                    historyCount = 0;
                }
                Integer totalCount =  historyCount + currentCount;
                //更新状态
                counter.update(totalCount);

            }

            //当闹钟到了指定的时间，就执行onTimer方法
            @Override
            public void onTimer(long timestamp, OnTimerContext ctx, Collector<Tuple2<String, Integer>> out) throws Exception {
                //定时器触发，输出当前的结果
                Integer value = counter.value();
                String currentKey = ctx.getCurrentKey();
                //输出key，Value
                //如果想要实现类似滚动窗口，不累加类似数据，只是累加当前窗口的数据，就清空状态
                //counter.update(0);
                out.collect(Tuple2.of(currentKey, value));
            }
        }).print();

        env.execute();

    }
}

```

>如果注册相同数据的TimeTimer，后面的会将前面的覆盖，即相同的timeTimer只会触发一次,每来某分钟的数据,就会生成一个定时器,当闹钟到了指定的时间，就执行onTimer方法,processElement方法的作用是累加状态;如果想要实现类似滚动窗口，不累加类似数据，只是累加当前窗口的数据，就清空状态


```
public class ProcessingTimeTimerDemo02 {

    public static void main(String[] args) throws Exception{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //spark,1
        //spark,2
        //hadoop,1
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndCount = lines.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String line) throws Exception {
                String[] fields = line.split(",");
                return Tuple2.of(fields[0], Integer.parseInt(fields[1]));
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndCount.keyBy(t -> t.f0);

        keyed.process(new KeyedProcessFunction<String, Tuple2<String, Integer>, Tuple2<String, Integer>>() {

            private transient ValueState<Integer> counter;

            @Override
            public void open(Configuration parameters) throws Exception {
                ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>("wc-state", Integer.class);
                counter = getRuntimeContext().getState(stateDescriptor);
            }

            @Override
            public void processElement(Tuple2<String, Integer> value, Context ctx, Collector<Tuple2<String, Integer>> out) throws Exception {
                //获取当前的ProcessingTime

                long currentProcessingTime = ctx.timerService().currentProcessingTime();
                long fireTime = currentProcessingTime - currentProcessingTime % 60000 + 60000;
                //下一分钟
                //如果注册相同数据的TimeTimer，后面的会将前面的覆盖，即相同的timeTimer只会触发一次
                ctx.timerService().registerProcessingTimeTimer(fireTime);

                Integer currentCount = value.f1;
                Integer historyCount = counter.value();
                if(historyCount == null) {
                    historyCount = 0;
                }
                Integer totalCount =  historyCount + currentCount;
                //更新状态
                counter.update(totalCount);

            }

            //当闹钟到了指定的时间，就执行onTimer方法
            @Override
            public void onTimer(long timestamp, OnTimerContext ctx, Collector<Tuple2<String, Integer>> out) throws Exception {
                //定时器触发，输出当前的结果
                Integer value = counter.value();
                String currentKey = ctx.getCurrentKey();
                //输出key，Value
                //如果想要实现类似滚动窗口，不累加类似数据，只是累加当前窗口的数据，就清空状态
                //counter.update(0);
                out.collect(Tuple2.of(currentKey, value));
            }
        }).print();

        env.execute();

    }
}
```

##2.侧流输出

SideOutput奇数与偶数的侧流输出
```
public class SideOutput  {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        OutputTag<String> tag = new OutputTag<String>("ou") {
        };

        OutputTag<String> tag1 = new OutputTag<String>("j") {
        };

        OutputTag<String> tag2 = new OutputTag<String>("no") {
        };

        SingleOutputStreamOperator<String> process = source.process(new ProcessFunction<String, String>() {
            @Override
            public void processElement(String value, Context ctx, Collector<String> out) throws Exception {
                int i = Integer.parseInt(value);
                try {
                    if (i % 2 == 0) {
                        ctx.output(tag, value);
                    } else {
                        ctx.output(tag1, value);
                    }
                } catch (Exception e) {
                    ctx.output(tag2, value);
                }
                out.collect(value);
            }
        });

        process.getSideOutput(tag).print("o");
        process.getSideOutput(tag1).print("j");
        process.print();
        environment.execute();

    }
}
```
##3.ProcessFunction

累加当前窗口的数据，并与历史数据进行累加

event time的触发时间
1、watermark时间 >= window_end_time 
2、在[window_start_time,window_end_time)区间中有数据存在，注意是左闭右开的区间

```
public class WindowAggregateFunctionDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());

        env.enableCheckpointing(10000);
        //1000,spark,3
        //1200,spark,5
        //2000,hadoop,2
        //socketTextStream返回的DataStream并行度为1
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<String> dataWithWaterMark = lines.assignTimestampsAndWatermarks(WatermarkStrategy
                .<String>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner((line, timestamp) -> Long.parseLong(line.split(",")[0])));


        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndCount = dataWithWaterMark.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                String[] fields = value.split(",");
                return Tuple2.of(fields[1], Integer.parseInt(fields[2]));
            }
        });

        //调用keyBy
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndCount.keyBy(t -> t.f0);

        OutputTag<Tuple2<String, Integer>> lateDataTag = new OutputTag<Tuple2<String, Integer>>("late-data") {
        };

        //NonKeyd Window： 不调用KeyBy，然后调用windowAll方法，传入windowAssinger
        // Keyd Window： 先调用KeyBy，然后调用window方法，传入windowAssinger
        WindowedStream<Tuple2<String, Integer>, String, TimeWindow> windowed = keyed
                .window(TumblingEventTimeWindows.of(Time.seconds(5)));

        //如果直接调用sum或reduce，只会聚合窗口内的数据，不去跟历史数据进行累加
        //需求：可以在窗口内进行增量聚合，并且还可以与历史数据进行聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = windowed.aggregate(new MyAggFunc(), new MyWindowFunc());

        result.print();

        env.execute();

    }


    private static class MyAggFunc implements AggregateFunction<Tuple2<String, Integer>, Integer, Integer> {

        //创建一个初始值
        @Override
        public Integer createAccumulator() {
            return 0;
        }

        //数据一条数据，与初始值或中间累加的结果进行聚合
        @Override
        public Integer add(Tuple2<String, Integer> value, Integer accumulator) {
            return value.f1 + accumulator;
        }

        //返回的结果
        @Override
        public Integer getResult(Integer accumulator) {
            return accumulator;
        }

        //如果使用的是非SessionWindow，可以不实现
        @Override
        public Integer merge(Integer a, Integer b) {
            return null;
        }
    }


    private static class MyWindowFunc extends ProcessWindowFunction<Integer, Tuple2<String, Integer>, String, TimeWindow> {

        private transient ValueState<Integer> sumState;

        @Override
        public void open(Configuration parameters) throws Exception {
            ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>("wc-state", Integer.class);
            sumState = getRuntimeContext().getState(stateDescriptor);
        }

        @Override
        public void process(String key, Context context, Iterable<Integer> elements, Collector<Tuple2<String, Integer>> out) throws Exception {

            Integer historyCount = sumState.value();
            if (historyCount == null) {
                historyCount = 0;
            }
            //获取到窗口聚合后输出的结果
            Integer windowCount = elements.iterator().next();
            Integer totalCount = historyCount + windowCount;
            //更新状态
            sumState.update(totalCount);

            //输出
            out.collect(Tuple2.of(key, totalCount));
        }
    }
}
```

```
//使用侧流输出获取窗口迟到的数据
public class WindowLateDateDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());

        //1000,spark,3
        //1200,spark,5
        //2000,hadoop,2
        //socketTextStream返回的DataStream并行度为1
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<String> dataWithWaterMark = lines.assignTimestampsAndWatermarks(WatermarkStrategy
                .<String>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner((line, timestamp) -> Long.parseLong(line.split(",")[0])));


        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndCount = dataWithWaterMark.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                String[] fields = value.split(",");
                return Tuple2.of(fields[1], Integer.parseInt(fields[2]));
            }
        });

        //调用keyBy
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndCount.keyBy(t -> t.f0);

        OutputTag<Tuple2<String, Integer>> lateDataTag = new OutputTag<Tuple2<String, Integer>>("late-data") {
        };

        //NonKeyd Window： 不调用KeyBy，然后调用windowAll方法，传入windowAssinger
        // Keyd Window： 先调用KeyBy，然后调用window方法，传入windowAssinger
        WindowedStream<Tuple2<String, Integer>, String, TimeWindow> windowed = keyed
                .window(TumblingEventTimeWindows.of(Time.seconds(5)));

        //如果直接调用sum或reduce，只会聚合窗口内的数据，不去跟历史数据进行累加
        //需求：可以在窗口内进行增量聚合，并且还可以与历史数据进行聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = windowed.reduce(new MyReduceFunc(), new MyWindowFunc());

        result.print();

        env.execute();

    }

    public static class MyReduceFunc implements ReduceFunction<Tuple2<String, Integer>> {

        @Override
        public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
            value1.f1 = value1.f1 + value2.f1;
            return value1;
        }
    }

    public static class MyWindowFunc extends ProcessWindowFunction<Tuple2<String, Integer>, Tuple2<String, Integer>, String, TimeWindow> {

        private transient ValueState<Integer> sumState;

        @Override
        public void open(Configuration parameters) throws Exception {
            ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>("wc-state", Integer.class);
            sumState = getRuntimeContext().getState(stateDescriptor);
        }

        @Override
        public void process(String key, Context context, Iterable<Tuple2<String, Integer>> elements, Collector<Tuple2<String, Integer>> out) throws Exception {

            Integer historyCount = sumState.value();
            if (historyCount == null) {
                historyCount = 0;
            }
            //获取到窗口聚合后输出的结果
            Tuple2<String, Integer> tp = elements.iterator().next();
            Integer windowCount = tp.f1;
            Integer totalCount = historyCount + windowCount;
            //更新状态
            sumState.update(totalCount);
            tp.f1 = totalCount;
            //输出
            out.collect(tp);
        }
    }
}
```


```

public class WindowProcessFunctionDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());

        env.enableCheckpointing(10000);
        //1000,spark,3
        //1200,spark,5
        //2000,hadoop,2
        //socketTextStream返回的DataStream并行度为1
        DataStreamSource<String> lines = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<String> dataWithWaterMark = lines.assignTimestampsAndWatermarks(WatermarkStrategy
                .<String>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner((line, timestamp) -> Long.parseLong(line.split(",")[0])));


        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndCount = dataWithWaterMark.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                String[] fields = value.split(",");
                return Tuple2.of(fields[1], Integer.parseInt(fields[2]));
            }
        });

        //调用keyBy
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndCount.keyBy(t -> t.f0);

        OutputTag<Tuple2<String, Integer>> lateDataTag = new OutputTag<Tuple2<String, Integer>>("late-data") {
        };

        //NonKeyd Window： 不调用KeyBy，然后调用windowAll方法，传入windowAssinger
        // Keyd Window： 先调用KeyBy，然后调用window方法，传入windowAssinger
        WindowedStream<Tuple2<String, Integer>, String, TimeWindow> windowed = keyed
                .window(TumblingEventTimeWindows.of(Time.seconds(5)));

        //如果直接调用sum或reduce，只会聚合窗口内的数据，不去跟历史数据进行累加
        //需求：可以在窗口内进行增量聚合，并且还可以与历史数据进行聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = windowed.process(new ProcessWindowFunction<Tuple2<String, Integer>, Tuple2<String, Integer>, String, TimeWindow>() {
            //窗口触发，才会调用process方法，该方法可以获取窗口内的全量获取窗口的数据，数据是缓存到windowstate中的
            @Override
            public void process(String s, Context context, Iterable<Tuple2<String, Integer>> elements, Collector<Tuple2<String, Integer>> out) throws Exception {

                for (Tuple2<String, Integer> element : elements) {
                    out.collect(element);
                }
            }
        });

        result.print();

        env.execute();

    }

}
```
