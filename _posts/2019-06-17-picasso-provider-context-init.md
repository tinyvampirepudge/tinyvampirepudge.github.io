---
layout: post
title:  PicassoProvider初始化时机
date:   2019-06-17 10:04:24 +0800
categories: Picasso
tag: [PicassoProvider]
---

* content
{:toc}



### PicassoProvider初始化时机
Picasso版本：

```
    // picasso
    implementation 'com.squareup.picasso:picasso:2.71828'
```

在学习Picasso源码的过程中，发现了Picasso对象的初始化不需要传入上下文对象了，示例代码如下：

```
        // 加载网络图片
        Picasso.get()
                .load("xxx")
                .into(iv1);
```

我们可以通过`Picasso.get()`方法直接获取Picasso的对象。

#### Picasso.get()源码
那么它到底需不需要context对象呢？我们点进去看一下。

```
  @SuppressLint("StaticFieldLeak") 
  static volatile Picasso singleton = null;
  
  ...
  
  public static Picasso get() {
    if (singleton == null) {
      synchronized (Picasso.class) {
        if (singleton == null) {
          if (PicassoProvider.context == null) {
            throw new IllegalStateException("context == null");
          }
          singleton = new Builder(PicassoProvider.context).build();
        }
      }
    }
    return singleton;
  }
```

我们可以看到，Picasso使用的是单例模式，是典型的`DCL（Double-checked-locking）——双重检查锁模式`。

Picasso对象singleton的最终初始化时通过给`Picasso.Builder`构造方法传入一个`PicassoProvider.context`参数，然后调用`Picasso.Builder#build()`方法完成`Picasso`对象的构建。

这说明Picasso还是需要Context对象的，只不过不需要用户传入。

#### Picasso.Builder

我们看下Picasso.Builder的实现：

```
    /** Start building a new {@link Picasso} instance. */
    public Builder(@NonNull Context context) {
      if (context == null) {
        throw new IllegalArgumentException("Context must not be null.");
      }
      this.context = context.getApplicationContext();
    }
```

这里可以明显的看出，我们传入的`PicassoProvider.context`参数就是`Context`类型，那么它是何时初始化的呢？

#### PicassoProvider

我们看下PicassoProvider的实现：

```
@RestrictTo(LIBRARY)
public final class PicassoProvider extends ContentProvider {

  @SuppressLint("StaticFieldLeak") static Context context;

  @Override public boolean onCreate() {
    context = getContext();
    return true;
  }

  @Nullable @Override
  public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection,
      @Nullable String[] selectionArgs, @Nullable String sortOrder) {
    return null;
  }

  @Nullable @Override public String getType(@NonNull Uri uri) {
    return null;
  }

  @Nullable @Override public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
    return null;
  }

  @Override public int delete(@NonNull Uri uri, @Nullable String selection,
      @Nullable String[] selectionArgs) {
    return 0;
  }

  @Override
  public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection,
      @Nullable String[] selectionArgs) {
    return 0;
  }
}
```

PicassoProvider继承自ContentProvider，PicassoProvider唯一的作用是`在onCreate方法内部初始化了PicassoProvider.context成员变量`，PicassoProvider中的其他方法都是默认实现。

我们知道ContentProvider是在Activity之前启动的，具体调用顺序为`Application#attachBaseContext --> ContentProvider#onCreate --> Application#onCreate --> Activity#onCreate`，所以我们在Activity中调用Picasso.get时，PicassoProvider.context早已初始化好了，我们就不必担心PicassoProvider.context赋值的问题。

我们知道，在正常使用ContentProvider时，还需要在AndroidManifest.xml中注册。我们找下注册的位置。

很明显，在使用Picasso时我们是不需要在AndroidManifest.xml中注册的，那么PicassoProvider就肯定是注册在picasso的aar文件中的AndroidManifest.xml中了。

这里我们在Project视图下，打开External Libraries目录，找到picasso依赖文件，如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190617-093736%402x.png)

比较疑惑的是，这里的Gradle依赖的是aar文件，我们应该是可以看到aar文件中的非java文件的，不过这里只显示了`classes.jar`包。

这样就只能找到我们的aar源文件查看了。右键我们Picasso的Gradle依赖，选择`Library Properties...`选项，在弹出的对话框中就可以看到aar的源文件了。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190617-094333%402x.png)

我们按图索骥，先找到这个jar文件的目录，目录如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190617-094639%402x.png)

虽然找到这个了，不过依然没什么用，我们想要的是aar文件。我们看下上面的文件夹：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190617-094656%402x.png)

这个就是我们要找的aar文件。

#### 查看aar文件的内容

我们先复制一份aar文件出来，然后将后缀改为`.zip`，接着解压该文件，解压后如下所示：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190617-095144%402x.png)

可以看到我们想找的AndroidManifest.xml，看下内容：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190617-095657%402x.png)

可以看到，PicassoProvider也注册到AndroidManifest.xml中了，至此Picasso中的context的获取原理搞清楚了。

#### 参考

[Application, Activity, ContentProvider启动顺序](https://blog.csdn.net/beyond702/article/details/49666809)


