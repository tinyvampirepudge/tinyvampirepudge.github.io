---
layout: post
title:  View事件分发相关结论的源码解析
date:   2019-07-28 11:25:12 +0800
categories: View事件分发
tag: [View事件分发]
---

* content
{:toc}



### View事件分发相关结论的源码解析

了解过View事件分发源码的同学或多或少都知道一些事件分发的相关结论，比如`某个View如果拦截了事件，那么它的onTouchEventIntercept方法就不会再次调用`，比如`事件如果被某个View消耗掉，那么该序列中的剩余事件都将交给该View处理`等等。

View事件分发的三个核心方法有三个，分别是`dispatchTouchEvent`方法，`onInterceptTouchEvent`方法和`onInterceptTouchEvent`方法。

dispatchTouchEvent方法主要用来进行事件的分发。如果事件能够传递给当前View，那么此方法一定会被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

onInterceptTouchEvent方法在dispatchTouchEvent方法内部调用，用来判断是否拦截某个事件，返回结果表示是否拦截当前事件。

onTouchEvent方法也在dispatchTouchEvent方法中调用，返回结果表示是否消耗当前事件。

#### View事件分发相关结论

今天打算针对其中两个最有用、最难记和最难理解的结论，从源码层面给出它们的解释，结论如下：

结论一、某个ViewGroup一旦决定拦截（ACTINON_DOWN: onInterceptTouchEvent的返回值为true），那么这一个事件序列都只能由它来处理（如果事件序列能够传递给它的话），并且它的onInterceptTouchEvent不会再被调用。

就是说当一个ViewGroup决定拦截一个事件后，那么系统会把同一个事件序列内的其他方法都直接交给它来处理，因此就不再调用这个View的onInterceptTouchEvent去询问它是否要拦截了。

结论二、某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回了false），那么同一事件序列中的其他事件都不会再交给它来处理，并且事件将重新交由它的父元素去处理，即父元素的onTouchEvent会被调用。

View事件分发其他的结论，本文这里暂不做分析，读者可自行了解。

#### 使用例子说明

##### 示例一：不做逻辑修改，只打印log
我们自定义三个View，分别继承自RelativeLayout、LinearLayout、TextView，分别重写他们的`dispatchTouchEvent`方法，`onInterceptTouchEvent`方法（这个方法是ViewGroup特有的，View没有）和`onInterceptTouchEvent`方法。

在这三个方法内部，不做任何逻辑修改，只依次打印ACTION_DOWN、ACTION_MOVE、ACTION_UP的log。具体代码如下所示：

```
public class MyLinearLayout extends LinearLayout {
    private static final String TAG = MyLinearLayout.class.getSimpleName();

    public MyLinearLayout(Context context) {
        super(context);
    }

    public MyLinearLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyLinearLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int action = event.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                LogUtils.e(TAG + " dispatchTouchEvent ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                LogUtils.e(TAG + " dispatchTouchEvent ACTION_MOVE");
                break;
            case MotionEvent.ACTION_UP:
                LogUtils.e(TAG + " dispatchTouchEvent ACTION_UP");
                break;
            default:
                break;
        }
        return super.dispatchTouchEvent(event);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                LogUtils.e(TAG + " onInterceptTouchEvent ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                LogUtils.e(TAG + " onInterceptTouchEvent ACTION_MOVE");
                break;
            case MotionEvent.ACTION_UP:
                LogUtils.e(TAG + " onInterceptTouchEvent ACTION_UP");
                break;
            default:
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int action = event.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                LogUtils.e(TAG + " onTouchEvent ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                LogUtils.e(TAG + " onTouchEvent ACTION_MOVE");
                break;
            case MotionEvent.ACTION_UP:
                LogUtils.e(TAG + " onTouchEvent ACTION_UP");
                break;
            default:
                break;
        }
        return super.onTouchEvent(event);
    }
}
```

