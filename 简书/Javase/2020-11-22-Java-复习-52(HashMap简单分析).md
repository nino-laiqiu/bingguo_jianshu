##1.再谈: ==  地址值   equals   hashcode 

当用（==）进行比较的时候，比较的 是他们在内存中的存放地址，所以，除非是同一个new出来的对象，他们比较地址值一样为true，否则比较后结果为false。
对象是放在堆中的，栈中存放的是对象的引用（地址）。由此可见“==”是对栈 中的值进行比较的。如果要比较堆中对象的内容是否想用，那么就要重写equals方法了。
为什么重写equals方法，还必须要重写hashcode方法?比较时,1.使用hashcode方法提前校验，可以避免每一次比对都调用equals方法，提高效率2.保证是同一个对象，如果重写了equals方法，而没有重写hashcode方法，会出现equals相等的对象，hashcode不相等的情况，重写hashcode方法就是为了避免这种情况的出现。


##2.hashmap的hashcode方法原码
```
    static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }

```
HashMap中hash(Object key)原理，为什么(hashcode ＞＞＞ 16)


##3.hashmap扩容机制
初始容量
加载因子


扩容的标准:
threshold = capacity * loadFactor*，当Size>=threshold的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 衡量数组是否需要扩增的一个标准。
```
// loadFactor加载因子
//loadFactor加载因子是控制数组存放数据的疏密程度，loadFactor越趋近于1，
// 那么 数组中存放的数据(entry)也就越多，也就越密，也就是会让链表的长度增加，loadFactor越小，也就是趋近于0，
// 数组中存放的数据(entry)也就越少，也就越稀疏。
//loadFactor太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。
// loadFactor的默认值为0.75f是官方给出的一个比较好的临界值。
//给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 
// 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。
//threshold
//*threshold = capacity * loadFactor*，当Size>=threshold的时候，
// 那么就要考虑对数组的扩增了，也就是说，这个的意思就是 衡量数组是否需要扩增的一个标准。
```
## put方法
```
//当桶数组 table 为空时，通过扩容的方式初始化 table
//查找要插入的键值对是否已经存在，存在的话根据条件判断是否用新值替换旧值
//如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
//判断键值对数量是否大于阈值，大于的话则进行扩容操作
```
