---
layout: post
title:  View的测量、布局和绘制过程中父View(当前View)和子View的先后顺序
date:   2019-07-01 18:25:32 +0800
categories: View
tag: [View测量]
---

* content
{:toc}



### View的测量、布局和绘制过程中父View(当前View)和子View的先后顺序

View的测量、布局和绘制过程中，到底是先测量（布局、绘制）父View，还是先测量子View，这篇文章会从源码角度给出答案。

#### onMeasure过程

View的测量是从measure方法开始的，我们就先看下View#measure的方法：

```
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    ...
    if (forceLayout || needsLayout) {
        ...
        if (cacheIndex < 0 || sIgnoreMeasureCache) {
            // measure ourselves, this should set the measured dimension flag back
            onMeasure(widthMeasureSpec, heightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        } else {
            ...
        }
        ...
    }
    ...
}
```
可以看出，measure会调用View#onMeasure方法进行测量。

再来看下View#onMeasure的实现：其实就是设置测量宽高。

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

##### LinearLayout#onMeasure
我们知道，ViewGroup一般会重写View#onMeasure方法，不同的ViewGroup的实现方式大同小异，我们以LinearLayout为例：

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    if (mOrientation == VERTICAL) {
        measureVertical(widthMeasureSpec, heightMeasureSpec);
    } else {
        measureHorizontal(widthMeasureSpec, heightMeasureSpec);
    }
}
```

我们这里选择垂直方向的measureVertical方法：

```
void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
    mTotalLength = 0;
    ...
    // 循环遍历所有child，调用View#measure进行测量
    // See how tall everyone is. Also remember max width.
    for (int i = 0; i < count; ++i) {
        final View child = getVirtualChildAt(i);
        ...
        if (heightMode == MeasureSpec.EXACTLY && useExcessSpace) {
            ...
        } else {
            ...
            // 测量child的宽高
            measureChildBeforeLayout(child, i, widthMeasureSpec, 0,
                    heightMeasureSpec, usedHeight);
            // child测量完成后，获取测量后的高度
            final int childHeight = child.getMeasuredHeight();
            ...

            // 将child的高度累加到mTotalLength中
            mTotalLength = Math.max(totalLength, totalLength + childHeight + lp.topMargin +
                   lp.bottomMargin + getNextLocationOffset(child));
        }
        ...
    }

    ...

    // Add in our padding
    mTotalLength += mPaddingTop + mPaddingBottom;
    
    // 将mTotalLength赋值给heightSize，并对heightSize进行转化
    int heightSize = mTotalLength;
    heightSize = Math.max(heightSize, getSuggestedMinimumHeight());
    
    // 将heightSize和heightMeasureSpec转化为heightSizeAndState
    // Reconcile our calculated size with the heightMeasureSpec
    int heightSizeAndState = resolveSizeAndState(heightSize, heightMeasureSpec, 0);
    ...
    
    // 调用View#setMeasuredDimension方法设置LinearLayout的的测量宽高
    setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
            heightSizeAndState);
    ...
}
```

①这里开启一个for循环，会先去测量每个子View的大小。
②子View测量完毕后，会将测量宽高分别赋值给每个View的`mMeasuredWidth`和`mMeasuredHeight`。
③接着会将每个子View的高度值累加给成员变量`mTotalLength`
④接着将`mTotalLength`转换后赋值给`heightSize`，再将`heightSize`转换后赋值给`heightSizeAndState`。
⑤在`measureVertical`方法最后调用`setMeasuredDimension`方法，利用得到的`heightSizeAndState`去设置LinearLayout的高度。

简而言之，就是先调用子view的measure进行测量，完成后将其宽高记录下来，等所有子View测量完成后，就可以得到当前View的宽高了。

所以测量过程是先测量子View，再测量父View，因为父View的宽高会用到子View的测量结果。

##### LinearLayout#measureChildBeforeLayout

我们这里简单看下LinearLayout#measureChildBeforeLayout方法：它调用了measureChildWithMargins

```
void measureChildBeforeLayout(View child, int childIndex,
        int widthMeasureSpec, int totalWidth, int heightMeasureSpec,
        int totalHeight) {
    measureChildWithMargins(child, widthMeasureSpec, totalWidth,
            heightMeasureSpec, totalHeight);
}