MyRelativeLayout的代码和MyLinearLayout完全一致，MyTextView稍有不同，因为它没有`onInterceptTouchEvent`方法。

接着在布局中使用这三个自定义View，最外侧是MyRelativeLayout，子View是一个MyLinearLayout，再内部是MyTextView。具体代码如下所示：

```
<com.tiny.demo.firstlinecode.view.dispatchevent.summary.MyRelativeLayout
            android:layout_width="match_parent"
            android:layout_height="300dp"
            android:background="#BCEAC1"
            android:contentDescription="MyLinearLayout">

            <com.tiny.demo.firstlinecode.view.dispatchevent.summary.MyLinearLayout
                android:layout_width="match_parent"
                android:layout_height="200dp"
                android:layout_centerInParent="true"
                android:background="#2DC98F"
                android:contentDescription="MyLinearLayout"
                android:orientation="vertical">

                <com.tiny.demo.firstlinecode.view.dispatchevent.summary.MyTextView
                    android:layout_width="200dp"
                    android:layout_height="100dp"
                    android:layout_gravity="center"
                    android:layout_marginTop="50dp"
                    android:background="#839885"
                    android:gravity="center"
                    android:text="MyTextView" />

            </com.tiny.demo.firstlinecode.view.dispatchevent.summary.MyLinearLayout>

        </com.tiny.demo.firstlinecode.view.dispatchevent.summary.MyRelativeLayout>
```

效果如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190726-232654%402x.png)

然后我们点击MyTextView区域，输出log如下：

```
MyRelativeLayout dispatchTouchEvent ACTION_DOWN
MyRelativeLayout onInterceptTouchEvent ACTION_DOWN
MyLinearLayout dispatchTouchEvent ACTION_DOWN
MyLinearLayout onInterceptTouchEvent ACTION_DOWN
MyTextView dispatchTouchEvent ACTION_DOWN
MyTextView onTouchEvent ACTION_DOWN
MyLinearLayout onTouchEvent ACTION_DOWN
MyRelativeLayout onTouchEvent ACTION_DOWN
```

可以看出，在不做任何处理的情况下，事件是从父View依次向子View传递的。会先调用父View的dispatchTouchEvent方法和onInterceptTouchEvent方法，接着调用子View的dispatchTouchEvent方法和onInterceptTouchEvent方法，依次递归，直到到达最底层View。

达到最底层View后，会调用最底层View的dispatchTouchEvent方法和onTouchEvent方法，接着事件又依次向上传递，直到被消费掉，如果没有被消费，则会传递给最外层View。

在我们这个例子中，具体传递就如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190727-154811%402x.png)

需要说明的是，我们这个例子中只打印了`ACTION_DOWN`的相关log，没有打印`ACTION_MOVE`和`ACTION_UP`的，不会因为我偷懒省略，而是因为在事件传递过程中没有View拦截`ACTION_DOWN`  事件，也没有View消费`ACTION_DOWN`事件，则同一事件序列中的其他事件都不会传递给这些View来进行处理了，都会交由父View的onTouchEvent方法进行处理。

这个例子很好的说明了`结论二`，至于具体源码我们后续再看。

##### 示例二：修改MyLinearLayout，让其拦截并消耗事件。

我们回顾下结论一，`某个ViewGroup一旦决定拦截（ACTINON_DOWN: onInterceptTouchEvent的返回值为true），那么这一个事件序列都只能由它来处理（如果事件序列能够传递给它的话），并且它的onInterceptTouchEvent不会再被调用`。

所以我们在示例一的基础上，修改MyLinearLayout类，让其拦截并消耗ACTINON_DOWN事件。

具体来说就是在MyLinearrLayout#onInterceptTouchEvent中，当MotionEvent为ACTION_DOWN时，就返回true；在MyLinearLayout#onTouchEvent方法返回true。具体代码如下所示：

