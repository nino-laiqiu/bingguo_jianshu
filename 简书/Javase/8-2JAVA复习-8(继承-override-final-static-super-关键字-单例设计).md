#1.javabean规范
avabean: 普通类, 只是用来描述一些对象.   Teacher,Student,Mobile,Rect,Circle

1. 成员变量私有化
2. 提供getters/setters
3. 提供无参的构造方法
4. 提供有参的构造方法
##在不同类中 
```
public class Demo10 {
    public static void main(String[] args) {
        User user = new User("9","0","0");//构造方法赋值
        User user1 = new User();
        user1.setId("9");
        user1.setName("9");
        user1.setUsername("0"); // 通过私有化set get 赋值
        User user2 = new User();
user2.id = "0";

    }
}class User{
   String name;
    private String id;
    private String username;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", id='" + id + '\'' +
                ", username='" + username + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public User() {
    }

    public User(String name, String id, String username) {
        this.name = name;
        this.id = id;
        this.username = username;
    }
}
```
##在一个类里
public class Product {

        private String proId;
        private String proName;
        private double proPrice;
        private int proNum;

        public String getProId() {
            return proId;
        }
        public void setProId(String proId) {
            this.proId = proId;
        }
        public String getProName() {
            return proName;
        }
        public void setProName(String proName) {
            this.proName = proName;
        }
        public double getProPrice() {
            return proPrice;
        }
        public void setProPrice(double proPrice) {
            this.proPrice = proPrice;
        }
        public int getProNum() {
            return proNum;
        }
        public void setProNum(int proNum) {
            this.proNum = proNum;
        }


        public Product() {

        }
        public Product(String proId, String proName, double proPrice, int proNum) {
            this.proId = proId;
            this.proName = proName;
            this.proPrice = proPrice;
            this.proNum = proNum;
        }

        //三种赋值
        public static void main(String[] args) {
            //1.
            Product p1 = new Product();
            p1.proId = "0001";
            p1.proName="拉条子";
            p1.proPrice = 5;
            p1.proNum = 100;
            System.out.println(p1);



            System.out.println(p1);
            //2.
            Product p2 = new Product();
            p2.setProId("0002");
            p2.setProName("罐头");
            p2.setProPrice(10);
            p2.setProNum(90);
            //System.out.println(p2);

            //3
            Product p3 = new Product("0003","袜子",10,1000);

        }
    }

>注意三种赋值方式 在同一个类中有三种赋值 在不同类里不能直接调用

2.
>https://www.cnblogs.com/yb38156/p/9599820.html
#覆盖 和 重载

##构造方法
　　又叫构造器，构造函数。通常有无参构造器和带参数的构造器2种，每个类都有一个构造方法(如果没有显式的给出来，
那么也有一个默认的无参构造器)无返回类型修饰符。访问修饰符可以是public，也可以是private，比如常见的单例模式就要求构造函数私有化。
##类方法
　　static修饰符修饰的方法。因为static修饰的方法是属于类的而不是属于实例的(这个描述各种书籍上非常常见)，因此不必去new 一个实例来调用，而是直接类名.方法名来调用这种方法，因此也称为类方法。
##final方法
　　最终的、不可改变的、终极的方法。怎么叫都行，单词修饰很清楚了，用了final表明设计上不再会去修改他，很巧，一个叫abstract的修饰符就是要设计者去实现去重写的，因此可以知道final和abstract永远不能共存。
#3覆盖
　　又叫重写，Override。多态是java的特性，覆盖是表现多态特性的具体做法。
##重载
　　重载只发生在一个类中，记住这点很重要，这也是跟覆盖这个概念撇清关系的最重要一点。重载要求同一个类中方法名相同而参数列表不同。参数列表，就是入参的个数，类型，顺序。抓住定义中的这两点，其他的通过什么返回值，访问权限，异常来重载一个方法那就是扯淡，混淆，不行。

#构造方法，类方法，final方法均可以被重载
#构造方法，类方法，final方法都不能覆盖



