---
layout: post
title:  flutter packages 开发实战——发布失败问题解决
date:   2019-04-15 15:40:07 +0800
categories: Flutter
tag: [Flutter, flutter packages pub publish failed]
---

* content
{:toc}


### flutter packages 开发实战——发布失败问题解决

Flutter 的库是以 package 的方式来管理。Package 分为两种，Dart package（也叫 library package） 和 plugin package。当我们说 Fluter 包的时候，指的其实也是 Dart 包，它只能使用 Dart 和 Flutter 提供的 API；而当我们说 Flutter 插件时指的是后者，也就是 plugin package。Flutter 插件通常会包含平台特定的代码。对包的使用者来说，两者没有区别。

这里我们创建一个package，并发布到dart[仓库](https://pub.dartlang.org/)上。

#### 1、package开发过程

##### 使用Android Studio新建一个flutter package项目

具体流程如下：
* 在菜单上选择 File -> New -> New Flutter Project。
* 在弹出的面板里选择 Flutter Package，点击 next。
* 填写package的相关信息，Project Name填入我们的项目名称即可，其他信息按需填写。

具体如下图。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190415-133705%402x.png)

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190415-134553%402x.png)

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190415-134626%402x.png)

##### 具体业务开发

接下来就可以在`lib/`包下写我们自己package中的代码了。

本文的重点不在这儿，如需了解，具体请查看[Flutter学习指南：封装 API 插件——玉刚说](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649493492&idx=1&sn=e1a0df76769eeb02d7422ae0f90e9941&chksm=8eec840bb99b0d1d32690593a54a12337b283bfa5abe69415d5caf5403dcb3a2865414c5c8d6&scene=21%23wechat_redirect)等资料。

#### 2、上传信息填写

##### 填写用户名、邮箱、个人主页信息

上传之前，我们需要完善本地的配置信息，具体在 `pubspec.yaml` 文件中。我们在使用as新建项目时，这个文件中已经自动生成了对应的信息，我们只需要填写用户名、个人邮箱和主页地址就好。

```
name: flutter_lifecycle_state
description: Make State support lifecycle method just like Android&#x27;s Activity#onCreate、 Activity#onResume、Activity#onPause、Activity#onDestroy.
version: 0.0.1
author: tinytongtong<tinyvampire1102@gmail.com>
homepage: https://blog.csdn.net/qq_26287435
```

这里的name和description字段的值，就是新建package对话框中填写的信息。

version字段是用来发版的，每次发新版记得增加版本号。

author字段中，填写的是作者名称和可用的Google邮箱，格式是username<gmail>。需要注意的是，这个gmail邮箱需要保证可用。

homepage字段，填写的是个人主页url即可，这里我填写的是我的csdn主页。

##### LICENSE文件
另外，发布到 Pub 上的包需要包含一个 LICENSE，关于 LICENSE 文件，最简单的方法就是在 GitHub 创建仓库的时候选中一个。

这里我是从新建的github项目中粘贴过来的。

##### 检查包的配置

我们在`项目的根目录`执行以下命令，检测一下包的配置是否问题：

```
flutter packages pub publish --dry-run
```

如果没有问题，输出如下：

```
...

Package has 0 warnings.
```

##### 发布包

发布包和上一步一样，只是少了 --dry-run 参数：

```
flutter packages pub publish
```

如果是第一次发布，会提示验证 Google 账号。google账号是以`https:开头，以.email结尾的地址`，防止因为地址不全导致打开页面错误。

授权后便可以继续上传，如果成功的话，会提示“Successful uploaded package”：

```
Looks great! Are you ready to upload your package (y/n)? y
Pub needs your authorization to upload packages on your behalf.
In a web browser, go to https://accounts.google.com/o/oauth2/auth?access_type=offline&approval_prompt=force&response_type=code&client_id=xxxxxxxxxxxxxx-8grd2eg9tj9f38os6f1urbcvsq399u8n.apps.googleusercontent.com&redirect_uri=http%3A%2F%2Flocalhost%3A52589&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email
Then click "Allow access".

Waiting for your authorization...
Successfully authorized.
Uploading...
```

当然了，一般情况下，你现在就是各种报错了，如果这么简单就可以成功的话，那我就不会写这篇文章了。莫慌，接着往下看。

#### 3、发布过程中常见错误

* 去掉host文件中针对`PUB_HOSTED_URL`、`FLUTTER_STORAGE_BASE_URL`的修改。

如果你看到下图第一行所示的这个错误，此时你需要去掉官方指引里面对`PUB_HOSTED_URL`、`FLUTTER_STORAGE_BASE_URL`的修改，这些修改会导致上传pub失败。[https://github.com/flutter/flutter/wiki/Using-Flutter-in-China](https://github.com/flutter/flutter/wiki/Using-Flutter-in-China)

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190415-150934%402x.png)

修改后的host如下:然后刷新终端。

```
# flutter
#export PUB_HOSTED_URL=https://pub.flutter-io.cn
#export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export PWD=/Users/tinytongtong/Documents/workspace/flutter/sdk/flutter/bin
export PATH="${PWD}:${PATH}"
```

* 未开代理
一般情况下，如果你没有`自备梯子`的话，就会看到上传步骤卡在这儿不同了。如下所示。

