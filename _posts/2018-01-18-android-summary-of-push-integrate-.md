---
layout: post
title:  Android推送集成方案总结
date:   2018-01-18 14:02:00 +0800
categories: Android
tag: [push]
---

* content
{:toc}



### Android推送集成方案总结
刚做完推送集成方案，记录下坑。

这里记录的特性和使用时针对写blog时采用的sdk的，具体使用流程和限制还请参考官方给出的sdk.
#### 1、推送规则
小米手机用小米推送；

华为手机用华为推送；

其他手机用友盟推送。

#### 2、总体流程

![这里写图片描述](http://img.blog.csdn.net/20180118135016885?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 3、小米推送：
①sdk地址：[https://dev.mi.com/console/doc/detail?pId=100#_1](https://dev.mi.com/console/doc/detail?pId=100#_1)

②常用方法：注册，设置别名，

③别名可设置多个，无效别名需要reset掉。

④消息接收是用的是广播接收者，继承自PushMessageReceiver即可。内部方法可获得注册结果、设置别名结果、消息到达时机、消息点击时机等。
 
⑤小米注册是在application#onCreate()方法中，需要在主进程中。

⑥支持设置通知声音

#### 4、友盟推送：
①sdk地址：http://dev.umeng.com/push/android/integration#3_4

②推荐使用推送基础包，不要使用友盟的集成推送，坑的一笔。

③友盟的最新文档有毛病，对不上号，必要时可以翻出旧的文档看看。

④友盟注册是在application#onCreate()方法中，不需要判断进程，否则出错。

⑤设置别名之后，使用别名和token都可以给设备发送消息。

⑥友盟的通知到达的时机，需要继承UMengMessageHandler类，里面的getNotification()方法

```
mPushAgent.setMessageHandler(new UMengPushHandler());

public class UMengPushHandler extends UmengMessageHandler {

    @Override
    public Notification getNotification(Context context, UMessage uMessage) {
        LogUtils.e("getNotification");
        // TODO: 2018/1/10 友盟推送的消息到达时会走这儿
        return super.getNotification(context, uMessage);
    }
}
```

⑦友盟的通知消息被点击

```
public class UmengPushClickHandler extends UmengNotificationClickHandler {
    @Override
    public void dealWithCustomAction(Context context, UMessage msg) {
        LogUtils.e("dealWithCustomMessage");
        try {
            LogUtils.e("message=" + msg);    //消息体
            LogUtils.e("custom=" + msg.custom);    //自定义消息的内容
            LogUtils.e("title=" + msg.title);    //通知标题
            LogUtils.e("text=" + msg.text);    //通知内容
            // code  to handle message here
            // ...

            // 对完全自定义消息的处理方式，点击或者忽略
            boolean isClickOrDismissed = true;
            if (isClickOrDismissed) {
                //完全自定义消息的点击统计
                UTrack.getInstance(context).trackMsgClick(msg);
            } else {
                //完全自定义消息的忽略统计
                UTrack.getInstance(context).trackMsgDismissed(msg);
            }

            // 使用完全自定义消息来开启应用服务进程的示例代码
            // 首先需要设置完全自定义消息处理方式
            String msgStr = msg.custom;
            if (!TextUtils.isEmpty(msgStr)) {
                Map<String, String> map = new HashMap<>();
                JSONObject jsonObject = new JSONObject(msgStr);
                Iterator<String> it = jsonObject.keys();
                while (it.hasNext()) {
                    String key = it.next();
                    String val = (String) jsonObject.get(key);
                    map.put(key, val);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

mPushAgent.setNotificationClickHandler(new UmengPushClickHandler());
```

⑧支持设置通知声音

#### 5、华为推送：
①sdk地址:

[http://developer.huawei.com/consumer/cn/service/hms/catalog/huaweipush_agent.html?page=hmssdk_huaweipush_sdkdownload_agent](http://developer.huawei.com/consumer/cn/service/hms/catalog/huaweipush_agent.html?page=hmssdk_huaweipush_sdkdownload_agent)

②华为的推送服务进行了升级，初始化应该在启动页中，并且要先让手机支持hms服务。

③华为注册成功之后推送服务器会返回token，这个token需要我自己上传给服务端的哥们。

```
public class HuaWeiPushReceiver extends PushReceiver {

    /**
     * token申请成功之后
     *
     * @param context
     * @param token
     * @param extras
     */
    @Override
    public void onToken(Context context, String token, Bundle extras) {
        String belongId = extras.getString("belongId");
        LogUtils.e("onToken token: " + token);
        //设备上报
    }

}
```

④华为不可设置别名，暂时不支持tag。

⑤华为推送通知的自定义操作比较麻烦，不能通过广播接受者处理，需要用隐式Intent调起一个Activity来获取服务端传递过来的数据。

自定义后续动作

![官方示例](http://img.blog.csdn.net/20180118135702540?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
⑥不支持设置通知声音

