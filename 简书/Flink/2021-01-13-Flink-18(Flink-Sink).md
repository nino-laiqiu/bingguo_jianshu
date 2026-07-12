
经过一系列Transformation转换操作后，最后一定要调用Sink操作，才会形成一个完整的DataFlow拓扑。只有调用了Sink操作，才会产生最终的计算结果，这些数据可以写入到的文件、输出到指定的网络端口、消息中间件、外部的文件系统或者是打印到控制台

##1.Print
**手动实现**
```
public class Print {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-14
     * Time: 20:05
     */
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);
        DataStreamSource<String> source = env.socketTextStream("localhost", 8888);
        DataStreamSink<String> sink = source.addSink(new Print_Sink());
        System.out.println(sink);
        env.execute("job");
    }

    public  static  class Print_Sink extends RichSinkFunction<String>{
        int indexOfThisSubtask = 0 ;
        @Override
        public void open(Configuration parameters) throws Exception {
              //获取分区
            indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();
        }

        @Override
        public void invoke(String value, Context context) throws Exception {
            System.out.println(indexOfThisSubtask + ">" +value);
        }
    }
}
```

**源码**
```
	public PrintSinkFunction(final String sinkIdentifier, final boolean stdErr) {
		writer = new PrintSinkOutputWriter<>(sinkIdentifier, stdErr);
	}

	@Override
	public void open(Configuration parameters) throws Exception {
		super.open(parameters);
		StreamingRuntimeContext context = (StreamingRuntimeContext) getRuntimeContext();
		writer.open(context.getIndexOfThisSubtask(), context.getNumberOfParallelSubtasks());
	}

	@Override
	public void invoke(IN record) {
		writer.write(record);
	}

	@Override
	public String toString() {
		return writer.toString();
	}
```

##2.KafkaSink
>在实际的生产环境中，经常会有一些场景，需要将Flink处理后的数据快速地写入到一个分布式、高吞吐、高可用、可用保证Exactly Once的消息中间件中，供其他的应用消费处理后的数据。Kafka就是Flink最好的黄金搭档，Flink不但可以从Kafka中消费数据，还可以将处理后的数据写入到Kafka，并且吞吐量高、数据安全、可以保证Exactly Once等。
Flink可以和Kafka多个版本整合，比如0.11.x、1.x、2.x等，从Flink1.9开始，使用的是kafka 2.2的客户端，所以这里使用kafka的版本是2.2.2，并且使用最新的API。
下面的例子就是将数据写入到Kafka中，首先要定义一个类实现KafkaSerializationSchema接口，指定一个泛型，String代表要写入到Kafka的数据为String类型。该类的功能是指定写入到Kafka中数据的序列化Schema，需要重写serialize方法，将要写入的数据转成二进制数组，并封装到一个ProducerRecord中返回。
```
//自定义String类型数据Kafka的序列化Schema
public class KafkaStringSerializationSchema implements KafkaSerializationSchema<String> {
    private String topic;
    private String charset;
    //构造方法传入要写入的topic和字符集，默认使用UTF-8
    public KafkaStringSerializationSchema(String topic) {
        this(topic, “UTF-8”);
    }
    public KafkaStringSerializationSchema(String topic, String charset) {
        this.topic = topic;
        this.charset = charset;
    }
    //调用该方法将数据进行序列化
    @Override
    public ProducerRecord<byte[], byte[]> serialize(
            String element, @Nullable Long timestamp) {
        //将数据转成bytes数组
        byte[] bytes = element.getBytes(Charset.forName(charset));
        //返回ProducerRecord
        return new ProducerRecord<>(topic, bytes);
    }
}
```

>然后将Kafka相关的参数设置到Properties中，再new FlinkKafkaProducer，将要写入的topic名称、Kafka序列化Schema、Properties和写入到Kafka的Semantic语义作为FlinkKafkaProducer构造方法参数传入。最好调用addSink方法将FlinkKafkaProducer的引用传入到该方法中。虽然下面的代码指定了EXACTLY_ONCE语义，但是没有开启Checkpointing，是没法实现的。

