##1.数据一致性

>可能未checkpoint程序就中断了,要重启,重新消费kafka的数据,同时更新Redis的数据,所以不是ExactlyOnce

>执行出现无法checkpoint则要把flink和hadoop整合的jar把拷贝到flink的lib目录中

```
/**
 * 当前的程序能不能容错（保证数据的一致性）
 * 当前程序如果可以保证数据的一致性，是使用ExactlyOnce还是AtLeastOnce，使用的是AtLeastOnce
 *
 * KafkaSource：可以记录偏移量，可以将偏移量保存到状态中（OperatorState）
 * keyBy后调用sum：sum有状态（ValueState）
 * RedisSink：使用HSET方法可以将数据覆盖（幂等性）
 *
 *
 */
public class KafkaToRedisWordCount {

    //--topic doit2021 --groupId g02 --redisHost node-3.51doit.cn --redisPwd 123456 --fsBackend hdfs://node-1.51doit.cn:9000/flinkck2021
    public static void main(String[] args) throws Exception{

        //System.setProperty("HADOOP_USER_NAME", "root");
        ParameterTool parameterTool = ParameterTool.fromArgs(args);

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(parameterTool.getLong("chkInterval", 30000)); //可以间内存中的状态持久化到StateBackend
        //设置状态存储的后端
        env.setStateBackend(new FsStateBackend(parameterTool.getRequired("fsBackend")));
        //如果你手动cancel job后，不删除job的checkpoint数据
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        //添加KafkaSource
        //设置Kafka相关参数
        Properties properties = new Properties();//设置Kafka的地址和端口
        properties.setProperty("bootstrap.servers", "linux03:9092,linx04:9092,linux05:9092");
        //读取偏移量策略：如果没有记录偏移量，就从头读，如果记录过偏移量，就接着读
        properties.setProperty("auto.offset.reset", "earliest");
        //设置消费者组ID
        properties.setProperty("group.id", parameterTool.get("groupId"));
        //开启checkpoint，不然让flink的消费（source对他的subtask）自动提交偏移量
        properties.setProperty("enable.auto.commit", "false");
        //创建FlinkKafkaConsumer并传入相关参数
        FlinkKafkaConsumer<String> kafkaConsumer = new FlinkKafkaConsumer<>(
                parameterTool.getRequired("topic"), //要读取数据的Topic名称
                new SimpleStringSchema(), //读取文件的反序列化Schema
                properties //传入Kafka的参数
        );
        kafkaConsumer.setCommitOffsetsOnCheckpoints(false); //设置在checkpoint是不将偏移量保存到kafka特殊的topic中

        //使用addSource添加kafkaConsumer
        DataStreamSource<String> lines = env.addSource(kafkaConsumer);

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = lines.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String line, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] words = line.split(" ");
                for (String word : words) {
                    //new Tuple2<String, Integer>(word, 1)
                    collector.collect(Tuple2.of(word, 1));
                }
            }
        });

        //分组
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndOne.keyBy(t -> t.f0);

        //聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> summed = keyed.sum(1);

        //将聚合后的结果写入到Redis中
        //调用Sink
        //summed.addSink()
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder()
                .setHost(parameterTool.getRequired("redisHost"))
                .setPassword(parameterTool.getRequired("redisPwd"))
                .setDatabase(9).build();

        summed.addSink(new RedisSink<Tuple2<String, Integer>>(conf, new RedisSinkDemo.RedisWordCountMapper()));

        env.execute();


    }

    private static class RedisWordCountMapper implements RedisMapper<Tuple2<String, Integer>> {

        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET, "WORD_COUNT");
        }

        @Override
        public String getKeyFromData(Tuple2<String, Integer> data) {
            return data.f0;
        }

        @Override
        public String getValueFromData(Tuple2<String, Integer> data) {
            return data.f1.toString();
        }
    }

}

```

```
/**
 * 从指定的socket读取数据，对单词进行计算，将结果写入到Redis中
 */
public class RedisSinkDemo {

    public static void main(String[] args) throws Exception {

        //创建Flink流计算执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

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
                    //new Tuple2<String, Integer>(word, 1)
                    collector.collect(Tuple2.of(word, 1));
                }
            }
        });

        //分组
        KeyedStream<Tuple2<String, Integer>, String> keyed = wordAndOne.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> tp) throws Exception {
                return tp.f0;
            }
        });

        //聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> summed = keyed.sum(1);

        //Transformation结束

        //调用Sink
        //summed.addSink()
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost("liunx03").setPassword("123456").setDatabase(8).build();

        summed.addSink(new RedisSink<Tuple2<String, Integer>>(conf, new RedisWordCountMapper()));
        //启动执行
        env.execute("StreamingWordCount");

    }

    public static class RedisWordCountMapper implements RedisMapper<Tuple2<String, Integer>> {

        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET, "WORD_COUNT");
        }

        @Override
        public String getKeyFromData(Tuple2<String, Integer> data) {
            return data.f0;
        }

        @Override
        public String getValueFromData(Tuple2<String, Integer> data) {
            return data.f1.toString();
        }
    }

}

```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7bd96e7e4f63a618.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意注意默认偏移量同时保持在kafka和hdfs中

>   kafkaConsumer.setCommitOffsetsOnCheckpoints(false); //设置在checkpoint是不将偏移量保存到kafka特殊的topic中

##2.flink的job重启
> //如果你手动cancel job后，不删除job的checkpoint数据
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

##3.Savepoint
> Checkpoint的生命周期由Flink管理，即Flink创建，拥有和发布Checkpoint，无需用户交互。作为一种恢复和定期触发的方法，Checkpoint实现的两个主要设计目标是：i）being as lightweight to create （轻量级），ii）fast restore （快速恢复）。针对这些目标的优化可以利用某些属性，例如，JobCode在执行尝试之间不会改变。
  与此相反，Savepoints由用户创建，拥有和删除。它们的用例是planned (计划) 的，manual backup( 手动备份 ) 和 resume（恢复）。例如，这可能是Flink版本的更新，更改Job graph ，更改 parallelism ，分配第二个作业，如红色/蓝色部署，等等。当然，Savepoints必须在终止工作后继续存在。从概念上讲，保存点的生成和恢复成本可能更高，并且更多地关注可移植性和对前面提到的作业更改的支持

>保存点的作用：(1) 应用程序代码升级：假设你在已经处于运行状态的应用程序中发现了一个 bug，并且希望之后的事件都可以用修复后的新版本来处理。通过触发保存点并从该保存点处运行新版本，下游的应用程序并不会察觉到不同（当然，被更新的部分除外）。
(2) Flink 版本更新：Flink 自身的更新也变得简单，因为可以针对正在运行的任务触发保存点，并从保存点处用新版本的 Flink 重启任务。
(3) 维护和迁移：使用保存点，可以轻松地“暂停和恢复”应用程序。这对于集群维护以及向新集群迁移的作业来说尤其有用。此外，它还有利于开发、测试和调试，因为不需要重播整个事件流。
(4) 假设模拟与恢复：在可控的点上运行其他的应用逻辑，以模拟假设的场景，这样做在很多时候非常有用。
(5) A/B 测试：从同一个保存点开始，并行地运行应用程序的两个版本，有助于进行 A/B 测试。
