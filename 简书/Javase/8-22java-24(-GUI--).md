
##1.GUI
![概述](https://upload-images.jianshu.io/upload_images/9049859-eeefbc94490b7539.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![概述](https://upload-images.jianshu.io/upload_images/9049859-974213e00048226a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![监听](https://upload-images.jianshu.io/upload_images/9049859-567c5a8ab56080e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![代码](https://upload-images.jianshu.io/upload_images/9049859-e2f1abd307f9bc7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###适配器
![适配器](https://upload-images.jianshu.io/upload_images/9049859-b482d0d7bffe9faf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###代码
![改进](https://upload-images.jianshu.io/upload_images/9049859-401a252d9341b722.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码
```
import java.awt.*;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.awt.event.WindowListener;

public class Demo1  {


    public static void main(String[] args) {
      //创建窗口对象 不可见
        Frame frame = new Frame();
        //创建窗口标题
        frame.setTitle("窗口");
        //可以通过构造设置,窗口标题
       // Frame chuangkou = new Frame("窗口");
        //设置窗口大小 单位像素
        frame.setSize(400,300);
        //设置窗口位置
        //可以一个方法搞定 setBounds
        //可以采用Dimension 和 Point方法来设置大小和位置
        frame.setLocation(400,200);
        //调用一个方法.让窗口可见
        frame.setVisible(true);
        //窗口的关闭 匿名内部类 要知道来源
      /*  frame.addWindowListener(new WindowListener() {
            @Override
            public void windowOpened(WindowEvent windowEvent) {
            }
            @Override
            public void windowClosing(WindowEvent windowEvent) {
// 关闭虚拟机
                System.exit(0);
            }
            @Override
            public void windowClosed(WindowEvent windowEvent) {
            }
            @Override
            public void windowIconified(WindowEvent windowEvent) {
            }
            @Override
            public void windowDeiconified(WindowEvent windowEvent) {
            }
            @Override
            public void windowActivated(WindowEvent windowEvent) {
            }
            @Override
            public void windowDeactivated(WindowEvent windowEvent) {

            }
        });*/
      //关闭案例的改进 适配器模式
        //WindowAdapter 是 WindowListener 的子类
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosing(WindowEvent e) {
        System.exit(0);
    }
});

    }
}
```
##添加
![功能布局](https://upload-images.jianshu.io/upload_images/9049859-67f4690d1a3af79b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###代码
```
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class Demo2 {
    public static void main(String[] args) {
        //创建一个有名字的不可见的窗口
        Frame frame1 = new Frame("窗口1");
        //添加属性
        frame1.setBounds(400,200,400,300);
        //设置为流式布局
        frame1.setLayout(new FlowLayout());

        //添加按钮对象 和 属性 名字
        Button button1 = new Button("$");
        button1.setSize(20,10);

        //创建文本框
        TextField textField = new TextField(20);
        //创建文本域
        TextArea textArea = new TextArea(10, 40);
        //按顺序添加组件
        frame1.add(textField);
        frame1.add(button1);
        frame1.add(textArea);
        //把按钮添加到窗口
      //  Component add = frame1.add(button1);

        //对按钮添加事件
       /* button1.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent actionEvent) {
                System.out.println("点我哇");
            }
        });*/

       //添加事件
        //getText setText 用法
        button1.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent actionEvent) {
                //获取文本框的值
                String st_str =  textField.getText().trim();
                //清空文本框数据  为什么在这里删除文本框的值
                textField.setText("");
                //设置给文本域 替换
                //textArea.setText(st_str);
                //设置给文本域  添加与换行
                textArea.append(st_str + "\r\n");
               // textField.setText("");  为什么不在此处删除
                //获取光标 作用是在添加时不用每次都要点文本框
                textField.requestFocus();

            }
        });
        //关闭窗口  适配器只需要重写一个方法
        frame1.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                System.exit(0);
            }
        });
          //窗口可视
        frame1.setVisible(true);

    }
}
```
##颜色案例代码
```
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class Demo3 {
    public static void main(String[] args) {
        Frame frame1 = new Frame("颜色更改");
        frame1.setBounds(400,300,300,200);
        frame1.setLayout(new FlowLayout());
        //Button会出现乱码  所以使用JButton
        JButton redbutton = new JButton("红色");
        JButton yellowbutton = new JButton("黄色");
        JButton bluebutton= new JButton("蓝色");
        JButton greenbutton = new JButton("绿色");
        frame1.add(redbutton);
        frame1.add(bluebutton);
        frame1.add(greenbutton);
        //对按钮添加动作事件
       /* redbutton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent actionEvent) {
                //设置背景色
                frame1.setBackground(Color.red);
            }
        });*/
       //对按钮添加鼠标点击事件
       /* redbutton.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent e) {
                frame1.setBackground(Color.RED);
            }
        });*/
        //对按钮添加鼠标进入事件
        redbutton.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseEntered(MouseEvent e) {
                frame1.setBackground(Color.RED);
            }
        });
        //对按钮添加鼠标离开事件
        redbutton.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseExited(MouseEvent e) {
                frame1.setBackground(Color.BLACK);
            }
        });
        bluebutton.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseEntered(MouseEvent e) {
                frame1.setBackground(Color.BLUE);
            }
        });
        //对按钮添加鼠标离开事件
        bluebutton.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseExited(MouseEvent e) {
                frame1.setBackground(Color.CYAN);
            }
        });
        greenbutton.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseEntered(MouseEvent e) {
                frame1.setBackground(Color.GREEN);
            }
        });
        //对按钮添加鼠标离开事件
        greenbutton.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseExited(MouseEvent e) {
                frame1.setBackground(Color.ORANGE);
            }
        });
        //退出
        frame1.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                System.exit(0);
            }
        });
        frame1.setVisible(true);
    }
}
```

###输入数字案例代码
```
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class Demo4 {
    public static void main(String[] args) {
        Frame frame = new Frame("输入数字案例");
        frame.setBackground(Color.WHITE);
        frame.setBounds(400,200,300,200);
        frame.setLayout(new FlowLayout());
//创建Lable标签对象
        //Lable汉字编码乱码 采取JLable
        JLabel label = new JLabel("请输入数字密码");
        //创建文本框
        TextField textField = new TextField(10);
        //添加
        frame.add(label);
        frame.add(textField);
        //给文本框添加事件
        textField.addKeyListener(new KeyAdapter() {
            @Override
            public void keyTyped(KeyEvent e) {
                //需要如果不是数字就取消输入
                //思路 先获取 判断 取消事件
                char keyChar = e.getKeyChar();
                if (!(keyChar>='0' && keyChar<='9')){
                    //取消输入
                   e.consume();
            }
                }
        });

        frame.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                System.exit(0);
            }
        });
        frame.setVisible(true);
    }
}
```
单级菜单案例代码





