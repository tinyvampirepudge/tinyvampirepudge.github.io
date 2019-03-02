---
layout: post
title:  Java代码实现归并排序
date:   2018-11-13 17:46:52 +0800
categories: 算法
tag: [归并排序]
---

* content
{:toc}



### Java代码实现归并排序

#### 归并排序（Merge Sort）
思路：如果要排序一个数组，我们先把数组从中间分成前后两部分，然后对前后两部分分别排序，再将排好序的两部分合并在一起，这样整个数组就都有序了。

所以说归并排序的核心思想是排序和合并两个有序数组，这个规程需要用递归来实现。
核心思想：
* 将数组拆分为两个数组，然后对两个数组各自进行排序。
* 合并两个排好序的数组。

需要注意的是，对子数组进行排序依然是使用归并排序，所以这就涉及到了递归。

下面是Java代码的实现：
```
public class MergeSort {
/**
* 归并排序
*
* @param a 待排序的数组
* @param n 数组大小
*/
public static void mergeSort(int[] a, int n) {
mergeSortRecursion(a, 0, n - 1);
}

/**
* 递归调用的函数
*
* @param a
* @param p
* @param r
*/
public static void mergeSortRecursion(int[] a, int p, int r) {
// 递归终止条件
if (p >= r) {
return;
}

// 取p到r之间的位置q
int q = p + (r - p) / 2;
// 分治递归
mergeSortRecursion(a, p, q);
mergeSortRecursion(a, q + 1, r);

// 将A[p...q]和A[q+1...r]合并为A[p...r]
merge(a, p, q, r);
}

/**
* 合并两个有序数组
*
* @param a 合并好的有序数组，需要放到这个位置上。
* @param p
* @param q
* @param r
*/
public static void merge(int[] a, int p, int q, int r) {
int i = p;
int j = q + 1;
int k = 0;
int[] tmp = new int[r - p + 1];

// 最少把一个数组中的数据取完。
while (i <= q && j <= r) {
if (a[i] <= a[j]) {
tmp[k++] = a[i++];
} else {
tmp[k++] = a[j++];
}
}

// 判断哪个子数组中有剩余的数据。
int start = i;
int end = q;
if (j <= r) {
start = j;
end = r;
}
// 将剩余的数据copy到临时数组 tmp。
while (start <= end) {
tmp[k++] = a[start++];
}

//将 tmp 中的数据拷贝回 a 中
System.arraycopy(tmp, 0, a, p, r - p + 1);
}
}
```

测试代码如下：
```
public void onBtnTest5Clicked() {
int size = 10;
if (size <= 0) {
throw new IllegalArgumentException("size cannot bo lower than 0");
}
int[] a = SortUtils.generateRandomArray(size, 100);
System.out.println("归并排序：");
System.out.println(SortUtils.getArrayToString(a));
MergeSort.mergeSort(a, size);
System.out.println(SortUtils.getArrayToString(a));
System.out.println();
}
```

输出结果：符合期望
```
I/System.out: 归并排序：
I/System.out: [78,33,20,43,22,71,40,93,20,50]
I/System.out: [20,20,22,33,40,43,50,71,78,93]
```

#### 完整代码请查看
项目中搜索MergeSort即可。
github传送门 [https://github.com/tinyvampirepudge/DataStructureDemo](https://github.com/tinyvampirepudge/DataStructureDemo)

gitee传送门 [https://gitee.com/tinytongtong/DataStructureDemo](https://gitee.com/tinytongtong/DataStructureDemo)

参考：
[排序（下）：如何用快排思想在O(n)内查找第K大元素？](https://time.geekbang.org/column/article/41913)