```

##### LinearLayout#measureChildWithMargins方法：

```
protected void measureChildWithMargins(View child,
        int parentWidthMeasureSpec, int widthUsed,
        int parentHeightMeasureSpec, int heightUsed) {
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```
该方法分为三步：
①获取child的MarginLayoutParams参数。
②通过ViewGroup#getChildMeasureSpec方法分别生成宽度和高度的MeasureSpec。
③利用生成的MeasureSpec参数，调用View#measure方法对子View进行测量。

我们仔细看下getChildMeasureSpec方法的参数，会发现第二个参数是当前View的padding值和child的margin的值，也就是说子View的测量大小受限于父View的padding和子View的margin。

##### ViewGroup#getChildMeasureSpec方法

接着我们来看下ViewGroup测量的核心：ViewGroup#getChildMeasureSpec方法。
我们常说的`子View的测量由父View和子View共同影响`，就是来源于这个方法。

```
// spec是父View的MeasureSpec参数；
// padding的值由父View的padding值和子View的margin值相加而来；
// childDimension是子View的MeasureSpec参数；
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec);
    int specSize = MeasureSpec.getSize(spec);

    int size = Math.max(0, specSize - padding);

    int resultSize = 0;
    int resultMode = 0;

    switch (specMode) {
    // Parent has imposed an exact size on us
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            resultSize = size;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent asked to see how big we want to be
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    //noinspection ResourceType
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

如下图所示：该图表中的内容其实是对getChildMeasureSpec方法的总结。
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190701-162359.png)

##### View#measure
前面说过，绘制完子View后，子View的测量宽高`mMeasuredWidth`和`mMeasuredHeight`会被赋值，接下来我们看下这个过程的源码。

```
View#measure -->
View#onMeasure -->
View#setMeasuredDimension --> 
View#setMeasuredDimensionRaw
```

我们看下View#setMeasuredDimensionRaw源码：

```
private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
    mMeasuredWidth = measuredWidth;
    mMeasuredHeight = measuredHeight;

    mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
}
```

可以很明显的看出，View的`mMeasuredWidth`和`mMeasuredHeight`会被赋值，这样在View测量完成后，我们就可以通过`View#getMeasuredWidth`和`View#getMeasuredHeight`方法获取到View的测量宽高了。

#### onLayout过程
布局的过程从View#layout方法开始.

##### View#layout 

```
public void layout(int l, int t, int r, int b) {
    ...
    // 对当前View进行布局
    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
        // 调用onLayout方法。
        onLayout(changed, l, t, r, b);
        ...
    }
    ...
}
```

这里根据isLayoutModeOptical的值会分别调用setOpticalFrame和setFrame方法，翻看源码可知，setOpticalFrame方法也是调用setFrame方法的，所以不论isLayoutModeOptical的值为true还是false，都会调用View#setFrame方法对当前View进行布局。

接下来会调用View#onLayout方法，通过该方法我们可以确定子View的位置。

通过查看源码可知，View和ViewGroup的onLayout方法都是空实现，而且ViewGroup#onLayout方法还被定义为了抽象方法，强制子类必须实现。

View#onLayout:

```
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```

ViewGroup#onLayout:

```
@Override
protected abstract void onLayout(boolean changed,
        int l, int t, int r, int b);
```

##### View#setFrame

```
protected boolean setFrame(int left, int top, int right, int bottom) {
    boolean changed = false;
    if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
        ...
        mLeft = left;
        mTop = top;
        mRight = right;
        mBottom = bottom;
        mRenderNode.setLeftTopRightBottom(mLeft, mTop, mRight, mBottom);
        ...
    }
    return changed;
}
```

可以看出，通过setFrame方法来设置View的四个顶点的位置，它们的值一旦确定，那么View在父容器中的位置也就确定了。

关于View#onLayout方法的实现，这里我们还是以LinearLayout为例进行说明。

##### LinearLayout#onLayout源码：

