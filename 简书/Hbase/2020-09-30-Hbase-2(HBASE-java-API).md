#####1.maven依赖

```
 <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-auth</artifactId>
            <version>3.2.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.2.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>2.2.5</version>
        </dependency>
        <!-- 使用mr程序操作hbase 数据的导入 -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-mapreduce</artifactId>
            <version>2.2.5</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.5</version>
        </dependency>
        <!-- phoenix 凤凰 用来整合Hbase的工具 -->
       <!-- <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-core</artifactId>
            <version>5.0.0-HBase-2.0</version>
        </dependency>-->
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <!-- bind to the packaging phase -->
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>
</project>
```

##2.JAVA-API
```
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.Arrays;

/**
 * @paramrowkey 唯一标识符
 */
public class HbaseMainAPI {
    public static void main(String[] args) throws IOException {
        // System.out.println(exitTable("tab_1"));
        System.out.println(deleteData("tab_4", "rk0001", "", "", ""));
    }

    //DDL 判断表是否存在
    public static boolean exitTable(String table) throws IOException {
        Connection connection = HaseUnil.getConnection();
        //获取管理员对象
        Admin admin = connection.getAdmin();
        boolean b = admin.tableExists(TableName.valueOf(table));
        admin.close();
        return b;

    }

    //DDL创建一张表  注意可变形参的用法
    public static boolean createTable(String table, String... cfs) throws IOException {
        Connection connection = HaseUnil.getConnection();
        //判断列族是否存在
        if (cfs == null) {
            return false;
        }
        //判断表是否存在
        if (exitTable(table)) {
            return true;
        }
        Admin admin = connection.getAdmin();
        //表的构建
        TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(TableName.valueOf(table));
        //列族的构建
        for (String cf : cfs) {
            ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder = ColumnFamilyDescriptorBuilder.newBuilder(cf.getBytes());
            tableDescriptorBuilder.setColumnFamily(columnFamilyDescriptorBuilder.build());
        }
        admin.createTable(tableDescriptorBuilder.build());
        return true;
    }

    //DDL 删除一张表
    public static boolean dropTable(String table) throws IOException {
        if (!exitTable(table)) {
            return true;
        }
        Connection connection = HaseUnil.getConnection();
        Admin admin = connection.getAdmin();
        TableName tableName = TableName.valueOf(table);
        admin.disableTable(tableName);
        admin.deleteTable(tableName);
        return true;
    }

    //DDL 创建名称空间  异常的学习  NamespaceExistException
    public static boolean createNameSpace(String ns) throws IOException {
        Connection connection = HaseUnil.getConnection();
        Admin admin = connection.getAdmin();
        NamespaceDescriptor namespaceDescriptor = NamespaceDescriptor.create(ns).build();
        try {
            admin.createNamespace(namespaceDescriptor);
        } catch (NamespaceExistException e) {
            System.out.println("存在");
        } catch (IOException e) {
            e.printStackTrace();
        }
        return true;
    }

    //DML 添加数据
    public static boolean putRow(String table, String rowkey, String columFamilyName, String qualifier, String value) throws IOException {
        Connection connection = HaseUnil.getConnection();
        //获取表
        Table table1 = connection.getTable(TableName.valueOf(table));
        //rowkey 唯一标识符
        Put put = new Put(Bytes.toBytes(rowkey));
        put.addColumn(Bytes.toBytes(columFamilyName), Bytes.toBytes(qualifier), Bytes.toBytes(value));
        table1.put(put);
        table1.close();
        return true;
    }

    //DML 获取数据 get方法
    public static boolean getData(String table, String rowkey, String columFamilyName, String qualifier, String value) throws IOException {
        Connection connection = HaseUnil.getConnection();
        Table table1 = connection.getTable(TableName.valueOf(table));
        Get get = new Get(Bytes.toBytes(rowkey));
        //2.1
        // Get get1 = get.addFamily(Bytes.toBytes(columFamilyName));
        //2.2
        // Get get2 = get.addColumn(Bytes.toBytes(columFamilyName), Bytes.toBytes(qualifier));
        Result result = table1.get(get);
        for (Cell cell : result.rawCells()) {
            System.out.println(new String(CellUtil.cloneFamily(cell)));
            System.out.println(Arrays.toString(CellUtil.cloneQualifier(cell)));
            System.out.println(Bytes.toString(CellUtil.cloneValue(cell)));
        }
        return true;
    }

    //DML 获取数据 scan方法
    public static boolean scanerTable(String table) throws IOException {
        Connection connection = HaseUnil.getConnection();
        Table table1 = connection.getTable(TableName.valueOf(table));
        ResultScanner scanner = table1.getScanner(new Scan());
        for (Result result : scanner) {
            for (Cell cell : result.rawCells()) {
                System.out.println(Arrays.toString(CellUtil.cloneRow(cell)));
                System.out.println(new String(CellUtil.cloneFamily(cell)));
                System.out.println(Arrays.toString(CellUtil.cloneQualifier(cell)));
                System.out.println(Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
        return true;
    }

    //DML 删除数据的操作
    public static boolean deleteData(String table, String rowkey, String columFamilyName, String qualifier, String value) throws IOException {
        Connection connection = HaseUnil.getConnection();
        Table table1 = connection.getTable(TableName.valueOf(table));
        Delete delete = new Delete(rowkey.getBytes());//删除列族 相当于命令行中的deleteall

        //2.0
        // delete.addFamily(columFamilyName.getBytes());
        //2.1
        // delete.addColumn(columFamilyName.getBytes(),qualifier.getBytes());//删除最新的版本
        //2.2//删除所有的版本,如果添加时间戳,则删除所有小于这个时间的所有版本
        // delete.addColumns(columFamilyName.getBytes(),qualifier.getBytes())
        table1.delete(delete);
        table1.close();
        return true;
    }
}
```



