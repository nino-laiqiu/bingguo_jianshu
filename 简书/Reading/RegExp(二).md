##断言
#####单词边界（Word Boundary）
我们就可以在正则中使用\b 来表示单词的边界。 \b 中的 b 可以理解为是边界（Boundary）这个单词的首字母
```

>>> import re
>>> test_str = "tom asked me if I would go fishing with him tomorrow."
>>> re.sub(r'\btom\b', 'jerry', test_str)
'jerry asked me if I would go fishing with him tomorrow.'
```
#####行的开始或结束
1. 日志起始行判断
2. 输入数据校验

#####环视（ Look Around）
![规则](https://upload-images.jianshu.io/upload_images/9049859-871e1f99bd92e403.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![示例一](https://upload-images.jianshu.io/upload_images/9049859-1fd23af12c227179.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
the little cat cat2 is in the hat hat2, we like it

\b(\w+)\s\1\b
```
##转义
#####转义字符
#####字符串转义和正则转义
```

>>> import re
>>> re.findall(r'\\', 'a*b+c?\\d123d\\')
['\\', '\\']

```
#####正则中元字符的转义
#####括号的转义
#####字符组中的转义
https://stackoverflow.com/questions/6967204/how-is-n-and-n-interpreted-by-the-expanded-regular-expression/59192811#59192811
##正则有哪些常见的流派及其特性
##正则如何处理 Unicode 编码的文本
##如何在编辑器中使用正则完成工作
##如何理解正则的匹配原理以及优化原则
#####有穷状态自动机
有穷状态是指一个系统具有有穷个状态，不同的状态代表不同的意义。自动机是指系统可以根据相应的条件，在不同的状态下进行转移。从一个初始状态，根据对应的操作（比如录入的字符集）执行状态转移，最终达到终止状态（可能有一到多个终止状态）。有穷自动机的具体实现称为正则引擎，主要有 DFA 和 NFA 两种，其中 NFA 又分为传统的 NFA 和 POSIX NFA。

```
DFA：确定性有穷自动机（Deterministic finite automaton）
NFA：非确定性有穷自动机（Non-deterministic finite automaton）
```
#####正则的匹配过程
#####POSIX NFA 与 传统 NFA 区别
#####回溯
##正则常见问题及解决方案
#####匹配数字
1. 数字在正则中可以使用 \d 或 [0-9] 来表示。
2. 如果是连续的多个数字，可以使用 \d+ 或 [0-9]+。
3. 如果 n 位数据，可以使用 \d{n}。
4. 如果是至少 n 位数据，可以使用 \d{n,}。
5. 如果是 m-n 位数字，可以使用 \d{m,n}。
#####小数
```
\d+(?:\.\d+)?
```
#####浮点数

负数浮点数表示：
```
-\d+(?:\.\d+)?。
```
正数浮点数表示：
```
\+?(?:\d+(?:\.\d+)?|\.\d+)
```
#####十六进制数
```
[0-9A-Fa-f]+
```
#####手机号码
[最新手机号段归属地数据库 (2022年5月版)](https://www.cnblogs.com/zengxiangzhan/p/phone.html)
#####身份证号码
```
[1-9]\d{14}(\d\d[0-9Xx])?
```
#####邮政编码
```
(?<!\d)\d{6}(?!\d)
```
####IPv4 地址
```
\d{1,3}(\.\d{1,3}){3}
```
#####日期和时间
```
\d{4}-\d{2}-\d{2}

\d{4}-(?:1[0-2]|0?[1-9])-(?:[12]\d|3[01]|0?[1-9])

(?:2[0-3]|1\d|0?\d):(?:[1-5]\d|0?\d)
```
#####邮箱
#####网页标签

收集一下常见的正则
