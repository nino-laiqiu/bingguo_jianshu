值得注意的要点:
HBaseConfiguration和Configuration 的区别 第一个加载了配置文件
记忆一下: connection = ConnectionFactory.createConnection();方法

##DDL
```
import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 * Description:
 * User: tongyongtao
 * Date: 2020-11-06
 * Time: 20:34
 * DDL语法
 */
public class DDLTable {
    public static void main(String[] args) throws IOException {
        // System.out.println(createTable("tab_111", "f1", "f2"));
        // System.out.println(isTableExist("tab_1"));
       // System.out.println(addFamily("tab_1", "f3"));
        //System.out.println(dropTable("tab_1"));
        System.out.println(createNamespace("myspace"));
    }

    //判断表是否存在
    public static Boolean isTableExist(String tableName) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        //对表的操作
        Admin admin = connection.getAdmin();
        return admin.tableExists(TableName.valueOf(tableName));
    }

    public static Boolean createTable(String tableName, String... cfs) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        //首先判断表是否存在
        //判断列族是否为空
        //发现一个错误,竟然没写!  查了好久
        if (!isTableExist(tableName)) {
            if (cfs.length > 0) {

                Admin admin = connection.getAdmin();
                //采用工厂方法
                TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(TableName.valueOf(tableName));
                //添加列族
                for (String cf : cfs) {
                    ColumnFamilyDescriptorBuilder columnFamily = ColumnFamilyDescriptorBuilder.newBuilder(cf.getBytes());
                    tableDescriptorBuilder.setColumnFamily(columnFamily.build());
                }
                admin.createTable(tableDescriptorBuilder.build());
                return true;
            }
        }
        return false;
    }

    public static boolean createTable1(String table, String... cfs) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        //判断列族是否存在
        if (cfs == null) {
            return false;
        }
        //判断表是否存在
        if (isTableExist(table)) {
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
    //添加列族
    public static Boolean addFamily(String tableName, String... cfs) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Admin admin = connection.getAdmin();
        //先判断表是否存在
        if (isTableExist(tableName)) {
            for (String cf : cfs) {
                ColumnFamilyDescriptorBuilder columnFamily = ColumnFamilyDescriptorBuilder.newBuilder(cf.getBytes());
                admin.addColumnFamily(TableName.valueOf(tableName), columnFamily.build());
            }
            return true;
        }
        return false;
    }
    //删除表
    public static Boolean dropTable(String tableName) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        if (isTableExist(tableName)) {

            Admin admin = connection.getAdmin();
            admin.disableTable(TableName.valueOf(tableName));
            admin.deleteTable(TableName.valueOf(tableName));
            return true;
        }
        return false;
    }
    //创建名称空间
    public  static  Boolean createNamespace(String spacename) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Admin admin = connection.getAdmin();
        NamespaceDescriptor build = NamespaceDescriptor.create(spacename).build();
        admin.createNamespace(build);
        return  true;
    }
}
```

