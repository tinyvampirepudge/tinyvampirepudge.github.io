---
layout: post
title:  显式Intent和隐式Intent解析
date:   2019-08-06 15:18:06 +0800
categories: Intent
tag: [IntentFilter]
---

* content
{:toc}




### 显式Intent和隐式Intent解析

Android中的Intent分为两种类型：

* `显式 Intent`：按名称（完全限定类名）指定要启动的组件。 通常，您会在自己的应用中使用显式 Intent 来启动组件，这是因为您知道要启动的 Activity 或服务的类名。例如，启动新 Activity 以响应用户操作，或者启动服务以在后台下载文件。

* `隐式 Intent` ：不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它。 例如，如需在地图上向用户显示位置，则可以使用隐式 Intent，请求另一具有此功能的应用在地图上显示指定的位置。

#### 显示Intent启动当前应用组件

显式Intent一般是在当前应用中调用，用来启动当前应用的指定组件。下面展示了几种常见的显式Intent启动实例：

```
// 显式Intent调用——构造方法传入Component
Intent intent = new Intent(this, TestActivity.class);
startActivity(intent);
```

```
// 显式Intent调用——setComponent
ComponentName componentName = new ComponentName(this, TestActivity.class);
Intent intent = new Intent();
intent.setComponent(componentName);
startActivity(intent);
```

```
// 显式Intent调用——setClass
Intent intent = new Intent();
intent.setClass(this, TestActivity.class);
startActivity(intent);
```

```
// 显式Intent调用——setClassName(packageContext, className)
Intent intent = new Intent();
//context, String
intent.setClassName(this, "com.tiny.demo.firstlinecode.test.view.TestActivity");
startActivity(intent);
```

```
// 显式Intent调用——setClassName(packageName, className)
Intent intent = new Intent();
//String, String
intent.setClassName("com.tiny.demo.firstlinecode", "com.tiny.demo.firstlinecode.test.view.TestActivity");
startActivity(intent);
```

#### 显示Intent启动其他应用组件
先看下错误示范：
目标Activity配置：不做任何额外配置。

```
<activity
    android:name=".TestExplicitIntentActivity"
    android:label="TestExplicitIntentActivity" />
```

```
// 启动其他应用的Activity，目标Activity不做任何配置，会报SecurityException错误
Intent intent = new Intent();
//String, String
intent.setClassName("com.tinytongtong.dividerviewdemo", "com.tinytongtong.dividerviewdemo.TestExplicitIntentActivity");
startActivity(intent);
```

具体错误如下：

```
2019-08-06 10:02:23.355 7230-7230/com.tiny.demo.firstlinecode E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.tiny.demo.firstlinecode, PID: 7230
    java.lang.SecurityException: Permission Denial: starting Intent { cmp=com.tinytongtong.dividerviewdemo/.TestExplicitIntentActivity } from ProcessRecord{2fe990c 7230:com.tiny.demo.firstlinecode/u0a397} (pid=7230, uid=10397) not exported from uid 10398
        ...
     Caused by: android.os.RemoteException: Remote stack trace:
        at com.android.server.am.ActivityStackSupervisor.checkStartAnyActivityPermission(Landroid/content/Intent;Landroid/content/pm/ActivityInfo;Ljava/lang/String;IIILjava/lang/String;ZZLcom/android/server/am/ProcessRecord;Lcom/android/server/am/ActivityRecord;Lcom/android/server/am/ActivityStack;)Z(libmapleservices.so:4243605)
        ...
```

这个SecurityException异常是完全可以避免的，我们给目标Activity设置`android:exported="true"`属性。

```
<activity
    android:name=".TestExplicitIntentActivity"
    android:exported="true"
    android:label="TestExplicitIntentActivity" />
```

然后再运行，就成功打开目标Activity了。

当然了，我们还有另一种方式打开其他应用的Activity，我们需要给目标Activity设置一个不相关的<intent-filter>。具体配置如下：

```
<activity
    android:name=".TestExplicitIntent1Activity"
    android:label="TestExplicitIntent1Activity">
    <intent-filter>
        <action android:name="com.tinytongtong.dividerviewdemo.action.TestExplicitIntent1Activity" />

        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="com.tinytongtong.dividerviewdemo.category.TestExplicitIntent1Activity" />

        <data
            android:host="www.tiny.com"
            android:mimeType="text/plain"
            android:port="8080"
            android:scheme="http" />
    </intent-filter>
</activity>
```

启动代码：

```
// 启动其他应用的Activity，目标Activity需要设置一个不相关的Intent-Filter
Intent intent = new Intent();
//String, String
intent.setClassName("com.tinytongtong.dividerviewdemo", "com.tinytongtong.dividerviewdemo.TestExplicitIntent1Activity");
startActivity(intent);
```

说了这么多，其实就是为了证明显式Intent是可以启动其他应用的Activity的。

