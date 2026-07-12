#####1.zookeeper的不足

![image.png](https://upload-images.jianshu.io/upload_images/9049859-0e331c64e8c221fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.添加依赖
```
 <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-framework</artifactId>
                <version>4.2.0</version>
            </dependency>
            <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-recipes</artifactId>
                <version>4.2.0</version>
            </dependency>
```

...上面这个版本会报错的采用下面版本

>java.lang.ClassNotFoundException: org.apache.curator.connection.ConnectionHandlingPolicy

```
 <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-recipes</artifactId>
                <version>2.12.0</version>
            </dependency>
            <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-framework</artifactId>
                <version>2.12.0</version>
            </dependency>
```
**(注意异步操作)**
##连接zookeeper

```
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.RetryOneTime;

public class CuraterUnil {
    static CuratorFramework build=null;
    public static CuratorFramework createConnection() {
        build = CuratorFrameworkFactory.builder()
                .connectString("192.168.88.3:2181,192.168.88.4:2181,192.168.88.5:2181")
                .sessionTimeoutMs(4000)
                //重连机制
                .retryPolicy(new RetryOneTime(1000))
                .namespace("create.curator")
                .build();
        build.start();
        System.out.println(build.isStarted() ? "创建成功" : "创建失败");
        return build;
    }
    public static void closeConnection() {
        build.close();
    }
}
```

##重连策略
![image.png](https://upload-images.jianshu.io/upload_images/9049859-0d708e365a0d5b2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##创建节点/异步创建
```

import org.apache.curator.framework.CuratorFramework;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.data.ACL;
import org.apache.zookeeper.data.Id;

import java.util.ArrayList;
import java.util.List;

public class Node_T {
    public static void main(String[] args) throws Exception {
     //   createNode1();
     //   createNode2();
      //  createNode3();
        createNode4();

    }
    //创建节点
    public static void createNode1() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.create()
                //节点类型
                .withMode(CreateMode.PERSISTENT)
                //节点权限
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                //节点路径
                //节点数据
                .forPath("/create1", "create1".getBytes());
    }
    //自定义创建....IP类型......
    public static void createNode2() throws Exception {
      List<ACL> acls = new ArrayList<>();
        Id ip = new Id("ip", "192.168.88.4");
        acls.add(new ACL(ZooDefs.Perms.ALL,ip));
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.create()
                .withMode(CreateMode.PERSISTENT)
                .withACL(acls)
                .forPath("/create2","create".getBytes());
        System.out.println("==");
    }
    //递归创建   world模式
    public static void createNode3() throws Exception {
        List<ACL> acls = new ArrayList<>();
        Id id = new Id("world", "anyone");
        acls.add(new ACL(ZooDefs.Perms.ALL,id));
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.create()
                .creatingParentsIfNeeded()
                .withMode(CreateMode.PERSISTENT)
                .withACL(acls)
                .forPath("/createfirst/create3","crteate3".getBytes());
        System.out.println("===");
    }
    //异步创建
    public static void createNode4() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.create()
                .withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                //异步
                .inBackground((curatorFramework,curatorEvent)->
                {
                    System.out.println(curatorEvent.getPath() + "=" + curatorEvent.getType());
                })
                .forPath("/create4","create4".getBytes());

    }
}
```


##更新节点/异步
```

import org.apache.curator.framework.CuratorFramework;
public class Node_Y {
    public static void main(String[] args) throws Exception {
        setNode1();
        setNode2();
        setNode3();
    }
    public static void setNode1() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.setData()
                .forPath("/create1", "setdata".getBytes());
    }
    public static void setNode2() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.setData()
                //版本号
                .withVersion(-1)
                .forPath("/create4", "setdata".getBytes());
    }
    public static void setNode3() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.setData()
                .withVersion(-1)
                .inBackground((curatorFramework, curatorEvent) ->
                        System.out.println(curatorEvent.getType())
                )
                .forPath("/create1", "setdata1".getBytes());
        Thread.sleep(1000);
    }
}
```
##删除节点/异步
```
import org.apache.curator.framework.CuratorFramework;

public class Curator_U {
    public static void main(String[] args) throws Exception {
       // deleteNode();
        deleteNode1();
        deleteNode2();
        deleteNode3();
    }
    //删除节点
    public static void deleteNode() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.delete()
                .forPath("/create1");
    }
    //删除节点 版本
    public static void deleteNode1() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.delete()
                .withVersion(-1)
                .forPath("/create");
    }
    //删除有子节点的节点
    public static void deleteNode2() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.delete()
                .deletingChildrenIfNeeded()
                .withVersion(-1)
                .forPath("/createfirst");
    }
    //异步删除节点
    public static void deleteNode3() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.delete()
                .withVersion(-1)
                .inBackground((curatorFramework, curatorEvent) -> System.out.println(curatorEvent.getType()))
                .forPath("/create4");
    }
}
```

##获取节点信息/异步
```
import org.apache.curator.framework.CuratorFramework;
import org.apache.zookeeper.data.Stat;

public class Node_Q {
    public static void main(String[] args) throws Exception {
        getData1();
        getData2();
        getData3();
    }
    public  static  void  getData1() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        byte[] bytes = connection.getData()
                .forPath("/create1");
        System.out.println(new String(bytes));
    }
    public  static  void  getData2() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        //读取字节属性
        Stat stat = new Stat();
        byte[] bytes = connection.getData()
                //读取属性
                .storingStatIn(stat)
                .forPath("/create1");
        System.out.println(new String(bytes));
        System.out.println(stat.getVersion());
    }
    public  static  void  getData3() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.getData()
                .inBackground(((curatorFramework, curatorEvent) ->
                {
                    System.out.println(curatorEvent.getPath());
                    System.out.println(curatorEvent.getType());
                    System.out.println(new String(curatorEvent.getData()));
                }))
                .forPath("/create1");
        Thread.sleep(1000);
        System.out.println("====");
    }
}
```

##查看子节点信息
```
import org.apache.curator.framework.CuratorFramework;

import java.util.List;

public class Curater_P {
    public static void main(String[] args) throws Exception {
         getchild1();
         getchild2();
    }
    public  static  void  getchild1() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        List<String> list = connection.getChildren()
                .forPath("/createfirst");
        for (String s : list) {
            System.out.println(s);
        }
    }
    public  static  void  getchild2() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.getChildren()
                .inBackground(((curatorFramework, curatorEvent) -> {
                    System.out.println(curatorEvent.getType());
                    System.out.println(curatorEvent.getPath());
                    List<String> children = curatorEvent.getChildren();
                    children.forEach(System.out::println);
                }))
                .forPath("/createfirst");
        Thread.sleep(1000);
    }
}
```

##获取节点是否存在
```
   public  static  void status() throws Exception {
        //判断节点是否存在
        CuratorFramework connection = CuraterUnil.createConnection();
        Stat stat = connection.checkExists()
                .forPath("/create1");
        System.out.println(stat.getVersion());
    }
```



