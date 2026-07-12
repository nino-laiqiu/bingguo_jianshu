##数据解析
1. 正则
2. bs4
3. xpath

数据解析原理概述:
1. 解析的局部的文本内容都会在标签之间或者标签对应的属性中进行存储
2. 进行指定标签的定位
3. 标签或者标签对应的属性中存储的数据值进行提取

##re模块
#####re模块的使用
```
# findall
# finditer,返回迭代器 group
# search 找到一个结果就返回
# match  从头开始匹配
# 预加载正则表达式 compile
```
#####豆瓣top100电影
```
# 豆瓣电影top250的爬取

import re
import time

import requests
import csv

lst = range(0, 100, 25)

for num in lst:
    print(num)
    time.sleep(1)
    url = f'https://movie.douban.com/top250?start={num}&filter='

    header = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.41 Safari/537.36 Edg/101.0.1210.32'
    }

    response = requests.get(url, headers=header)
    print(response.text)

    obj = re.compile(r'<span class="title">(?P<name>.*?)</span>.*?<br>(?P<year>.*?)&nbsp;.*?<span class="rating_num" '
                     r'property="v:average">(?P<rating>.*?)</span>.*?<span>(?P<people_num>\d*?)人评价</span>', re.S)

    result = obj.finditer(response.text)

    with open('demo2.csv', 'a', encoding='utf-8', newline='') as fp:
        for i in result:
            dit = i.groupdict()
            dit['year'] = i.group('year').strip()
            csv.writer(fp).writerow(dit.values())

```
#####电影天堂电影下载地址
注意点乱码的处理,verify=False,子页面的处理,转义字符的处理
```
import re
import csv
import requests

url = 'https://dytt89.com'
header = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.41 Safari/537.36 Edg/101.0.1210.32'
}
# 设置verify
response = requests.get(url, headers=header, verify=False)
# 设置编码
response.encoding = 'gbk'
lst = []
# print(response.text)
obj1 = re.compile(r'2022必看热片.*?<ul>(?P<child_url>.*?)</ul>', re.S)
# 转义
obj2 = re.compile(r"<a href=\\'(?P<child_url>.*?)' title", re.S)
# 获取子页面的下载地址
obj3 = re.compile(r'<br />◎片　　名　(?P<title>.*?)<br />.*?<td style="WORD-WRAP: break-word" bgcolor="#fdfddf"><a href="('
                  r'?P<download>.*?)&tr', re.S)
result = obj1.findall(response.text)
# print(str(result))
result1 = obj2.finditer(str(result))
for html in result1:
    lst.append(url + html.group('child_url').strip("\\"))

for child_url in lst:
    child_response = requests.get(child_url, headers=header, verify=False)
    child_response.encoding = 'gbk'
    result3 = obj3.finditer(child_response.text)
    with open('demo3.csv', 'a', encoding='utf-8', newline='') as fp:
        for i in result3:
            dit = i.groupdict()
            dit['title'] = i.group('title').strip()
            csv.writer(fp).writerow(dit.values())
```
#####猪八戒网
好像不对,正则爬取有一点不好就是可能不同模块的规则是不一致的,这样爬取就出问题了
```
import re
import requests
import csv

url = 'https://shanghai.zbj.com/search/f/?kw=sass'

header = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.41 Safari/537.36 Edg/101.0.1210.32'
}

response = requests.get(url, headers=header)
obj = re.compile(r"<span class='price'>¥(?P<price>.*?)</span>.*?<p class='title'>(?P<name>.*?)<hl>.*?</hl>.*?<div "
                 r"data-shopname='(?P<shopname>.*?)' class", re.S)
result = obj.finditer(response.text)

with open('demo5.csv', 'a', encoding='utf-8', newline='') as fp:
    for i in result:
        dit = i.groupdict()
        csv.writer(fp).writerow(dit.values())

print('写入完成')
```
##bs4
bs4通过html的标签和属性定位要爬取的数据内容
[Python BS4解析库用法详解](http://c.biancheng.net/python_spider/bs4.html)


##xpath
一种表达xml的语法
[XPath 语法 | 菜鸟教程 (runoob.com)](https://www.runoob.com/xpath/xpath-syntax.html)
