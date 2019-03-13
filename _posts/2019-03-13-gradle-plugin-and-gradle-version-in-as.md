---
layout: post
title:  Android Studio中修改gradle插件版本和Gradle版本
date:   2019-03-13 14:43:53 +0800
categories: Gradle
tag: [Gradle]
---

* content
{:toc}



### Android Studio中修改gradle插件版本和Gradle版本

Android项目中，我们一般要设置gradle插件版本和gradle版本。

#### gradle插件

项目根目录下的build.gradle文件中，通过classpath可以指定gradle插件的版本。

```
buildscript {
    repositories {
        // Gradle 4.1 and higher include support for Google's Maven repo using
        // the google() method. And you need to include this repo to download
        // Android Gradle plugin 3.0.0 or higher.
        google()
        ...
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.3.2'
    }
}
```

在AndroidStudio中，你也可以通过`File > Project Structure > Project Menu`来指定gradle插件的版本，如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190313-142127%402x.png)

##### 如何更新gradle插件

当你指定的gradle插件版本没有下载下来时，在你下次构建项目时会自动下载gradle插件，或者在Android Studio的工具类中，进行`Tools > Android > Sync Project with Gradle Files`操作。

#### 更新Gradle版本

下图展示了gradle插件和gradle版本的对应关系：
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190313-142445.png)

指定Gradle的版本也有两种方式，第一种是通过Android Studio工具栏中`File > Project Structure > Project`的方式，如下图所示：
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190313-143438%402x.png)

另一种是方法是通过修改项目根目录下的`gradle/wrapper/gradle-wrapper.properties`文件，如下所示：
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190313-143505%402x.png)

参考：

[https://developer.android.com/studio/releases/gradle-plugin](https://developer.android.com/studio/releases/gradle-plugin)

