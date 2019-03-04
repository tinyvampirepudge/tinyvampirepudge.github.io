---
layout: post
title:  Android中的Binder概述
date:   2018-03-18 21:56:04 +0800
categories: Android
tag: [Binder]
---

* content
{:toc}



# Android中的Binder概述
	内容来自blog或者书籍。
	
Android应用的开发离不开四大组件（Activity，Service，BroadcastReceiver，ContentProvider），而这四大组件所涉及的通信底层都是依赖于Binder IPC机制的。例如当进程A中的Activity要向进程B中的Service通信，这便需要依赖于Binder IPC。不仅如此，整个Android系统架构中，大量采用了Binder机制作为IPC方案，当然也存在部分其它的IPC方式，比如Zygote通信便是采用Socket。

概念：Binder是Android中的一种IPC方式，提供远程过程调用(RFC)功能。

### 从进程角度来看IPC机制：

![图片来Gityuan的blog。](http://img-blog.csdn.net/20180318215216953?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

图片来Gityuan的blog。

每个Android的进程，只能运行在自己进程所拥有的虚拟地址空间，对应一个4G的虚拟地址空间，其中3GB是用户空间，1GB是内核空间，当然，内核空间的大小是可以通过参数配置调整的。对于用户空间，不同进程之间彼此是不能共享的，而内核空间却是可共享的。Client进程向Server进程通信，恰恰是利用进程间可共享的内核内存空间来完成底层通信工作的，Client端和Server端往往采用ioctl等方法跟内核空间的驱动进行交互。
		
### 从组成元素角度看：

Android系统的Binder机制包含Client、Server、Service Manager和Binder驱动程序，其中Client、Server和Service Manager运行在用户空间，Binder驱动程序运行在内核空间。Binder就是一种把这四个组件粘合在一起的粘合剂，核心组件是Binder驱动程序。Service Manager提供了辅助管理的功能，Client和Server正是在Binder驱动和Service Manager提供的基础设施上，进行Client-Server通信。

架构图如下图所示：

![图片来Gityuan的blog。](http://img-blog.csdn.net/20180318220107464?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可以看出无论是注册服务和获取服务的过程都需要ServiceManager，需要注意的是此处的Service Manager是指Native层的ServiceManager（C++），并非指framework层的ServiceManager(Java)。ServiceManager是整个Binder通信机制的大管家，是Android进程间通信机制Binder的守护进程。

Binder通信采用C/S架构：
	上图中Client/Server/ServiceManage之间的相互通信都是基于Binder机制。既然基于Binder机制通信，那么同样也是C/S架构，则图中的3大步骤都有相应的Client端与Server端。

1. 注册服务(addService)：Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。

2. 获取服务(getService)：Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。

3. 使用服务：Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：client是客户端，server是服务端。

图中的Client,Server,Service Manager之间交互都是虚线表示，是由于它们彼此之间不是直接交互的，而是都通过与Binder驱动进行交互的，从而实现IPC通信方式。其中Binder驱动位于内核空间，Client,Server,Service Manager位于用户空间。Binder驱动和Service Manager可以看做是Android平台的基础架构，而Client和Server是Android的应用层，开发人员只需自定义实现client、Server端，借助Android的基本平台架构便可以直接进行IPC通信。
	
### Binder的使用场景：
Android中的跨进程通信方式有很多，比方说Bundle，文件共享，Binder系列（Messenger、AIDL、ContentProvider），Socket方式。

它们各自的优缺点和使用场景如下所示:
![这里写图片描述](http://img-blog.csdn.net/20180318215415557?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

参考：

①老罗
[http://blog.csdn.net/luoshengyang/article/details/6618363](http://blog.csdn.net/luoshengyang/article/details/6618363)

②Android开发艺术探索

③[http://gityuan.com/2015/10/31/binder-prepare/](http://gityuan.com/2015/10/31/binder-prepare/)

