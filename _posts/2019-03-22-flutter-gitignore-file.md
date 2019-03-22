---
layout: post
title:  给Flutter项目添加.gitignore文件以及如何修改.gitignore文件并生效
date:   2019-03-22 11:40:14 +0800
categories: Flutter
tag: [Flutter, .gitignore]
---

* content
{:toc}



### 给Flutter项目添加.gitignore文件以及如何修改.gitignore文件并生效

flutter项目的开发，一般来说都是与原生开发混合进行的，单纯的flutter开发局限性很大，需要与原生进行配合。

#### flutter项目集成的两种方式

这就涉及到如何将flutter与现有的项目进行融合。这里以客户端的Android/Ios开发为例，使用flutter开发项目大体有两种集成方式，第一种就是新建flutter项目，android端就在flutter/android目录下开发，ios端就在flutter/ios目录下开发，这种方式对新开发一个flutter项目来说很方便。第二种集成方式是在已有的Android、Ios项目中添加flutter，通过module依赖的方式加入，具体请参照[向现有app添加flutter模块](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)，这种方式的优点是已有的客户端项目不需要做太大的变动，Android端和ios端只需要新增flutter module即可。

下面我们讲讲flutter项目的.gitignore文件。

#### flutter项目的.gitignore文件

.gitignore文件如果完全自己去写的话会很麻烦，幸运的是我们直接去github上的flutter项目中可以找到最权威的[.gitignore文件](https://github.com/flutter/flutter/blob/master/.gitignore)。

美中不足的是，这个.gitignore可以覆盖掉第一种集成方式的文件，针对flutter module这种集成方式，android端项目是以`.android/`文件形式存在于flutter module中的，ios端项目是以`.ios/`文件形式存在的，所以针对这两个文件需要做些相应的修改。

主要是将`.gitignore`文件中针对`/android/`文件夹和`/ios/`文件夹的忽略，仿照这添加一份`/.android/`和`/.ios/`的忽略。

修改的部分如下：

```
# Android related
**/android/**/gradle-wrapper.jar
**/android/.gradle
**/android/captures/
**/android/gradlew
**/android/gradlew.bat
**/android/local.properties
**/android/**/GeneratedPluginRegistrant.java
**/android/key.properties
*.jks

# iOS/XCode related
**/ios/**/*.mode1v3
**/ios/**/*.mode2v3
**/ios/**/*.moved-aside
**/ios/**/*.pbxuser
**/ios/**/*.perspectivev3
**/ios/**/*sync/
**/ios/**/.sconsign.dblite
**/ios/**/.tags*
**/ios/**/.vagrant/
**/ios/**/DerivedData/
**/ios/**/Icon?
**/ios/**/Pods/
**/ios/**/.symlinks/
**/ios/**/profile
**/ios/**/xcuserdata
**/ios/.generated/
**/ios/Flutter/App.framework
**/ios/Flutter/Flutter.framework
**/ios/Flutter/Generated.xcconfig
**/ios/Flutter/app.flx
**/ios/Flutter/app.zip
**/ios/Flutter/flutter_assets/
**/ios/ServiceDefinitions.json
**/ios/Runner/GeneratedPluginRegistrant.*
```

修改之后：

```
# Android related
**/android/**/gradle-wrapper.jar
**/android/.gradle
**/android/captures/
**/android/gradlew
**/android/gradlew.bat
**/android/local.properties
**/android/**/GeneratedPluginRegistrant.java
**/android/key.properties
*.jks

# 针对flutter当做module集成进现有项目时，.android目录下的文件
**/.android/**/gradle-wrapper.jar
**/.android/.gradle
**/.android/captures/
**/.android/gradlew
**/.android/gradlew.bat
**/.android/local.properties
**/.android/**/GeneratedPluginRegistrant.java
**/.android/key.properties


# iOS/XCode related
**/ios/**/*.mode1v3
**/ios/**/*.mode2v3
**/ios/**/*.moved-aside
**/ios/**/*.pbxuser
**/ios/**/*.perspectivev3
**/ios/**/*sync/
**/ios/**/.sconsign.dblite
**/ios/**/.tags*
**/ios/**/.vagrant/
**/ios/**/DerivedData/
**/ios/**/Icon?
**/ios/**/Pods/
**/ios/**/.symlinks/
**/ios/**/profile
**/ios/**/xcuserdata
**/ios/.generated/
**/ios/Flutter/App.framework
**/ios/Flutter/Flutter.framework
**/ios/Flutter/Generated.xcconfig
**/ios/Flutter/app.flx
**/ios/Flutter/app.zip
**/ios/Flutter/flutter_assets/
**/ios/ServiceDefinitions.json
**/ios/Runner/GeneratedPluginRegistrant.*

# 针对flutter当做module集成进现有项目时，.ios目录下的文件
**/.ios/**/*.mode1v3
**/.ios/**/*.mode2v3
**/.ios/**/*.moved-aside
**/.ios/**/*.pbxuser
**/.ios/**/*.perspectivev3
**/.ios/**/*sync/
**/.ios/**/.sconsign.dblite
**/.ios/**/.tags*
**/.ios/**/.vagrant/
**/.ios/**/DerivedData/
**/.ios/**/Icon?
**/.ios/**/Pods/
**/.ios/**/.symlinks/
**/.ios/**/profile
**/.ios/**/xcuserdata
**/.ios/.generated/
**/.ios/Flutter/App.framework
**/.ios/Flutter/Flutter.framework
**/.ios/Flutter/Generated.xcconfig
**/.ios/Flutter/app.flx
**/.ios/Flutter/app.zip
**/.ios/Flutter/flutter_assets/
**/.ios/ServiceDefinitions.json
**/.ios/Runner/GeneratedPluginRegistrant.*
```

#### 修改.gitignore文件并让其生效
.gitignore文件修改完了，此时可能会存在一个新的问题。如果`.android/`和`.ios/`文件夹下的文件已经提交到远程仓库了，就算我们这里修改并提交了.gitignore文件，这次针对`.android/`和`.ios/`的修改不会生效的。因为git会继续追踪已经追踪的文件。怎么办呢？执行下面的命令就好。

first:

```
git rm -r --cached . 
git add .
```

second: 

```
git commit -am "Remove ignored files"
```

last:

```
git pull origin branch-name
git push origin branch-name
```


参考：

[flutter官方demo的.gitignore文件](https://github.com/flutter/flutter/blob/master/.gitignore)

[向现有app添加flutter模块](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)

[How to make Git “forget” about a file that was tracked but is now in .gitignore?](https://stackoverflow.com/questions/1274057/how-to-make-git-forget-about-a-file-that-was-tracked-but-is-now-in-gitignore/1274447#1274447)

