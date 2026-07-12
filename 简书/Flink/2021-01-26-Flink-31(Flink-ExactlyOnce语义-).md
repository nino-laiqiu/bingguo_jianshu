>理解两阶段提交

##1.KafkaToKafka

> 从Kafka中读取数据，并且将数据处理后在写回到Kafka
要求：保证数据的一致性
 ExactlyOnce（Source可以记录偏移量【重放】，如果出现异常，的偏移量不更新），Sink要求支持事务
开启Checkpointping，Source的偏移量保存到状态中（OperatorState），然后将处理的数据也保存状态中

```

public class KafkaToKafka {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //开启checkpointing
        env.enableCheckpointing(30000, CheckpointingMode.EXACTLY_ONCE);
        env.setStateBackend(new FsStateBackend("file:///Users/xing/Desktop/flinkck20210123"));

        //设置Kafka相关参数
        Properties properties = new Properties();//设置Kafka的地址和端口
        properties.setProperty("bootstrap.servers", "linux03:9092,linux04:9092,linux05:9092");
        //读取偏移量策略：如果没有记录偏移量，就从头读，如果记录过偏移量，就接着读
        properties.setProperty("auto.offset.reset", "earliest");
        //设置消费者组ID
        properties.setProperty("group.id", "g1");
        //没有开启checkpoint，让flink提交偏移量的消费者定期自动提交偏移量
        properties.setProperty("enable.auto.commit", "false");
        //创建FlinkKafkaConsumer并传入相关参数
        FlinkKafkaConsumer<String> kafkaConsumer = new FlinkKafkaConsumer<>(
                "kafka2021", //要读取数据的Topic名称
                new SimpleStringSchema(), //读取文件的反序列化Schema
                properties //传入Kafka的参数
        );
        //使用addSource添加kafkaConsumer
        kafkaConsumer.setCommitOffsetsOnCheckpoints(false); //在checkpoint时，不将偏移量写入到kafka特殊的topic中

        DataStreamSource<String> lines = env.addSource(kafkaConsumer);

        SingleOutputStreamOperator<String> filtered = lines.filter(e -> !e.startsWith("error"));

        //使用的是AtLeastOnce
////        FlinkKafkaProducer<String> kafkaProducer = new FlinkKafkaProducer<>(
////                "linux:9092", "out2021", new SimpleStringSchema()
////        );


        //写入Kafka的topic
        String topic = "out2021";
        //设置Kafka相关参数
        properties.setProperty("transaction.timeout.ms",1000 * 60 * 5 + "");

        //创建FlinkKafkaProducer
        FlinkKafkaProducer<String> kafkaProducer = new FlinkKafkaProducer<String>(
                topic, //指定topic
                new KafkaStringSerializationSchema(topic), //指定写入Kafka的序列化Schema
                properties, //指定Kafka的相关参数
                FlinkKafkaProducer.Semantic.EXACTLY_ONCE //指定写入Kafka为EXACTLY_ONCE语义
        );

        filtered.addSink(kafkaProducer);

        env.execute();


    }
}
```
```
/**
 * 自定义String类型数据Kafka的序列化Schema
 */
public class KafkaStringSerializationSchema implements KafkaSerializationSchema<String> {

    private String topic;
    private String charset;
    //构造方法传入要写入的topic和字符集，默认使用UTF-8
    public KafkaStringSerializationSchema(String topic) {
        this(topic, "UTF-8");
    }
    public KafkaStringSerializationSchema(String topic, String charset) {
        this.topic = topic;
        this.charset = charset;
    }
    //调用该方法将数据进行序列化
    @Override
    public ProducerRecord<byte[], byte[]> serialize(String element, @Nullable Long timestamp) {
        //将数据转成bytes数组
        byte[] bytes = element.getBytes(Charset.forName(charset));
        //返回ProducerRecord
        return new ProducerRecord<>(topic, bytes);
    }
}
```

##2.CheckPoint
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a8c6086d0336b125.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>1.triger checkpiont(rpc)
2.snapshont(hdfs)
3.向jobmanager通知此当前的subtask快照成功
4.checkpointlister

##3.Exactly-once 两阶段提交

>我们知道 Flink 由 JobManager 协调各个 TaskManager 进行 checkpoint 存储，checkpoint 保存在 StateBackend 中，默认 StateBackend 是内存级的，也可以改为文件级的进行持久化保存

![image.png](https://upload-images.jianshu.io/upload_images/9049859-be43819be2cff5df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>当 checkpoint 启动时，JobManager 会将检查点分界线（barrier）注入数据流；barrier 会在算子间传递下去。

![image.png](https://upload-images.jianshu.io/upload_images/9049859-dce8ff97c6ce8114.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>每个算子会对当前的状态做个快照，保存到状态后端。对于 source 任务而言，就会把当前的 offset 作为状态保存起来。下次从 checkpoint 恢复时，source 任务可以重新提交偏移量，从上次保存的位置开始重新消费数据。

![image.png](https://upload-images.jianshu.io/upload_images/9049859-7efbb8ad1c947eeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>每个内部的 transform 任务遇到 barrier 时，都会把状态存到 checkpoint 里。sink 任务首先把数据写入外部 kafka，这些数据都属于预提交的事务（还不能被消费）；当遇到 barrier 时，把状态保存到状态后端，并开启新的预提交事务。

![image.png](https://upload-images.jianshu.io/upload_images/9049859-a2b502534c3be846.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>当所有算子任务的快照完成，也就是这次的 checkpoint 完成时，JobManager 会向所有任务发通知，确认这次 checkpoint 完成。
当 sink 任务收到确认通知，就会正式提交之前的事务，kafka 中未确认的数据就改为“已确认”，数据就真正可以被消费了。

![image.png](https://upload-images.jianshu.io/upload_images/9049859-ad61147f19ea2830.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>所以我们看到，执行过程实际上是一个两段式提交，每个算子执行完成，会进行“预提交”，直到执行完 sink 操作，会发起“确认提交”，如果执行失败，预提交会放弃掉。


>第一条数据来了之后，开启一个 kafka 的事务（transaction），正常写入
kafka 分区日志但标记为未提交，这就是“预提交”
jobmanager 触发 checkpoint 操作，barrier 从 source 开始向下传递，遇到
barrier 的算子将状态存入状态后端，并通知 jobmanager
sink 连接器收到 barrier，保存当前状态，存入 checkpoint，通知
jobmanager，并开启下一阶段的事务，用于提交下个检查点的数据
jobmanager 收到所有任务的通知，发出确认信息，表示 checkpoint 完成
sink 任务收到 jobmanager 的确认信息，正式提交这段时间的数据
外部 kafka 关闭事务，提交的数据可以正常消费了。
