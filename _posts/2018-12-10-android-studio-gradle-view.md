---
layout: post
title:  Android Studio项目中的Gradle视图内容
date:   2018-12-10 11:40:13 +0800
categories: [Android]
tag: [Gradle]
---

* content
{:toc}



### Android Studio项目中的Gradle视图内容

使用Android Studio的同学都知道，我们可以方便的在Gradle视图中查看项目中的Task, 双击Task就可以执行它。如下图所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20181210-105515.png)

那么视图中的元素跟我们的项目都有什么对应关系呢？请往下看。

#### Gradle视图中的子元素
我们先看下项目的结构，项目中包含三个module: app、mylibrary、myotherlibrary。app是application类型，mylibrary和myotherlibrary是library类型。它们的依赖关系是，app依赖于mylibrary，mylibrary依赖于myotherlibrary。如下图所示。
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20181210-110050.png)

跟项目结果对应，Gradle视图中的元素也包含了三个module对应的Gradle对象：`:app、:mylibrary、:myotherlibrary`，另外它还有一个root类型的gradle对象——`gradleDemo(root)`。

关于这些信息，我们都可以通过执行gradle命令行来查看。当然了，你可能需要安装gradle，安装去官网即可。如果不想安装的话，用`./gradlew`提到我使用的`gradle`命令也可以。

在项目根目录下，执行`gradle projects`，效果如下所示：
```
tinytongtongdeMacBook-Pro% gradle projects

> Configure project :

> Configure project :app

> Configure project :mylibrary

> Configure project :myotherlibrary

> Task :projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'gradleDemo'
+--- Project ':app'
+--- Project ':mylibrary'
\--- Project ':myotherlibrary'

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :app:tasks

...

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
tinytongtongdeMacBook-Pro% 
```
输出结果跟我们的Gradle视图中的内容是对应的。

#### 查看具体的任务信息
在项目根目录下执行`gradle tasks --all`命令，查看项目的所有task及其信息，结果如下所示：
```
tinytongtongdeMacBook-Pro% gradle tasks --all
> Configure project :

> Configure project :app

> Configure project :mylibrary

> Configure project :myotherlibrary

> Task :tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Android tasks
-------------
app:androidDependencies - Displays the Android dependencies of the project.
mylibrary:androidDependencies - Displays the Android dependencies of the project.
myotherlibrary:androidDependencies - Displays the Android dependencies of the project.
app:signingReport - Displays the signing info for each variant.
mylibrary:signingReport - Displays the signing info for each variant.
myotherlibrary:signingReport - Displays the signing info for each variant.
...

Build tasks
-----------
app:assemble - Assembles all variants of all applications and secondary packages.
mylibrary:assemble - Assembles all variants of all applications and secondary packages.
myotherlibrary:assemble - Assembles all variants of all applications and secondary packages.
...

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Cleanup tasks
-------------
...

Help tasks
----------
...

Install tasks
-------------
...

Verification tasks
------------------
...

Other tasks
-----------
...

Deprecated Gradle features were used in this build, making it incompatible with Gradle 6.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/5.0/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
tinytongtongdeMacBook-Pro% 
```

上图的输出内容，跟Gradle视图中的gradleDemo(root)对象的内容是对应的，如下图所示。你可以自己去核对task细节。另外，上面的输出内容中不仅列出了对应的task名称，还有他们的简介。
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20181210-113027.png)


#### 如何在命令行中执行Gradle视图中的任务
其实我们刚才在命令行中执行的任务，都是可以在Gradle视图中双击来执行的。如下图所示，分别对应刚才执行的`gradle projects`和`gradle tasks`命令。
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20181210-113349.png)

这里给出说明，Gradle执行任务：`gradle task-name`：
* `gradle clean`，是执行清理任务
* `gradle properties`，用来查看所有属性信息

所以说，如果你想执行某个Gradle中的Task，你找到它的名字之后，然后打开命令行，进入项目目录，然后执行`gradle tasl-name`命令，刚才那个task就执行了，是不是逼格满满。

需要说明的是，Gradle视图中的gradleDemo(root)对象，它的内容是子module对应的Gradle对象的并集。你可以自己动手试试。

