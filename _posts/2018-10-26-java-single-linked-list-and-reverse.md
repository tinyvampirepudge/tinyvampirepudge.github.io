---
layout: post
title:  使用Java实现单向链表，并完成链表反转。
date:   2018-10-26 21:30:07 +0800
categories: 数据结构
tag: [面试题, 单向链表, 链表反转]
---

* content
{:toc}




### 使用Java实现单向链表，并完成链表反转。

算法和数据结构是程序员逃不过的一个坎，所以趁着闲余时间，开始学习基础的算法和数据结构。这里记录下自己实现简单的单项链表的过程，如有错误，敬请指正。

#### 明确需求
在Java中，常用的数据容器里面，跟链表关系紧密的当属LinkedList了，它的底层实现为双向链表，这里就以它为参照物，实现自己的简单的单向链表。另外，还需要支持增删改查、获取大小等功能。
如下所示，先定义了增删改查方法支持的范围。
```
    /**
     * 增删改查方法
     * add(index, E)        index >= 0 && index <= size
     * remove(index)        index >= 0 && index < size
     * set(index, E)        index >= 0 && index < size
     * get(index)           index >= 0 && index < size
     */
```

#### 定义链表结点Node
结点的定义很简单，这里为了简单起见，不支持泛型，结点存储的数据类型固定为int。
```
    private static class Node {
        private int item;
        private Node next;

        public Node(int item, Node next) {
            this.item = item;
            this.next = next;
        }

        public void setItem(int item) {
            this.item = item;
        }

        @Override
        public String toString() {
            return String.valueOf(this.item);
        }
    }
```

#### 定义辅助方法
既然是单向链表，容器中只需要持有first引用即可，不需要last引用。再需要定义一个size用来表示链表大小；另外，为了方便查看测试结果，我们重写toString方法。代码如下：
```
public class SingleLinkedList {

    private int size = 0;
    private Node first = null;
    ...
}
```
```
    /**
     * 返回当前容器大小。
     * @return
     */
    public int getSize() {
        return size;
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append('[');
        if (first != null) {
            Node curr = first;
            while (curr != null) {
                sb.append(String.valueOf(curr));
                if (curr.next != null) {
                    sb.append(",").append(" ");
                }
                curr = curr.next;
            }
        }
        sb.append("]");
        return sb.toString();
    }
```

#### 定义查询方法get(index)
在已有数据中获取数据，这个最为简单，由于单向链表容器中只存储首个结点first引用，所以不能直接角标index获取，需要从first中根据角标依次遍历，代码如下：
```
    /**
     * 获取角标上的数据。
     *
     * @param index
     * @return
     */
    public Node get(int index) {
        checkElementIndex(index);

        Node x = first;
        for (int i = 0; i < index; i++) {
            x = x.next;
        }
        return x;
    }
```

#### 定义增加方法add(index, value)
与另外三组方法（获取、修改、删除）均不同，增加方法不仅仅要支持链表现有的数据范围，而且还需要支持角标为size的位置。
在当前位置插入结点，需要获取到前一个结点，让前一个结点的next指向新增的结点，再让新增结点的next指向该位置原来的结点。需要注意一些特殊情况的处理，还有不要忘了size的数值增加。代码如下：
```
    /**
     * 在角标为index的位置插入数据
     *
     * @param index
     * @param value
     * @return
     */
    public boolean add(int index, int value) {
        checkPositionIndex(index);

        if (index == 0) {
            if (first == null) {
                first = new Node(value, null);
            } else {
                Node next = get(0);
                Node node = new Node(value, next);
                first = node;
            }
        } else {
            Node prev = get(index - 1);
            Node node = new Node(value, prev.next);
            prev.next = node;
        }
        size++;
        return true;
    }
```

#### 定义修改方法set(index, value)
修改方法也很简单，只需根据角标获取已经存在结点，然后将结点中存储的数据修改即可，代码如下：
```
    public boolean set(int index, int value) {
        checkElementIndex(index);
        Node node = get(index);
        node.setItem(value);
        return true;
    }
```

#### 定义删除方法remove(index)
删除当前位置的结点，核心是让上一个结点的next指向当前位置的下一个结点即可，与添加方法类似，需要做好极端情况的处理，代码如下：
```
    public boolean remove(int index) {
        checkElementIndex(index);

        // size > 0
        if (getSize() == 1) {//size == 1
            first = null;
        } else {// size > 1
            if (index == 0) {// 删除第一个数据
                first = first.next;
            } else if (getSize() - 1 == index) {// 删除最后一个数据
                Node prev = get(index - 1);
                prev.next = null;
            } else {// 删除中间的数据
                get(index - 1).next = get(index).next;
            }
        }
        size--;
        return true;
    }
```

#### 实现链表反转
为了实现单向链表的反转，在遍历的过程中，让每个节点的next指向上一个结点。代码如下：
```
    /**
     * 反转链表
     *
     * @return
     */
    public Node reverseList() {
        Node prev = null;
        Node curr = first;
        while (curr != null) {
            Node temp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = temp;
        }
        logFromHead("reverseList", prev);
        return prev;
    }
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。
github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

#### 参考
[https://leetcode.com/problems/reverse-linked-list/description/](https://leetcode.com/problems/reverse-linked-list/description/)

