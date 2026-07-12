##print函数
#####将数据输出到文件中
```
# 将数据输出到文件中
# a+ 的含义是如果存在文件就追加
fp = open('F:/project/python/fp/a.txt', 'a+')
print('hello world', file=fp)
fp.close()
```
##### 在字符串中使用变量
```
first_name = 'abc'
last_name = 'love'
full_name = f'{first_name}{last_name}'
print(full_name)
print(f'hello,{full_name.upper()}')
```
##### Scanner
```
name = input()
print(name)
```
##转移字符
1. \t \n   \r   \b \\ 
2. 原字符
```
# 区分\t会补全四个格子,\t空格的作用
print('hello\tworld')

print('helloooo\tworld')

# \n起换行的作用
print('hello\nworld')

# \r起覆盖的作用
print('hellooo\rworld')

# \b退一个格子
print('helloo\bworld')

# \\转义
print('http:\\z')

print('\'a\'')

# 原字符,不希望字符串中的转义字符起作用,就使用原字符,就是在字符之前加上r/R
print(r'hello\rworld')
```
![转义字符](https://upload-images.jianshu.io/upload_images/9049859-9519544625baaa9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##字符串运算符
![字符串运算符](https://upload-images.jianshu.io/upload_images/9049859-a8edaf806fcc75ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更多运算符[Python 字符串 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/python-strings.html)
##关键字
![python关键字](https://upload-images.jianshu.io/upload_images/9049859-17bdef8fcaa4d94b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##注释
Python 中单行注释使用 #，多行注释使用三个单引号（'''）或三个双引号（"""）。如下所示：
```
# 我是单行注释

'''
我是多行注释
我是多行注释
'''

"""
我是多行注释
我是多行注释
"""
```
##数据类型
1. 整数：可以为任意大小、包含负数
2. 浮点数：就是小数
```
a = 1.1
b = 2.2
c = 1.2

from decimal import Decimal
print(Decimal('1.1')+Decimal('2.2'))
print(a+b)
```
3. 字符串：以单引号 '、双引号"、三引号 ''' 或 """括起来的文本
4. 布尔：只有 True、False 两种值
5. 空值：用 None 表示
6. 变量：是可变的
7. 常量：不可变
##数据类型转换
```
a1 = 100
a2 = '100'
a3 = 100.0
print(str(a1) + a2, type(str(a1)))
print(float(a1) + int(a2), type(float(a1)), type(int(a2)))
```
##运算符
#####常用运算符
![常用运算符](https://upload-images.jianshu.io/upload_images/9049859-e2d919b735028491.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####运算优先级
![运算优先级](https://upload-images.jianshu.io/upload_images/9049859-8cd8f637bd1425e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**特殊的:**
```
print(-9 % 4)
print(9 % -4)  # 公式 余数 = 被除数-除数*商 9-(-4)*(-3)
print(9 // -4)  # 公式 (一正一负向下取整)
```
**交换两个数**
```
b1, b2 = 1, 2
b1, b2 = b2, b1
print(b1, b2)
```
**比较对象的标识**
```
c1 = c2 = 2
print(c1 is c2)  # 标识的比较
print(c1 == c2)  # 值的计较

# 对象的比较

d1 = [1, 2, 3, 4]
d2 = [1, 2, 3, 4]
print(d1 == d2)
print(d1 is d2)
```
**位运算原理**

[位运算（&、|、^、~、>>、 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/bit-operation.html)
```
0&0=0  0&1=0  1&0=0  1&1=1
0|0=0  0|1=1  1|0=1  1|1=1
0^0=0  0^1=1  1^0=1  1^1=0
```
##基本语句结构
#####对象的布尔值
python一切皆是对象,所有对象都有一个布尔值,获取对象的布尔值使用内置函数 bool()

以下对象的布尔值为false
1. flase
2. 数值
3. none
4. 空字符串
5. 空列表
6. 空元组
7. 空字典
8. 空集合
#####顺序结构
#####选择结构
#####循环结构
#####if选择结构
```
money = 100
get_money = int(input(' 请输入多少钱'))
if get_money <= 100:
    money = money - get_money
    print(f'取走{get_money}元\n还剩{money}元')
else:
    print('余额不足,请充值')
    put_money = int(input('充值金额为多少钱'))
    money = money + put_money
    print(f'余额还剩{money}')
```
注意运算符的执行顺序
```
money = 10000
get_money = int(input(' 请输入多少钱'))
if get_money <= 100:
    money = money - get_money
    print(f'取走{get_money}元\n还剩{money}元')
elif (get_money > 100) & (get_money <= money):
    print(get_money > 100 & get_money <= money)
    print('一次只能取走低于100元\n退出')
else:
    print('余额不足,请充值')
```
嵌套if
```
member = input('您是会员吗?是输入Y,不是输入N')
money = float(input('请输入您的消费金额'))
print('购汇规则如下\n如果是会员:\n1. 消费金额>=200打8折\n2. 消费金额<=200and消费金额>=100打9折\n3. 消费金额<100打9.5折\n如果不是会员:\n1. 消费金额>=1000打9折\n2. '
      '其他不打折')
if member == 'Y':
    if money >= 200:
        print('打8折,您的购物金额为', str(money * 0.8))
    elif 100 <= money < 200:
        print('打9折,您的购物金额为', money * 0.9)
    else:
        print('打9.5折,您的购物金额为', money * 0.95)
elif member == 'N':
    if money >= 1000:
        print('打9折,您的购物金额为', str(money * 0.9))
    else:
        print('您的购物金额为', money)
else:
    print('退出')
```
条件表达式
```
num1 = int(input('前输入第一个数字'))
num2 = int(input('前输入第一个数字'))
print(str(num1) + ' >= ' + str(num2) if num1 >= num2 else str(num1) + ' < ' + str(num2))
```
#####pass语句占位

```
if 1 > 2:
    pass
else:
    pass
```
##### range() 函数
range(start, stop[, step])
**返回对象是一个迭代器**
默认步长是1,默认从0开始,注意含头不含尾
```
start: 计数从 start 开始。默认是从 0 开始。例如range（5）等价于range（0， 5）;
stop: 计数到 stop 结束，但不包括 stop。例如：range（0， 5） 是[0, 1, 2, 3, 4]没有5
step：步长，默认为1。例如：range（0， 5） 等价于 range(0, 5, 1)
``` 
```
print(list(range(10)))
print(range(1, 10, 2))
print(list((range(1, 10, -1))))
```
#####while循环语句
if是判断一次,条件是True执行一次
while是判断N+1次,条件为True执行N次
python切记空格对结果的影响很大,他没有java中的{}来起隔离的作用.....
每个while循环都必须有停止运行的途径，这样才不会没完没了地执行下去
```
# 注意 a % 2 和 a % 2 == 0 的区别
number = 0
a = 1
while a <= 100:
    if not bool(a % 2):
        # if a % 2
        number = number + a
        print(number)
    a = a + 1
print(number)

print(2 % 2)
print(2 % 2 == 0)
```
#####for in 循环结构
感觉python对齐这个要求很垃圾😡,为什么不隔离....
```
for i in range(1, 10):
    print(i)

for i in 'abcd':
    print(i)

num = 0
for i in range(101):
    if i % 2 == 0:
        num = num + i
        print(num)
    print('1')
    print(num)
print(num)    
```
#####break流程控制
要立即退出while循环，不再运行循环中余下的代码，也不管条件测试的结果如何，可使用break语句。break语句用于控制程序流程，可用来控制哪些代码行将执行、哪些代码行不执行，从而让程序按你的要求执行你要执行的代码。
```
for i in range(3):
    password = input("前输入密码")
    if password == '666':
        print("密码正确")
        break
    else:
        print("密码错误")

a = 0
while a < 2:
    password = input("前输入密码")
    if password == '666':
        print("密码正确")
        break
    else:
        print("密码错误")
        a += 1
```
#####continue流程控制
要返回循环开头，并根据条件测试结果决定是否继续执行循环，可使用continue语句，它不像break语句那样不再执行余下的代码并退出整个循环。
```
a = 1
while a <= 100:
    a += 1
    if a % 2 == 0:
        continue
    else:
        print(a)
```
#####嵌套循环
九九乘法表
```
for i in range(1, 10):
    for j in range(1, i+1):
        print(f'{j}' + ' * ' + f'{i}' + ' = ' + f'{i * j}', end='\t')
    print()
```
#####二重循环中的break和continue
只控制本层循环
```
for i in range(1, 6):
    for j in range(1, 11):
        if j % 2 == 1:
            print(j, end='\t')
        else:
            None
# continue
# break
    print()
```
