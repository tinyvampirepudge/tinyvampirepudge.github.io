---
layout: post
title:  记一次HuaWei p9输入法的bug
date:   2019-09-03 11:38:03 +0800
categories: 输入法
tag: [输入法]
---

* content
{:toc}



### 记一次HuaWei p9输入法的bug

今天刚上班，刚跟测试小姐姐打了个招呼，小姐姐反手就给我丢了一个bug，我....

bug如下：

```
Bug描述：XXApp华为P9一般需要点击两次发送按钮才能发送成功。
```

现象如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vZ2lmaG9tZV80MzJ4NzY4XzEzcy5naWY)

可以看出，第一次点击后有个类似蒙层的东西消失了，初步断定这个bug跟机型和输入法有关。

#### 区分是否特定App

如果是该手机输入法的问题，那么别的app也应该有这个问题（如果他们不做处理的话），我们看下p9手机上的微信，是否也有这个问题。具体如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vZ2lmaG9tZV80MzJ4NzY4XzdzXzE1Njc0ODA3Mzg4NjQ0OTYuZ2lm)

可以看到，p9手机上，微信也有这个问题，那我们可以初步确定这个问题跟我们的app设置无关，而是跟机型或者输入法相关了。

#### 区分是否特定机型

再看下机型，当前机型为`华为P9 EVA-AL00`，我再祭出一个华为手机`华为 Mate10 Pro BLA-AL00`，他两的输入法都一致，均为`百度输入法华为版`，只是输入法版本不一样。

经过测试，发现`华为 Mate10 Pro BLA-AL00`手机上是没有这个问题的，那么问题就可能出在`输入法设置`或者`输入法版本上`了。

#### 甄别输入法设置

除了上面的两个线索，我还发现了下面这个现象：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vZ2lmaG9tZV80MzJ4NzY4XzhzXzE1Njc0ODA0MzE0OTAyMzguZ2lm)

多余的蒙层好像输入法的手写板啊。

于是我们直接对比`华为P9 EVA-AL00`和`华为 Mate10 Pro BLA-AL00`两个手机的输入法设置中，关于手写板的设置，发现他们的设置不同。具体如下：

`华为 Mate10 Pro BLA-AL00`：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV1gyMDE5MDkwMy0xMTMyNDUlNDAyeC5wbmc?x-oss-process=image/format,png)

`华为P9 EVA-AL00`:

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV1gyMDE5MDkwMy0xMTMyNTglNDAyeC5wbmc?x-oss-process=image/format,png)

看着好像就是`华为P9 EVA-AL00`手机上多了这个设置，那么就把键盘手写禁掉。

然后返回我们的app看效果，完全正常了，完美。

实际定位问题的过程中，肯定不是这么一帆风顺的，肯定有误区，这里做下记录，希望对大家有所帮助。


