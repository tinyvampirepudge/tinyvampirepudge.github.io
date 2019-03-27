---
layout: post
title:  Dart中Class、mixin、interface三者间关系及异同
date:   2019-03-27 18:26:08 +0800
categories: Dart
tag: [mixin, Dart]
---

* content
{:toc}



### Dart中Class、mixin、interface三者间关系及异同

#### Class
Dart中一切皆为对象，而每个对象都是一个类的实例，所有的类都继承于Object。

除了普通的构造方法，Dart中的Class还提供了不同用途的构造方法，比如命名构造方法、重定向构造方法、常量构造方法、工厂构造方法，还有初始化参数列表等。

##### 抽象类
抽象类使用abstract关键字定义，是不能被实例化的，通常用来定义接口以及部分实现。
但与其他语言不太一样的地方是，抽象方法也可以定义在非抽象类中。

#### Interface:
每个类的内部都隐式的定义了一个接口，这个接口包含类的成员的所有实例，以及类实现的所有接口。
如果你想让A类支持B类所有的API，并且不通过继承B类来实现，那么A类应该实现B接口。
一个类可以实现一个或多个接口，通过implements关键字。

#### mixin
Dart语言集合了现代编程语言的众多优点，Mixin继承机制也是其一。具体的读法应该叫做 mix in，翻译下就是混入。

`mixins`是一种实现多重继承的方式，通过它可以给现有的类添加特性。

在`with`关键字后面可以跟随一个或多个`mixin`的名称。如下所示：

```
class Musician extends Performer with Musical {
  // ···
}

class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

想要实现一个mixin，你可以创建一个继承自Object的、没有构造器的类。

如果你不想让mixin类可以当做普通class一样使用的话，就是用mixin关键字替换class关键字。
换句话说，mixin也可以使用class关键字定义(也可以是抽象类)，也可以当做普通class一样使用。

```
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

使用`on`关键字可以指定mixin的父类，

```
mixin MusicalPerformer on Musician {
  // ···
}
```

```
注意：对mixin关键字的支持是在Dart 2.1 版本引入的，早期的版本使用`abstract class`来代替的。
```

#### 异同
mixin也可以使用class关键字定义，也可以当做普通class一样使用。
mixin可以使用with定义，这样定义的mixin就只能通过with关键字引用了。

Dart是没有interface这种东西的，但并不意味着这门语言没有接口，事实上，Dart任何一个类都是接口，你可以实现任何一个类，只需要重写那个类里面的所有具体方法。

所以说，一个普通类class，即是普通类，也是接口，也可以当做mixin来使用。

参考：

[https://www.dartlang.org/guides/language/language-tour#classes](https://www.dartlang.org/guides/language/language-tour#classes)

[Flutter基础：理解Dart的Mixin继承机制](https://kevinwu.cn/p/ae2ce64/#%E5%9C%BA%E6%99%AF)

[Mixin是什么概念?——知乎](https://www.zhihu.com/question/20778853)

