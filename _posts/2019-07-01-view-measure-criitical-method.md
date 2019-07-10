---
layout: post
title:  View的测量、布局和绘制过程中的关键方法 
date:   2019-07-01 14:42:00 +0800
categories: View
tag: [View的测量、布局和绘制]
---

* content
{:toc}

### View的测量、布局和绘制过程中的关键方法 
我们这里说的View的测量、布局和绘制，实质上是针对ViewGroup的，简单起见就不区分View和ViewGroup。View的测量、布局和绘制是包含在ViewGroup流程中的。

#### 流程概览
View的测量、布局、绘制是从`ViewRootImpl`开始的，它是Activity中的根View。具体是在`ViewRootImpl#performTraversals`方法中依次调用performMeasure、performLayout、performDraw。

我们常见的更新UI的方法`View#invalidate`最终就是通过调用ViewRootImpl#performTraversals方法来完成的，具体参见[View#invalidate方法是如何更新UI的](https://blog.csdn.net/qq_26287435/article/details/94380452)。

如下图所示：

![ViewRootImpl#performTraversals图解](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190701-110402.png)

本文的源码基于`android-28`的api。

#### Measure流程

Measure的具体流程如下：

```
ViewRootImpl#performTreversals() -->
ViewRootImpl#performMeasure() -->
View#measure() --> 
View#onMeasure
```

```
如果是ViewGroup，一般会重写View#onMeasure方法，会在ViewGroup#onMeasure方法中循环调用子View的measure方法，如此循环往复，直到最终调用了所有的View#onMeasure方法。
```

#### Layout流程
Layout的具体流程如下：

```
ViewRootImpl#performTreversals() -->
ViewRootImpl#performLayout() -->
View#layout() --> 
View#onLayout()
```

View#onLayout方法默认空实现，如下所示，
```
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```
所以onLayout方法需要View根据需求各自实现。

```
与onMeasure方法类似，ViewGroup一般会重写View#onLayout方法。
然后在ViewGroup#onLayout方法中循环调用子View的layout方法，如此循环往复，直到最终调用了所有的View#onLayout方法。
```

这里以LinearLayout为例， 

```
LinearLayout#onLayout -->
Linearlayout#layoutVertical -->
循环遍历每个子View，调用Linearlayout#setChildFrame -->
最终调用View#layout方法。
```

#### Draw流程
Draw的具体流程如下：

```
ViewRootImpl#performTreversals() -->
ViewRootImpl#performDraw() -->
ViewRootImpl#draw(boolean fullRedrawNeeded) --> 
View#drawSoftware -->
View#draw(Canvas canvas) -->
绘制当前View：View#onDraw(Canvas canvas) --> 
绘制子View：View#dispatchDraw(Canvas canvas)
```

在`View#draw(Canvas canvas)`方法中，有如下注释：

```
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
```

绘制流程按照以下六个步骤执行：
①绘制背景
②如果需要，保存图层
③绘制当前View的内容
④绘制子View
⑤如果需要，绘制边界，恢复图层
⑥绘制相关装饰（比如滚动条）

对应的源码为：为了方便阅读，省略了无关代码。

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

