---
layout: post
title:  Flutter环境搭建、运行gallary项目
date:   2018-11-06 14:05:03 +0800
categories: Flutter
tag: [Flutter, gallary]
---

* content
{:toc}



# Flutter环境搭建、运行gallary项目
### 主体步骤
1、从github clone flutter的sdk，
```
git clone -b beta https://github.com/flutter/flutter.git
```
具体步骤参照官方文档。[https://flutter.io/get-started/install/](https://flutter.io/get-started/install/)

2、配置环境变量
zsh用户配置~/.zshrc文件，添加进环境变量。如下所示。
前两个是国内用户配置的镜像地址，第三个第四个是刚才clone下来的项目的文件目录，具体到bin目录下。
```
//flutter
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export PWD=/Users/XXX/flutter/bin
export PATH="${PWD}:${PATH}"
```
配置完成之后，刷新终端。

使用`echo $PATH`命令查看环境变量是否配置成功。

3、使用`flutter doctor`命令来执行Flutter的安装程序了。这里贴上我执行完doctor命令之后的诊断信息，如下所示：
```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel beta, v0.9.4, on Mac OS X 10.13.6 17G2208, locale en-CN)
[!] Android toolchain - develop for Android devices (Android SDK 28.0.2)
! Some Android licenses not accepted.  To resolve this, run: flutter doctor --android-licenses
[!] iOS toolchain - develop for iOS devices
✗ Xcode installation is incomplete; a full installation is necessary for iOS development.
Download at: https://developer.apple.com/xcode/download/
Or install Xcode via the App Store.
Once installed, run:
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
✗ libimobiledevice and ideviceinstaller are not installed. To install, run:
brew install --HEAD libimobiledevice
brew install ideviceinstaller
✗ ios-deploy not installed. To install:
brew install ios-deploy
✗ CocoaPods not installed.
CocoaPods is used to retrieve the iOS platform side's plugin code that responds to your plugin usage on the Dart side.
Without resolving iOS dependencies with CocoaPods, plugins will not work on iOS.
For more info, see https://flutter.io/platform-plugins
To install:
brew install cocoapods
pod setup
[✓] Android Studio (version 3.1)
✗ Flutter plugin not installed; this adds Flutter specific functionality.
✗ Dart plugin not installed; this adds Dart specific functionality.
[!] VS Code (version 1.28.2)
[!] Connected devices
! No devices available

! Doctor found issues in 4 categories.
```
总结一下，关键信息如下:
* Flutter的版本号及相关信息。
* Android工具链信息，Android SDK 版本等。
* IOS工具链信息，xcode等相关工具需要安装。
* Android Studio相关信息，需要安装Flutter和Dart插件。
* VS Code相关信息。
* 已连接的设备信息：无。

这里给出的提示很详细，均提供了对应的解决方式，根据提示去逐步安装即可。

4、Android Studio插件安装失败。
需要更新Android Studio到最新版，然后再手动安装Dart和flutter插件。

* 去官网下载最新版的Andorid Studio，这里是3.2.1,下载完成之后覆盖安装。
* 下载与Android Studio兼容的Dart插件。

5、运行代码：下载一个IntelliJ，然后打开项目
项目根目录为 ../flutter/examples/flutter_gallary
错误解决：pubspec.yaml中，版本号不匹配
```
Running "flutter packages get" in flutter_gallery...            
Because flutter_gallery depends on flutter_driver any from sdk which depends on source_maps 0.10.7, source_maps 0.10.7 is required.
So, because flutter_gallery depends on source_maps 0.10.8, version solving failed.

pub get failed (1)
Process finished with exit code 1
```
解决方式：切换到beta分支即可。
```
git checkout -b beta origin/beta
```

### 参考
[官方文档](https://flutter.io/)
[github地址](https://github.com/flutter/flutter)
[插件开发，引用插件报plugin “XXX”is incompatible with this installation](https://blog.csdn.net/huangxiaoguo1/article/details/80762620)
[玉刚说](https://mp.weixin.qq.com/s/6lmhHNBRcmoNqkMHaCBx8A)


