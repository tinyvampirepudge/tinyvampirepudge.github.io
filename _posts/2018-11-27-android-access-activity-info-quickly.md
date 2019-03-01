---
layout: post
title:  Android快速查看某个Activity的信息
date:   2018-11-27 19:00:24 +0800
categories: [Android]
tag: [dumpsys, adb shell]
---

* content
{:toc}



### Android快速查看某个Activity的信息

Android中，如果能快速获取某个Activity的名称，我们就不用必须顺着代码逻辑，一步一步的去查找我们想查找的页面了，这就能极大的提高开发速度。

前提条件是手机通过adb连接上电脑，至于环境变量啥的，就不废话了。
然后打开你想知道信息的页面，执行`adb shell dumpsys activity top`，这里以QQ的页面为例，效果如下：
![](https://img1.qdingnet.com/5a94e1b253068cfec38354fe77aa4d1b.png)

如果我们只想要获取栈顶Activity的名称，可以执行这样代码`adb shell dumpsys activity top | grep "ACTIVITY" -A 0`，效果如下：
![](https://img1.qdingnet.com/bea25a183621e32b2ea081b5c055ad01.png)

如果有用的话，记得点赞哦。

#### 参考
[http://gityuan.com/2016/05/14/dumpsys-command/](http://gityuan.com/2016/05/14/dumpsys-command/)
[https://juejin.im/entry/57bfe2c12e958a0069629d7e](https://juejin.im/entry/57bfe2c12e958a0069629d7e)

