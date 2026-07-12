##1.Wordcount案例
```
/**
 * 这是一个离线的WordCount，对有限的数据进行计算
 *
 * groupBy之后直接调用sum,没有进行局部聚合，而是全局聚合
 */
public class BatchWordCount {

    public static void main(String[] args) throws Exception{

        //实时：StreamExecutionEnvironment
        //离线：ExecutionEnvironment
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //创建数据集：DataSet
        DataSource<String> lines = env.readTextFile("");

        AggregateOperator<Tuple2<String, Integer>> result = lines.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        }).groupBy(0).sum(1);

        result.print();

        //离线计算可以不用写env.execute()
    }
}
```

```
/**
 * 这是一个离线的WordCount，对有限的数据进行计算
 *
 * 优化：离线计算，最好先局部聚合，然后在全局聚合
 */
public class BatchWordCountV2 {

    public static void main(String[] args) throws Exception{

        //实时：StreamExecutionEnvironment
        //离线：ExecutionEnvironment
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //创建数据集：DataSet
        DataSource<String> lines = env.readTextFile("");

        FlatMapOperator<String, Tuple2<String, Integer>> wordAndOne = lines.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });

        //先在每个分区先分组，然后在每个分区中进行局部聚合
        UnsortedGrouping<Tuple2<String, Integer>> unsortedGrouping = wordAndOne.groupBy(0);

        //然后在每个分区中进行局部聚合
        unsortedGrouping.combineGroup(new GroupCombineFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {

            @Override
            public void combine(Iterable<Tuple2<String, Integer>> values, Collector<Tuple2<String, Integer>> out) throws Exception {

                Integer total = 0;
                String key = null;
                for (Tuple2<String, Integer> tp : values) {
                    key = tp.f0;
                    total += tp.f1;
                }
                out.collect(Tuple2.of(key, total));
            }
        }).groupBy(0)
                .sum(1)//全局聚合
                .print();


    }
}

```

##2.排序案例
```
public class WordCountAndGlobalSort {

	public static void main(String[] args) throws Exception {

		ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

		DataSet<String> text = env.fromElements(
				"hadoop flink spark flink",
				"spark hadoop flink flink spark",
				"hadoop hbase hive hbase hive hue"
		);

		DataSet<Tuple2<String, Integer>> summed = text
				.flatMap(new LineSplitter())
				.groupBy(0)
				.sum(1);



		SortPartitionOperator<Tuple2<String, Integer>> result = summed.partitionByRange(1).withOrders(Order.DESCENDING).sortPartition(1, Order.DESCENDING);
		result.writeAsText("");
                env.execute();
		//result.print();

	}


	public static class LineSplitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
		@Override
		public void flatMap(String line, Collector<Tuple2<String, Integer>> out) {
			for (String word : line.split(" ")) {
				out.collect(new Tuple2<String, Integer>(word, 1));
			}
		}
	}

}
```
```
public class NumbersGlobalSort {

    public static void main(String[] args) throws Exception {

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSet<Integer> numbers = env.fromElements(
                10, 3, 6, 2, 9, 7, 5, 4, 2, 1, 8
        );

		SortPartitionOperator<Integer> result = numbers
				.partitionByRange(new MyKeySelector())
				.withOrders(Order.ASCENDING)
				.sortPartition(new MyKeySelector(), Order.ASCENDING);

		result.writeAsText("/Users/xing/Desktop/out7");
		env.execute();
	}
    public static class MyKeySelector implements KeySelector<Integer, Integer> {
        @Override
        public Integer getKey(Integer value) throws Exception {
            return value;
        }
    }
}
```

##3.Source案例

```
public class ReadCsvFileDemo {

    public static void main(String[] args) throws Exception {

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //指定数据的路径，数据的内容为：10001,佩奇,5,0和10002,乔治,3,1两行。
        String path = "";
        CsvReader csvReader = env.readCsvFile(path); //调用readCsvFile方法指定路径，返回CsvReader
        DataSource<Tuple4<Long, String, Integer, Boolean>> user1 = csvReader
                .types(Long.class, String.class, Integer.class, Boolean.class); //分别指定四个字段的类型
        DataSource<Tuple2<String, Integer>> user2 = csvReader
                .includeFields(false, true, true, false) //仅返回指定true对应的字段
                .types(String.class, Integer.class); //返回字段的类型
        DataSource<User> user3 = env.readCsvFile(path)
                .pojoType(User.class, "id", "name", "age", "gender"); //指定返回POJO的类型和字段名称

    }

}
```
```
public class ReadFileDemo {

    public static void main(String[] args) throws Exception {

        //创建离线计算的ExecutionEnvironment
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        String path = "hdfs://linux03:9000/words";
        //传入TextInputFormat类型的FileInputFormat
        DataSource<String> lines = env.readFile(new TextInputFormat(null), path);
        lines.print();
    }
}
```

