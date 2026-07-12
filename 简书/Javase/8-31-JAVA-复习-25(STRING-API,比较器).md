1.compare To 方法
参见博客关于compareTo基础用法
>https://blog.csdn.net/weixin_39981912/article/details/96873239?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param

注意当两个比较的字符串是英文且长度相等时 : 比较char
1.一个字符
2.多个字符首字母不等 直接比较首字母
3.多个字符首字母相等,取比较下一个不等的字母

###比较器
Comparable和Comparator的区别
Comparable: 内部比较器,java.lang; 如果一个List想要使用Collections.sort() 做排序,需要集合中的元素所在的类实现Comparable<T>接口,重写compareTo: 

         this在前,升序;        参数在前 : 降序

Comparator: 外部比较器,java.util;  如果一个类中不能实现Comparable,或者是对应Comparable中的排序方式不满意,可以通过Comparator重新定义排序的规则,而不需要修改原有类的结构, Collections.sort(list,Comparator)//匿名内部类;
**TreeSet,TreeMap用法(重点)**
TreeSet 和TreeMap 排序比较有两种方法,以TreeSet为例:如果是自定义对象,在那个对象类继承comparable方法,泛型为那个类型,或者在new TreeSet构造方法new comparator 匿名内部类重写compareTo方法;

