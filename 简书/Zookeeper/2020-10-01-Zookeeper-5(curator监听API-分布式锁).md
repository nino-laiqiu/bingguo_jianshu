##1.事件监听机制


>ava.lang.NullPointerException
会报这个错,空指针??有的节点没有值吗? 实践了一下发现不对.....为解决

```
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.curator.framework.recipes.cache.PathChildrenCache;

public class Curater_Zookeeper1 {
    public static void main(String[] args) throws Exception {
     // watcher1();
         watcher2();
        //watcher3();
    }
    public  static  void  watcher1() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        NodeCache watcher = new NodeCache(connection, "/create1");
        watcher.start();
        watcher.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
            //    System.out.println(watcher.getCurrentData().getPath());
                System.out.println(new String(watcher.getCurrentData().getData()));
            }
        });
   /*     watcher.getListenable().addListener(()-> {
            System.out.println(watcher.getCurrentData().getPath());
            System.out.println(new String(watcher.getCurrentData().getData()));
        });*/
        Thread.sleep(100000);
        watcher.close();
    }
    public  static  void  watcher2() throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        PathChildrenCache pathChildrenCache = new PathChildrenCache(connection, "/createfirst", true);
        pathChildrenCache.start();
        pathChildrenCache.getListenable().addListener((curatorFramework,childrenCacheEvent)->{
            System.out.println(childrenCacheEvent.getType());
             System.out.println(childrenCacheEvent.getData().getPath());
            System.out.println(new String(childrenCacheEvent.getData().getData()));
        });
        Thread.sleep(10000000);
        pathChildrenCache.close();
    }

    public  static  void watcher3() throws Exception{
        //监视某个结点 arg1 连接对象 arg2 监视路径
        CuratorFramework client = CuraterUnil.createConnection();
        NodeCache nodeCache=new NodeCache(client,"/watcher1");
        //启动监视器
        nodeCache.start();
        System.out.println("监视器已打开");
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                System.out.println("结点路径： "+nodeCache.getCurrentData().getPath());
            }
        });
        Thread.sleep(150000);
        //关闭监视器
        nodeCache.close();
        System.out.println("监视器已关闭");
    }
}
```

##curator事务命令
```
import org.apache.curator.framework.CuratorFramework;

public class Curatur_K {
    public static void main(String[] args) throws Exception {
        CuratorFramework connection = CuraterUnil.createConnection();
        connection.inTransaction()
                .create().forPath("/create5","create5".getBytes())
                .and()
                .delete().withVersion(-1).forPath("/create1")
                .and()
                .commit();
    }
}
```
##curator分布式锁
**InterProcessMutex 分布式可重入排它锁**
同时启动两个客户端
```
 @Test
    public void lock1() throws Exception{
        //排它锁
        InterProcessLock interProcessLock=new InterProcessMutex(client,"/lock1");
        System.out.println("等待获取锁对象");
        interProcessLock.acquire();
        for(int i=1;i<=10;i++){
            Thread.sleep(3000);
            System.out.println(i);
        }
        interProcessLock.release();
        System.out.println("等待释放锁");
    }
```
**InterProcessReadWriteLock 分布式读写锁
读锁之间是共享的**

```
	@Test
    public void lock2() throws Exception{
        //读写锁
        InterProcessReadWriteLock interProcessReadWriteLock=new InterProcessReadWriteLock(client,"/lock1");
        InterProcessMutex interProcessLock = interProcessReadWriteLock.readLock();
        System.out.println("等待读锁对象");
        interProcessLock.acquire();
        for(int i=1;i<=10;i++){
            Thread.sleep(3000);
            System.out.println(i);
        }
        interProcessLock.release();
        System.out.println("等待释放锁");
    }
	@Test
    public void lock3() throws Exception{
        //读写锁
        InterProcessReadWriteLock interProcessReadWriteLock=new InterProcessReadWriteLock(client,"/lock1");
        InterProcessMutex interProcessLock = interProcessReadWriteLock.writeLock();
        System.out.println("等待写锁对象");
        interProcessLock.acquire();
        for(int i=1;i<=10;i++){
            Thread.sleep(3000);
            System.out.println(i);
        }
        interProcessLock.release();
        System.out.println("等待释放锁");
    }
```
