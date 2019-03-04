---
layout: post
title:  Andorid分组Item顶部悬停 + 交互同步
date:   2018-04-24 10:52:37 +0800
categories: Android
tag: [自定义View, Item分组, 顶部悬停, 交互同步]
---

* content
{:toc}



# Andorid分组Item顶部悬停 + 交互同步

### 需求概述
　　项目中某些页面中的分组数据的顶部需要悬停，并且悬停的View要与ItemView中同样布局的View进行操作同步，也就是相互同步。大家都知道，Android中有"The specified child already has a parent. You must call removeView() on the child's parent first."这个异常，意味着同一个View对象不能有两个Parent。我们就不能简单粗暴的将同一个View对象添加进两个parent了，需要另谋出路。

### 方案选择：
①sitckyScrollView悬停：不支持list的复用，主线程会卡顿，pass。

②在listview的顶部覆盖一个View，重新生成需要悬停的View，并做到悬停View和原View的同步。针对listview的滑动和分组悬停，这个方案工作量太大，可行性不高。

③NestedScrollingChild和NestedScrollingParent方案，不支持分组，不合适。

④recyclerView + ItemDecoration方案 + View.draw(canvas) + motionEvent.offLocation()：可行。

　　我为什么选择第四个方案呢？主要原因如下：
　　
　　首先我们操作的是个列表，在控制悬停的View的显示和移动时必须要知道顶部的Item的信息，RecyclerView.ItemDecoration可以很好的解决这个问题。在ItemDecoration中可以轻松获取到RecyclerView、可见的position以及RecyclerView.Adapter中的可见View等信息，这样我们获取到需要悬停的View就很容易了。
　　
　　第二，在ItemDecoration#onDrawView( )方法中我们可以将需要悬停的View绘制出来。
　　
　　ItemDecoration轻松帮我们实现了悬停View的绘制，我们只需要处理真实View与悬绘制出来的悬停View的状态同步即可。至于如何实现状态同步，这个问题留待后面再说明。

### ItemDecoration
　　这里先说下ItemDecoration的实现，它是一个接口，内部各个方法的作用如下图所示：
![ItemDecoration内部方法](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/ItemDecoration.png)

如上所述，我们绘制View的时机应该是在onDrawOver方法中。

### 如何顶部的View
先上代码，

```javascript
//获取最顶部的ItemView
View adapterView = parent.getChildAt(0);
        if (adapterView != null) {
			//获取需要绘制的View，这里我们需要绘制的包括一个title，一个NewCHLayoutUnScroll的Header。
			//顺便获取这两个View的高度，后面我们需要他们的高度来实现悬停View异动的效果
            View title = adapterView.findViewById(R.id.title);
            int titleHeight = 0;
            if (title != null && View.VISIBLE == title.getVisibility()) {
                titleHeight = title.getMeasuredHeight();
            }

            int saveCount = canvas.save();
            //设置总体偏移量，需要用到我们上边获取到的高度
            stickyViewHeight = titleHeight;
            if (adapterView.getBottom() < stickyViewHeight) {
                offsetY = stickyViewHeight - adapterView.getBottom();
                canvas.translate(0, -offsetY);
            }
            //渲染View
            if (title != null) {
                title.draw(canvas);
                isTitleDrawed = true;
            }
            canvas.restoreToCount(saveCount);
        } else {

        }
```

接下来对上述代码进行说明：

1、获取最顶部的ItemView
我们知道，RecyclerView#getChildren方法可以获取到当前所有可见的ItemView，同理，RecyclerView#getChildAt(int index)就可以根据position获取到对应位置的View，这里我们就可以通过View adapterView = parent.getChildAt(0)来获取到最顶部的ItemView了。

2、获取需要绘制的View
我们需要绘制在顶部的View是最顶部的ItemView的子View，根据view.findviewById(id)就可以获取到需要绘制的View了。

3、为了实现竖直方向RecyclerView时悬停的View同步上下滑动的效果，我们需要找到悬停View显示完全与不完全的临界值，如下图所示：

![情况一和情况二](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/ItemDecoration%23onDrawOver1.png)

![情况三](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/ItemDecoration%23onDrawOver2.png)

　　如上图所分析，在绘制悬停View时，我们可以根据悬停View的高度和最上方ItemView.getBottom( )的大小来确定悬停View绘制的offset，从而就可以实现悬停View在合适的时机跟随RecyclerView滑动。

### 将ItemView中的状态变化同步给悬停View。
　　这里说的状态变化同步主要包括itemView中的列表左右滑动和ItemView中的title的点击事件触发的悬停View的状态更新。实现起来其实很简单，只需要在ItemView中更新状态时调用下面这行代码即可：

```
mRecyclerView.invalidateItemDecorations();
```
　　RecyclerView#invalidateItemDecorations( )方法会引起ItemDecoration的重绘，onDrawOver方法势必会重新调用，所以悬停View也就会重新绘制，就会跟顶部ItemView的title保持一致。

