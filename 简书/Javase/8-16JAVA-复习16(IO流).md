**1.Arraylist打印有[ ] 和 "," 把它去掉  但是怎么重写tostring方法  此处提供另一种  形式如下  liststring(list)
//如果是一个用户类,则可以重写tostring 简单有参数  形式如下 list.tostring**
```
import java.util.ArrayList;
import java.util.List;

public class Demo2  {
  

    public static void main(String[] args) {
        ArrayList<String> objects = new ArrayList<>();
        objects.add("uu");
        objects.add("uu1");
        objects.add("uu2");
        objects.add("uu3");
        System.out.println(listToString(objects));

    }
    public static String listToString(List<?> list) {
        String result = null;
        for (int i = 0; i < list.size(); i++) {
            result += " " + list.get(i);
        }
        return result;
    }
}
```
2.IO流
ByteArrayInputStream、StringBufferInputStream、FileInputStream是三中基本的戒指，他们分别从数组、StringBuffer、和本地文件中读取数据。
ByteArrayOutputStream、FileOutputStream 是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。
InputStreamReader 、OutputStreamWriter 要InputStream或OutputStream作为参数，实现从字节流到字符流的转换

##缓冲流
![内存可以省略](https://upload-images.jianshu.io/upload_images/9049859-360d1a0ddd07e9fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
