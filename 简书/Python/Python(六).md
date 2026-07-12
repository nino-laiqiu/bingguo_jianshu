
参考书籍:[Python爬虫开发：从入门到实战（微课版）-谢乾坤-微信读书 (qq.com)](https://weread.qq.com/web/reader/e30323f0716ac22fe3092b2k8f132430178f14e45fce0f7)

学习爬虫的主要目的是
1. 工作方向:理解流量数据,埋点,UBT治理等
2. 计算机基础方面:理解计算机网络应用,前端知识
3. python方向:巩固基础知识

话说这个课程也太老了吧......凑活学学吧

##爬虫合法性
1. 在法律中是不被禁止的
2. 具有违法风险
3. 爬虫干扰了访问网站的正常运营
4. 爬虫抓取了法律保护的特定数据或信息
5. 在使用,传播爬取到数据时,审查抓取到的内容,如果发现了涉及用户隐私商业机密等敏感内容要及时停止爬取或传播


##爬虫在使用场景上的分类
1. 通用爬虫
抓取系统的重要组成部分,抓取的是一整张页面数据
2. 聚焦爬虫
抓取的是页面中特定的局部内容
3. 增量式爬虫
只会抓取网站中最新更新出来的数据

##爬虫中的矛和盾
#####反爬机制
#####反反爬机制
#####robots.txt协议
规定了网站中哪些数据可以被爬取哪些数据不可以被爬取

https://www.bilibili.com/robots.txt

##http & https协议
http协议:就是服务器和客户端交互的一种形式

常用的请求头:
1. User-Agent :请求载体的身份标识
2. Connection:请求完毕后,是断开连接还是保持连接

常用的响应头:
1. Conent-Type:服务器响应回客户端的数据类型

https:安全的超文本传输协议

加密方式:
1. 对称密钥加密
2. 非对称密钥加密
3. 证书密钥加密 

##requests模块
[Requests: 让 HTTP 服务人类 — Requests 2.18.1 文档 (python-requests.org)](https://docs.python-requests.org/zh_CN/latest/)
[Python Requests库](http://c.biancheng.net/python_spider/requests.html)

#####requests模块的编码流程
1. 指定url
2. 发起请求
3. 获取响应数据
4. 持久化存储

基本使用
```
import requests


url = 'https://www.bilibili.com/'
response = requests.get(url=url)
page_text = response.text
print(page_text)
with open('./bilibili.html', 'w', encoding='utf-8') as fp:
    fp.write(page_text)
print('爬取数据结束')
```
巩固案例
#####网页采集器(UA伪装)
[ Python爬虫 之UA伪装](https://blog.csdn.net/qq_45856289/article/details/108048989)
![网络-F12-JS-User-Agent](https://upload-images.jianshu.io/upload_images/9049859-0406b2c87fea3307.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import requests

url = 'https://www.sogou.com/web'
v = input('查询')
param = {
    'query': v
}

header = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3775.400 QQBrowser/10.6.4208.400'
}
response = requests.get(url=url, params=param, headers=header)
filename = v + '.html'
with open(filename, 'w', encoding='utf-8') as fp:
    fp.write(response.text)
print('持久化完成')
```
#####破解百度翻译
[Microsoft Edge DevTools 文档 - Microsoft Edge Development | Microsoft Docs](https://docs.microsoft.com/zh-cn/microsoft-edge/devtools-guide-chromium/landing/)
[AJAX 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/ajax/ajax-tutorial.html)
```
import requests
import json

url = 'https://fanyi.baidu.com/sug'
v = input('查询')
# 注意一定是kw
data = {
    'kw': v
}

header = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.41 Safari/537.36 Edg/101.0.1210.32'
}
response = requests.post(url=url, data=data, headers=header)
# 要确定返回的对象就是json格式
json_obj = response.json()
print(json_obj)
filename = v + '.json'
with open(filename, 'w', encoding='utf-8') as fp:
    json.dump(json_obj, fp=fp, ensure_ascii=False)

print('持久化完成')
```
#####豆瓣电影TOP100
```
import json

import requests


url = 'https://movie.douban.com/j/chart/top_list'

param = {
    'type': '24',
    'interval_id': '100:90',
    'action': '',
    'start': '0',
    'limit': '100'
}

header = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3775.400 QQBrowser/10.6.4208.400'
}
response = requests.get(url=url, params=param, headers=header)
list_obj = response.json()


filename = 'movie.html'
with open(filename, 'w', encoding='utf-8') as fp:
    json.dump(list_obj, fp=fp, ensure_ascii=False)
print('持久化完成')
```

#####肯德基餐厅信息查询
```
import requests

url = 'http://www.kfc.com.cn/kfccda/ashx/GetStoreList.ashx?op=keyword'

data = {
    'cname': '',
    'pid': '',
    'keyword': '上海',
    'pageIndex': '3',
    'pageSize': '10'
}

header = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.41 Safari/537.36 Edg/101.0.1210.32'
}
response = requests.post(url=url, data=data, headers=header)
# 要确定返回的对象就是json格式
test_obj = response.text
filename = 'kdj.html'
with open(filename, 'a', encoding='utf-8') as fp:
    fp.write(test_obj)
    fp.write('\n')

print('持久化完成')
```
1. [F12进入调试界面总是停留在Paused in debugger解决办法](https://blog.csdn.net/YesterdayLee/article/details/110947374?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-3-110947374.pc_agg_new_rank&utm_term=%E5%B7%B2%E5%9C%A8%E8%B0%83%E8%AF%95%E8%BF%87%E7%A8%8B%E4%B8%AD%E6%9A%82%E5%81%9C&spm=1000.2123.3001.4430)

[药监局化妆品生产许可练习](https://blog.csdn.net/haimian_baba/article/details/103713089?spm=1001.2014.3001.5502)
