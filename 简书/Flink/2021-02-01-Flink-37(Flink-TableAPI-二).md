##1.从文件中读取数据
```
public class Test1 {

    public static void main(String[] args) throws Exception {
        LocalStreamEnvironment environment = StreamExecutionEnvironment.createLocalEnvironment();
        StreamTableEnvironment tableEnvironment = StreamTableEnvironment.create(environment);
        String PATH = "C:\\Users\\hp\\IdeaProjects\\Flink_Java\\src\\main\\resources\\1.csv";
        //注册一张表
        //指定路径,格式,元数据字段类型
        tableEnvironment.connect(new FileSystem().path(PATH)
        ).withFormat(new Csv()).withSchema(new Schema()
                .field("id", DataTypes.STRING())
                .field("age", DataTypes.INT())
        ).createTemporaryTable("myTable");

        Table table = tableEnvironment.from("myTable");
        tableEnvironment.toRetractStream(table, Row.class).print();
        environment.execute("job");
    }
}
```

##2.更新模式
![image.png](https://upload-images.jianshu.io/upload_images/9049859-25ff6d3d6f6f54be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ecf4f48a2d177585.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3.时间特性

**processing Time**
```
public class Test1 {
    public static void main(String[] args) throws Exception {
        LocalStreamEnvironment environment = StreamExecutionEnvironment.createLocalEnvironment();
        String PATH = "C:\\Users\\hp\\IdeaProjects\\Flink_Java\\src\\main\\resources\\1.csv";
        DataStreamSource<String> source = environment.readTextFile(PATH);
        StreamTableEnvironment tableEnvironment = StreamTableEnvironment.create(environment);
        SingleOutputStreamOperator<Tuple2<String, String>> streamOperator = source.map(x -> Tuple2.of(x.split(",")[0], x.split(",")[1])).returns(TypeInformation.of(new TypeHint<Tuple2<String, String>>() {
        }));
        Table table = tableEnvironment.fromDataStream(streamOperator, "id,age,pt.proctime");
        tableEnvironment.toAppendStream(table, Row.class).print();
        environment.execute();

    }
}
```
**event Time**
```
public class Test1 {
    public static void main(String[] args) throws Exception {
        LocalStreamEnvironment environment = StreamExecutionEnvironment.createLocalEnvironment();
        environment.setParallelism(1);
        String PATH = "C:\\Users\\hp\\IdeaProjects\\Flink_Java\\src\\main\\resources\\1.csv";
        DataStreamSource<String> source = environment.readTextFile(PATH);
        StreamTableEnvironment tableEnvironment = StreamTableEnvironment.create(environment);
        SingleOutputStreamOperator<String> operator = source.assignTimestampsAndWatermarks(WatermarkStrategy
                .<String>forBoundedOutOfOrderness(Duration.ZERO)
                .withTimestampAssigner((line, timestamp) -> Long.parseLong(line.split(",")[0])));
        SingleOutputStreamOperator<Tuple2<Long, String>> operator1 = operator.map(x -> Tuple2.of(Long.parseLong(x.split(",")[0]), x.split(",")[1])).returns(TypeInformation.of(new TypeHint<Tuple2<Long, String>>() {
        }));
        Table table = tableEnvironment.fromDataStream(operator1, "id.rowtime  ,age");
        tableEnvironment.toAppendStream(table, Row.class).print();
        environment.execute();

    }
}
```
##4.窗口
![image.png](https://upload-images.jianshu.io/upload_images/9049859-797ae32810c03e17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-6feb4e340a2643e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-88307e3aa51f0521.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b3c4a8841f5c747d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
