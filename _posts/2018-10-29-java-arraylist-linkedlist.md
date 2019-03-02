---
layout: post
title:  Java实现队列——顺序队列、链式队列
date:   2018-10-29 17:17:23 +0800
categories: 数据结构
tag: [数据队列,  链式队列]
---

* content
{:toc}



### Java实现队列——顺序队列、链式队列
#### 概念
先进者先出，这就是典型的“队列”。(First In, First Out，FIFO)。

我们知道，栈只支持两个基本操作：入栈push()和出栈pop()。队列跟栈非常相似，支持的操作也很有限，最基本的操作也是两个：入队和出队。入队 `enqueue()`，让一个数据到队列尾部；出队 `dequeue()`，从队列头部取一个元素。

![栈和队列](https://static001.geekbang.org/resource/image/9e/3e/9eca53f9b557b1213c5d94b94e9dce3e.jpg)

所以，队列跟栈一样，也是一种操作受限的线性表数据结构。

跟栈类似，用数组实现的队列叫做顺序队列，用链表实现的队列叫做链式队列。下面我们看下如何用Java代码如何实现。

#### 顺序队列
代码如下：

```
public class QueueBasedArray implements QueueInterface {
    private String[] values;// 数组
    private int capacity = 0;// 数组容量
    private int head = 0;// 头部下标
    private int tail = 0;// 尾部下标

    public QueueBasedArray(int capacity) {
        values = new String[capacity];
        this.capacity = capacity;
    }

    @Override
    public Boolean enqueue(String value) {
        // tail == capacity 表示队列末尾没有空间了
        if (tail == capacity) {
            // tail == capacity && head == 0 表示整个队列都占满了。
            if (head == 0) {
                return false;
            }
            // 数据搬移
            for (int i = head; i < tail; i++) {
                values[i - head] = values[i];
            }
            // 搬移完成后更新 head 和 tail
            tail -= head;
            head = 0;
        }
        values[tail] = value;
        tail++;
        return true;
    }

    @Override
    public String dequeue() {
        // 如果 head == tail 表示队列为空
        if (0 == tail) {
            return null;
        }
        String result = values[head];
        head++;
        return result;
    }

    @Override
    public String toString() {
        return "QueueBasedArray{" +
                "values=" + Arrays.toString(values) +
                ", capacity=" + capacity +
                ", head=" + head +
                ", tail=" + tail +
                '}';
    }
}
```

测试代码：

```
    QueueBasedArray qba = new QueueBasedArray(10);
    System.out.println(qba);

    for (int i = 0; i < 10; i++) {
        qba.enqueue("" + i + i + i);
    }
    System.out.println(qba);

    for (int i = 0; i < 10; i++) {
        qba.dequeue();
        System.out.println(qba);
    }

    for (int i = 11; i < 20; i++) {
        qba.enqueue("" + i + i + i);
    }
    System.out.println(qba);
```

输出结果：符合预期

```
QueueBasedArray{values=[null, null, null, null, null, null, null, null, null, null], capacity=10, head=0, tail=0}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=0, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=1, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=2, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=3, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=4, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=5, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=6, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=7, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=8, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=9, tail=10}
QueueBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, head=10, tail=10}
QueueBasedArray{values=[111111, 121212, 131313, 141414, 151515, 161616, 171717, 181818, 191919, 999], capacity=10, head=0, tail=9}
```

#### 链式队列
代码如下：

```
public class QueueBasedLinkedList implements QueueInterface {
    private Node head;
    private Node tail;

    /**
     * 入队
     *
     * @param value
     * @return
     */
    @Override
    public Boolean enqueue(String value) {
        Node newNode = new Node(value, null);

        // tail为null，表示队列中没有数据
        if (null == tail) {
            head = newNode;
            tail = newNode;
        } else {
            tail.next = newNode;
            tail = newNode;
        }

        return true;
    }

    /**
     * 出队
     *
     * @return
     */
    @Override
    public String dequeue() {
        // head == null,表示队列为空。
        if (null == head) {
            return null;
        }

        // 获取数据
        String value = head.getItem();
        // 移除头结点，让head指向下一个结点。
        head = head.next;
        // 如果此时的头结点指向null，说明队列已空，需要将tail指向null.
        if (null == head) {
            tail = null;
        }

        return value;
    }

    @Override
    public String toString() {
        return "QueueBasedLinkedList{" +
                "head=" + head +
                ", tail=" + tail +
                '}';
    }

    private static class Node {
        String item;
        private Node next;

        public Node(String item, Node next) {
            this.item = item;
            this.next = next;
        }

        public String getItem() {
            return item;
        }

        @Override
        public String toString() {
            return "Node{" +
                    "item='" + item + '\'' +
                    ", next=" + next +
                    '}';
        }
    }
}
```

测试代码：

```
    // 空队列
    QueueBasedLinkedList qbll = new QueueBasedLinkedList();
    System.out.println("空队列 " + qbll);
    System.out.println();

    // 入队一个数据
    System.out.println("数据入队是否成功：" + qbll.enqueue("0000"));
    System.out.println("入队一个数据后：" + qbll);
    System.out.println();

    // 出队一个数据
    System.out.println("出队的数据是：" + qbll.dequeue());
    System.out.println("出队一个数据后：" + qbll);
    System.out.println();

    // 异常测试：从空队列中出队，看结果
    System.out.println("出队的数据是1：" + qbll.dequeue());
    System.out.println("出队一个数据后1：" + qbll);
    System.out.println();

    // 入队十条数据
    for (int i = 0; i < 10; i++) {
        System.out.println("数据入队是否成功：" + qbll.enqueue("" + i + i + i));
        System.out.println(qbll);
    }
```

输出结果：符合预期

```
空队列 QueueBasedLinkedList{head=null, tail=null}

数据入队是否成功：true
入队一个数据后：QueueBasedLinkedList{head=Node{item='0000', next=null}, tail=Node{item='0000', next=null}}

出队的数据是：0000
出队一个数据后：QueueBasedLinkedList{head=null, tail=null}

出队的数据是1：null
出队一个数据后1：QueueBasedLinkedList{head=null, tail=null}

数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=null}, tail=Node{item='000', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=null}}, tail=Node{item='111', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=null}}}, tail=Node{item='222', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=Node{item='333', next=null}}}}, tail=Node{item='333', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=Node{item='333', next=Node{item='444', next=null}}}}}, tail=Node{item='444', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=Node{item='333', next=Node{item='444', next=Node{item='555', next=null}}}}}}, tail=Node{item='555', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=Node{item='333', next=Node{item='444', next=Node{item='555', next=Node{item='666', next=null}}}}}}}, tail=Node{item='666', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=Node{item='333', next=Node{item='444', next=Node{item='555', next=Node{item='666', next=Node{item='777', next=null}}}}}}}}, tail=Node{item='777', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=Node{item='333', next=Node{item='444', next=Node{item='555', next=Node{item='666', next=Node{item='777', next=Node{item='888', next=null}}}}}}}}}, tail=Node{item='888', next=null}}
数据入队是否成功：true
QueueBasedLinkedList{head=Node{item='000', next=Node{item='111', next=Node{item='222', next=Node{item='333', next=Node{item='444', next=Node{item='555', next=Node{item='666', next=Node{item='777', next=Node{item='888', next=Node{item='999', next=null}}}}}}}}}}, tail=Node{item='999', next=null}}
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。

github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)


参考：
[队列：队列在线程池等有限资源池中的应用](https://time.geekbang.org/column/article/41330)