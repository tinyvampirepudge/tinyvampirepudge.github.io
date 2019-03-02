---
layout: post
title:  Android中动态获取最新的git commit_id
date:   2019-01-22 16:04:50 +0800
categories: Android
tag: [git, commit_id, 动态获取commitId]
---

* content
{:toc}



### Android中动态获取最新的git commit_id

项目中有时会遇到需要我们获取最新版本号的逻辑，方便我们定位错误等。

主要步骤如下：
①在app/build.gradle中使用gradle脚本获取到最新的commit_id，并将其写入gradle.properties中。
②将gradle.properties中的属性赋值给BuildConfig。
③在运行时即可通过BuildConfig来获取之前写入的commit_id了。

#### 1、获取commit_id，并将其写入gradle.properties文件中。
首先，我们现在gradle.properties中定义一个字段用来存储获取到的commit_id，如下所示：

```
# 用来存储最新的一次commit_id
GIT_COMMIT_ID=0
```

然后，在app/build.gradle中定义一个方法，用于获取最新的commit_id，代码如下：

```
// 获取当前的git的commit_id
def getGitRevision() {
    return "git rev-parse --short HEAD".execute().text.trim()
}
```

接着，将获取到的commit_id的值赋给GIT_COMMIT_ID即可：

```
// 给gradle.properties中的GITEST_COMMIT_ID赋值
GIT_COMMIT_ID = getGitRevision()
```

#### 2、将commit_id的值赋给BuildConfig
在buildType中添加配置：

```
// Java代码调用
buildConfigField "String", "gitCommitId", "\"${GIT_COMMIT_ID}\""
```

如上所示，这里，我将GIT_COMMIT_ID的值赋值给了BuildConfig中的latestCommitId字段。

#### 3、通过BuildConfig获取commit_id即可

```
private String getGitRevision(){
    return BuildConfig.gitCommitId;
}
```

完整项目请看：
[https://github.com/tinyvampirepudge/GradleDemo](https://github.com/tinyvampirepudge/GradleDemo)

参考:
[Android studio项目中的gradle.properties详解](https://blog.csdn.net/qq_26287435/article/details/84873965)