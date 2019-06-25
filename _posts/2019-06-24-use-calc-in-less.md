---
layout: post
title:  less中使用calc计算高度注意事项
date:   2019-06-24 18:59:18 +0800
categories: less
tag: [calc, less]
---

* content
{:toc}



### less中使用calc计算高度注意事项

我们知道，在css中我们可以使用`100vh`表示屏幕高度，我们还可以通过`calc(expression)`来动态计算宽高，于是便有了如下代码：

```
height: calc(100vh - 50px);
```

然而事与愿违，我们得到的结果却是这样的：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WechatIMG91.png)

我们得到的是50vh,相当于屏幕高度的一半。

google一波，我们修改代码如下：

```
height: calc(~"100vh - 50px");
```

此时看效果，已然正常。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190624-185653.png)

#### 参考
[https://stackoverflow.com/questions/42548630/css3-calc-minus-vh-with-pixel/42556033](https://stackoverflow.com/questions/42548630/css3-calc-minus-vh-with-pixel/42556033)

