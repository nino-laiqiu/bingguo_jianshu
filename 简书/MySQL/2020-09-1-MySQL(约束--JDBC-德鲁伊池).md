1.约束
![6](https://upload-images.jianshu.io/upload_images/9049859-a934f14be9dda05f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![例子](https://upload-images.jianshu.io/upload_images/9049859-85e583f47b2ac8af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![例子](https://upload-images.jianshu.io/upload_images/9049859-0e190b6437040cef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![例子](https://upload-images.jianshu.io/upload_images/9049859-67b3140e0c581165.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2.JDBC
代码和注意事项:
**注意不要导错包**
```
import java.sql.*;

public class Demo4 {
    
    public static void main(String[] args) throws SQLException, ClassNotFoundException {

        /*java连接mysql 执行的六个步骤
        * 在这之前先导入jar包,从mvnreposit网站下载 拖入idea, add as Library...作用*/

        //1.首先告诉软件,我们要用谁的驱动
        // DriverManager.registerDriver(new Driver());

         //或者采取反射的方法 获取因为在Driver类里 static静态代码快有 new Driver
        //Class.forName("com.mysql.jdbc.Driver");
        //对于8.0之后的版本加入cj
        Class.forName("com.mysql.cj.jdbc.Driver");

         //2.获取连接 返回一个  connection对象
        //对 URL 的说明:
        /*JDBC 是一个协议
        用冒号连接 mysql 是一个自协议
        用:// 连接资源地址
        对资源地址的说明 : 主机地址:端口/数据库
        * */
        Connection connection = DriverManager.getConnection("JDBC:mysql://localhost:3306/test","root","123456");
        //3.获取执行SQL语句 创建声明 返回一个statement对象
        Statement statement = connection.createStatement();
        //4.执行SQL语句  返回一个结果集
        ResultSet resultSet = statement.executeQuery("select * from course");
        //5.接收结果集
        while (resultSet.next()){
            //从角标1开始
 /* String cid = resultSet.getString(1);
            String cname= resultSet.getString(2);
            String tid = resultSet.getString(3);*/
            String cid = resultSet.getString("cid");
            String cname= resultSet.getString("cname");
            String tid = resultSet.getString("tid");
            System.out.println(cid +" " +cname + " " + tid);
        }
        //结束关闭连接 重置为null  2,3,4步骤  关闭的步骤从4-3-2
        resultSet.close();
        resultSet=null;
        statement.close();
        statement=null;
        connection.close();
        connection =null;
    }
}
```
**关于优化 把第一步改成反射获取 注意5.0 和 8.0 版本路径的不同
com.mysql.jdbc.Driver   com.mysql.cj.jdbc.Driver  
把第二部获取连接放在static中  把connection提升为成员变量**
注意关闭连接的顺序4-3-2


##2.最终优化
代码:
```
package Demo1;

import java.io.IOException;
import java.sql.*;
import java.util.Collection;
import java.util.Properties;

public class JDBCClass {
   static String driver;
   static String url;
   static String root;
   static String password;
   //配置文件加载一次
    static {
         Properties properties = new Properties();
       try {
           properties.load(Thread.currentThread().getContextClassLoader().getResourceAsStream("Demo1/JDBC.properties"));
       } catch (IOException e) {
           e.printStackTrace();
       }
       driver = properties.getProperty("driver");
         url = properties.getProperty("url");
         root = properties.getProperty("root");
         password = properties.getProperty("password");
    }

    public static Connection getConnection() throws ClassNotFoundException, SQLException {
          Class.forName(driver);
          return DriverManager.getConnection(url,root,password);
    }
    public  static  void  getIOclose(ResultSet resultSet,Statement statement, Connection connection) throws SQLException {
        if (resultSet != null){
            resultSet.close();
            resultSet =null;
        }
        if (statement!=null){
            statement.close();
            statement=null;
            if (connection!=null){
                connection.close();
            }
        }
    }

}

```
```
package Demo1;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class Day9_1test {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        Connection connection = JDBCClass.getConnection();
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("select * from course");
        while (resultSet.next()) {
            String string = resultSet.getString(1);
            String string1 = resultSet.getString(2);
            System.out.println(string + " " +string1);
        }
        JDBCClass.getIOclose(resultSet,statement,connection);
    }
}
```

##3.德鲁伊数据库池
代码:
```
package Demo1;

import com.alibaba.druid.pool.DruidAbstractDataSource;
import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class Demo10 {
    public static void main(String[] args) throws Exception {
        Properties properties = new Properties();
        properties.load(Thread.currentThread().getContextClassLoader().getResourceAsStream("Demo1/drird.properties"));
        DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
        Connection connection = dataSource.getConnection();
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("select  * from course");
        while (resultSet.next()) {
            String string = resultSet.getString(1);
            String string1 = resultSet.getString(2);
            String string2 = resultSet.getString(3);
            System.out.println(string + " " + string1 + " " + string2 );
        }
        JDBCClass.getIOclose(resultSet,statement,connection);
    }
}
```

##4.德鲁伊池封装代码
```
package Demo1;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class DriuidUill {
   static DataSource poll;
    static {
        Properties properties = new Properties();
        try {
            properties.load(Thread.currentThread().getContextClassLoader().getResourceAsStream("Demo1/drird.properties"));
            poll = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
                e.printStackTrace();
        }
    }
    public  static Connection getConnction() throws SQLException {
        return  poll.getConnection();
    }
    public  static  void  closeIO(ResultSet resultSet, Statement statement,Connection connection){
        try {
            if (resultSet!= null){
                resultSet.close();
                resultSet =null;
            }
            if (statement!=null){
                statement.close();
                statement =null;
            }
            if (connection!=null){
                connection.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
```
package Demo1;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class DrirdTest {
    public static void main(String[] args) throws SQLException {
        test();
       // test1();
    }


    private static void test() throws SQLException {
         //1,2步骤执行
        Connection connction = DriuidUill.getConnction();
        //3,声明
        Statement statement = connction.createStatement();
        ResultSet resultSet = statement.executeQuery("select  * from course");
        while (resultSet.next()) {
            String string = resultSet.getString(1);
            String string1 = resultSet.getString(2);
            String string2 = resultSet.getString(3);
            System.out.println(string + " " + string1 + " " + string2);
        }
        DriuidUill.closeIO(resultSet,statement,connction);
    }
    //添加数据
    private static void test1() throws SQLException {
        Connection connction = DriuidUill.getConnction();
        //3,声明
        Statement statement = connction.createStatement();
        int i = statement.executeUpdate("insert into course values('04','哲学','04') ");
        DriuidUill.closeIO(null,statement,connction);
    }

}
```
