##Request
(1) 原理
![image.png](https://upload-images.jianshu.io/upload_images/9049859-82d856a054f7d8f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0763d7fef03389ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(2)继承结构
![image.png](https://upload-images.jianshu.io/upload_images/9049859-f306de0214623b9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(3).方法的介绍
![image.png](https://upload-images.jianshu.io/upload_images/9049859-dc166df7ea68cbf5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(4).创建
**Servlet 快速创建**
![image.png](https://upload-images.jianshu.io/upload_images/9049859-e61e371a7aa5e9df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-127dcc1c6483d69a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
@WebServlet(urlPatterns =  "/ServletDemo1")
public class ServletDemo1 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
         //请求头
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            String header = request.getHeader(name);
            System.out.println(name+ "---" + header);
        }
        //获取头
        String header = request.getHeader("user-agent");
        String reforer = request.getHeader("reforer");
        System.out.println(reforer);
        //为什么使用 contains
        if (header.contains("chrome")){
            System.out.println("谷歌来了");
        }
        //请求行
        String method = request.getMethod();
        System.out.println(method);
        System.out.println(request.getContextPath());
        //比较URL 和 URI 的区别
        System.out.println(request.getRequestURI());
        System.out.println(request.getRequestURL());
        //GET
        ///MyWeb_war
        ///MyWeb_war/ServletDemo1
        //http://localhost:8080/MyWeb_war/ServletDemo1
```
#####中文乱码问题
![image.png](https://upload-images.jianshu.io/upload_images/9049859-37ab88b436689b85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####请求访问
![image.png](https://upload-images.jianshu.io/upload_images/9049859-406b5a5f22c5b8e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-ed3cfbe1fc0fb28a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####共享数据
![image.png](https://upload-images.jianshu.io/upload_images/9049859-f17664074bb361e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##登录案例
![image.png](https://upload-images.jianshu.io/upload_images/9049859-6e5422a294509c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(BeanUtils的使用)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-b19560cbbbe5888c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