```
public class MyLinearLayout extends LinearLayout {
    ...

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                LogUtils.e(TAG + " onInterceptTouchEvent ACTION_DOWN");
                return true;
            case MotionEvent.ACTION_MOVE:
                LogUtils.e(TAG + " onInterceptTouchEvent ACTION_MOVE");
                break;
            case MotionEvent.ACTION_UP:
                LogUtils.e(TAG + " onInterceptTouchEvent ACTION_UP");
                break;
            default:
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int action = event.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                LogUtils.e(TAG + " onTouchEvent ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                LogUtils.e(TAG + " onTouchEvent ACTION_MOVE");
                break;
            case MotionEvent.ACTION_UP:
                LogUtils.e(TAG + " onTouchEvent ACTION_UP");
                break;
            default:
                break;
        }
        return true;
    }
}
```

布局不变，跟上个例子一致，然后我们点击MyTextView区域，输出log如下：

```
MyRelativeLayout dispatchTouchEvent ACTION_DOWN
MyRelativeLayout onInterceptTouchEvent ACTION_DOWN
MyLinearLayout dispatchTouchEvent ACTION_DOWN
MyLinearLayout onInterceptTouchEvent ACTION_DOWN
MyLinearLayout onTouchEvent ACTION_DOWN
MyRelativeLayout dispatchTouchEvent ACTION_MOVE
MyRelativeLayout onInterceptTouchEvent ACTION_MOVE
MyLinearLayout dispatchTouchEvent ACTION_MOVE
MyLinearLayout onTouchEvent ACTION_MOVE
MyRelativeLayout dispatchTouchEvent ACTION_UP
MyRelativeLayout onInterceptTouchEvent ACTION_UP
MyLinearLayout dispatchTouchEvent ACTION_UP
MyLinearLayout onTouchEvent ACTION_UP
```

可以看出事件没有传递给子View MyTextView，而是被MyLinearLayout#onInterceptTouchEvent方法拦截，接着被MyLinearLayout#onTouchEvent方法消费了。

接着同一时间序列中剩下的`ACTION_MOVE`、`ACTION_UP`事件传递给了MyLinearLayout，也由其消费掉了。

另外，MyLinearLayout#onInterceptTouchEvent只被调用了一次，后续的`ACTION_MOVE`、`ACTION_UP`事件到来时均没有再次调用onInterceptTouchEvent方法。

这个例子完美的展示了我们`结论一`的现象，具体过程如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190727-161128%402x.png)

#### 相关源码解读
例子演示完毕，接下来我们从源码角度对这两个结论进行解析。

我们知道，ViewGroup的onInterceptTouchEvent方法默认返回false，也就是不拦截。ViewGroup中没有重写`onTouchEvent`方法，它使用的是View中的`onTouchEvent`方法。

通过查看常用的LinearLayout、RealtiveLayout、FrameLayout的源码可以发现，它们均没有重写`dispatchTouchEvent`、`onInterceptTouchEvent`、`onTouchEvent`这三个方法，也就是说他们中的事件分发都是遵循ViewGroup中的规则。

源码基于android-28。

##### 结论一解读

为了简单起见，我们分段解读ViewGroup#dispatchTouchEvent方法源码代码。

跟事件序列保持一致，我们依次分析MotionEvent.ACTION_DOWN、MotionEvent.ACTION_MOVE、MotionEvent.ACTION_UP事件。

###### MotionEvent.ACTION_DOWN流程
为了方便阅读，我们将核心代码分为了四部分：

part1:

```
// 如果是ACTION_DOWN事件的话，就重置状态，包括将mFirstTouchTarget置为null
if (actionMasked == MotionEvent.ACTION_DOWN) {
    cancelAndClearTouchTargets(ev);
    resetTouchState();
}
```

上述这两个方法内部都会调用clearTouchTargets方法，该方法会将mFirstTouchTarget置为null。

何为mFirstTouchTarget？
当子View成功处理事件时，mFirstTouchTarget会被赋值并指向子元素。

接下来看下是否拦截的逻辑，这里调用了onInterceptTouchEvent方法。