官方是不推荐使用显式Intent启动其他应用的Activity的，我们一般也不会这么写。因为我们启动使用的Intent#setClassName方法的两个参数均是String类型，目标应用的包名和目标应用的全路径都是以String类型体现的，这就是我们应该尽力避免的硬编码了。一旦目标Activity修改了类名、修改了包名或者移动了位置，那么我们之前写的启动代码都会失败，这明显不符合我们的代码规范。

Intent#setClassName源码：

```
public @NonNull Intent setClassName(@NonNull String packageName, @NonNull String className) {
    mComponent = new ComponentName(packageName, className);
    return this;
}
```

所以说，启动其他应用的组件时，应该使用隐式Intent，具体来说就是使用Intent-Filter进行匹配。

#### 隐式Intent启动实例

隐式Intent不会指定特定的组件，而是声明要执行的常规操作，系统会根据Intent的内容去匹配对应的Activity并启动。

官网上是这么介绍的：

```
创建隐式 Intent 时，Android 系统通过将 Intent 的内容与在设备上其他应用的清单文件中声明的 Intent-Filter 进行比较，从而找到要启动的相应组件。 如果 Intent 与 Intent-Filter 匹配，则系统将启动该组件，并向其传递 Intent 对象。 如果多个 Intent 过滤器兼容，则系统会显示一个对话框，支持用户选取要使用的应用。
```

所以说隐式Intent既可以启动当前应用的组件，也可以启动其他应用的组件。下面会给出两个最简单的隐式Intent启动Activity实例。

1、启动当前应用组件的示例如下：
目标activity配置：

```
<activity android:name=".kfysts.chapter01.intent.implicit.ImplicitIntentTestAActivity">
    <intent-filter>
        <action android:name="com.tiny.demo.firstlinecode.kfysts.chapter01.intent.implicit.action.a" />

        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

Intent代码：

```
// 启动当前应用的Activity
Intent intent = new Intent();
//action
intent.setAction("com.tiny.demo.firstlinecode.kfysts.chapter01.intent.implicit.action.a");
//Category可以不设置，因为一般在AndroidManifest.xml会设置Default，startActivity方法中也会默认添加Default。
if (intent.resolveActivity(getPackageManager()) != null) {
    LogUtils.e("match success");
    startActivity(intent);
} else {
    LogUtils.e("match failure");
}
```


2、启动其他应用组件的示例如下：
目标activity配置（其他应用）：

```
<activity
    android:name=".TestImplicitIntentActivity"
    android:label="TestImplicitIntentActivity">
    <intent-filter>
        <action android:name="com.tinytongtong.dividerviewdemo.action.a" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

Intent代码：

```
// 启动其他应用的Activity
Intent intent = new Intent();
//action
intent.setAction("com.tinytongtong.dividerviewdemo.action.a");
//Category可以不设置，因为一般在AndroidManifest.xml会设置Default，startActivity方法中也会默认添加Default。
if (intent.resolveActivity(getPackageManager()) != null) {
    LogUtils.e("match success");
    startActivity(intent);
} else {
    LogUtils.e("match failure");
}
```

#### IntentFilter匹配规则

隐式Intent调用分为两部分，一部分是AndroidManifest中组件的<intent-filter>配置，一部分是Intent对象的构建。

只有当我们构建的Intent对象符合目标组件的<intent-filter>配置的时候，才能成功启动目标组件。

那么如何才能匹配上<intent-filter>的配置呢？这个就是我们要说的IntentFilter的匹配规则。

<intent-filter>中的过滤信息有三种，分别是action、category、data。下面是一个过滤规则的实例：

```
<activity
    android:name=".IActivity"
    android:label="IActivity"
    android:launchMode="singleTask">
    <intent-filter>
        <action android:name=“com.tinytongtong.dividerviewdemo.action.11" />
        <action android:name=“com.tinytongtong.dividerviewdemo.action.22" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name=“com.tinytongtong.dividerviewdemo.category.11" />
        <data
            android:host="www.tiny.com"
            android:mimeType="text/plain"
            android:port="8080"
            android:scheme="http" />
    </intent-filter>
</activity>
```

#### 匹配规则
为了匹配过滤列表，需要同时匹配过滤列表中的action、category、data信息，否则匹配失败。

一个过滤列表中的action、category和data可以有多个，所有的action、category、data分别构成不同类别，同一类别的信息共同约束当前类别的匹配过程。

只有一个Intent同时匹配action、category、data才算完全匹配，只有完全匹配才能成功启动Activity。

另外一点，一个activity中可以有多个intent-filter，一个Intent只要能匹配任何一组Intent-filter即可成功启动对应的activity。

##### action
action是一个字符串，该字符串区分大小写。系统预定义了一些action，同时我们也可以在应用中定义自己的action。

一个<intent-filter>中可以有多个action，此时Intent中的action能够和<intent-filter>中的任何一个action相同即可匹配成功。

另外，<intent-filter>中的action和Intent中的action都是必须的，就是说<intent-filter>中至少指定一个action，同理Intent中也必须设置action，否则就没有任何意义了。

