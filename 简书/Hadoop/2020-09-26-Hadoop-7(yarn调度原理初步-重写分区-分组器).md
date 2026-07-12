##1.windows上提交MR程序
代码同下需要变化的如下
```
  // 1 配置对象
        System.setProperty("HADOOP_USER_NAME", "root");
        Configuration configuration = new Configuration();
        //处理HDFS中的数据
        configuration.set("fs.default.name", "hdfs://linux03:8020");
       // 设置MR程序运行模式
        configuration.set("mapreduce.framework.name", "yarn");
        // 程序yarn的位置
        configuration.set("yarn.resourcemanager.hostname", "linux03");
        //设置跨平台参数
        configuration.set("mapreduce.app-submission.cross-platform", "true");
        Job job = Job.getInstance(configuration, "max3");
        //获取对象 设置包的位置
        job.setJar("C:\\Users\\hp\\Desktop\\考试题及课堂笔记\\demo.jar");
```
##2.linux提交MR程序
hadoop伪分布式wordcount报错Container exited with a non-zero exit code 1. Error file: prelaunch.err
>错误的解决mapred-site.xml文件中添加mapreduce所需要用到的classpath
```
<property>
   <name>mapreduce.application.classpath</name>
   <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*, $HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
</property>
<property>
   <name>mapreduce.application.classpath</name>
   <value>/opt/hadoop/hadoop-3.1.1/share/hadoop/mapreduce/*, /opt/hadoop/hadoop-3.1.1/share/hadoop/mapreduce/lib/*</value>
</property>
```

代码:
```
import com.google.gson.Gson;
import com.google.gson.JsonSyntaxException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.DoubleWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

//电影评分和
public class MapReduce_Map_Reduce1 {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        configuration.set("fs.default.name", "hdfs://linux03:8020");
        configuration.set("mapreduce.framework.name", "yarn");
        configuration.set("yarn.resourcemanager.hostname", "linux03");
        configuration.set("mapreduce.app-submission.cross-platform", "true");
        Job job = Job.getInstance(configuration, "max3");
        job.setJarByClass(MapReduce_Map_Reduce1.class);
        job.setMapperClass(Map_Test1.class);
        job.setReducerClass(Reduce_Text1.class);
        //设置输入输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(DoubleWritable.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(DoubleWritable.class);
        job.setNumReduceTasks(2);
        //设置来源和写入路径
        //可变参数 可传输多个路径
        FileInputFormat.setInputPaths(job, new Path("/tmp/data/test.json"));
        FileOutputFormat.setOutputPath(job, new Path("/tmp/consequence/consequencedata"));
        job.waitForCompletion(true);
    }

   public static class Map_Test1 extends Mapper<LongWritable, Text, Text, DoubleWritable> {
        Gson gson = new Gson();
        Text text = new Text();
        DoubleWritable doubleWritable = new DoubleWritable();

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            try {
                String string = value.toString();
                MoviePoint moviePoint = gson.fromJson(string, MoviePoint.class);
                text.set(moviePoint.getMovie());
                doubleWritable.set(moviePoint.getRate());
                context.write(text, doubleWritable);

            } catch (Exception e) {

            }
        }
    }
    public static class Reduce_Text1 extends Reducer<Text, DoubleWritable, Text, DoubleWritable> {
        @Override
        protected void reduce(Text key, Iterable<DoubleWritable> values, Context context) throws IOException, InterruptedException {
            DoubleWritable doubleWritable = new DoubleWritable();
            double sum = 0;
            for (DoubleWritable value : values) {
                sum += value.get();
            }
            doubleWritable.set(sum);
            context.write(key, doubleWritable);
        }
    }
}

```
>mapred-site.xml(添加配置文件)
```
<configuration>
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1 </value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1 </value>
</property>
</configuration>
```
在Linux上添加配置文件

>vi /opt/hadoop/hadoop-3.1.1/etc/hadoop/ mapred-site.xml