**递归读取**
```
public class RecursiveFileEnumerationDemo {

    public static void main(String[] args) throws Exception{

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //创建一个Configuration用于设置参数
        Configuration parameters = new Configuration();
        //设置递归读取目录下数据的参数为true
        parameters.setBoolean("recursive.file.enumeration", true);
        //将配置好的参数使用withParameters方法传入
        DataSet<String> lines = env
                .readTextFile("hdfs://linux03:9000/words")
                .withParameters(parameters); //使用withParameters方法设置参数
        lines.print();

    }
}
```

##4.Transformations案例
```

public class MapPartitionDemo1 {
    public static void main(String[] args) throws Exception {

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //调用ExecutionEnvironment的fromElements方法创建DataSet
        env.setParallelism(2); //设置并行度为2
        DataSource<Integer> numbers = env.fromElements(1, 2, 3, 4, 5);
        //调用DataSet的mapPartition方法将每个分区中的每一个数据乘以10
        MapPartitionOperator<Integer, Integer> result = numbers.mapPartition(
                new MapPartitionFunction<Integer, Integer>() {
                    @Override
                    public void mapPartition(Iterable<Integer> values, Collector<Integer> out) throws Exception {
                        values.forEach(i -> out.collect(i * 10)); //将每一条数字乘以10，使用Collector输出
                    }
                });

        MapPartitionOperator<Integer, String> result1 = numbers.mapPartition(
            new RichMapPartitionFunction<Integer, String>() {
                @Override
                public void mapPartition(Iterable<Integer> values, Collector<String> out) throws Exception {
                    int index = getRuntimeContext().getIndexOfThisSubtask(); //获取当前subtask的编号
                    //遍历分区中的每一条数据，将当前subtask的编号和数据拼接成字符串，然后使用Collect输出
                    values.forEach(i -> out.collect("partition : " + index + " , value :" + i));
                }
            });

        result1.print();

        //result.print(); //调用print sink输出数据

    }
}

```
```
public class AggregateDemo1 {
    public static void main(String[] args) throws Exception{

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<Tuple2<String, Integer>> wordAndOne = env.fromElements(
                Tuple2.of("hadoop", 1), Tuple2.of("spark", 1), Tuple2.of("flink", 1),
                Tuple2.of("hadoop", 1), Tuple2.of("spark", 1), Tuple2.of("flink", 1)
        );
        AggregateOperator<Tuple2<String, Integer>> aggResult = wordAndOne.aggregate(Aggregations.SUM, 1);
        aggResult.print();
    }
}
```

```
public class CombineGroupDemo {
    public static void main(String[] args) throws Exception {

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<String> lines = env.readTextFile("hdfs://node-1.51doit.cn:9000/words");
        FlatMapOperator<String, Tuple2<String, Integer>> words = lines.flatMap(
                new FlatMapFunction<String, Tuple2<String, Integer>>() {
                    @Override
                    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                        Arrays.stream(value.split(" ")).forEach(w -> out.collect(Tuple2.of(w, 1)));
                    }
                });
        GroupCombineOperator<Tuple2<String, Integer>, Tuple2<String, Integer>> combinedWords = words
                .groupBy(0) //分组后调用combineGroup将一个分区内同一组的数据进行局部聚合（未shuffle）
                .combineGroup(new GroupCombineFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {
                    @Override
                    public void combine(Iterable<Tuple2<String, Integer>> inputs,
                                        Collector<Tuple2<String, Integer>> out) throws Exception {
                        String word = null;
                        int count = 0;
                        for (Tuple2<String, Integer> input : inputs) {
                            word = input.f0;
                            count++; //局部聚合
                        }
                        out.collect(Tuple2.of(word, count));
                    }
                });

        ReduceOperator<Tuple2<String, Integer>> result = combinedWords
                .groupBy(0) //再分组shuffle后进行全局聚合
                .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> v1,
                                                          Tuple2<String, Integer> v2) throws Exception {
                        return Tuple2.of(v1.f0, v1.f1 + v2.f1); //将次数相加
                    }
                });

        result.print();

    }
}
```

