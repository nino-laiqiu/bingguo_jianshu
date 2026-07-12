##1.数据模型
![image.png](https://upload-images.jianshu.io/upload_images/9049859-295244a017922316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2.基础命令(增删改查)

create [-s] [-e]  path data     -s(有序的) -e(临时的)
set path data  更新节点  (可以传入版本号)
delete  path  (当某节点有子节点不能使用这个命令)
rmr  path   删除节点(子节点也删除)  
get  path 获取节点信息   stat path  只返回属性
ls path (查看子节点信息)     ls2  path(ls2是ls的增强，除了列出子结点还有当前结点的属性)

##登录客户端
bin/zkCli.sh    
bin/zkCli.sh    -server  linux03:2181
bin/zkCli.sh    -server  linux03:2181,linux04:2181,linux05:2181

##ACL权限控制

#####scheme ：权限模式

world 只有一个用户，代表登陆zookeeper的所有人（默认）
ip 对客户端使用ip地址认证
auth 使用已添加认证的用户认证
digest 使用用户名：密码方式认证

#####permission ：授予的权限

create 简写c 可以创建子结点
delete 简写d 可以删除子结点（仅下一级）
read 简写r 可以读取结点数据以及显示子结点
write 简写w 可以设置结点数据
admin 简写a 可以设置节点访问控制列表权限

getAcl path 读取权限
setAcl path acl 设置权限
addauth schema suth 添加认证用户

world授权模式
>setAcl path world:anyone:<acl>

ip授权模式
>setAcl path ip:<ip>:<acl>

auth模式
>addauth digest <user>:<password> 添加认证用户
>setAcl <path> auth:<user>:<acl>  授权

digest授权模式
>setAcl <path> digest:<user>:<password>:<acl>
>echo -n sjh:sjh2019. | openssl dgst -binary -sha1 | openssl base64生成密文

多种授权模式
同一个结点可以使用多种授权模式，用逗号隔开就行

超级管理员
>生成密文添加到脚本/bin/zkServer.sh

##zookeeper java版基础命令(增删改查)
1.创建代码如下
```
import org.apache.hadoop.yarn.webapp.hamlet2.Hamlet;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.ACL;
import org.apache.zookeeper.data.Id;

import java.util.ArrayList;
import java.util.List;

public class CreateNode_Test {
    public static void main(String[] args) throws KeeperException, InterruptedException {
        ZooKeeper zooKeeper = ZooKeeper_Test.getZooKeeper();
        // 参数1 结点路径
        // 参数2 结点数据
        // 参数3权限列表 OPEN_ACL_UNSAFE代表world方式授权 cdrwa
        // 参数4 结点类型 persistent表示持久化结点

        //创建一个节点  全权限
        zooKeeper.create("/create/node1", "node1".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);

        //创建一个节点:world模式 自定义(创建 和 删除)
        List<ACL> acls = new ArrayList<>();
        Id id = new Id("world", "anyone");
        acls.add(new ACL(ZooDefs.Perms.CREATE, id));
        acls.add(new ACL(ZooDefs.Perms.DELETE, id));
        zooKeeper.create("create/node2", "node2".getBytes(), acls, CreateMode.PERSISTENT);
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);

        //创建一个节点 ip模式授权
        List<ACL> acls1 = new ArrayList<>();
        Id ip = new Id("ip", "192.168.88.3");
        acls1.add(new ACL(ZooDefs.Perms.WRITE, ip));
        acls1.add(new ACL(ZooDefs.Perms.READ, ip));
        zooKeeper.create("/create/node3", "node3".getBytes(), acls1, CreateMode.PERSISTENT);
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);

        // auth方式授权  可自定义设置权限
        zooKeeper.addAuthInfo("digest", "user:123456".getBytes());
        zooKeeper.create("/create/node4", "node4".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);


        //digest方式授权
        List<ACL> acls2 = new ArrayList<>();
        Id digest = new Id("digest", "user:123243546:密文");
        acls2.add(new ACL(ZooDefs.Perms.WRITE, digest));
        zooKeeper.create("/create/node5", "node5".getBytes(), acls2, CreateMode.PERSISTENT);
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);


        //创建 临时节点 创建有序节点
        //修改第四项

        //异步创建节点
        zooKeeper.create("/create/node6","node6".getBytes(),ZooDefs.Ids.CREATOR_ALL_ACL,CreateMode.PERSISTENT,
                (rc,path,name,txt)->{
                    System.out.println("是否创建成功" +rc);
                    System.out.println("节点路径"+path);
                    System.out.println("节点内容"+name);
                    System.out.println("节点上下文"+txt);
                } ,"contxt");
        Thread.sleep(1000);
    }
}

```
其他的命令java版本
```
import org.apache.zookeeper.AsyncCallback;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;

import java.util.List;

public class OthersOrder {
    public static void main(String[] args) throws KeeperException, InterruptedException {
        ZooKeeper zooKeeper = ZooKeeper_Test.getZooKeeper();
        //更新操作
        //第二个参数的说明要更新的内容(data)
        //第三个参数的说明 版本号 -1表示不参与更新 如果版本号不匹配则报错
        Stat stat = zooKeeper.setData("/set/node1", "node1".getBytes(), -1);
        System.out.println(stat.getVersion());//获取版本号
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);


        //异步更新
        zooKeeper.setData("/set/node2","node2".getBytes(),-1,
                (rc,path,data,txt)->{
                    System.out.println(rc);
                    System.out.println(path);
                    System.out.println(data);
                    System.out.println(txt);
                },"contxt" );
        Thread.sleep(1000);
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);

        //删除节点 同样可以异步删除同上
        zooKeeper.delete("delete/node",-1);
        ZooKeeper_Test.ZooKeeperClose(zooKeeper);



        //获取节点的信息
        //同步读取数据
        //第一个参数是路径
        //第二个参数是watch,先填false（以后在讲）
        //第三个参数用于获取结点属性
        Stat stat1 = new Stat();
        byte[] data = zooKeeper.getData("/get/node", false, stat1);
        System.out.println(new String(data));//查看数据信息
        System.out.println(stat1.getCversion());//查看版本

       //异步查看
        zooKeeper.getData("/get/node", false,
                new AsyncCallback.DataCallback() {
                    @Override
                    public void processResult(int i, String s, Object o, byte[] bytes, Stat stat) {

                    }
                },
                "contxt");

        zooKeeper.getData("/get/node2",false,
                (rc,path,ctx,data1,stat2)->{
                    System.out.println(rc);
                    System.out.println(path);
                    System.out.println(ctx);
                    System.out.println(new String(data1));
                    System.out.println(stat2.getVersion());
                },"contxt");
        Thread.sleep(1000);




        // 获取子节点数据
        List<String> children = zooKeeper.getChildren("/children", false);
        for (String child : children) {
            System.out.println(child);
        }

        //异步查看......


    
       //检查结点是否存在
        Stat exists = zooKeeper.exists("/exist", false);
        System.out.println(exists==null);

        //异步查看......
    }
}

```




