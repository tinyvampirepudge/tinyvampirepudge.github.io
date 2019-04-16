---
layout: post
title:  dart中箭头表达式与js中箭头表达式对比
date:   2019-04-16 10:50:50 +0800
categories: Dart
tag: [Dart, 箭头表达式]
---

* content
{:toc}



### dart中箭头表达式与js中箭头表达式对比

#### 1、unexpected text ‘if’
```
  List<int> list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  list.forEach((num) => {
    if (num % 2 == 0) {
      
    }
  });
```

if这儿报错，报错提示是`unexpected text ‘if’`，如下图所示：


![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190416-100118%402x.png)

一提起箭头表达式，我们先想起的肯定是js，那我们看下js中箭头表达式的用法。

#### 2、js中的箭头表达式

我们看下js中箭头表达式的定义：[箭头函数表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions#Syntax).

基础语法如下：

```
(参数1, 参数2, …, 参数N) => { 函数声明 }

//相当于：(参数1, 参数2, …, 参数N) =>{ return 表达式; }
(参数1, 参数2, …, 参数N) => 表达式（单一）

// 当只有一个参数时，圆括号是可选的：
(单一参数) => {函数声明}
单一参数 => {函数声明}

// 没有参数的函数应该写成一对圆括号。
() => {函数声明}
```

在js中执行下面代码，不会报错。

```
var list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
list.forEach((num) => {
    if (num % 2 == 0) {
        console.log('num:' + num);
    }
});
```

输出如下：

```
num:2
num:4
num:6
num:8
num:10
```

这说明js中的箭头表达式的方法体中是支持if语句的。

#### 3、Dart中的箭头表达式

既然js中的箭头表达式中的方法体可以包含if语句，那么我们就在dart的文档中找关于箭头表达式的用法了。

我们看下dart官网的说明:[Functions](https://www.dartlang.org/guides/language/language-tour#functions)，具体如下：

```
isNoble(atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```

对于只包含`一个表达式`的方法，你可以使用简写语法：

```
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```

这里的`=> expr`语法是对`{ return expr; }`的简写，`=>`符号`有时`可以当做`箭头表达式`。

```
Note: 只有表达式——而不是声明——才能出现在 箭头(=>) 和 分号(;) 之间。比方说，你不能在这里使用if声明，但是你可以使用条件表达式。
```

好了，dart官网已经说的很清楚了，它里面的 `=>` 并不等同于我们所说的箭头表达式，是有田间限制的。方法体只包含一个表达式时，可以使用箭头表达式方法进行简写。


好了，既然dart中的箭头表达式只支持单行表达式，那么我们这里是不能使用它了，那么我们去掉箭头，使用匿名方法即可，如下所示：

```
  List<int> list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  list.forEach((num) {
    if (num % 2 == 0) {
      print('num:$num');
    }
  });
```

输出如下：

```
num:2
num:4
num:6
num:8
num:10
```


#### 4、结论

在使用Dart时，我们只需要记住一句话，`方法体只包含一个表达式时，可以使用箭头表达式方法进行简写。`这个跟语句加不加括号无关。

