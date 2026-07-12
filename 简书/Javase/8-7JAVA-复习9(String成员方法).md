#1.一个关于例题的代码
```
import java.util.Scanner;

public class Demo2 {
    public static void main(String[] args) {
        //第一种添加方式
       /* Scanner sc =new Scanner(System.in);
        Student [] stu =new Student[2];//这里面的数值 2 可以通过Scanner来获得
        for (int i = 0; i < 2; i++) {
            System.out.println("请输入name");
            String i1 = sc.next();
            System.out.println("请输入age");
            int i2 = sc.nextInt();
            Student student = new Student(i1, i2);
            stu[i] = student;
        }
        for (int i = 0; i < stu.length; i++) {
            System.out.println(stu[i].getAge() + "==" + stu[i].getName());
        }*/
       //第二种添加方式;

       Student[] stu = new Student[2];
        Student s1 = new Student("xiaom", 19);
        Student s2 = new Student("xiaoping", 25);
        stu[0]  =s1;
        stu[1] =s2;
        for (int i = 0; i < stu.length; i++) {
            //System.out.println(stu[i].getAge()+ "====" + stu[i].getName());
            System.out.println(stu[i]);

        }
        for (Student student : stu) {
            System.out.println(student);
        }


    }
}
class Student{
    private  String name;
    private  int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Student() {
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
问题 tostring方法为什么要重写,及打印的结果为什么会那样的显示(println方法自动调用)
对student数组的理解 存一个student 及打印

#2.统计某一段字符串在字符串出现的次数
```
//两种方法
public class Demo4 {
    public static void main(String[] args) {
        String test ="woaijavawxihuanjiavajavabuzhiddjava";
        //方法一
       /* int length = test.length();
        int num =0;
        int count =0;
        while (num < length){
            int java = test.indexOf("java", num);
            if (java == -1){
                break;
            }
            num += java +1;
            count++;
        }
        System.out.println(count);*/
       //方法二
       int count =0;
        int java = test.indexOf("java");
        while (java != -1){
            count++;
            int java1 = test.indexOf("java", java + 1);
            java =java1;

        }
        System.out.println(count);
    }
}
```
#3.字母大小的转换
```
public class Demo6 {
    public static void main(String[] args) {


        String st = "wHUIOPDHFF";
        String s1 = st.substring(1);
        String s2 = st.substring(0, 1);
        String st1 = s1.toLowerCase() + s2.toUpperCase();
        System.out.println(st1);
    }
}
```
#4.String成员方法
boolean contains(String str)                      	是否包含
boolean startsWith(String str)                 		是否以...开始
boolean endsWith(String str)                    	是否以...结束
boolean isEmpty()                                         	是否为空串
boolean equals(Object obj)                        	比较字符串的内容
int length()                                                       获取字符串的长度
char charAt(int index)                    	                得到index(从0开始)对应的字符//索引值
int indexOf(String str)               返回str在字符串中第一次出现的索引值,不存在返回-1
String substring(int start)             	字符串的截取,从start一直截取到最后,包含start
String substring(int start,int end)                     截取,[start-end)包含start,不包含end
char[] toCharArray()                                        转成char数组
```
String s2 = "大家好";
char[] chs = s2.toCharArray();
```
static String valueOf(int i)                               把int 转成字符串
```
 String s4 = String.valueOf(34);
```
String toLowerCase()                                      转成小写
String toUpperCase()                                      转成大写
String concat(String str)   		 	       字符串的拼接     
String replace(char old,char new)                      替换
String replace(String old,String new)                 替换
String trim()                                                            去除首尾空格


#5.hashCode和equals

hashCode: 
	如果两个对象的hashCode值不同,代表他们一定不是同一个对象,
        如果得到的hashCode值相同, 也不一定是同一个对象
	hashCode默认比较的是地址值
equals: 
	判断两个对象是否是同一个对象(默认是== 实现的,可重写)
用法: 
	一般我们在比较对象是否是同一个对象的时候,是同时使用hashCode和equals
          1. 先用hashCode作比较:
                      如果hashCode不同:  一定不是同一个对象
                      如果hashCode相同:  不一定,继续比较equals:
                                    equals:   
                                                true 说明是同一个对象
                                                false:  不是同一个对象
           2. 为了保证equals和 hashCode结果的一致性
                     我们在重写equals方法的同时,也要重写hashCode(约定俗称)
重写hashCode和equals比较属性值的方法

#6.split方法的一个例题
```
public class Demo7 {
    public static void main(String[] args) {
        String st ="小明,20,男;小华,21,男;小里,29,女;小白,80,男";
       // Student1 student1 = new Student1();
        String[] split1 = st.split(";");
        //for (String s : split1) {
            //System.out.println(s);
        Student1 [] array =new Student1[split1.length];
        for (int i = 0; i < split1.length; i++) {
            String[] split2 = split1[i].split(",");
            Student1 student1 = new Student1(split2[0],split2[1],split2[2]);
           /* for (String s : split2) {
                System.out.println(s); }*/
            array[i]= student1;
        }
        for (Student1 student1 : array) {
            System.out.println(student1);
        }

       /* int x =0;
        int y =0;
        for (int i = 0; i < 3; i++) {
            String substring = st.substring(0 + x, 2 + y);
            String substring1 = st.substring(3 + x, 5 + y);
            String substring2 = st.substring(6 + x, 7 + y);
            x +=8;
            y+=8;
            Student1 student1 = new Student1(substring, substring1, substring2);
            array[i] =student1;
        }
        for (Student1 student1 : array) {
            System.out.println(student1);
        }*/

    }
}
class  Student1{
    String name;
    String age;
    String sex;

    @Override
    public String toString() {
        return "Student1{" +
                "name='" + name + '\'' +
                ", age='" + age + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }

    public Student1() {
    }

    public Student1(String name, String age, String sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;

    }
}
```
	    


