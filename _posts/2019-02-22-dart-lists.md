---
layout: post
title:  Lists——Dart
date:   2019-02-22 16:59:04 +0800
categories: Dart
tag: [Lists]
---

* content
{:toc}



### Lists——Dart
在Dart中，数组是 [List](https://api.dartlang.org/stable/2.1.1/dart-core/List-class.html) 对象，所以很多人称呼它为lists。

Dart中的list字面值类似js中的数组字面值。
```
var list = [1, 2, 3];
assert(list is List<int>);// true
```
note:Dart分析器会推断出list的类型为`List<int>`，如果你此时尝试想list中添加一个非int类型的对象，则分析器或运行时会跑出一个错误。更多信息请阅读[type inference](https://www.dartlang.org/guides/language/sound-dart#type-inference).

Dart中的List跟js中List很相似，如下所示：

```
var list = [1, 2, 3];
assert(list.length == 3);
assert(list[1] == 2);

list[1] = 1;
assert(list[1] == 1);
```

在list字面值前面添加 const 关键字，就可以定义一个编译时常量的数组。
```
var constantList = const [1, 2, 3];
// constantList 指向的是一个常量，我们不能给它添加元素（不能修改它）
constantList[1] = 1;       // error
// constantList 本身不是一个常量，所以它可以指向另一个对象
constantList = [4, 5, 6];     // it's fine
```


参考：
[https://www.dartlang.org/guides/language/language-tour#lists](https://www.dartlang.org/guides/language/language-tour#lists)



