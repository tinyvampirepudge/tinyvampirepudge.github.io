---
layout: post
title:  Android加载drawable中图片后自动缩放的原理
date:   2019-08-15 16:29:34 +0800
categories: drawable
tag: [drawable]
---

* content
{:toc}




### Android加载drawable中图片后自动缩放的原理

日常开发中我们少不了要根据设计图绘制UI，一般而言设计师给的都是设计图都是`750*1334`的，给的切图也一般是`2x`、`3x`图。

简单起见，我们只将对应的`2x`图标放到`res/drawable-xhdpi`目录下即可。

当然了，对于要求高的图标，我们需要添加对应的多套图，分别放置到`drawable-mdpi`、`drawable-hdpi`、`drawable-xhdpi`、`drawable-xxhdpi`、`drawable-xxxhdpi`、目录下。

```
ps: `drawable-ldpi`太低了，可以忽略掉。
```

图标的容器我们一般使用ImageView，代码如下：

```
<ImageView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/empty_icon" />
```

实际开发中我们直接给对应的src属性设置对应的图片资源id即可，不同机型上的适配工作，Android系统会自动帮我们完成。

系统会自动根据当前机型的dpi，到对应的drawable目录下获取图片，接着再对图片进行适当的缩放操作，最后完成显示。

关于dpi和drawable是如何匹配的，请移步郭霖大神的[Android drawable微技巧，你所不知道的drawable的那些细节](https://blog.csdn.net/guolin_blog/article/details/50727753)。

这篇文章中有一个很重要的dpi匹配的表，如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-145647.png)

这里我想讲下Android在获取到对应的图片后，是如何进行对应的缩放操作的。

#### 实例讲解

我们先从一个例子讲起。

##### 1、图片准备
我们先准备对应的五张图片 `empty_icon` ，他们的分辨率分别是

```
mdpi		106*106
hdpi		159*159``
xhdpi		213*213
xxxhdpi 	319*319
xxxhdpi		426*426
```

我们看下官网的说明，如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-143040.png)

这说明我们的图片是符合比例规则的，接着将他们放入到对应的drawable目录下。

ps：这几张图片来自于bravh这个库，如有侵权，请联系我，侵删。

##### 2、展示图片信息

接着我们将这个图片设置给ImageView对象，代码如下：

```
<ImageView
    android:id="@+id/iv"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"
    android:src="@drawable/empty_icon" />
```

然后获取当前图片的实际宽高，代码如下：

```
iv.post {
    var sb = StringBuffer("图片实际宽高：\n")
    sb.append("width: " + iv.width)
    sb.append("\n")
    sb.append("height: " + iv.height)

    tvIvInfo.text = sb.toString()
}
```

接着运行代码，我们看下现象，效果图如下：这个测试机的dpi为440。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-150914%402x.png)

咦，这里图片的尺寸怎么不对呢？drawable-xxhdpi文件夹下的图片尺寸命名是`319`啊，这里为何不一致，而且我们也没有这个尺寸的图片啊，为什么呢？

莫慌，我们这次换一个xxhdpi的模拟器试下，运行效果如下图：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-150942.png)

这次效果正确了吧，可是这个缩放原理是什么呢？

##### 3、图片缩放原理解析

我们都知道Android会为我们自动缩放drawable中的图片，可是到底缩放了多少，什么情况下缩放呢？

我们先看下dpi和drawable的对应关系：需要说明的是，这里的范围都是`不包含头包含尾`，可以用`(a,b]`来表示。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-145647.png)


我们第一次运行的测试机的dpi为440，第二次的为480，他两对应的dpi等级均是`xxhdpi`，所以它们对应的drawable目录为`drawable-xxhdpi`，我们这里对应的图片的尺寸应该是`319*319`。

可是为啥两个手机上显示的尺寸不一样呢？一个是`292*292`，一个是`319*319`。有什么规律呢？

眼尖的同学应该已经看出来了，`实际的图片尺寸`的`缩放比例`跟`实际dpi`和`对应dpi等级`相关，规律如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-152253%402x.png)

对第一个实例来说，`实际dpi`这里对应的分别是440，`对应的dpi`通过上面的表格可以得出为480，`对应drawable图片尺寸`为319px，`实际显示尺寸`为我们获取到的292.

上述公式转换下，如下所示：计算实际显示尺寸的值。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-162148%402x.png)

针对某个图片来说，`实际dpi`、`对应的dpi等级`、`对应drawable下图片尺寸`这三个值都是已知的，根据这三个值我们就可以获取到实际显示的尺寸。

