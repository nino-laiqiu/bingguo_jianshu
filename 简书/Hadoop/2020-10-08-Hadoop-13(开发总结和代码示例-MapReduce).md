##1.MapReduce原理

**maptask工作机制:**

read阶段 默认testinputformat一行一行读取数据
map阶段  返回相应(k,v),并写入到context(k,v)
collect阶段 向环形缓冲区写入(k,v)数据,默认100m 80%反向 分区(可以重写分区方法)排序(按分区规则来排序 所以重写这个排序没有意义)
溢出阶段 溢出到文件
combine阶段 merge并归排序(此处有接口combiner(减少了io传输),特定业务)(这里排序吗???)

**reducetask工作机制**

copy阶段:所以maptask结束.reducetask端主动拉取数据,下载到reduce本地磁盘(存不下溢出到磁盘,存的下就在内存中)
merge阶段 sout 阶段  合并文件并归排序(groupcomparater方法)
reduce阶段 相同keycopy到一个迭代器中

**shuffle机制 (map之前 reduce之后)**



##2.开发总结:

数据块与数据切片

输入数据接口:Inputformat
Textinputformat(默认)
keyvaluetextinputformat
combinetextinputformat(处理小文件)
自定义Fileinputformat

combiner框架(减少了io传输)

map reduce端的方法(setup cleanup的应用)

重写分区规则

重写排序规则
:部分排序(maptask)
:全排序(reducetask设置为一个)
:辅助排序(reducetask)

输出数据接口Outputformat
默认
自定义

join开发
reduce端join
map端join


##配置详情

>hadoop-env.sh

```
export JAVA_HOME=/usr/apps/jdk1.8.0_141
```

>core-site.xml

配置程序操作的文件系统
```
<property>
<name>fs.defaultFS</name>
<value>hdfs://doit01:9000</value>
</property>
```
>HDFS常用参数
```
确定namenode运行的机器,确定secondary namenode运行的机器,确定name元数据存储目录 ,确定data存储目录
<property>
<name>dfs.namenode.rpc-address</name>
<value>doit01:9000</value>
</property>
<property>
<name>dfs.namenode.secondary.http-address</name>
<value>doit02:50090</value>
</property>
<property>
<name>dfs.namenode.name.dir</name>
<value>/opt/hdpdata/name/</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>/opt/hdpdata/data/</value>
</property>
```
>yarn常用参数
```
确定resourcemanager的位置,为MR程序提供shuffle服务 nodemanager的内存 ,cpu ,内存检查设置
<property>
<name>yarn.resourcemanager.hostname</name>
<value>doit01</value>
</property>
<!--  为mr程序提供shuffle服务 -->
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<!--  一台NodeManager的总可用内存资源 -->
<property>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>2048</value>
</property>
<!--  一台NodeManager的总可用（逻辑）cpu核数 -->
<property>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>2</value>
</property>
<!--  是否检查容器的虚拟内存使用超标情况 -->
<property>
  <name>yarn.nodemanager.vmem-check-enabled</name>
  <value>false</value>
</property>
<!--  容器的虚拟内存使用上限：与物理内存的比率 -->
<property>
  <name>yarn.nodemanager.vmem-pmem-ratio</name>
  <value>2.1</value>
</property>
```

>MapReduce参数

```
MapReduce程序的一般设置参数
<--!MR程序默认运行在yarn上  -->
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
<property>
  <description> 跨平台运行 If enabled, user can submit an application cross-platform
  i.e. submit an application from a Windows client to a Linux/Unix server or
  vice versa.
  </description>
  <name>mapreduce.app-submission.cross-platform</name>
  <value>false</value>
</property>
如果主程序运行的机器上,hadoop中对应的配置文件有做对应的设置 ,那么以下参数是读取的配置文件
如果在程序中再设置会覆盖配置文件中的配置
如果配置文件中没有对应的设置 ,那么我们可以做出对象的设置
System.setProperty("HADOOP_USER_NAME", "root");
Configuration conf = new Configuration();
 设置访问的集群的位置
 conf.set("fs.defaultFS", "hdfs://doit01:9000");
 设置yarn的位置
 yarn的resourcemanager的位置
conf.set("yarn.resourcemanager.hostname", "doit01");
 设置MapReduce程序运行在windows上的跨平台参数
 conf.set("mapreduce.app-submission.cross-platform","true");
```