part2:

```
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    intercepted = true;
}
```

先看外层的if判断，当是ACTION_DOWN事件或者mFirstTouchTarget不为空时，就会走if内部逻辑，否则就给intercepted赋值为true，表示拦截事件。我们这次是ACTION_DOWN事件，所以就会走if内部逻辑。

我们再看if为true时的内部逻辑：
先判断是否设置了`FLAG_DISALLOW_INTERCEPT`标记位，默认不设置。该标记位通过`requestDisallowInterceptTouchEvent`方法设置，这跟我们这次分析关系不大，暂且不关注。

disallowIntercept的值默认为false，所以就走调用子if内部的逻辑，会调用我们的`onInterceptTouchEvent`方法，该方法默认返回值为false，表示不拦截。

而我们重写了MyLinearLayout的onInterceptTouchEvent方法，所以我们最终的intercepted的值为true。

接着往下看part3:

```
// 我们intercepted的值为true，所以不会走下面的if内部逻辑。
if (!canceled && !intercepted) {
    ...
    
    // 如果是ACTION_DOWN事件
    if (actionMasked == MotionEvent.ACTION_DOWN
            || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
            || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
        ...
        final int childrenCount = mChildrenCount;
        
        if (newTouchTarget == null && childrenCount != 0) {
            ...
            final View[] children = mChildren;
            
            // 开启for循环，依次遍历所有子View
            for (int i = childrenCount - 1; i >= 0; i--) {
                ...
                
                // 调用dispatchTransformedTouchEvent方法进行事件分发。
                // dispatchTransformedTouchEvent方法如果返回true，表示事件被消费掉了。
                if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                    ...
                    
                    // 在addTouchTarget方法内部，会给mFirstTouchTarget赋值。
                    newTouchTarget = addTouchTarget(child, idBitsToAssign);
                    alreadyDispatchedToNewTouchTarget = true;
                    break;
                }
                ...
            }
            ...
        }
        ...
    }
}
```
此时intercepted的值为true，所以不会走下面的if内部逻辑。也就不会给mFirstTouchTarget赋值，mFirstTouchTarget的值依然为null。

接着看，part4:

```
// Dispatch to touch targets.
// mFirstTouchTarget的值为空，所以会走if内部的逻辑，会调用dispatchTransformedTouchEvent方法，不过这次的child参数为null。
if (mFirstTouchTarget == null) {
    // No touch targets so treat this as an ordinary view.
    handled = dispatchTransformedTouchEvent(ev, canceled, null,
            TouchTarget.ALL_POINTER_IDS);
} else {
    ...
}
```
这里mFirstTouchTarget为null，所以会调用dispatchTransformedTouchEvent方法，且child参数值为null。

我们这里看下ViewGroup#dispatchTransformedTouchEvent方法：

```
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
        View child, int desiredPointerIdBits) {
    ...

    // Perform any necessary transformations and dispatch.
    // 判断child参数是否为空
    if (child == null) {
        // 调用View#dispatchTouchEvent方法。
        handled = super.dispatchTouchEvent(transformedEvent);
    } else {
        ...
        // 将事件传递给子View。
        handled = child.dispatchTouchEvent(transformedEvent);
    }
    ...
    return handled;
}
```

上述代码可以看出，在dispatchTransformedTouchEvent方法内部，如果child不为空，就会调用child.dispatchTouchEvent方法，将事件传递给child。

如果child参数为空，就调用super.dispatchTouchEvent(transformedEvent)方法，也就是View#dispatchTouchEvent方法进行事件分发，在其内部就会调用当前ViewGroup的onTouchEvent方法进行事件处理。

我们这里Child为null，所以会调用View#dispatchTouchEvent方法，在其内部会调用onTouchEvent方法。因为我们重写了MyLinearLayout#onTouchEvent方法，让其返回值为true，表示事件被消费掉，所以事件就不会再向上传递了。

