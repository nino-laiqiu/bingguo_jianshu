正则是一个非常强大的文本处理工具，它的应用极其广泛。我们可以利用它来校验数据的有效性，比如用户输入的手机号是不是符合规则；也可以从文本中提取想要的内容，比如从网页中抽取数据；还可以用来做文本内容替换，从而得到我们想要的内容

![正则表达式能做什么](https://upload-images.jianshu.io/upload_images/9049859-bcf8f1615aff1b04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[在线正则表达式测试 (oschina.net)](https://tool.oschina.net/regex)
##元字符
所谓元字符就是指那些在正则表达式中具有特殊意义的专用字符，元字符是构成正则表达式的基本元件.可以把元字符大致分成这几类：表示单个特殊字符的，表示空白符的，表示某个范围的，表示次数的量词，另外还有表示断言的，我们可以把它理解成边界限定

![元字符的分类与记忆技巧](https://upload-images.jianshu.io/upload_images/9049859-420607d0c4440a5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####特殊单字符
1. 英文的点（.）表示换行以外的任意单个字符
2. \d 表示任意单个数字
3. \w 表示任意单个数字或字母或下划线
4. \s 表示任意单个空白符
5. 还有与之对应的三个 \D、\W 和 \S，分别表示着和原来相反的意思

#####空白符
1. \r 回车符
2. \n  换行符
3. \f  换页符
4. \t  制表符
5. \v  垂直制表符
6. \s  任意空白符

#####量词
我们需要匹配单个字符，或者某个部分“重复 N 次”“至少出现一次”“最多出现三次”等等这样的字符

1. 英文的星号（*）代表出现 0 到多次
2. 加号（+）代表 1 到多次
3. 问号（?）代表 0 到 1 次
4. {m,n}代表 m 到 n 次
5. {m}代表 出现m次
6. {m,}代表至少出现m次

![示例理解\d*](https://upload-images.jianshu.io/upload_images/9049859-3d3f6396a084c62b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![示例理解\d{1,4}](https://upload-images.jianshu.io/upload_images/9049859-d4da75a049af5942.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####范围
1. |或,如ab|bc表示ab或bc
2. [...]多选一,括号中任意单个元素
3. [a-z]匹配a到z之间任意单个元素(按ASCII表,包含a,z)
4. [^...]取反,不能是括号中的任意单个元素


##为什么会有贪婪与非贪婪模式？
贪婪模式，简单说就是尽可能进行最长匹配。非贪婪模式呢，则会尽可能进行最短匹配。
```

>>> import re
>>> re.findall(r'a*', 'aaabb')
['aaa', '', '', '']

```
##贪婪、非贪婪与独占模式
#####贪婪匹配（Greedy）

贪婪模式的特点就是尽可能进行最大长度匹配。所以要不要使用贪婪模式是根据需求场景来定的。如果我们想尽可能最短匹配呢？那就要用到非贪婪匹配模式了。

![我们来看一下在字符串 aaabb 中使用正则 a* 的匹配过程](https://upload-images.jianshu.io/upload_images/9049859-53c6983b01e4d2ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####非贪婪匹配（Lazy）
```
>>> import re
>>> re.findall(r'a*', 'aaabb')  # 贪婪模式
['aaa', '', '', '']
>>> re.findall(r'a*?', 'aaabb') # 非贪婪模式
['', 'a', '', 'a', '', 'a', '', '', '']

```

![这两者之间的区别](https://upload-images.jianshu.io/upload_images/9049859-cc17b4702a1bff15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![示例两者的区别](https://upload-images.jianshu.io/upload_images/9049859-adcb6462f4d75525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####独占模式（Possessive）
![贪婪模式](https://upload-images.jianshu.io/upload_images/9049859-d472ac33b9aeaaf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![非贪婪模式](https://upload-images.jianshu.io/upload_images/9049859-957c63ca69972ab4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

独占模式和贪婪模式很像，独占模式会尽可能多地去匹配，如果匹配失败就结束，不会进行回溯，这样的话就比较节省时间。具体的方法就是在量词后面加上加号（+）
![独占模式](https://upload-images.jianshu.io/upload_images/9049859-f457be9a7f14b77d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```

注意：需要先安装 regex 模块，pip install regex

>>> import regex
>>> regex.findall(r'xy{1,3}z', 'xyyz')  # 贪婪模式
['xyyz']
>>> regex.findall(r'xy{1,3}+z', 'xyyz') # 独占模式
['xyyz']
>>> regex.findall(r'xy{1,2}+yz', 'xyyz') # 独占模式
[]
```
##正则回溯引发的血案
[一个正则表达式引发的血案，让线上CPU100%异常！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/38229530)

[New Tool: SDL Regex Fuzzer - Microsoft Security Blog](https://www.microsoft.com/security/blog/2010/10/12/new-tool-sdl-regex-fuzzer/)

练习:
we found “the little cat” is in the hat, we like “the little cat”

其中 the little cat 需要看成一个单词
```
\w+|“[^”]+”
```

一个分支能匹配上就不会看另在一个分支，如果失败了看另外一个分支

##分组与引用
#####不保存子组
![不保留子组](https://upload-images.jianshu.io/upload_images/9049859-9363ab60407d5a31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####括号嵌套
#####命名分组
由于编号得数在第几个位置，后续如果发现正则有问题，改动了括号的个数，还可能导致编号发生变化，因此一些编程语言提供了命名分组（named grouping），这样和数字相比更容易辨识，不容易出错。命名分组的格式为(?P<分组名>正则)。
#####分组引用
在知道了分组引用的编号 （number）后，大部分情况下，我们就可以使用 “反斜扛 + 编号”，即 \number 的方式来进行引用
#####分组引用在查找中使用
![前面出现的单词再次出现](https://upload-images.jianshu.io/upload_images/9049859-07c1e38f61b5f195.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（\w+）代表分组，此时只有一个分组，“\1”代表第一个分组的内容于是，该正则意思是：某单词+空格+某单词，这样就实现了连续重复单词的匹配

#####分组引用在替换中使用
![分组引用在替换中使用](https://upload-images.jianshu.io/upload_images/9049859-23a222921a8e9405.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import re
text = '''the little cat cat is in the hat hat hat , we like it'''
pattern = re.compile(r'''(\w+)(\s+\1)+''')
a = pattern.findall(text)
print(re.sub(pattern, r"\1", text))

# (\s+\1)+ 里面的(\s+\1)重复一次或多次
```

##不区分大小写模式（Case-Insensitive）
不区分大小写是匹配模式的一种。当我们把模式修饰符放在整个正则前面时，就表示整个正则表达式都是不区分大小写的。模式修饰符是通过 (? 模式标识) 的方式来表示的。  我们只需要把模式修饰符放在对应的正则前，就可以使用指定的模式了。
![案例一](https://upload-images.jianshu.io/upload_images/9049859-3848cd34aed2bc13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![案例二](https://upload-images.jianshu.io/upload_images/9049859-2123432b695c3b46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![案例三](https://upload-images.jianshu.io/upload_images/9049859-d47c9ca044f098d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##点号通配模式（Dot All）
英文的点（.）有什么用吗？它可以匹配上任何符号，但不能匹配换行。
![案例一](https://upload-images.jianshu.io/upload_images/9049859-77f9b93b74e626e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##多行匹配模式（Multiline）
通常情况下，^ 匹配整个字符串的开头，$ 匹配整个字符串的结尾。多行匹配模式改变的就是 ^ 和 $ 的匹配行为

![案例一](https://upload-images.jianshu.io/upload_images/9049859-9bcf09272df58e95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

多行模式的作用在于，使  ^ 和 $ 能匹配上每行的开头或结尾，我们可以使用模式修饰符号 (?m) 来指定这个模式。
![案例二](https://upload-images.jianshu.io/upload_images/9049859-2b846c8a8f7c996b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##注释模式（Comment）
```

(\w+)(?#word) \1(?#word repeat again)
```

HTML 标签是不区分大小写的，比如我们要提取网页中的 head 标签中的内容，用正则如何实现呢？
[正则前面的 (?i) (?s) (?m) (?is) (?im)](https://blog.csdn.net/codepen/article/details/40396769)

# 单行模式
```
[-+](?si)<head>(.*)<\/head>
```


