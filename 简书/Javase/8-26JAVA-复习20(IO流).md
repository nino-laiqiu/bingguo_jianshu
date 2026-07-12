1.二分查找
代码....
```
import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;

//二分查找  采用方法二 联系快速排序的递归思想
public class Demo1  {
    public static void main(String[] args) {
        int[] arr ={1,5,7,89,4,3,2,67,8};
        //排序 使用实现或者调用
//implements Comparable<T> 用来自定义排序.........
/*实现方法时怎么只重写一个方法 采用他的子类
对象名.接口(new 接口的子类) 匿名内部类重写方法*/
        Arrays.sort(arr);
        System.out.println(Arrays.toString(arr));
        System.out.println(paixu(arr, 0, arr.length, 5));
        System.out.println(paixu1(arr, 0, arr.length, 5));
    }
    //方法1
    private static int paixu1(int[] arr, int low, int length, int key) {
        if (low>length){return -1;}
        int mid =low +(length-low)/2;
        while (low <=length){
            if (key< arr[mid]&&low<length){
                int temp =mid;
                length=temp;
                mid=low+(length-low)/2;
            }
            else if(key>arr[mid]&&low<length){
                int temp =mid;
                low=temp;
                mid=low+(length-low)/2;
            }
            else {return mid;}
        }
        return -1;
    }
    //方法二
    private static int paixu(int[] arr, int low, int length,int key) {
if (low > length ){return -1;}
int mid =low +(length-low)/2;
if (key < arr[mid]){paixu(arr,low,mid-1,key);}
if (key > arr[mid]){paixu(arr,mid+1,length,key);}
return mid;
    }
}
```
