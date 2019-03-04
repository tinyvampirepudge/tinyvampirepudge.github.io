---
layout: post
title:  kotlin中书写点击事件的几种方式
date:   2018-08-24 11:33:03 +0800
categories: kotlin
tag: [kotlin点击事件]
---

* content
{:toc}



### kotlin中书写点击事件的几种方式
布局文件：

```javascript
<Button
                android:id="@+id/btnTestThread"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="10dp"
                android:text="测试1"
                android:textAllCaps="false" />
```

1、View#setOnClickListener()
常规写法：

```javascript
val btn1:Button = findViewById(R.id.btnTestThread)
        btn1.setOnClickListener(object : View.OnClickListener {
            override fun onClick(v: View?) {
                println("王蛋蛋的father")
            }
        })
```

使用lambda进行简化：

```javascript
btn1.setOnClickListener { println("王蛋蛋的father") }
```

2、Activity实现View.OnClickListener接口的onClick方法

```javascript
class CoroutineActivity : AppCompatActivity(), View.OnClickListener {
    override fun onClick(v: View?) {
        when (v?.id) {
            R.id.btnTestThread -> {
                println("王蛋蛋的father")
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_coroutine)
        val btn: Button = findViewById(R.id.btnTestThread)
        btn.setOnClickListener(this)
    }
}
```

3、在xml中指定点击事件的方法名

```javascript
<Button
                android:id="@+id/btnTestThread"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="10dp"
                android:onClick="test1"
                android:text="测试1"
                android:textAllCaps="false" />
				
				
	fun test1(V: View) {
        println("王蛋蛋的father")
    }
```

4、使用kotlin的android拓展库
[传送门](https://www.kotlincn.net/docs/tutorials/android-plugin.html)

```javascript
<Button
                android:id="@+id/btnTestThread"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="10dp"
                android:text="测试1"
                android:textAllCaps="false" />
				
class CoroutineActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_coroutine)
        btnTestThread.setOnClickListener {
            println("王蛋蛋的father")
        }
    }
}
```


