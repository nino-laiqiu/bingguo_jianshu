##1.Tomcat

> ipconfig 查看IP地址

> netstat -ano 查看端口信息

>  startup.bat  启动Tomcat
启动报错:
1.版本不对应
2.java环境变量配置错误
(netstat -ano) 重新启动找到对应pid杀死进程

>server.xml (connector port 修改端口号)

>关闭Tomcat  正常关闭(shutdown.bat 或者 Ctrl+ C)

#####Tomcat部署方法
![image.png](https://upload-images.jianshu.io/upload_images/9049859-1da710c36eb02344.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####动态项目和动态项目
![image.png](https://upload-images.jianshu.io/upload_images/9049859-73f2fbe9daf69d8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####idea 和 Tomcat
web exploded和 war exploded 和 war 的区别:
war模式：
将WEB工程以包的形式上传到服务器 ；
war exploded模式：
将WEB工程以当前文件夹的位置关系上传到服务器；

##2.Servlet

#####概论与快速入门
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9512e7d2d63588b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>http://localhost:8080/MyWeb_war/demo1(前面要加上虚拟目录)....不加配置在如下配置
![image.png](https://upload-images.jianshu.io/upload_images/9049859-54b41e180a39a98b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
案例:
```
import javax.servlet.*;
import java.io.IOException;

public class MyServlet   implements Servlet {

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }
    //服务的启动
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("hello world &&");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```
```
<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <servlet>
        <servlet-name>demo1</servlet-name>
        <servlet-class>main.webapp.MyWeb.MyServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>demo1</servlet-name>
        <url-pattern>/demo1</url-pattern>
    </servlet-mapping>
</web-app>
```
(注意在web-app中添加)

#####IntelliJ IDEA: 无法创建Java Class文件

>https://blog.csdn.net/weixin_41986726/article/details/80767267?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160352654119724813211225%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=160352654119724813211225&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all

#####生命周期
![image.png](https://upload-images.jianshu.io/upload_images/9049859-9b4ea0de2d42f276.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####注解配置
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d35816793380192a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(@WebServlet(urlPatterns = "/demo3"))

#####idea相关配置
![image.png](https://upload-images.jianshu.io/upload_images/9049859-746b69a112fc13c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
