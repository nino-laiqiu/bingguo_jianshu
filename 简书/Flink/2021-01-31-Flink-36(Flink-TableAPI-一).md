##1.概况
![image.png](https://upload-images.jianshu.io/upload_images/9049859-f73be43a2a9ed77f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>第一，Table API & SQL 是一种声明式的 API。用户只需关心做什么，不用关心怎么做，比如图中的 WordCount 例子，只需要关心按什么维度聚合，做哪种类型的聚合，不需要关心底层的实现。
第二，高性能。Table API & SQL 底层会有优化器对 query 进行优化。举个例子，假如 WordCount 的例子里写了两个 count 操作，优化器会识别并避免重复的计算，计算的时候只保留一个 count 操作，输出的时候再把相同的值输出两遍即可，以达到更好的性能。
第三，流批统一。上图例子可以发现，API 并没有区分流和批，同一套 query 可以流批复用，对业务开发来说，避免开发两套代码。
第四，标准稳定。Table API & SQL 遵循 SQL 标准，不易变动。API 比较稳定的好处是不用考虑 API 兼容性问题。
第五，易理解。语义明确，所见即所得。

![image.png](https://upload-images.jianshu.io/upload_images/9049859-0925940fcc2d0a86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>第一，Table API 使得多声明的数据处理写起来比较容易。
怎么理解？比如我们有一个 Table（tab），并且需要执行一些过滤操作然后输出到结果表，对应的实现是：tab.where(“a < 10”).inertInto(“resultTable1”)；此外，我们还需要做另外一些筛选，然后也对结果输出，即 tab.where(“a > 100”).insertInto(“resultTable2”)。你会发现，用 Table API 写起来会非常简洁方便，两行代码就把功能实现了。
第二，Table API 是 Flink 自身的一套 API，这使得我们更容易地去扩展标准的 SQL。当然，在扩展 SQL 的时候并不是随意的去扩展，需要考虑 API 的语义、原子性和正交性，并且当且仅当需要的时候才去添加。
对比 SQL，我们可以认为 Table API 是 SQL 的超集。SQL 有的操作，Table API 可以有，然而我们又可以从易用性和功能性地角度对 SQL 进行扩展和提升。


##2.Wordcount案例

#####SQL
```
public class Wordcount {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-31
     * Time: 19:47
     */
    private String Word;
    private Long counts;

    @Override
    public String toString() {
        return "Wordcount{" +
                "Word='" + Word + '\'' +
                ", counts=" + counts +
                '}';
    }

    public Wordcount() {
    }

    public String getWord() {
        return Word;
    }

    public void setWord(String word) {
        Word = word;
    }

    public Long getCounts() {
        return counts;
    }

    public void setCounts(Long counts) {
        this.counts = counts;
    }

    public Wordcount(String word, Long counts) {
        Word = word;
        this.counts = counts;
    }
}

```
```
public class Wordcount {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-31
     * Time: 19:41
     */
    public static void main(String[] args) throws Exception {
        LocalStreamEnvironment environment = StreamExecutionEnvironment.createLocalEnvironment();
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<String> flatMap = source.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String s, Collector<String> collector) throws Exception {
                for (String s1 : s.split(" ")) {
                    collector.collect(s1);
                }
            }
        });
        //创建表的执行环境
        StreamTableEnvironment tableEnvironment = StreamTableEnvironment.create(environment);
        tableEnvironment.registerDataStream("Wordcount",flatMap,"Word");

        Table Wordcounttable = tableEnvironment.sqlQuery("select Word,count(1) as counts from Wordcount group by Word");

        //转成流
        DataStream<Tuple2<Boolean, Wordcount>> stream = tableEnvironment.toRetractStream(Wordcounttable, Wordcount.class);
        stream.print();
        environment.execute("job");

    }
```
#####DSL
```
public class wordcount2 {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-31
     * Time: 20:33
     */
    public static void main(String[] args) throws Exception {
        LocalStreamEnvironment environment = StreamExecutionEnvironment.createLocalEnvironment();
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = source.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                for (String s1 : s.split(" ")) {
                    collector.collect(Tuple2.of(s1, 1));
                }
            }
        });

        StreamTableEnvironment tableEnvironment = StreamTableEnvironment.create(environment);
        Table dataStream = tableEnvironment.fromDataStream(flatMap, "word,wcount");
        Table wcount = dataStream.groupBy("word").select("word,count(wcount)");

        DataStream<Tuple2<Boolean, Row>> stream = tableEnvironment.toRetractStream(wcount, Row.class);

        stream.filter(f -> f.f0).print();
        environment.execute("job");
    }
}
```
