---
layout: post
title:  Java compiler error: constant string too long
date:   2019-01-16 18:08:07 +0800
categories: [Java]
tag: [String]
---

* content
{:toc}



### Java compiler error: constant string too long

最近项目中遇到解析图片的base64字符串需求，在测试时遇到了`error: constant string too long`这个错误，代码如下：
```
String base64 = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAZAAAAKoCAYAAABZ..."
```
注意，这里实际上的字符串长度为752378，这里为了节省篇幅，只能简写。
长度如何查看呢？将光标至于字符串末尾，然后查看Android Studio下方，如下图所示，冒号后面的为总大小。然后再减去多余字符即可。
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190116-173832.png)

```
752406 - （26 + 2） = 752378
```
运行代码之后，报错如下：
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190116-174245.png)

为了不干扰测试，将这段字符串写到资源文件strings.xml中，然后再获取即可规避这个错误，代码如下：
```
String base64 = getResources().getString(R.string.string_base64);
```
这样就不会报错了，继续测试即可。

#### 错误原理分析：
我们报的错是编译期错误，编译器提示我们String常量的长度太长。我们知道，通过双引号定义的字符串都会被放进常量池的，会不会是常量池对字符串大小做了限制？运行时会不会也有这个限制？

参考[Size of Initialisation string in java](https://stackoverflow.com/questions/8323082/size-of-initialisation-string-in-java)，Java中对String字面值的限制是65535。

基于此，我们先进行如下测试，
```
StringBuffer sb = new StringBuffer();
for (int i = 0; i < 65536; i++) {
sb.append("a");
}
String result = sb.toString();
System.out.println(result);
```
运行结果OK，不会报异常，说明运行时对字符串的长度限制不是65534。

接下来测试我们尝试着将双引号定义字符串长度设为65535附近的值，校验下Java中对String字面值的限制是否准确。

我们先设置字符串常量的长度为65535，会发现依然报这个错误，如下图：
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190116-180149.png)

接下来设置字符串常量的长度为65534，发现编译成功，如下图：
![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190116-180256.png)

在此我们可以得出结论，`Java compiler error: constant string too long`这个错误的出现，是因为字符串常量值的长度超过了65534，编译期检查没通过。运行时不存在这个限制，运行时的内存限制走的是堆内存，跟CPU分配内存相关。

这个错误如何规避呢？在Android中，将这个字符串写入资源文件，然后在运行期间获取即可规避这个限制。

参考：
[https://netbeans.org/bugzilla/show_bug.cgi?id=212752](https://netbeans.org/bugzilla/show_bug.cgi?id=212752)
[https://stackoverflow.com/questions/8323082/size-of-initialisation-string-in-java](https://stackoverflow.com/questions/8323082/size-of-initialisation-string-in-java)
[Java中String接受的最大字符串的长度是多少](https://blog.csdn.net/wolfking0608/article/details/78583944)

