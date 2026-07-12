1.我们通过Test.getStaticTest()来调用第二种：Test t = new Test();t.getStaticTest()来调用第二种的调用方式一般不推荐！首先是静态方法不需要实例化对象，实例化之后再调用会造成内存空间的浪费。其次，会让阅读代码的人产生误解，以为此方法为非静态的方法。
问题是new 浪费空间  还是调用浪费空间
2.多线程
![同步方法解决线程安全问题](https://upload-images.jianshu.io/upload_images/9049859-daad071a281cc291.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![lock](https://upload-images.jianshu.io/upload_images/9049859-2954d6dd97861f32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##一个例题
```
//有俩个储户向同一个账户存钱,一次存1000 存10次
public class Demo10 {
    public static void main(String[] args) {
        Account account = new Account(0);
        Consumer consumer = new Consumer(account);
        Consumer consumer1 = new Consumer(account);
        consumer.setName("貂蝉");
        consumer1.setName("吕布");
        consumer.start();
        consumer1.start();
        //测试
    }
}
class Account {
    private double monney;
    public Account(double monney) {
        this.monney = monney;
    }
    //提供一个存钱方法
    //多线程
    public synchronized void add(double monney) throws InterruptedException {
        if (monney > 0) {
            this.monney += monney;
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + "账户有" + this.monney + "元");
        }
    }
}

class Consumer extends Thread {
    private Account account;
    public Consumer(Account account) {
        this.account = account;
    }
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
                account.add(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

