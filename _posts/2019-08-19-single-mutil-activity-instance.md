---
layout: post
title:  singleTop启动模式真的可以防止多次打开栈顶的Activity么？
date:   2019-08-19 12:23:49 +0800
categories: 启动模式
tag: [singleTop]
---

* content
{:toc}




### singleTop启动模式真的可以防止多次打开栈顶的Activity么？

开发过程中我们经常会遇到各式各样的bug，比如说测试小姐姐告诉我们，由于无操作，某个按钮她`快速点击了两次`（或者由于卡顿之类的延迟），`打开了两个详情页`，希望把这个`禁止掉`，只让打开一个详情页。

#### 快速点击按钮，多次打开默认启动模式的Activity

我们先来复现下这种场景，Activity配置和按钮点击代码很简单，如下所示：
activity配置：

```
<activity
    android:name=".kfysts.chapter01.activity.LauncherModeSecondActivity"
    android:launchMode="standard" />
```

按钮的点击事件：

```
@OnClick(R.id.btn_test1)
public void onBtnTest1Clicked() {
    startActivity(new Intent(this, LauncherModeSecondActivity.class));
}
```

我们快速点击三下按钮，效果大概如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-19-2019%2011-45-10.gif)

我们看下任务栈信息，在终端中输入如下命令，将数据导入到log.txt中：

不了解日志重定向的同学，请看[重定向adb logcat输出到文件](https://juejin.im/post/5d3565ac6fb9a07ef81a3b0c).

```
adb shell dumpsys activity activities > log.txt
```

打开log.txt，我们看下任务栈信息：

```
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)
...
    * TaskRecord{1779ff5 #157 A=com.tiny.demo.firstlinecode U=0 StackId=2 sz=7}
      userId=0 effectiveUid=u0a85 mCallingUid=u0a85 mUserSetupComplete=true mCallingPackage=com.tiny.demo.firstlinecode
      ...
      Activities=[ActivityRecord{991bdbe u0 com.tiny.demo.firstlinecode/.MainActivity t157}, ActivityRecord{9eb4346 u0 com.tiny.demo.firstlinecode/.kfysts.AndroidKfystsActivity t157}, ActivityRecord{d4b56c9 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.AndroidKfystsChapter01Activity t157}, ActivityRecord{ce38d30 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LauncherModeEntryActivity t157}, ActivityRecord{704de09 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LauncherModeSecondActivity t157}, ActivityRecord{e063635 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LauncherModeSecondActivity t157}, ActivityRecord{bd81a7a u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LauncherModeSecondActivity t157}]
      ...
```

可以看到我们的`LauncherModeSecondActivity`打开了三次。

#### 使用singleTop进行防止多次打开

小姐姐提的bug我们不能不管，针对这个bug我们第一反应是做一个`防止View的多次点击`，不过这里我们有更好的选择。针对`点击打开Activity`这种操作，可以使用`Activity启动模式——singleTop`来解决，这种启动模式的核心就是，位于栈顶的Activity不会再次创建实例。

singTop的解释如下：

```
singleTop：栈顶复用模式。

在这种模式下，如果新Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，同时它的onNewIntent方法会被调用，通过此方法的参数我们可以取出当前请求的信息。

如果新Activity的实例已存在但不是位于栈顶，那么新Activity仍然会重新创建。
```

话不多说，我们使用singleTop模式来优化下，在AndroidManifest.xml文件中，给对应的Activity添加`android:launchMode="singleTop"`，代码如下：

```
<activity
    android:name=".kfysts.chapter01.activity.LaunchModeSingleTopTestActivity"
    android:launchMode="singleTop" />
```

点击启动的代码如下：
这里新增了一条log，我们可以通过log来确定具体点击了几次。

```
@OnClick(R.id.btn_test5)
public void onBtnTest5Clicked() {
    LogUtils.e("single Top clicked");
    startActivity(new Intent(this, LaunchModeSingleTopTestActivity.class));
}
```

然后我们尝试多次点击按钮，会发现多次打开Activity的问题已经解决了。

先看下log，我们确实多次点击了按钮：

```
com.tiny.demo.firstlinecode E: single Top clicked
com.tiny.demo.firstlinecode E: single Top clicked
com.tiny.demo.firstlinecode E: single Top clicked
```

再看下效果图，如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-19-2019%2012-01-39.gif)