```
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1 </value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=/opt/hadoop/hadoop-3.1.1 </value>
</property>
<property>
   <name>mapreduce.application.classpath</name>
   <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*, $HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
</property>
<property>
   <name>mapreduce.application.classpath</name>
   <value>/opt/hadoop/hadoop-3.1.1/share/hadoop/mapreduce/*, /opt/hadoop/hadoop-3.1.1/share/hadoop/mapreduce/lib/*</value>
</property>
```
打包执行
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c3d6cc51a767d588.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下载 到Linux03(rz)

>hadoop jar /demo.jar HDFSUNIL/MapReduce_Map_Reduce1

##3.重写分区器和分组器
**json脏数据捕捉只是按格式捕捉,并没有捕捉全部的脏数据
注意 继承WritableComparable 重写compareTo方法的完整性目的性
hadoop自定义类型 那个类型一定要有一个无参构造方法**
**重写分区器方法**
```
public static class MyPartion extends Partitioner<Movie, NullWritable> {
        @Override
        public int getPartition(Movie movie, NullWritable nullWritable, int i) {
            return (movie.getUid().hashCode() & Integer.MAX_VALUE) % i;
        }
    }
```
**重写分组器方法
WritableComparator**


案例电影评分topN
```
import com.google.gson.Gson;
import com.google.gson.JsonSyntaxException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;

import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;

import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
//用户评分排名前三的电影
public class YarnTest {
    static Gson gson = new Gson();

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration, "job");
        //设置map 和 reduce 的来源
        job.setMapperClass(MapperR.class);
        job.setReducerClass(ReduceM.class);
        //设置map 和 reduce 输入输出 的类型
        job.setMapOutputKeyClass(Movie.class);
        job.setMapOutputValueClass(NullWritable.class);
        job.setOutputKeyClass(Movie.class);
        job.setOutputValueClass(NullWritable.class);
        //设置分区器 在map阶段的分区标准
        job.setPartitionerClass(MyPartion.class);
        //设置分组标准在reduce阶段 key值重组的标准
        job.setGroupingComparatorClass(MyWritableComparter.class);
        job.setNumReduceTasks(3);
        FileInputFormat.setInputPaths(job,new Path("C:\\Users\\hp\\IdeaProjects\\maven1\\src\\main\\resources\\test.json"));
        FileOutputFormat.setOutputPath(job,new Path("C:\\Mapreduce"));
        job.waitForCompletion(true);
    }
    public static class MapperR extends Mapper<LongWritable, Text, Movie, NullWritable> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            try {
                Movie movie = gson.fromJson(value.toString(), Movie.class);
                context.write(movie,NullWritable.get());
            } catch (Exception e) {

            }
        }
    }
    public static class ReduceM extends Reducer<Movie, NullWritable, Movie, NullWritable> {
        @Override
        protected void reduce(Movie key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
             //key经过排序重写是按uid来排序的
            int  num =0;
            for (NullWritable value : values) {
                 context.write(key,value);
                 num++;
                 if (num==3){
                   //break;
                     return;
                   
                 }
            }

        }
    }
    //分区器按照uid来划分  方便比较重写compareTo方法
    //相同uid划分一个组别
    public static class MyPartion extends Partitioner<Movie, NullWritable> {

        @Override
        public int getPartition(Movie movie, NullWritable nullWritable, int i) {
            return (movie.getUid().hashCode() & Integer.MAX_VALUE) % i;
        }
    }
    //分组器按照uid来划分
    public static class MyWritableComparter extends WritableComparator {
        public MyWritableComparter() {
            super(Movie.class, true);
        }
        @Override
        public int compare(WritableComparable a, WritableComparable b) {
            Movie A1 = (Movie) a;
            Movie B1 = (Movie) b;
            return  A1.getUid().compareTo(B1.getUid());
        }
    }
}
```
(说明不重写分区器不同uid分布到不同的reducetask上 多出了结果)

（出现的bug）
重写方法类型的不一致
重写方法比较的前后顺序不一致

