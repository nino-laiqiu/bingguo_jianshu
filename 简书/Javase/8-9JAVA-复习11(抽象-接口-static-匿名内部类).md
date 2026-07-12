1.复习8.8日的部分内容
```
public class Demo1 {
    public static void main(String[] args) {
        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append("1234");
        stringBuffer.insert(1,45);
        System.out.println(stringBuffer);
        stringBuffer.delete(1,3);
        System.out.println(stringBuffer);
        stringBuffer.deleteCharAt(0);
        System.out.println(stringBuffer);
        stringBuffer.reverse();
        String s = stringBuffer.toString();
        System.out.println(stringBuffer);
        System.out.println(s.equals("432"));
        String st = "12345";
        String substring = st.substring(0, 2);
        System.out.println(substring);
        char c = st.charAt(0);
        System.out.println(c);
        char[] chars = st.toCharArray();
        for (char aChar : chars) {
            System.out.println(aChar);}
            System.out.println(Arrays.toString(chars));
    }
    }
```
快速排序代码
```
import java.util.Arrays;

public class Demo2 {
    public static void main(String[] args) {
        int [] array = {1,4,5,7,3,8};
        get(array,0,array.length-1);
        System.out.println(Arrays.toString(array));
    }


    public static void get(int[] array, int low, int hight ){
        if (low > hight){
            return;
        }
        int i = low;
        int j =hight;
        int base = array[low];

        while (i < j){

            while (array[j] >= base && i < j) {
                j--;
            }
            while (array[i] <= base &&i <j ){
                i ++;
            }
            if (i <j) {
                int temp = array[i];
                array[i] = array[j];
                array[j] = temp;
            }
            }

            array[low] = array[i];
            array[i] = base;

       get(array,low,i-1);
       get(array,i+1,hight);

        }
    }
```
2.多态的好处
![方便](https://upload-images.jianshu.io/upload_images/9049859-1070f27865c192e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.递归思想
![概述](https://upload-images.jianshu.io/upload_images/9049859-db7416d3de8a31f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![求和概述](https://upload-images.jianshu.io/upload_images/9049859-3ceb2d9a253f2962.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.static归纳
static随着类的加载而加载,不能修饰构造器
![概述](https://upload-images.jianshu.io/upload_images/9049859-238c7f742b972a31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![类变量与实例变量内存分析](https://upload-images.jianshu.io/upload_images/9049859-53417ad5b3398a56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![注意事项](https://upload-images.jianshu.io/upload_images/9049859-b3fdcdbbcc2db7be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.抽象归纳
![抽象](https://upload-images.jianshu.io/upload_images/9049859-77a27082f4287cea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6.内部类概述
![成员内部类](https://upload-images.jianshu.io/upload_images/9049859-864dee83b83c1762.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7.成员变量 局部变量 静态变量
![区别](https://upload-images.jianshu.io/upload_images/9049859-ccd6cec486d6dd12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








