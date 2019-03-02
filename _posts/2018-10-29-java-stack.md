---
layout: post
title:  Java代码实现顺序栈和链式栈
date:   2018-10-29 14:53:09 +0800
categories: 数据结构
tag: [Stack, 顺序栈, 链式栈]
---

* content
{:toc}



### Java代码实现顺序栈和链式栈

栈(stack)又名堆栈，它是一种运算受限的线性表。其限制是仅允许在表的一端进行插入或者删除运算。后进先出（Last In First Out）。

栈中的数据操作主要有push(压入)和pop(弹出)操作。

实际上，栈就可以用数组来实现，也可以用链表来实现。用数组实现的叫做顺序栈，用链表实现的叫做链式栈。

为了简单起见，我们的栈不支持泛型。

#### 顺序栈

```
public class StackBasedArray implements StackInterface{

    private String[] values; // 存储栈中元素的数组
    private int capacity;// 栈的容量
    private int count;// 栈中已有数据个数

    // 初始化栈，容量确定。
    public StackBasedArray(int capacity) {
        this.values = new String[capacity];
        this.capacity = capacity;
        this.count = 0;
    }

    /**
     * 入栈操作
     *
     * @param value
     * @return
     */
    public Boolean push(String value) {
        // 数组空间不够了
        if (capacity == count) {
            return false;
        }
        values[count] = value;
        count++;
        return true;
    }

    /**
     * 出栈操作
     *
     * @return
     */
    public String pop() {
        // 栈为空，则直接返回null
        if (0 == count) {
            return null;
        }
        String tmp = values[count - 1];
        --count;
        return tmp;
    }

    @Override
    public String toString() {
        return "StackBasedArray{" +
                "values=" + Arrays.toString(values) +
                ", capacity=" + capacity +
                ", count=" + count +
                '}';
    }
}
```

测试代码：

```
        StackBasedArray as = new StackBasedArray(10);
        System.out.println(as);

        Boolean r1 = as.push("000");
        System.out.println(as + ",r1:" + r1);

        for (int i = 1; i < 11; i++) {
            boolean r2 = as.push("" + i + i + i);
            System.out.println(as + ",r2:" + r2);
        }

        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
        System.out.println("as.pop():" + as.pop() + "," + as);
```

输出结果：符合预期

```
StackBasedArray{values=[null, null, null, null, null, null, null, null, null, null], capacity=10, count=0}
StackBasedArray{values=[000, null, null, null, null, null, null, null, null, null], capacity=10, count=1},r1:true
StackBasedArray{values=[000, 111, null, null, null, null, null, null, null, null], capacity=10, count=2},r2:true
StackBasedArray{values=[000, 111, 222, null, null, null, null, null, null, null], capacity=10, count=3},r2:true
StackBasedArray{values=[000, 111, 222, 333, null, null, null, null, null, null], capacity=10, count=4},r2:true
StackBasedArray{values=[000, 111, 222, 333, 444, null, null, null, null, null], capacity=10, count=5},r2:true
StackBasedArray{values=[000, 111, 222, 333, 444, 555, null, null, null, null], capacity=10, count=6},r2:true
StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, null, null, null], capacity=10, count=7},r2:true
StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, null, null], capacity=10, count=8},r2:true
StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, null], capacity=10, count=9},r2:true
StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=10},r2:true
StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=10},r2:false
as.pop():999,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=9}
as.pop():888,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=8}
as.pop():777,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=7}
as.pop():666,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=6}
as.pop():555,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=5}
as.pop():444,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=4}
as.pop():333,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=3}
as.pop():222,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=2}
as.pop():111,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=1}
as.pop():000,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=0}
as.pop():null,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=0}
as.pop():null,StackBasedArray{values=[000, 111, 222, 333, 444, 555, 666, 777, 888, 999], capacity=10, count=0}
```

#### 链式栈
使用链表来实现栈，push进的数据添加至链表的头结点，pop数据时取的也是链表的头结点，这样就实现了栈的后进先出。代码如下：

```
public class StackBasedLinkedList implements StackInterface {
    private Node first;

    /**
     * 每次添加数据都向链表头结点添加。
     *
     * @param value
     * @return
     */
    @Override
    public Boolean push(String value) {
        Node newNode = new Node(value, null);
        if (null == first) {
            first = newNode;
        } else {
            newNode.next = first;
            first = newNode;
        }
        return true;
    }

    /**
     * 每次获取数据都从链表头结点获取。
     *
     * @return
     */
    @Override
    public String pop() {
        if (null == first) {
            return null;
        } else {
            String tmp = first.getValue();
            first = first.next;
            return tmp;
        }
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

    private static class Node {
        Node next;
        String value;

        public Node(String value, Node next) {
            this.next = next;
            this.value = value;
        }

        public String getValue() {
            return value;
        }

        @Override
        public String toString() {
            return value;
        }
    }
}
```

测试代码：

```
    StackBasedLinkedList sbll = new StackBasedLinkedList();
    System.out.println(sbll);

    // 添加数据
    for (int i = 0; i < 10; i++) {
        sbll.push("" + i + i + i);
    }
    System.out.println(sbll);

    // 获取数据
    for (int i = 0; i < 11; i++) {
        System.out.println("sbll.pop():" + sbll.pop());
    }
```

输出结果：符合预期。

```
[]
[999, 888, 777, 666, 555, 444, 333, 222, 111, 000]
sbll.pop():999
sbll.pop():888
sbll.pop():777
sbll.pop():666
sbll.pop():555
sbll.pop():444
sbll.pop():333
sbll.pop():222
sbll.pop():111
sbll.pop():000
sbll.pop():null
```

#### 完整代码请查看
项目中搜索SingleLinkedList即可。

github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：
[如何实现浏览器的前进和后退功能？](https://time.geekbang.org/column/article/41222)