---
layout: post
title:  bug系列—1像素引发的血案
date:   2019-12-25 09:40:30 +0800
categories: Android
tag: [TextView]
---

* content
{:toc}


### bug系列—1像素引发的血案

#### 背景描述

使用RecyclerView实现了一个类似WheelView的效果，这里RecyclerView的高度是5个item的高度。大致效果如下图所示：

![](https://user-gold-cdn.xitu.io/2019/12/25/16f3ac62b60bdcd7?w=406&h=740&f=gif&s=3299992)

具体项目地址：

[WheelViewByRecyclerView](https://github.com/tinyvampirepudge/WheelViewByRecyclerView)

这里选用RecyclerView自己实现这个效果的原因是，UI给的设计图中，要求文案支持两行，如果超过两行了，需要支持文字自动缩小。而现有的三方库不支持`换行`这个操作，即使在三方库的基础上进行更改也不容易，所以这里决定自己实现。

为了实现RecyclerView滑动停止后的弹性滑动，这里使用了`LinearSnapHelper`，这样在滑动停止后，一定会有item停留在中间的选中位置。

接着就是获取停止后的位置，这里通过`linearLayoutManager.findFirstVisibleItemPosition()`方法获取到第一个可见item的位置`pos`，根据这`pos`就可以获取到中间item的角标。

RecyclerView的布局文件如下：

```
<android.support.v7.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="225dp"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toBottomOf="@id/view_line" />
```

RecyclerView的adapter布局如下：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="45dp"
        android:layout_marginLeft="@dimen/dimen_16dp"
        android:layout_marginRight="@dimen/dimen_16dp"
        android:gravity="center"
        android:textColor="@color/c_0B1D32"
        android:textSize="@dimen/dimen_18dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="啊哈哈哈" />

</android.support.constraint.ConstraintLayout>
```

我们给RecyclerView固定了高度225dp，刚好是5个item的高度（45dp * 5），理论上我们的逻辑没毛病

到此我们的核心逻辑完成，大部分机型上都正常工作。

#### 1、问题描述

```
诺基亚  X71安卓系统 项目选择出现错位，最后一个项目选择不到
```

```
上述问题 更换 华为DLI-AL10 也是一样
```

错位的效果是选择的都是上一个的数据，这也就表示最后一个元素无法选中。

初步猜想是机型分辨率适配问题，导致上方多显示了一个item，进而导致获取到的第一个可见item的位置错误。

#### 2、问题定位

我们先看下正常显示的手机：

RecyclerView的实际高度：625px

![](https://user-gold-cdn.xitu.io/2019/12/25/16f3abb8c7057056?w=2584&h=1162&f=png&s=1283582)

单个item的实际高度：125px

![](https://user-gold-cdn.xitu.io/2019/12/25/16f3abb8c7ad83c8?w=2546&h=1020&f=png&s=1142556)

经过计算：625 = 125 * 5，所以显示正常。


接下来我们看下出现异常的手机：

如下图所示，RecyclerView的实际高度为391px：

![](https://user-gold-cdn.xitu.io/2019/12/25/16f3abb8c7a6c311?w=2602&h=1128&f=png&s=1339870)

单个item的实际高度为78px：

![](https://user-gold-cdn.xitu.io/2019/12/25/16f3abb8c6f32dad?w=2584&h=1146&f=png&s=1351777)

我们发现低版本手机上，RecyclerView的实际高度比5个item高度总和多了一像素（`391 - (78 * 5) = 1` ），导致屏幕上实际显示了6个item。

所以最终问题出现的原因确定了，是dp转换成px后的误差问题。

具体原因是  `ScreenUtil.dip2px(mContext, 45) * 5;`的值跟`ScreenUtil.dip2px(mContext, 225)`的值不相等，他们的值在这里相差1px,但就是这一像素导致了我这里的错位bug。

#### 3、方案确定

既然是误差问题，那就有可能多，也有可能少，所以我们使用`linearLayoutManager.findFirstCompletelyVisibleItemPosition();`代替`linearLayoutManager.findFirstVisibleItemPosition();`的方案也依然会遇到错位的问题。

最终我决定在代码中动态设置一次RecyclerView的高度，以此来修复误差问题。

```
// 重新设置RecyclerView高度，避免误差
ConstraintLayout.LayoutParams lp = (ConstraintLayout.LayoutParams) recyclerView.getLayoutParams();
lp.height = ScreenUtil.dip2px(mContext, 45) * 5;
recyclerView.setLayoutParams(lp);
```

至此问题解决，如果觉得还可以的话记得点个赞哦！