---
layout: post
title:  View#invalidate方法是如何更新UI的
date:   2019-07-01 09:56:06 +0800
categories: View
tag: [View#invalidate]
---

* content
{:toc}




### View#invalidate方法是如何更新UI的

这里拿TextView#setText 举一个例子：

```
TextView.setText()  —> 
TextView#checkForRelayout  —> 
View#invalidate —> 
View#invalidateInternal -> 
ViewParent#invalidateChild  —> 
ViewGroup#invalidateChild do-while循环调用 —> 
ViewRootImpl#invalidateChildInParent —> 
viewRootImpl#checkThread 和
viewRootImpl#invalidate —> 
viewRootImpl#scheduleTraversals —> 
mChoreographer(内部维护了一个FrameHandler).postCallback(mTraversalRunnable….) —> 
doTraversal -> 
ViewRootImpl#doTraversal --> 
ViewRootImpl#performTraversals 就会调用 onMeasure onLayout onDraw。
```

从整个调用链来看，应该会将刷新视图事件包装成一个Runable，添加至MessageQueue中，Looper轮询取出，Handler#handleMessage接收到绘制事件回调并调用viewRootImpl#performTraversals完成最终绘制.


