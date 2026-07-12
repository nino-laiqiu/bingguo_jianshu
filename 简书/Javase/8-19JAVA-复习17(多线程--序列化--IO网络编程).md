#1.消费者生产者
```
package Day19;
//反思多线程的逻辑
//类的设计逻辑
//notify位置应该添加在哪
//sleep位置及sleep的影响
//方法二 使用集合
public class Demo1 {
    public static void main(String[] args) {
        Cherk cherk = new Cherk();
        Product product = new Product(cherk);
        Product product1 = new Product(cherk);
        Cosumer cosumer = new Cosumer(cherk);
       // Cosumer cosumer1 = new Cosumer(cherk);
        product.setName("貂蝉");
        cosumer.setName("吕布");
        product1.setName("杨玉环");
        product.start();
        cosumer.start();
        product1.start();
        //cosumer1.start();
    }
}
class Cherk {
    private double no;
    public synchronized void Product() throws InterruptedException {
        //notify();
        if (no < 20) {
            no++;
            System.out.println(Thread.currentThread().getName() + "生产" + no);
            notify();
        } else {
           // notify();//唤醒另一个线程
            wait();//结束改线程
           // notify();
        }
    }
    public synchronized void Cosumertest() throws InterruptedException {
        // notify();
        if (no > 0) {
            System.out.println(Thread.currentThread().getName() + "取出" + no);
            no--;
            notify();
        } else {
           // notify();
//???????
            wait();
          /* while (no > -9){
                return;
            }*/
           // notify();

        }
    }
}
class Product extends Thread {
    Cherk cherk;
    public Product(Cherk cherk) {
        this.cherk = cherk;
    }
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() +"开始生产");
        // Thread.sleep(10000);
        while (true) {
            try {
                cherk.Product();
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
class Cosumer extends Thread {
    Cherk cherk;
    public Cosumer(Cherk cherk) {
        this.cherk = cherk;
    }
    @Override
    public void run() {
        System.out.println("开始消费");
        //Thread.sleep(10);
      lo:  while (true) {
            try {
                Thread.sleep(1);
                cherk.Cosumertest();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
通过改变 sleep的时间来控制  生成的数量 和 消费的数量 同步或者产生一些不同的打印结果
如下:改变 Thread.sleep(100); Thread.sleep(101);
![结果](https://upload-images.jianshu.io/upload_images/9049859-cd89d08540f0236a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

notify()的作用在于唤醒,如果不加最终的结果如下,我们发现最后的结果在20处
notify()唤醒  产生上面的结果
notify()的具体位置在哪会对最终结果产生影响
要避免把notify() 和 wait() 放在相邻的位置
![结果](https://upload-images.jianshu.io/upload_images/9049859-d6f7159c050fc474.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

采取一个容器 
private LinkedList<Object> list = new LinkedList<>();
当缓冲区已满时，生产者线程停止执行，放弃锁，使自己处于等状态，让其他线程执行；
当缓冲区已空时，消费者线程停止执行，放弃锁，使自己处于等状态，让其他线程执行。
当生产者向缓冲区放入一个产品时，向其他等待的线程发出可执行的通知，同时放弃锁，使自己处于等待状态；
当消费者从缓冲区取出一个产品时，向其他等待的线程发出可执行的通知，同时放弃锁，使自己处于等待状态。
>
https://blog.csdn.net/ldx19980108/article/details/81707751?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522159782955019724846463815%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=159782955019724846463815&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v3~pc_rank_v2-11-81707751.first_rank_ecpm_v3_pc_rank_v2&utm_term=%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85&spm=1018.2118.3001.4187

较好理解的代码

2.序列化
![概念](https://upload-images.jianshu.io/upload_images/9049859-4d4837cace75e50c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-2733a9b80641708a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-65d9787274da85de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7b26f3c3e20e4a53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.网络编程
![概述](https://upload-images.jianshu.io/upload_images/9049859-6bb74d929e8fcd6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3c07653534fda6a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-855bf55c73cedec9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
import java.net.InetAddress;
import java.net.UnknownHostException;

public class Demo2 {
    public static void main(String[] args) throws UnknownHostException {
        InetAddress localHost = InetAddress.getLocalHost();
        System.out.println(localHost);
        InetAddress byName = InetAddress.getByName("198.168.1.88");
        System.out.println(byName.getHostAddress());
        System.out.println(byName.getHostName());
    }
}
```
![image.png](https://upload-images.jianshu.io/upload_images/9049859-bf9163723b870983.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/9049859-74b1df1fc8bd8d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-57c2356c45ac1dc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-3d372ce695666243.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-d4ae1dd953476dc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-c321c20a67608bad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-4c5f9534814e4fdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
















