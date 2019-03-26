---
layout: post
title:  RecyclerView#smoothScrollToPosition调用RecyclerView.OnScrollListener的过程
date:   2019-03-26 23:07:21 +0800
categories: RecyclerView
tag: [RecyclerView.OnScrollListener]
---

* content
{:toc}



### RecyclerView#smoothScrollToPosition调用RecyclerView#OnScrollListener的过程

项目中使用到了RecyclerView#smoothScrollToPosition(0)方法让Recyclerview滚动到顶部，同时给Recyclerview设置了监听器RecyclerView.OnScrollListener，代码如下所示：

```
recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
    
    @Override
    public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
        super.onScrolled(recyclerView, dx, dy);
    }
});

recyclerView.smoothScrollToPosition(0);
```

这里想简单聊下代码调用的过程。

```
RecyclerView#addOnScrollListener():
实现如下：
public void addOnScrollListener(OnScrollListener listener) {
    if (mScrollListeners == null) {
        mScrollListeners = new ArrayList<>();
    }
    mScrollListeners.add(listener);
}
    
    
将监听器添加进RecyclerView的成员变量List<OnScrollListener> mScrollListeners中。
```

我们看下使用mScrollListeners的值的地方，有两处：

```
RecyclerView#dispatchOnScrolled

RecyclerView#dispatchOnScrollStateChanged
```

好，接下来我们看下recyclerView.smoothScrollToPosition()是如何调用到我们刚才添加的OnScrollListener监听器的：

```
RecyclerView#smoothScrollToPosition(int position) 
-->

RecyclerView.LayoutManager#smoothScrollToPosition(RecyclerView recyclerView, State state, int position)

--> 实现类

LinearLayoutManager#smoothScrollToPosition(RecyclerView recyclerView, State state, int position)

-->

LinearLayoutManager#startSmoothScroll(SmoothScroller smoothScroller)

-->

mRecyclerView.mViewFlinger.postOnAnimation();

-->

Recyclerview.SmoothScroller#start(RecyclerView recyclerView, LayoutManager layoutManager)

-->

mRecyclerView.mViewFlinger.postOnAnimation();

-->

RecyclerView.ViewFlinger#postOnAnimation()

-->

ViewCompat.postOnAnimation(RecyclerView.this, this);

-->

RecyclerView.ViewFlinger#run()

-->

if (hresult != 0 || vresult != 0) {
    dispatchOnScrolled(hresult, vresult);
}
```

好了，这里调用到了我们刚才记录下来的RecyclerView#dispatchOnScrolled了，这个方法里面我们会依次调用RecyclerView的mScrollListeners。

代码如下所示：

```
void dispatchOnScrolled(int hresult, int vresult) {
    ...
    if (mScrollListeners != null) {
        for (int i = mScrollListeners.size() - 1; i >= 0; i--) {
            mScrollListeners.get(i).onScrolled(this, hresult, vresult);
        }
    }
    mDispatchScrollCounter--;
}
```



