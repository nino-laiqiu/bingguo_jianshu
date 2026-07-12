1.数据结构概况
2. LinkList中特有方法

addFirst/addLast:                      		添加头/添加尾
getFirst/getLast                         		获取头/获取尾
removeFirst/removeLast                   	移除头/移除尾
```
import java.util.Iterator;
import java.util.LinkedList;

//迭代器的并发 使用remove  打印遍历的变量 ???  删除操作
public class Demo2 {
    public static void main(String[] args) {
        LinkedList<String> objects = new LinkedList<>();
        objects.add("1");
        objects.add("2");
        objects.add("3");
        objects.add("4");
        /*objects.remove();
        System.out.println(objects);*/
        System.out.println(objects);
        Iterator<String> iterator = objects.iterator();
        while (iterator.hasNext()) {
            //System.out.println(iterator.next());
            String next = iterator.next();
            System.out.println(next);
            if (next.equals("1")) {
                //objects .remove(); 不能使用这个 会报错多线程
                iterator.remove();
            }
           //System.out.println(iterator.next());
        }
        System.out.println(objects);
    }
}
```
```
ort java.util.Arrays;
import java.util.List;

//数组转集合  集合转数组
public class Demo1 {
    public static void main(String[] args) {
        Integer[] st = {1, 2, 3, 4, 5, 67, 7, 8};
        List<Integer> strings = Arrays.asList(st);//得到的是Arrays中的内部类,该类并没有实现add/remove
        //使用Arrays.asList  得到的集合是一个只读的集合,不能进行add/remove的操作
        List<Integer> array = new ArrayList<>(strings);
        System.out.println(array);
        //集合转数组
        List<String> strings1 = new ArrayList<>();
        strings1.add("1");
        strings1.add("2");
        strings1.add("3");
        Object[] objects = strings1.toArray();
        // Integer[] array1 = (Integer[]) objects;//这种写法不行
        Integer[] array1 =new Integer[objects.length];
        for (int i = 0; i < objects.length; i++) {
            //先把object类型转为string类型  接着把string类型转为Integer类型
            array1[i] = Integer.parseInt(objects[i].toString());
        }
        System.out.println(Arrays.toString(array1));
    }
}
```
3, 增强for循环
4.可变参数
5.set集合]
6.Map集合
###统计次数 和钱的总和
```\port java.util.HashMap;

public class Demo10 {
    public static void main(String[] args) {
        HashMap<String, Stara> objectObjectHashMap = new HashMap<>();
        String st ="成龙,2;李小姐,1;成龙,80;哈哈哈,2;李小姐,1";
        //切割 按照  ;
        String[] split = st.split(";");
        for (int i = 0; i < split.length; i++) {
            String[] split1 = split[i].split(",");
            //创建对象
            Stara stara = new Stara(split1[0], Integer.parseInt(split1[1]), 1);
            if (!objectObjectHashMap.containsKey(split1[0])){
                objectObjectHashMap.put(split1[0],stara);
            }
            else {
                Stara stara1 = objectObjectHashMap.get(split1[0]);
                //多少次获取
              int money = stara1.money;
              int count =stara1.ci;
              money +=Integer.parseInt(split1[1]);
                      count++;
                 stara1.money =money;
               stara1.ci =count;
               objectObjectHashMap.put(split1[0],stara1);
            }
        } System.out.println(objectObjectHashMap);
    }
}
class Stara {
    String name;
    int money;
    int ci;
    @Override
    public String toString() {
        return "Stara{" +
                "name='" + name + '\'' +
                ", money=" + money +
                ", ci=" + ci +
                '}';
    }
    public Stara() {
    }
    public Stara(String name, int money, int ci) {
        this.name = name;
        this.money = money;
        this.ci = ci;
    }
}
```

统计字母出现的次数
```
mport java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Iterator;
//统计每个字符串出现的次数
public class Demo8 {
    public static void main(String[] args) {
        String st = "aabbcccddaabbc";
        char[] chars = st.toCharArray();//char数组
        //HashMap<String, String> hs = new HashMap<>();
        HashSet<Object> objects = new HashSet<>();
        for (int i = 0; i < st.length(); i++) {
            char c = st.charAt(i);
            boolean add = objects.add(c);
        }
        int count =0;
        //Iterator<Object> iterator = objects.iterator();
        Object[] objects1 = objects.toArray();
        System.out.println(Arrays.toString(objects1));
        for (int i = 0; i < objects1.length; i++) {
           // boolean equals = objects1[i].equals((Object) st);
            for (int i1 = 0; i1 < chars.length; i1++) {
               //objects1[i].toString().charAt(0).equals(); //把object 转为char 类型
                if ( objects1[i].equals(chars[i1])){
                    count++;
                }
            }
            System.out.println(objects1[i] );
            System.out.println(count);
            count =0;
            }
        }
    }
```
第二种方法(简洁) 及hashmap的四种遍历
```
import java.util.*;

public class Demo11 {
    public static void main(String[] args) {
        String st ="aabbccdddeaaabc";
        HashMap<Object, Integer> objectObjectHashMap = new HashMap<>();
        for (int i = 0; i < st.length(); i++) {
            char c = st.charAt(i);
            //判断是否包含
            if (!objectObjectHashMap.containsKey(c)){
                objectObjectHashMap.put(c,1);
            }
            //如果已经包含了 就把次数取出 然后重新给hashmap赋值
            else {
                Integer integer = objectObjectHashMap.get(c);
                integer++;
                objectObjectHashMap.put(c,integer);
            }
        }
        System.out.println(objectObjectHashMap);
        //对hashmap的遍历
        //第一种方法  keyset方法
        Set<Object> objects = objectObjectHashMap.keySet();
        for (Object object : objects) {//这一部分是对object的遍历
            System.out.println(object + ":"+objectObjectHashMap.get(object)); //get方法获取value  object是key值
        }
        //第二种方法  使用entryset方法
        Set<Map.Entry<Object, Integer>> entries = objectObjectHashMap.entrySet();
        for (Map.Entry<Object, Integer> entry : entries) {
            System.out.println(entry.getKey()+ ":" +entry.getValue());//这俩个方法的来源
        }
        //第三种方法 使用迭代器
        Iterator<Map.Entry<Object, Integer>> iterator = entries.iterator();//注意是entries
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        //第四种方法
        Collection<Integer> values = objectObjectHashMap.values();
        for (Integer value : values) {
            System.out.println(value + ":" );
        }
    }
}
```
