---
layout: post
title:  PagerFragmentAdapter中Fragment的懒加载问题
date:   2017-03-16 23:22:09 +0800
categories: Android
tag: [PagerFragmentAdapter, Fragment懒加载]
---

* content
{:toc}



### PagerFragmentAdapter中Fragment的懒加载问题
严格来说来说不是类的懒加载，而是针对业务中的特殊需求实现的,让fragment在可见的时候再进行网络请求。

在viewpager+PagerFragentAdapter里面的Fragment里面，初次加载时，offset范围内的所有fragment的oncreateView方法都会执行，默认情况下，fragment原有的逻辑中，页面的初始化和网络请求都会触发，这就会导致初次进入时，即使在用户面前只展示了一个tab和fragment,但是却所有tab下页面的数据都请求了，会浪费用户流量。正确的做法是在用户打开fragment之后才请求数据。

默认情况下，ViewPager+PagerFragmentAdapter里面的fragment被全部加载（offset范围内。offset默认值为1，最小值也为1），我们需要做一些处理，让fragment在真正对用户可见时再请求数据。
	
1、原理：setUserVisibleHint方法会在onCreateView()方法之前调用.

在BaseFragment中：

```
/**
 * 跟PagerFragmentAdapter配合使用时起作用。
 * 判断fragment是否可见，在onCreateView方法之前调用。
 *
 * @param isVisibleToUser
 */
@Override
public void setUserVisibleHint(boolean isVisibleToUser) {
    super.setUserVisibleHint(isVisibleToUser);
    if (getUserVisibleHint()) {
        isVisiable = true;
        onHiddenChanged(false);
    } else {
        isVisiable = false;
        onHiddenChanged(true);
    }
}

@Override
public void onHiddenChanged(boolean hidden) {
    super.onHiddenChanged(hidden);
    hiddenChange(hidden);
}

protected void hiddenChange(boolean hidden) {
}
```
2、对于默认打开的fragment，我们需要在它可见时才请求数据。如果不可见，就不会发送请求。

在第一次请求网络的地方，添加判断，如果可见再请求：

```
if (isVisiable) {
    reqServerData();//请求网络数据
}
```
3、对于那些已经初始化完毕但是不可见的页面，由于已经预加载了，所以onCreateView方法不会再次调用，这样请求方法只能写在onHidden()方法里面。这里还存在一个问题，由于setUserVisibleHint方法是在onCreateView方法之前调用，所以界面不一定初始化完毕，需要界面初始化完毕之后才能请求网络；另外，页面每次在不可见和可见之间切换时都会请求网络，这样不符合业务需要，所以这里也需要添加状态进行判断。

```
//页面是初始化完毕。
private boolean isInited = false;
private boolean reqSuccess = false;//第一次请求是否成功。
/**
 * 防止pagerFragmentAdapter的预加载
 *
 * @param hidden
 */
@Override
protected void hiddenChange(boolean hidden) {
    super.hiddenChange(hidden);
    if (hidden) {//页面不可见

    } else {//页面可见
        if (isInited && !reqSuccess) {
            reqServerData();
        }
    }
}
```

4、最后，在onCreateView方法的最后，将isInited的值置为true.在请求成功之后，记得将reqSuccess 的值置为true.

