# JAVA数据结构

## Map

### 源码解析

[史上最详细的 JDK 1.8 HashMap 源码解析](https://joonwhee.blog.csdn.net/article/details/78996181)

### 基本信息

```java

// 默认容量16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
 
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;    
// 因为最大的正n次方是 0100000.... 
 
// 默认负载因子0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f; 
 
// 链表节点转换红黑树节点的阈值, 9个节点转
static final int TREEIFY_THRESHOLD = 8; 
 
// 红黑树节点转换链表节点的阈值, 6个节点转
static final int UNTREEIFY_THRESHOLD = 6;   
 
// 转红黑树时, table的最小长度
static final int MIN_TREEIFY_CAPACITY = 64; 
 
// 链表节点, 继承自Entry
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
 
    // ... ...
}
 
// 红黑树节点
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
   
    // ...
}
```

### 定位桶

```java
static final int hash(Object key) { // 计算key的hash值
    int h;
    // 1.先拿到key的hashCode值; 2.将hashCode的高16位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// 代码2
int n = tab.length;
// 将(tab.length - 1) 与 hash值进行&运算
int index = (n - 1) & hash;
```

1. 拿到 key 的 hashCode 值
2. 将 hashCode 的高位参与运算，重新计算 hash 值
3. 将计算出来的 hash 值与 (table.length - 1) 进行 & 运算

在 JDK1.8 的实现中，还优化了高位运算的算法，将 hashCode 的高 16 位与 hashCode 进行异或运算，主要是为了在 table 的 length 较小的时候，让高位也参与运算，并且不会有太大的开销。

### Get

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
 
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 1.对table进行校验：table不为空 && table长度大于0 && 
    // table索引位置(使用table.length - 1和hash值进行位与运算)的节点不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 2.检查first节点的hash值和key是否和入参的一样，如果一样则first即为目标节点，直接返回first节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 3.如果first不是目标节点，并且first的next节点不为空则继续遍历
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                // 4.如果是红黑树节点，则调用红黑树的查找目标节点方法getTreeNode
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // 5.执行链表节点的查找，向下遍历链表, 直至找到节点的key和入参的key相等时,返回该节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    // 6.找不到符合的返回空
    return null;
}
```

如果是红黑树节点，find方法如下

```java
final TreeNode<K,V> getTreeNode(int h, Object k) {
    // 1.首先找到红黑树的根节点；2.使用根节点调用find方法
    return ((parent != null) ? root() : this).find(h, k, null);
}

