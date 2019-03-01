---
layout: post
title:  Numbers——Dart
date:   2019-02-22 15:44:04 +0800
categories: Dart
tag: [Numbers]
---

* content
{:toc}



### Numbers——Dart
Dart中的数字类型有两种，int和double。
#### int
int 数值的范围不再是64位，取决于平台。
在Dart虚拟机上，范围是-2^63 to 2^63 - 1.
在编译成JavaScript上时使用的是[JavaScript numbers](https://stackoverflow.com/questions/2802957/number-of-bits-in-javascript-numbers/2803010#2803010)，范围是-2^53 to 2^53 - 1。

#### double
双精度浮点型数字类型，在IEEE 754 standard中指定。

int 和 double 都是num的子类。[num](https://api.dartlang.org/stable/dart-core/num-class.html)类型包含一些基本操作，加减乘除，还有 abs(), ceil(), and floor()等方法。

int 类中还定义了按位操作符 shift(<<, >>), AND (&), and OR (|)。
```
assert((3 << 1) == 6); // 0011 << 1 == 0110
assert((3 >> 1) == 1); // 0011 >> 1 == 0001
assert((3 | 4) == 7); // 0011 | 0100 == 0111
```

更多的算术操作符，可以查看 [dart:math](https://api.dartlang.org/stable/2.1.1/dart-math/dart-math-library.html) 库。

int是没有小数点的num，如下：
```
var x = 1;
var hex = 0xDEADBEEF;
```

如果数字包含小数点，那就是double类型的。如下所示：
```
var y = 1.1;
var exponents = 1.42e5;
```

在Dart2.1版本中，必要时，int 字面值可以自动转化为double.
```
double z = 1; // Equivalent to double z = 1.0.
```
需要注意的是：在Dart2.1版本以前，int字面值不能当做double使用，会报错。

下面展示了如何将String转化为number类型，反之亦然：
```
// String -> int
var one = int.parse('1');
assert(one == 1);

// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);

// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');

// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
assert(piAsString == '3.14');
```

numbers 字面值是编译时常量。许多数学表达式也是编译时常量，只需要确保他们的操作数也是numbers类型的编译时常量即可。


参考:
[https://www.dartlang.org/guides/language/language-tour#numbers](https://www.dartlang.org/guides/language/language-tour#numbers)