##DML
```
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.CompareOperator;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.exceptions.DeserializationException;
import org.apache.hadoop.hbase.filter.*;
import org.apache.hadoop.hbase.util.Bytes;


import java.io.IOException;
import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class DMLTable {
    /**
     * Created with IntelliJ IDEA.
     * Description:
     * User: tongyongtao
     * Date: 2020-11-06
     * Time: 22:00
     */

    public static void main(String[] args) throws Exception {
        // putData("tab_2", "60005", "status", "name", "小胡");
        // getData("tab_2","10001");
        //  scanData("tab_2","","");
        // filterData("tab_2");
        // filterData1("tab_2","status","age","19");
          // filterData2("tab_2","status");
        filterData3("tab_2","name");
    }

    public static void putData(String tableName, String rowkey, String cf, String cm, String values) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Put put = new Put(Bytes.toBytes(rowkey));
        put.addColumn(Bytes.toBytes(cf), Bytes.toBytes(cm), Bytes.toBytes(values));
        table.put(put);
    }

    //获取数据get方法
    public static void getData(String tableName, String rowkey) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(Bytes.toBytes(rowkey));
        Result result = table.get(get);
        for (Cell cell : result.rawCells()) {
            System.out.println(new String(CellUtil.cloneFamily(cell)));
            // System.out.println(Arrays.toString(Bytes.toBytes(ByteBuffer.wrap(CellUtil.cloneQualifier(cell)))));
            System.out.println(new String(CellUtil.cloneQualifier(cell)));
            System.out.println(new String(CellUtil.cloneValue(cell)));
            System.out.println(new String(CellUtil.cloneRow(cell)));
        }
    }

    //get方法获取数据
    public static void getData1(String tableName, String rowkey) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));

        List<Get> gets = new ArrayList<>();
        //获取多个rowkey
        gets.add(new Get(Bytes.toBytes("100001")));
        gets.add(new Get(Bytes.toBytes("10002")));
        Result[] results = table.get(gets);
        for (Result result : results) {
            for (Cell cell : result.rawCells()) {

            }
        }
    }

    //scan方法
    public static void scanData(String tableName, String start, String stop) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        try (ResultScanner scanner = table.getScanner(scan)) {
            for (Result result : scanner) {
                for (Cell cell : result.rawCells()) {
                    System.out.println(((Bytes.toString(CellUtil.cloneRow(cell)))));
                    System.out.println(Bytes.toString(CellUtil.cloneFamily(cell)));
                }
            }
        }
    }

    //关于scan的过滤器  列族过滤
    public static void filterData(String tableName) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        Filter filter = new RowFilter(CompareOperator.EQUAL, new SubstringComparator("10001"));
        scan.setFilter(filter);
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            for (Cell cell : result.rawCells()) {
                System.out.println(Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
    }

    //关于scan过滤  值过滤 一般采用SingleColumnValueFilter
    public static void filterData1(String tableName, String cf, String cm, String values) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        Filter filter = new SingleColumnValueFilter
                (Bytes.toBytes(cf), Bytes.toBytes(cm), CompareOperator.EQUAL, Bytes.toBytes(values));
        scan.setFilter(filter);
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            for (Cell cell : result.rawCells()) {
                System.out.println(Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
    }
    //列族的过滤
    public static void filterData2(String tableName, String cf) throws Exception {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        FamilyFilter filter = new FamilyFilter(CompareOperator.EQUAL, new BinaryComparator(Bytes.toBytes(cf)));
        scan.setFilter(filter);
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            for (Cell cell : result.rawCells()) {
                System.out.println(Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
    }

    public  static  void  filterData3(String tableName, String cm) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        QualifierFilter filter = new QualifierFilter(CompareOperator.EQUAL,new  SubstringComparator(cm));
        scan.setFilter(filter);
        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            for (Cell cell : result.rawCells()) {
                System.out.println(Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
    }
}
```
```
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 * Description:
 * User: tongyongtao
 * Date: 2020-11-07
 * Time: 14:41
 */
public class DeleteData {
    public static void main(String[] args) throws IOException {
         // deleteData("tab_2","10001");
          deleteData1("tab_2","40001","status");
    }

     //删除rowkey
    public  static  void  deleteData(String tableName,String rowkey) throws IOException {
        Connection connection = HbaseUtils.getConnection();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(Bytes.toBytes(rowkey));
        table.delete(delete);
    }

    public  static  void  deleteData1(String tableName,String rowkey ,String cf ) throws IOException {
        Table table = HbaseUtils.getConnection().getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(Bytes.toBytes(rowkey));
        delete.addFamily(Bytes.toBytes(cf));
        table.delete(delete);
    }
}
```


(怎么在某个命名空间创建表: 在表之前添加spacename:tablename)

##hbase中的过滤器
>https://blog.csdn.net/weixin_42047967/article/details/105757146?biz_id=102&utm_term=SubstringComparator&utm_medium=distribute.pc_search_result.none-task-blog-2~all

例如:
SubstringComparator  判断提供的子串是否出现在value中，并且不区分大小写。包含字串返回0，不包含返回1，仅支持 EQUAL 和非 EQUAL
BinaryComparator  二进制比较器，用于按字典顺序比较指定字节数组。