```
DataStream<String> dataSteam = …
//写入Kafka的topic
String topic = “test”;
//设置Kafka相关参数
Properties properties = new Properties();
properties.setProperty(“bootstrap.servers”, “node-1:9092,node-2:9092,node-3:9092”);
//创建FlinkKafkaProducer
FlinkKafkaProducer<String> kafkaProducer = new FlinkKafkaProducer<String>(
        topic, //指定topic
        new KafkaStringSerializationSchema(topic), //指定写入Kafka的序列化Schema
        properties, //指定Kafka的相关参数
        FlinkKafkaProducer.Semantic.EXACTLY_ONCE //指定写入Kafka为EXACTLY_ONCE语义
);
//添加KafkaSink
dataSteam.addSink(kafkaProducer);
```

##3.RedisSink
```
<dependency>
    <groupId>org.apache.bahir</groupId>
    <artifactId>flink-connector-redis_2.12</artifactId>
    <version>1.1-SNAPSHOT</version>
</dependency>
```
>定义一个类（或者静态内部类）实现RedisMapper即可，需要指定一个泛型，这里是Tuple2<String, Integer>，即写入到Redis中的数据的类型，并实现三个方法。第一个方法是getCommandDescription方法，返回RedisCommandDescription实例，在该构造方法中可以指定写入到Redis的方法类型为HSET，和Redis的additionalKey即value为HASH类型外面key的值；第二个方法getKeyFromData是指定value为HASH类型对应key的值；第三个方法geVauleFromData是指定value为HASH类型对应value的值
```
public static class RedisWordCountMapper implements RedisMapper<Tuple2<String, Integer>> {
    @Override
    public RedisCommandDescription getCommandDescription() {
        //写入Redis的方法，value使用HASH类型，并指定外面key的值得名称
        return new RedisCommandDescription(RedisCommand.HSET, “WORD_COUNT”);
    }
    @Override
    public String getKeyFromData(Tuple2<String, Integer> data) {
        return data.f0; //指定写入Redis的value里面key的值
    }
    @Override
    public String getValueFromData(Tuple2<String, Integer> data) {
        return data.f1.toString(); //指定写入value里面value的值
    }
}
```

```
DataStream<Tuple2<String, Integer>> result = wordAndOne.keyBy(0).sum(1);//设置
Redis的参数，如地址、端口号等
FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost(“localhost”).setPassword(“123456”).build();

//将数据写入Redis
result.addSink(new RedisSink<>(conf, new RedisWordCountMapper()));
```

##4.StreamFileDataSink
```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-filesystem_2.11</artifactId>
    <version>1.9.2</version>
</dependency>
```
>通过DefaultRollingPolicy这个工具类，指定文件滚动生成的策略。这里设置的文件滚动生成策略有两个，一个是距离上一次生成文件时间超过30秒，另一个是文件大小达到100 mb。这两个条件只要满足其中一个即可滚动生成文件。然后StreamingFileSink.forRowFormat方法将文件输出目录、文件写入的编码传入，再调用withRollingPolicy关联上面的文件滚动生成策略，接着调用build方法构建好StreamingFileSink，最后将其作为参数传入到addSink方法中
```
DataStream<String> dataSteam = …//构建文件滚动生成的策略
DefaultRollingPolicy<String, String> rollingPolicy = DefaultRollingPolicy.create()
        .withRolloverInterval(30 * 1000L) //30秒滚动生成一个文件
        .withMaxPartSize(1024L * 1024L * 100L) //当文件达到100m滚动生成一个文件
        .build();
//创建StreamingFileSink，数据以行格式写入
StreamingFileSink<String> sink = StreamingFileSink.forRowFormat(
                new Path(outputPath), //指的文件存储目录
                new SimpleStringEncoder<String>(“UTF-8”)) //指的文件的编码
        .withRollingPolicy(rollingPolicy) //传入文件滚动生成策略
        .build();
//调用DataStream的addSink添加该Sink
dataSteam.addSink(sink);
```
