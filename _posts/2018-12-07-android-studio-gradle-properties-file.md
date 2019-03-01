---
layout: post
title:  Android studio项目中的gradle.properties详解
date:   2018-12-07 14:18:38 +0800
categories: [Android]
tag: [Gradle, gradle.properties]
---

* content
{:toc}



### Android studio项目中的gradle.properties详解

在使用Android Studio新建Android项目之后，在项目根目录下会默认生成一个gradle.properties文件，我们可以在里面做一些Gradle文件的全局性的配置，也可以将比较私密的信息放在里面，防止泄露。

下面我们就来分析下IDE自动生成的gradle.properties文件，及其常见的用法。

#### gradle.properties文件模板

```
# Project-wide Gradle settings.

# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.

# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html

# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx1536m

# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true

```

#### gradle.properties注释的翻译
项目级别的Gradle配置文件

亲爱的Android Studio开发者们：
在IDE中指定的的配置可以覆盖这个文件中相应的配置。

如果想了解更多如何配置编译环境，访问[http://www.gradle.org/docs/current/userguide/build_environment.html](http://www.gradle.org/docs/current/userguide/build_environment.html)


#### 在各个模块的build.gradle里面直接引用

gradle.properties里面定义的属性是全局的，可以在各个模块的build.gradle里面直接引用.

在gradle.properties文件中新增下面属性：
```
COMPILE_SDK_VERSION=28
MIN_SDK_VERSION=15
TARGET_SDK_VERSION=28

SUPPORT_APPCOMPAT_V7_VERSION=28.0.0
```

注意：在gradle.properties中定义的属性默认是String类型的，如果需要int类型，需要添加`XXX as int`后缀。

在根目录的settings.gradle中引用：
```
// 输出Gradle对象的一些信息
def printGradleInfoInRoot(){
println "COMPILE_SDK_VERSION:" + COMPILE_SDK_VERSION
}

printGradleInfoInRoot()
```

在app/build.gradle文件中引用：
```
android {
compileSdkVersion COMPILE_SDK_VERSION as int
defaultConfig {
applicationId "com.tinytongtong.gradle"
minSdkVersion MIN_SDK_VERSION as int
targetSdkVersion TARGET_SDK_VERSION as int
...
}
...
}

dependencies {
...
implementation "com.android.support:appcompat-v7:${SUPPORT_APPCOMPAT_V7_VERSION}"
...
}
```

另外，在gradle.properties中配置的属性，我们可以在执行`gradle properties`后看到，结果如下：
```
...

> Task :properties

------------------------------------------------------------
Root project
------------------------------------------------------------

COMPILE_SDK_VERSION: 28
INTERNAL__CHECKED_MINIMUM_PLUGIN_VERSIONS: true
MIN_SDK_VERSION: 15
SUPPORT_APPCOMPAT_V7_VERSION: 28.0.0
TARGET_SDK_VERSION: 28

...
```

#### 在Java代码或者xml布局文件中使用

1、在gradle.properties中添加配置：
```
# Java 代码使用
DEFAULT_NICK_NAME=maolegemi
DEFAULT_NUMBER=10086

# xml文件调用
USER_NAME=wangdandan
TEXT_SIZE=20sp
TEXT_COLOR=#ef5350
```

2、在app/build.gradle中读取属性，对这些属性进行转化：
```
android {
...
buildTypes {
release {
minifyEnabled false
proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
}

debug{
// Java代码调用
buildConfigField "String", "defaultNickName", "\"${DEFAULT_NICK_NAME}\""
buildConfigField "Integer", "defaultNumber", DEFAULT_NUMBER

// xml布局文件调用
resValue "string", "user_name", "${USER_NAME}"
resValue "dimen", "text_size", "${TEXT_SIZE}"
resValue "color", "text_color", "${TEXT_COLOR}"
}
}
}
```
上述代码中的buildConfigField方法的是BuildType#buildConfigField，包含三个参数：类型、变量名、变量值。resValue方法定义在BuildType#resValue中，也包含三个参数：类型、变量名、变量值。

3、在Java中调用：
```
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);

System.out.println("defaultNickName:" + BuildConfig.defaultNickName);
System.out.println("defaultNickName.type:" + BuildConfig.defaultNickName.getClass().getSimpleName());

System.out.println("defaultNumber:" + BuildConfig.defaultNumber);
System.out.println("defaultNumber.type:" + BuildConfig.defaultNumber.getClass().getSimpleName());
}
```

输出结果：符合预期。
```
2018-12-07 13:59:38.703 9854-9854/com.tinytongtong.gradle I/System.out: defaultNickName:maolegemi
2018-12-07 13:59:38.703 9854-9854/com.tinytongtong.gradle I/System.out: defaultNickName.type:String
2018-12-07 13:59:38.704 9854-9854/com.tinytongtong.gradle I/System.out: defaultNumber:10086
2018-12-07 13:59:38.704 9854-9854/com.tinytongtong.gradle I/System.out: defaultNumber.type:Integer
```

4、在xml布局文件中调用：
```
<TextView
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="Hello World!"
app:layout_constraintBottom_toBottomOf="parent"
app:layout_constraintLeft_toLeftOf="parent"
app:layout_constraintRight_toRightOf="parent"
app:layout_constraintTop_toTopOf="parent"
android:textSize="@dimen/text_size"
android:textColor="@color/text_color"
/>
```
运行结果:
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/50EB9F76E845152940E36CA5974A8E5C.jpg)



#### 参考

[android studio 中使用gradle.properties](https://www.jianshu.com/p/a6d1c4a0550e)

项目地址：[https://github.com/tinyvampirepudge/GradleDemo](https://github.com/tinyvampirepudge/GradleDemo)


