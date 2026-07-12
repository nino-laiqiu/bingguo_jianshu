##字符串
#####创建和驻留机制
在python中字符串是基本数据类型,是一个不可变的字符序列

什么叫字符串驻留机制:仅保存一份相同且不可变字符串的方法,不同的值被存放在字符串的驻留池中,python的驻留机制对相同的字符串只保留一份拷贝,后续创建相同字符串时,不会开辟新空间,而是把该字符串的地址赋给新创建的变量

**驻留机制的几种情况(交互模式)**
1. 字符串为0/1时
2. 符合标识符的字符串
3. 字符串只在编译时进行驻留,而非运行时
4. [-5,256]之间的整数

![驻留机制的几种情况](https://upload-images.jianshu.io/upload_images/9049859-472af4488c0188d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

sys中的intern方法强制2个字符串指向同一个对象

**字符串驻留机制的优缺点**
1. 当需要值相同的字符串,可以直接从字符串池里拿来使用,避免频繁的创建和销毁,提升效率和节约内存,因此拼接字符串和修改字符串是比较影响性能的
2. 在需要进行字符串拼接建议使用str类型的join方法,而不是+,因为join方法是先计算出所有字符串中的长度,然后再拷贝,只new一次对象
#####查询
1. index
查找子串第一次出现的位置,如果不存在就会报错
2. rindex
查找子串最后一次出现的位置,如果不存在就会报错
3. find
查找子串第一次出现的位置,如果不存在不会报错
4. rfind
查找子串最后一次出现的位置,如果不存在不会报错
#####大小转换
1. upper
2. lower
3.  swapcase
把字符串中所有大写字母都转成小写字母,把所有小写字母都转成大写字母
```
s = 'python,java'
print(s.capitalize())
```
4. capitalize
把第一个字符转成大写,把其余字符转成小写
5. title
把每个单词的第一个字符转换成大写,把每个单词的剩余字符转换成小写
#####内容对齐操作
1. center
2. ljust
3. rjust
4. zfill:右对齐,使用0来填充

[字符串的对齐](https://blog.csdn.net/Jiang2624/article/details/104165625)
#####劈分
1. split
2. rsplit:从右边切分

sep：用于指定分隔符，可以包含多个字符。此参数默认为 None，表示所有空字符，包括空格、换行符“\n”、制表符“\t”等
maxsplit：可选参数，用于指定分割的次数，最后列表中子串的个数最多为 maxsplit+1。如果不指定或者指定为 -1，则表示分割次数没有限制
```
s4 = 'java scala go'
print(s4.split())
s5 = 'java|scala|go'
print(s5.split(sep='|'))
s6 = 'java|scala|go'
print(s6.split(sep='|', maxsplit=1))
print(s6.rsplit(sep='|', maxsplit=1))
```
#####判断
**python合法标识符的命名规范是：**
1、标识符由英文字母、数字和下划线组成，但第一个字符不能是数字；
2、标识符不能和python中的保留字相同；
3、标识符中不能包含空格、@等特殊字符。

**判断字符串操作的方法**
1. isidentifier
判断指定的字符串是不是合法的标识符
2. isspace
判断指定的字符串是否全部由空白字符组成
3. isalpha
判断指定的字符串是否全部由字母组成
4. isdecimal
判断指定字符串是否全部由十进制的数字组成
5. isnumeric
判断指定的字符串是否全部由数字组成
6. isalnum
判断指定字符串是否全部由字母和数字组成
#####替换与合并
1. replace
第一个参数指定被替换的子串,第二个参数指定替换子串的字符串,该方法替换前的字符串不发生变化,调用该方法可以通过第三个参数指定最大的替换次数
2. join
该方法将列表或元组的字符串合并成一个字符串
```
s5 = 'java scala go go'
print(s5.replace('go', 'python'))
print(s5.replace('go', 'python', 2))

s6 = ('java', 'scala')
print('|'.join(s6))

s6 = ['java', 'scala']
print('|'.join(s6))

print("*".join('scala'))
```
#####比较
比较原理:两两字符比较,比较的是原始值,调用ord可以,得到指定字符的原始值.与内置函数ord对应的是内置函数chr,调用内置函数chr时可以得到其对应的字符
```
print('a' >= 'b')
print(ord('a'), ord('b'))
print(chr(ord('a')), chr(ord('b')))
```
**== %  is 的区别**
==比较的是值,is比较的是id地址值
#####切片
熟练使用步长
```
s = 'hello scala'
print(s[:5])  # hello
print(s[6:])  # scala
print(s[::2])  # hlosaa
print(s[-5::1])  # scala
print(s[::-1])  # alacs olleh
```
#####格式化字符串
[Python 占位符格式化详解](https://www.pythontab.com/html/2017/pythonjichu_1122/1186.html#:~:text=Python%20%E5%8D%A0%E4%BD%8D%E7%AC%A6%E6%A0%BC%E5%BC%8F%E5%8C%96%E8%AF%A6%E8%A7%A3.%20%E5%8D%A0%E4%BD%8D%E7%AC%A6%EF%BC%8C%E9%A1%BE%E5%90%8D%E6%80%9D%E4%B9%89%E5%B0%B1%E6%98%AF%E6%8F%92%E5%9C%A8%E8%BE%93%E5%87%BA%E9%87%8C%E7%AB%99%E4%BD%8D%E7%9A%84%E7%AC%A6%E5%8F%B7%E3%80%82.%20%E5%8D%A0%E4%BD%8D%E7%AC%A6%E6%98%AF%E7%BB%9D%E5%A4%A7%E9%83%A8%E5%88%86%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80%E9%83%BD%E5%AD%98%E5%9C%A8%E7%9A%84%E8%AF%AD%E6%B3%95%EF%BC%8C%20%E8%80%8C%E4%B8%94%E5%A4%A7%E9%83%A8%E5%88%86%E9%83%BD%E6%98%AF%E7%9B%B8%E9%80%9A%E7%9A%84%EF%BC%8C%20%E5%AE%83%E6%98%AF%E4%B8%80%E7%A7%8D%E9%9D%9E%E5%B8%B8%E5%B8%B8%E7%94%A8%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%A0%BC%E5%BC%8F%E5%8C%96%E7%9A%84%E6%96%B9%E5%BC%8F%E3%80%82.%201.%20%E5%B8%B8%E7%94%A8%E5%8D%A0%E4%BD%8D%E7%AC%A6%E7%9A%84%E5%90%AB%E4%B9%89.,i%20%3C%3D%201114111%EF%BC%88py27%E5%88%99%E5%8F%AA%E6%94%AF%E6%8C%810-255%EF%BC%89%EF%BC%9B%E5%AD%97%E7%AC%A6%EF%BC%9A%E5%B0%86%E5%AD%97%E7%AC%A6%E6%B7%BB%E5%8A%A0%E5%88%B0%E6%8C%87%E5%AE%9A%E4%BD%8D%E7%BD%AE.%20o%20%3A%20%E5%B0%86%E6%95%B4%E6%95%B0%E8%BD%AC%E6%8D%A2%E6%88%90%20%E5%85%AB%20%E8%BF%9B%E5%88%B6%E8%A1%A8%E7%A4%BA%EF%BC%8C%E5%B9%B6%E5%B0%86%E5%85%B6%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%88%B0%E6%8C%87%E5%AE%9A%E4%BD%8D%E7%BD%AE.)

**占位符**
```
name = '小米'
age = 24
print('我叫%s 我今年%d岁' % (name, age))
print('我叫{0} 我今年{1}岁'.format(name, age))
print(f'我叫{name} 我今年{age}岁')
```
**宽度和精度**
```
print('%10d' % 100)
print('%f' % 3.11111)
print('%.3f' % 3.11111)
print('%10.3f' % 3.11111)
print('{0:.3}'.format(3.1111))
print('{0:.3f}'.format(3.1111))
```
#####编码和解码
```
s = '我是小米'
print(s.encode(encoding='GBK'))  # 一个中文两个字符
print(s.encode(encoding='UTF-8'))  # 一个中文三个字符


print(s.encode(encoding='GBK').decode(encoding='GBK'))
print(s.encode(encoding='UTF-8').decode(encoding='UTF-8'))
```
#####小结
![字符串小结](https://upload-images.jianshu.io/upload_images/9049859-74b3067542a17c47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##函数
#####定义和调用
什么是函数:
函数就是执行特定任何以完成特定功能的一段代码

为什么需要函数:
1. 复用代码
2. 隐藏实现细节
3. 提高可维护性
4. 提高可读性

函数的创建

#####参数传递
1. 形参:在函数的定义处
2. 实参:函数的调用处
```
def num_sum(a, b):
    return a + b


print(num_sum(1, 10))
# 关键字传参
print(num_sum(b=10, a=1))
```
#####参数传递的内存分析
1. 如果是不可变对象,在函数体的修改不会影响实参的值
2. 如果是可变对象,在函数体的修改会影响实参的值
```
def f(num1, num2):
    print(num1)
    print(num2)
    num1 = 100
    num2.append('222')
    print(num1)
    print(num2)


s1 = 1
s2 = ['111']
f(s1, s2)
print(s1)
print(s2)
```
#####返回值
1. 如果函数没有返回值,return可以不写
2. 函数的返回值,如果是1个,直接返回类型
3. 函数的返回值,如果是多个,返回的结果为元组
4. return写不写视情况而定
```
def ff(num2):
    s = []
    s1 = []
    for i in num2:
        if i % 2 == 0:
            s.append(i)
        else:
            s1.append(i)
    return s, s1


num2 = [1, 2, 3, 4, 5]
re = ff(num2)
print(re)
```
#####参数定义_默认值参数
```
def f(num1, num2=10):
   return num1 + num2


print(f(10))
print(f(10, 20))
```
#####参数定义_个数可变的位置形参和关键字形参
```
def f2(num, *args):
    print(num)
    print(args, type(args))  # 元组


f2(11, 12, 12, 34)


def f3(**kwargs):
    print(kwargs, type(kwargs))


f3(name='xxx', id=123)  # 字典 
```
#####参数小结
![参数使用小结](https://upload-images.jianshu.io/upload_images/9049859-669a7969015436ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**注意如下的写法哈**
```
def f3(a, b, c):
    print(a)
    print(b)
    print(c)


lst = [1, 2, 3]
f3(*lst)

dic = {'a': 1, 'b': 123, 'c': 123}  # 必须和形参一致
f3(**dic)
```
#####变量的作用域
1. 局部变量 
2. 全局变量
#####递归函数
如果在一个函数体内调用了该函数本身,这个函数就称为递归函数
```
def f4(n):
    if n == 1:
        return 1
    else:
        return n * f4(n-1)


print(f4(10))
```
斐波那契数列
```
def f5(n):
    if n == 1:
        return 1
    elif n == 2:
        return 1
    else:
        return f5(n-1) + f5(n-2)


for i in range(1, 7):
    print(f5(i))
```

##异常
#####粗心导致的错误
1. 漏写了末尾的冒号
2. 缩进错误
3. 把中文字符当成英文字符
4. 字符串拼接中,把字符串和数据进行拼接
5. 没有定义在结构中的变量
6. 比较运算符的混用
#####错误点不熟悉导致的错误
#####思路不清晰导致的错误
#####被动掉坑 try except
```
try:
    num1 = int(input('请数据数字'))
    num2 = int(input('请数据数字'))
    print(num1 / num2)
except ZeroDivisionError as a:
    print("被除数应该不为0", a)
except ValueError as b:
    print("只能输入数字", b)
else:
    print('正确执行')    
finally:
    print('报异常也会执行')
```
#####try except & try except  else 
#####try except  else finally
#####python中的常见异常
#####traceback模块的使用
