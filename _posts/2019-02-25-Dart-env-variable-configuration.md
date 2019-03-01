---
layout: post
title:  Dart配置环境变量
date:   2019-02-25 15:05:26 +0800
categories: Dart
tag: [Dart]
---

* content
{:toc}


### Dart配置环境变量
在学习dart语言时，当你遇到`zsh: command not found: dart`这个错误时，这说明你的dart没有添加进环境变量中。

1、首先打开你的配置文件，
```
vi ~/.zshrc
```

2、将你的dart sdk路径添加进去
路径为`你的sdk路径/flutter/bin/cache/dart-sdk/bin`
```
# dart
export DART_HOME=/Users/xxx/sdk/flutter/bin/cache/dart-sdk/bin
export PATH="${DART_HOME}:${PATH}"
```

3、关闭终端，然后重新打开。执行`dart --version`，效果如下：
```
tinytongtongdeMacBook-Pro% dart --version
Dart VM version: 2.1.1-dev.0.1.flutter-ec86471ccc (Thu Jan 3 22:43:43 2019 +0000) on "macos_x64"
```
这说明安装成功了。