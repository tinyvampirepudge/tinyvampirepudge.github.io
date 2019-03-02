---
layout: post
title:  Java实现二分查找
date:   2018-11-24 11:11:27 +0800
categories: 算法
tag: [二分查找]
---

* content
{:toc}



### Java实现二分查找
二分查找针对的是一个有序的数据集合，查找思想有点类似分治思想。每次都通过跟区间的中间元素对比，将待查找的区间缩小为之前的一半，直到找到要查找的元素，或者区间被缩小为0。

这里我们实现最简单情况下的二分查找。升序排列的数组，无重复元素，查找其中是否包含单个元素。
非递归方式实现：

```
	/**
     * 二分查找，针对不存在重复元素的。非递归实现。
     *
     * @param a     升序数组
     * @param n     数组大小
     * @param value 要查找的数值
     * @return 返回值表示要查找的数据在数组中的角标，如果没有则返回-1.
     */
    public static int binarySearch1(int[] a, int n, int value) {
        int low = 0;
        int high = n - 1;

        while (low <= high) {
            int mid = low + ((high - low) >> 1);
            if (a[mid] == value) {
                return mid;
            } else if (a[mid] < value) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }

        return -1;
    }
```

当然，也可以使用递归方式实现：

```
	/**
     * 二分查找，针对不存在重复元素的。递归实现。
     *
     * @param a     升序数组
     * @param n     数组大小
     * @param value 要查找的数值
     * @return 返回值表示要查找的数据在数组中的角标，如果没有则返回-1.
     */
    public static int binarySearch2(int[] a, int n, int value) {
        return binarySearchInternally(a, 0, n - 1, value);
    }

    private static int binarySearchInternally(int[] a, int low, int high, int value) {
        if (low > high) {
            return -1;
        }

        int mid = low + ((high - low) >> 1);
        if (a[mid] == value) {
            return mid;
        } else if (a[mid] < value) {
            return binarySearchInternally(a, mid + 1, high, value);
        } else {
            return binarySearchInternally(a, low, mid - 1, value);
        }
    }
```