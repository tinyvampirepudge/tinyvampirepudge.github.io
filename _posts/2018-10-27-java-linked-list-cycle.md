---
layout: post
title:  Java实现有环的单向链表，并判断单向链表是否有环
date:   2018-10-27 14:23:49 +0800
categories: 算法
tag: [面试题,  单向链表,  有环的单向链表]
---

* content
{:toc}



### Java实现有环的单向链表，并判断单向链表是否有环
有一个单向链表，链表当中有可能出现环，就像下图这样。我们如何判断一个单向链表是否有环呢？
![有环的单向链表](http://jbcdn2.b0.upaiyun.com/2016/09/264a694e303e5c817dbc14956575fda9.jpg)

那么第一步，我们先实现一个这样的链表，接着再说如何判断这样的链表。

#### 实现有环的单向链表
1、定义add(Node node)方法

```
    /**
     * 向链表末尾添加结点
     *
     * @param node 结点的next指向为null，表示尾结点。
     * @return
     */
    public boolean add(SingleNode node) {
        if (first == null) {
            first = node;
        } else {
            SingleNode last = get(getSize() - 1);
            last.next = node;
        }
        size++;
        return true;
    }

    /**
     * 末尾添加一个特殊结点，形成一个环，为了测试用。
     *
     * @param node 这个node的next指向单项链表中已经存在的结点。
     * @return
     */
    public boolean addCycleNode(SingleNode node) {
        if (getSize() < 2) {
            return false;
        }
        SingleNode last = get(getSize() - 1);
        last.next = node;
        return true;
    }
```

2、先正常生成一个无环的单向链表，最后在末尾添加一个结点，让该结点的next指向前面的某个结点。

```
        // 循环链表
        SingleLinkedList sll = new SingleLinkedList();
        SingleNode head = new SingleNode(0, null);
        sll.add(head);
        for (int i = 1; i < 10; i++) {
            SingleNode node = new SingleNode(i, null);
            sll.add(node);
        }
        SingleNode sn = new SingleNode(111, null);
        sll.add(sn);
        for (int i = 10; i < 20; i++) {
            SingleNode node = new SingleNode(i, null);
            sll.add(node);
        }
        System.out.println(sll.toString());
```

输出如下：此时打印的还是一个无环单向链表。

```
SingleLinkedList:[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 111, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

接着添加最后一个结点，让其next指向之前的sn结点，这样就形成了一个闭环。

```
        // 最后一个结点的next指向之前添加的某个结点。
        SingleNode last = new SingleNode(222, sn);
        sll.add(last);

```

接着打印结果：需要注意的是，这里的打印log的方法我做了处理，具体细节请查看源码。

```
cycle:[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 111, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 222, 111]
```

最后插入的结点的值为222，它的next指向的是111结点，如上所示，在222之后打印的就是111节点了，如果继续打印，就会一直循环打印下去。

#### 判断单向链表是否有环

方法一：使用Hash Table
判断一个链表是否有环，我们只需要判断某个结点是否之前被访问过即可。这里我们优先想到的就是使用Hash表进行存储。
我们依次遍历访问每个结点，并将结点的引用存储到hash表中。如果当前结点是null，说明我们到达链表未尾部了，则当前链表一定无环；如果当前结点在hash表中已经存储过了，此时返回true即可。代码如下：

```
    /**
     * 判断单链表中是否有环
     * 借助HashSet来判断。
     *
     * @param head
     * @return
     */
    public boolean hasCycleByHashSet(SingleNode head) {
        Set<SingleNode> set = new HashSet<>();
        SingleNode node = head;
        while (node != null) {
            if (set.contains(node)) {
                return true;
            } else {
                set.add(node);
            }
            node = node.next;
        }
        return false;
    }
```

方法二：使用快慢指针
首先创建两个指针1和2(在Java里就是两个对象引用)，同时指向这个链表的头结点。然后开始一个大循环，在循环体中，1每次移动一个结点，2每次移动2个结点。最后比较这两个结点指向的结点是否相同。如果相同，则判断出链表有环，如果不同，则继续下一次循环。

代码如下：

```
    /**
     * 判断单链表中是否有环
     * 借助快慢指针来判断。
     *
     * @param head
     * @return
     */
    public boolean hasCycleByTowPointers(SingleNode head) {
        // 排除无数据或者只有一个数据且无闭环的情况
        if (head == null || head.next == null){
            return false;
        }
        SingleNode slow =  head;
        SingleNode fast = head.next;
        while(slow != fast){
            // 判断快指针结点是否为null，如果为null，则说明到达单向链表的结尾了。
            if(fast == null || fast.next == null){
                return false;
            }
            slow = slow.next;
            fast = fast.next.next;
        }
        return true;
    }
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。

github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：

1、[linked-list-cycle](https://leetcode.com/problems/linked-list-cycle/description/)

2、[漫画算法：如何判断链表有环？](http://blog.jobbole.com/106227/)

3、[使用Java实现单向链表，并完成链表反转。](https://blog.csdn.net/qq_26287435/article/details/83421036)