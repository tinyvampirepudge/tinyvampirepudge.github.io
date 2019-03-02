---
layout: post
title:  创建文章demo
date:   2018-11-01 20:07:51 +0800
categories: 算法
tag: [冒泡排序, 插入排序, 选择排序]
---

* content
{:toc}



### 冒泡排序、插入排序、选择排序


#### 冒泡排序（Bubble Sort）
**冒泡排序只会操作相邻的两个数据**。两两比较，将较大的数换到后边。
代码如下：
```
/**
* 冒泡排序
* 两两比较，将较大的数移到后边。
*
* @param a
* @param n
*/
public void bubbleSort(int[] a, int n) {
if (n <= 1) return;
for (int i = 0; i < n; i++) {
for (int j = 0; j < n - i - 1; j++) {
// 交换条件
if (a[j] > a[j + 1]) {
int tmp = a[j];
a[j] = a[j + 1];
a[j + 1] = tmp;
}
}
}
}
```

这里我们还可以进行优化，如果某次冒泡之后已经没有数据可以交换时，说明已经达到完全有序，不用再继续执行后续的冒泡操作。代码如下：
```
/**
* 优化后的冒泡排序
*
* @param a
* @param n
*/
public void bubbleSortOptimize(int[] a, int n) {
if (n <= 1) return;
for (int i = 0; i < n; i++) {
// 提前退出冒泡循环的标志位
boolean flag = false;
for (int j = 0; j < n - i - 1; j++) {
if (a[j] > a[j + 1]) {
int tmp = a[j];
a[j] = a[j + 1];
a[j + 1] = tmp;
flag = true;
}
}
// 没有数据交换，提前退出
if (!flag) break;
}
}
```

#### 插入排序（Insertion Sort）
我们将待排序的数据分为两个区间，已排序区间和未排序区间。初始状态下，已排序区间为空。

插入算法的核心思想是取未排序区间的元素，在已排序区间中找到合适的位置插入，并保证已排序区间数据一直有序。重复这个过程，直到未排序区间中元素为空，算法结束。

代码如下：
```
/**
* 插入排序
* 将数据插入到有序数组中。
*
* @param a
* @param n
*/
public void insertSort(int[] a, int n) {
if (n <= 1) return;

for (int i = 1; i < n; i++) {
int value = a[i];
int j = i - 1;
// 查找插入的位置
for (; j >= 0; j--) {
if (a[j] > value) {
a[j + 1] = a[j];// 数据移动
} else {
break;
}
}
a[j + 1] = value;// 插入数据
}
}
```

#### 选择排序（Selection Sort）
跟插入排序类似，选择排序也区分已排序区间和未排序区间。初始状态下，已排序区间为空。
选择排序的核心是每次会从未排序区间中找到最小的元素，将其放到已排序区间的末尾。

代码如下：
```
/**
* 插入排序
* 从未排序区间中找到最小元素，将其放到已排序区间的末尾。
*
* @param a
* @param n
*/
public void selectSort(int[] a, int n) {
if (n <= 1) return;
for (int i = 0; i < n - 1; i++) {
// 查找最小值的角标
int minIndex = i;
for (int j = i + 1; j < n; j++) {
if (a[j] < a[minIndex]) {
minIndex = j;
}
}

// 减少无用的交换。
if (minIndex == i) {
continue;
}

// 交换
int tmp = a[i];
a[i] = a[minIndex];
a[minIndex] = tmp;
}
}
```

