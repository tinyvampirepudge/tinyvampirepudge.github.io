---
layout: post
title:  ScrollView和HorizontalScrollView无法设置点击事件的源码解析
date:   2019-07-23 13:32:51 +0800
categories: HorizontalScrollView
tag: [demo]
---

* content
{:toc}



### ScrollView和HorizontalScrollView无法设置点击事件的源码解析

最近的开发过程中，发现存在ScrollView和HorizontalScrollView无法设置点击事件的现象。

我们知道，通常在设置点击事件时，位于View树上方的子View的OnClickListener，会优先于父View的OnClickListener执行。

开发过程中我们会经常使用类似的方式来给布局设置点击事件，比如给ListView的Item背景设置OnClickListener，用于点击item空白区域的跳转操作；然后再给item内部的子元素分别设置OnClickListener用于各自不同的点击操作。

比如下图：
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190723-100254%402x.png)

灰色背景是个RelativeLayout，它内部我们放置了一个TextView，然后我们分别给它们设置点击事件。

点击TextView区域会触发TextView的点击事件，点击RelativeLayout内部除了TextView的其他区域（也就是灰色背景部分），会触发我们设置给RelativeLayout的点击事件。

上面展示了我们`设置点击事件的规律`，即如果同时给父布局和子布局设置点击事件，子布局的点击事件会优先执行，父布局的点击事件会在子View之外的空白区域执行。我们一般都是通过这种方式设置布局的点击事件的。

#### HorizontalScrollView不包含子布局

凡事总有例外，当子布局包含HorizontalScrollView的时候，情况就有所不同了。

我们先来看第一个例子，父布局是RelativeLayout（灰色背景），子布局包含一个上方的HorizontalScrollView（蓝色背景）和下方的一个TextView，这个HorizontalScrollView`没有设置子布局`。如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190723-100329%402x.png)

布局代码如下所示：

```
<RelativeLayout
    android:id="@+id/rl11"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:layout_marginTop="20dp"
    android:background="@color/lb_speech_orb_not_recording"
    app:layout_constraintTop_toBottomOf="@id/rl1">

    <HorizontalScrollView
        android:id="@+id/hsv11"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="@color/font_blue_2371e9"
        android:text="HorizontalScrollView">

    </HorizontalScrollView>

    <TextView
        android:id="@+id/tv11"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:background="@color/bg"
        android:padding="10dp"
        android:text="RelativeLayot + TextView"
        android:textAllCaps="false" />

</RelativeLayout>
```

接着我们给三个布局依次设置点击事件，依次点击RelativeLayout、HorizontalScrollView和TextView，点击顺序如下，分别点击1、2、3位置。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190723-105659%402x.png)

输出如下：

```
rl11 clicked
rl11 clicked
tv11 clicked
```

输出结果中的`rl11 clicked`是RelativeLayout的点击事件中打印的log，`tv11 clicked`是TextView的点击事件中打印的log。

也就是说，我们给HorizontalScrollView设置的点击事件中的log没有打印，取而代之打印的父布局的log，说明我们给HorizontalScrollView设置的点击事件无效。我们先记下这个现象，后续从源码角度进行说明。

#### HorizontalScrollView包含子布局

我们再来看第二个例子，父布局是RelativeLayout（灰色背景），子布局包含一个上方的HorizontalScrollView（蓝色背景）和下方的一个TextView，这个HorizontalScrollView中还`设置了一个子布局TextView`。如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190723-100351%402x.png)

布局代码如下所示：

```
<RelativeLayout
    android:id="@+id/rl12"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:layout_marginTop="20dp"
    android:background="@color/lb_speech_orb_not_recording"
    app:layout_constraintTop_toBottomOf="@id/rl11">

    <HorizontalScrollView
        android:id="@+id/hsv12"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="@color/font_blue_2371e9"
        android:text="HorizontalScrollView">

        <TextView
            android:id="@+id/tv120"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentBottom="true"
            android:background="@color/bg"
            android:padding="10dp"
            android:text="HorizontalScrollView + TextView"
            android:textAllCaps="false" />

    </HorizontalScrollView>

    <TextView
        android:id="@+id/tv121"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:background="@color/bg"
        android:padding="10dp"
        android:text="RelativeLayot + TextView"
        android:textAllCaps="false" />

</RelativeLayout>
```

接着我们给四个布局依次设置点击事件，依次点击RelativeLayout、HorizontalScrollView、HorizontalScrollView中的TextView和跟HorizontalScrollView平级的TextView，点击顺序如下，分别点击1、2、3、4位置。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190723-110001%402x.png)

输出如下：

```
rl12 clicked
无输出
tv120 clicked
tv121 clicked
```

输出结果中的`无输出`表示，点击蓝色区域无响应，也就是说我们给HorizontalScrollView设置的点击事件没有被调用。


#### 源码解析

了解过事件分发的同学都知道，View事件分发的三个核心方法分别是`dispatchTouchEvent`、`onInterceptTouchEvent`、`onTouchEvent`。

