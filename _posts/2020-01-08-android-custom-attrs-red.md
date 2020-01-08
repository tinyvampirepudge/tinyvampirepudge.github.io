---
layout: post
title:  自定义属性爆红的问题
date:   2020-01-08 11:02:59 +0800
categories: Android
tag: [Android]
---

* content
{:toc}



### 自定义属性爆红的问题

#### 1、问题描述

在使用三方库[BackgroundLibrary](https://github.com/JavaNoober/BackgroundLibrary)的过程中，我参照的是它官方配置，却出现了如下问题：

![](https://user-gold-cdn.xitu.io/2020/1/8/16f830ef317f7b39?w=2598&h=534&f=png&s=383883)

虽然不影响使用，但是就是爆红，让我这种强迫症患者很难受，简直夜不能寐。

#### 2、解决方式

经过查找，发现了两种解决方式，一种是在使用到这个属性的地方，同步添加` tools:ignore="MissingPrefix"`。

另一种方式是使用`AppCompat前缀`的控件替换掉普通的控件，比如说`TextView`替换为`AppCompatTextView`，`EditText`替换为`AppCompatEditText`等，其他控件类似。

#### 3、参考

[https://stackoverflow.com/questions/40972522/unexpected-namespace-prefix-app-found-for-tag-relativelayout-android](https://stackoverflow.com/questions/40972522/unexpected-namespace-prefix-app-found-for-tag-relativelayout-android)



