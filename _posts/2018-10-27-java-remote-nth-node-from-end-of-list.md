---
layout: post
title:  Java代码实现——删除链表倒数第 n 个结点
date:   2018-10-27 17:15:50 +0800
categories: 算法
tag: [面试题, 删除链表倒数第 n 个结点]
---

* content
{:toc}



### Java代码实现：删除链表倒数第 n 个结点

问题描述：
给你一个单向链表，删除链表倒数第n个结点，然后返回head结点。这里的数字n是有效数字。

```
Given linked list: 1->2->3->4->5, and n = 2.

移除倒数第二个结点之后： 1->2->3->5.
```

方法一：先遍历获取链表长度，接着获取要移除的前一个元素，修改该元素的node.next --> node.next.next。
![](https://leetcode.com/media/original_images/19_Remove_nth_node_from_end_of_listA.png)

代码如下：

```
    /**
     * 移除链表倒数第n个结点，先遍历获取链表长度，接着获取要移除的前一个元素，修改该元素的node.next --> node.next.next。
     *
     * @param head
     * @param n
     * @return
     */
    public SingleNode removeNthFromEnd1(SingleNode head, int n) {
        SingleNode dummy = new SingleNode(0, head);

        //获取链表长度
        int length = 0;
        SingleNode first = head;
        while (first != null) {
            length++;
            first = first.next;
        }

        // 找到角标为(length - n - 1)的结点，让其next指向下下一个结点。
        first = head;
        int index = 0;
        while (index < length - n - 1) {
            first = first.next;
            index++;
        }
        first.next = first.next.next;
        return dummy.next;
    }

```

测试代码如下：

```
        SingleLinkedList sll = new SingleLinkedList();
        for (int i = 0; i < 5; i++) {
            sll.addLast(i + 1);
        }
        System.out.println(sll.toString());
        SingleNode node1 = removeNthFromEnd1(sll.getFirst(), 2);
        sll.logFromHead("removeNthFromEnd1", node1);

```

输出结果如下：符合预期

```
I/System.out: SingleLinkedList:[1, 2, 3, 4, 5]
I/System.out: removeNthFromEnd1:[1, 2, 3, 5]
```

感兴趣的同学可自行修改测试用例。

方法二：使用两个指针，两个指针保持固定间距n+1，接着开始遍历。当前面的指针指向null的时候，后面的那个指针刚好指向要移除的结点前一个，我们让其next指向其next.next即可。
![](https://leetcode.com/media/original_images/19_Remove_nth_node_from_end_of_listB.png)

```
    /**
     * 移除链表倒数第n个结点，使用两个指针实现。
     *
     * @param head
     * @param n
     * @return
     */
    public SingleNode removeNthFromEnd2(SingleNode head, int n) {
        SingleNode dummy = new SingleNode(0, head);
        SingleNode left = dummy;
        SingleNode right = dummy;
        for (int i = 0; i <= n; i++) {
            right = right.next;
        }
        while (right != null) {
            left = left.next;
            right = right.next;
        }
        left.next = left.next.next;
        return dummy.next;
    }
```

测试代码如下：

```
        SingleLinkedList sll = new SingleLinkedList();
        for (int i = 0; i < 5; i++) {
            sll.addLast(i + 1);
        }
        System.out.println(sll.toString());

        SingleNode node2 = removeNthFromEnd2(sll.getFirst(), 2);
        sll.logFromHead("removeNthFromEnd2", node2);
```

输出结果如下：跟预期一致。

```
    SingleLinkedList:[1, 2, 3, 4, 5]
    removeNthFromEnd2:[1, 2, 3, 5]
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。

github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：
1、[remove-nth-node-from-end-of-list](https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/)

2、[使用Java实现单向链表，并完成链表反转。](https://blog.csdn.net/qq_26287435/article/details/83421036)