```
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        layoutVertical(l, t, r, b);
    } else {
        layoutHorizontal(l, t, r, b);
    }
}
```

我们选择LinearLayout#layoutVertical进行分析：

```
void layoutVertical(int left, int top, int right, int bottom) {
    ...
    for (int i = 0; i < count; i++) {
        final View child = getVirtualChildAt(i);
        if (child == null) {
            childTop += measureNullChild(i);
        } else if (child.getVisibility() != GONE) {
            ...
            setChildFrame(child, childLeft, childTop + getLocationOffset(child),
                    childWidth, childHeight);
            ...
        }
    }
}
```

可以看出，这里循环遍历所有子View，并调用LinearLayout#setChildFrame方法对子View进行布局定位。

##### LinearLayout#setChildFrame

```
private void setChildFrame(View child, int left, int top, int width, int height) {
    child.layout(left, top, left + width, top + height);
}
```
它的实现很简单，就是调用View#layout方法进行定位。

综上所述，View布局的核心方法是View#layout方法，先对父View（当前View）进行布局，然后调用onLayout方法对子View进行布局定位。

#### onDraw过程

View的绘制过程也是从View#draw(Canvas canvas)方法开始的，

```
public void draw(Canvas canvas) {
    ...

    /*
     * Draw traversal performs several drawing steps which must be executed
     * in the appropriate order:
     *
     *      1. Draw the background
     *      2. If necessary, save the canvas' layers to prepare for fading
     *      3. Draw view's content
     *      4. Draw children
     *      5. If necessary, draw the fading edges and restore layers
     *      6. Draw decorations (scrollbars for instance)
     */

    // Step 1, draw the background, if needed
    if (!dirtyOpaque) {
        drawBackground(canvas);
    }
    ...

    // Step 2, save the canvas' layers
    ...
    saveCount = canvas.getSaveCount();

    // Step 3, draw the content
    if (!dirtyOpaque) onDraw(canvas);

    // Step 4, draw the children
    dispatchDraw(canvas);

    // Step 5, draw the fade effect and restore layers
    ...

    canvas.restoreToCount(saveCount);
    drawAutofilledHighlight(canvas);

    // Step 6, draw decorations (foreground, scrollbars)
    onDrawForeground(canvas);

}
```

如上可知，绘制流程按照以下六个步骤执行：
①绘制背景
②如果需要，保存图层
③绘制当前View的内容
④绘制子View，具体是dispatchDraw方法
⑤如果需要，绘制边界，恢复图层
⑥绘制相关装饰（比如滚动条）

不出意外，dispatchDraw方法在View中是空实现，如下所示：

```
protected void dispatchDraw(Canvas canvas) {

}
```

##### ViewGroup#dispatchDraw

我们看下dispatchDraw的具体实现，是在ViewGroup当中：

```
    @Override
    protected void dispatchDraw(Canvas canvas) {
        ...
        for (int i = 0; i < childrenCount; i++) {
            ...
            if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {
                more |= drawChild(canvas, child, drawingTime);
            }
        }
        ...
    }

```

这里也是循环遍历每个子View，然后调用ViewGroup#drawChild方法对每个子View进行绘制。

我们看下ViewGroup#drawChild的实现：

```
protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
    return child.draw(canvas, this, drawingTime);
}
```
实现很简单，就是调用View#draw方法对每个子View进行绘制。

总的来说，View绘制的核心方法是View#draw方法，先对父View（当前View）进行绘制，然后调用dispatchDraw方法对子View进行绘制。


#### 总结
1、测量过程是先测量子View，再测量父View（当前View），因为父View的宽高需要用到子View的测量结果。
2、View布局的核心方法是View#layout方法，先对父View（当前View）进行布局，然后调用onLayout方法对子View进行布局定位。
3、View绘制的核心方法是View#draw方法，先对父View（当前View）进行绘制，然后调用dispatchDraw方法对子View进行绘制。

#### 参考

1、[图解View测量、布局及绘制原理](https://www.jianshu.com/p/3d2c49315d68)

2、[View的测量、布局和绘制过程中的关键方法](https://blog.csdn.net/qq_26287435/article/details/94391111)


