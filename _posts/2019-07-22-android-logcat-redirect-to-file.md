---
layout: post
title:  创建文章demo
date:   2019-07-22 15:26:37 +0800
categories: demo
tag: [demo]
---

* content
{:toc}



### 重定向adb logcat输出到文件
在使用Android Studio开发时，经常会遇到logcat的日志无法显示的问题。比如说app运行时发生了崩溃，logcat中的日志就一闪而过，当Activity重启过后，logcat的日志就是新的日志了，无法显示刚才奔溃时的日志，这就很蛋疼。

那么有没有什么好办法让我们看到刚才的日志呢？办法当然是有的，在终端中输入`adb logcat`，就可以看到跟logcat中一毛一样的日志了。

#### 在terminal中查看adb logcat输出：

```
tinytongtongdeMacBook-Pro% adb logcat
```

不过这些日志是没有经过筛选的，看起来很费劲。

#### 筛选特定项目相关的日志
双引号中的是筛选相关的字符串，这里我写的是我自己应用的appId.

```
tinytongtongdeMacBook-Pro% adb logcat -d | grep "com.tiny.tongtong"
```

#### 重定向logcat输出到文件

```
tinytongtongdeMacBook-Pro% adb logcat -d > logcat.log
```

这个命令每次写入都会覆盖logcat.log文件内容，如果要尾部追加，将 `>` 缓存 `>>` 即可。

综合来说，如果我们想将某个应用相关的日志转存到文件中，那么命令如下：

```
tinytongtongdeMacBook-Pro% adb logcat -d | grep "com.tiny.tongtong" > logcat.log
```

#### 注意事项

上述操作成功的前提是，在你的错误信息输出到logcat后，你没有执行`adg shell -c`命令进行清除，你也没有点击as中的logcat视图下左上角的清除按钮。

good luck!

参考：

[logcat 命令行工具](https://developer.android.com/studio/command-line/logcat?hl=zh-CN)

[Save LogCat To A Text File](https://sites.google.com/site/androidhowto/how-to-1/save-logcat-to-a-text-file)

