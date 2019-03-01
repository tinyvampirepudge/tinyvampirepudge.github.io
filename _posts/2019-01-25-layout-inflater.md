---
layout: post
title:  LayoutInflater总结
date:   2019-01-25 14:47:05 +0800
categories: Android
tag: [LayoutInflater]
---

* content
{:toc}



### LayoutInflater总结

参考：
[三个案例带你看懂LayoutInflater中inflate方法两个参数和三个参数的区别](https://blog.csdn.net/u012702547/article/details/52628453)

```
LayoutInflater#inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot)
```
总结：
1、root不为空，attachToRoot为true。
此时布局会直接添加到root上，并且root布局中设置的宽高参数会生效。
此时如果再执行root.addView(view)的话会报错。

2、root不为空，attachToRoot为false。
此时生成的view的宽高参数有效，但是view不会直接添加进root。

3、root为空，attachToRoot的值随意。
当root为null时，不论attachToRoot为true还是为false，显示效果都是一样的。
此时layout_linearlayout.xml 中的根布局设置的宽高属性会失效。

代码测试如下：
1、activity布局：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:id="@+id/ll"
android:layout_height="match_parent"
android:orientation="vertical"
tools:context=".test.view.LayoutInflaterTestActivity">

</LinearLayout>
```

2、layout_linearlayout.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="200dp"
android:layout_height="200dp"
android:background="@color/colorPrimary"
android:gravity="center"
android:orientation="vertical">

<Button
android:layout_width="wrap_content"
android:layout_height="wrap_content" />

</LinearLayout>
```

3、Java代码：
```
package com.tiny.demo.firstlinecode.test.view;

import android.content.Context;
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.LinearLayout;

import com.tiny.demo.firstlinecode.R;

/**
* @Description: LayoutInflater#inflate()原理解析
* https://blog.csdn.net/u012702547/article/details/52628453
* @Author wangjianzhou@qding.me
* @Date 2019/1/25 11:23 AM
* @Version
*/
public class LayoutInflaterTestActivity extends AppCompatActivity {
public static void actionStart(Context context) {
Intent starter = new Intent(context, LayoutInflaterTestActivity.class);
context.startActivity(starter);
}

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_layout_inflater_test);

// 1、root不为空，attachToRoot为true
// 下面这段代码，会直接让布局显示出来。
LinearLayout ll = findViewById(R.id.ll);
LayoutInflater inflater = LayoutInflater.from(this);
View view = inflater.inflate(R.layout.layout_linearlayout, ll, true);

//这里会报错，原因就是因为当第三个参数为true时，会自动将第一个参数所指定的View添加到第二个参数所指定的View中。
// java.lang.IllegalStateException: The specified child already has a parent. You must call removeView() on the child's parent first.
ll.addView(view);

// 2、root不为空，attachToRoot为false
// 如果我想让linearlayout的根节点有效，又不想让其处于某一个容器中，那我就可以设置root不为null，而attachToRoot为false。
LinearLayout ll = findViewById(R.id.ll);
LayoutInflater inflater = LayoutInflater.from(this);
View view = inflater.inflate(R.layout.layout_linearlayout, ll, false);
ll.addView(view);

// 3、root为null，attachToRoot的值随意
// 当root为null时，不论attachToRoot为true还是为false，显示效果都是一样的。
// 此时layout_linearlayout.xml 中的根布局设置的宽高属性会失效。
LinearLayout ll = findViewById(R.id.ll);
LayoutInflater inflater = LayoutInflater.from(this);
View view = inflater.inflate(R.layout.layout_linearlayout, null, true);
ll.addView(view);
}
}

```


参考：
[三个案例带你看懂LayoutInflater中inflate方法两个参数和三个参数的区别](https://blog.csdn.net/u012702547/article/details/52628453)


