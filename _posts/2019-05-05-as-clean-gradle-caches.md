---
layout: post
title:  Android Studio中如何清理gradle缓存
date:   2019-05-05 12:16:01 +0800
categories: Gradle缓存
tag: [Gradle缓存]
---

* content
{:toc}


### Android Studio中如何清理gradle缓存
as使用过程中，经常会遇到gradle缓存问题，常用的清理方式如下：

1、Build --> Clean Project

2、Build --> Rebuild Project

3、File -> Invalidate Caches/Restart

4、删除项目根目录下`.idea/caches`和`.idea/libraries`目录，然后`Invalidate Caches/Restart`

5、在as终端中执行`./gradlew clean`



