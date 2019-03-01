# tinyvampirepudge.github.io

这是使用github-pages + jekyll搭建的第一个个人博客主页，有好多不完善的地方，望大家多多指正。


如何写文章
------------------------------------

在`project/_posts`目录下新建一个文件，可以创建文件夹并在文件夹中添加文件，方便维护。
在新建文件中粘贴如下信息，并修改以下的`titile`,`date`,`categories`,`tag`的相关信息，添加`* content {:toc}`为目录相关信息。

在进行正文书写前需要在目录和正文之间输入至少2行空行。然后按照正常的Markdown语法书写正文。

``` bash
---
layout: post
#标题配置
title:  标题
#时间配置
date:   2016-08-27 01:08:00 +0800
#大类配置
categories: document
#小类配置
tag:[教程, 学习]
---

* content
{:toc}


我是正文。我是正文。我是正文。我是正文。我是正文。我是正文。
```

执行
------------------------------------
进入项目根目录，然后执行
``` bash
jekyll serve --watch
```

效果
------------------------------------
打开浏览器并输入URL`http://localhost:4000/`,回车。


关于作者
====================================
喜欢瞎折腾的程序猿一枚。

主要从事Android开发，写过h5，flutter等。

邮箱：tinytongtong@163.com

致谢
====================================
感谢[LessOrMore](https://github.com/luoyan35714/LessOrMore.git)项目的作者。