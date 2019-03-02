---
layout: post
title:  HashMap 1.8 源码简读
date:   2018-12-14 15:26:01 +0800
categories: Java
tag: [HashMap]
---

* content
{:toc}



### HashMap 1.8 源码简读

#### 初始大小:默认为16

```
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

如果指定了大小，则会在初始化的时候将初始容量大小设置为大于等于指定初始值、且最小的2的整数次幂。

原理：
①在`HashMap`的构造方法`HashMap(int initialCapacity, float loadFactor)`或`HashMap(int initialCapacity)`中，会通过`tableSizeFor`方法将`threshold`的值置为2的整数次幂。
②在第一次调用`put(K key, V value)`方法添加元素时，会调用`resize()`方法来初始化`table`，而在`resize()`方法中，此时初始容量会被`threshold`的值代替。

关键代码如下所示：

```
public HashMap(int initialCapacity, float loadFactor) {
    ...
    this.threshold = tableSizeFor(initialCapacity);
}
```

请看resize()方法中的这行注释：initial capacity was placed in threshold

```
final Node<K,V>[] resize() {
    ...
    if (oldCap > 0) {
        ...
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        ...
    }
    ...
}
```

如上所示，HashMap在指定了初始化容量之后，初始容器的大小会变为大于等于指定初始值、且最小的2的整数次幂。

tableSizeFor方法如下所示：

```
/**
 * Returns a power of two size for the given target capacity.
 */
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

返回大于等于输入参数且最近的2的整数次幂的数.
测试结果如下：

```
tableSizeFor(0):1
tableSizeFor(1):1
tableSizeFor(2):2
tableSizeFor(3):4
tableSizeFor(4):4
tableSizeFor(5):8
tableSizeFor(6):8
tableSizeFor(7):8
tableSizeFor(8):8
tableSizeFor(9):16
```

#### 装载因子和动态扩容：
装载因子：默认为0.75，可在HashMap初始化时设置。

```
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

扩容规则是新的容量是原来容量的两倍。
HashMap中table数组的初始化和动态扩容的方法是resize()，关于扩容容量的代码如下：
注意这行代码：`newCap = oldCap << 1`

```
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
final Node<K,V>[] resize() {
    ...
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        ...
    else { // zero initial threshold signifies using defaults
        ...
    }
    ...
}
```

##### 触发扩容操作的阈值：threshold
计算规则：`threshold = capacity * loadFactor`
变量定义如下：

```
/**
 * The next size value at which to resize (capacity * load factor).
 *
 * @serial
 */
// (The javadoc description is true upon serialization.
// Additionally, if the table array has not been allocated, this
// field holds the initial array capacity, or zero signifying
// DEFAULT_INITIAL_CAPACITY.)
int threshold;
```

扩容触发条件：数据数量大于阈值时。
代码在resize()方法中：

```
if (++size > threshold)
    resize();
```

触发扩容操作以后，会在resize()方法中进行数据迁移。是一次性数据迁移，细节请看源码。

#### 散列冲突解决方法：链表法加红黑树
链表转化成红黑树，红黑树转化成链表。
1、链表转化成红黑树：
HashMap#putVal方法中，当在该角标下，原有存储的链表结点个数大于等于7的时候，就将链表转化为红黑树。具体方法代码如下：

```
if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
    treeifyBin(tab, hash);
```

具体的转换过程请看`treeifyBin`方法。

2、红黑树转化成链表：
当冲突以红黑树形态情况下，进行扩充时，将树转成两棵树，若树的的结点数小于等于UNTREEIFY_THRESHOLD，则转为链表形式。
细节请看`split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit)`方法。

#### HashMap中的散列函数
hash方法如下：

```
/**
 * Computes key.hashCode() and spreads (XORs) higher bits of hash
 * to lower.  Because the table uses power-of-two masking, sets of
 * hashes that vary only in bits above the current mask will
 * always collide. (Among known examples are sets of Float keys
 * holding consecutive whole numbers in small tables.)  So we
 * apply a transform that spreads the impact of higher bits
 * downward. There is a tradeoff between speed, utility, and
 * quality of bit-spreading. Because many common sets of hashes
 * are already reasonably distributed (so don't benefit from
 * spreading), and because we use trees to handle large sets of
 * collisions in bins, we just XOR some shifted bits in the
 * cheapest possible way to reduce systematic lossage, as well as
 * to incorporate impact of the highest bits that would otherwise
 * never be used in index calculations because of table bounds.
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

在插入或查找的时候，计算key被映射到的位置

```
int index = hash(key) & (capacity - 1)
```

JDK HashMap中hash函数的设计，确实很巧妙：

首先hashcode本身是个32位整型值，这个值对于不同的对象必须保证唯一（Java规范），这也是大家常说的，重写equals必须重写hashcode的重要原因。

获取对象的hashcode以后，先进行移位运算，然后再和自己做异或运算，及hashcode ^ (hashcode >>> 16)，这一步甚是巧妙，是将高16位移到低16位，这样计算出来的整型值将“具有”高位和低位的性质。

最后，用hash表当前的容量减去一，再和刚刚计算出来的整型值做位与运算。进行位与运算，很好理解，是为了计算出数组中的位置。但这里有个问题：`为什么要用容量减去一？`
因为 A % B = A & (B - 1)，所以`(h ^ (h >>> 16)) & (capitity -1) = (h ^ (h >>> 16)) % capitity`，可以看出这里本质上是使用了[除留余数法]。

综上，可以看出，hashcode的随机性，加上移位异或运算，得到一个非常随机的hash值，再通过[除留余数法]，得到index，整体的设计过程和“散列函数”设计原则非常吻合。

#### 参考：
[https://time.geekbang.org/column/article/64586 的第一条评论](https://time.geekbang.org/column/article/64586)

[Java8 HashMap之tableSizeFor](https://www.cnblogs.com/loading4/p/6239441.html)

[散列表——维基百科](https://zh.wikipedia.org/wiki/%E5%93%88%E5%B8%8C%E8%A1%A8)

[Java8-HashMap 详解](http://www.manuu.vip/2017/05/21/Map-HashMap/)