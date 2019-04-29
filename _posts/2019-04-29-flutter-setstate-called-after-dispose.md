---
layout: post
title:  Flutter中setState导致的内存泄漏——setState() called after dispose()
date:   2019-04-29 14:33:53 +0800
categories: Flutter
tag: [Flutter]
---

* content
{:toc}


### Flutter中setState导致的内存泄漏——setState() called after dispose()

#### 错误原因
flutter端请求网络时，调用的是宿主App的网络请求。

flutter通过消息通道发送一个消息，然后`await`等待消息返回，最终宿主app会调用`reply.reply(obj)`方法返回数据。如果在这个过程中，flutter页面关闭，就会出现如下异常，类似Android中的内存泄漏。

```
2019-04-29 14:01:43.593 10294-10635/com.tinytongtong.flutter E/flutter: [ERROR:flutter/lib/ui/ui_dart_state.cc(148)] Unhandled Exception: setState() called after dispose(): _SystemMessageListRouteState#017f6(lifecycle state: defunct, not mounted)
    This error happens if you call setState() on a State object for a widget that no longer appears in the widget tree (e.g., whose parent widget no longer includes the widget in its build). This error can occur when code calls setState() from a timer or an animation callback. The preferred solution is to cancel the timer or stop listening to the animation in the dispose() callback. Another solution is to check the "mounted" property of this object before calling setState() to ensure the object is still in the tree.
    This error might indicate a memory leak if setState() is being called because another object is retaining a reference to this State object after it has been removed from the tree. To avoid memory leaks, consider breaking the reference to this object during dispose().
```

#### 解决方式

上面的日志说的很明白。

我们的错误原因是异步消息未返回，所以在`setState`方法之前调用`mouted`属性进行判断即可。具体示例如下：

```
      if(mounted){
        setState(() {
          _listData.addAll(list);
      }
```

