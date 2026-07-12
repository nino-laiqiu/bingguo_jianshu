#1.单例设计模式
单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。
注意： 
1、单例类只能有一个实例。
2、单例类必须自己创建自己的唯一实例。
3、单例类必须给所有其他对象提供这一实例。
```
public class SingleObject {
 
   //创建 SingleObject 的一个对象
   private static SingleObject instance = new SingleObject();
 
   //让构造函数为 private，这样该类就不会被实例化
   private SingleObject(){}
 
   //获取唯一可用的对象
   public static SingleObject getInstance(){
      return instance;
   }
 
   public void showMessage(){
      System.out.println("Hello World!");
   }
}
```
```
public class SingletonPatternDemo {
   public static void main(String[] args) {
 
      //不合法的构造函数
      //编译时错误：构造函数 SingleObject() 是不可见的
      //SingleObject object = new SingleObject();
 
      //获取唯一可用的对象
      SingleObject object = SingleObject.getInstance();
 
      //显示消息
      object.showMessage();
   }
}
```
```
Hello World!
```
###懒汉式
```
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
  
    public static Singleton getInstance() {  
    if (instance == null) {  
        instance = new Singleton();  
    }  
    return instance;  
    }  
}
```
###饿汉式
```
ublic class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
}

一个小例题
```
/* 1）定义一个抽象类Animal，其中包括属性name，相关构造方法，抽象方法enjoy()表示动物高兴时动作。

 2）定义Cat类继承于Animal类，其中包括属性eyesColor，相关构造方法，同时具体化父类中的抽象方法。

 3）定义Dog类继承于Animal类，其中包括属性furColor，相关构造方法，同时具体化父类中的抽象方法。

 4）定义Lady类，其中包括属性name,以及Animal 类型的属性pet表示女士所养的宠物，定义构造方法，
        生成女士对象时初始化姓名和她所养的宠物。
        定义一个方法：myPetEnjoy表示此女士的宠物在高兴时的动作。提示：对于此类的定义中需要使用到多态性。*/

 5）定义测试类。
```
public class Demo6 {
    public static void main(String[] args) {
        Cat1 cat1 = new Cat1();
        Lady name = new Lady("nam", cat1);
        name.mypetenjoy();

    }
}
abstract  class Animal1{
    String name;

    public Animal1() {
    }

    public Animal1(String name) {
        this.name = name;
    }

    abstract public  void  enjoy();
}
class  Cat1 extends  Animal1{
String eyeColor;

    public Cat1() {
    }

    public Cat1(String eyeColor) {
        this.eyeColor = eyeColor;
    }

    @Override
    public void enjoy() {
        System.out.println("wwww");

    }
    // 什么叫具体化方法
}
class  Dog extends Animal1{
    String furColor;

    public Dog() {
    }

    public Dog(String furColor) {
        this.furColor = furColor;
    }

    @Override
    public void enjoy() {
        System.out.println("hhhhh");

    }
}
class  Lady{
    String name;
    Animal1 pet;


    @Override
    public String toString() {
        return "Lady{" +
                "name='" + name + '\'' +
                ", animal1=" + pet +
                
    }
    public Lady() {
    }
    public Lady(String name, Animal1 animal1) {
        this.name = name;
        this.pet = animal1;

    }
    //pet ?定义构造方法?
public  void  mypetenjoy(){
        pet.enjoy();
}

}
//class  Pet{
```

#2.多态
三个前提:

	有继承关系

	父类的引用指向子类的对象

	**方法的重写**

四种调用

	同名的成员变量:    父类的

	同名的静态方法: 	父类的

	同名的成员方法: 	子类的

	同名的静态变量:	父类的

	无法调用子类中独有的属性和方法

	父类中独有:    父类的



二种转型

	向上转型: 		Person p = new Teacher();

	向下转型			Teacher t = (Teacher)p;

#3.抽象
用abstract 修饰的类:

         抽象方法格式:

                  abstract 修饰符 返回值类型  方法名(参数列表);

         抽象类的定义格式:

                   abstract class 类名{}

注意事项:

	1. 抽象类中可以没有抽象方法,有抽向方法的类一定是抽象类

	2. 抽象类不能创建对象,需要使用子类向上转型

	3. 抽象的子类要么实现抽象类中所有的抽象方法,要么自己是一个抽象类

	4. 抽象类有构造方法

	5. abstract 不能和final共存


#4.接口
   
定义格式:

                   interface  接口名{}

         注意事项:

		1. 接口中只能定义常量,默认public static final修饰

		2. 接口中只能定义抽象方法(1.8之前) 默认是public abstract 修饰

		3. 接口不能创建对象,使用子类向上转型

		4. 接口的子类: 实现了接口的类	class 子类名 implements  接口1,接口2{}

		5. 接口的子类要么实现接口中所有的抽象方法要么自己是一个抽象类

		6. 一个类可以实现多个接口,并且可以在继承类的同时实现多个接口

		7. 接口中没有构造方法

		8. jdk8之后接口中可以定义已经实现的方法,但是必须使用static/default 修饰

		9. 接口不能实现接口,只能继承接口,并且可以多继承

			class 子类名 implements  接口1,接口2{}

 
#5.抽象类和接口的区别:

1.一个类最多只能继承一个抽象类,但是可以实现多个接口

2.抽象类中既可以定义变量也可以定义常量,接口中只能定义常量

3.抽象类中既可以定义抽象方法,也可以定义非抽象方法,接口中能定义抽象方法(jdk8之前)

4.接口中没有构造方法,抽象类中有构造方法

5.接口只能继承接口不能实现接口,并且可以多继承





	
