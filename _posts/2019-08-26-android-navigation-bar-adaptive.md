---
layout: post
title:  虚拟导航(NavigationBar)栏适配
date:   2019-03-01 18:29:58 +0800
categories: NavigationBar
tag: [NavigationBar]
---

* content
{:toc}



### 虚拟导航(NavigationBar)栏适配

做过屏幕适配的同学都知道Android的NavigationBar适配是个问题，尤其是那些NavigationBar还可以动态隐藏显示的，那就更蛋疼了。

比如下面：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vMTU2Njc4NDQyNjMwNTkyMi5naWY)

NavigationBar的显示与隐藏，会直接改变屏幕的可用高度。

如果我们的操作是跟`动态获取的屏幕高度`相关的，那就悲剧了，如果不做特殊处理，是不会主动响应屏幕高度变化的。

#### NavigationBar显示的实例
我们来看个实例，先看下显示NavigationBar时手机的高度：

代码如下：
在Activity#onCreate方法里面添加如下代码：

```
LogUtils.e(TAG, "ScreenWidth:" + ScreenUtils.getScreenW(this));
LogUtils.e(TAG, "ScreenHeight:" + ScreenUtils.getScreenH(this));
```

工具类如下：

```
/**
 * 获取屏幕宽度
 */
public static int getScreenW(Context context) {
    DisplayMetrics dm = context.getResources().getDisplayMetrics();
    int w = dm.widthPixels;
    return w;
}

/**
 * 获取屏幕高度
 */
public static int getScreenH(Context context) {
    DisplayMetrics dm = context.getResources().getDisplayMetrics();
    int h = dm.heightPixels;
    return h;
}
```

具体操作如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vMTU2Njc4NDQyNjMxMTk0Ni5naWY)

输出log如下：

```
E: ScreenWidth:1080
E: ScreenHeight:1792
```

#### NavigationBar隐藏的实例

再看下隐藏NavigationBar时手机的高度：
代码不变，具体操作如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vMTU2Njc4NDQyNjI5OTAyNS5naWY)

输出log如下：

```
E: ScreenWidth:1080
E: ScreenHeight:1920
```

可以看出NavigationBar隐藏与显示时获取到的屏幕高度是不同的。NavigationBar隐藏时屏幕高度为1920px，NavigationBar显示时屏幕高度为1792px。

#### NavigationBar动态显示和隐藏

下面我们看个例子，模拟用户在当前Avtivity内部隐藏、显示导航栏操作。我们通过给根View注册`ViewTreeObserver.OnGlobalLayoutListener`监听器来监听NavigationBar的隐藏于显示。代码如下：

```
globalLayoutListener = () -> {
    LogUtils.e(TAG, "ScreenWidth111:" + ScreenUtils.getScreenW(AdaptiveNavigationBarActivity.this));
    LogUtils.e(TAG, "ScreenHeight111:" + ScreenUtils.getScreenH(AdaptiveNavigationBarActivity.this));
};
        rootView.getViewTreeObserver().addOnGlobalLayoutListener(globalLayoutListener);
```

我们进行如下操作：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vMTU2Njc4NTEzOTcyMTY1OC5naWY)

可以看到输出如下：

NavigationBar从显示到隐藏：

```
E: ScreenWidth111:1080
E: ScreenHeight111:1920
```

NavigationBar从隐藏到显示：

```
E: ScreenWidth111:1080
E: ScreenHeight111:1920
```

可以看出，在NavigationBar的隐藏显示状态变更时，我们通过常规方式获取到的屏幕高度是不变的。

我们如何才能获取到真实的可用屏幕高度呢？

我们在监听器中添加如下代码：

```
Rect r = new Rect();
//获取当前界面可视部分，如果NavigationBar可用，就包含其高度。
getWindow().getDecorView().getWindowVisibleDisplayFrame(r);
int realHeight = r.bottom;
LogUtils.e(TAG, "realHeight:" + realHeight);
```

然后我们再做NavigationBar隐藏显示的操作，可以看到如下log：

NavigationBar从显示到隐藏：

```
E: ScreenWidth111:1080
E: ScreenHeight111:1920
E: realHeight:1920
```

NavigationBar从隐藏到显示：

```
E: ScreenWidth111:1080
E: ScreenHeight111:1920
E: realHeight:1792
```

此时获取到的高度`r.bottom`就是真实的可见高度。

如果我们遇到类似的需求时，用这种方式进行适配即可。