##### category
category也是一个字符串，也区分大小写。系统预定义了一些category，同时我们也可以在应用中定义自己的category。

我们一般说category有默认值，是由于系统在调用startActivity或者startActivityForResult的时候会默认为Intent加上“android.intent.category.DEFAULT”这个category。

因此，我们的<intent-filter>配置中必须添加对应的配置，不然会匹配失败。

```
<intent-filter>
    ...
    <category android:name="android.intent.category.DEFAULT"/>
</intent-filter>
```

Intent中我们可以不设置category，因为系统默认给我们添加了“android.intent.category.DEFAULT”。如果我们要添加category的话，这个category就必须跟</intent-filter>的任意一个匹配，否则会匹配失败。


##### data

###### data语法
data语法如下所示：

```
<data android:scheme="string"

      android:host="string"

      android:port="string"

      android:path="string"

      android:pathPattern="string"

      android:pathPrefix="string"

      android:mimeType="string" />
```

data由两部分组成，mimeType和URI。
mimeType指媒体类型，比如`image/jpeg`、`audio/mpeg4-generic`和`video/*`等，可以表示图片、文本、视频等不同的媒体格式。

URI包含的数据比较多，结构如下所示：

```
<scheme>://<host>:<port>[<path>|<pathPrefix>|<pathPattern>]
```

具体示例如下所示：

```
content://com.example.project:200/folder/subfolder/etc
http://www.baidu.com:80/search/info
```

接下来介绍每一个数据的含义：

①android:scheme
URI的模式，比如http、file、content等。如果URI中没有指定scheme，那么整个URI的其他参数无效，这也意味着URI是无效的。

②android:host
URI的主机名，比如www.baidu.com。如果host未指定，那么整个URI中的其他参数无效，这也意味着URI是无效的。

③Android:port
URI中的端口号，比如80，仅当URI中指定了scheme和host参数的时候port参数才是有意义的。

④android:path、android:pathPrefix、android:pathPattern
这三个参数表述路径信息，其中path表示完整的路径信息；
pathPrefix表示路径的前缀信息；
pathPattern也表示完整的路径信息，但是它里面可以包含通配符`“*”`，`“*”`表示0个或多个任意字符，需要注意的事，由于正则表达式的规范，如果想表示真实的字符串，那么`“*”`要写成`“\\*”`，`“\”`要写成`“\\\\”`。

另外，data有两种特殊写法：下面两种写法是等价的。

```
<intent-filter . . . >
    <data android:scheme="something" android:host="project.example.com" />
    . . .
</intent-filter>
```

```
<intent-filter . . . >
    <data android:scheme="something" />
    <data android:host="project.example.com" />
    . . .
</intent-filter>
```

###### data的匹配规则

data是非必须的，可以不设置。但是如果在</intent-filter>定义了data，那么Intent中也必须设置可匹配的data。

再来看看data内部：

</intent-filter>的URI有默认值file和content，如果设置了URI，则默认值就失效。

</intent-filter>的mimeType可以不设置。

data的匹配意味着mimeType和URI同时匹配。

综合以上所有情况，这里分几种情况：

①data中只配置了mimeType:

```
<intent-filter>
    ...
    <data android:mimeType="image/*" />
</intent-filter>
```

由于这里只配置了mimeType，所以会使用默认的URI，默认的URI的scheme为file或content。

所以使用下面这两段代码可以匹配：

```
intent.setDataAndType(Uri.parse("content://maolegemi"), "image/jpeg");
```

```
// 下面这段在api大于24的版本上会报错FileUriExposedException，需要将file替换为content
intent.setDataAndType(Uri.parse("file://maolegemi"), "image/jpeg");
```

②data中只配置了URI:

```
<intent-filter>
    ...
    <data
        android:host="www.tiny.com"
        android:port="8080"
        android:scheme="http" />
</intent-filter>
```

对应匹配代码如下：

```
intent.setDataAndType(Uri.parse("http://www.tiny.com:8080/abcdefg"), null);
```

③data中同时配置了mimeType和URI:

```
<intent-filter>
    ...
    <data
        android:host="www.tiny.com"
        android:port="8080"
        android:mimeType="text/plain"
        android:scheme="http" />
</intent-filter>
```

对应的匹配代码如下:

```
intent.setDataAndType(Uri.parse("http://www.tiny.com:8080/abcdefg"), "text/plain");
```

#### 总结

综上所述，对<intent-filter>而言，必不可少的配置是<cation>和默认的category。

对Intent而言，必不可少的是action，因为默认的category会添加。

如果<intent-filter>定义了data，不管mimeType是否设置，Intent中都必须设置uri，因为uri有默认值。

#### 参考

Android开发艺术探索

[https://developer.android.com/guide/components/intents-filters?hl=zh-cn](https://developer.android.com/guide/components/intents-filters?hl=zh-cn)

[https://developer.android.com/guide/topics/manifest/data-element](https://developer.android.com/guide/topics/manifest/data-element)


