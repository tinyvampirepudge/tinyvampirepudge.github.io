---
layout: post
title:  gradle中统一配置版本的小技巧
date:   2018-08-29 10:09:47 +0800
categories: Gradle
tag: [Gradle]
---

* content
{:toc}



# gradle中统一配置版本的小技巧。

## 在Project/build.gradle中定义，在module/build.gradle中使用
### 1、直接在Project/build.gradle中定义和引用：

```javascript
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext.compileSdkVersion = 26
    ext.targetSdkVersion = 26
    ext.support_appcompat_v7 = '26.1.0'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

...
```

数字引用：

```javascript
compileSdkVersion rootProject.ext.compileSdkVersion
```

三方库版本号引用：

```javascript
implementation "com.android.support:appcompat-v7:$support_appcompat_v7"
```

### 2、单独在xxx.gradle中定义和引用：
①在Project 层级下新建config.build文件（这里的config可以替换为任何你喜欢的名字），在里面书写配置信息：

```javascript
ext {//定义所有project公用参数

    android = [
            compileSdk: 27,
            buildTools: "27.0.3",
            minSdk    : 19,
            minLimitSdk: 19,//限制低版本用户安装
            targetSdk : 27,
    ]

    dependencies = [
            // App dependencies
            junit                : '4.12',
            espresso             : '2.2.2',
            supportLibraryVersion: '27.1.1',
            supportPercentVersion: '25.3.1',
            butterknife          : '8.8.1',
            gson                 : '2.7',
            retrofit             : '2.4.0',
            rxjava               : '1.1.6',
            rxandroid            : '1.2.1',
            loggingInterceptor   : '3.1.0',
            stetho               : '1.4.2',
            guavaVersion         : '18.0',
            leakcanary           : '1.5.4'
    ]
}
```

②在Project/build.gradle中引用刚才定义好的config.gradle文件：

```javascript
apply from: "config.gradle"
```

③数字引用：

```javascript
compileSdkVersion rootProject.ext.android.compileSdk
```

④三方库版本号引用：

```javascript
api "com.android.support:appcompat-v7:$rootProject.ext.dependencies.supportLibraryVersion"
```

## 在Project/gradle.properties中配置，在mudule/build.gradle中使用.
①在Project/gradle.properties中定义：

```javascript
COMPILE_SDK_VERSON = 26
BUILD_TOOLS_VERSION = 25.0.2

SUPPORTV7_VERSON=25.0.1
```

②引用到的变量默认是String类型，如果需要in类型，需要在后面添加 as int 声明

```javascript
compileSdkVersion COMPILE_SDK_VERSON as int
buildToolsVersion BUILD_TOOLS_VERSION

compile "com.android.support:appcompat-v7:${SUPPORTV7_VERSON}"
```

