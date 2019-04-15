---
layout: post
title:  flutter中实现仿Android端的onResume和onPause方法
date:   2019-04-15 11:17:36 +0800
categories: Flutter
tag: [Flutter, onResume, onPause]
---

* content
{:toc}



### flutter中实现仿Android端的onResume和onPause方法

#### Android端Activity的生命周期
Android中的Activity的生命周期方法如下所示：

![Activity生命周期图](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190412-1763452%402x.png)

这些方法中，对我们比较重要的如下：

* onCreate方法：页面创建时调用。
* onDestroy方法：页面销毁时调用，除了正常关闭页面，还包括异常销毁，比如kill掉应用进程。
* onResume方法：页面由不可见变为可见时调用。
* onPause方法：页面由可见变为不可见时调用，与onResume方法成对出现。

在使用过程中，onCreate和onDestroy方法成对出现，只会调用一次。onResume和onPause方法也是成对出现，会出现多次。

这几个方法为何如此重要呢？为什么非要在flutter端获取到这几个方法呢？
这是因为在现有条件下，flutter相关的社区环境还不够强大，flutter端并不能实现一套代码适配多种终端的效果，相反，它还会严重的依赖宿主App端的实现。

这时，就需要我们在flutter页面中合适的时机，发送消息给宿主App，让其完成对应的实现。

当然了，这类问题在以后可能会得到很好的解决，不必像现在这么费劲。

#### 实现效果

##### demo的结构

![flutter_lifecycle_state](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190415-103854%402x.png)

##### 效果图

1、桌面 --> home --> a --> c  --> a --> home --> 桌面

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Apr-15-2019%2010-50-57.gif)

对应log：
```
I/flutter ( 2376): Home:onMockCreate
I/flutter ( 2376): Home:onMockResume _isVisiableToUser:true
I/flutter ( 2376): ARoute:onMockCreate
I/flutter ( 2376): ARoute:onMockResume _isVisiableToUser:true
I/flutter ( 2376): Home:onMockPause _isVisiableToUser:false
I/flutter ( 2376): CRoute:onMockCreate
I/flutter ( 2376): CRoute:onMockResume _isVisiableToUser:true
I/flutter ( 2376): ARoute:onMockPause _isVisiableToUser:false
I/flutter ( 2376): ARoute:onMockResume _isVisiableToUser:true
I/flutter ( 2376): CRoute:onMockPause _isVisiableToUser:false
I/flutter ( 2376): CRoute:onMockDestroy
I/flutter ( 2376): Home:onMockResume _isVisiableToUser:true
I/flutter ( 2376): ARoute:onMockPause _isVisiableToUser:false
I/flutter ( 2376): ARoute:onMockDestroy
I/flutter ( 2376): Home:onMockPause _isVisiableToUser:true
```

这里解释下：
桌面 --> home操作对应log： 1、2行
home --> a操作对应log： 3、4、5行
a --> c操作对应log： 6、7、8行
c --> a操作对应log： 9、10、11行
a --> home操作对应log： 12、13、14行
home --> 桌面操作对应log： 15行


2、桌面 --> home --> b --> e  --> b --> home --> 桌面

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Apr-15-2019%2010-56-14.gif)

对应log：

```
I/flutter ( 2480): Home:onMockCreate
I/flutter ( 2480): Home:onMockResume _isVisiableToUser:true
I/flutter ( 2480): BRoute:onMockCreate
I/flutter ( 2480): BRoute:onMockResume _isVisiableToUser:true
I/flutter ( 2480): Home:onMockPause _isVisiableToUser:false
I/flutter ( 2480): ERoute:onMockCreate
I/flutter ( 2480): ERoute:onMockResume _isVisiableToUser:true
I/flutter ( 2480): BRoute:onMockPause _isVisiableToUser:false
I/flutter ( 2480): BRoute:onMockResume _isVisiableToUser:true
I/flutter ( 2480): ERoute:onMockPause _isVisiableToUser:false
I/flutter ( 2480): ERoute:onMockDestroy
I/flutter ( 2480): Home:onMockResume _isVisiableToUser:true
I/flutter ( 2480): BRoute:onMockPause _isVisiableToUser:false
I/flutter ( 2480): BRoute:onMockDestroy
I/flutter ( 2480): Home:onMockPause _isVisiableToUser:true
```

这里解释下：
桌面 --> home操作对应log： 1、2行
home --> b操作对应log： 3、4、5行
b --> e操作对应log： 6、7、8行
e --> b操作对应log： 9、10、11行
b --> home操作对应log： 12、13、14行
home --> 桌面操作对应log： 15行


#### 项目地址：

[flutter_lifecycle_state](https://pub.dartlang.org/packages/flutter_lifecycle_state)

#### 使用方式：

##### 1、添加依赖：

在pubspec.yaml文件中添加如下依赖：这里选择最新版本即可。

```
dependencies:
  flutter_lifecycle_state: ^0.0.x
```

note:最新配置请看[https://pub.dartlang.org/packages/flutter_lifecycle_state#-installing-tab-](https://pub.dartlang.org/packages/flutter_lifecycle_state#-installing-tab-)页面。

##### 2、给MaterialApp#navigatorObservers属性设置routeObserver。

这个值定义在我们的package包中，需要导包。

```
import 'package:flutter/material.dart';
import 'package:flutter_lifecycle_state/flutter_lifecycle_state.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      ...
      navigatorObservers: [routeObserver],
    );
  }
}
```

##### 3、页面中使用StateWithLifecycle替换State。

将每个页面级别的Widget的State替换为我们的StateWithLifecycle，导包即可。然后我们可以选择重写onCreate、onPause、onResume、onDestroy方法，在这些方法内部执行对应的业务逻辑即可。

如果需要自定义当前页面的log标识的话，如下所示：给`tagInStateWithLifecycle`字段赋值即可。

```
@override
  void initState() {
    tagInStateWithLifecycle = "WidgetsTestPage";
    super.initState();
  }
```

#### 注意事项

需要注意的是：

1、onDestroy方法某些情况下不会调用

在flutter项目的根页面中，在它正常销毁时，它的的dispose方法是不会调用的，因此我们的onDestroy方法也不会调用。

2、应用非正常关闭时，生命周期方法不会调用。

这就意味着，如果应用在后台被回收，或者其他方式非正常关闭，则某些页面的生命周期方法可能不会正常的调用。

#### 参考：

[https://developer.android.com/guide/components/activities?hl=zh-cn](https://developer.android.com/guide/components/activities?hl=zh-cn)

[demo地址](https://github.com/tinyvampirepudge/flutter_lifecycle_state_test)