```
tinytongtongdeMacBook-Pro% flutter packages pub publish
Publishing flutter_lifecycle_state 0.0.1 to https://pub.flutter-io.cn:
...
Looks great! Are you ready to upload your package (y/n)? y
Uploading...

```

你此时敲击下回车键，就会出现如下错误提示：

```
Looks great! Are you ready to upload your package (y/n)? y
Uploading...

Unhandled exception:
SocketException: Write failed (OS Error: Broken pipe, errno = 32), port = 0
#0      _rootHandleUncaughtError.<anonymous closure> (dart:async/zone.dart:1112:29)
#1      _microtaskLoop (dart:async/schedule_microtask.dart:41:21)
#2      _startMicrotaskLoop (dart:async/schedule_microtask.dart:50:5)
#3      _runPendingImmediateCallback (dart:isolate/runtime/libisolate_patch.dart:115:13)
#4      _RawReceivePortImpl._handleMessage (dart:isolate/runtime/libisolate_patch.dart:172:5)
tinytongtongdeMacBook-Pro% 
```

这个错误根源是没有开代理。

* 开了`lantern`代理。

如果你已经开了代理，比方说lantern，你可能就会看到如下错误：

```
bogon% flutter packages pub publish
Publishing flutter_lifecycle_state 0.0.1 to https://pub.dartlang.org:
...

Looks great! Are you ready to upload your package (y/n)? y
Pub needs your authorization to upload packages on your behalf.
In a web browser, go to https://accounts.google.com/o/oauth2/auth?access_type=xx1urbcvsq39xxe=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email
Then click "Allow access".

Waiting for your authorization...
Authorization received, processing...
It looks like accounts.google.com is having some trouble.
Pub will wait for a while before trying to connect again.
OS Error: Operation timed out, errno = 60, address = accounts.google.com, port = 58913
pub finished with exit code 69
bogon%
```

或者下面这种：

```
Waiting for your authorization...
Authorization received, processing...
Connection closed before full header was received
pub finished with exit code 69
```

这两个错误产生的根源是，你需要给`终端设置代理`，也就是`命令行代理`。


#### 4、最终解决方式——命令行翻墙工具

更新：使用lantern给终端设置代理请移步[mac中使用lantern给终端设置代理](https://blog.csdn.net/qq_26287435/article/details/89321578).

这里推荐一种解决方式：mac端可使用`proxifier + shadowsocks`的方式。

[Proxifier+Shadowshocks系统全局代理的正确姿势](http://blackwolfsec.cc/2016/09/19/Proxifier_Shadowshocks/)

[Shadowsocks X](https://www.sednax.com/faq-0-cn.php)

按照教程下载配置即可。

命令行代理成功校验方式：

以 mac 为例 你输入`curl google.com`,如果有成功的回文(一个 html 格式的文本信息)说明成功了,如果没有就说明你的终端还在墙内,你需要自行保证 curl 能连接成功

下面这个是我在项目根目录下执行`curl google.com`的结果，说明成功了

```
tinytongtongdeMacBook-Pro% curl google.com            
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
tinytongtongdeMacBook-Pro% 

```

#### 5、一次成功上传的示例
在所有的问题都解决之后，再次`flutter packages pub publish`命令，输出如下：

```
tinytongtongdeMacBook-Pro% flutter packages pub publish
Publishing flutter_lifecycle_state 0.0.2 to https://pub.dartlang.org:
|-- .gitignore
|-- .idea
|   |-- codeStyles
|   |   '-- Project.xml
|   |-- dbnavigator.xml
|   |-- encodings.xml
|   |-- libraries
|   |   |-- Dart_Packages.xml
|   |   |-- Dart_SDK.xml
|   |   '-- Flutter_Plugins.xml
|   |-- markdown-navigator
|   |   '-- profiles_settings.xml
|   |-- markdown-navigator.xml
|   |-- misc.xml
|   |-- modules.xml
|   |-- vcs.xml
|   '-- workspace.xml
|-- .metadata
|-- CHANGELOG.md
|-- LICENSE
|-- README.md
|-- android
|   |-- app
|   |   '-- src
|   |       '-- main
|   |           '-- java
|   |               '-- io
|   |                   '-- flutter
|   |                       '-- plugins
|   |                           '-- GeneratedPluginRegistrant.java
|   '-- local.properties
|-- lib
|   '-- flutter_lifecycle_state.dart
|-- pubspec.yaml
'-- test
    '-- flutter_lifecycle_state_test.dart

Looks great! Are you ready to upload your package (y/n)? y
Uploading...
Successfully uploaded package.
tinytongtongdeMacBook-Pro% 
```

发布成功后，你的gmail就会收到通知邮件了，邮件中有你项目的链接，如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190415-152956%402x.png)

#### 参考

[Flutter学习指南：封装 API 插件——玉刚说](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649493492&idx=1&sn=e1a0df76769eeb02d7422ae0f90e9941&chksm=8eec840bb99b0d1d32690593a54a12337b283bfa5abe69415d5caf5403dcb3a2865414c5c8d6&scene=21%23wechat_redirect)

[flutter pub 发布失败](https://www.kikt.top/posts/flutter/package/publish-fail/)

[Failed to upload the package #16658](https://github.com/flutter/flutter/issues/16658)

[Publish plugin in china #17070](https://github.com/flutter/flutter/issues/17070)

[https://zhuanlan.zhihu.com/p/60136574](https://zhuanlan.zhihu.com/p/60136574)