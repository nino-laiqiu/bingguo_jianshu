> 问题:map中一对一的类型匹配问题?

##Map 实现

实现一:
```
public class MapTransform {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-14
     * Time: 22:38
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Integer> map = source.transform("map", TypeInformation.of(Integer.class), new StreamMap<>(new MapFunction<String, Integer>() {
            @Override
            public Integer map(String s) throws Exception {
                return Integer.parseInt(s);
            }
        }));
        map.print();
        environment.execute("job");
    }
}
```

实现二:
```
public class MapTransform {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-14
     * Time: 22:38
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<Integer> source = environment.fromElements(1, 2, 3, 4);
        SingleOutputStreamOperator<Integer> map= source.transform("MyMap", TypeInformation.of(Integer.class), new Mymap());
        map.print();
        environment.execute("job");
    }

    //指定一个out 指定输入类型和输出类型
    public static class Mymap extends AbstractStreamOperator<Integer>
            implements OneInputStreamOperator<Integer, Integer> {
        @Override
        public void processElement(StreamRecord<Integer> element) throws Exception {
            Integer value = element.getValue();
            element.replace(value * 2);
            output.collect(element);
        }
    }
}
```

##2.FlatMap的实现

```
public class Flatmap {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-14
     * Time: 23:31
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<String> flatmap = source.transform("flatmap", TypeInformation.of(String.class), new MyFlatmap());
        flatmap.print();
        environment.execute("job");
    }

    public static class MyFlatmap extends AbstractStreamOperator<String>
            implements OneInputStreamOperator<String, String> {

        @Override
        public void processElement(StreamRecord<String> element) throws Exception {
            String value = element.getValue();
            String[] split = value.split(",");
            for (String s : split) {
                if (!"error".equals(s)) {
                      output.collect(element.replace(s));
                }
            }
        }
    }
}
```

##3.Fliter
```
public class Fliter {
/**
 * Created with IntelliJ IDEA.
 * Description: 
 * User: 
 * Date: 2021-01-15
 * Time: 10:26
 */
public static void main(String[] args) throws Exception {
    StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
    DataStreamSource<Integer> source = environment.fromElements(1, 2, 3, 4, 5, 6, 7);
    SingleOutputStreamOperator<Integer> fliter = source.transform("fliter", TypeInformation.of(Integer.class), new Fliter1());
    fliter.print();
    environment.execute("job");
}
   static  class Fliter1 extends AbstractStreamOperator<Integer>
         implements OneInputStreamOperator<Integer, Integer> {
     @Override
     public void processElement(StreamRecord<Integer> element) throws Exception {
          if (element.getValue() % 2 != 0){
                output.collect(element);
          }
     }
 }
}
```
##4.keyBy
> 在Flink开发中，如果定义一个JavaBean来封装数据，通常定义成public的成员变量，这是为了以后赋值和反射更加方便，如果定义了有参的构造方法，那么一定要再定义一个无参的构造方法，不然运行时会出现异常。并且最好定义一个静态的of方法用来赋值，这样更加方便。可以查看一下Flink中Tuple的源代码也是这么实现的
```
public class CountBean {
    public String word;
    public Integer count;
    public CountBean(){}
    public CountBean(String word, Integer count) {
        this.word = word;
        this.count = count;
    }
    public static CountBean of(String word, Integer count) {
        return new CountBean(word, 1);
    }
    @Override
    public String toString() {
        return “CountBean{” + “word='” + word + ‘\” + “, count=” + count + ‘}’;
    }    @Override
    public int hashCode() {
       return word.hashCode();
    }
}
```
```
//将一行数据切分后得到的每一个单词和1组合放入到自定义的bean实例中
DataStream<CountBean> wordAndOne = lines.flatMap(
        new FlatMapFunction<String, CountBean>() {
            @Override
            public void flatMap(String line,Collector<CountBean> out) throws Exception {
                String[] words = line.split(“\\W+”);
                for (String word : words) {
                    //将切分后的但是循环放入到bean中
                    out.collect(CountBean.of(word, 1));
                }
            }
        }
);
//按照Bean中的属性名word进行分组
KeyedStream<CountBean, Tuple> keyed = wordAndOne.keyBy(“word”);
```
```
public class Keyby {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-15
     * Time: 21:27
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.fromElements("flink,spark,flume", "scala,java,spark");
        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = source.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] split = s.split(",");
                for (String s1 : split) {
                    collector.collect(Tuple2.of(s1, 1));
                }
            }
        });

        KeyedStream<Tuple2<String, Integer>, Tuple> stream = flatMap.keyBy(1);
        stream.print();
        environment.execute("job");
    }
}
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-360a713981738181.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##5.Max
min和minBy的区别在于，min返回的是参与分组的字段和要比较字段的最小值，如果数据中还有其他字段，其他字段的值是总是第一次输入的数据的值。而minBy返回的是要比较的最小值对应的全部数据
取每个分区的最大值或者最小值
```
public class Max {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 11:18
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Tuple2<String, Integer>> map = source.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String s) throws Exception {
                String[] split = s.split(" ");
                return Tuple2.of(split[0], Integer.parseInt(split[1]));
            }
        });
        KeyedStream<Tuple2<String, Integer>, String> keyBy = map.keyBy(x -> x.f0);
        SingleOutputStreamOperator<Tuple2<String, Integer>> max = keyBy.max(1);
        max.print();
        environment.execute("job");
    }
```

##6.Min/Minby
>两个算子都是实现滚动比较最小值的功能，只有KeyedStream才可以调用min、minBy方法，如果Tuple类型数据，可以传入一个要比较的字段对应Tuple的下标（数字），如果是自定义的POJO类型数据，可以传入一个要聚合的字段名称。
```
public class Minby {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 14:26
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Tuple3<String, String, Integer>> map = source.map(new MapFunction<String, Tuple3<String, String, Integer>>() {
            @Override
            public Tuple3<String, String, Integer> map(String s) throws Exception {
                String[] split = s.split(",");
                String city = split[0];
                String country = split[1];
                int number = Integer.parseInt(split[2]);
                return Tuple3.of(city, country, number);


            }
        });
        KeyedStream<Tuple3<String, String, Integer>, Tuple> keyBy = map.keyBy("f0");
        SingleOutputStreamOperator<Tuple3<String, String, Integer>> min = keyBy.min(2);
        //SingleOutputStreamOperator<Tuple3<String, String, Integer>> minBy = keyBy.minBy(2);
        min.print();
        environment.execute("job");
    }
}
```

##7.Reduce

```
public class Reduce {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User:
     * Date: 2021-01-16
     * Time: 15:01
     */
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
        DataStreamSource<String> source = environment.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Tuple2<String, Integer>> map = source.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String s) throws Exception {
                String[] split = s.split(",");
                return Tuple2.of(split[0], Integer.parseInt(split[1]));
            }
        });
        KeyedStream<Tuple2<String, Integer>, Tuple> keyBy = map.keyBy(0);
        SingleOutputStreamOperator<Tuple2<String, Integer>> reduce = keyBy.reduce(new ReduceFunction<Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> reduce(Tuple2<String, Integer> stringIntegerTuple2, Tuple2<String, Integer> t1) throws Exception {
                //stringIntegerTuple2 是中间值或者是初始值
                stringIntegerTuple2.f1 += t1.f1;
                return stringIntegerTuple2;
                // return  Tuple2.of(stringIntegerTuple2.f0,stringIntegerTuple2.f1);
            }
        });
        reduce.print();
        environment.execute("job");
```
