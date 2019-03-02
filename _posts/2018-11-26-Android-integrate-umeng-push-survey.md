---
layout: post
title:  Android集成友盟集成推送方案调研
date:   2018-11-26 17:49:23 +0800
categories: Android
tag: [push]
---

* content
{:toc}



### Android集成友盟集成推送方案调研
鉴于项目apk瘦身的需求，经过调研，发现现有的推送集成方案可以优化。现有的推送方案是华为 + 小米 + 友盟推送，分别针对的是华为（包括荣耀）手机、小米手机、其他类型手机。这样做的好处是，华为、小米系列的手机都可以支持离线消息，推送消息送达率有保证。缺点是推送的SDK的包比较大，会浪费用户流量。

鉴于项目中已经集成了友盟统计相关sdk，这里决定对友盟的集成推送方案进行调研，如果可以满足现有需求，则完全可以替换现有推送方案。

#### U-push方案集成步骤
参考文档：[U-push集成文档](https://developer.umeng.com/docs/66632/detail/66744)

集成步骤主要包括普通集成和通道集成。

**1、普通集成。**

这里说的普通集成，是指在华为、小米手机上不支持离线消息的情况。集成完这个步骤之后，在小米、华为手机上是收不到推送消息的。这个跟我们的需求不符，需要进一步优化，具体请看第2步。

* 友盟官网上获取AppKey和Umeng Message Secret，配置进AndroidManifest.xml
* 导入PushSDK

```
//PushSDK必须依赖基础组件库，所以需要加入对应依赖
implementation 'com.umeng.sdk:common:1.5.3'
//PushSDK必须依赖utdid库，所以需要加入对应依赖
implementation 'com.umeng.sdk:utdid:1.1.5.3'
//PushSDK
implementation 'com.umeng.sdk:push:4.2.0'
```

* 初始化PushSDK

```
UMConfigure.init(context, UMConfigure.DEVICE_TYPE_PHONE, umengMessageSecret);
```

* 注册推送服务，注册成功之后可以获取到token。

```
PushAgent mPushAgent = PushAgent.getInstance(this);
//注册推送服务，每次调用register方法都会回调该接口
mPushAgent.register(new IUmengRegisterCallback() {

    @Override
    public void onSuccess(String deviceToken) {
        //注册成功会返回device token
    }

    @Override
    public void onFailure(String s, String s1) {

    }
});
```

* 自定义通知打开动作。这个动作在通知栏消息被点击时触发。

```
UmengNotificationClickHandler notificationClickHandler = new UmengNotificationClickHandler() {

    @Override
    public void dealWithCustomAction(Context context, UMessage msg) {
        Toast.makeText(context, msg.custom, Toast.LENGTH_LONG).show();
    }
};
mPushAgent.setNotificationClickHandler(notificationClickHandler);
```

* 混淆配置。

**2、小米、华为Push通道集成。**

华为、小米对后台进程做了诸多限制。若使用一键清理，应用的channel进程被清除，将接收不到推送。为了增加推送的送达率，可选择接入华为、小米托管弹窗功能，通知将由华为系统托管弹出。

主要步骤如下：
* 登录华为、小米开发平台，创建对应的应用，启用推送服务，获取相应的应用信息。
* 导入华为、小米Push通道SDK

```
//华为Push通道
implementation 'com.umeng.sdk:push-huawei:1.0.0'
//小米Push通道
implementation 'com.umeng.sdk:push-xiaomi:1.0.0'
```

* 华为、小米Push初始化

在Application类的onCreate方法中添加：

```
HuaWeiRegister.register(final Context context);
MiPushRegistar.register(final Context context, final String XIAOMI_ID, final String XIAOMI_KEY);
```

注意：
华为Push通道：
①仅在华为EMUI设备上生效。
②集成华为Push的版本暂不支持多包名。
③若使用华为Push通道，则app的targetSdkVersion必须设置为25或25以下，设置为26及以上，会导致EMUI 8.0设备无法弹出通知。

小米Push通道：
①仅在小米MIUI设备上生效。
②集成小米push的版本暂不支持多包名。

* 使用华为、小米弹窗功能。

通知将由华为、小米系统托管弹出，点击通知栏将跳转到指定的Activity。该Activity需继承自UmengNotifyClickActivity，同时实现父类的onMessage方法，对该方法的intent参数进一步解析即可，该方法异步调用，不阻塞主线程。示例如下：

```
public class MipushTestActivity extends UmengNotifyClickActivity {

    private static String TAG = MipushTestActivity.class.getName();

    @Override
    protected void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_mipush);
    }

    @Override
    public void onMessage(Intent intent) {
        super.onMessage(intent);  //此方法必须调用，否则无法统计打开数
        String body = intent.getStringExtra(AgooConstants.MESSAGE_BODY);
        Log.i(TAG, body);
    }
}
```

别忘了注册该Activity：

```
<activity
      android:name="com.umeng.message.example.MipushTestActivity"
      android:launchMode="singleTask"
      android:exported="true" />
```

在【友盟+】推送后台发送通知时，勾选若设备离线转为系统通道下发，并填写Activity的完整包路径（该Activity需继承自UmengNotifyClickActivity）。
![](http://dev.umeng.com/system/images/W1siZiIsIjIwMTgvMDEvMTEvMTlfMTNfMzdfNF9tZWl6dTUucG5nIl1d/meizu5.png)

**注意:**

华为：
对于EMUI 4.1及以下版本系统，若要使用华为弹窗功能，则需在华为设备上的【手机管家】App中，开启应用的“自启动权限”。
使用华为弹窗下发的通知，将只能被统计到通知的【打开数】，而该条通知的【收到数】、【忽略数】将无法被统计到。

小米：
使用小米系统通道下发的消息，将只能被统计到消息的【打开数】，而该条消息的【收到数】、【忽略数】将无法被统计到。
若要使用小米系统通道下发通知，则通知的标题（title）不允许全是空白字符且长度小于50，通知的内容（text）不允许全是空白字符且长度小于128（通知的标题和内容必填，一个中英文字符均计算为1）。


在调用API接口实现推送消息时，如果需要使用华为、小米弹窗，需添加：

```
"mipush":true
"mi_activity":"com.umeng.message.example.MipushTestActivity"        //此处请填写Activity完整包路径
```
API接口添加位置参考：
```
{
"appkey": "", 
"mi_activity": "com.umeng.message.example.MipushTestActivity"
"mipush": true,
"timestamp": 1473225266373,
"production_mode": "true",
"type": "unicast", 
"device_tokens": "", 
"payload":
    {"body": 
       {"text": "from pa36a", 
        "after_open": "go_app", 
        "ticker": "Hello World", 
        "title": "listcastpa43"
       }, 
     "display_type": "notification", 
    }
}
```

最后对友盟统计集成方案做下总结：
1、在集成U-push的基础上，还需要集成华为、小米Push通道。它们暂时均不支持多包名。

2、华为手机需要注意的问题：
①若使用华为Push通道，则app的targetSdkVersion必须设置为25或25以下，设置为26及以上，会导致EMUI 8.0设备无法弹出通知。
②对于EMUI 4.1及以下版本系统，若要使用华为弹窗功能，则需在华为设备上的【手机管家】App中，开启应用的“自启动权限”。

3、小米手机需要注意的问题：
①若要使用小米系统通道下发通知，则通知的标题（title）不允许全是空白字符且长度小于50，通知的内容（text）不允许全是空白字符且长度小于128（通知的标题和内容必填，一个中英文字符均计算为1）。

4、华为、小米通道的统计问题:
使用华为弹窗下发的通知，将只能被统计到通知的【打开数】，而该条通知的【收到数】、【忽略数】将无法被统计到。


#### bug解决：
1、utdid冲突：

```
Warning: Exception while processing task java.io.IOException: Can't write [/Users/xxx/ABC/app/build/intermediates/transforms/proguard/api_15_/release/0.jar] (Can't read [/Users/xxx/.gradle/caches/modules-2/files-2.1/com.umeng.sdk/utdid/1.1.5.3/989c3bb13060da1e3154bfe00236f76453a2725f/utdid-1.1.5.3.jar(;;;;;;**.class)] (Duplicate zip entry [utdid-1.1.5.3.jar:com/ta/utdid2/device/UTDevice.class]))
```

解决方式：注释掉这里的utdid依赖。

```
    //PushSDK必须依赖基础组件库，所以需要加入对应依赖
    implementation 'com.umeng.sdk:common:1.5.3'
    //PushSDK必须依赖utdid库，所以需要加入对应依赖
//    implementation 'com.umeng.sdk:utdid:1.1.5.3'
    //PushSDK
    implementation 'com.umeng.sdk:push:4.2.0'
```

#### 参考
[https://developer.umeng.com/docs/66632/detail/66744](https://developer.umeng.com/docs/66632/detail/66744)