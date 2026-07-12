1.二分查找
代码:
```
import java.util.Arrays;

//请使用二分查找代码，对以下数据进行排序[5,8,1,9,7,6,3]，然后查找值为7的元素索引值并输出索引 值
public class BinarySearch {
    public static void main(String[] args) {

        int[] arr = {5, 8, 1, 9, 7,7,7,9,8,8};
        //二分查找必须排序
        Arrays.sort(arr);
        System.out.println(Arrays.toString(arr));
        int binary = binary(arr, 7);
        int binary1 = binary1(7, 0, arr.length - 1, arr);
        int binary2 = binary2(7, 0, arr.length - 1, arr);
        System.out.println(binary);//10
        System.out.println(binary1);//10
        System.out.println(binary2);//9
        //这三种方法结果可能不同
    }

    private static int  binary1(int key, int low, int hight,int[] arr ) {
        if (hight < low){return low;};
        int mid = low +(hight-low)/2;
        if (key < arr[mid] ){return  binary1(key,low,mid-1,arr);}
        else  if (key > arr[mid]){return binary1(key,mid+1,hight,arr);}
        else {return  mid;}
    }

    private static int  binary2(int key, int low, int hight,int[] arr ) {
        if (hight < low){return low;};
        int mid = low +(hight-low)/2;
        if (key < arr[mid] ){return  binary1(key,low,mid,arr);}
        else  if (key > arr[mid]){return binary1(key,mid,hight,arr);}
        else {return  mid;}
    }

    private static int binary(int[] arr,int a) {
        int low = 0;
        int hight =arr.length;
        int median = (low + hight)/2;
        while (low<=hight){
            if (arr[median] < a && low <=hight){
                int temp = median;
                low = temp;
                median = (low + hight)/2;
            }
            if (arr[median] > a && low <= hight){
                int temp = median;
                hight =temp;
                median = (low + hight)/2;
            }
            else {
                return median;
            }
        }
        return -1;
    }

}
```

API:
![概述](https://upload-images.jianshu.io/upload_images/9049859-bf1c030ba6bb052c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![概述](https://upload-images.jianshu.io/upload_images/9049859-6090eae73b802234.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![概述](https://upload-images.jianshu.io/upload_images/9049859-ac02cec8156d209d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![面试题](https://upload-images.jianshu.io/upload_images/9049859-0ca926877bf1a0c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
防止两个对象指向同一个地址值,如果一个改变,另一个也会改变.....
![image.png](https://upload-images.jianshu.io/upload_images/9049859-07c7221888c84f44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-13d4707bc112bf74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

补充:
![一道面试题 值传递和引用传递](https://upload-images.jianshu.io/upload_images/9049859-4db597622f1aa1a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-f5981338a26ead38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/9049859-7024030eaf3cfc75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![char和string的转换](https://upload-images.jianshu.io/upload_images/9049859-6ee4f644e0543004.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![byte 和 string ](https://upload-images.jianshu.io/upload_images/9049859-6e0f046e96d41a1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![常量在常量池](https://upload-images.jianshu.io/upload_images/9049859-6c58ed7eb90e6863.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








