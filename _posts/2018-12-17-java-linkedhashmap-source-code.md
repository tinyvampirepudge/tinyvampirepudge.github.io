---
layout: post
title:  LinkedHashMap源码简读
date:   2018-12-17 18:50:16 +0800
categories: Java
tag: [LinkedHashMap]
---

* content
{:toc}



### LinkedHashMap源码简读

1、LinkedHashMap继承自HashMap，HashMap具有的特性它都具有。

2、实际上，LinkedHashMap是通过双向链表和散列表这两种数据组合实现的。LinkedHashMap中的“Linked”实际上指的是双向链表，并非指“用链表法解决散列冲突”。

3、LinkedHashMap不仅支持按照插入顺序遍历数据，还支持按照访问顺序来遍历数据。通过设置`accessOrder`属性为true即可。也就是说它本身就是一个支持LRU缓存淘汰策略的缓存系统。

#### 查询、插入、删除方法。
LinkedHashMap继承自HashMap，而HashMap是用散列表实现的，所以LinkedHashMap在HashMap的基础上增加了双向链表来存储数据。那么是在哪儿添加的呢？我们对get、put、remove三个方法逐个分析。

首先LinkedHashMap中关于双向链表的变量定义：
```
/**
* The head (eldest) of the doubly linked list.
*/
transient LinkedHashMapEntry<K,V> head;

/**
* The tail (youngest) of the doubly linked list.
*/
transient LinkedHashMapEntry<K,V> tail;
```

##### LinkedHashMap#get(Object key)方法
```
public V get(Object key) {
Node<K,V> e;
if ((e = getNode(hash(key), key)) == null)
return null;
if (accessOrder)
afterNodeAccess(e);
return e.value;
}
```
LinkedHashMap重写了HashMap中的get(Object key)方法，主要就是增加了accessOrder相关操作。核心的查找操作还是通过HashMap中的方法去进行。

##### LinkedHashMap#put(K key, V value)方法
LinkedHashMap完全继承了HasMap的get(K key, V value)方法，没有做任何修改，那么是它是如何将新增的数据也添加进双向链表中呢？

HashMap中，put方法调用了putVal方法，代码如下：
```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
boolean evict) {
Node<K,V>[] tab; Node<K,V> p; int n, i;
if ((tab = table) == null || (n = tab.length) == 0)
n = (tab = resize()).length;
if ((p = tab[i = (n - 1) & hash]) == null)
// 注释一
tab[i] = newNode(hash, key, value, null);
else {
Node<K,V> e; K k;
if (p.hash == hash &&
((k = p.key) == key || (key != null && key.equals(k))))
e = p;
else if (p instanceof TreeNode)
e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
else {
for (int binCount = 0; ; ++binCount) {
if ((e = p.next) == null) {
// 注释二
p.next = newNode(hash, key, value, null);
if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
treeifyBin(tab, hash);
break;
}
if (e.hash == hash &&
((k = e.key) == key || (key != null && key.equals(k))))
break;
p = e;
}
}
if (e != null) { // existing mapping for key
V oldValue = e.value;
if (!onlyIfAbsent || oldValue == null)
e.value = value;
// 注释三
afterNodeAccess(e);
return oldValue;
}
}
++modCount;
if (++size > threshold)
resize();
afterNodeInsertion(evict);
return null;
}
```
如上所示，HashMap在插入数据时，会先去查找该数据是否已存在。如果不存在，就调用`newNode`方法来创建新元素；如果存在，就进行相应的处理，同时调用`afterNodeAccess`方法。需要注意的是`newNode`和`afterNodeAccess`均已被LinkedHashMap重写，做了额外的工作，来讲数据添加进`双向链表`里面。

下面针对细节进行解释：
注释一：
这里新生成了一个node，然后将其插入到数组中。
另外newNode方法被LinkedHashMap重写了，它在new的同时，还将node添加进了双向链表。
LinkedHashMap#newNode():
```
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
LinkedHashMapEntry<K,V> p =
new LinkedHashMapEntry<K,V>(hash, key, value, e);
linkNodeLast(p);
return p;
}
```
LinkedHashMap#linkNodeLast():将node链接到双向链表尾部。
```
// link at the end of list
private void linkNodeLast(LinkedHashMapEntry<K,V> p) {
LinkedHashMapEntry<K,V> last = tail;
tail = p;
if (last == null)
head = p;
else {
p.before = last;
last.after = p;
}
}
```

注释二：操作同上。

注释三：
先看下HashMap中afterNodeAccess方法的定义，注释写的很明白，下面三个方法是专门让LinkedHashMap去重写的。
```
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```
那我们来看下LinkedHashMap中的afterNodeAccess方法：将node移动到链表结尾。
```
void afterNodeAccess(Node<K,V> e) { // move node to last
LinkedHashMapEntry<K,V> last;
if (accessOrder && (last = tail) != e) {
LinkedHashMapEntry<K,V> p =
(LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after;
p.after = null;
if (b == null)
head = a;
else
b.after = a;
if (a != null)
a.before = b;
else
last = b;
if (last == null)
head = p;
else {
p.before = last;
last.after = p;
}
tail = p;
++modCount;
}
}
```

##### LinkedHashMap#remove(Object key)方法
LinkedHashMap也是继承自
```
public V remove(Object key) {
Node<K,V> e;
return (e = removeNode(hash(key), key, null, false, true)) == null ?
null : e.value;
}
```
LinkedHashMap#removeNode方法：
```
final Node<K,V> removeNode(int hash, Object key, Object value,
boolean matchValue, boolean movable) {
Node<K,V>[] tab; Node<K,V> p; int n, index;
if ((tab = table) != null && (n = tab.length) > 0 &&
...
if (node != null && (!matchValue || (v = node.value) == value ||
(value != null && value.equals(v)))) {
...
afterNodeRemoval(node);
return node;
}
}
return null;
}
```

可以看到，删除时调用了afterNodeRemoval方法，这个方法在HashMap中是空实现，在LinkedHashMap中被重写。
LinkedHashMap#afterNodeRemoval方法：将node元素从双向链表中移除。
```
void afterNodeRemoval(Node<K,V> e) { // unlink
LinkedHashMapEntry<K,V> p =
(LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after;
p.before = p.after = null;
if (b == null)
head = a;
else
b.after = a;
if (a == null)
tail = b;
else
a.before = b;
}
```

#### 总结：
HashMap底层是用数组来实现的，利用散列表我们可以实现快速访问、存取元素，但是元素的顺序得不到保障。
LinkedHashMap在HashMap的基础上，将元素添加进了双向链表中，所以可以在快速访问、存取元素的同时，还可以保证元素的顺序。
另外，LinkedHashMap天生就是一个支持LRU缓存淘汰策略的缓存系统。

参考：
[https://time.geekbang.org/column/article/64858](https://time.geekbang.org/column/article/64858)

