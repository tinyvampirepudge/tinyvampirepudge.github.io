---
layout: post
title:  Android Studio插件的源文件位置——mac端
date:   2019-05-29 18:21:47 +0800
categories: Android Studio
tag: [Android Studio]
---

* content
{:toc}



@[toc]
### Android Studio插件的源文件位置——mac端

Android Studio中我们可以通过菜单栏的`Android Studio --> preferences --> plugins`来查看我们安装的插件。这里介绍下插件的实际安装位置。

#### 系统安装的插件
在第一次安装as的时候，系统会为我们安装一堆插件，他们的位置位于`/Applications/Android Studio.app/Contents/plugins`目录，如下图所示：

```
tinytongtongdeMacBook-Pro% pwd
/Applications/Android Studio.app/Contents/plugins
tinytongtongdeMacBook-Pro% ls -al
total 0
drwxrwxr-x@ 41 tinytongtong  admin  1312 Apr 11 07:42 .
drwxrwxr-x@ 12 tinytongtong  admin   384 May 26 12:10 ..
drwxrwxr-x@  3 tinytongtong  admin    96 Apr 11 07:42 Groovy
drwxrwxr-x@  3 tinytongtong  admin    96 Apr 11 07:42 IntelliLang
drwxrwxr-x@  4 tinytongtong  admin   128 Apr 11 07:42 Kotlin
drwxrwxr-x@  4 tinytongtong  admin   128 Apr 11 07:42 android
drwxrwxr-x@  3 tinytongtong  admin    96 Apr 11 07:42 android-apk
drwxrwxr-x@  3 tinytongtong  admin    96 Apr 11 07:42 android-ndk
...
tinytongtongdeMacBook-Pro%
```

当然，你在也可以在Finder中查看，步骤如下：
①在Finder中打开Applications目录，选中`Android Studio`，右键选择`Show Package Contents`，就进入到`Android Studio`的安装目录了，如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190529-180514%402x.png)

②选择Content/plugins目录，就可以看到系统预安装的插件了，如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190529-180536%402x.png)

#### 自己安装的插件

我们一般都会自己安装插件，安装的目录位于`/Users/xxx/Library/Application Support/AndroidStudio3.4`下，后面的3.4代表Android Studio的版本。

目录内容如下所示，可以看出都是我自己后来安装的插件。

```
tinytongtongdeMacBook-Pro% pwd
/Users/tinytongtong/Library/Application Support/AndroidStudio3.4
tinytongtongdeMacBook-Pro% ls -al
total 9480
drwxr-xr-x  24 tinytongtong  staff      768 May 29 17:53 .
drwx------+ 78 tinytongtong  staff     2496 May 29 18:14 ..
-rw-r--r--   1 tinytongtong  staff     6148 Nov  6  2018 .DS_Store
-rw-r--r--   1 tinytongtong  staff    85559 Aug  7  2018 ADBWIFI.jar
drwxr-xr-x   3 tinytongtong  staff       96 May 28 14:07 Codota
drwxr-xr-x   4 tinytongtong  staff      128 May 26 12:10 DBNavigator
drwxr-xr-x   3 tinytongtong  staff       96 May 26 12:10 Dart
-rw-r--r--   1 tinytongtong  staff   157627 Aug  7  2018 GsonFormat.jar
...
tinytongtongdeMacBook-Pro%
```

当然了，你也可以在Finder中查看，如下所示：需要注意的是，Library是隐藏目录，你需要让他显示出来。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190529-181858%402x.png)

#### 参考

[https://stackoverflow.com/questions/23537065/where-is-the-plugin-folder-for-android-studio-on-mac](https://stackoverflow.com/questions/23537065/where-is-the-plugin-folder-for-android-studio-on-mac)

