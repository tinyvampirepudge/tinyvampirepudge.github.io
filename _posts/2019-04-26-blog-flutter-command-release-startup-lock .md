---
layout: post
title:  Waiting for another flutter command to release the startup lock... 异常解决
date:   2019-04-26 11:24:07 +0800
categories: Flutter
tag: [Flutter]
---

* content
{:toc}



### Waiting for another flutter command to release the startup lock... 异常解决


平时我们在开发flutter过程中，在执行`flutter packages get`命令之后，如果运气不好的，命令没有执行成功的话，我们就会遇到这个错误提示：

```
Waiting for another flutter command to release the startup lock...
```

然后你会发现会发现在任何地方执行flutter命令，都会遇到这个错误：

```
tinytongtongdeMacBook-Pro% flutter doctor
Waiting for another flutter command to release the startup lock...

```

一般情况下，你会关闭项目，重启IDE，但这些操作都无效，除非你重启电脑。

这里给出一个非常简单的解决方法：

进入到你的flutter sdk目录中，然后找到`bin/cache/lockfile`文件，删除它即可。

删除之后你再运行`flutter doctor`，你会发现错误已经解决了。

参考：

[https://github.com/flutter/flutter/issues/7768](https://github.com/flutter/flutter/issues/7768)



