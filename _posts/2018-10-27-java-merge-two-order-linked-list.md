---
layout: post
title:  Java实现两个有序的链表合并
date:   2018-10-27 15:11:03 +0800
categories: 算法
tag: [面试题, 合并两个有序链表]
---

* content
{:toc}



### Java实现两个有序的链表合并

#### 先实现两个有序链表
代码如下：

```
        SingleLinkedList sll1 = new SingleLinkedList();
        for (int i = 0; i < 10; i += 2) {
            SingleNode node = new SingleNode(i, null);
            sll1.add(node);
        }
        System.out.println("单向链表1：" + sll1.toString());

        SingleLinkedList sll2 = new SingleLinkedList();
        for (int i = 1; i < 12; i += 2) {
            SingleNode node = new SingleNode(i, null);
            sll2.add(node);
        }
        System.out.println("单向链表2：" + sll2.toString());
```

输出：如下所示，跟预期一致。

```
    单向链表1：SingleLinkedList:[0, 2, 4, 6, 8]
    单向链表2：SingleLinkedList:[1, 3, 5, 7, 9, 11]
```

#### 嵌套调用，完成两个有序链表的合并。

```
    public SingleNode mergeTwoLists(SingleNode l1, SingleNode l2) {
        if (l1 == null) {
            return l2;
        }
        if (l2 == null) {
            return l1;
        }
        SingleNode mergedNode;
        if (l1.getItem() < l2.getItem()) {
            mergedNode = l1;
            mergedNode.next = mergeTwoLists(l1.next, l2);
        } else {
            mergedNode = l2;
            mergedNode.next = mergeTwoLists(l2.next, l1);
        }
        return mergedNode;
    }
```

下面进行测试，代码如下：

```
        SingleNode mergedNode = mergeTwoLists(sll1.getFirst(), sll2.getFirst());
        sll1.logFromHead("mergedNode", mergedNode);
```

输出结果如下：结果跟预期的一致。

```
    单向链表1：SingleLinkedList:[0, 2, 4, 6, 8]
    单向链表2：SingleLinkedList:[1, 3, 5, 7, 9, 11]
    mergedNode:[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 11]
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。

github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：

1、[merge-two-sorted-lists](https://leetcode.com/problems/merge-two-sorted-lists/description/)

2、[使用Java实现单向链表，并完成链表反转。](https://blog.csdn.net/qq_26287435/article/details/83421036)

3、[Java实现有环的单向链表，并判断单向链表是否有环](https://blog.csdn.net/qq_26287435/article/details/83445982)

