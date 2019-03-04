---
layout: post
title:  成员变量初始化的问题
date:   2017-03-16 22:46:16 +0800
categories: Java
tag: [成员变量初始化]
---

* content
{:toc}



###  成员变量初始化的问题
#### java编程思想，第五章，5.7.2，静态数据的初始化，p96：
静态初始化只有在必要时刻才会进行。如果不创建Table对象，也不引用Table.b1或Table.b2，那么静态的Bowl b1和b2就永远不会被创建。只有第一个Table对象被创建（或第一次访问静态数据）的时候，它们才会被初始化。此后，静态对象不会再次被初始化。

针对上方标红的这段话，可能看着很普通，但是却让我踩了一个坑，我把这个坑记录下来，希望对大家有帮助。
     

实际工作中，我们会有这个需求，对url实施统一管理，一般分为生产环境和测试环境，我们默认都会设置一个开关，用来控制线上线下的URL切换，众所周知，这个变量我们通常用的是BuildConfig.DEBUG，它可以根据你的debug或者release环境来自动切换true或false。对于一般的需求我们是可以满足的。

代码如下：

```
public class RequestURL {
    public static String URL_ROOT = BuildConfig.DEBUG ? "http://api.abc.com" : "http://api1.abc.com";

    public static final String URL_FEEDBACK = URL_ROOT + "/def/feedback";//意见反馈
}

```

最下方的url_feedback是业务里面使用的url，url_root代表的是请求的base_url。相信这种写法可以满足大多数人的需求了。可是有时候我们需要使用线下环境的release包，需要线上环境的debug包等等，还要频繁的在它们之间进行切换，该怎么办呢？可能有人会说，我每次手动修改BuildConfiig.DEBUG位置的值，我不怕麻烦。。。好了，接下来跟大家展示下我当时的做法，思路很耿直，定义一个boolean类型的开关，默认值为true，然后给它一个set方法用来控制开关。代码如下：

```
public class RequestURL {
    private static boolean isOnline = true;

    public static void setOnline(boolean online) {
        isOnline = online;
    }

    public static String URL_ROOT = isOnline ? "http://api.abc.com" : "http://api1.abc.com";

    public static final String URL_FEEDBACK = URL_ROOT + "/def/feedback";//意见反馈

}
```


是不是很科学？是不是看着没毛病？no no no，当我调用了RequestUrl.setOnLine(false)方法之后，发现url还是线上的，切换不到线下。这让我很郁闷，debug之后发现URL__ROOT的初始化只会第一次调用，之后就不会被调用了，我们"="后边的三元表达式也就只能是判断个寂寞了。后来回家翻书，就找到了文章最开始的那段话了。

同理，成员变量也只会被初始化一次。

记录下来，希望对大家有用。

