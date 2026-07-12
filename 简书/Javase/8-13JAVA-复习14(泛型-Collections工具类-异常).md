1.泛型
2. 比较器 Comparable   Collections
(Comparable)案例:实现对于List<Person> 年龄的升序**
实现排序的步骤:
1. 让 Person 类实现  Comparable<Person>
2. 重写compareTo 方法:   定义排序的规则
                           升序: this和 参数
                           降序: 参数和this 
(Collections)
```
不使用匿名内部类实现: 自己写一个Comparator的子类
	 * Collections.sort(list,new IntegerComparator());
		System.out.println(list);*/
		
		//使用匿名内部类实现
		Collections.sort(list,new Comparator<Integer>() {
			/**
			 * 用第一个和第二个比较:  升序
			 * 第二个和第一个比较:   降序
			 */
			@Override
			public int compare(Integer o1, Integer o2) {
				return o1 - o2;
			}
		}
);
		System.out.println(list);
	}
}

class IntegerComparator implements Comparator<Integer>{
	@Override
	public int compare(Integer o1, Integer o2) {
		return o2- o1;
	}
}
```
3.Collections工具类
public static <T> void sort(List<T> list)            		排序,升序
public static <T> int binarySearch(List<?> list,T key)           二分查找,不存在返回负数,只能针对升序集合
        public static <T> T max(Collection<?> coll)              	 最大值
        public static void reverse(List<?> list)              		反转
         public static void shuffle(List<?> list)               		 随机打乱                   
        public static <T> void sort(List<T> list, Comparator<? super T> c)  排序,和比较器配合使用


