---
layout: post
title:  dart中将方法当做参数传递时的注意事项
date:   2019-04-29 20:35:41 +0800
categories: Dart
tag: [Dart Functions]
---

* content
{:toc}



@[toc]


### dart中将方法当做参数传递时的注意事项

众所周知，Dart是一门面向对象的语言，比Java更纯粹，Dart中的方法也是对象，也有类型[Function](https://api.dartlang.org/stable/2.2.0/dart-core/Function-class.html)。这意味着方法可以被分配给对象，也可以当做参数传递给其他方法。

#### 方法当做参数传递给另一个方法

这里我们先定义一个返回值为void，入参为一个int的`printElement`方法，它的作用很简单，就是打印这个int值。

接着我们初始化一个数组，然后将`printElement`方法当做参数传递给我们的`list.forEach`方法，这样我们将list中的每个元素，依次交给`printElement`方法去打印。

```
void printElement(int element) {
  print(element);
}

main() {
  var list = [1, 2, 3];

  // Pass printElement as a parameter.
  list.forEach(printElement);
}
```

如果不太好理解的话，可以看下list.forEach方法源码，也就是Iterator#forEach()，如下所示。

```
  /**
   * Applies the function [f] to each element of this collection in iteration
   * order.
   */
  void forEach(void f(E element)) {
    for (E element in this) f(element);
  }
```

可以看出，我们的forEach方法接受的参数是一个方法，该方法的返回值void，一个入参f，入参类型这里用的是泛型。

方法内部调用了for循环，依次获取到Iterator（也就是我们的List）中的元素element，然后将element作为参数传递给f方法，同时调用这个f方法。这个f方法就是我们之前传入的`printElement`方法。

#### 重要细节

接下来我们重点说下几个细节。

1、第一个细节，就是`方法当做参数传递`的时候，`只需要传递方法名`即可，不需要带上方法的`括号`。具体请看我们的`printElement`方法是如何定义，以及是如何传递给`list.forEach`方法的。

2、方法作为参数的时候传递给其他方法的时候，不会立即执行。

方法当做参数传递的时候，方法名表示该方法的引用，这个引用当做参数传递的时候不会立即执行，只会再调用的时候执行。

3、入参方法在被实际调用时，会添加括号，当做正常的方法调用。

具体请看`printElement`方法当做参数传递给`list.forEach`方法的过程，以及`list.forEach`方法内部是如何调用入参方法f的。

#### 方法当做参数传递时，无括号和有括号的区别

这里之所以一再强调方法调用有没有括号，是因为我在这里吃过亏，另一方面对这些细节理解到位有助于我们加深对Dart语言的理解。

我们看个例子就明白了。

这里我们定义一个add方法，返回值为int，入参为两个int。

然后定义一个test方法，返回值为int，前两个入参都是int类型，第三个参数是Function类型。

然后在main方法中，将add方法作为参数传递给test方法，并打印test方法的执行结果。

```
int add(int a, int b) {
  return a + b;
}

int test(int a, int b, Function operation) {
  return operation(a, b);
}

main() {
  print(test(5, 2, add));
}
```

这里`add方法`当做参数传递时值传递了方法名`add`，没有括号也没有参数，一起跟预期一样。

接下来我们修改下main方法中的测试方法，我们第三个参数修改为`add(1,1)`，代码如下所示：

```
main() {
  print(test(5, 2, add(1,1)));
}
```

具体错误提示如下：

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190429-202545.png)

提示的意思是说，int类型的参数不能赋值给Function类型的参数，也就是我们的`add(1,1)`结果为`int`，跟test方法中定义的第三个入参的类型不匹配。

好了，我们应该已经看到方法名（不带括号）和方法执行语句（带括号）的区别了。

结论如下：
* 方法名（不带括号）作为参数传递给其他方法时，传递的是方法的引用，该方法不会立即执行。
* 方法执行语句（带括号）作为参数传递给其他方法时，该方法会立即执行，我们传递过去的是方法的返回值。

在java中，方法不是对象，所以我们不会遇到`将方法当做参数传递给其他方法`这种情况。我们经常使用的事直接将方法执行语句（也就是它们的返回值）作为参数直接传递。

最后，Flutter和Dart语言的很多思想和语法，都是参考或照搬的前端，如果学习或开发中遇到困惑，那么从前端查起应该会容易好多。

#### 参考

[Dart-Functions](https://www.dartlang.org/guides/language/language-tour#functions)

[javascript函数（二）--将函数作为参数传递](https://blog.csdn.net/xingxing513234072/article/details/7718430)

[In dart , What does passing a function or a class without "()" parenthesis does?](https://www.reddit.com/r/dartlang/comments/bemub7/in_dart_what_does_passing_a_function_or_a_class/)


[tear-off](https://www.dartlang.org/guides/language/effective-dart/usage#dont-create-a-lambda-when-a-tear-off-will-do)

Lint rules: [unnecessary_lambdas](https://dart-lang.github.io/linter/lints/unnecessary_lambdas.html)