##setup 和 cleanup用法

```
import com.google.gson.Gson;
        import org.apache.hadoop.conf.Configuration;
        import org.apache.hadoop.fs.Path;
        import org.apache.hadoop.io.*;
        import org.apache.hadoop.mapreduce.Job;
        import org.apache.hadoop.mapreduce.Mapper;
        import org.apache.hadoop.mapreduce.Partitioner;
        import org.apache.hadoop.mapreduce.Reducer;
        import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
        import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

        import java.io.IOException;
        import java.util.*;


//求电影评论数量排名前三的电影
public class Yarn_1 {
    static Gson gson = new Gson();

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration, "job");
        //设置map 和 reduce 的来源
        job.setMapperClass(MapperR.class);
        job.setReducerClass(ReduceM.class);
        //设置map 和 reduce 输入输出 的类型
        job.setMapOutputKeyClass(Movie.class);
        job.setMapOutputValueClass(IntWritable.class);
        job.setOutputKeyClass(Movie.class);
        job.setOutputValueClass(IntWritable.class);
        //设置分区器 在map阶段的分区标准
        job.setPartitionerClass(MyPartion.class);
        //设置分组标准在reduce阶段 key值重组的标准
        job.setGroupingComparatorClass(MyWritableComparter.class);
        job.setNumReduceTasks(4);
        FileInputFormat.setInputPaths(job,new Path("C:\\Users\\hp\\IdeaProjects\\maven1\\src\\main\\resources\\test.json"));
        FileOutputFormat.setOutputPath(job,new Path("C:\\Mapreduce1"));
        System.out.println(job.waitForCompletion(true));
    }
    public static class MapperR extends Mapper<LongWritable, Text, Movie, IntWritable> {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            try {
                Movie movie = gson.fromJson(value.toString(), Movie.class);
                context.write(movie,new IntWritable(1));
            } catch (Exception e) {

            }
        }
    }

    public static class ReduceM extends Reducer<Movie, IntWritable, Movie, IntWritable> {
        @Override
        protected void reduce(Movie key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        static Map<Movie,Integer> map =  new HashMap<Movie,Integer>();
       //一个reducetask
            Movie movie = new Movie();
         //   map =  new HashMap<Movie,Integer>();
            int  num =0;
            for (IntWritable value : values) {
                try {
                    BeanUtils.copyProperties(movie,key);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                } catch (InvocationTargetException e) {
                    e.printStackTrace();
                }
                num++;
                // System.out.println(num);
            }
            //一个迭代器存储相同的key 每次加一
            //这里只存储了最后的key
            map.put(movie,num);
        }
        @Override
        protected void cleanup(Context context) throws IOException, InterruptedException {
            ArrayList<Map.Entry<Movie, Integer>> entries = new ArrayList<>(map.entrySet());
            entries.sort((t1,t2)->t2.getValue().compareTo(t1.getValue()));
            for (int i = 0; i < Integer.min(4,entries.size()); i++) {
                context.write(entries.get(i).getKey(),new IntWritable(entries.get(i).getValue()));
            }
        }
    }

    //分区器按照Movie来划分  方便比较重写compareTo方法
    //相同uid划分一个组别
    public static class MyPartion extends Partitioner<Movie,IntWritable> {

        @Override
        public int getPartition(Movie movie, IntWritable nullWritable, int i) {
            // System.out.println("pppppppppppppppppppppppppppppppppp");
            return (movie.getMovie().hashCode() & Integer.MAX_VALUE) % i;
        }
    }
    //分组器按照Movie来划分
    public static class MyWritableComparter extends WritableComparator {
        public MyWritableComparter() {
            super(Movie.class, true);
        }
        @Override
        public int compare(WritableComparable a, WritableComparable b) {
            Movie A1 = (Movie) a;
            Movie B1 = (Movie) b;
            return  B1.getMovie().compareTo(A1.getMovie());
        }
    }
}
```

(hashmap中存放了每个reducetask的获取的数据)

