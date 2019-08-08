---
layout: post
title:  RecyclerView实现吸底效果—ItemDecoration
date:   2019-08-08 11:19:20 +0800
categories: RecyclerView
tag: [ItemDecoration]
---

* content
{:toc}




### RecyclerView实现吸底效果—ItemDecoration

这些天遇到一个列表数据吸底需求，如果不满一屏就全部展示，如果超过一屏就让底部悬浮在屏幕底部。

大概效果如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-07-2019%2022-38-47.gif)

列表我们一般用RecyclerView来实现，关于底部悬浮这里有两种实现方法，一种是通过测量RecyclerView内容高度，另一种是用我们熟悉的ItemDecoration来实现。

下面就具体介绍这两种实现方式。

#### 测量RecyclerView内容高度实现

这种方式很直观，我们先获取RecyclerView控件的高度h1，设置完数据后再获取RecyclerView的内容高度h2，然后将h1与h2进行比较：

①如果h1大于等于h2，则说明内容没有超出屏幕高度，此时只需要将数据完全展示即可。

②如果h1小于h2，则说明RecyclerView内容高度超出屏幕，此时RecyclerView可滚动，所以我们需要在RecyclerView底部显示吸底的View。

##### 原理示意图

RecyclerView控件的高度我们定义为h1，如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-074518%402x.png)

通过recyclerView#getHeight方法获取到的高度是固定的，就是布局文件中设定的recyclerView高度。

具体代码为：

```
// 获取RecyclerView控件高度
int recyclerViewHeight = recyclerView.getHeight();
LogUtils.e(TAG, "recyclerViewHeight: " + recyclerViewHeight);
```

RecyclerView内容的高度我们定义为h2，如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-090522%402x.png)


由上图可知，h2的高度需要在RecyclerView绘制完成以后动态获取，具体代码如下所示：

```
// 获取recyclerView的内容高度
int recyclerViewRealHeight = recyclerView.computeVerticalScrollRange();
LogUtils.e(TAG, "recyclerViewRealHeight: " + recyclerViewRealHeight);
```

h1>=h2的情况，具体如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-074619%402x.png)

我们只需要让Recycler的Adapter普通Item布局和底部的Footer布局就可以了。

最后我们看下h1<h2的情况，具体如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-074629%402x.png)

我们在RecyclerView控件的上方，盖一个布局，这个悬浮布局的实现要和Adapter中的Footer布局实现一样。

##### 具体实现方式

接着我们看下如何实现。具体分为如下几个步骤：
①将RecyclerView的父布局修改为RelativeLayouot，在RelativeLayouot的底部、RecyclerView的上方添加一个Footer布局。
②让Adapter支持两种布局，普通Item和Footer布局
③在给RecyclerView设置完数据后，获取RecyclerView的控件高度h1和RecyclerView的内容高度h2
④如果h1<h2，就让RecyclerView上方的Footer布局显示，否则就不显示。

接下来看代码：

①布局

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.view.RecyclerViewBottomFloatByViewHeightActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </android.support.v7.widget.RecyclerView>

    <TextView
        android:id="@+id/tv_bottom"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_alignParentBottom="true"
        android:background="#BCEAC1"
        android:gravity="center"
        android:text="我是底部"
        android:visibility="gone" />

