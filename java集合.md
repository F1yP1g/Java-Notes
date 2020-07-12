
容器主要包括 `Collection` 和 `Map` 两种，`Collection` 存储着对象的集合，而 `Map` 存储着键值对（两个对象）的映射表。

[java容器](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/Java%20%E5%AE%B9%E5%99%A8.md)
# 1 Collection 

   ![](pic/Collection.png "Collection")
## ArrayList
[ArrayList介绍](https://www.cnblogs.com/skywang12345/p/3308556.html)
1. 数组队列，相当于动态数组
2. 继承`AbstractList`，实现了`List, RandomAcess（随机访问功能，根据序号获取元素）,  Cloneable（拷贝）,java.io.Serializable（序列化）`接口。
3. **线程不安全！**，多线程环境建议使用`Vector`或者`CopyOnWriteArrayList`。
4. 默认容量`10`，扩容机制：`新容量 = 旧容量*1.5`。扩容调用`Arrays.copyOf()`将原数组复制到新数组中，代价很高，建议初始化时大概指定容量大小，减少扩容次数。
5. `ArrayList`删除元素的时间复杂度`O(N)`
6. 第一次`add`时分配空间
## Vector
1. 默认大小为`10`
2. `Vector`与`ArrayList`类似，但使用`synchorized`进行**同步**。
3. 默认扩容机制为**两倍**，可在构造函数中指定增长的量大小。
## CopyOnWriteArrayList
1. 读写分离
   - 写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。
   - 写操作需要加锁，防止并发写入时导致写入数据丢失。
   - 写操作结束之后需要把原始数组指向新的复制数组。
2. 适用场景
   - 解决了多线程`List`线程不安全的问题
   - 适合**读多写少**的场景。
   - 不适合**内存敏感以及对实时性要求很高**的场景。
## LinkedList
[LinkedList介绍](https://www.cnblogs.com/skywang12345/p/3308807.html)
1. 基于双向链表实现，使用`Node`存储节点信息.
2. 继承`AbstractSequentialList`，实现了`List, Qeque, Cloneable, java.io.Serializable`接口。
3. 非同步的
4. 顺序访问非常高效，而随机访问效率比较低。
5. `LinkedList`可以作为队列、双端队列和栈使用。
## PriorityQueue
1. 堆，默认是最小堆
## HashSet
`HashSet`（⽆序，唯⼀）: 基于` HashMap `实现的，底层采⽤` HashMap `来保存元素
## LinkedHashSet
`LinkedHashSet` 继承于` HashSet`，并且其内部是通过` LinkedHashMap `来实现
的。有点类似于我们之前说的`LinkedHashMap `其内部是基于` HashMap `实现⼀样，不过还是有⼀
点点区别的
## TreeSet
`TreeSet`（有序，唯⼀）： 红⿊树(⾃平衡的排序⼆叉树)
# 2 Map
![](pic/Map.png "Map")
## HashMap
[HashMap介绍-cnblog](https://www.cnblogs.com/skywang12345/p/3310835.html)

[HashMap介绍-github](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

[HashMap介绍-知乎](https://zhuanlan.zhihu.com/p/30360734)

[HahMap遍历方式](https://mp.weixin.qq.com/s/Zz6mofCtmYpABDL1ap04ow)

![](pic/HashMap.png "HashMap存储结构")
内部包含了一个 `Entry` 类型的数组 `table`。`Entry` 存储着键值对。它包含了四个字段，从 `next` 字段我们可以看出 `Entry` 是一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。`HashMap` 使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的 `Entry`。
1. 继承于`AbstractMap`，实现了`Map、Cloneable、java.io.Serializable`接口。
2. 默认大小`16`，即桶（bucket）数组的大小
3. 不是线程安全的。
4. key、value都可以为`null`。
5. `HashMap`中的映射不是有序的。
6. JDK1.7及之前：数组+链表实现；JDK1.8：数组+链表或树实现（一个桶存储的链表长度大于等于 `8` 时会将链表转换为红黑树。）
7. 两个重要参数：**初始容量**，**装载因子**
   - 容量：哈希表中桶的数量
   - 装载因子：容量可以达到多满的尺度，默认为`0.75`，当哈希表中的条目数超过加载因子和当前容量的乘积，则进行`rehash`操作（重建内部数据结构），哈希表将具有大约两倍的桶数。 
8. `new`后不会立即分配`bucket`数组，在第一次`put`时初始化。
9. `bucket`数组大小一定是`2`的幂。
10. `put`操作
    1.  对key的hashCode()进行hash后计算数组下标index;
    2.  如果当前数组table为null，进行resize()初始化；
    3.  如果没碰撞直接放到对应下标的bucket里；
    4.  如果碰撞了，且节点已经存在，就替换掉 value；
    5.  如果碰撞后发现为树结构，挂载到树上。
    6.  如果碰撞后为链表，添加到链表尾，并判断链表如果过长(大于等于TREEIFY_THRESHOLD，默认8)，就把链表转换成树结构；
    7.  数据 put 后，如果数据量超过threshold，就要resize。
11. 与`HashTable`比较
    1.  `HashTable`内部的方法使用`synchronized`进行同步
    2.  `HashMap` 可以插入键为 `null` 的 `Entry`，`hashTable`不支持`null`键。
    3.  `HashMap` 不能保证随着时间的推移 `Map` 中的元素次序是不变的。
    4.  `HashTable`默认初始化大小为`11`，每次扩容，容量变为`2n+1`，`HashMap`默认`16`扩容为`2`倍。
## ConcurrentHashMap
和 `HashMap` 实现上类似，但它是线程安全的，且效率较高。
1. JDK 1.7
   1. 分段琐（Segment），每个琐维护几个桶，多个线程可以同时访问不同分段琐上的桶，从而提高并发度（Segment的个数）。
   2. 默认的并发级别为`16`，也就是默认创建`16`个`Segment`。
        ```java
        static final int DEFAULT_CONCURRENCY_LEVEL = 16;
        ```
2. JDK 1.8
   1. 取消了`Segment`，采用`CAS`和`synchronized`保证并发安全。`CAS(原子操作)`失败时使用`sytnchronized`。
   2. 数组+链表/红⿊⼆叉树。
   3. `synchronized`只锁定当前链表/红黑树的**首节点**，这样只要hash不冲突就不会产生并发，提升效率N倍。
# 3 问题
## List, Set, Map 的区别
- List：存储元素是有序的，可重复的
- Set：元素是无序的、不可重复
- Map：键值对，键是无序、不可重复的，值是无序的、可重复的。
## ArrayList 和 Vector 区别
1. ArrayList：List 的主要实现类，底层用 Object[] 存储，适用于频繁的查找工作，线程不安全
2. Vector：List 的古老实现类，底层用 Object[] 存储，线程安全
## 快速失败（fail-fast）
Java 集合的一种错误检测机制。在使用迭代器对集合遍历时，在多线程下操作非安全失败（fail-safe）的集合类可能会触发 fail-fast 机制，导致抛出 `ConcurrentModificationException` 异常。另外，在单线程下如果遍历过程中对集合对象的内容进行了修改的话也会触发 fail-fast 机制。
>增强 for 循环也是借助迭代器进行遍历

原因：
每当迭代器使用 `hasNext()/next()` 遍历下个元素之前，都会检测 `modCount` 变量是否为 `expectedModeCount` 值，是的话就返回遍历，否则抛出异常终止遍历。如果在遍历期间修改元素，就会改变 `mdCount` 值，从而抛出异常。
>通过 `Iterator` 的方法修改集合会修改 `expectedCount` 的值，不会抛出异常。

示例：
```java
   List<String> list = new ArrayList<>();
   list.add("1");
   list.add("2");
   //正例
   Iterator<String> iterator = list.iterator();
   while(iterator.hasNext()){
      String item = iterator.next();
      if(删除元素的条件){
         iterator.remove();
      }
   }
   //反例
   for(String item : list){
      if("1".equals(item)){
         list.remove(item);
      }
   }
```
例如：HashMap，Vector，ArrayList,HashSet
## 安全失败（fail-safe）
采用 fail-safe 的集合容器，遍历时是先复制原有集合内容，在拷贝集合上进行遍历，因此，在遍历过程中对原集合所作的修改并不能被迭代器检测到，因此不会抛出异常。
例如：CopyOnWriteArrayList，ConcurrentHashMap
