

getChild1() 方法获取的只包含子节点的名称,所以要切割!!!!!!!!!


1.监听机制(自定义 实现)

(解决监听器 一次性问题)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-048845087882d53d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(注册多个监听器)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2c8ce081d0ecc6c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;

public class ZKWatcher  implements Watcher {
    public static void main(String[] args) throws IOException, InterruptedException {
        ZooKeeper zooKeeper = new ZooKeeper("192.168.88.3:2181", 5000, new ZKWatcher());
        Thread.sleep(1000);
        zooKeeper.close();
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        try {
            if (watchedEvent.getType()==Event.EventType.None){
                 if (watchedEvent.getState()==Event.KeeperState.Disconnected){
                     System.out.println("断开连接");
                 }
                 else  if (watchedEvent.getState()==Event.KeeperState.AuthFailed){
                     System.out.println("认证失败");
                 }
                 else if (watchedEvent.getState()==Event.KeeperState.Expired){
                     System.out.println("会话超时");
                 }
                 else if (watchedEvent.getState()==Event.KeeperState.SyncConnected){
                     System.out.println("创建连接成功");
                 }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

关于配置中心的案例(mysql的URL username password)的动态变化
```
import com.sun.scenario.effect.impl.sw.sse.SSEBlend_SRC_OUTPeer;
import org.apache.zookeeper.*;

import javax.security.auth.login.Configuration;
import java.io.IOException;
import java.util.concurrent.CountDownLatch;

//配置中心案例
public class ConfigurationCentre implements Watcher {
    static ZooKeeper zooKeeper;
    static CountDownLatch countDownLatch = new CountDownLatch(1);
    static String IP = new String();
    private String url;
    private String username;
    private String password;

    public static void main(String[] args) {
         //测试
        ConfigurationCentre configurationCentre = new ConfigurationCentre();
        try {
            for (int i = 0; i < 20; i++) {
                Thread.sleep(5000);
                System.out.println("url"+configurationCentre.getUrl());
                System.out.println("username"+configurationCentre.getUsername());
                System.out.println("password"+configurationCentre.getPassword());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }


    }
     //构造方法
    public ConfigurationCentre() {
        intTvalue();
    }

    public void intTvalue() {
        try {
            zooKeeper = new ZooKeeper(IP, 5000, this);
            countDownLatch.await();//阻塞线程,等待连接成功
            // zooKeeper.create("/create/node", "node".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
            this.url = new String(zooKeeper.getData("/Data/url", true, null));
            this.username = new String(zooKeeper.getData("/Data/username", true, null));
            this.password = new String(zooKeeper.getData("/Data/password", true, null));
            Thread.sleep(1000);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            try {
                zooKeeper.close();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        //连接超时要重新连接
        if (watchedEvent.getType() == Event.EventType.None) {
            if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                System.out.println("连接成功");
                countDownLatch.countDown();//开启线程
            } else if (watchedEvent.getState() == Event.KeeperState.Expired) {
                System.out.println("连接超时");
                //继续连接
                try {
                    zooKeeper = new ZooKeeper("192.168.88.1:2181", 5000, this);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            } else if (watchedEvent.getState() == Event.KeeperState.AuthFailed) {
                System.out.println("验证失败");
            } else if (watchedEvent.getState() == Event.KeeperState.Disconnected) {
                System.out.println("断开连接");
            }
            //当配置信息发生改变了,重新获取
            else if (watchedEvent.getType()== Event.EventType.NodeDataChanged){
                      intTvalue();
            }

        }
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

唯一ID的生成(节点是临时的????)
```
import org.apache.zookeeper.*;

import java.io.IOException;
import java.util.concurrent.CountDownLatch;

public class GloballyUniqueID implements Watcher {
    static CountDownLatch countDownLatch = new CountDownLatch(1);
    static ZooKeeper zooKeeper;

    public static void main(String[] args) {
        GloballyUniqueID globallyUniqueID = new GloballyUniqueID();
        for (int i = 0; i < 5; i++) {
            System.out.println(globallyUniqueID.getUniqueID());
        }
    }
    @Override
    public void process(WatchedEvent watchedEvent) {
        //连接超时要重新连接

        if (watchedEvent.getType() == Event.EventType.None) {
            if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                System.out.println("连接成功");
               // countDownLatch.countDown();//开启线程
            } else if (watchedEvent.getState() == Event.KeeperState.Expired) {
                System.out.println("连接超时");
                //继续连接
                try {
                    zooKeeper = new ZooKeeper("192.168.88.1:2181", 5000, this);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            } else if (watchedEvent.getState() == Event.KeeperState.AuthFailed) {
                System.out.println("验证失败");
            } else if (watchedEvent.getState() == Event.KeeperState.Disconnected) {
                System.out.println("断开连接");
            }
        }
    }
    public String getUniqueID()  {
        String path ="";
        try {
            //这是一个临时的
            path= zooKeeper.create("/create", new byte[0], ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.EPHEMERAL_SEQUENTIAL);
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return path.substring(5);
    }
}
```
##分布式锁
(注意绝对路径和相对路径)
(注意主线程的阻塞)
```
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.io.IOException;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class MyLock implements Watcher {
    public CountDownLatch countDownLatch = new CountDownLatch(1);
    public ZooKeeper zooKeeper;
    public static final String LOCK_ROOT_PATH = "Lock";
    public static final String LOCK_NODE_PATH = "lock_";
    static String lockpath;
    static String IP = "192.168.88.3:2181";

    //测试方法
    public static void main(String[] args) throws InterruptedException, IOException, KeeperException {
             //售票案例
        MyLock myLock = new MyLock();
        myLock.createMylock();
        sell();
        myLock.deletenode();
    }
     public static void  sell(){
         System.out.println("卖票");
         Long min = 50000L;
         try {
             Thread.sleep(min);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
         System.out.println("结束");
     }
    //节点锁的生成
    public void createMylock() throws IOException, InterruptedException, KeeperException {
        zooKeeper = new ZooKeeper(IP, 5000, this);
        //阻塞主线程,等待连接
        countDownLatch.await();
        //判断主节点是否存在
        Stat exists = zooKeeper.exists(LOCK_ROOT_PATH, false);
        if (exists==null){
            zooKeeper.create(LOCK_ROOT_PATH,new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
        }
        lockpath = zooKeeper.create(LOCK_ROOT_PATH + "/" + LOCK_ROOT_PATH, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        //获取状态
        System.out.println("节点创建" + lockpath);

    }
    Watcher watcher = new Watcher() {
        @Override
        public void process(WatchedEvent watchedEvent) {
                  if (watchedEvent.getType()==Event.EventType.NodeDeleted){
                      //监听上一个节点 假如上一个节点删除了 则唤醒主线程
                      synchronized (watcher){
                          notifyAll();
                      }
                  }
        }
    };
    //节点锁的获取
    public  void  attemptLock() throws KeeperException, InterruptedException {
        //获取子节点...子节点路径为相对路径
        List<String> children = zooKeeper.getChildren(LOCK_ROOT_PATH, false);
        //对子节点进行排序
        Collections.sort(children);
        //获取节点
        int i = children.indexOf(LOCK_ROOT_PATH.length() + 1);
        //判断节点锁是否为第一个,是则进行下一步的操作,不是进行判断
        if (i==0){
            return;
        }
        else {
            //先判断第一个节点是否,存在 因为是多线程 在执行中可能消失
            //获取节点的相对路径
            String path = children.get(i-1);
            //exit 填写绝对路径
            //监听是否存在
            Stat exists = zooKeeper.exists(LOCK_ROOT_PATH + "/" + path, watcher);
             if (exists ==null){
                 attemptLock();
             }
             else {
                 synchronized (watcher){
                      //阻塞,要等到上一个节点删除唤醒
                     watcher.wait();
                         //wait();
                         attemptLock();
                 }
             }
        }
    }
    //节点锁的删除
    public  void  deletenode() throws KeeperException, InterruptedException {
        zooKeeper.delete(this.lockpath,-1);
        zooKeeper.close();
        System.out.println("删除锁");
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
          if (watchedEvent.getType()==Event.EventType.None){
              if (watchedEvent.getState()==Event.KeeperState.SyncConnected){
                  System.out.println("连接成功");
                  countDownLatch.countDown();
              }
              if (watchedEvent.getState()==Event.KeeperState.Expired){
                  System.out.println("会话超时");
                  //是否再次连接
                /*  try {
                      zooKeeper = new ZooKeeper(IP, 5000, this);
                  } catch (IOException e) {
                      e.printStackTrace();
                  }*/
              }
          }
    }
}
```