</RelativeLayout>
```

②关于RecyclerView.Adapter如何支持多种ViewType，这里就不再细说了，具体代码实现文末有链接。

③获取h1和h2的值：为了避免recyclerView获取到的高度0，我们需要在给RecyclerView设置完数据之后，通过View#post(Runnable)方法获取。具体代码如下：

```
recyclerView.post(() -> {

    // 获取RecyclerView控件高度
    int recyclerViewHeight = recyclerView.getHeight();
    LogUtils.e(TAG, "recyclerViewHeight: " + recyclerViewHeight);

    // 获取recyclerView的内容高度
    int recyclerViewRealHeight = recyclerView.computeVerticalScrollRange();
    LogUtils.e(TAG, "recyclerViewRealHeight: " + recyclerViewRealHeight);
});
```

④默认情况下悬浮布局不显示，只有h1<h2时，该悬浮布局才显示，核心代码如下：

```
// 根据剩余空间确定是否需要显示吸底的图表底部
if (recyclerViewHeight < recyclerViewRealHeight) {
    tvBottom.setVisibility(View.VISIBLE);
} else {
    tvBottom.setVisibility(View.GONE);
}
```

##### 总结

需要说明的是，这种`通过获取View高度来实现单个View悬浮效果`的方式，不仅仅适用于RecyclerView，它更是一种通用的方式。但它的缺点也很明显，需要根据不容的业务去计算不同的View的高度。

一般不推荐这种方式去实现，不过它可以当做一个保底方案，毕竟简单粗暴易理解易实现。

#### ItemDecoration实现分组悬停原理

接下来我们来讲解如何使用ItemDecoration来实现底部View悬浮效果。

我们知道，系统提供了[DividerItemDecoration](https://developer.android.com/reference/android/support/v7/widget/DividerItemDecoration)组件，让我们方便的给RecyclerView绘制分割线。

DividerItemDecoration的具体使用方式请看[RecyclerView设置分割线---DividerItemDecoration](https://www.jianshu.com/p/86aaaa49ed3e)，具体代码示例请看[RecyclerViewDividerItemDecorationActivity](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/ui/view/RecyclerViewDividerItemDecorationActivity.java)。

这里简单介绍下ItemDecoration。

接触过ItemDecoration的同学知道，通过自定义ItemDecoration就可以实现酷炫的分组悬停效果。

ItemDecoration中有三个重要方法，源码如下：

```
public static abstract class ItemDecoration {
    ...
    public void onDraw(Canvas c, RecyclerView parent, State state) {
        onDraw(c, parent);
    }
    public void onDrawOver(Canvas c, RecyclerView parent, State state) {
        onDrawOver(c, parent);
    }
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
        getItemOffsets(outRect, ((LayoutParams) view.getLayoutParams()).getViewLayoutPosition(),
                parent);
    }
}
```

这三个方法的作用如下：

* ItemDecoration#getItemOffsets：通过Rect为每个Item设置偏移，为onDraw和onDrawOver方法中的绘制预留空间。

* ItemDecoration#onDraw：通过该方法，在Canvas上绘制内容，在绘制Item之前调用。（如果没有通过getItemOffsets设置偏移的话，Item的内容会将其覆盖）

* ItemDecoration#onDrawOver：通过该方法，在Canvas上绘制内容,在Item之后调用。(画的内容会覆盖在item的上层)

他们的层级关系如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-094401%402x.png)

需要说明的是，这三个方法都是针对每个可见Item的区域的，如果不加限制的话，每个Item都会调用它。

如果我们重写了`ItemDecoration#getItemOffsets`方法，该方法就会在现有Item空间的基础上新增空间，所以这个操作也会修改我们RecyclerView内容高度。

具体实例请看[RecyclerViewCustomItemDecorationDividerActivity](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/ui/view/RecyclerViewCustomItemDecorationDividerActivity.java)和[MyDividerItemDecoration](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/ui/widget/MyDividerItemDecoration.java)。页面打开方式如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-08-2019%2009-59-58.gif)

在用ItemDecoration实现分组悬停的过程中，又可以细分为两种方法。

一种是通过getItemOffsets方法预留空间，然后在onDrawOver中对应的区域绘制悬停的头部。悬停的部分需要额外绘制，不会复用Adapter中的Item的View。

另一种方法是，将需要悬停的部分也绘制到Item中，Adapter中的Item是以组为基本单位，一个Item会包含组中的所有View，Item内部第一个元素就是需要绘制的悬停头部。然后我们就可以在onDrawOver获取第一个可见Item的头部View，接着复用这个头部View，将其绘制在顶部即可。

接下来对这两种方式进行介绍。

##### 分组悬停实现方式一：getItemOffsets预留空间，onDrawOver中重新绘制悬停View，不复用

先看下不添加ItemDecoration的效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-08-2019%2009-57-32.gif)

再看下添加完ItemDecoration后的效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-08-2019%2010-07-51.gif)

具体代码请参照[RecyclerViewCustomItemDecorationFloatGroupActivity](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/ui/view/RecyclerViewCustomItemDecorationFloatGroupActivity.java)。这个类中的实现其实是简化了[Gavin-ZYX/StickyDecoration](https://github.com/Gavin-ZYX/StickyDecoration)项目中的实现。

这里需要说明的是，这种方法实现的核心是`getItemOffsets预留空间，onDrawOver直接在Item上层绘制新的悬停布局，悬停布局不复用ItemView`。从上面的示例可以看出，分组的头部View是在ItemDecoration中绘制的，在Adapter中不用绘制分组的头部。

##### 分组悬停实现方式二：onDrawOver中获取Item中的可见View，从中获取分组头部View进行复用

这种方法，将需要悬停的部分也绘制到Item中，Adapter中的Item是一个组的所有元素，Item内部第一个元素就是需要绘制的悬停头部。然后我们就可以在onDrawOver获取第一个可见Item的头部View，接着复用这个头部View，将其绘制在顶部即可。

示意图如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-102340%402x.png)