#3.对java中public、static的理解
>static表示“全局”或者“静态”的意思，用来修饰成员变量和成员方法，也可以形成静态static代码块，但是Java语言中没有全局变量的概念
用public修饰的static成员变量和成员方法本质是全局变量和全局方法，当声明它类的对象市，不生成static变量的副本，而是类的所有实例共享同一个static变量
```
public class Demo1 {
    public static int i = 1;
    public int j = 2;
    
    public static int getNumber(){
        return i;
//        这个return返回的是全局变量的i，即前面多创建的i
    }
    
    public int getDealNumber(int j){
        return j;
//        这个return返回的是所传进来的参数，是(int j)这个东西
    }
}

public class Demo2 {
    public static void main(String args[]){
//        想要得到Demo1中的静态的（即全局的）变量i，直接用类名引用就可以了
        int i = Demo1.i;
//        但是想要得到Demo1中的实例的变量j，我需要怎么做呢？（此刻牢记java面向对象的思想！）
//        首先我要先new一个Demo1的对象,然后才可以通过new出来的对象得到Demo1中的j
        Demo1 demo1 = new Demo1();
        int j = demo1.j;
//        同理，java中的static方法和非static方法都是一样的区别
//        下面一行的方法是静态的,我可以直接根据类名调用方法
        int ii = Demo1.getNumber();
//        但是想要调用实例的方法,就需要利用前面所new出来的Demo1的对象来调用了
        int jj = demo1.getDealNumber(1);
```
>静态方法可以直接通过类名调用，任何的实例也都可以调用静态方法。前面的代码中有直接通过类名调用的例子静态方法中不能用this和super关键字，不能直接访问所属类的实例变量和实例方法(就是不带static的成员变量和成员成员方法)，只能访问所属类的静态成员变量和成员方法。因为实例成员与特定的对象关联！就是java面向对象的思想，实例是你这个类本身的属性，你会用这个本身的属性去做一些事情，而这些事情不是固定的，不能像静态方法一样一成不变。换位思考，人是一个java类，手是类的实例方法，而人身上有手机，这个手机就是静态方法；static方法必须被实现，而不能是抽象的abstract。静态方法是类内部的一类特殊方法，只有在需要时才将对应的方法声明成静态的，一个类内部的方法一般都是非静态的

#4.同名成员变量和局部变量
```
public class Person {
	String name = "张三";
	
	public void print(){//1
		System.out.println(name);
	}
	public void print(String name){//2
		System.out.println(name);//
	}
	
	public void print1(String name){//3
		System.out.println(this.name);
	}
	public static void main(String[] args) {
		Person p = new Person();
		System.out.println(p.name);//
		
		p.name = "李四";
		p.print();//
		
		p.print("王五");//
		
		p.print1("赵六");//
	}
}



//结果:

//张三
//李四
//王五
//李四
```
# 5. static关键字

1.类中的非静态成员 使用对象调用

2.静态的方法中只能调用外部用static修饰的变量和方法,如果想要调用非静态的,必须要创建对象

3.static 用于修饰变量和方法,被static修饰的变量和方法就变成了静态变量和静态方法



#6.继承
java中的继承只支持单继承,不支持多继承(C++),支持多层继承

Object:(万类之祖) 如果一个类没有继承任何类,那么他默认继承自Object

父类中的私有成员不能被继承

###子类中成员变量的关系
	先去子类中找,再去父类中找,没有就报错
 	super:  当子类中的成员(变量,方法)和父类中的成员重名的时候, super: 指代的父类中的
 	super只能在子类中用

#7.super关键字

super: (父类中的对象) 用于子类的成员和父类的成员重名是,		super 指代父类中的

this: (本类中的对象)  用于成员变量和局部变量重名时,   this指代的是成员变量

在静态的方法中是不能使用this/super

调用成员:

	this.成员(成员变量,成员方法) 

	super.成员(成员变量,成员方法) 

#8. final 关键字

可以用来修饰变量和方法



 final: 修饰符,  可以用来修饰 类, 方法, 变量
		被final修饰的类不能被继承
		被final修饰的方法不能被重写
		被final修饰的变量值不能改变
 	        final  修饰的变量一定要有初始值
         	final 修饰的引用数据数据: 地址值不能改变, 可以改变里面的属性值
  	        一般我们在定义常量的时候,往往是final 和 static 一起使用

#9. 单例设计模式

目的: 让类只能产生一个对象

#10形参和实参:
 	形参:  定义方法时参数列表上的变量
 	实参:  调用方法时传进去的值
 	基本数据类型做参数,形参的改变不影响实参的值
 	引用数据类型做参数,形参的改变影响实参的值(String 和 包装类除外)

一道题目
```
/*编写一个Java应用程序，该程序包括3个类：Monkey类、People类和主类E。要求：
        (1) Monkey类中有个构造方法：Monkey (String s)，并且有个public void speak()方法，在speak方法中输出“咿咿呀呀......”的信息。
        (2)People类是Monkey类的子类，在People类中重写方法speak(),在speak方法中输出“小样的，不错嘛！会说话了！”的信息。
        (3)在People类中新增方法void think()，在think方法中输出“别说话！认真思考！”的信息。
        (4)在主类E的main方法中创建Monkey与People类的对象类测试这2个类的功能。*/
public class E {
    public static void main(String[] args) {
        //Monkey monkey = new Monkey("4");
        //System.out.println(monkey.s);
        Monkey monkey = new Monkey();
        monkey.speak();
        People people = new People();
        people.speak();
    }
}
class Monkey{
    String s;

    public Monkey() {
    }

    public Monkey(String s) {
       // String s=
     // s ="q";
        //System.out.println(s);

        this.s = s;
    }
    public void  speak(){
        System.out.println("咿咿呀呀");
    }
}
class  People extends Monkey{
    @Override
    public void speak() {
        //super.speak();
        System.out.println("别说话,认真思考");
    }
    public  void think(){
        System.out.println("别说话2");

    }
}
```







 



