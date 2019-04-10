---
layout: post
title:  No toolchains found in the NDK toolchains错误解析与解决。
date:   2019-04-10 11:38:40 +0800
categories: Ndk
tag: [toolchains]
---

* content
{:toc}


### No toolchains found in the NDK toolchains错误解析与解决。

#### 错误详情

之前的android项目中没有集成ndk，后来在sdk中下载了ndk相关文件，再次打开一个没有集成ndk的项目后，构建时会出现如下异常：

```
No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android
```

#### 错误原因分析


解决错误第一步，去Google，然后我就找到了[完美解决 No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android](https://blog.csdn.net/qq_24118527/article/details/82867864)这篇文章，按照步骤去官网下载最新的ndk包，发现toolchains目录下并没有`mips64el-linux-android`前缀的文件。

去官网下载最新的ndk，[ndk下载地址](https://developer.android.com/ndk/downloads/?hl=zh-cn)，我这里下载是最新稳定版本r19c，如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-104739%402x.png)

我们的期望效果是toolchains目录下会包含`mips64el-linux-android`前缀的文件，如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-104944%402x.png)

但现实很无情，并没有这个文件。效果如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-104835%402x.png)

最开始我以为是我没下载对文件，但比对文件的sha1值后发现一致，这就说明我没下载错。

另外，上面那篇blog中作者下载ndk版本是r16b，我这里下载的版本是r19c。如果简单粗暴的将r16b中的`mips64el-linux-android`前缀的文件copy进r19c的文件中，及时问题得到了解决，那也是不合理的，没有解释清楚为什么新版中没有`mips64el-linux-android`前缀的文件。

那我们就去看下为什么`mips64el-linux-android`前缀的文件在新版ndk中被移除了。

我们先去看下ndk的版本日志。[ndk版本更新日志-r18](https://github.com/android-ndk/ndk/wiki/Changelog-r18)

在这个页面的`Known Issues`模块中，我们发现有下面这段话，它的意思就是，这版的ndk所示不兼容`Android Gradle`插件的3.0及以下版本的。如果你在这类版本中看到了`No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android`这类提示，就升级你的gradle插件版本到3.1及以上。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-105726%402x.png)

检查了下当前项目的gradle plugin版本，是3.0.0。

#### 解决方式——升级gradle插件版本

通过以上分析，我们只需要升级android项目中的gradle插件版本就可以了。

Android项目中Gradle插件的版本是在哪儿设置的呢？在项目根目录下的build.gradle配置文件中，据图如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-111647%402x.png)

具体版本号我们又该如何修改呢？请参看[Android Gradle plugin](https://developer.android.com/studio/releases/gradle-plugin)。这里有个小技巧，如果你不知道最新的Gradle版本如何写，那么你创建一个新的Android项目，看下as默认使用的gradle插件版本号是多少，抄过来即可。这里我们选择3.2.1，设置完成后如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-111904%402x.png)

同步下gradle配置，会发现出现了下面这个错误提示，提示我们升级gradle版本，当前版本是4.1，最少需要升级到4.6。如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-111919%402x.png)

我们打开/gradle/wrapper/gradle-wrapper.properties文件，进行修改，具体指需要将版本号4.6修改为4.6即可。
修改之前：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-112157%402x.png)

修改之后：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190410-112340%402x.png)

接着我们同步下gradle文件即可。

#### 参考

[完美解决 No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android](https://blog.csdn.net/qq_24118527/article/details/82867864)


[ndk下载地址](https://developer.android.com/ndk/downloads/?hl=zh-cn)

[ndk版本更新日志-r18](https://github.com/android-ndk/ndk/wiki/Changelog-r18)

[Android Gradle plugin](https://developer.android.com/studio/releases/gradle-plugin)