再来看下具体的任务栈信息，执行`adb shell dumpsys activity activities > log.txt`，结果如下：

```
* TaskRecord{b982558 #158 A=com.tiny.demo.firstlinecode U=0 StackId=3 sz=5}
      userId=0 effectiveUid=u0a85 mCallingUid=u0a85 mUserSetupComplete=true mCallingPackage=com.tiny.demo.firstlinecode
      ...
      Activities=[ActivityRecord{47ddc34 u0 com.tiny.demo.firstlinecode/.MainActivity t158}, ActivityRecord{206fabc u0 com.tiny.demo.firstlinecode/.kfysts.AndroidKfystsActivity t158}, ActivityRecord{fa98f6d u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.AndroidKfystsChapter01Activity t158}, ActivityRecord{a41b176 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LauncherModeEntryActivity t158}, ActivityRecord{e2f82e5 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LaunchModeSingleTopTestActivity t158}]
```

从结果中可以看到，`LaunchModeSingleTopTestActivity`在栈顶确实只有一个实例。


综合以上的结果，我们可以得出结论，singleTop启动模式确实解决了`栈顶Activity重复打开`的问题，在多次点击的情况下，栈顶Activity只打开了一次。


#### singleTop真的能完全防止多次打开栈顶的Activity么？
虽然我们的bug完美解决了，但作为程序员，我们还是需要杠精下的。

这里我改下点击事件的代码，瞬间多次调用启动activity的代码，代码如下：

```
@OnClick(R.id.btn_test6)
public void onBtnTest6Clicked() {
    for (int i = 0; i < 5; i++) {
        startActivity(new Intent(this, LaunchModeSingleTopTestActivity.class));
    }
}
```

然后我们看下效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-19-2019%2012-11-38.gif)

纳尼，竟然打开了多个。

再看下任务栈信息：

```
* TaskRecord{f8d39d #159 A=com.tiny.demo.firstlinecode U=0 StackId=4 sz=9}
      userId=0 effectiveUid=u0a85 mCallingUid=u0a85 
      ...
      Activities=[ActivityRecord{9ed78cc u0 com.tiny.demo.firstlinecode/.MainActivity t159}, ActivityRecord{be3aa3c u0 com.tiny.demo.firstlinecode/.kfysts.AndroidKfystsActivity t159}, ActivityRecord{1654b04 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.AndroidKfystsChapter01Activity t159}, ActivityRecord{a9084f6 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LauncherModeEntryActivity t159}, ActivityRecord{932877f u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LaunchModeSingleTopTestActivity t159}, ActivityRecord{c4238aa u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LaunchModeSingleTopTestActivity t159}, ActivityRecord{e751011 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LaunchModeSingleTopTestActivity t159}, ActivityRecord{fe1ffe4 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LaunchModeSingleTopTestActivity t159}, ActivityRecord{c0d8413 u0 com.tiny.demo.firstlinecode/.kfysts.chapter01.activity.LaunchModeSingleTopTestActivity t159}]
```

可以看到，我们的`LaunchModeSingleTopTestActivity`确实打开了五次。

其实这也很好理解，不管任何操作都是需要时间去执行的，我们的activity的启动过程也是。

即使我们给Activity设置了启动模式，他们也不是立刻生效的，也需要执行到对应的代码逻辑后才会生效。

所以如果我在for循环里面瞬间执行多次打开Activity的操作，那么启动模式生效的代码还未执行到，所以启动模式就不会生效。

当然了，代码中一般也不会这么写，知其然并知其所以然才是我们的目的。

github项目地址:[Android_Base_Demo](https://github.com/tinyvampirepudge/Android_Base_Demo)

具体页面打开路径：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Aug-19-2019%2012-21-22.gif)

具体页面地址：[https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/kfysts/chapter01/activity/LauncherModeEntryActivity.java](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/kfysts/chapter01/activity/LauncherModeEntryActivity.java)


#### 参考
Android开发艺术探索

