##面向对象
#####两大编程思想
![面对对象和面对过程](https://upload-images.jianshu.io/upload_images/9049859-1747a2efe26b855d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####类和对象的创建
类是多个类似事物组成的群体的统称.能够帮助我们快速理解和判断事物的性质

在python中一切皆对象
```
class Cat:
    pass


print(Cat)
print(type(Cat))
print(id(Cat))
```
def _init_ 在类之外定义的称为函数,在类之内定义的称为方法


对象的创建又称为类的实例化
语法是: 实例名 = 类名()

方法`__init__()`定义成包含三个形参：self、name和id。在这个方法的定义中，形参self必不可少，而且必须位于其他形参的前面。为何必须在方法定义中包含形参self呢？因为Python调用这个方法来创建Dog实例时，将自动传入实参self。每个与实例相关联的方法调用都自动传递实参self，它是一个指向实例本身的引用，让实例能够访问类中的属性和方法。
```
class Dog:
    # 类属性
    native_num = 1

    # 初始化属性
    def __init__(self, name, id):
        self.name = name
        self.id = id

    # 实例方法
    def sit(self):
        print('我是实例方法')

    # 静态方法
    @staticmethod
    def method():
        print('我是静态方法')

    # 类方法
    @classmethod
    def cls(cls):
        print('我是类方法')


dog = Dog('小米', 123)
print(dog)
print(type(dog))
print(id(dog))


Dog.sit(dog)  # 类名.方法名
dog.sit()   # 对象名.方法名
```
对象名.方法名??这个在Java中不是是调用静态方法的吗
#####类属性&类方法&静态方法
1. 类属性:类中方法外的变量称为类属性,被该类的所有对象所共享
2. 类方法:使用类名直接访问的方法
3. 静态方法:使用类目直接访问的方法
#####动态绑定属性和动态绑定方法
```
class Pig:
    native_num = 1

    def __init__(self, name, id):
        self.name = name
        self.id = id

    def eat(self):
        print(self.name, '吃')


pig1 = Pig('A', '100')
pig2 = Pig('B', '200')
print(pig1, pig2)
# 动态绑定参数
pig1.gender = '公的'
print(pig1, pig2)

pig1.eat()
pig2.eat()
print(pig1.gender)


# 动态绑定方法

def ship():
    print('......')


pig1.ship = ship
# pig1.ship,注意要带括号
pig1.ship()
```
##面对对象的特性
1. 封装:提高程序的安全性
2. 继承:提高代码的复用性
3. 多态:提高程序的可扩展性和可维护性
#####封装
```
class A:
    def __init__(self,name,id):
        self.name = name
        self.__id = id

    def a(self):
        print(self.__id)


a1 = A(name='xiaom', id='0001')
print(a1)
print(a1.name)
# print(a1.__id) 无法访问

print(dir(a1))
# 其实也可以访问
print(a1._A__id) 
```
#####继承
多继承
#####方法重写
```
class Animal:
    native_num = 0

    def __init__(self, name, uid):
        self.name = name
        self.uid = uid

    def eat(self):
        print(self.name)


class Person(Animal):
    native_num = 1

    def __init__(self, name, uid, gender):
        super().__init__(name, uid)
        self.gender = gender

    def info(self):
        print(self.name, 'Person')


p = Person('a', 1111, '男')
p.eat()


class Student(Person):
    native_num = 10

    def __init__(self, name, uid, gender):
        super().__init__(name, uid, gender)

    def info(self):
        super(Student, self).info()
        #  super().info()   
        print(self.name, 'Student')


s = Student('s', 111111, '男')
s.info()
```
#####object类
object类是所有类的父类,因此所有类都有object类的属性和方法,内置函数dir()可以查看指定对象所有属性

`__str__`函数的使用,相当于Java中的toString();return要直接返回字符串的
```
class A:
    def __init__(self, name, uid):
        self.name = name
        self.uid = uid

    def __str__(self):
        return f'{self.name}.....{self.uid}'


s = A('小米', '10001')
print(s)
```
#####多态
即便是不知道一个变量所引用的对象到底是什么类型,仍然可以通过这个变量调用方法,在运行过程中根据变量所引用对象的类型,动态决定调用哪个对象中的方法

**静态语言和动态语言关于多态的区别**
1. 静态语言实现多态的三个必要条件:继承,方法重写,父类引用指向子类对象
2. 动态语言的多态崇尚"鸭子类型"当看到一只鸟走起来向鸭子,游泳起来像鸭...那么这只鸟就可以被称作鸭子,在鸭子类型中,不需要关心对象是什么类型,到底是不是鸭子,只关心对象的行为
```
class Cat:
    def __init__(self, name):
        self.name = name


    def eat(self):
        print('cat')


class Dog:
    def __init__(self, name):
        self.name = name


    def eat(self):
        print('Dog')


def fun(a):
    a.eat()


fun(Cat(''))
fun(Dog(''))
```
#####特殊方法&特殊属性
![特殊方法&特殊属性](https://upload-images.jianshu.io/upload_images/9049859-d1890b5ea05a3832.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意这里:other.name!!!
```
a = 1
b = 2
print(a + b)
print(a.__add__(b))


class A:
    def __init__(self, name):
        self.name = name

    def __add__(self, other):
        return self.name + other.name


a1 = A('AAA')
a2 = A('AAA')
print(a1.__add__(a2))

lst = [1, 2]
print(len(lst))


class B:
    def __init__(self, name):
        self.name = name

    def __len__(self):
        return len(self.name)


b1 = B('BBB')
b2 = B('bbb')
print(len(b1))
```
![__new__](https://upload-images.jianshu.io/upload_images/9049859-3ff01bce96c45576.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####类的浅拷贝和深拷贝
变量的操作赋值:只是形成两个变量,实际上还是指向同一个对象
浅拷贝:python拷贝一般都是浅拷贝,拷贝时,**对象包含的子对象内容不拷贝**,因此,源对象和拷贝对象引用同一个子对象
```
class Name:
    def __init__(self):
        pass


class Gender:
    def __init__(self):
        pass


class Student:
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender


name = Name()
gender = Gender()

student = Student(name, gender)

import copy

student1 = copy.copy(student)

print(id(student), id(student.name), id(student.gender))
print(id(student1), id(student1.name), id(student1.gender))

'''
2294896209504 2294896211472 2294896210512
2294888037440 2294896211472 2294896210512

'''
```

深拷贝:使用copy模块的deepcopy函数,递归拷贝对象中包含的子对象,源对象和拷贝对象所有的子对象也不相同
#####小结
![面对对象小结](https://upload-images.jianshu.io/upload_images/9049859-95fbb7976baf6628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
[Python编程：从入门到实践（第2版）-埃里克·马瑟斯-微信读书 (qq.com)](https://weread.qq.com/web/reader/08232ac0720befa90825d88k6f4322302126f4922f45dec)

##模块和包
#####什么叫模块
#####模块的导入
1. python自带模块
```
import math  # as ma
from math import pi
print(math.pi)
print(pi)
```
2. 自己写的模块,在pycharm中要点击sources root
#####以主程序方式运行
在每个模块的定义中都包含一个记录模块名称的变量 `__name__`,程序可以检查该变量,以确定他们在哪个模块运行.如果一个模块不是被导入其他程序中执行,那么它可能在解释器的顶级模块中执行
```
if __name__ == '__main__':
    pass
```

#####什么叫包
包是一个分层次的目录结构,它将一组功能相近的模块组织在一个目录下
**包和目录**
1. 包含`__init__,py`文件的目录称为bao
2. 目录里通常不包含`__init__,py`文件
#####常用的内容模块
![常用的内容模块](https://upload-images.jianshu.io/upload_images/9049859-cdb68eeb901cc770.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####第三方模块的安装和使用
pip install 模块名称
![在pycharm中安装第三方库](https://upload-images.jianshu.io/upload_images/9049859-acd5c090c7f55078.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##文件操作
#####编码格式
![常见的字符编码格式](https://upload-images.jianshu.io/upload_images/9049859-83511c18b5cd343a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####文件读写的原理
![IO原理](https://upload-images.jianshu.io/upload_images/9049859-767089dfaf270dbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![文件读写操作](https://upload-images.jianshu.io/upload_images/9049859-1e16ca2b65a3bce9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####常用的文件打开模式
[Python常用的文件打开模式](https://blog.csdn.net/weixin_45042494/article/details/110084947)
```
file = open('a.txt', 'r')
print(file.readline())


file = open('a.txt', 'a')
file.write('....')
```
#####文件对象常用方法


[Python File(文件) 方法 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/file-methods.html)

[Python文件基本操作整理 | w3c笔记 (w3cschool.cn)](https://www.w3cschool.cn/article/25704283.html)

#####上下文管理器
[上下文管理器？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/317360115)
#####os模块
[Python OS 文件/目录方法 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/os-file-methods.html)
#####os.path模块
[Python os.path() 模块 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/python-os-path.html)

**列出当前目录下的所有py文件**
```
import  os
dirs = os.getcwd()
lst_dirs = os.listdir(dirs)
for dir in lst_dirs:
    if dir.endswith('.py'):
        print(dir)
```
**流程当前目录下的所有文件**
```
import os
dirs = os.getcwd()
files = os.walk(dirs)
print(type(files))
for file in files:
    print(file[0], file[1], file[2])
```
```
import os
import os.path
dirs = os.getcwd()
files = os.walk(dirs)
print(type(files))
for a, b, c in files:
    for dir in b:
        print(os.path.join(a, dir))
    for file in c:
        print(os.path.join(a, file))
```
