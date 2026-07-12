###1. Character的用法
	public static boolean isUpperCase(char ch)//判断是否字母为大
	public static boolean isLowerCase(char ch)//为小写
	public static boolean isDigit(char ch)//判断是否为数字
###2.Date SimpleDateFormat的用法
SimpleDateFormat 主要用于将日期进行格式化,可以将Date转成指定格式的字符串,也可以将字符串转成日期
构造方法
		public Date()				表示当前时间的一个对象
		public Date(long date)		得到指定long值对应时间的对象
成员方法
		public long getTime()	                把日期对象转成毫秒值
		public void setTime(long time)		将日期设置为毫秒值对应的时间
构造方法
	        public SimpleDateFormat();					按默认格式做转换
	        **public SimpleDateFormat(String pattern);	按指定格式做转换**
成员方法
 	        **public final String format(Date date)	把 Date  转成String
	        public Date parse(String source)		把String 转成Date**

```
public class SimpleDateFormatTest {
	public static void main(String[] args) throws ParseException {
		String str = "2018-11-25 11:11:11";
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		Date date = sdf.parse(str);
		
		SimpleDateFormat sdf2 = new SimpleDateFormat("HH:mm:ss MM/dd/yyyy");
		String str2 = sdf2.format(date);
		System.out.println(str2);
	}
}
```
```
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class Demo11 {
    public static void main(String[] args) throws ParseException {
        //标准时间
        Date date = new Date(0);
        Date date1 = new Date();
        long time1 = date.getTime();
        long time3 = System.currentTimeMillis();
        date.setTime(time3);
        String st = "1998-04-24 00:00:00";
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date parse = simpleDateFormat.parse(st);
        long time2 = parse.getTime();
        System.out.println((time3-time2)/60/60/60/24);
        System.out.println(date1);
```
###3.正则表达式
```
import java.util.Arrays;

/*我有如下一个字符串:"19 89 76 3 65"
        请写代码实现最终输出结果是："3 19 65 76 89"
        思路: 1. 使用空格对字符串做切割, 得到一个String[]
        2. 创建一个等长的int[] ,
        3. 把String[] 中的每个元素转成int,存到int[]中
        4. 对int[] 做升序
        5. 拼接成想要的结果*/
public class Demo12 {
    public static void main(String[] args) {
        String st ="19 89 76 3 65";
        String[] split = st.split("\\s");
        int[] array = new int[split.length];
        for (int i = 0; i < split.length; i++) {
            array[i] = Integer.parseInt(split[i]);
        }
        Arrays.sort(array);
        System.out.println(Arrays.toString(array));
        StringBuffer stringBuffer = new StringBuffer();
        for (int i = 0; i < array.length; i++) {
            stringBuffer.append(array[i]).append(" ");
        }
        System.out.println(stringBuffer.toString().trim());
    }
}
```
```
String path = "F:\\授课资料\\java基础\\javase06\\day13\\资料";
		String[] arr4 = path.split("\\\\");
		System.out.println(arr4.length);
		System.out.println(arr4[2]);//java基础
//注意是四个\\\\
```

#4.集合
Collection中的方法
boolean add(E e)                           添加
boolean remove(Object o)             移除
void clear()                                     清空
boolean contains(Object o)            是否包含
boolean isEmpty()                          是否为空
int size()                                         集合的长度
boolean addAll(Collection c)                  添加所有
boolean removeAll(Collection c)           移除所有(凡是参数中包含的内容都会被移除
boolean containsAll(Collection c)         是否全部包含
boolean retainAll(Collection c)              取交集
**Object[] toArray()              把集合转成数组，可以实现集合的遍历
Iterator iterator()              得到一个迭代器的对象**
```
ublic class CollectionDemo3 {
	public static void main(String[] args) {
		Collection c = new ArrayList();
		c.add("好好学习");
		c.add("不打游戏");
		c.add("生死看淡");
		c.add("不服就干");
		
		//1.
		Object[] array = c.toArray();
		for(int i=0;i<array.length;i++) {
			System.out.println(array[i]);
		}
		System.out.println("--------------");
		
		//2. 使用迭代器遍历
		//2.1得到迭代器
		Iterator it = c.iterator();
		//2.2
		while(it.hasNext()) {
			System.out.println(it.next());
		}
	}
}
```

##List中的方法
void add(int index,E element)		 插入       
 E remove(int index)         			 把指定索引的值移除掉       
 E get(int index)                  			 索取指定索引对应的元素        
 E set(int index,E element) 			 替换        
 ListIterator listIterator() 			获取ListIterator对象,用于迭代

##**遍历的四种方法**
1. 转数组
2. Iterator
3. for+get
4. ListIterator
例题如下
```
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.ListIterator;

/*String str = "张三,18,男;李四,39,女;王五,36,男";     把这个字符串中分离出三个Person对象,存到一个集合中*/
public class Demo13 {
    public static void main(String[] args) {
     List<Person> strings = new ArrayList<>();
        String str = "张三,18,男;李四,39,女;王五,36,男";
        String[] split = str.split(";");
        for (int i = 0; i < split.length; i++) {
            String[] split1 = split[i].split(",");
            // 将string类型 转为int 和 char 类型
            Person person = new Person(split1[0], Integer.parseInt(split1[1]), split1[2].charAt(0));
            strings.add(person);
        }
        //第一种遍历 for循环
       for (int i = 0; i < strings.size(); i++) {
            System.out.println(strings.get(i));
        }
       //第二种遍历 转数组
        Object[] objects = strings.toArray();
        for (int i = 0; i < objects.length; i++) {
            System.out.println(objects[i]);
        }
        //第三种遍历 迭代器
        Iterator<Person> iterator = strings.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        //第四种遍历 listIterator  list中方法
        ListIterator<Person> personListIterator = strings.listIterator();
        while (personListIterator.hasNext()) {
            System.out.println(personListIterator.next());
        }
        //逆向遍历
        while (personListIterator.hasPrevious()) {
            System.out.println(personListIterator.previous());
        }
    }
}
class  Person{
    String name;
    int age;
    char sex;
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                '}';
    }
    public Person(String name, int age, char sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }
    public Person() {
    }
}
```


