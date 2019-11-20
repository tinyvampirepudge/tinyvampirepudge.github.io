---
layout: post
title:  Android中TextView字体加粗小技巧
date:   2019-11-20 12:15:18 +0800
categories: Android
tag: [TextView]
---

* content
{:toc}


### Android中TextView字体加粗小技巧

开发中经常会遇到字体加粗的需求，在使用系统字体的情况下，我们一般是通过在布局文件中给TextView设置`android:textStyle="bold"`属性。

如果你们的设计师小姐姐不想使用Android的这种加粗效果，只是想要接近于`PingFang SC Medium`的效果，那么TextView的`bold`就有点没脸看了。

标注图如下：

![](https://user-gold-cdn.xitu.io/2019/11/20/16e8702c42e63a69?w=297&h=223&f=png&s=12441)

怎么办呢？我们可以通过比较折中的方式来设置，代码如下：

```
tv.getPaint().setFakeBoldText(true);
```

它加粗后的效果就比较接近medium的效果。

不设置加粗、设置bold属性、代码调用`setFakeBoldText(true)`三种方式的效果如下：

![](https://user-gold-cdn.xitu.io/2019/11/20/16e8702c42d80451?w=834&h=760&f=png&s=91779)