/**
 * 从调用此方法的节点开始查找, 通过hash值和key找到对应的节点
 * 此方法是红黑树节点的查找, 红黑树是特殊的自平衡二叉查找树
 * 平衡二叉查找树的特点：左节点<根节点<右节点
 */
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    // 1.将p节点赋值为调用此方法的节点，即为红黑树根节点
    TreeNode<K,V> p = this;
    // 2.从p节点开始向下遍历
    do {
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;
        // 3.如果传入的hash值小于p节点的hash值，则往p节点的左边遍历
        if ((ph = p.hash) > h)
            p = pl;
        else if (ph < h) // 4.如果传入的hash值大于p节点的hash值，则往p节点的右边遍历
            p = pr;
        // 5.如果传入的hash值和key值等于p节点的hash值和key值,则p节点为目标节点,返回p节点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if (pl == null)    // 6.p节点的左节点为空则将向右遍历
            p = pr;
        else if (pr == null)    // 7.p节点的右节点为空则向左遍历
            p = pl;
        // 8.将p节点与k进行比较
        else if ((kc != null ||
                  (kc = comparableClassFor(k)) != null) && // 8.1 kc不为空代表k实现了Comparable
                 (dir = compareComparables(kc, k, pk)) != 0)// 8.2 k<pk则dir<0, k>pk则dir>0
            // 8.3 k<pk则向左遍历(p赋值为p的左节点), 否则向右遍历
            p = (dir < 0) ? pl : pr;
        // 9.代码走到此处, 代表key所属类没有实现Comparable, 直接指定向p的右边遍历
        else if ((q = pr.find(h, k, kc)) != null) 
            return q;
        // 10.代码走到此处代表“pr.find(h, k, kc)”为空, 因此直接向左遍历
        else
            p = pl;
    } while (p != null);
    return null;
}
```

### Put

put方法内部调用了putVal

1. 如果定位到的数组位置没有元素 就直接插入。
2. 如果定位到的数组位置有元素就和要插入的 key 比较，如果 key 相同就直接覆盖，如果 key 不相同，就判断 p 是否是一个树节点，如果是就调用`e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)`将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

![image-20210313202705172](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210313202705172.png)

> - 直接覆盖之后应该就会 return，不会有后续操作。参考 JDK8 HashMap.java 658 行（[issue#608](https://github.com/Snailclimb/JavaGuide/issues/608)）。
> - 当链表长度大于阈值（默认为 8）并且 HashMap 数组长度超过 64 的时候才会执行链表转红黑树的操作，否则就只是对数组扩容。参考 HashMap 的 `treeifyBin()` 方法（[issue#1087](https://github.com/Snailclimb/JavaGuide/issues/1087)）。

### resize

1. 判断原本哈希表的size，如果是0，就说明刚刚初始化，用默认值来初始化。否则判断是否超过最大容量，如果是的话直接返回老表，否则设置新表容量为老表两倍(<<1)，新的阈值为原来两倍

2. 转移节点。如果老表头.next==NULL，说明只有一个节点，直接转移即可。否则判断是红黑树还是链表，分别进行rehash计算

	值得一提的是，这里用了一个很巧妙的操作进行重定位，节点重 hash 为什么只可能分布在 “原索引位置” 与 “原索引 + oldCap 位置”。

	所以只需要把旧节点的hash值与旧容量进行与运算，如果是0那么位置不变，如果不为0，那么新位置为“原索引 + oldCap 位置”

	![image-20210313203711448](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210313203711448.png)

### JDK1.7和JDK1.8下的区别

1. 最重要的一点是底层结构不一样，1.7是数组+链表，1.8则是数组+链表+红黑树结构;

2. jdk1.7中当哈希表为空时，会先调用inflateTable()初始化一个数组；而1.8则是直接调用resize()扩容;

3. 插入键值对的put方法的区别，1.8中会将节点插入到链表尾部，而1.7中是采用头插；

4. 扩容时1.8会保持原链表的顺序，而1.7会颠倒链表的顺序；而且1.8是在元素插入后检测是否需要扩容，1.7则是在元素插入前；

5. 计算 table 初始容量的方式发生了改变，老的方式是从1开始不断向左进行移位运算，直到找到大于等于入参容量的值；新的方式则是通过“5个移位+或等于运算”来计算。

  ```java
  
  // JDK 1.7.0
  public HashMap(int initialCapacity, float loadFactor) {
      // 省略
      // Find a power of 2 >= initialCapacity
      int capacity = 1;
      while (capacity < initialCapacity)
          capacity <<= 1;
      // ... 省略
  }
  // JDK 1.8.0_191
  static final int tableSizeFor(int cap) {
      int n = cap - 1;
      n |= n >>> 1;
      n |= n >>> 2;
      n |= n >>> 4;
      n |= n >>> 8;
      n |= n >>> 16;
      return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
  }
  ```

6. 优化了 hash 值的计算方式，老的通过一顿瞎JB操作，新的只是简单的让高16位参与了运算。

	```java
	// JDK 1.7.0
	static int hash(int h) {
	    h ^= (h >>> 20) ^ (h >>> 12);
	    return h ^ (h >>> 7) ^ (h >>> 4);
	}
	// JDK 1.8.0_191
	static final int hash(Object key) {
	    int h;
	    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
	}
	```

### 为什么说是线程不安全的？

HashMap在并发场景下可能存在以下问题：

1. 死循环
2. 数据丢失

#### 死循环

关于死循环的问题，在Java8中个人认为是不存在了，在Java8之前的版本中之所以出现死循环是因为在resize的过程中对链表进行了倒序处理；在Java8中不再倒序处理，自然也不会出现死循环。

见https://joonwhee.blog.csdn.net/article/details/106324537

简单来说，jdk1.7的时候，底层是链表实现的，rehash的时候采用了头插法，多线程并发下，可能产生循环链表，因为rehash的时候终止条件是node.next==null，所以会死循环

在jdk1.8中，用了loHead/loTai和hiHead/hiTail，保证尾插法

#### 数据丢失

比如jdk1.8，比如在putVal的时候，两个线程都进这个分支的话会数据覆盖，从而造成数据丢失

```java
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```



### 总结

[面试阿里，HashMap 这一篇就够了](https://joonwhee.blog.csdn.net/article/details/106324537)

插入流程图

![image-20210313214535625](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210313214535625.png)

resize流程图

![image-20210313214608168](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210313214608168.png)

## ConcurrentHashMap

### 1.7

* 概述

ConcurrentHashMap 和 HashMap 思路是差不多的，但是因为它支持并发操作，所以要复杂一些。

整个 ConcurrentHashMap 由一个个 Segment 组成，Segment 代表”部分“或”一段“的意思，所以很多地方都会将其描述为**分段锁**。注意，行文中，我很多地方用了“**槽**”来代表一个 segment。

简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

* 初始化

initialCapacity：初始容量，这个值指的是整个 ConcurrentHashMap 的初始容量，实际操作的时候需要平均分给每个 Segment。

loadFactor：负载因子，之前我们说了，Segment 数组不可以扩容，所以这个负载因子是给每个 Segment 内部使用的。

* put

	同map1.7，只是put的时候要会调用 node = tryLock() ? null : scanAndLockForPut(key, hash, value)，也就是说先进行一次 tryLock() 快速获取该 segment 的独占锁，如果失败，那么进入到 scanAndLockForPut 这个方法来获取锁

* 扩容

	segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry\<K,V>[] 进行扩容，扩容后，容量为原来的 2 倍。

	首先，我们要回顾一下触发扩容的地方，put 的时候，如果判断该值的插入会导致该 segment 的元素个数超过阈值，那么先进行扩容，再插值，读者这个时候可以回去 put 方法看一眼。

	该方法不需要考虑并发，因为到这里的时候，是持有该 segment 的独占锁的。

### 1.8

结构上和 Java8 的 HashMap 基本上一样，不过它要保证线程安全性，所以在源码上确实要复杂一些

#### 扩容

https://blog.csdn.net/ZOKEKAI/article/details/90051567

1、如果新增节点之后，所在链表的元素个数达到了阈值 8，则会调用`treeifyBin`方法把链表转换成红黑树，不过在结构转换之前，会对数组长度进行判断，实现如下：

如果数组长度n小于阈值`MIN_TREEIFY_CAPACITY`，默认是64，则会调用`tryPresize`方法把数组长度扩大到原来的两倍，并触发`transfer`方法，重新调整节点的位置。

2、新增节点之后，会调用`addCount`方法记录元素个数，并检查是否需要进行扩容，当数组元素个数达到阈值时，会触发`transfer`方法，重新调整节点的位置。

扩容过程：

1. 根据当前数组长度n，新建一个两倍长度的数组`nextTable`；
2. 计算一个stride， 计算每条溴铵从处理的桶个数，如果小于16的话就设为16
3. 初始化`ForwardingNode`节点，其中保存了新数组`nextTable`的引用，在处理完每个槽位的节点之后当做占位节点，表示该槽位已经处理过了；
4. 循环扩容，一个槽一个槽处理，如果遇到某个槽中节点的hash为MOVE，也就是`ForwardingNode`，说明被其他线程处理过了，跳过，否则synchronized加锁扩容

### 总结

Java7 中 ConcruuentHashMap 使用的分段锁，也就是每一个 Segment 上同时只有一个线程可以操作，每一个 Segment 都是一个类似 HashMap 数组的结构，它可以扩容，它的冲突会转化为链表。但是 Segment 的个数一但初始化就不能改变。

Java8 中的 ConcruuentHashMap 使用的 Synchronized 锁加 CAS 的机制。结构也由 Java7 中的 **Segment 数组 + HashEntry 数组 + 链表** 进化成了 **Node 数组 + 链表 / 红黑树**，Node 是类似于一个 HashEntry 的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表。

## List

### ArrayList

[Java集合：ArrayList详解](https://joonwhee.blog.csdn.net/article/details/79190114)

主要注意扩容是1.5倍，默认容量`DEFAULT_CAPACITY = 10`，扩容用到了`Arrays.copyOf`

### LinkedList

[Java集合：LinkedList详解](https://joonwhee.blog.csdn.net/article/details/79247389)

双向链表

## String

### String 

字符串广泛应用 在Java 编程中，在 Java 中字符串属于对象，Java 提供了 String 类来创建和操作字符串。

需要注意的是，String的值是不可变的，这就导致每次对String的操作都会生成**新的String对象**，这样不仅效率低下，而且大量浪费有限的内存空间。我们来看一下这张对String操作时内存变化的图：

![image-20210323093104405](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210323093104405.png)

https://blog.csdn.net/zp357252539/article/details/97916254

注意点:

* 这种直接通过双引号""声明字符串的方式, 虚拟机首先会到字符串常量池中查找该字符串是否已经存在. 如果存在会直接返回该引用, 如果不存在则会在堆内存中创建该字符串对象, 然后到字符串常量池中注册该字符串

* 用new关键字创建字符串对象的时候, JVM将不会查询字符串常量池, 它将会直接在堆内存中创建一个字符串对象, 并返回给所属变量。

* String类的源码中有对 intern()方法的详细介绍, 翻译过来的意思是: 当调用 intern()方法时, 首先会去常量池中查找是否有该字符串对应的引用, 如果有就直接返回该字符串; 如果没有, 就会在常量池中注册该字符串的引用, 然后返回该字符串。

* 两个String对象相加的原理：创建一个StringBuilder对象，相加，然后调用toString，![image-20210323094609999](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210323094609999.png)

	所以不会在字符串常量池中创建！

### StringBuilder

对于String, 凡是涉及到返回参数类型为String类型的方法, 在返回的时候都会通过new关键字创建一个新的字符串对象; 而对于StringBuilder, 大多数方法都会返回StringBuilder对象自身。

内部***可变数组\***，存在初始化StringBuilder对象中***字符数组容量为16，存在扩容\***。

以String为参数进行构造，在参数Str 数组长度的基础上再增加16个字符长度，作为StringBuilder实例的初始数组容量，并将str字符串 append到StringBuilder的数组中。

### StringBuffer

基本上与StringBuilder一致，但其为线程安全的字符串操作类，大部分方法都采用了Synchronized关键字修改，以此来实现在多线程下的操作字符串的安全性。