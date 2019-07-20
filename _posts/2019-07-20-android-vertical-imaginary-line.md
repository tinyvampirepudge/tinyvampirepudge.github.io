---
layout: post
title:  Android绘制竖直虚线完美解决方案—自定义View
date:   2019-07-20 14:04:44 +0800
categories: Android
tag: [DividerView]
---

* content
{:toc}



[TOC]

### Android绘制竖直虚线完美解决方案—自定义View

开发中我们经常会遇到绘制虚线的需求，一般我们使用一个drawable文件即可实现，下面我会先列举常规drawable文件的实现方式。

#### 使用drawable绘制水平虚线

水平方向的虚线最好绘制，drawable文件如下所示：

```
drawable/imaginary_line.xml:

<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line">

    <stroke
        android:width="1dp"
        android:color="#000"
        android:dashWidth="5dp"
        android:dashGap="2dp" />

</shape>
```

在布局中使用如下：

```
<!-- 这里的高度必须大于drawable中设置的虚线宽度 -->
<View
    android:layout_width="match_parent"
    android:layout_height="2dp"
    android:layout_marginTop="50dp"
    android:background="@drawable/imaginary_line"
    android:layerType="software" />
```

这里我们需要注意一下几点，第一最好设置`android:layerType="software"`属性，第二View的高度，最好大于drawable中设置的虚线高度。不然都可能导致虚线不显示。

#### 使用drawable绘制竖直方向虚线
与水平方向虚线相比，竖直方向虚线就麻烦的多了，而且有很多缺陷。

drawable代码如下所示：

```
drawable/vertical_imaginary_line.xml:

<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromDegrees="90"
    android:toDegrees="90">
    <shape android:shape="line">
        <stroke
            android:width="1dp"
            android:color="#000"
            android:dashWidth="5dp"
            android:dashGap="2dp" />
    </shape>
</rotate>
```

可以看出，实质上是通过View动画，对水平方向的View进行了旋转操作。

具体使用如下：跟水平方向使用方式一样。

```
 <View
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:layout_alignParentRight="true"
    android:background="@drawable/vertical_imaginary_line"
    android:layerType="software" />
```

因为View是先绘制水平方向的虚线，然后进行旋转，所以竖直虚线默认就会有偏移量，我们需要手动的去调整位置。

实现效果如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190720-125117%402x.png)

单个虚线还好说，如果需要绘制图表的网格线之类的需求，那就要欲哭无泪了。

#### 自定义DividerView
接下来祭出我们的大杀器自定义View。

先定义下需求，我们的虚线需要支持自定义背景色，支持自定义虚线宽度，支持水平和竖直方向，支持虚线的dash宽度和dash间隔，所以我们的自定义属性就如下所示：

```
attrs.xml:

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- 垂直方向的虚线 -->
    <declare-styleable name="DividerView">
        <!-- 虚线颜色 -->
        <attr name="divider_line_color" format="color"/>
        <!-- 虚线宽度 -->
        <attr name="dashThickness" format="dimension"/>
        <!-- 虚线dash宽度 -->
        <attr name="dashLength" format="dimension"/>
        <!-- 虚线dash间隔 -->
        <attr name="dashGap" format="dimension"/>
        <!-- 虚线朝向 -->
        <attr name="divider_orientation" format="enum">
            <enum name="horizontal" value="0"/>
            <enum name="vertical" value="1"/>
        </attr>
    </declare-styleable>
</resources>
```

接下来我们看下DividerView的具体实现：
自定义View的第一步，通常是获取自定义的属性值，具体如下所示：

```
public DividerView(Context context, AttributeSet attrs) {
    super(context, attrs);
    int dashGap, dashLength, dashThickness;
    int color;

    TypedArray a = context.getTheme().obtainStyledAttributes(attrs, R.styleable.DividerView, 0, 0);

    try {
        dashGap = a.getDimensionPixelSize(R.styleable.DividerView_dashGap, 5);
        dashLength = a.getDimensionPixelSize(R.styleable.DividerView_dashLength, 5);
        dashThickness = a.getDimensionPixelSize(R.styleable.DividerView_dashThickness, 3);
        color = a.getColor(R.styleable.DividerView_divider_line_color, 0xff000000);
        orientation = a.getInt(R.styleable.DividerView_divider_orientation, ORIENTATION_HORIZONTAL);
    } finally {
        a.recycle();
    }

    mPaint = new Paint();
    mPaint.setAntiAlias(true);
    mPaint.setColor(color);
    mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setStrokeWidth(dashThickness);
    mPaint.setPathEffect(new DashPathEffect(new float[]{dashGap, dashLength,}, 0));
}
```

我们通过TypedArray获取到我们设置的自定义属性值，并给各个属性设置默认值；接着初始化我们的画笔paint。

初始化工作完毕后，就是绘制工作了，代码如下所示：

```
@Override
protected void onDraw(Canvas canvas) {
    if (orientation == ORIENTATION_HORIZONTAL) {
        float center = getHeight() * 0.5f;
        canvas.drawLine(0, center, getWidth(), center, mPaint);
    } else {
        float center = getWidth() * 0.5f;
        canvas.drawLine(center, 0, center, getHeight(), mPaint);
    }
}
```

具体使用如下所示：

横向虚线：

```
<com.tinytongtong.dividerviewdemo.DividerView
    android:layout_width="match_parent"
    android:layout_height="1dp"
    android:layout_marginTop="50dp"
    android:layerType="software"
    custom:dashGap="4dp"
    custom:dashLength="1dp"
    custom:dashThickness="1dp"
    custom:divider_line_color="#ef5350"
    custom:divider_orientation="horizontal" />
```

竖向虚线：

```
<com.tinytongtong.dividerviewdemo.DividerView
    android:layout_width="1dp"
    android:layout_height="match_parent"
    android:layout_alignParentRight="true"
    android:layout_marginRight="50dp"
    android:layerType="software"
    custom:dashGap="4dp"
    custom:dashLength="1dp"
    custom:dashThickness="1dp"
    custom:divider_line_color="#ef5350"
    custom:divider_orientation="vertical" />
```

效果图如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190720-135007%402x.png)

[DividerView项目地址](https://github.com/tinyvampirepudge/DividerView)

参考：
[Android竖虚线绘制](https://blog.csdn.net/nalw2012/article/details/50978656)

