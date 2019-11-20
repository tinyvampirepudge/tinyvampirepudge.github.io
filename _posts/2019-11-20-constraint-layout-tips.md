---
layout: post
title:  ConstraintLayout实战小技巧—实现布局跟随效果
date:   2019-11-20 12:16:00 +0800
categories: Android
tag: [ConstraintLayout]
---

* content
{:toc}



### ConstraintLayout实战小技巧—实现布局跟随效果

#### 需求

有时UI小姐姐可能想要这样的效果，左侧的标题长度不定，标题后面跟着一个标签，根据标题长度不同，可以有以下几种样式：

①标题超过一行

②标题无内容

③标题不足一行

具体效果如下图

![](https://user-gold-cdn.xitu.io/2019/11/20/16e8701a6e942cf5?w=830&h=872&f=png&s=234741)

#### 实现方式

效果有了，接下来是如何实现，不使用ConstraintLayout的前提下，我们使用常规的RelativeLayout、LinearLayout等布局去做，做到最后会发现不好做或者做不出来，真心难过。

幸好我们有ConstraintLayout(真香)。

怎么写呢？核心就是，先将元素串成一个`chain`，链的样式选择`packed`，这还不够，同时在给第一个元素设置下面两个属性：

```
app:layout_constrainedWidth="true"
app:layout_constraintHorizontal_bias="0"
```

大功告成，具体代码如下：

```
<android.support.constraint.ConstraintLayout
        android:id="@+id/cl"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_marginTop="10dp"
        android:background="#eee"
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        app:layout_constraintTop_toBottomOf="@id/tv_desc">

        <TextView
            android:id="@+id/tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:ellipsize="end"
            android:gravity="center"
            android:maxLines="1"
            android:text="我是文本我是文本我是文本我是文本我是文本我是文本我是文本我是文本我是文本"
            android:textColor="#FDA413"
            android:textSize="12sp"
            app:layout_constrainedWidth="true"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintHorizontal_bias="0"
            app:layout_constraintHorizontal_chainStyle="packed"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toLeftOf="@id/iv"
            app:layout_constraintTop_toTopOf="parent" />

        <ImageView
            android:id="@+id/iv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:src="@drawable/icon_birthday_cake"
            android:visibility="visible"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toRightOf="@id/tv"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            tools:visibility="visible" />

    </android.support.constraint.ConstraintLayout>
```



