---
layout: post
title:  Runes——Dart
date:   2019-02-24 23:22:29 +0800
categories: Dart
tag: [Runes]
---

* content
{:toc}



### Runes——Dart
Dart中，Runes是指UTF-32定义的Unicode字符串。

Unicode使用数字表示世界上所有的字母、数字和符号。因为Dart中的String是一系列UTF-16字节单元，而在String中想要表示32位的Unicode值，则需要特殊的语法。

一般我们使用 `\uXXXX` 这种形式表示一个Unicode码，`XXXX`表示4个十六进制值。例如，字符（♥）的Unicode字符是`\u2665`。

当十六进制数据多余或者少于4位时，将十六进制数放入到花括号中，例如，微笑表情（😆）是`\u{1f600}`。

String类中有几个属性你可以用来获取`rune`信息。`codeUnitAt`和`codeUnits`属性返回十六位的字符单元。通过`runes`属性可以获取一个字符串的runes形式的值。

下面这个例子展示了runes、十六位的字节单元以及三十二位代码点之间的关系。
```
var clapping = '\u{1f44f}';
print(clapping);
print(clapping.codeUnits);
print(clapping.runes.toList());

Runes input = new Runes(
    '\u2665  \u{1f605}  \u{1f60e}  \u{1f47b}  \u{1f596}  \u{1f44d}');
print(new String.fromCharCodes(input));

输出：
👏
[55357, 56399]
[128079]
♥  😅  😎  👻  🖖  👍
```

注意：使用数组操作符操作runes时要特别消息。取决于特定语言、字符集、和操作，这个方法特别容易失败。更多信息请参考 [How do I reverse a String in Dart?](https://stackoverflow.com/questions/21521729/how-do-i-reverse-a-string-in-dart) on Stack Overflow.

参考:
[https://www.dartlang.org/guides/language/language-tour#runes](https://www.dartlang.org/guides/language/language-tour#runes)
