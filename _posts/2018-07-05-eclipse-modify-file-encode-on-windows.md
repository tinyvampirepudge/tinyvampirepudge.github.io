---
layout: post
title:  eclipse修改文件编码，window系统
date:   2018-07-05 16:54:56 +0800
categories: eclipse
tag: [eclipse文件编码]
---

* content
{:toc}



### eclipse修改文件编码，window系统

Eclipse中经常会遇到中文乱码的问题，一般都是编码格式不一致，eclipse默认的编码格式是GBK，这里推荐统一使用UTF-8。
乱码如下所示：
![图1](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/eclipse1.png)

下面说设置方式，一般网上都推荐下面这种方式：
File --> Properties，对话框如下所示，选择Others --> UTF-8。最后点击OK即可。
![图2](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/eclipse2.png)

上面这种设置方式某些会有问题，比方说你给某个文件设置的编码格式不一致，也会导致中文乱码。这里介绍另外一种方式设置编码格式，在你项目的根目录上右键，在弹出框中选择 Properties，如下图所示，也选择UTF-8即可。
![图3](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/eclipse3.png)

同理，你也可以给某个文件设置单独的编码格式，这个留个读者自己去测试。

