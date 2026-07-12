##列表是什么
列表由一系列按特定顺序排列的元素组成。你可以创建包含字母表中所有字母、数字0～9或所有家庭成员姓名的列表；也可以将任何东西加入列表中，其中的元素之间可以没有任何关系。列表通常包含多个元素.在Python中。用方括号（[]）表示列表，并用逗号分隔其中的元素。
#####列表的特点
1. 列表元素按顺序有序排序
2. 索引映射唯一数据
3. 列表可以存储重复的数据
4. 任意数据类型混存
5. 根据需要动态分配和回收内存
#####访问列表元素
注意步长为负数的情况,和可省略的情况
```
lst = ['小米', 10, 1.00, True]
print(lst[0].title())
print(lst[-1])
print(lst[0: 2])
print(lst[0: 3: 2])
print(lst[lst.index('小米')])
print(lst[::-1])
print(lst[3::-1])

```
**Python index()方法**
[Python index()方法 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/att-string-index.html)
判断指定元素是否在列表中及遍历
```
lst = ['小米', 10, 1.00, True]
print(10 in lst)
print('11' in lst)
for i in lst:
    print(i, end='\t')
```
#####索引从0而不是1开始
#####修改列表元素
理解省略语法
```
lst = list([100, 200, 300, 400, 500])
lst[0] = '100'
print(lst)
lst[1:1:1] = [400, 300, 200]
print(lst)

lst1 = list([100, 200, 300, 400, 500])
lst1[1::1] = [400, 300, 200]
print(lst1)

lst2 = list([100, 200, 300, 400, 500])
lst2[1:3:1] = [400, 300, 200]
print(lst2)
```
#####在列表中添加元素

```
lst = list([100, 200, 300, 400, 500])
print(id(lst))
lst.append(600)
# 地址值不变
print(lst, id(lst))
```

注意append和extend的区别
```
lst1 = list([100, 200, 300, 400, 500])
lst2 = [600, 700]
lst1.append(lst2)
print(lst1)


lst1 = list([100, 200, 300, 400, 500])
lst2 = [600, 700]
lst1.extend(lst2)
print(lst1)
```
切片
```
lst3 = list([100, 200, 300, 400, 500])
lst3[1::] = ['iiii', '']
print(lst3)
```
指定位置插入
```
lst4 = list([100, 200, 300, 400, 500])
lst4.insert(0, '100')
print(lst4)
```
#####从列表中删除元素
1. remove
一次删除一个元素,重复元素只删除第一个,元素不存在抛出异常
2. pop
删除一个指定索引位置上的元素,不指定索引,删除列表中最后一个元素
3. 切片
至少删除一个元素
4. clear
5. del
```
# remove
lst = list([100, 200, 300, 400, 500])
lst.remove(100)
print(lst)

# pop
lst1 = list([100, 200, 300, 400, 500])
lst1.pop()
print(lst1)
lst1.pop(0)
print(lst1)

#
lst2 = list([100, 200, 300, 400, 500])
new_lst2 = lst2[1:3]
lst2[1:3] = []
print(new_lst2)
print(lst2)

# clear
lst3 = list([100, 200, 300, 400, 500])
lst3.clear()
print(lst3)

lst4 = list([100, 200, 300, 400, 500])
del lst4
# print(lst4)
lst5 = list([100, 200, 300, 400, 500])
del lst5[0]
print(lst5)
```
#####组织列表
1. 使用方法sort()对列表永久排序
sort()方法传递参数reverse=True,降序排序
2. 使用函数sorted()对列表临时排序
3. 倒着打印列表
reverse()不是按与字母顺序相反的顺序排列列表元素，而只是反转列表元素的排列顺序
4. 确定列表的长度
使用函数len()可快速获悉列表的长度
#####生成列表式
```
lst3 = [i * i for i in range(1, 10)]
print(lst3)
```
#####复制列表
```
lst4 = [1, 2, 3]
copy_lst4 = lst4[:]
print(copy_lst4)
```
#####深入了解循环
其实就是缩进问题......

