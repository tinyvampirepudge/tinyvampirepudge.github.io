---
layout: post
title:  Functions——Dart
date:   2019-02-25 22:04:30 +0800
categories: Dart
tag: [Functions]
---

* content
{:toc}



### Functions——Dart
Dart是一个完全面向对象的语言，它的方法也是对象，对应的类型为[Function](https://api.dartlang.org/stable/2.1.1/dart-core/Function-class.html)。

这意味着方法也能被赋值给变量，或者当做参数传递给其他方法。

下面是一个方法的的示例：

```
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```

#### [函数参数](https://blog.csdn.net/qq_26287435/article/details/87914620)
* 可选命名参数
* 可选位置参数
* 参数默认值

具体请看[Functions Paramaters——Dart](https://blog.csdn.net/qq_26287435/article/details/87914620)

####  main()方法
每个app都必须有一个顶级的main()方法来作为app的入口。main()方法的返回值为`void`类型，方法参数为`List<String>`的可选参数。

下面的main方法可以接受命令行携带过来的参数：

```
// Run the app like this: dart args.dart 1 test
void main(List<String> arguments) {
  print(arguments);

  assert(arguments.length == 2);
  assert(int.parse(arguments[0]) == 1);
  assert(arguments[1] == 'test');
}

```

你可以使用[args library](https://pub.dartlang.org/packages/args)

#### 函数作为一等公民
##### 函数可以作为参数传递给另一个函数。
例如：

```
void printElement(int element) {
  print(element);
}

main(){
  var list = [1, 2, 3];

  // Pass printElement as a parameter.
  list.forEach(printElement);
}

output:
1
2
3
```

##### 将函数赋值给一个变量

```
var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';
assert(loudify('hello') == '!!! HELLO !!!');
```

这里使用了匿名方法。

#### 匿名方法
大多数方法都会被命名，比如说`main()`和`printElement()`。你也可以创建没有名字的方法，我们称之为匿名函数；还可以创建lambda表达式或者闭包。

你也可以将匿名函数赋值给一本变量，例如你将这个变量从集合中添加或者移除。

匿名方法的参数跟正常方法类似——0或多个参数，由逗号分隔，可选的类型注解，在一对圆括号内部。

下面的代码块包含了方法体：

```
([[Type] param1[, …]]) { 
  codeBlock; 
}; 
```

下面的示例定义了一个包含无类型参数item的匿名方法。这个方法被list中的每个item调用，会根据角标获取item的值，然后打印出对应的String。

```
var list = ['apples', 'bananas', 'oranges'];
list.forEach((item) {
  print('${list.indexOf(item)}: $item');
});

output:
0: apples
1: bananas
2: oranges
```

##### 使用箭头表达式简写
如果匿名方法的方法体中只包含一行表达式，这时可以使用箭头表达式来简化。

```
var list = ['apples', 'bananas', 'oranges'];
list.forEach((item) => print('${list.indexOf(item)}: $item'));

output:
0: apples
1: bananas
2: oranges
```

#### Lexical scope——作用域
Dart语言支持作用域，这意味着变量的作用域是由代码所处的位置静态决定的。

你可以根据方法体的花括号来判断变量是否在作用域内部。

```
bool topLevel = true;

void main() {
  var insideMain = true;

  void myFunction() {
    var insideFunction = true;

    void nestedFunction() {
      var insideNestedFunction = true;

      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
```

现在最内层的nestedFunction方法可以调用外部各个层级的变量，包括顶级的成员变量。

#### Lexical closures——闭包
闭包是一个可以访问自己的作用域变量的函数对象，甚至闭包当原始作用域的外部使用时也可以。


闭包的概念参考[学习Javascript闭包（Closure）——阮一峰](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

在下面例子中，`makeAdder()`方法捕获了变量`addBy`，不管返回的方法如何，它已经记录下了`addBy`。

```
/// Returns a function that adds [addBy] to the
/// function's argument.
Function makeAdder(num addBy){
  abc(num i){
    return addBy + i;
  }
  return abc;
}

void main() {
  // Create a function that adds 2.
  var add2 = makeAdder(2);

  // Create a function that adds 4.
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```

注意，这里的闭包方法可以进行简写，比如省略返回值：

```
 makeAdder(num addBy){
  abc(num i){
    return addBy + i;
  }
  return abc;
}
```

内部方法可以使用匿名方法代替：

```
Function makeAdder(num addBy) {
  return (num i) => addBy + i;
}
```

当然，方法的返回值也可以省略

```
makeAdder(num addBy) {
  return (num i) => addBy + i;
}
```

#### 测试方法的一致性

下面这个例子测试了顶级方法、静态方法和实例方法的一致性。

```
void foo() {} // A top-level function

class A {
  static void bar() {} // A static method
  void baz() {} // An instance method
}

void main() {
  var x;

  // Comparing top-level functions.
  x = foo;
  assert(foo == x);// true

  // Comparing static methods.
  x = A.bar;
  assert(A.bar == x);// true

  // Comparing instance methods.
  var v = A(); // Instance #1 of A
  var w = A(); // Instance #2 of A
  var y = w;
  x = w.baz;

  // These closures refer to the same instance (#2),
  // so they're equal.
  assert(y.baz == x);// true

  // These closures refer to different instances,
  // so they're unequal.
  assert(v.baz != w.baz);// true
}
```

#### 返回值
所有非void方法都有返回值。如果没有指定返回值，则方法体的末尾会隐式地添加一句`return null;`。

```
foo() {}

void baz() {}

main() {
  assert(foo() == null);
//  assert(baz() == null);// 编译期异常
}
```



参考：
[https://www.dartlang.org/guides/language/language-tour#functions](https://www.dartlang.org/guides/language/language-tour#functions)