上面分析过，dispatchTransformedTouchEvent方法中的child参数如果为null的话，就会调用当前View#dispatchTouchEvent方法。

我们看下View的dispatchTouchEvent方法，源码如下：

```
public boolean dispatchTouchEvent(MotionEvent event) {
    ...
    boolean result = false;

    ...

    if (onFilterTouchEventForSecurity(event)) {
        ...
        //noinspection SimplifiableIfStatement
        
        // 如果设置了OnTouchListener，就调用mOnTouchListener#onTouch方法。
        // 如果mOnTouchListener#onTouch方法返回值为true，则result的值为true。
        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnTouchListener != null
                && (mViewFlags & ENABLED_MASK) == ENABLED
                && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }

        // 如果result为false，则调用onTouchEvent方法。
        // 如果onTouchEvent方法返回值为true，则result赋值为true，表示事件被处理。
        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }

    ...

    return result;
}
```

我们没有给MyLinearLayout设置OnTouchListener，所以MyLinearLayout的onTouchEvent方法会执行。

通过上面的分析，我们可以解释ACTION_DOWN的相关log了：

```
MyRelativeLayout dispatchTouchEvent ACTION_DOWN
MyRelativeLayout onInterceptTouchEvent ACTION_DOWN
MyLinearLayout dispatchTouchEvent ACTION_DOWN
MyLinearLayout onInterceptTouchEvent ACTION_DOWN
MyLinearLayout onTouchEvent ACTION_DOWN
```

事件传递给MyRelativeLayout，先调用MyRelativeLayout#dispatchTouchEvent方法，接着将MyRelativeLayout#onInterceptTouchEvent方法的返回值（false）赋值给intercepted，然后调用dispatchTransformedTouchEvent方法（child不为空），将事件分发给子View MyLinearLayout。

在MyLinearLayout内部也是先调用MyLinearLayout#dispatchTouchEvent方法，接着将MyLinearLayout#onInterceptTouchEvent方法的返回值（true）赋值给intercepted，由于这里进行了拦截，所以事件不会分发给子View(`part3部分`)，所以mFirstTouchTarget也为null，所以会调用dispatchTransformedTouchEvent方法（child参数值为null, `part4部分`），接着调用MyLinearLayout#onTouchEvent方法，该方法也被重写了，返回值为true。

由于MyLinearLayout#onTouchEvent方法返回true，所以MyLinearLayout#dispatchTouchEvent方法也就返回true，所以在MyRelativeLayout#dispatchTouchEvent方法中，将事件分发给子View时(`part3部分`)，dispatchTransformedTouchEvent方法的返回值为true，所以mFirstTouchEvent的值不为null，所以事件不会再次向上传递给MyRelativeLayout了。

做下总结，在ACTION_DOWN事件被MyLinearLayout处理完之后，`mFirstTouchTarget`的值为null。

###### MotionEvent.ACTION_MOVE流程
ACTION_MOVE事件到来时，会依旧走一遍dispatchTouchEvent方法。我们依旧按照part1、part2、part3、part4的顺序来看。

part1:

```
// 如果是ACTION_DOWN事件的话，就重置状态，包括将mFirstTouchTarget置为null
if (actionMasked == MotionEvent.ACTION_DOWN) {
    cancelAndClearTouchTargets(ev);
    resetTouchState();
}
```
这次传递的事ACTION_MOVE事件，所以这个逻辑不会走.

part2:

```
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    intercepted = true;
}
```

这里我们是ACTION_MOVE事件，且`mFirstTouchTarget`为null，所以直接走的是外层if的else逻辑，即`intercepted = true`，所以这里不会调用onInterceptTouchEvent方法。

接着往下看,part3:

