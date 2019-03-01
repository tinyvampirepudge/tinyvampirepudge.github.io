---
layout: post
title:  Functions Paramaters——Dart
date:   2019-02-25 12:04:23 +0800
categories: dart
tag: [Dart函数参数类型, Functions Paramaters]
---

* content
{:toc}



### Functions Paramaters——Dart
Dart是一个完全面向对象的语言，它的方法也是对象，对应的类型为[Function](https://api.dartlang.org/stable/2.1.1/dart-core/Function-class.html)。

这意味着方法也能被赋值给变量，或者当做参数传递给其他方法。

你也可以像操作方法那样操作Dart的类的实例，具体请参照[Callable classes](https://www.dartlang.org/guides/language/language-tour#callable-classes)

下面是一个方法的的示例：
```
bool isNoble(int atomicNumber) {
return _nobleGases[atomicNumber] != null;
}
```

#### 省略参数类型
你可以省略参数类型，
```
isNoble(atomicNumber) {
return _nobleGases[atomicNumber] != null;
}
```

#### 只有一行表达式时可以使用简写语法
如果方法只有一行表达式，你可以使用简写语法：
```
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```
`=> expr`语法是`{ return expr; }`的简写形式.`=>`符号可以参考箭头表达式。

注意：只有表达式——不是一个声明——可以出现在`=>`和`;`之间。例如，你不能放入if语句，但是你可以使用一个条件表达式。

方法的参数类型有两种，`required`和`optional`。

`required`参数会优先展示，后面跟着可选参数。

#### 可选参数
可选参数也可以添加命名或者位置参数，但不能同时添加。

##### 可选命名参数—— {}
定义方法时，通过`{param1, param2, …}`的形式来指定命名参数，如下所示：
```
/// Sets the [bold] and [hidden] flags ...
void enableFlags({bool bold, bool hidden}) {

}
```
调用该方法时，通过`paramName: value`的形式来指定命名参数。如下所示：
```
// 正常调用
enableFlags(bold: false, hidden: false);
// 参数调换顺序
enableFlags(hidden: false, bold: false);
// 只传入第一个
enableFlags(bold: false);
// 只传入第二个参数
enableFlags(hidden: false);
```

Flutter中的widget构造器只能采用命名参数的形式来定义。

在Dart代码中， 你可以给任意的命名参数添加`@required`参数，表示这个命名参数是必不可少的。代码如下：
```
const Scrollbar({Key key, @required Widget child})
```

[Required](https://pub.dartlang.org/documentation/meta/latest/meta/required-constant.html)定义在[meta](https://pub.dartlang.org/packages/meta)包下。
直接通过`import package:meta/meta.dart`导入包，或者导入其他包含meta的包，比如Flutter的`package:flutter/material.dart`。

##### 可选位置参数—— []
使用`[]`将参数(一个或多个)包裹起来，即可将它们标记为可选参数。
另外，可选位置参数需要放在参数列表的末尾。
```
// 可选位置参数
String say(String from, String msg, [String device, String name]) {
var result = '$from says $msg';
if (device != null) {
result = '$result with a $device';
}
return result;
}
```
```
// 不调用可选参数
print('${say("China", "Beijing")}');
// 调用第一个可选参数
print('${say("China", "Beijing", "mate 10 pro")}');
// 调用两个可选参数
print('${say("China", "Beijing", "mate 20 X", "wangcai")}');
```
输出结果：
```
China says Beijing
China says Beijing with a mate 10 pro ,
China says Beijing with a mate 20 X , name: wangcai
```

#### 参数默认值—— =
可以使用`=`给参数设置默认值，命名参数和可选参数均可。默认值必须是编译器常量。如果没有提供默认值，则默认值就是null。

##### 给命名参数设置默认值
代码如下：
```
void setDefaultValues(
{bool value1 = true,
String value2 = "value2",
int value3 = 10,
double value4}) {
print('value1: $value1, value2: $value2, value3: $value3, value4: $value4');
}
```
```
// 参数默认值
setDefaultValues();
setDefaultValues(value1: false, value2: "abc", value3: 100, value4: 1000.11);
setDefaultValues(value1: true);
setDefaultValues(value2: "tiny");
setDefaultValues(value3: 1000);
setDefaultValues(value4: 100.0);
```
输出结果：
```
value1: true, value2: value2, value3: 10, value4: null
value1: false, value2: abc, value3: 100, value4: 1000.11
value1: true, value2: value2, value3: 10, value4: null
value1: true, value2: tiny, value3: 10, value4: null
value1: true, value2: value2, value3: 1000, value4: null
value1: true, value2: value2, value3: 10, value4: 100.0
```
需要注意的是，老版本代码可能使用`:`来设置参数默认值，这个已经被废弃了，现在推荐使用`=`来指定参数默认值。

##### 给可选参数设置默认值
```
// 可选参数设置默认值
void setDefaultValuesToOptional(bool value1, String value2,
[int value3 = 10, double value4]) {
print('value1: $value1, value2: $value2, value3: $value3, value4: $value4');
}
```
```
// 参数默认值——可选参数
setDefaultValuesToOptional(true,"11111");
setDefaultValuesToOptional(true,"11111",100);
setDefaultValuesToOptional(true,"11111",100,1000.0);
```
输出结果：
```
value1: true, value2: 11111, value3: 10, value4: null
value1: true, value2: 11111, value3: 100, value4: null
value1: true, value2: 11111, value3: 100, value4: 1000.0
```

##### 默认参数也可以设置list和map
```
void doStuff(
{List<int> list = const [1, 2, 3],
Map<String, String> gifts = const {
'first': 'paper',
'second': 'cotton',
'third': 'leather'
}}) {
print('list:  $list');
print('gifts: $gifts');
}
```
```
doStuff();
doStuff(list: [4,5,6],gifts: {"11":"11","22":"22"});
doStuff(list: [1]);
doStuff(gifts: {"33":"33"});
```
输出结果：
```
list:  [1, 2, 3]
gifts: {first: paper, second: cotton, third: leather}
list:  [4, 5, 6]
gifts: {11: 11, 22: 22}
list:  [1]
gifts: {first: paper, second: cotton, third: leather}
list:  [1, 2, 3]
gifts: {33: 33}
```
