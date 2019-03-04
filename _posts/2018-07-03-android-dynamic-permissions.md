---
layout: post
title:  Android动态权限（兼容6.0以下和魅族手机方案）
date:   2018-07-03 18:19:15 +0800
categories: Android
tag: [Android动态权限]
---

* content
{:toc}


### Android动态权限（兼容6.0以下和魅族手机方案）
这里以照相机权限为例说明问题。实际开发过程中遇到了不少的坑。

1、一般情况下，6.0以上的手机：
①判断是否具有某项权限：
[ContextCompat.checkSelfPermission()](https://developer.android.com/reference/android/support/v4/content/ContextCompat?hl=zh-cn#checkSelfPermission%28android.content.Context,%20java.lang.String%29)
下面这段代码展示了判断手机是否具有相机权限：

```
boolean hasCameraPermission = hasPermission(Manifest.permission.CAMERA);

@TargetApi(Build.VERSION_CODES.M)
public boolean hasPermission(String permission) {
    return ContextCompat.checkSelfPermission(this, permission) == PackageManager.PERMISSION_GRANTED;
}
```

2、在需要使用相机权限的页面中，我们需要判断应用是否具有相机权限，如果没有就进行申请。

以下代码可以检查应用是否具备调用照相机的权限，并根据需要请求该权限：

```
// Here, thisActivity is the current activity

if (ContextCompat.checkSelfPermission(thisActivity,Manifest.permission.CAMERA)
        != PackageManager.PERMISSION_GRANTED) {

    //①如果用户之前请求过此权限但用户拒绝了请求，并且用户没有勾选Don’t ask again选项，此方法会返回true；
    //②如果用户之前请求过此权限但用户拒绝了请求，并且用户勾选了Don’t ask again选项，此方法会返回false；
    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
            Manifest.permission.CAMERA)) {

        // 这里给用户一个提示对话框，让用户选择是否再次请求权限
    } else {
        
        // 逻辑走到这里说明用户已经拒绝了相机权限，并且都选了Don’t ask again选项，就算在这里直接请求相机权限，用户界面也不会弹出权限对话框，而是直接返回权限拒绝的结果。针对这种情况，我们可以考虑让用户进入设置页面，手动开启相机权限。
        ActivityCompat.requestPermissions(thisActivity,
                new String[]{Manifest.permission.CAMERA},
                MY_PERMISSIONS_REQUEST_CAMERA);
    }

}
```

上述代码用流程图表示，如下所示：
![图1](https://img-blog.csdn.net/20180703181548548?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3、请求权限的对话框：
当应用请求权限之后：
①如果用户之前没有做“拒绝并勾选’Don’t ask again’”的操作，那么系统会向用户展示一个对话框，如下所示：
Ps：第一次请求时是没有Don’t ask again选择框的。

<img src="https://img-blog.csdn.net/20180703181617452?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" height="600" alt="图2"/>

该对话框是个阻塞操作，不论用户点击了拒绝还是确认按钮，系统都会有对应的结果返回。
②如果用户之前做了“拒绝并勾选’Don’t ask again’”的操作，那么该对话框就不会展示，而是直接返回权限拒绝的结果。

4、请求权限的结果处理：
权限请求的回调方法是Activity#onRequestPermissionResult方法，如下所示。

```
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
        @NonNull int[] grantResults) {
    /* callback - no nothing */
}
```

这里我们需要重写该方法，获取权限请求的结果。
官方demo如下所示：

```
@Override

public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {

    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_CAMERA: {

            // 如果请求取消，grantResults会为空，所以这里需要做非空判断。
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        
                // 权限被授予了。恭喜！

            } else {

                // 权限被拒绝了。这里需要提示用户，并提供去设置页面开启的选项。

            }

            return;

        }

        //对其他权限进行处理，如果有的话。

    }

}
```
5、6.0以上手机（不包含魅族）完整版的解决方案：如下图所示：
![图3](https://img-blog.csdn.net/2018070318171910?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

6、说完正常的之后，就说说不正常的，在实际开发中发现，6.0以下的某些手机（华为、OPPO等）和魅族手机（包括6.0以上）的表现有些与众不同，在需要使用相机权限之前，系统会额外弹出一个“禁止、允许”对话框，用户点击了允许按钮的话还好，如果点击了禁止按钮，那么上面对6.0手机的处理会对魅族高版本手机失效，很蛋疼。经过Google，发现了一篇帖子是可以适配这种情况的，具体链接在文章末尾的参考3中。
文章中介绍了一种可行的处理方法，使用抓取Camera.open()异常的方法来解决，比较暴力。

```
/**
 * 判断摄像头是否可用
 * 主要针对6.0 之前的版本，现在主要是依靠try...catch... 报错信息，感觉不太好，
 * 以后有更好的方法的话可适当替换
 *
 * https://blog.csdn.net/jm_beizi/article/details/51728495
 *
 * @return
 */
public static boolean isCameraCanUse() {
    boolean canUse = true;
    Camera mCamera = null;
    try {
        mCamera = Camera.open();
        // setParameters 是针对魅族MX5 做的。MX5 通过Camera.open() 拿到的Camera
        // 对象不为null
        Camera.Parameters mParameters = mCamera.getParameters();
        mCamera.setParameters(mParameters);
    } catch (Exception e) {
        canUse = false;
    }
    if (mCamera != null) {
        mCamera.release();
    }
    return canUse;
}
```

7、最终方案：
![图4](https://img-blog.csdn.net/20180703181757437?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

参考：

1、[https://developer.android.com/training/permissions/requesting?hl=zh-cn](https://developer.android.com/training/permissions/requesting?hl=zh-cn)

2、[https://github.com/android-cn/android-discuss/issues/174](https://github.com/android-cn/android-discuss/issues/174)

3、[https://blog.csdn.net/jm_beizi/article/details/51728495](https://blog.csdn.net/jm_beizi/article/details/51728495)