我们在onDrawOver中获取到第一个可见子View，然后根据id从里面获取到头部View，接着将这个用canvas将这个View绘制出来即可。

有兴趣的同学可以自行实现。

#### ItemDecoration实现吸底效果

我们的这个吸底效果跟分组悬停效果是有所不同的，分组悬停效果针对的是第一个可见的子View，吸底效果针对的是最后一个可见的子View。

我们的实现思路如下：
①让RecyclerView.Adapter支持普通的Item和Footer类型的Item。
②通过ItemDecoration绘制悬停View。

emmmmm，看起来很简单的样子。

通过上面对ItemDecoration中三个核心方法的分析，这里我们选择onDrawOver方法来完成绘制，直接在最后一个Item上方绘制一个一模一样的Footer即可。

我们前面说过，onDrawOver这几个方法是针对所有Item的，如果不加限制，则所有的Item都会绘制。

接下来就是选择使用哪个`可见子View`绘制这个Footer的问题了。我们有两种选择，一个是`最后一个可见的子View——lastView`，一个是`最后一个完全可见的子View——lastVisibleView`，他们的位置分别通过下面方法获取到:

```
int lastPosition = ((LinearLayoutManager)recyclerView.getLayoutManager()).findLastVisibleItemPosition();
```

```
int lastCompletelyVisibleItemPosition = ((LinearLayoutManager)parent.getLayoutManager()).findLastCompletelyVisibleItemPosition();
```

关于RecyclerView常用方法的总结，请看[RecyclerView常用方法总结](https://blog.csdn.net/qq_26287435/article/details/98849022)。

在多数情况下，lastView跟lastVisibleView不是同一个，只有在最后一个可见View的底部刚好达到RecyclerView下边界的时候，lastView跟lastVisibleView就是同一个了。

大多数情况下，lastView跟lastVisibleView都不是同一个，具体如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-104015%402x.png)

当某个Item的底部与RecyclerView的底部重叠时，lastView跟lastVisibleView就是同一个了，具体如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-104227%402x.png)

我们先看使用lastVisibleView来绘制底部悬浮View的情况。
lastVisibleView永远在RecyclerView内部显示，它的bottom的值会一直小于等于RecyclerView.getHeight的值的。

默认情况下，悬浮View会绘制在lastVisibleView内部，跟lastVisibleView底部对齐。所以我们需要给悬浮View设置一个向下的偏移量，这个偏移量的值就是RecyclerView.getHeight - lastVisibleView.getBottom的值。具体如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-105050%402x.png)

我们只需要给绘制好的Footer添加一个`offset`的值，让其向下偏移offset的值即可。

然而不幸的是，通过onDrawOver绘制的View，是`不能超出Item下边界范围`的。如果超出对应Item的bottom区域的话就无法显示，也就是说此路不通。

没办法了，只能看下lastView了。

我们以`lastView.getTop的值-悬浮View高度`的结果作为绘制悬浮View的top值，所以悬浮View相当于一直悬浮在lastView的顶部。

幸运的是，即使超出Item上方区域，onDrawOver的内容也是正常显示的。

接下来我们需要给top值设置一个偏移量，这个偏移量就是RecyclerView.getHeight - lastVisibleView.getTop的值。

具体如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190808-110110%402x.png)

最后我们看下效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-08-2019%2011-03-01.gif)

具体实现请看[RecyclerViewBottomFloatByItemDecorationActivity](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/ui/view/RecyclerViewBottomFloatByItemDecorationActivity.java)和[BottomFloatItemDecoration](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/ui/widget/BottomFloatItemDecoration.java)。


github项目地址：[Android_Base_Demo](https://github.com/tinyvampirepudge/Android_Base_Demo)

RecyclerView相关的demo打开方式如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-08-2019%2010-01-51.gif)

喜欢的话就点个赞吧！

#### 参考

1、[【Android】RecyclerView：打造悬浮效果](https://www.jianshu.com/p/b335b620af39)

2、[Gavin-ZYX/StickyDecoration](https://github.com/Gavin-ZYX/StickyDecoration)

3、[RecyclerView设置分割线---DividerItemDecoration](https://www.jianshu.com/p/86aaaa49ed3e)

4、[RecyclerView常用方法总结](https://blog.csdn.net/qq_26287435/article/details/98849022)

