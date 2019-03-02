---
layout: post
title:  Java代码实现快速排序(QuickSort)
date:   2018-11-14 09:27:45 +0800
categories: 算法
tag: [快速排序]
---

* content
{:toc}



### Java代码实现快速排序(QuickSort)

#### 核心思想
如果要排序数组中下标从p到r之间的一组数据，我们选择p到r之间的任意一个数据为pivot（分区点）。

我们遍历p到r之间的数据，将小于pivot的放到左边，将大于pivot的放到右边，将pivot放到中间。经过这一步骤之后，数组p到r之间的数据就被分成了三个部分，前面p到q-1之间都是小于pivot的，中间是pivot，后面的q=1到r之间是大于pivot的。
![](https://static001.geekbang.org/resource/image/4d/81/4d892c3a2e08a17f16097d07ea088a81.jpg)

根据分治、递归的处理思想，我们可以用递归排序下标从p到q-1之间的数据和下标为q+1到r之前的数据，直到区间缩小为1，就说明所有的数据都有序了。

#### Java代码实现

```
public class QuickSort {

/**
* 快速排序
*
* @param a
* @param n
*/
public static void quickSort(int[] a, int n) {
quickSortRecursive(a, 0, n - 1);
}

/**
* 快速排序递归函数，p,r为下标。
*
* @param a
* @param p
* @param r
*/
public static void quickSortRecursive(int[] a, int p, int r) {
if (p >= r) {
return;
}
// 获取分区点
int q = partition(a, p, r);
quickSortRecursive(a, p, q - 1);
quickSortRecursive(a, q + 1, r);
}

public static int partition(int[] a, int p, int r) {
int pivot = a[r];
int i = p;
for (int j = p; j < r; j++) {
if (a[j] < pivot) {
int t = a[i];
a[i] = a[j];
a[j] = t;
i++;
}
}
int t = a[i];
a[i] = a[r];
a[r] = t;
return i;
}
}
```

测试代码：
```
int size = 10;
if (size <= 0) {
throw new IllegalArgumentException("size cannot bo lower than 0");
}
int[] a = SortUtils.generateRandomArray(size, 100);
System.out.println("快速排序：");
System.out.println(SortUtils.getArrayToString(a));
QuickSort.quickSort(a, size);
System.out.println(SortUtils.getArrayToString(a));
System.out.println();
```

测试结果：符合预期。
```
I/System.out: 快速排序：
I/System.out: [63,81,43,54,37,84,45,97,50,87]
I/System.out: [37,43,45,50,54,63,81,84,87,97]
```

#### 完整代码请查看
项目中搜索QuickSort即可。
github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：
[排序（下）：如何用快排思想在O(n)内查找第K大元素？](https://time.geekbang.org/column/article/41913)