[操作列表:从入门到实践（第2版）-埃里克·马瑟斯-微信读书 (qq.com)](https://weread.qq.com/web/reader/08232ac0720befa90825d88kc51323901dc51ce410c121b)
#####小结
[Python 列表(List) | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/python-lists.html)
![列表知识点小结](https://upload-images.jianshu.io/upload_images/9049859-75921b82ebe8467e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##字典是什么
在Python中，字典是一系列键值对。每个键都与一个值相关联，你可使用键来访问相关联的值。与键相关联的值可以是数、字符串、列表乃至字典。事实上，可将任何Python对象用作字典中的值。在Python中，字典用放在花括号（{}）中的一系列键值对表示
#####字典的创建
```
tinydict = {'小米': 10001, '小红': 10002}
print(tinydict)

tinydict1 = {}
print(tinydict1)

tinydict2 = dict(name='小米', id=19)
print(tinydict2)
```
#####访问字典中的值
注意使用[]这种方式访问,如果不存在就会报错
get方式不会报错返回空值,可以给一个默认值的
```
dct = {'小米': 10001, '小红': 10002}
print(dct['小米'])
print(dct.get('小米'))
print(dct.get('', 'null'))
```
#####添加键值对&修改字典中的值&删除键值对
```
dct1 = {'小米': 10001, '小红': 10002}

dct1['小明'] = 10003
dct1['小米'] = 10000
del dct1['小红']
print(dct1)
```
#####遍历所有键值对&遍历字典中的所有键&遍历字典中的所有值
注意遍历的时候要使用items()函数
```
dct2 = {'小米': 10001, '小红': 10002}

for key in dct2:
    print(key, dct2[key], dct2.get(key))

for k, v in dct2.items():
    print(k, v)

print(dct2.items())
print(type(dct2.items()))
print(list(dct2.items()))

print(dct2.keys())
print(type(dct2.keys()))


print(dct2.values())
print(list(dct2.values()))


print(list(dct2.items()))
```
这是啥???
```
dct3 = {'小米': 10001, '小红': 10002}


for k, v in dct3:
    print(k, v)
```
#####字典生成式
zip函数,如果两个列表不一致,则取短的
```
id = [1, 2, 3]
name = ['a', 'b', 'c']
dct4 = {k: v for k, v in zip(id, name)}
print(dct4)
```
#####小结
![字典的特点](https://upload-images.jianshu.io/upload_images/9049859-33cf6cff89556b0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![字典小结](https://upload-images.jianshu.io/upload_images/9049859-35710a20680847af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##元组是什么
元组看起来很像列表，但使用圆括号而非中括号来标识。定义元组后，就可使用索引来访问其元素，就像访问列表元素一样。试图修改元组的操作是被禁止的，因此Python指出不能给元组的元素赋值

元组是由逗号标识的，圆括号只是让元组看起来更整洁、更清晰。如果你要定义只包含一个元素的元组，必须在这个元素后面加上逗号

#####修改元组
元组中的元素值是不允许修改的，但我们可以对元组进行连接组合
#####删除元组
元组中的元素值是不允许删除的，但我们可以使用del语句来删除整个元组
#####元组运算符
![元组运算符](https://upload-images.jianshu.io/upload_images/9049859-76df2e53b1265bd1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####为什么要将元组设计成不可变序列
![为什么要将元组设计成不可变序列](https://upload-images.jianshu.io/upload_images/9049859-a661a916ebe7ec14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![为什么要将元组设计成不可变序列](https://upload-images.jianshu.io/upload_images/9049859-0dde79a1892cc395.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####元组的遍历
```
t = (1, 2, '1')
t1 = tuple((1, 2, '1'))
print(t[0])
print(t1)
for i in t:
    print(i, end='\t')
```
##集合是什么
1. 集合是没有value的字典
2. 与列表,字典一样都是属于可变类型的序列
3. 集合中的元素不能重复
#####创建方式
```
s = {1, 2, 3, 4}
print(s)
s1 = set({1, 2, 3, 4})
print(s1)
s2 = set('python')
s3 = set([1, 2, 3, 4])
s4 = set()  # 不能使用s = {}
s5 = set(('python', 'java'))
print(s2, type(s2))
print(s3, type(s3))
print(s4, type(s4))
print(s5, type(s5))
```
#####集合相关操作
1. 添加元素
add
update 至少添加一个元素
2. 移除元素
remove,不存在则会抛出异常
discard,不存在则不会抛出异常
pop,随机删除,不能指定元素
clear
3. 集合内置方法完整列表
![集合内置方法完整列表](https://upload-images.jianshu.io/upload_images/9049859-5ba61f40d6690ff4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
[Python3 集合 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python3/python3-set.html)
#####集合间的关系
#####集合的数据操作
```
a = {1, 2, 3, 4, 5}
b = {2, 3, 4, 5, 6}
# 交集
print(a.intersection(b))
print(a & b)
# 并集
print(a.union(b))
print(a | b)
# 差集
print(a.difference(b))
print(a - b)
print(b.difference(a))
# 对称差集
print(a.symmetric_difference(b))
print((a-b) | (b-a))
```
#####集合生成式
```
s = {i*i for i in range(11)}
print(s)
```
##列表,字典,元组,集合小结
![列表,字典,元组,集合小结](https://upload-images.jianshu.io/upload_images/9049859-47314a301baa4cc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![列表,字典,元组,集合小结](https://upload-images.jianshu.io/upload_images/9049859-6041a376832b9ceb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


