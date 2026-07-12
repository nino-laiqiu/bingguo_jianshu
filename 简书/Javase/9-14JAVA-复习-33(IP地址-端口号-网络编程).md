1.IP地址
![image.png](https://upload-images.jianshu.io/upload_images/9049859-32e0a15f1dccfa66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
(本机地址 127.0.0.1)
2.端口号(常用端口号)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-57dab533d7fe4673.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.TCP概述
![image.png](https://upload-images.jianshu.io/upload_images/9049859-a5456817d8eda42a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c5eb5c61a3a68df9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4.代码代码实现
![image.png](https://upload-images.jianshu.io/upload_images/9049859-55a1a3e519bffaf5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
public class TcpSeverSocket {
    public static void main(String[] args) throws IOException {
        //创建一个客户端
        ServerSocket serverSocket = new ServerSocket(8888);
        //获取客户端
        Socket accept = serverSocket.accept();
        //读取客户端的信息
        InputStream inputStream = accept.getInputStream();
        byte[] by = new byte[1024];
        int read = inputStream.read(by);
        System.out.println(new String(by,0,read));
        //传给客户端
        OutputStream outputStream = accept.getOutputStream();
        outputStream.write("你好客户端".getBytes());
        inputStream.close();
        outputStream.close();
    }
}
```
```
public class TcpSocket {

    public static void main(String[] args) throws IOException {
        //创建一个客户端 绑定IP和端口号
        Socket socket = new Socket("127.0.0.1", 8888);
        //传输
        OutputStream out = socket.getOutputStream();
        out.write("你好服务端".getBytes());
        //接收服务端的信息
        InputStream inputStream = socket.getInputStream();
        byte[] bytes = new byte[1024];
        int read = inputStream.read(bytes);
        System.out.println(new String(bytes, 0, read));
        out.close();
        inputStream.close();
    }

}
```
案例:从客户端上传图片到服务端的本机
堵塞问题与解决
![image.png](https://upload-images.jianshu.io/upload_images/9049859-1d24b73e724c497a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9049859-0cc2c4785a807b85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


BS案例分析
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c67893491ff8e5a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码实现:






