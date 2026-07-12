[Python发送邮件](https://www.jianshu.com/p/abb2d6e91c1f)

##邮件传输协议(SMTP)
[如何验证 Email 地址：SMTP 协议入门教程 ](https://www.ruanyifeng.com/blog/2017/06/smtp-protocol.html#:~:text=SMTP%20%E6%98%AF%22%E7%AE%80%E5%8D%95%E9%82%AE%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE%22%EF%BC%88Simple%20Mail%20Transfer%20Protocol%EF%BC%89%E7%9A%84%E7%BC%A9%E5%86%99%EF%BC%8C%E5%9F%BA%E4%BA%8E%20TCP%20%E5%8D%8F%E8%AE%AE%EF%BC%8C%E7%94%A8%E6%9D%A5%E5%8F%91%E9%80%81%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E3%80%82%20%E5%8F%AA%E8%A6%81%E8%BF%90%E8%A1%8C%E4%BA%86%E8%AF%A5%E5%8D%8F%E8%AE%AE%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%EF%BC%88daemon%EF%BC%89%EF%BC%8C%E5%BD%93%E5%89%8D%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%B0%B1%E5%8F%98%E4%B8%BA%E9%82%AE%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E5%8F%AF%E4%BB%A5%E6%8E%A5%E6%94%B6%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E3%80%82,%E5%A6%82%E6%9E%9C%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%BF%94%E5%9B%9E%20250%20%E6%88%96%20251%20%E7%8A%B6%E6%80%81%E7%A0%81%EF%BC%8C%E9%82%AE%E7%AE%B1%E5%B0%B1%E6%98%AF%E7%9C%9F%E7%9A%84%EF%BC%9B%E5%A6%82%E6%9E%9C%E8%BF%94%E5%9B%9E%205xx%EF%BC%88500%EF%BD%9E599%EF%BC%89%EF%BC%8C%E5%B0%B1%E6%98%AF%E5%81%87%E7%9A%84%E3%80%82%20%E6%B3%A8%E6%84%8F%EF%BC%8C%E5%8D%B3%E4%BD%BF%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%A1%AE%E8%AE%A4%E9%82%AE%E7%AE%B1%E6%98%AF%E7%9C%9F%E7%9A%84%EF%BC%8C%20%E4%B9%9F%E4%B8%8D%E4%BB%A3%E8%A1%A8%E9%82%AE%E4%BB%B6%E4%B8%80%E5%AE%9A%E4%BC%9A%E5%8F%91%E9%80%81%E5%88%B0%E8%AF%A5%E9%82%AE%E7%AE%B1%EF%BC%8C%E6%9B%B4%E4%B8%8D%E4%BB%A3%E8%A1%A8%E7%94%A8%E6%88%B7%E4%B8%80%E5%AE%9A%E4%BC%9A%E8%AF%BB%E5%88%B0%E8%AF%A5%E9%82%AE%E4%BB%B6%E3%80%82)

##SSL
为Netscape所研发，用以保障在Internet上数据传输之安全，利用数据加密(Encryption)技术，可确保数据在网络上之传输过程中不会被截取及窃听。

SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。

##以QQ邮箱为例子:开启smtp协议 和 一些设置
![开启smtp协议](https://upload-images.jianshu.io/upload_images/9049859-b2064a6fe748e44c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![一些设置](https://upload-images.jianshu.io/upload_images/9049859-17025ad21ce88f7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. [ [汉字编码报错] UnicodeEncodeError- 'ascii' codec can't encode characters in position 0-1- ordinal not in ](https://blog.csdn.net/sinsa110/article/details/78669767)
2. [python 出现socket.gaierror: [Errno 11004] getaddrinfo failed错误](https://blog.csdn.net/liubo37/article/details/92796535)
```
import smtplib

mail = ''
authorization_code = ''
send_mail = ''
# 服务的设置
server = smtplib.SMTP_SSL('smtp.qq.com', 465)
# 登陆
server.login(mail, authorization_code)

# 发送信息
# 如果是汉字则
news = 'this is a news'

# 邮件的发送
server.sendmail(mail, send_mail, news)
# 退出
server.quit()
```
包装一下
```
import smtplib
from email.mime.text import MIMEText
mail = ''
authorization_code = ''
send_mail = ''
server = smtplib.SMTP_SSL('smtp.qq.com', 465)
# 登陆
server.login(mail, authorization_code)

# 发送信息
# 如果是汉字则
news = 'this is a news'

# 邮件的发送
server.sendmail(mail, send_mail, str(MIMEText(news)))
# 退出
server.quit()
```

设置发送者和接收者
```
import smtplib
from email.mime.text import MIMEText
mail = ''
authorization_code = ''
send_mail = ''
server = smtplib.SMTP_SSL('smtp.qq.com', 465)
# 登陆
server.login(mail, authorization_code)

# 发送信息
# 如果是汉字则
news = 'this is a news'
massage = MIMEText(news, 'plain', 'utf8')

# 设置主题
title = 'SMTP TEST'

# 设置邮件发送者和邮件接收者
massage['FROM'] = mail
massage['TO'] = send_mail

# 把主题加入到massage
massage['Subject'] = title


# 邮件的发送
server.sendmail(mail, send_mail, str(massage))
# 退出
server.quit()
```
##附件
1. [Python_报错：SyntaxError: (unicode error) 'unicodeescape' codec can't decode bytes in position 2-3: truncated \UXXXXXXXX escape](https://www.cnblogs.com/rychh/p/9743864.html)
2. pdf打不开的问题
3. ppt/word乱码的问题
```
import smtplib
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart

mail = ''
authorization_code = ''
send_mail = ''


# 发送信息
news = 'this is a news'
massage = MIMEText(news, 'plain', 'utf8')

# 设置主题
title = 'SMTP TEST'


# 附件的部分
# content-disposition:保证图片是可以显示的
# attachment:保证已发送的附件是可以预览和下载的
image_path = 'preview.jpg'
image_part = MIMEImage(open(image_path, 'rb').read(), image_path.split('.')[-1])
# filename可以自定义
image_part.add_header('content-disposition', 'attachment', filename=image_path)

# 邮件的内容和附件进行拼接
m = MIMEMultipart()
m.attach(image_part)
m.attach(massage)


server = smtplib.SMTP_SSL('smtp.qq.com', 465)
# 登陆
server.login(mail, authorization_code)


# 设置邮件发送者和邮件接收者
m['FROM'] = mail
m['TO'] = send_mail

# 把主题加入到massage
m['Subject'] = title


# 邮件的发送
server.sendmail(mail, send_mail, str(m))
# 退出
server.quit()

```
```
import smtplib
import os.path
from email import encoders
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.application import MIMEApplication
mail = ''
authorization_code = ''
send_mail = ''


# 发送信息
news = 'this is a news'
massage = MIMEText(news, 'plain', 'utf8')

# 设置主题
title = 'SMTP TEST'


# 附件的部分
# content-disposition:保证图片是可以显示的
# attachment:保证已发送的附件是可以预览和下载的
image_path = 'preview.jpg'
image_part = MIMEImage(open(image_path, 'rb').read(), image_path.split('.')[-1])
# filename可以自定义
image_part.add_header('content-disposition', 'attachment', filename=image_path)

# pdf的部分
pdf_path = r'E:\工具箱\身份资料\数据工程师.pdf'
pdf_part = MIMEApplication(open(image_path, 'rb').read(), image_path.split('.')[-1])
# filename可以自定义
pdf_part.add_header('content-disposition', 'attachment', filename='test.pdf')

# ppt
ppt_path = 'kylin基础.pptx'
ppt_part = MIMEApplication(open(image_path, 'rb').read(), image_path.split('.')[-1])
# filename可以自定义
ppt_part.add_header('content-disposition', 'attachment', filename='test.pptx')


# 邮件的内容和附件进行拼接
m = MIMEMultipart()
m.attach(image_part)
m.attach(massage)
# 添加多张图片则添加连接

m.attach(pdf_part)
m.attach(ppt_part)

server = smtplib.SMTP_SSL('smtp.qq.com', 465)
# 登陆
server.login(mail, authorization_code)


# 设置邮件发送者和邮件接收者
m['FROM'] = mail
m['TO'] = send_mail

m['Accept-Language'] = 'zh-CN'
m['Accept-Charset'] = 'ISO-8859-1,utf-8'


# 把主题加入到massage
m['Subject'] = title


# 邮件的发送
server.sendmail(mail, send_mail, str(m))
# 退出
server.quit()

```

??乱码谁帮忙解决一下,PDF打不开??谁帮忙解决一下,网上查了好久全是......
