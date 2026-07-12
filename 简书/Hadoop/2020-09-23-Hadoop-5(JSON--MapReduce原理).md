
问题:
 job.setNumReduceTasks(2) 关于分区指的是什么
分区器 hashcode%n 把经过map映射排序完的(K,V)标号为(K,V,n)分别在reduce上排序分组聚合

pom.xml 作用增添依赖jar包

一些方法:
BeanUtils.copyProperties(a, b);
BeanUtils是org.springframework.beans.BeanUtils， a拷贝到b
BeanUtils是org.apache.commons.beanutils.BeanUtils，b拷贝到a

Integer.min(a,b)
a>b取b  a<b取a

maven alibaba JSON的依赖架包
```
 <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>fastjson</artifactId>
                <version>1.2.41</version>
            </dependency>
````

##MapReduce原理
![image.png](https://upload-images.jianshu.io/upload_images/9049859-bd277e34fe67a1c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


数据分析案例:
注意脏数据的捕捉
注意集合的位置影响结果<归根结底是地址值的问题>
```
import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class MoviePoint implements Writable {
    private String movie;
    private double rate;
    private long timeStamp;
    private String uid;

    @Override
    public String toString() {
        return "MoviePoint{" +
                "movie='" + movie + '\'' +
                ", rate=" + rate +
                ", timeStamp=" + timeStamp +
                ", uid='" + uid + '\'' +
                '}';
    }

    public MoviePoint() {
    }

    public String getMovie() {
        return movie;
    }

    public void setMovie(String movie) {
        this.movie = movie;
    }

    public double getRate() {
        return rate;
    }

    public void setRate(double rate) {
        this.rate = rate;
    }

    public long getTimeStamp() {
        return timeStamp;
    }

    public void setTimeStamp(long timeStamp) {
        this.timeStamp = timeStamp;
    }

    public String getUid() {
        return uid;
    }

    public void setUid(String uid) {
        this.uid = uid;
    }

    public MoviePoint(String movie, double rate, long timeStamp, String uid) {
        this.movie = movie;
        this.rate = rate;
        this.timeStamp = timeStamp;
        this.uid = uid;
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeUTF(movie);
        dataOutput.writeDouble(rate);
        dataOutput.writeLong(timeStamp);
        dataOutput.writeUTF(uid);
    }

    @Override
    public void readFields(DataInput dataInput) throws IOException {
        this.movie = dataInput.readUTF();
        this.rate = dataInput.readDouble();
        this.timeStamp = dataInput.readLong();
        this.uid = dataInput.readUTF();
    }

    @Override
    protected MoviePoint clone() throws CloneNotSupportedException {
        return new MoviePoint(getMovie(),getRate(),getTimeStamp(),getUid());
    }
}
```
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
        Job job= Job.getInstance(configuration, "max3");
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
        FileInputFormat.setInputPaths(job,new Path("C:\\Users\\hp\\IdeaProjects\\maven1\\src\\main\\resources\\test.json"));
        FileOutputFormat.setOutputPath(job,new Path("C:\\MapReduce"));
        job.waitForCompletion(true);
    }
    }
class Map_Test1 extends Mapper<LongWritable, Text, Text, DoubleWritable> {
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
            context.write(text,doubleWritable);

        } catch (Exception e) {

        }
    }
}
class Reduce_Text1 extends Reducer<Text, DoubleWritable,Text, DoubleWritable>{
    @Override
    protected void reduce(Text key, Iterable<DoubleWritable> values, Context context) throws IOException, InterruptedException {
        DoubleWritable doubleWritable = new DoubleWritable();
        double sum = 0;
        for (DoubleWritable value : values) {
             sum+=value.get();
        }
        doubleWritable.set(sum);
        context.write(key,doubleWritable);
    }
}
```
```
import com.alibaba.fastjson.JSON;
import com.google.gson.Gson;
import com.google.gson.JsonSyntaxException;
import org.apache.commons.beanutils.BeanUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;


import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class MapReduce_Map_Reduce {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "");
        job.setMapperClass(Map_Test.class);
        job.setReducerClass(Reduce.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(MoviePoint.class);
        job.setOutputKeyClass(MoviePoint.class);
        job.setOutputValueClass(NullWritable.class);
        FileInputFormat.setInputPaths(job, new Path("C:\\Users\\hp\\IdeaProjects\\maven1\\src\\main\\resources\\test.json"));
        FileOutputFormat.setOutputPath(job, new Path("C:\\MapReduce"));
        job.waitForCompletion(true);
    }
}

class Map_Test extends Mapper<LongWritable, Text, Text, MoviePoint> {
    Gson gson = new Gson();
    Text text = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        try {
            String len = value.toString();
            MoviePoint moviePoint = gson.fromJson(len, MoviePoint.class);
            text.set(moviePoint.getMovie());
            context.write(text, moviePoint);
        } catch (Exception e) {

        }
    }
}
class Reduce extends Reducer<Text, MoviePoint, MoviePoint, NullWritable> {
    @Override
    protected void reduce(Text key, Iterable<MoviePoint> values, Context context) throws IOException, InterruptedException {
        //注意不能放在外面 编程成员方法
        List<MoviePoint> list = new ArrayList<>();
        for (MoviePoint value : values) {
            // MoviePoint moviePoint = new MoviePoint();

          /*     moviePoint.setMovie(value.getMovie());
            moviePoint.setRate(value.getRate());
            moviePoint.setTimeStamp(value.getTimeStamp());
            moviePoint.setRate(value.getRate());*/
         /* try {
                BeanUtils.copyProperties(moviePoint,value);
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }*/
            try {
                list.add(value.clone());
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }

        }
        list.sort((o1,o2)->Double.compare(o2.getRate(), o1.getRate()));
        for (int i = 0; i < Integer.min(3, list.size()); i++) {
            context.write(list.get(i), NullWritable.get());
        }
    }
}
```
