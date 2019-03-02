---
layout: post
title:  Java数组实现循环队列
date:   2018-10-29 20:55:01 +0800
categories: 数据结构
tag: [循环队列]
---

* content
{:toc}



### Java数组实现循环队列

上一节([Java实现队列——顺序队列、链式队列](https://blog.csdn.net/qq_26287435/article/details/83509346))我们使用数组实现了顺序队列，但是在`tail == n`时会有数据搬移操作，这样入队操作性能就会受到影响。这里我们使用循环队列的解决思路。

#### 循环队列
顾名思义，首尾相连就形成了循环队列，如下图所示：
![](https://static001.geekbang.org/resource/image/58/90/58ba37bb4102b87d66dffe7148b0f990.jpg)

实现循环队列最关键的部分是确定队列何时为空何时满。在用数组实现的非循环队列中，队满的判断条件是tail == n，队空的判断条件是head == tail。在循环队列中，队列为空的判断条件仍然是head == tail，但队列满的判断条件有所更改，是`( tail + 1 ) % capacity == head`。
另外一点需要注意的是，当队列满时，tail指针指向的位置实际上是没有存储数据的，所以会浪费一个数组的存储空间。

代码实现如下：
```
public class CircularQueue implements QueueInterface {
private String[] values;// 容器
private int capacity = 0;// 容量
// head 表示队头下标，tail表示对尾下标
private int head = 0;
private int tail = 0;

public CircularQueue(int capacity) {
values = new String[capacity];
this.capacity = capacity;
}

@Override
public Boolean enqueue(String value) {
// 队列满了
if ((tail + 1) % capacity == head) {
return false;
}
values[tail] = value;
tail = (tail + 1) % capacity;
return true;
}

@Override
public String dequeue() {
// 如果head == tail 表示队列为空
if (head == tail) {
return null;
}
String ret = values[head];
head = (head + 1) % capacity;
return ret;
}

@Override
public String toString() {
return "CircularQueue{" +
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
CircularQueue cq = new CircularQueue(10);
System.out.println(cq);
System.out.println();

//正常添加的数据
System.out.println("正常添加的数据:");
for (int i = 0; i < 9; i++) {
System.out.println("cq.enqueue():" + cq.enqueue("" + i + i + i));
System.out.println(cq);
}
System.out.println();

// 队列满了以后添加数据
System.out.println("队列满了以后添加数据:");
System.out.println("cq.enqueue():" + cq.enqueue("aaa"));
System.out.println(cq);
System.out.println();

// 清空队列
System.out.println("清空队列:");
for (int i = 0; i < 9; i++) {
System.out.println("cq.dequeue():" + cq.dequeue());
System.out.println(cq);
}
System.out.println();

// 队列清空以后，继续清空
System.out.println("队列清空以后，继续清空:");
System.out.println("cq.dequeue():" + cq.dequeue());
System.out.println(cq);
```
输出结果：符合期望
```
CircularQueue{values=[null, null, null, null, null, null, null, null, null, null], capacity=10, head=0, tail=0}

正常添加的数据:
cq.enqueue():true
CircularQueue{values=[000, null, null, null, null, null, null, null, null, null], capacity=10, head=0, tail=1}
cq.enqueue():true
CircularQueue{values=[000, 111, null, null, null, null, null, null, null, null], capacity=10, head=0, tail=2}
cq.enqueue():true
CircularQueue{values=[000, 111, 222, null, null, null, null, null, null, null], capacity=10, head=0, tail=3}
cq.enqueue():true
CircularQueue{values=[000, 111, 222, 333, null, null, null, null, null, null], capacity=10, head=0, tail=4}
cq.enqueue():true
CircularQueue{values=[000, 111, 222, 333, 444, null, null, null, null, null], capacity=10, head=0, tail=5}
cq.enqueue():true
CircularQueue{values=[000, 111, 222, 333, 444, 555, null, null, null, null], capacity=10, head=0, tail=6}
cq.enqueue():true
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, null, null, null], capacity=10, head=0, tail=7}
cq.enqueue():true
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, null, null], capacity=10, head=0, tail=8}
cq.enqueue():true
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=0, tail=9}

队列满了以后添加数据:
cq.enqueue():false
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=0, tail=9}

清空队列:
cq.dequeue():000
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=1, tail=9}
cq.dequeue():111
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=2, tail=9}
cq.dequeue():222
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=3, tail=9}
cq.dequeue():333
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=4, tail=9}
cq.dequeue():444
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=5, tail=9}
cq.dequeue():555
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=6, tail=9}
cq.dequeue():666
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=7, tail=9}
cq.dequeue():777
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=8, tail=9}
cq.dequeue():888
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=9, tail=9}

队列清空以后，继续清空:
cq.dequeue():null
CircularQueue{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, head=9, tail=9}
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。
github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：
[队列：队列在线程池等有限资源池中的应用](https://time.geekbang.org/column/article/41330)


