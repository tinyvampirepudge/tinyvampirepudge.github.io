---
layout: post
title:  Android使用本地svg及不显示问题解决
date:   2019-07-25 14:50:17 +0800
categories: svg
tag: [svg]
---

* content
{:toc}



### Android使用本地svg及不显示问题解决

今天UI小姐姐给切图时，里面有部分svg文件。本来想让UI小姐姐全部换成png格式，转念一想那岂不是太low，Android又不是不支持svg。

#### 错误示范

于是就直接将svg图片copy进res/drawable目录下，然后给ImageView的background属性引用，接下来就啪啪打脸了，xml文件直接标红，build之后也报aapt异常，这下就尴尬了。

具体异常如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-162401%402x.png)

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-162701%402x.png)

于是就赶紧老老实实的去翻官网文档——[添加多密度矢量图形](https://developer.android.com/studio/write/vector-asset-studio?hl=zh-CN#running)。

#### 引入本地矢量图正确步骤

第一步，添加Gradle配置

具体来说，是在app/build.gradle文件中添加如下配置：

```
android {
  defaultConfig {
    //vector to svg, and need
    vectorDrawables.useSupportLibrary = true
  }
}

dependencies {
  implementation 'com.android.support:appcompat-v7:28.0.0'
}
```

需要注意的是，`com.android.support:appcompat-v7`的版本需要`23.2`及以上，不过现在绝大部分项目都已经支持了。

第二步，导入本地svg文件

①在Android Studio的Project窗口中，切换到Android视图，具体如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-164220.png)

②然后右键点击`res`文件夹，选择`New > Vector Asset`，
具体如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-164307.png)

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-164326.png)

③在打开的对话框中，选择本地文件，具体如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-164341.png)

需要注意的是，我们的svg源文件名称中不能包含汉字，只支持小写字符、下划线和数字。

④最后点击`Next` --> `Finish`即可，就会生成相应的xml文件。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-164440.png)

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190724-164453.png)

第三步，当做资源文件使用

```
<ImageView
    android:layout_width="50dp"
    android:layout_height="50dp"
    android:layout_centerHorizontal="true"
    android:layout_below="@id/tvi"
    android:layout_marginTop="20dp"
    app:srcCompat="@drawable/ic_icon_delete"
    />
```

需要注意的是这里要使用`app:srcCompat`，而不是`android:src`。

#### 注意事项

还有一点需要注意，如果你的Activity不是继承自`AppCompatActivity`，那么就使用如下代码显示设置svg背景：

```
imageView.setImageResource(R.drawable.ic_svg_image);
```

#### 参考
[添加多密度矢量图形](https://developer.android.com/studio/write/vector-asset-studio?hl=zh-CN#running)

[Svg not visible in device but visible in android xml](https://stackoverflow.com/questions/43594751/svg-not-visible-in-device-but-visible-in-android-xml)