`dispatchTouchEvent`主要负责事件的分发，`onInterceptTouchEvent`用来表示ViewGroup是否拦截事件，`onTouchEvent`表示当前View对事件进行处理。

我们知道，ViewGroup的onInterceptTouchEvent方法默认返回false，也就是不拦截。ViewGroup中没有重写`onTouchEvent`方法，它使用的是View中的`onTouchEvent`方法。

通过查看常用的LinearLayout、RealtiveLayout、FrameLayout的源码可以发现，它们均没有重写`dispatchTouchEvent`、`onInterceptTouchEvent`、`onTouchEvent`这三个方法，也就是说他们中的事件分发都是遵循ViewGroup中的规则。

HorizontalScrollView继承自FrameLayout，通过查看HorizontalScrollView的源码发现，它没有重写`dispatchTouchEvent`方法，而是重写了`onInterceptTouchEvent`和`onTouchEvent`方法。

我们接下来就分析下HorizontalScrollView的`onInterceptTouchEvent`和`onTouchEvent`方法。

#### HorizontalScrollView#onInterceptTouchEvent

```
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    
    final int action = ev.getAction();
    if ((action == MotionEvent.ACTION_MOVE) && (mIsBeingDragged)) {
        return true;
    }

    if (super.onInterceptTouchEvent(ev)) {
        return true;
    }

    switch (action & MotionEvent.ACTION_MASK) {
        ...
    }

    /*
    * The only time we want to intercept motion events is if we are in the
    * drag mode.
    */
    return mIsBeingDragged;
}
```

国际惯例，我们省略掉无关代码，重点关注onInterceptTouchEvent方法的返回值，也就是三个return的地方。

1、先看第一个if判断，

```
if ((action == MotionEvent.ACTION_MOVE) && (mIsBeingDragged)) {
    return true;
}
```
当手势为MotionEvent.ACTION_MOVE并且正在拖动过程中，就返回true，表示拦截当前事件，事件就交由HorizontalScrollView#onTouchEvent方法处理。


2、接着看第二个if判断，

```
if (super.onInterceptTouchEvent(ev)) {
    return true;
}
```

这个`super.onInterceptTouchEvent(ev)`调用的就是ViewGroup#方法，默认返回false，所以if内部代码一般是不会走的。

3、在看最后一个return语句，

```
 /*
* The only time we want to intercept motion events is if we are in the
* drag mode.
*/
return mIsBeingDragged;
```

上方的注释说的很明白，只有在拖动时我们才会拦截事件，也就是返回true。

#### HorizontalScrollView#onTouchEvent

接下来我们看下HorizontalScrollView的onTouchEvent方法，代码如下：依旧省略无关代码。

```
@Override
public boolean onTouchEvent(MotionEvent ev) {
    ...
    switch (action & MotionEvent.ACTION_MASK) {
        case MotionEvent.ACTION_DOWN: {
            if (getChildCount() == 0) {
                return false;
            }
            ...
        }
        ...
    }
    return true;
}
```

HorizontalScrollView#onTouchEvent方法总结起来有两点：
1、代码中没有调用OnClickListener的地方，也没有调用super.onTouchEvent方法，所以说我们给HorizontalScrollView设置的OnClickListener都是无效的，不会被调用。

2、在看返回值，如果HorizontalScrollView没有子View，那么onTouchEvent就返回false，表示不消耗事件；其余情况都会返回true，表示消耗事件。

#### 原理分析

通过以上的分析，我们可以解释前面两个例子了。

点击操作不会导致`HorizontalScrollView#onInterceptTouchEvent`方法返回true，因为该方法只拦截滑动事件，所以它的返回值均为false。

针对第一个例子，因为HorizontalScrollView没有子View，所以它的HorizontalScrollView#onTouchEvent返回的就是false，表示不消耗事件，事件会继续向上传递给父View。


我们在点击没有子View的HorizontalScrollView区域时，触发的是父RelativeLayout的点击事件，具体原理如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190723-121519.png)


接下来解释第二个例子，当HorizontalScrollView有子View时，点击HorizontalScrollView的剩余区域是无反应的。具体原理如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190723-122045.png)

通过上述分析，我们知道给HorizontalScrollView设置点击事件是没有意义的，而且当HorizontalScrollView有子View时，它的空白区域还会消耗掉点击事件，让我们父布局的点击事件无法生效。为了避免这种情况，我们可以通过调整HorizontalScrollView区域范围或者给HorizontalScrollView的子View设置点击事件的方式来规避这些问题。

ScrollView的原理跟HorizontalScrollView是一样的，读者可自行验证。

#### github项目地址

[项目地址](https://github.com/tinyvampirepudge/Android_Base_Demo)

具体搜索[HorizontalScrollviewDispatchTouchEventActivity](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/view/dispatchevent/scrollview/HorizontalScrollviewDispatchTouchEventActivity.java)即可；

