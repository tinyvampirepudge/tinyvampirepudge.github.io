---
layout: post
title:  TextView文本尾部添加标签，支持自动换行
date:   2019-09-02 16:16:11 +0800
categories: SpannableStringBuilder
tag: [SpannableStringBuilder]
---

* content
{:toc}



### TextView文本尾部添加标签，支持自动换行

#### 需求

开发过程中我们经常会遇到文字尾部添加标签的需求，看是很简单，其实蛮难做的。比如我们的设计稿如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV2VjaGF0SU1HMjguanBlZw?x-oss-process=image/format,png)

打眼一看，一个水平方向线性布局就解决了，内部写两个TextView就行。

```
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="20dp">

    <TextView
        android:id="@+id/tv3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="天行健，君子以自强不息；"
        android:textColor="#000"
        android:textSize="15sp" />

    <TextView
        android:id="@+id/tv4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@drawable/room_member_role_bg"
        android:paddingLeft="6dp"
        android:paddingRight="6dp"
        android:text="王蛋蛋的芭比"
        android:textColor="#000"
        android:textSize="15sp" />

</LinearLayout>
```

emmmmm，但是，要是文字是多行的呢，我们的标签还能顺利的放在后面么？

如果第一行文本过长，我们就会得到下面的结果：
尾部标签显示不全：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV2VjaGF0SU1HMzAuanBlZw?x-oss-process=image/format,png)

或者如下，尾部标签直接不显示。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV2VjaGF0SU1HMzEuanBlZw?x-oss-process=image/format,png)

如果文字最后一行的剩余空间放不下我们的尾部标签，比较通用的做法是让标签换行。

我们想要的是这样的：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV2VjaGF0SU1HMzIuanBlZw?x-oss-process=image/format,png)

接下来我们一步步实现这个需求。

#### 使用SpannableStringBuilder + ImageSpan实现

使用过SpannableStringBuilder的同学都惊叹于它的强大，通过`SpannableStringBuilder#setSpan()`我们可以给TextView的文本设置独特的样式，比如加粗、斜体、字体大小、字体颜色、背景颜色、图片、删除线、下划线、点击事件等等。

我们这里也使用`SpannableStringBuilder + 特定Span`的方式来实现。

我们的尾部标签实质上是`给特定文本添加一个drawable背景`，经过调研发现，虽然我们可以通过`ReplacementSpan`等方式给文字绘制drawable背景，但是背景上方的文字却不显示，这个方案就暂时夭折了。

回过头来，我们发现`SpannableStringBuilder + ImageSpan`可以实现将图片自动换行，并且如果剩余空间不足时图片会自动换行，如下所示：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV2VjaGF0SU1HMzMuanBlZw?x-oss-process=image/format,png)

从第二个TextView可以看出，当前行的剩余空间不够放置`ImageSpan`图片时，会自动换行。这样就解决了我们的自动换行的问题。

我们接下来看下ImageSpan的构造方法，会发现它不仅支持Drawable，还支持Bitmap对象，具体如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV1gyMDE5MDkwMi0xNTM3NDIlNDAyeC5wbmc?x-oss-process=image/format,png)

我们还知道，View有个`getDrawingCache()`方法，它可以生成当前View的Bitmap缓存。

此时我们的实现思路可以串起来了，具体如下图：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vV1gyMDE5MDkwMi0xNTUxMDMlNDAyeC5wbmc?x-oss-process=image/format,png)

#### 代码实现

具体分为四步：
①创建TextView对象，设置drawable背景，设置字体样式，设置间距，设置文本等
②将View生成Bitmap对象
③根据Bitmap对象生成ImageSpan对象
④将ImageSpan对象设置到SpannableStringBuilder的对应位置

具体代码如下：注释写的很清楚，这里就不再赘述。

```
private void addTagToTextView(TextView target, String title, String tag) {
    if (TextUtils.isEmpty(title)) {
        title = "";
    }

    String content = title + tag;


    /**
     * 创建TextView对象，设置drawable背景，设置字体样式，设置间距，设置文本等
     * 这里我们为了给TextView设置margin，给其添加了一个父容器LinearLayout。不过他俩都只是new出来的，不会添加进任何布局
     */
    LinearLayout layout = new LinearLayout(this);
    TextView textView = new TextView(this);
    textView.setText(tag);
    textView.setBackground(getResources().getDrawable(R.drawable.room_member_role_bg));
    textView.setTextSize(12);
    textView.setTextColor(Color.parseColor("#FDA413"));
    textView.setIncludeFontPadding(false);
    textView.setPadding(ScreenUtils.dip2px(this, 6), 0,
            ScreenUtils.dip2px(this, 6), 0);
    textView.setHeight(ScreenUtils.dip2px(this, 17));
    textView.setGravity(Gravity.CENTER_VERTICAL);

    LinearLayout.LayoutParams layoutParams = new LinearLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
    // 设置左间距
    layoutParams.leftMargin = ScreenUtils.dip2px(this, 6);
    // 设置下间距，简单解决ImageSpan和文本竖直方向对齐的问题
    layoutParams.bottomMargin = ScreenUtils.dip2px(this, 3);
    layout.addView(textView, layoutParams);

    /**
     * 第二步，测量，绘制layout，生成对应的bitmap对象
     */
    layout.setDrawingCacheEnabled(true);
    layout.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED));
    // 给上方设置的margin留出空间
    layout.layout(0, 0, textView.getMeasuredWidth() + ScreenUtils.dip2px(this, (6 + 3)), textView.getMeasuredHeight());
    // 获取bitmap对象
    Bitmap bitmap = Bitmap.createBitmap(layout.getDrawingCache());
    //千万别忘最后一步
    layout.destroyDrawingCache();

    /**
     * 第三步，通过bitmap生成我们需要的ImageSpan对象
     */
    ImageSpan imageSpan = new ImageSpan(this, bitmap);


    /**
     * 第四步将ImageSpan对象设置到SpannableStringBuilder的对应位置
     */
    SpannableStringBuilder ssb = new SpannableStringBuilder(content);
    //将尾部tag字符用ImageSpan替换
    ssb.setSpan(imageSpan, title.length(), content.length(), Spanned.SPAN_EXCLUSIVE_INCLUSIVE);
    target.setText(ssb);
}
```

项目地址：[Android_Base_Demo](https://github.com/tinyvampirepudge/Android_Base_Demo)


具体代码请看：[SpannableStringBuilderActivity](https://github.com/tinyvampirepudge/Android_Base_Demo/blob/master/app/src/main/java/com/tiny/demo/firstlinecode/uicomponents/textview/SpannableStringBuilderActivity.java)

页面入口展示：

![页面入口展示](https://imgconvert.csdnimg.cn/aHR0cHM6Ly90aW55dG9uZ3RvbmctMTI1NTY4ODQ4Mi5jb3MuYXAtYmVpamluZy5teXFjbG91ZC5jb20vU2VwLTAyLTIwMTklMjAxNi0wOC0yNy5naWY)

当然了，实现这种需求的方式有很多种，这里的这种方式比较取巧。大家如果有好的实现方式，还望不吝赐教，比心。

#### 参考

[Textview转化成Bitmap对象](https://blog.csdn.net/qq_17422503/article/details/50346445)

