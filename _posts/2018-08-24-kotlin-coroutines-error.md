---
layout: post
title:  kotlin协程库报错“Program type already present”解决
date:   2018-08-24 18:51:54 +0800
categories: kotlin
tag: [kotlin, coroutines]
---

* content
{:toc}



### kotlin协程库报错“Program type already present”解决

最近在学习kotlin，学习到协程库这一块了，针对Android的话就是coroutines-android库。本来学习就不容易了，再加上kotlin现在还处于快速变化期，那个酸爽简直了，废话不多说，进入正题。
		
[kotlinx.coroutines的github地址](https://github.com/Kotlin/kotlinx.coroutines)
		
[协程库中关于Android的文档说明](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/coroutines-guide-ui.md#android)
		
为了在Android中kotlin协程库的一些特性，我们就需要引入coroutines-android库，代码如下：
ps: 这里我使用的kotlin-gradle-plugin插件和kotlinx-coroutines-android的版本都是最新的，分别是1.2.61和0.25.0.
1、build.gradle:

```javascript
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext.kotlin_version = '1.2.61'
    ext.coroutines_version = '0.25.0'
    repositories {
        mavenCentral()
        jcenter()
        google()
        maven { url 'https://maven.google.com' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.4'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        mavenCentral()
        jcenter()
        google()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

2、app/build.gradle:

```javascript
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "com.example.tinytongtong.kotlincoroutineapplication"
        minSdkVersion 15
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    // Configure only for each module that uses Java 8
    // language features (either in its source code or
    // through dependencies).
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'com.android.support:appcompat-v7:27.1.1'
    implementation 'com.android.support.constraint:constraint-layout:1.1.2'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'

    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$coroutines_version"

    //okhttp
    api 'com.squareup.okhttp3:okhttp:3.11.0'
    api 'com.squareup.okhttp3:logging-interceptor:3.4.1'
}

kotlin {
    experimental {
        coroutines "enable"
    }
}

```

2、配置好了之后，先sync再run，然后的Build窗口就会出现下面这个页面：

![kotlin协程库报错](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20180824-182658.png)

### 问题解决：
最简单粗暴的方式就是回退版本，如下所示：

```javascript
ext.kotlin_version = '1.2.61'
ext.coroutines_version = '0.25.0'
```

接下来还是sync --> run，代码就可以跑起来了。


