1.与HTML结合的方式
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d1c6988d97bffd5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7c7735743a16d55f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.数据类型
原始数据类型
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ff336a21e754637f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
变量概念
![image.png](https://upload-images.jianshu.io/upload_images/9049859-40653e5c32f774a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-cedb25987c2faafa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0421740d13acdf08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![概述](https://upload-images.jianshu.io/upload_images/9049859-ba773e40946626ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###运算符
一元运算符
![image.png](https://upload-images.jianshu.io/upload_images/9049859-13b3a8dafec21b78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
比较运算符
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e1e1b338ea6dcb95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意不同类型的比较......)
逻辑运算符(注意类型的转换)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-cd3913510393f5cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-36b4b4c4eca24db5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(注意上面的写法)
三目运算符
![image.png](https://upload-images.jianshu.io/upload_images/9049859-4aec8370c1b63c50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
两个注意事项(不推荐使用)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-78ec2d045c899fcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
流程控制语句
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2a6d1a6b7506287a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一个例题(j九九乘法表)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-f75a5c8796aeac21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
完善一下
```
document.write("<table  align='center'>");
        for (var i = 1; i <= 9; i++) {
            document.write("<tr>");
            for (var j = i; j <= 9; j++) {
                document.write("<td>");
                document.write(i + " * " + j + " = " + (i * j) + "&nbsp;&nbsp;&nbsp;");
                document.write("</td>");
            }
            document.write("</tr>")
        }
        document.write("</table>");
        document.write("<table  align='center'>");
        //方法二
        for (let i1 = 1; i1 <= 9; i1++) {
            document.write("<tr>");
            for (let j1 = 1; j1 <= i1; j1++) {
                document.write("<td>");
                document.write(i1 + " * " + j1 + " = " + (i1 * j1) + "&nbsp;&nbsp;&nbsp;");
                document.write("</td>");
            }
            document.write("</tr>");
        }
        document.write("</table>")
```

##函数
function()
![image.png](https://upload-images.jianshu.io/upload_images/9049859-24c256a3515bc10f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Array(注意数组的长度可改变)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7bc0878fbf452481.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Date函数
![image.png](https://upload-images.jianshu.io/upload_images/9049859-4417fe44cf6f5c16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Math函数
![image.png](https://upload-images.jianshu.io/upload_images/9049859-6c13c5c1e8e008ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
正则表达
![image.png](https://upload-images.jianshu.io/upload_images/9049859-496d7ddd427fcbc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