```
// 此时intercepted的值为true，所以不会走下面的if内部逻辑。
if (!canceled && !intercepted) {
    ...
    
    // 如果是ACTION_DOWN事件
    if (actionMasked == MotionEvent.ACTION_DOWN
            || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
            || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
        ...
        final int childrenCount = mChildrenCount;
        
        if (newTouchTarget == null && childrenCount != 0) {
            ...
            final View[] children = mChildren;
            
            // 开启for循环，依次遍历所有子View
            for (int i = childrenCount - 1; i >= 0; i--) {
                ...
                
                // 调用dispatchTransformedTouchEvent方法进行事件分发。
                // dispatchTransformedTouchEvent方法如果返回true，表示事件被消费掉了。
                if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                    ...
                    
                    // 在addTouchTarget方法内部，会给mFirstTouchTarget赋值。
                    newTouchTarget = addTouchTarget(child, idBitsToAssign);
                    alreadyDispatchedToNewTouchTarget = true;
                    break;
                }
                ...
            }
            ...
        }
        ...
    }
}
```
此时intercepted的值为true，所以不会走下面的if内部逻辑。

接着看，part4:

```
// Dispatch to touch targets.
// mFirstTouchTarget的值为空，所以会走if内部的逻辑，会调用dispatchTransformedTouchEvent方法，不过这次的child参数为null。
if (mFirstTouchTarget == null) {
    // No touch targets so treat this as an ordinary view.
    handled = dispatchTransformedTouchEvent(ev, canceled, null,
            TouchTarget.ALL_POINTER_IDS);
} else {
    ...
}
```
这里会调用onTouchEvent方法，将其返回值赋给handled。

我们分析下ACTION_MOVE相关log：

```
MyRelativeLayout dispatchTouchEvent ACTION_MOVE
MyRelativeLayout onInterceptTouchEvent ACTION_MOVE
MyLinearLayout dispatchTouchEvent ACTION_MOVE
MyLinearLayout onTouchEvent ACTION_MOVE
```

事件传递给MyRelativeLayout，先调用MyRelativeLayout#dispatchTouchEvent方法，接着将MyRelativeLayout#onInterceptTouchEvent方法的返回值（false）赋值给intercepted，然后调用dispatchTransformedTouchEvent方法（child不为空），将事件分发给子View MyLinearLayout。

在MyLinearLayout内部也是先调用MyLinearLayout#dispatchTouchEvent方法，接着由于不满足`part2`中if判断的条件，所以intercepted的值为true，也就没有调用MyLinearLayout#onInterceptTouchEvent方法。

由于intercepted值为true，所以不会走`part3`的逻辑，所以mFirstTouchTarget的值为null。

接着由于mFirstTouchTarget值为null，所以在`part4`中会调用MyLinearLayout#dispatchTransformedTouchEvent方法（child为null），将事件传递给MyLinearLayout#onTouchEvent方法。

由于我们重写了MyLinearLayout#onTouchEvent方法，让其返回值为true，所以MyLinearLayout#dispatchTouchEvent方法也就返回true，所以在MyRelativeLayout#dispatchTouchEvent方法中，将事件分发给子View时，dispatchTransformedTouchEvent方法的返回值为true，所以MyRelativeLayout#mFirstTouchEvent的值不为null，所以事件不会再次向上传递给MyRelativeLayout了。

###### MotionEvent.ACTION_UP流程

ACTION_UP流程与ACTION_MOVE流程完全一致，这里不再赘述。

通过以上分析，我们从源码角度解释了结论一：`某个ViewGroup一旦决定拦截（ACTINON_DOWN: onInterceptTouchEvent的返回值为true），那么这一个事件序列都只能由它来处理（如果事件序列能够传递给它的话），并且它的onInterceptTouchEvent不会再被调用`。

##### 结论二解读

这里我们也按照MotionEvent.ACTION_DOWN、MotionEvent.ACTION_MOVE、MotionEvent.ACTION_UP事件的顺序分析。

###### MotionEvent.ACTION_DOWN流程
part1:

