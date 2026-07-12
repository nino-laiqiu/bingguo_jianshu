1.总结
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a52731e694f7a218.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.Properties
![image.png](https://upload-images.jianshu.io/upload_images/9049859-00bc8c663d2cfdb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

与HASHMAP 相较的特殊方法
```
import java.util.Properties;
//特殊功能
import java.util.Set;

public class Demo2 {
    public static void main(String[] args) {
        Properties properties = new Properties();
        Object n1 = properties.put("1", new People("小明", "123456"));
        Object n2 = properties.put("2", new People("小童", "12345"));
        Object n3 = properties.put("3", new People("小化", "1234"));
        //MAP集合的遍历
        Set<Object> objects = properties.keySet();
        for (Object key : objects) {
            Object o = properties.get(key);
           // System.out.println(key +" " + o);
        }
        Properties p1 = new Properties();
        //setProperty 方法添加元素
         p1.setProperty("哈哈", "1");
         p1.setProperty("哈哈h", "2");
         p1.setProperty("哈哈hh", "3");
         //返回key值的集合
        Set<String> strings = p1.stringPropertyNames();
        for (String key : strings) {
            String property = p1.getProperty(key);
            System.out.println(property +"  " +key);
        }
    }
}
class People{
    String name;
    String password;

    public People(String name, String password) {
        this.name = name;
        this.password = password;
    }

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```
**修改文本的信息**
![image.png](https://upload-images.jianshu.io/upload_images/9049859-cb88ed102ea40f02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码:
对properties类方法应用
load 加载文本的信息 ---是一个MAP集合
score 把集合的内容设置到文本  如何设置集合的内容  采用方法setproperties 方法
stringPropertyNames 方法 返回集合的键值集合,采取遍历

```
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.io.Serializable;
import java.util.Properties;
import java.util.Set;

public class Demo3 {
    public static void main(String[] args) {
        Properties properties = new Properties();
        //覆盖原数据
        properties.setProperty("1","小猴");
        properties.setProperty("2","小明");
        properties.setProperty("444","小化");
        try {
            FileWriter fileWriter = new FileWriter("C:\\Users\\hp\\IdeaProjects\\untitled1\\s.txt");
            properties.store(fileWriter,"2020");
            fileWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        Properties properties1 = new Properties();
        try {
            FileReader fileReader = new FileReader("C:\\Users\\hp\\IdeaProjects\\untitled1\\s.txt");
            properties1.load(fileReader);
            Set<String> strings = properties1.stringPropertyNames();
            for (String key : strings) {
                String property = properties1.getProperty(key);
                System.out.println(key + "  " +property);
            }
            fileReader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
class  User implements Serializable {
    String name;
    String pasword;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", pasword='" + pasword + '\'' +
                '}';
    }
    public User() {
    }

    public User(String name, String pasword) {
        this.name = name;
        this.pasword = pasword;
    }
}
```
