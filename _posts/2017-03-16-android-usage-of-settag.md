---
layout: post
title:  view.setTag()的正确使用姿势
date:   2017-03-16 22:54:53 +0800
categories: Android
tag: [view.setTag]
---

* content
{:toc}



### view.setTag()的正确使用姿势
开发中，我们经常要进行数据的传递，会使用到view.setTag()和view.getTag()方法，主要用在view的点击事件中，可以让数据跟着view走，这种方法很方便。一般情况下给view设置一个tag就够用了，某些情况下我们需要给一个view设置多个tag，在需要的时候再分别取出来，这就需要用到view.setTag()的一个重载方法view.setTag(int key,final Object Tag)了。

在view.setTag(key)方法的注释里面有这么一句：

```
The specified key should be an id declared in the resources of the
* application to ensure it is unique …
```

```
@throws IllegalArgumentException If they specified key is not valid
```
意思是说key必须是个唯一的资源id，就会报错。

好了，注意事项说完了，接下来该使用了。
#### ①定义id，res/values/ids.xml文件下定义需要的id：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- 使用tag传递数据。 -->
    <item name="tag_txt_more_news" type="id"/>
    <item name="tag_txt_more_announce" type="id"/>
    <item name="tag_txt_more_report" type="id"/>
</resources>
```
#### ②settag，添加数据
```
txMoreNews.setTag(R.id.tag_txt_more_news, newsCount)
```
#### ③gettag获取数据并使用
```
int count = (int) v.getTag(R.id.tag_txt_more_news);
```
另外，view.setTag()/getTag()方法和view.setTab(key,value)/getTag(key)方法可以同时使用，他们的值不会冲突，view.setTag()添加的值是存在View对象的一个Object类型的成员变量里，而通过key添加的数据是存在一个SparseArray<Object>里面，他们的值不会发生冲突.

做下总结，希望可以帮到你。

