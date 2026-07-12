#####1.定义
hdfs是一个文件系统,用于储存文件,通过目录树来定位文件;其次,它是分布式的.由很多服务器联合起来实现其功能,集群中的服务器有各自角色
适合一次写入,多次读出的场景,不支持文件的修改

#####2.优缺点
优点:
高容错性:
数据自动保存在多个副本,并通过增加副本形式提高容错性
某一个副本丢失以后,它可以自动恢复
适合处理大数据
可构建在廉价机器上,通过多副本机制提高可靠性

缺点:
不适合低延时的数据访问
**无法高效的对大量小文件进行存储,namenode内存总是有限的**
小文件存储的寻址时间会超过读取时间.违法了hdfs设计目标
不支持并发写入及文件随机修改
**一个文件只能一次写,不能多个线程同时写**
仅支持数据追加(append),不支持文件的随机修改

#####大量小文件不适合存储于HDFS的原因


1、小文件过多，会过多占用namenode的内存，并浪费block。
- 文件的元数据（包括文件被分成了哪些blocks，每个block存储在哪些服务器的哪个block块上），都是存储在namenode上的。
HDFS的每个文件、目录、数据块占用150B，因此300M内存情况下，只能存储不超过300M/150=2M个文件/目录/数据块的元数据
- dataNode会向NameNode发送两种类型的报告：增量报告和全量报告。
增量报告是当dataNode接收到block或者删除block时，会向nameNode报告。
全量报告是周期性的，NN处理100万的block报告需要1s左右，这1s左右NN会被锁住，其它的请求会被阻塞。

2、文件过小，寻道时间大于数据读写时间，这不符合HDFS的设计:
HDFS为了使数据的传输速度和硬盘的传输速度接近，则设计将寻道时间（Seek）相对最小化，将block的大小设置的比较大，这样读写数据块的时间将远大于寻道时间，接近于硬盘的传输速度。



#####3.组成架构初步

**namenode:主管 管理者**
管理hdfs的名称空间
配置副本的策略
管理数据块(block)映射信息
处理客户端读写请求

**datanode:执行实际的操作**
储存实际的数据块
执行数据块的读写操作

**client:客户端**
文件切分
与namenode交互,获取文件的位置信息
与datanode交互,读取或者写入数据
client提供一些命令管理hdfs
client提供一些命令来访问hdfs

**secondary:并非namenode的热备,当namenode挂掉的时候,它并不马上替换掉namenode提供服务**
辅助namenode.分担其工作量,比如定期合并fsimage 和 edits ,并推送给namenode
在紧急的情况下 可辅助恢复namenode

#####4.块的大小设置

>  vi hdfs.site.xml

```
<property>
 <name>dfs.block.size</name>
 <value>51200</value>
 </property>
```

参考

>https://blog.csdn.net/xiaozelulu/article/details/80570187?biz_id=102&utm_term=dfs.namenode.fs-limits.min-blo&utm_medium=distribute.pc_search_result.none-task-blog-2~


>https://blog.csdn.net/wx1528159409/article/details/84260023?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160180656419725255545175%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160180656419725255545175&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~



![image.png](https://upload-images.jianshu.io/upload_images/9049859-7b201ab0c1798b4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-d8e8c0fbf8d591e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##5.文件io流上传下载操作

![image.png](https://upload-images.jianshu.io/upload_images/9049859-b6afb9a662608084.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(注意事项)
```
import java.io.File;
import java.io.FileInputStream;
import java.net.URI;
import java.net.URISyntaxException;


public class Test_Hdfs {
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        FileSystem fs  = FileSystem.newInstance(new URI("hdfs://linux03:8020"), configuration, "root");
        //获取输入流
        //必须是一个文件 不然拒绝访问....
        FileInputStream fileInputStream = new FileInputStream(new File("C:\\Users\\hp\\Desktop\\数据结构\\尚硅谷韩顺平老师数据结构分享\\ReadMe.txt"));
        //获取输出流
        //不能是"/"  /p相当是重命名了......
        FSDataOutputStream fsDataOutputStream = fs.create(new Path("/p"));
        //流的拷贝
        IOUtils.copyBytes(fileInputStream,fsDataOutputStream,configuration);
        //关流
        fs.close();
        fileInputStream.close();
        fsDataOutputStream.close();
    }
}
```

```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import java.io.File;
import java.io.FileOutputStream;
import java.net.URI;


public class Test_Hdfs {
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        FileSystem fs  = FileSystem.newInstance(new URI("hdfs://linux03:8020"), configuration, "root");
        //获取输入流
        FSDataInputStream fileInputStream = fs.open(new Path("/p"));
        //获取输出流
        //不能再C路径下直接创建
        FileOutputStream fileOutputStream = new FileOutputStream(new File("C:\\Users\\hp\\Desktop\\数据结构\\p.txt"));
        //流的拷贝
        IOUtils.copyBytes(fileInputStream,fileOutputStream,configuration);
        //关流
        fs.close();
        fileInputStream.close();
        fileOutputStream.close();
    }
}
```

还有这种
```
public class ReadHdfsData {
    public static void main(String[] args) throws Exception {
        FileSystem fs = Utils.getHDFSfs();
        FSDataInputStream fis = fs.open(new Path("/word.txt"));
        fis.seek(3); // skip
       /* fis.read();
        fis.read();
        fis.read();
        int i = fis.read();
        System.out.println(i); //97*/
        // 使用缓冲字符流
        BufferedReader br = new BufferedReader(new InputStreamReader(fis));
        long len = 0 ;
        String line = null ;
         while((line=br.readLine())!=null){
             // line.length() 一行数据的内容长度  每行需要+ 换行长度 在windows中+2 \r\n  在linux中的文件 +1 \r
             len+= line.length()+ 2 ;
             if(len >= 100){
                 break ;
             }
             System.out.println(line);
         }
        System.out.println(len);
 
        fis.close();
        fs.close();
 
    }
}
```

补充一下
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

import java.io.*;
import java.net.URI;


public class Test_Hdfs {
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        FileSystem fs = FileSystem.newInstance(new URI("hdfs://linux03:8020"), configuration, "root");
        //获取输入流
        FSDataInputStream fileInputStream = fs.open(new Path("/p"));
        //字符流
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(fileInputStream));
        //输出流
        BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter("C:\\Users\\hp\\Desktop\\数据结构\\p.txt"));
        String len =null;
        while ((len = bufferedReader.readLine()) != null) {
                 bufferedWriter.write(len);
                 bufferedWriter.newLine();
                 bufferedWriter.flush();
        }
        //关流  顺序

        fileInputStream.close();
        bufferedReader.close();
        bufferedWriter.close();
        fs.close();
    }
}
```

 