### 将悬停View的事件同步给顶部的ItemView。
　　这一步骤是最棘手的一步，这个问题可以理解为如何将canvas绘制的View的事件同步到被绘制的View上去。首先绘制出来的悬停View并不是真正的View，它的事件默认是传递给RecyclerView的，即使在RecyclerView中直接拦截了这个事件，如何处理也是个问题，因为很难定位MotionEvent的实际位置。
到了这一步，我们就可以借鉴前面提到过的StickyScrollView中对绘制出的悬停View的处理方法了，核心代码如下所示：

```
StickyScrollView#onTouchEvent
@Override
    public boolean onTouchEvent(MotionEvent ev) {
        if (redirectTouchesToStickyView) {
            ev.offsetLocation(0, ((getScrollY() + stickyViewTopOffset) - getTopForViewRelativeOnlyChild(currentlyStickingView)));
        }

        ...
        return super.onTouchEvent(ev);
    }
```

核心代码是ev.offsetLocation( )，我们看下它的源码：

```
MotionEvent#offsetLocation
/**
     * Adjust this event's location.
     * @param deltaX Amount to add to the current X coordinate of the event.
     * @param deltaY Amount to add to the current Y coordinate of the event.
     */
    public final void offsetLocation(float deltaX, float deltaY) {
        if (deltaX != 0.0f || deltaY != 0.0f) {
            nativeOffsetLocation(mNativePtr, deltaX, deltaY);
        }
    }
```
　　这个方法会将MotionEvent的作用位置偏移一定的位置，也就是说会将事件传递到别的位置上。另外，在ViewGroup的事件分发的源码中，也是通过MotionEvent#offsetLocation(offsetX, offsetY)来对事件进行处理的。

　　通过以上分析，我们可以通过MotionEvent#offsetLocation(offsetX, offsetY)方法将悬停View的MotionEvent传递给真实的View区域即可，唯一需要做的就是计算offsetY的值。

　　还有一个环节需要注意，我们在哪儿获取到这个MotionEvent，如何获取到RecyclerView.Item的事件呢？请看这儿，[Passing MotionEvents from RecyclerView.OnItemTouchListener to GestureDetectorCompat](https://stackoverflow.com/questions/26524003/passing-motionevents-from-recyclerview-onitemtouchlistener-to-gesturedetectorcom)，首先给recyclerView添加OnItemTouchListener，然后在OnItemTouchListener#onInterceptTouchEvent方法中就可以获取到事件了；获取到事件之后我们还需要借助手势相关的类来对事件进行处理。
　　
具体代码如下：

```javascript
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
                         Bundle savedInstanceState) {
    View rootView = inflater.inflate(R.layout.myfrag, container, false);

    detector = new GestureDetectorCompat(getActivity(), new RecyclerViewOnGestureListener());

    recyclerView = (RecyclerView) rootView.findViewById(R.id.recyclerview);

    layoutManager = new LinearLayoutManager(getActivity());
    recyclerView.setLayoutManager(layoutManager);
    recyclerView.addOnItemTouchListener(this);

    adapter = new MyAdapter(myData));
    recyclerView.setAdapter(adapter);
    return rootView;
}

private class RecyclerViewOnGestureListener extends SimpleOnGestureListener {

    @Override
    public boolean onSingleTapConfirmed(MotionEvent e) {
        View view = recyclerView.findChildViewUnder(e.getX(), e.getY());
        int position = recyclerView.getChildPosition(view);

        // handle single tap

        return super.onSingleTapConfirmed(e);
    }

    public void onLongPress(MotionEvent e) {
        View view = recyclerView.findChildViewUnder(e.getX(), e.getY());
        int position = recyclerView.getChildPosition(view);

        // handle long press

        super.onLongPress(e);
    }
}

@Override
public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {
    detector.onTouchEvent(e);
    return false;
}

@Override
public void onTouchEvent(RecyclerView rv, MotionEvent e) {
}
```

　　好了，上面分析了如何实现类似IOS的分组悬停效果，对解决这个问题的思路进行了阐述，这里大致总结下：
![Android分组悬停 + 事件处理思路总结](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Android%E5%88%86%E7%BB%84%E6%82%AC%E5%81%9C%20%2B%20%E6%82%AC%E5%81%9CView%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86.png)

参考：

1、[深入理解ItemDecoration](https://blog.piasy.com/2016/03/26/Insight-Android-RecyclerView-ItemDecoration/)

2、[灵感来源](https://mp.weixin.qq.com/s/-5ENiSSzulvu0yYokfLphg)

3、[recyclerView的事件处理](https://stackoverflow.com/questions/26524003/passing-motionevents-from-recyclerview-onitemtouchlistener-to-gesturedetectorcom)

4、[手势检测](http://www.gcssloop.com/customview/gestruedector)


