1.关于setInputPaths()读取多个文件的业务
重写setup方法获取文件名
```
FileSplit f = (FileSplit) context.getInputSplit();
            fileName = f.getPath().getName();
```
join端:按文件名读取数据,目的是格式化存储
map端:把格式化的数据分别放到不同的容器中,区别的唯一标识符(自己设定的)
错误:

Mapreduce之hbase报错java.lang.NoSuchMethodException: HbaseMapReduce.ReadHbase$ReadMap.()
解决:

java lang NumberFormatException empty String
解决:

##3.MapReduce原理

上课总结:
1.linerecordreader(读取器)读取数据块
2.读取每一行切割(业务需求)映射为map(k,v,context)类型
3.分区器设置分区标准(hashpartioner),缓存数组方法(mapouputbuffer)映射map类型为collect(k,v,partion)类型
4.把这个类型读入到环形缓冲区(数组)进行区内排序在到达80%是溢出
5.spill方法把溢出的数据写入到硬盘
6.merge方法读取文件数据,并归和排序,继续写入到磁盘
7.默认在所有map阶段执行完,reducetask拷贝(HTTP)所有对应分区的map阶段数据,如果数据过少直接放在内存中,数据过多直接写入磁盘
8.merge 方法读取磁盘或者内存数据合并(按相同key)排序,迭代器迭代(key)x相同的数据
9.reduce方法遍历迭代器,聚合操作,写入到磁盘(textoutputformat方法)

##关于join的代码
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.util.Random;

public class MapReduce_9 {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration, "");
        job.setMapperClass(Map_9.class);
        job.setReducerClass(Reduce_9.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        job.setNumReduceTasks(2);
        FileInputFormat.setInputPaths(job,new Path("C:\\Users\\hp\\IdeaProjects\\maven1\\src\\main\\resources\\a.txt"));
        FileOutputFormat.setOutputPath(job,new Path("C:\\MapReduce"));
        job.waitForCompletion(true);
    }

   static   class Map_9 extends Mapper<LongWritable, Text,Text, IntWritable> {
        int numTasks = 0;

        @Override
        protected void setup(Context context) throws IOException, InterruptedException {

            numTasks = context.getNumReduceTasks();
        }

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            Text text = new Text();
            String[] words = value.toString().split("\\s+");
            //获取随机数字
            for (String word : words) {
                Random random = new Random();
                int i = random.nextInt(numTasks);
                text.set(word + "-" + i);

                context.write(text, new IntWritable(1));
            }
        }
    }
        static   class  Reduce_9 extends Reducer<Text, IntWritable,Text, IntWritable>{
            @Override
            protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
                           int sum =0;
                for (IntWritable value : values) {
                       sum+=value.get();
                }
                context.write(key, new IntWritable(sum));
            }
        }
}

```
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Hdfs_Jion {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration, "");
        job.setMapperClass(MapPort.class);
        job.setReducerClass(ReducrPort.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        job.setNumReduceTasks(1);
        FileInputFormat.setInputPaths(job,new Path("C:\\Users\\hp\\Desktop\\user.txt"),new Path("C:\\Users\\hp\\Desktop\\p.txt"));
        FileOutputFormat.setOutputPath(job,new Path("C:\\MAP"));
        job.waitForCompletion(true);

    }

    static String name;

    public static class MapPort extends Mapper<LongWritable, Text, Text, Text> {
        @Override
        protected void setup(Context context) throws IOException, InterruptedException {
            FileSplit inputSplit = (FileSplit) context.getInputSplit();
            name = inputSplit.getPath().getName();
        }

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            if ("user.txt".equals(name)) {
                String[] split = value.toString().split("\\s+");
                String uid = split[0];
                String uidName = split[1];
                context.write(new Text( uid), new Text(uidName));
            } else {
                String[] split1 = value.toString().split("\\s+");
                String uid = split1[0];
                String thing = split1[1];
                context.write(new Text(uid ), new Text(thing));
            }

        }
    }

    public static class ReducrPort extends Reducer<Text, Text, Text, NullWritable> {
        static Map<String, String> tab1 = new HashMap<>();
        static Map<String, String> tab2 = new HashMap<>();

        @Override
        protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
                for (Text value : values) {
                    if (value.toString().length()==3){
                    tab1.put(key.toString(), value.toString());}
                    else {  tab2.put(key.toString(), value.toString());}}
                }
        @Override
        protected void cleanup(Context context) throws IOException, InterruptedException {
            List<Map.Entry<String, String>> tab1Entries = new ArrayList<>(tab1.entrySet());
            List<Map.Entry<String, String>> tab2Entries = new ArrayList<>(tab2.entrySet());
            for (Map.Entry<String, String> tab1Entry : tab1Entries) {
                for (Map.Entry<String, String> tab2Entry : tab2Entries) {
                    if (tab1Entry.getKey() .equals(tab2Entry.getKey())) {
                        context.write(new Text(tab1Entry.getKey() + tab1Entry.getValue()
                                + tab2Entry.getValue()), NullWritable.get());
                    }
                }
            }
        }
    }
}
```