##JOIN案例 reduce端:
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * Created with IntelliJ IDEA.
 * Description:
 * User: tongyongtao
 * Date: 2020-10-08
 * Time: 22:32
 * 简单join的实现 两张表  实现在reduce端的join
 * 首先在setup方法中获取文件的名字 比如 order.txt 和 pd.txt
 * key 和 value 的设置,两张表有重合部分作为key 这样排序后在同一个迭代器中
 * 集合遍历取代原来的
 * order.txt  id pid amount
 * pd.txt   pid pname
 * ...这个案例检查了好久 出现如下bug :dataInput和 dataOutput写入读取的顺序不一致,spilt[]的顺序不一致
 * 没有往集合中添加数据........
 */
public class MR_Join1 {
    static Configuration con;

    public static void main(String[] args) throws Exception {
        args = new String[]{"C:\\Users\\hp\\IdeaProjects\\GitHub_Maven\\src\\main\\resources\\jointask", "C:\\MAPREDYCE1"};
        con = new Configuration();
        Job job = Job.getInstance(con, "MR_Join1");
        job.setMapperClass(Map_Join1.class);
        job.setReducerClass(Reduce_Join1.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(TableBean1.class);
        job.setOutputKeyClass(TableBean1.class);
        job.setOutputValueClass(NullWritable.class);
        job.setNumReduceTasks(1);
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.out.println(job.waitForCompletion(true) ? "zq" : "cw");
    }

    public static class Map_Join1 extends Mapper<LongWritable, Text, Text, TableBean1> {
        String name = null;

        @Override
        protected void setup(Context context) throws IOException, InterruptedException {
            //获取读取的文件的名字
            FileSplit inputSplit = (FileSplit) context.getInputSplit();
            name = inputSplit.getPath().getName();
        }

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            //按行切
            TableBean1 tableBean = new TableBean1();
            Text text = new Text();
            String[] split = value.toString().split("\\s+");
            if (name.startsWith("order")) {
                tableBean.setID(Integer.parseInt(split[0]));
                tableBean.setPid(split[1]);
                tableBean.setAmount(Integer.parseInt(split[2]));
                tableBean.setPname(" ");
                tableBean.setFlog("order");
                text.set(split[1]);
            } else {
                tableBean.setID(0);
                tableBean.setPid(split[0]);
                tableBean.setAmount(0);
                tableBean.setPname(split[1]);
                tableBean.setFlog("pd");
                text.set(split[0]);
            }
            context.write(text, tableBean);
        }
    }

    public static class Reduce_Join1 extends Reducer<Text, TableBean1, TableBean1, NullWritable> {
        @Override
        protected void reduce(Text key, Iterable<TableBean1> values, Context context) throws IOException, InterruptedException {
            //存放订单表
            List<TableBean1> tableBean1s = new ArrayList<>();
            //存放pd表
            TableBean1 tableBeanPD = new TableBean1();
            for (TableBean1 value : values) {
                TableBean1 clone = null;
                if ("order".equals(value.getFlog())) {
                    try {
                        clone = (TableBean1) value.clone();
                    } catch (CloneNotSupportedException e) {
                        e.printStackTrace();
                    }
                    tableBean1s.add(clone);
                } else {
                    try {
                        tableBeanPD = (TableBean1) value.clone();
                    } catch (CloneNotSupportedException e) {
                        e.printStackTrace();
                    }
                }
            }
            for (TableBean1 tableBean1 : tableBean1s) {
                tableBean1.setPid(tableBeanPD.getPname());
                context.write(tableBean1, NullWritable.get());
            }
        }
    }
}
```







