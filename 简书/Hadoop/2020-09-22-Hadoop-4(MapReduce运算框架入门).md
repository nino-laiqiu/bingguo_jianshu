#####MapReduce运算框架

HDFS  文件分布式系统  
传统的计算:统一运算
MapReduce 分布运算(分区器:hashcode%n)  统一结果
分布运算以 行 为单位

关于多台机器并行运算:
HDFS上分布存储的文件,假设存储文件的机器与计算的机器不一致,采取的策略有移动数据或移动运算

#####序列化流的补充和不足
读
readobject
readint  readutf
写:
writeobject
writeint writeutf
不足:
Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系。），不便于在网络中高效传输；

##对map和reduce函数的说明(问题是储存在散列表中吗)

map(起始量,要读取的一行数据的类型,要存储的key的类型,要存储的value值的类型)
reduce(Reduce的前两个就是Map的后两个，这个也很容易理解，因为Reduce的输入是Map通过调用Context.write方法产生的)

##MapReduce入门
1.设置对象
2.获取job对象
3,设置map  reduce类的反射
4.输入输出类型
5.设置reduce的个数
6.设置处理数据输入输出的路径
7.提交job

入门代码:
```
import org.apache.hadoop.io.DoubleWritable;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class MapTest extends Mapper<LongWritable, Text, Text, DoubleWritable> {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User: p
     * Date: 2020-09-22
     * Time: 17:42
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //每次读一行
        String[] split = value.toString().split("\\s+");
        //判断是否为空  网址k
        String k = split[0];
        //获取长度
        int l = split.length;
        double v = 0;
        // split[l-2].matches("^[1-9\\d*$]");
        if (l >5 ) {
            if (split[l - 2].matches("^[1-9]\\d*$") && split[l - 3].matches("^[1-9]\\d*$")) {
                //v = Double.parseDouble(split[l - 2] + split[l - 3]);
                v = Double.parseDouble(split[l - 2] )+ Double.parseDouble(split[l - 3]);
            }
        }
        context.write(new Text(k), new DoubleWritable(v));

    }
}
```

```
import org.apache.hadoop.io.DoubleWritable;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class ReduceTest extends Reducer<Text, DoubleWritable,Text,DoubleWritable> {
    @Override
    protected void reduce(Text key, Iterable<DoubleWritable> values, Context context) throws IOException, InterruptedException {
        double sum =0;
        for (DoubleWritable value : values) {
            sum+=value.get();
        }
        context.write(key,new DoubleWritable(sum));
    }
}
```
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.DoubleWritable;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class MapReduce_1 {
      //设置对象
public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
    //设置对象
    Configuration configuration = new Configuration();
     //获取job对象
    Job job= Job.getInstance(configuration, "sumnumber");
    //设置map 和 reduce 的来源
    job.setMapperClass(MapTest.class);
    job.setReducerClass(ReduceTest.class);
    //设置类型
    job.setMapOutputKeyClass(Text.class);
    job.setMapOutputValueClass(DoubleWritable.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(DoubleWritable.class);
    //设置reduce的个数
     job.setNumReduceTasks(2);
     //设置处理路径
    FileInputFormat.setInputPaths(job,new Path("C:\\Users\\hp\\Desktop\\flow.log"));
    FileOutputFormat.setOutputPath(job,new Path("C:\\mapreduce"));
    job.waitForCompletion(true);
}
}

```
数据如下:
```
1363157985066 	13726230503	00-FD-07-A4-72-B8:CMCC	120.196.100.82	i02.c.aliimg.com		24	27	2481	24681	200
1363157995052 	13826544101	5C-0E-8B-C7-F1-E0:CMCC	120.197.40.4			4	0	264	0	200
1363157991076 	13926435656	20-10-7A-28-CC-0A:CMCC	120.196.100.99			2	4	132	1512	200
1363154400022 	13926251106	5C-0E-8B-8B-B1-50:CMCC	120.197.40.4			4	0	240	0	200
1363157993044 	18211575961	94-71-AC-CD-E6-18:CMCC-EASY	120.196.100.99	iface.qiyi.com	视频网站	15	12	1527	2106	200
1363157995074 	84138413	5C-0E-8B-8C-E8-20:7DaysInn	120.197.40.4	122.72.52.12		20	16	4116	1432	200
1363157993055 	13560439658	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			18	15	1116	954	200
1363157995033 	15920133257	5C-0E-8B-C7-BA-20:CMCC	120.197.40.4	sug.so.360.cn	信息安全	20	20	3156	2936	200
1363157983019 	13719199419	68-A1-B7-03-07-B1:CMCC-EASY	120.196.100.82			4	0	240	0	200
1363157984041 	13660577991	5C-0E-8B-92-5C-20:CMCC-EASY	120.197.40.4	s19.cnzz.com	站点统计	24	9	6960	690	200
1363157973098 	15013685858	5C-0E-8B-C7-F7-90:CMCC	120.197.40.4	rank.ie.sogou.com	搜索引擎	28	27	3659	3538	200
1363157986029 	15989002119	E8-99-C4-4E-93-E0:CMCC-EASY	120.196.100.99	www.umeng.com	站点统计	3	3	1938	180	200
1363157992093 	13560439658	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			15	9	918	4938	200
1363157986041 	13480253104	5C-0E-8B-C7-FC-80:CMCC-EASY	120.197.40.4			3	3	180	180	200
1363157984040 	13602846565	5C-0E-8B-8B-B6-00:CMCC	120.197.40.4	2052.flash2-http.qq.com	综合门户	15	12	1938	2910	200
1363157995093 	13922314466	00-FD-07-A2-EC-BA:CMCC	120.196.100.82	img.qfc.cn		12	12	3008	3720	200
1363157982040 	13502468823	5C-0A-5B-6A-0B-D4:CMCC-EASY	120.196.100.99	y0.ifengimg.com	综合门户	57	102	7335	110349	200
1363157986072 	18320173382	84-25-DB-4F-10-1A:CMCC-EASY	120.196.100.99	input.shouji.sogou.com	搜索引擎	21	18	9531	2412	200
1363157990043 	13925057413	00-1F-64-E1-E6-9A:CMCC	120.196.100.55	t3.baidu.com	搜索引擎	69	63	11058	48243	200
1363157988072 	13760778710	00-FD-07-A4-7B-08:CMCC	120.196.100.82			2	2	120	120	200
1363157985066 	13726238888	00-FD-07-A4-72-B8:CMCC	120.196.100.82	i02.c.aliimg.com		24	27	2481	24681	200
1363157993055 	13560436666	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			18	15	1116	954	200
1363157993055 	13560436666	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			18	15	111s6	954	200
1363157993055
1363157985066 	13726238888	00-FD-07-A4-72-B8:CMCC	120.196.100.82	i02.c.aliimg.com		24	27	2481	24681	200
```
