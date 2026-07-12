####1.分发

将phoenix安装目录下的
phoenix-5.0.0-HBase-0.98-client.jar
phoenix-core-5.0.0-HBase-0.98.jar
拷贝到hbase的lib目录下

将hbase下的site文件覆盖到Phoenix的bin目录下的site文件


##2.增删改查
> ! table 查看表

> 创建一张表(不加双引号就是大写)
```
create table user(
id varchar primary key,
name varchar,
password varchar
);
```


>  upsert into table values ()  添加和修改

(联合主键)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3f52a44726a505c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d1150e0b48b2ee29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3.JDBC

(依赖,注意com.lmax)
```
 <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client -->
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>2.2.5</version>
    </dependency>
         <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-server -->
         <dependency>
             <groupId>org.apache.hbase</groupId>
             <artifactId>hbase-server</artifactId>
             <version>2.2.5</version>
         </dependency>
         <!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-core -->
         <dependency>
             <groupId>org.apache.phoenix</groupId>
             <artifactId>phoenix-core</artifactId>
             <version>5.0.0-HBase-2.0</version>
         </dependency>
         <!-- https://mvnrepository.com/artifact/com.lmax/disruptor -->
         <dependency>
             <groupId>com.lmax</groupId>
             <artifactId>disruptor</artifactId>
             <version>3.3.6</version>
         </dependency>
```
**简单版本**
```
public class HbaseP1 {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {

        Class.forName("org.apache.phoenix.jdbc.PhoenixDriver");
        Connection conn = DriverManager.getConnection("jdbc:phoenix:linux03:2181");
        String sql = "select  * from TAB_11";
        PreparedStatement preparedStatement = conn.prepareStatement(sql);
        ResultSet resultSet = preparedStatement.executeQuery();
        //execute(); executeQuery();
        while (resultSet.next()) {
            System.out.println(resultSet.getString(1));
            System.out.println(resultSet.getString(2));
            System.out.println(resultSet.getString(3));
        }
        resultSet.close();
        preparedStatement.close();
        conn.close();
    }
}
```

##4.二级索引


##把hbase上的数据导入到pheonix上 : 

> java.lang.NoSuchMethodError: org.apache.hadoop.hbase.KeyValueUtil.length(Lorg/apache/hadoop/hbase/Cell;)I

如下错误:但确实数据迁移过来了

drop table tablename 会把hbase上的表也会删除