4.一道大题
```
package Day13;
/*
        A:B,C,D,F,E,O;B:A,C,E,K;C:F,A,D,I;D:A,E,F,L;E:B,C,D,M,L;F:A,B,C,D,E,O,M;G:A,C,
        D,E,F;H:A,C,D,E,O;I:A,O;J:B,O;K:A,C,D;L:D,E,F;M:E,F,G;O:A,H,I,J

        1.获取所有用户对应的好友数量
        2 获取指定两个用户的共同好友
        3 获取两两人的共同好友
        */


import java.util.*;

public class Demo2 {
    public static void main(String[] args) {
        String st = " A:B,C,D,F,E,O;B:A,C,E,K;C:F,A,D,I;D:A,E,F,L;E:" +
                "B,C,D,M,L;F:A,B,C,D,E,O,M;G:A,C," +
                " D,E,F;H:A,C,D,E,O;I:A,O;J:B,O;K:A" +
                ",C,D;L:D,E,F;M:E,F,G;O:A,H,I,J";
        // number(st);
        //mutualfriend(st);
        mutualfriend1(st);
        //getSampleFriend(st);
    }

    //1.获取所有用户对应的好友数量
    public static void number(String st) {
    /*HashMap<String, ArrayList<String>> ob= new HashMap<>();
    ArrayList<String> obj = new ArrayList<>();*/
        String[] split = st.split(";");
        for (int i = 0; i < split.length; i++) {
            //按照":"来切 分成两个部分 用户和好友
            String[] split1 = split[i].split(":");//用户
            String[] split2 = split1[1].split(",");//好友
            System.out.println(split1[0] + "的好友数" + split2.length);
            //可以用hashmap 把数量存入value
        }
    }

    //获取指定两个用户的共同好友
    public static void mutualfriend(String st) {
        Scanner scanner = new Scanner(System.in);
        ArrayList<String> obj = new ArrayList<>();
        //重点....
        HashMap<String, ArrayList<String>> ob = new HashMap<>();
        String[] split = st.split(";");
        for (int i = 0; i < split.length; i++) {
            String[] split1 = split[i].split(":");
            String[] split2 = split1[1].split(",");
            //集合转数组.....
            List<String> strings = Arrays.asList(split2);
            //构造方法把List 转为ArrayList
            ArrayList<String> st1 = new ArrayList<>(strings);
            ob.put(split1[0], st1);
        }
        //System.out.println(ob);
        //以上 把数据都存入hashmap
        //
        System.out.println("请输入两个字母");
        System.out.println("请输入第一个字母");
        String name1 = scanner.next();
        System.out.println("请输入第二个字母");
        String name2 = scanner.next();
        String s = name1.toUpperCase();
        String s1 = name2.toUpperCase();
        //判断是否包含改用户
        /*if (!ob.containsKey(s) || !ob.containsKey(s1)) {
            System.out.println("没有这个用户");
            return;
        }*/
        //获取 value值
        ArrayList<String> strings = ob.get(s);
        ArrayList<String> strings1 = ob.get(s1);
        // 比较 把交集放在strings 中
        strings.retainAll(strings1);
        if (strings.isEmpty()) {
            System.out.println(name1.toUpperCase() + "和" + name2.toUpperCase() + "没有共同好友");
        } else {
            System.out.println(name1.toUpperCase() + "和" + name2.toUpperCase() + strings);
        }
    }
    public static void   mutualfriend1(String st){
        ArrayList<String> obj = new ArrayList<>();
        //重点....
        HashMap<String, ArrayList<String>> ob1 = new HashMap<>();
        String[] split = st.split(";");
        for (int i = 0; i < split.length; i++) {
            String[] split1 = split[i].split(":");
            String[] split2 = split1[1].split(",");
            //集合转数组.....
            List<String> strings = Arrays.asList(split2);
            //构造方法把List 转为ArrayList
            ArrayList<String> st1 = new ArrayList<>(strings);
            ob1.put(split1[0], st1);
        }
        //取出 key 放入ArrayList 中 使用ArrayList中的get()方法  注意 与hashmap中的get()方法是不同的
        Set<String> strings = ob1.keySet();
        ArrayList<String> strings1 = new ArrayList<>(strings);
        for (int i = 0; i < ob1.size(); i++) {
            for (int j = 1 +i; j < ob1.size(); j++) {
                String key1 = strings1.get(i);
                String key2 = strings1.get(j);
                ArrayList<String> strings2 = ob1.get(key1);
                //中间  temp的作用
                ArrayList<String> temp = new ArrayList<>(strings2);
                ArrayList<String> strings3 = ob1.get(key2);
//这里使用中间temp 是因为如果是string1则再次使用获取string时 不在发生变化 get(key1) 已然发生变化
                temp.retainAll(strings3);
                if (temp.size() == 0){
                    System.out.println(key1 + "与" + key2 +"无共同好友");
                }
                else {
                    System.out.println(key1 + "与" + key2 + "的共同好友有" +temp);
                }

            }
        }
    }
    private static void getSampleFriend(String str) {
        //第一次切割  区分用户和用户之间
        String[] sp = str.split(";");

        HashMap<String, ArrayList<String>> hm = new HashMap<String, ArrayList<String>>();

        //第二次切割  将用户和其对应的好友部分分裂
        for (String s : sp) {
            String[] persons = s.split(":");

            //第三次切割  将每个用户对应的好友进行分离操作
            String[] friends = persons[1].split(",");

            //将好友数组变成集合
            List<String> list = Arrays.asList(friends);

            ArrayList<String> a = new ArrayList<>(list);

            //将用户和他对应的好友都存进集合中
            hm.put(persons[0], a);
        }


        //求交集
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入第一个用户名");
        String name1 = sc.next();

        System.out.println("请输入第二个用户名");
        String name2 = sc.next();

        //判断集合中是否包含对应的用户,如果不包含  就打印用户不存在
        if (!hm.containsKey(name1.toUpperCase()) || !hm.containsKey(name2.toUpperCase())) {
            System.out.println("用户不存在,请重新运行");
            return;
        }

        //将用户对应的好友的集合取出  然后求交集
        ArrayList<String> a1 = hm.get(name1.toUpperCase());
        ArrayList<String> a2 = hm.get(name2.toUpperCase());

        a1.retainAll(a2);


        //判断交集是否为空  如果为空  证明没有共同好友
        if (a1.isEmpty()) {
            System.out.println(name1.toUpperCase() + "和" + name2.toUpperCase() + "没有共同好友");
        } else {
            System.out.println(name1.toUpperCase() + "和" + name2.toUpperCase() + "的共同好友为:" + a1);
        }


    }

}
```
5.第二道大题
6.单链表
7.二叉树
