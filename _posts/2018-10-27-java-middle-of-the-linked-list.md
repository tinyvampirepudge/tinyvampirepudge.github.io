---
layout: post
title:  求单向链表的中间结点
date:   2018-10-27 17:57:38 +0800
categories: 算法
tag: [面试题, 单向链表的中间结点]
---

* content
{:toc}



### 求单向链表的中间结点

#### 需求
非空的单向链表，返回其中间节点。如果有两个中间结点，返回第二个。
链表大小控制在1~100之间。

示例1：
```
Input: [1,2,3,4,5]
Output: Node 3 from this list (Serialization: [3,4,5])
```
示例2：
```
Input: [1,2,3,4,5,6]
Output: Node 4 from this list (Serialization: [4,5,6])
```

#### 方法一：使用数组实现
循环遍历链表，统计链表长度，同时将每个结点存储数组中，最后返回数组中角标为(链表长度/2)的结点即可。
```
/**
* 求单向链表的中间结点，使用数组实现
* @param head
* @return
*/
public SingleNode middleNodeByArrays(SingleNode head) {
SingleNode[] arrays = new SingleNode[100];
int count = 0;
while (head != null) {
arrays[count] = head;
count++;
head = head.next;
}
return arrays[count / 2];
}
```
测试代码：
```
// 求单向链表的中间结点
SingleLinkedList sll1 = new SingleLinkedList();
for (int i = 1; i < 10; i++) {
SingleNode node = new SingleNode(i, null);
sll1.add(node);
}
System.out.println(sll1.toString());

SingleNode node1 = middleNodeByArrays(sll1.getFirst());
sll1.logFromHead("middleNodeByArrays1", node1);

SingleLinkedList sll2 = new SingleLinkedList();
for (int i = 1; i < 11; i++) {
SingleNode node = new SingleNode(i, null);
sll2.add(node);
}
System.out.println(sll2.toString());

SingleNode node2 = middleNodeByArrays(sll2.getFirst());
sll2.logFromHead("middleNodeByArrays2", node2);
```
输出结果：符合预期。
```
SingleLinkedList:[1, 2, 3, 4, 5, 6, 7, 8, 9]
middleNodeByArrays1:[5, 6, 7, 8, 9]
SingleLinkedList:[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
middleNodeByArrays2:[6, 7, 8, 9, 10]

```

#### 方法二：使用快慢指针
定义两个指针slow和fast，slow每次移动一步，fast每次移动两步，这样当fast尾结点的时候，slow一定位于链表的中间结点上。
代码如下：
```
/**
* 求单向链表的中间结点，使用快慢指针实现
* @param head
* @return
*/
public SingleNode middleNodeByTwoPointers(SingleNode head) {
SingleNode slow = head;
SingleNode fast = head;

while(fast != null && fast.next != null){
slow = slow.next;
fast = fast.next.next;
}
return slow;
}
```
测试代码如下：
```
// 求单向链表的中间结点
SingleLinkedList sll1 = new SingleLinkedList();
for (int i = 1; i < 10; i++) {
SingleNode node = new SingleNode(i, null);
sll1.add(node);
}
System.out.println(sll1.toString());

SingleNode node1 = middleNodeByTwoPointers(sll1.getFirst());
sll1.logFromHead("middleNodeByTwoPointers1", node1);

SingleLinkedList sll2 = new SingleLinkedList();
for (int i = 1; i < 11; i++) {
SingleNode node = new SingleNode(i, null);
sll2.add(node);
}
System.out.println(sll2.toString());
SingleNode node2 = middleNodeByTwoPointers(sll2.getFirst());
sll2.logFromHead("middleNodeByTwoPointers2", node2);
```
输出结果如下：符合预期
```
SingleLinkedList:[1, 2, 3, 4, 5, 6, 7, 8, 9]
middleNodeByTwoPointers1:[5, 6, 7, 8, 9]
SingleLinkedList:[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
middleNodeByTwoPointers2:[6, 7, 8, 9, 10]
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。
github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：
1、[middle-of-the-linked-list](https://leetcode.com/problems/middle-of-the-linked-list/description/)

2、[使用Java实现单向链表，并完成链表反转。](https://blog.csdn.net/qq_26287435/article/details/83421036)

