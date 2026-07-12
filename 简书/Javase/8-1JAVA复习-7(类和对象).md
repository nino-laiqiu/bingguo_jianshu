1.多维数组

```
public class Demo2 {
    public static void main(String[] args) {



       int [][] array = new int[3][4];
        System.out.println(Arrays.deepToString(array));
        array[0]=new int[]{1,2,3,7,8,8,8,9,90};//[[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]
        array[2] =new int[]{1,3,4,5,6,7};
        array[0][6] =4;
        array[1] =new int[]{2,3};
        //array[1] ={2};
        System.out.println(array.length);//3
        System.out.println(Arrays.deepToString(array));
        System.out.println(array[0][4]);//8
        System.out.println(array[0]);//地址值
        for (int i = 0; i < array[0].length; i++) {
            System.out.print(array[0][i]);// 1 2 3 7 8 8
        }
        for (int[] arrayA : array){
            for (int arrayB : arrayA){
                System.out.print(arrayB +" ");
            }
        }
}
}
```
2.面对对象
类:  一种数据类型,引用数据类型,自定义的一种类型.用变量表示属性,用方法表示行为
对象:  具体存在的事物,符合类的定义特征

如何创建对象:
	类名  对象名 = new 类名();




```
ublic class Demo4 {
    public static void main(String[] args) {


        Person person = new Person();
        person.hight=180.1;
        person.name="问题";
        person.sex='男';
        person.year=10;
        person.eat("栗子");
        person.sleep();
        person.get();
    }
}

class Person{
    String name;
    char sex;
    double hight;
    int year;
    public void  eat(String eat){
        System.out.println("吃饭" + eat );

    }
    public  void  sleep(){
        System.out.println("睡觉");

    }
    public  void  get(){
        System.out.println("姓名:" + name + "," +"性别:" + sex+"," + "身高:" + hight +"," + "年龄:" +year);
    }

}
```
```
public class Demo5 {
    public static void main(String[] args) {
        Teacher t = new Teacher();
        t.name = "小涛";
        t.age = 18;
        t.gender = '男';
        t.salary = 600000;

        Teacher t2 = new Teacher();
        t2.name = "找砍";
        t2.age = 48;
        t2.gender = '女';
        t2.salary = 200;

        Teacher t3 = new Teacher();
        t3.name = "小娟";
        t3.age = 28;
        t3.gender = '女';
        t3.salary = 20000;

        // 数据类型[] 数组名 = new 数据类型[长度];
        Teacher[] teachers = new Teacher[3];
        teachers[0] = t;
        teachers[1] = t2;
        teachers[2] = t3;

        Teacher[] teachers2 = new Teacher[] {t,t2,t3};

        for(int i=0;i<teachers.length;i++) {
            Teacher teacher = teachers[i];//teacher: 代表的是数组中的每一个老师
            teacher.teach();
            System.out.println(teacher.name);
            System.out.println(teachers[i].name);

        }
    }
}
 class Teacher {
    //属性: 成员变量
    String name;
    char gender;
    double salary;
    int age;

    static int a;//静态变量

    //成员方法
    public void teach() {
        System.out.println("毁人不倦");
    }

    public static void test(int x,int y) {//静态方法
        int abc = 100;//局部变量
    }
}
```
3.成员变量和局部变量
成员变量： 定义在类中方法外,没有static修饰, 存储在堆内存中,有初始值; 随着对象的创建而产生,随着对象的消失而消失
局部变量: 定义在方法中或者是方法的参数列表上,存储在栈内存中,没有初始值;局部变量随着方法的调用而产生,随着方法的结束而消失

4.私有化提供get和set的方法
```
public static void main(String[] args) {


        Rect rect = new Rect();
rect.setLon(17);
        System.out.println(rect.setLon(17));


    }
}
class  Rect {
    private int lon;
    private int wid;


    public int setLon(int lon) {
        if (lon > 7) {
            System.out.println("不合法");
           this.lon = -12;
            return this.lon;
            //return
        }

            this.lon = lon;
            return this.lon;


        }
    }
```

```
Rect rect = new Rect();
rect.setLon(17);
System.out.println(rect.getLon());


    }
}
class  Rect {
    private int lon;
    private int wid;

    public int getLon() {
        return lon;
    }

    public void setLon(int lon) {
        if (lon > 5){
            System.out.println("错误");
            this.lon =7;
            return;//return 的作用在于由于上一项已经赋值了 如果没有return就会执行下一下重新赋值
        }
        this.lon = lon;
    }
}
```

5.构造方法
造方法的格式:
修饰符   类名(参数列表){
	方法体;
}


构造方法的注意事项:

1. 构造方法没有返回值,连void都没有
2. 方法名和类名相同
3. 构造方法是可以重载的(一个类中可以存在多个名字相同的方法,但是必须保证参数的个数或类型不同,与返回值无关
构造方法何时被调用:
使用new关键字创建对象的时候
如果我们没有在类中写构造方法,系统会默认为我们生成一个无参的构造方法

如果我们自己写了构造方法,那么系统则不再为我们提供