##### 4、结论验证

接下来我们验证下我们这个结论。我们先写一个工具类，根据`实际dpi`获取`对应的dpi等级`，代码如下所示：

```
/**
 * 根据实际dpi获取对应的dpi等级。
 * 登记表参见 https://blog.csdn.net/guolin_blog/article/details/50727753
 * @param srcDpi
 * @return
 */
public static int getTargetDpi(int srcDpi) {
    int targetDpi = 0;

    if (srcDpi <= 120) {// ldpi
        targetDpi = 120;
    } else if (srcDpi <= 160) {// mdpi
        targetDpi = 160;
    } else if (srcDpi <= 240) {// hdpi
        targetDpi = 240;
    } else if (srcDpi <= 320) {// xhdpi
        targetDpi = 320;
    } else if (srcDpi <= 480) {// xxhdpi
        targetDpi = 480;
    } else if (srcDpi <= 640) {// xxxhdpi
        targetDpi = 640;
    }

    return targetDpi;
}
```

接着我们再写一个方法，根据对应的dpi等级，可以获取 R.drawable.empty_icon 的实际尺寸：

```
/**
 * 根据对应的dpi等级，获取 R.drawable.empty_icon 对应的实际尺寸
 */
fun getDrawableSize(targetDpi: Int): Int {
    var drawableSize = 0;
    when (targetDpi) {
        120, 160 -> drawableSize = 106
        240 -> drawableSize = 159
        320 -> drawableSize = 213
        480 -> drawableSize = 319
        640 -> drawableSize = 426
    }
    return drawableSize
}
```

接下来就是核心代码了，如下所示：

```
// 获取当前手机实际的dpi
var srcDpi = this.resources.displayMetrics.densityDpi

// 获取实际dpi对应的dpi等级
var targetDpi = ScreenUtils.getTargetDpi(srcDpi)

// 根据dpi等级，获取 R.drawable.empty_icon 实际尺寸
var drawableSize = getDrawableSize(targetDpi)

// 根据实际尺寸计算缩放后的尺寸
var resultSize = drawableSize * 1.0f / targetDpi * srcDpi

tvExpect.text = String.format("期望结果：%.2f", resultSize)
```

我们先在刚才的440dpi的手机上验证下，效果如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-153548%402x.png)

可以看出，我们计算出的`期望值`，`四舍五入`后就是实际显示的值。

接下来看下mdpi的手机上的显示效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-154452%402x.png)

接下来看下hdpi的手机上的显示效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-154625%402x.png)

接下来看下xhdpi的手机上的显示效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-154845.png)

接下来看下420hdpi的手机上的显示效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-155110%402x.png)

接下来看下440hdpi的手机上的显示效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-155411.png)

接下来看下xxhdpi的手机上的显示效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-155523.png)

接下来看下560hdpi的手机上的显示效果：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-155801.png)

由于Android Studio自带的模拟器中没有xxxhdpi等级的模拟器，所以这里就只能使用560dpi近似展示了，他两是一个dpi等级的。

#### 总结

通过上述分析，我们以后可以放心的使用Android自带的drawable文件进行图标的适配了，只要我们正确的提供了多套图，Android系统会自动为我们加载合适的图片，再进行适当的缩放。

之前不是很懂这个原理的时候，我都是通过给ImageView设置高度来适配的，代码如下：

```
<ImageView
    android:id="@+id/iv"
    android:layout_width="106dp"
    android:layout_height="106dp"
    android:layout_gravity="center_horizontal"
    android:background="@drawable/empty_icon" />
```

这次搞清楚原理之后，这种比较苟的方式就可以废弃了，直接使用Android提供的默认方式即可，这样ImageView的书写方式就极其简单了：

```
<ImageView
    android:id="@+id/iv"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"
    android:src="@drawable/empty_icon" />
```

#### github项目地址
具体demo地址位于：
[https://github.com/tinyvampirepudge/Android_Base_Demo](https://github.com/tinyvampirepudge/Android_Base_Demo)

具体页面：[https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/drawable/DrawableTest1Activity.kt](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/drawable/DrawableTest1Activity.kt)

打开方式如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190815-155801.png)

#### 参考

[Android drawable微技巧，你所不知道的drawable的那些细节](https://blog.csdn.net/guolin_blog/article/details/50727753)

[https://developer.android.com/guide/topics/resources/providing-resources.html?hl=zh-cn#AlternativeResources](https://developer.android.com/guide/topics/resources/providing-resources.html?hl=zh-cn#AlternativeResources)