```
// 如果是ACTION_DOWN事件的话，就重置状态，包括将mFirstTouchTarget置为null
if (actionMasked == MotionEvent.ACTION_DOWN) {
    cancelAndClearTouchTargets(ev);
    resetTouchState();
}
```
当前为ACTION_DOWN事件，所以会走if内部逻辑，会依次调用`cancelAndClearTouchTargets`和`resetTouchState`方法。

上述这两个方法内部都会调用`clearTouchTargets`方法，该方法会将mFirstTouchTarget置为null。

接下来看part2:

```
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    intercepted = true;
}
```

先看外层的if判断，我们这次是ACTION_DOWN事件，所以就会走if内部逻辑。

我们再看if为true时的内部逻辑：
我们disallowIntercept的值默认为false，所以就会调用子if内部的逻辑，会调用我们的`onInterceptTouchEvent`方法，该方法默认返回值为false，表示不拦截。

所以我们最终的intercepted的值为false。

接着往下看，part3：

```
// 我们intercepted的值我为false，所以会走下面的if内部逻辑。
if (!canceled && !intercepted) {
    ...
    
    // 我们是ACTION_DOWN事件，所以下面的条件也满足
    if (actionMasked == MotionEvent.ACTION_DOWN
            || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
            || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
        ...
        final int childrenCount = mChildrenCount;
        
        // 我们的MyLinearLayout是有子View的，所以下面的条件也满足
        if (newTouchTarget == null && childrenCount != 0) {
            ...
            final View[] children = mChildren;
            
            // 开启for循环，依次遍历所有子View
            for (int i = childrenCount - 1; i >= 0; i--) {
                ...
                
                // 调用dispatchTransformedTouchEvent方法进行事件分发。
                // dispatchTransformedTouchEvent方法如果返回true，表示事件被消费掉了。
                // 这里的dispatchTransformedTouchEvent方法返回值为false，表示子View不处理事件。
                if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                    ...
                    
                    // 在addTouchTarget方法内部，会给mFirstTouchTarget赋值。
                    newTouchTarget = addTouchTarget(child, idBitsToAssign);
                    alreadyDispatchedToNewTouchTarget = true;
                    break;
                }
                ...
            }
            ...
        }
        ...
    }
}
```

上述代码可以看出，在dispatchTransformedTouchEvent方法内部，调用了child.dispatchTouchEvent方法，将事件传递给child。

而在我们的例子中，MyLinearLayout的子View为MyTextView，我们知道TextView的onTouchEvent方法默认返回的是false，所以我们这里的dispatchTransformedTouchEvent方法的返回值为false。

所以我们这里没有子View处理事件，我们的mFirstTouchTarget的值为null。

接着看part4：

```
// Dispatch to touch targets.
// mFirstTouchTarget的值为空，所以会走if内部的逻辑，会再次调用dispatchTransformedTouchEvent方法，不过这次的child参数为null。
if (mFirstTouchTarget == null) {
    // No touch targets so treat this as an ordinary view.
    handled = dispatchTransformedTouchEvent(ev, canceled, null,
            TouchTarget.ALL_POINTER_IDS);
} else {
    ...
}
```

上面分析过，dispatchTransformedTouchEvent方法中的child参数如果为null的话，就会调用当前View#dispatchTouchEvent方法，进而调用View#onToucheEvent方法。

我们没有给MyLinearLayout设置OnTouchListener，所以MyLinearLayout的onTouchEvent方法会执行。

所以我们可以解释ACTION_DOWN相关的log了：

```
MyRelativeLayout dispatchTouchEvent ACTION_DOWN
MyRelativeLayout onInterceptTouchEvent ACTION_DOWN
MyLinearLayout dispatchTouchEvent ACTION_DOWN
MyLinearLayout onInterceptTouchEvent ACTION_DOWN
MyTextView dispatchTouchEvent ACTION_DOWN
MyTextView onTouchEvent ACTION_DOWN
MyLinearLayout onTouchEvent ACTION_DOWN
MyRelativeLayout onTouchEvent ACTION_DOWN
```

