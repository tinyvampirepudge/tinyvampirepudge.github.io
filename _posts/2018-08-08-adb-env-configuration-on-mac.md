---
layout: post
title:  mac下配置adb环境变量
date:   2018-08-08 18:52:08 +0800
categories: adb
tag: [adb环境变量]
---

* content
{:toc}



### mac下配置adb环境变量

&emsp;&emsp;在终端中输入adb命令时，会提示 command not found ，这是是因为mac电脑下没有配置Android环境变量或者环境变量配置错误。

下面我们就来配置环境变量：
1、打开配置文件，terminal中 输入 vi ~/.bash_profile

```
tinytongtongdeMacBook-Pro:~ tinytongtong$ vi ~/.bash_profile
```

此时我们会进入vi模式。
2、在vi模式下输入下面几段代码：

```
export ANDROID_HOME=/Users/你的用户名/Documents/develop/sdk
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
```

如下图所示：
![环境变量](https://img-blog.csdn.net/20180808184916194?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3、退出vi模式，刷新环境变量：source ~/.bash_profile

4、执行adb version命令，查看是否成功。
如下所示，配制成功。

```
tinytongtongdeMacBook-Pro:~ tinytongtong$ adb version
Android Debug Bridge version 1.0.40
Version 4797878
Installed as /Users/tinytongtong/Documents/develop/sdk/platform-tools/adb
tinytongtongdeMacBook-Pro:~ tinytongtong$
```