```
public class DistinctDemo1 {

    public static void main(String[] args) throws Exception{

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        DataSource<Integer> numbers = env.fromElements(1,2,3,4,5,6,7,8,9,3,5);
        //对DataSet中的数字进行去重
        DistinctOperator<Integer> distinct1 = numbers.distinct();
        distinct1.print();

        DataSource<WC> wordAndCount = env.fromElements(
                new WC("hadoop", 1), new WC("hadoop", 1),
                new WC("spark", 1), new WC("spark", 1),
                new WC("flink", 1), new WC("flink", 1)
        );
        //对DataSet中的自定义的POJO对象进行去重
        DistinctOperator<WC> distinct2 = wordAndCount.distinct();
        distinct2.print();



    }
}
```
```
public class GroupByDemo1 {
    public static void main(String[] args) throws Exception{

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //调用ExecutionEnvironment的fromElements方法创建DataSet
        DataSource<Tuple2<String, Integer>> wordAndOne = env.fromElements(
                Tuple2.of("hadoop", 1), Tuple2.of("spark", 1), Tuple2.of("flink", 1),
                Tuple2.of("hadoop", 1), Tuple2.of("spark", 1), Tuple2.of("flink", 1)
        );
        //调用groupBy方法进行分组，返回的类型为UnsortedGrouping
        UnsortedGrouping<Tuple2<String, Integer>> grouped = wordAndOne.groupBy(0);
        //分组后需要调用sum进行聚合（UnsortedGrouping不能直接调用Sink方法）
        DataSet<Tuple2<String, Integer>> result = grouped.sum(1);
        result.print();
    }
}
```
```
public class JoinDemo1 {


    public static class Employee {

        public Long empId;

        public String empName;

        public Long deptId;

        public Employee() {}

        public Employee(Long empId, String empName, Long deptId) {
            this.empId = empId;
            this.empName = empName;
            this.deptId = deptId;
        }

        @Override
        public String toString() {
            return "Employee{" +
                    "empId=" + empId +
                    ", empName='" + empName + '\'' +
                    ", deptId=" + deptId +
                    '}';
        }

    }

    public static class Department {

        public Long id;

        public String deptName;

        public Department() {}

        public Department(Long id, String deptName) {
            this.id = id;
            this.deptName = deptName;
        }

        @Override
        public String toString() {
            return "Department{" +
                    "id=" + id +
                    ", deptName='" + deptName + '\'' +
                    '}';
        }
    }

    public static void main(String[] args) throws Exception {

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        DataSource<Employee> employees = env.fromElements(
                new Employee(1L, "佩奇", 1001L),
                new Employee(2L, "乔治", 1002L),
                new Employee(3L, "苏西", 1003L)
        );
        DataSource<Department> departments = env.fromElements(
                new Department(1001L, "财务部"), new Department(1002L, "技术部"),
                new Department(1003L, "销售部"), new Department(1004L, "人事部")
        );
        DataSet<Tuple2<Employee, Department>> joined = employees.join(departments)
                .where("deptId")
                .equalTo("id");
        joined.print();


    }


}
```
```
public class ReduceDemo1 {
    public static void main(String[] args) throws Exception{

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //调用ExecutionEnvironment的fromElements方法创建DataSet
        DataSource<Long> numbers = env.generateSequence(1, 10);
        //调用未分组的DataSet的reduce方法，将DataSet中的数字进行聚合
        ReduceOperator<Long> result = numbers.reduce(new ReduceFunction<Long>() {
            @Override
            public Long reduce(Long v1, Long v2) throws Exception {
                return v1 + v2; //将数字相加
            }
        });
        result.print();
    }
}
```
```
public class SortGroupDemo {
    public static void main(String[] args) throws Exception {

        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //调用ExecutionEnvironment的fromElements方法创建DataSet
        DataSource<Tuple3<String, String, Double>> provinceCityAndMoney = env.fromElements(
                Tuple3.of("河北省", "廊坊市", 100.0), Tuple3.of("辽宁省", "沈阳市", 111.0),
                Tuple3.of("河北省", "唐山市", 120.0), Tuple3.of("辽宁省", "本溪市", 108.0),
                Tuple3.of("河北省", "邢台市", 115.0), Tuple3.of("辽宁省", "大连市", 116.0)
        );

        DataSet<Tuple3<String, String, Double>> result = provinceCityAndMoney
                .groupBy(0) //按照省份分组
                .sortGroup(2, Order.DESCENDING)
                .first(2);
        result.print();

    }


}
```