事件传递给MyRelativeLayout，先调用MyRelativeLayout#dispatchTouchEvent方法，接着将MyRelativeLayout#onInterceptTouchEvent方法的返回值（false）赋值给intercepted，然后调用dispatchTransformedTouchEvent方法，将事件分发给子View MyLinearLayout。

在MyLinearLayout内部也是先调用MyLinearLayout#dispatchTouchEvent方法，接着将MyLinearLayout#onInterceptTouchEvent方法的返回值（false）赋值给intercepted（`part2部分`），然后调用dispatchTransformedTouchEvent方法(`part3部分`)，将事件分发给子View MyTextView。

在MyTextView内部也是先调用MyTextView#dispatchTouchEvent方法，接着会调用MyTextView#onTouchEvent方法，因为MyTextView继承自TextView，其`CLICKABLE`和`LONG_CLICKABLE`属性默认均为false，所以MyTextView#onTouchEvent方法返回值为false。表示事件没有被MyTextView消费掉，所以会向上传递给父View。

接着事件会向上传递给MyLinearLayout（`part4部分`），调用super.dispatchTouchEvent，也就是View#dispatchTouchEvent方法，在该方法内部会调用onTouchEvent方法。不过跟MyTextView一样，这个onTouchEvent方法也返回false。所以事件会继续向上传递。

接着事件会向上传递给MyRelativeLayout，该过程和MyLinearLayout过程一样。

总结一下，由于没有子View消耗掉ACTION_DOWN事件，当前的mFirstTouchTarget值为null。

###### MotionEvent.ACTION_MOVE流程

接下来分析ACTION_MOVE事件传递的过程，还是ViewGroup#dispatchTouchEvent方法：

part1：

```
// 如果是ACTION_DOWN事件的话，就重置状态，包括将mFirstTouchTarget置为null
if (actionMasked == MotionEvent.ACTION_DOWN) {
    cancelAndClearTouchTargets(ev);
    resetTouchState();
}
```
当前为ACTION_MOVE事件，所以不会走if内部逻辑。

接着看part2：

```
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
        || mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    // There are no touch targets and this action is not an initial down
    // so this view group continues to intercept touches.
    intercepted = true;
}
```

由于这次传递的是ACTION_MOVE事件，且经过ACTION_MOVE事件后，mFirstTouchTarget的值依然为null，所以外层if的条件不满足，直接会走else分支，此时intercepted的值为true。

由于不走if分支，所以我们的onInterceptTouchEvent方法也不会被再次调用。

接下来看part3：

```
if (!canceled && !intercepted) {
    ...
}
```
if条件语句也不满足，内部逻辑不会走，也就不会下发事件给子View了。

再往下看part4：

```
// Dispatch to touch targets.
if (mFirstTouchTarget == null) {
    // No touch targets so treat this as an ordinary view.
    handled = dispatchTransformedTouchEvent(ev, canceled, null,
            TouchTarget.ALL_POINTER_IDS);
} else {
   ...
}
```

由于我们的mFirstTouchTarget的值为null，所以会调用这个dispatchTransformedTouchEvent方法。由于这里传递的child参数为null，所以实际上调用的事当前ViewGroup的onTouchEvent方法。

可以看出ACTION_MOVE事件并没有向下分发给子View，而是在调用当前View的onTouchEvent方法后，统一交给父View处理了。

我们知道事件来源于`DecorView`，它继承自FrameLayout，所以这里的ACTION_MOVE事件默认就交给DecorView处理了，没有向下下发，所以我们的三个自定义View没有收到出ACTION_DOWN以外的事件。

###### MotionEvent.ACTION_UP流程

这种情况下ACTION_UP的分析与ACTION_MOVE完全相同，这里不再赘述。

至此相关源码已经分析完毕，有兴趣的同学，可以写一个自定义的Button，替换掉MyTextView，看下现象是不是有所不同，也可以尝试从源码角度解释。

#### 参考

1、Android开发艺术探索（Android开发神书，不解释）




