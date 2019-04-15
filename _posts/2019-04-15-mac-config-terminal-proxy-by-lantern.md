---
layout: post
title:  mac中使用lantern给终端设置代理
date:   2019-04-15 21:38:24 +0800
categories: lantern
tag: [lantern]
---

* content
{:toc}



### mac中使用lantern给终端设置代理

Lantern默认对浏览器的请求进行了代理，但是对终端中的没有进行代理，这里记录下这个有效方式。

#### 1、查看lantern端口号

打开lantern的页面，点击“设置” --> “高级设置”，此时就可以看到我们的端口号了。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190415-212916%402x.png)

#### 2、配置临时代理
如果只是单次使用的话，使用这个方法即可：

打开终端，依次输入这两个命令：

```
export http_proxy=127.0.0.1:54818
export https_proxy=127.0.0.1:54818
```

然后执行`env`，查看下是否写入了环境变量。这个环境变量在终端窗口关闭之后就自动失效了，所以比较安全。

接下里就可以在终端中翻墙了。

#### 3、通过脚本配置永久代理

上述从设置中查看到的端口号其实是不固定的，我们不能简单的将这两个值写入环境变量文件中。

这里我们通过脚本来实现，打开`~/.zshrc`文件:

```
bogon% vi ~/.zshrc
```

输入下面这段脚本：

```
export http_proxy=`scutil --proxy | awk '\
  /HTTPEnable/ { enabled = $3; } \
  /HTTPProxy/ { server = $3; } \
  /HTTPPort/ { port = $3; } \
  END { if (enabled == "1") { print "http://" server ":" port; } }'`
export https_proxy=`scutil --proxy | awk '\
  /HTTPSEnable/ { enabled = $3; } \
  /HTTPSProxy/ { server = $3; } \
  /HTTPSPort/ { port = $3; } \
  END { if (enabled == "1") { print "http://" server ":" port; } }'`
```

然后重启终端，测试效果即可。

#### 参考：

[https://www.mustu.cn/force-macosx-bash-use-lantern-proxy/](https://www.mustu.cn/force-macosx-bash-use-lantern-proxy/)


