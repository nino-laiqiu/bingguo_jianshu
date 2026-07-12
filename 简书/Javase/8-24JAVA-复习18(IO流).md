1.IO流
代码:多级文件夹的复制
```
import java.io.*;

//多级文件夹和单级文件夹的复制
public class Demo2 {
    public static void main(String[] args) {
        File fromFile = new File("C:\\Users\\hp\\Desktop\\课程\\多级文件夹");
        File toFile = new File("C:\\Users\\hp\\Desktop\\课程\\" +fromFile.getName() +"1");
        if (!toFile.exists()){
            toFile.mkdirs();
        }
        copy(fromFile,toFile);
        System.out.println("复制完成");
    }
    private static void copy(File fromFile, File toFile) {
        File[] files = fromFile.listFiles();
        for (File file : files) {
            if (file.isFile()){
                copyFile(file,toFile);
            }
            else  if (file.isDirectory()){
                File newfile1 = new File(toFile, file.getName());
                if (!newfile1.exists()){
                    newfile1.mkdirs();
                }
                copy(file,newfile1);
            }
        }
    }

    private static void copyFile(File file, File toFile)  {

        try {
            File newfilewenjian = new File(toFile, file.getName());
            BufferedInputStream  bufferedInputStream = new BufferedInputStream(new FileInputStream(file));
            BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(new FileOutputStream(newfilewenjian));
            byte[] ch = new byte[200];
            int len =0;
            while ((len=bufferedInputStream.read(ch))!=-1){
                bufferedOutputStream.write(ch,0,len);
            }
            bufferedInputStream.close();
            bufferedOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {

        }

    }
}
```
###代码实现:FileInputStream  FileOutputStream的代码
```
import java.io.*;

public class Deno3 {
    public static void main(String[] args) throws IOException {
        File file = new File("C:\\Users\\hp\\IdeaProjects\\untitled1\\src\\Day23\\a.txt");
        if (!file.isFile()){
            return;
        }
        //写
        /*try {
            FileOutputStream fileOutputStream =new FileOutputStream(file,true);
            fileOutputStream.write("你好2!!!!!".getBytes());
fileOutputStream.write("\r\n".getBytes());
            fileOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }*/
        //System.out.println("执行完毕");
//读
        FileInputStream fileInputStream = new FileInputStream(file);
        //这是不行的
        /*int read = fileInputStream.read();
        char read1 =(char)read;
        System.out.println(read1);
        fileInputStream.close();*/
        //乱码了
        /*int len =0;
        while ((len=fileInputStream.read())!=-1){
            System.out.print((char)len);
        }*/
        byte[] bytes =new byte[1024];
        int len1 =0;
        while ((len1=fileInputStream.read(bytes))!=-1){
            System.out.println(new String(bytes,0,len1));
        }
        fileInputStream.close();
        System.out.println("执行完毕");
    }
}
```
###编码知识
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d94972b67ee475ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-678fd02bf5abf369.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![编码一致性](https://upload-images.jianshu.io/upload_images/9049859-d71b51fb23ce35ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
编码一致性问题
![读和写的编码要一制](https://upload-images.jianshu.io/upload_images/9049859-899680aa61fd1b0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
UTF-8编码 如果s.txt是用GBK编码写的则读不出来 要保证文本书写编码一致性的问题

![flush](https://upload-images.jianshu.io/upload_images/9049859-3e92b65c80b5f545.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**解释了flush 为什么要刷新**
注意 方法void  write(int c) 是写一个字符  写入文本的是一个字符
![面试题](https://upload-images.jianshu.io/upload_images/9049859-61392367b7b96b60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![字符流](https://upload-images.jianshu.io/upload_images/9049859-8f45cef400ed3854.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![注意一下](https://upload-images.jianshu.io/upload_images/9049859-8f814b29e68b970e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![为什么要加flash](https://upload-images.jianshu.io/upload_images/9049859-44158d83af065f1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意一下
```
import java.io.*;

public class Demo4 {
    public static void main(String[] args) throws IOException {
        /*try {
            FileOutputStream fileOutputStream = new FileOutputStream("s.txt");
            fileOutputStream.write("中国".getBytes());
            fileOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }*/
       /* try {
            OutputStreamWriter gbk = new OutputStreamWriter(new FileOutputStream("s.txt"), "UTF-8");
            gbk.write("中国");
            gbk.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            //UTF-8编码 如果s.txt是用GBK编码写的则读不出来 要保证文本书写编码一致性的问题
            InputStreamReader inputStreamReader = new InputStreamReader(new FileInputStream("s.txt"),"UTF-8");
            //读取数据一次读一个字符
            //...............
            int len =0;
            while ((len=inputStreamReader.read())!=-1){
                System.out.print((char)len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }*/
       /* InputStreamReader inputStreamReader = new InputStreamReader(new FileInputStream("a.txt"));
        inputStreamReader.close();*/
        //读文件必须要先有文件

        /*OutputStreamWriter outputStreamWriter = new OutputStreamWriter(new FileOutputStream("l.txt"));
        outputStreamWriter.close();*/
        //写文件则不需要直接创建文件
        InputStreamReader inputStreamReader = new InputStreamReader(new FileInputStream("s.txt"));
        OutputStreamWriter outputStreamWriter = new OutputStreamWriter(new FileOutputStream("c.txt"));
        char[] ch =new char[111];
        int len =0;
        while ((len=inputStreamReader.read(ch))!=-1){
            outputStreamWriter.write(ch,0,len);
            outputStreamWriter.flush();
        }
        inputStreamReader.close();
        outputStreamWriter.close();


    }
}
```
**问题:转换流和简化流的溯源
因为转化流书写复杂,所以采取了简化流**
![概述](https://upload-images.jianshu.io/upload_images/9049859-5a9709044354976b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)









