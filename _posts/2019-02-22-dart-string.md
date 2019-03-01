---
layout: post
title:  String——Dart
date:  2019-02-22 11:07:50 +0800
categories: Dart
tag: [String]
---

* content
{:toc}



### String——Dart
Dart中的String是一系列的UTF-16的字符单元。

#### 1、使用单引号或者双引号均可创建一个String：
```
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

#### 2、${expression}

使用 ${expression} 可以在字符串中直接添加表达式的值。

如果表达式是一个标识符，`{}`还可以省略，直接使用`$expression`即可。

你可以通过Object#toString()方法，得到对象的string表示。
```
var s = 'string interpolation';

assert('Dart has $s, which is very handy.' ==
'Dart has string interpolation, ' +
'which is very handy.');
assert('That deserves all caps. ' +
'${s.toUpperCase()} is very handy!' ==
'That deserves all caps. ' +
'STRING INTERPOLATION is very handy!');
```

```
Note： == 操作符可以检测两个对象是否相等。对String来说，如果它们拥有相同的字符单元序列，那它们就是相等的。
```

#### 3、字符串连接
##### 单行字符串
Dart中，单行字符串既可以使用 + 号链接，也可以使用相邻的字符串字面值。如下所示：
```
var s1 = 'String '
'concatenation'
" works even over line breaks.";
assert(s1 ==
'String concatenation works even over '
'line breaks.');

var s2 = 'The + operator ' + 'works, as well.';
assert(s2 == 'The + operator works, as well.');
```

##### 多行字符串
使用三个单引号或者双引号创建多行字符串。
```
var s1 = '''
You can create
multi-line strings like this one.
''';

var s2 = """This is also a
multi-line string.""";
```

#### 4、原始字符串—— "raw" string
你可以通过前缀 r 生成一个原始字符串。
```
var s = r'In a raw string, not even \n gets special treatment.';
```
如何在String中表示Unicode字符，参见[Runes](https://www.dartlang.org/guides/language/language-tour#runes)。

#### 编译时常量

字符串字面值是编译时常量，前提是任何字符串插入值的类型也是编译时常量，并且是null、数值类型、字符串、布尔类型 之一。
```
main() {
// These work in a const string.
const aConstNull = null;
const aConstNum = 0;
const aConstNum1 = 0.1;
const aConstBool = true;
const aConstString = 'a constant string';

const validConstString = '$aConstNull $aConstNum $aConstNum1 $aConstBool $aConstString';

print('validConstString:$validConstString');


// These do NOT work in a const string.
var aNull = null;
var aNum = 0;
var aNum1 = 0.1;
var aBool = true;
var aString = 'a string';
const aConstList = [1, 2, 3];

//下面这行代码编译不通过
//   const invalidConstString = '$aNum $aBool $aString $aConstList';
}
```
更多信息请参阅：[Strings and regular expressions](https://www.dartlang.org/guides/libraries/library-tour#strings-and-regular-expressions)

参考：[https://www.dartlang.org/guides/language/language-tour#strings](https://www.dartlang.org/guides/language/language-tour#strings)

