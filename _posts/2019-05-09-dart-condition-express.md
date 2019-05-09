---
layout: post
title:  Dart条件表达式
date:   2019-05-09 11:15:59 +0800
categories: Dart
tag: [Dart]
---

* content
{:toc}



### Dart条件表达式

Dart中的条件表达式有两种形式，用来替换简单的`if-else`语句。

#### condition ? expr1 : expr2
常见的形式如下，跟Java中条件表达式形式一致。

```
condition ? expr1 : expr2
```

如果condition为true，则执行并返回expr1，否则就执行并返回expr2。

当你的赋值操作依赖于一个boolean类型表达式时，可以考虑使用它。

#### expr1 ?? expr2
我们看下第二种形式：

```
expr1 ?? expr2
```
如果expr1的值非空，就返回它，否则就返回expr2的值。

当你的Boolean表达式依赖于是否为null时，开率使用它。

#### 实例

下面通过例子来说明。

```
String playerName(String name) {
  if (name != null) {
    return name;
  } else {
    return 'Guest';
  }
}

main(){
  print('${playerName("猫了个咪")}');
  print('${playerName(null)}');
}

output:
猫了个咪
Guest
```

上面的playerName方法使用的是普通的`if-else`语句，我们先用`condition ? expr1 : expr2`形式来简化，代码如下：

```
String playerName(String name) => name != null ? name : 'Guest';
```

这个例子还可以用`expr1 ?? expr2`来优化，代码如下：

```
String playerName(String name) => name ?? 'Guest';
```

参考：

[https://www.dartlang.org/guides/language/language-tour#conditional-expressions](https://www.dartlang.org/guides/language/language-tour#conditional-expressions)

