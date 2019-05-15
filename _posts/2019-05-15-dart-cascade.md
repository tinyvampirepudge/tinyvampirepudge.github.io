---
layout: post
title:  Dart中的Cascade符号——".."
date:   2019-05-15 13:49:40 +0800
categories: Dart
tag: [Cascade]
---

* content
{:toc}



### Dart中的Cascade符号——".."

级联表达式（..）允许你在同一个对象上连续使用操作符。

除了方法调用之外，你还可以获取同一个对象上的成员变量。这样做通常省去了创建临时变量的步骤，同时允许你写出更流畅的代码。

严格来说，级联表达式的两个点（”..“）的语法并不能算作操作符，它仅仅是Dart语法的一部分。


```
Here, ".." is the cascaded method invocation operation.  The ".." syntax invokes a method (or setter or getter) but discards the result, and returns the original receiver instead. 
```

".."是级联方法调用操作符。".."语法调用一个方法（getter或setter）并丢弃它的返回值，同时返回级联操作符最初的接收者。

```
In brief, method cascades provide a syntactic sugar for situations where the receiver of a method invocation might otherwise have to be repeated. 
```

简单来说，当方法调用的接收者重复时，方法的级联操作符就是为这种情况提供的语法糖。

#### 示例1

我们定义一个Student对象，然后创建一个Student对象，通过级联表达式依次调用它的各个方法和setter属性。


```
class Student {
  String string;

  void testMethod() {
    print("This is a  test method");
  }

  void testMethod1() {
    print("This is a  test method1");
  }

  String printString() {
    print("string: $string");
    return string;
  }
}

main() {
  Student()
    ..testMethod()
    ..testMethod1()
    ..string = "猫了个咪"
    ..printString();
}
```

上面的级联表达式调用下方的调用是等效的。

```
main() {
  var student = Student();
  student.testMethod();
  student.testMethod1();
  student.string = "猫了个咪";
  student.printString();
}
```

通过对比，可以明显的看出，是不是少了临时变量stud

#### 示例2

当你在`有具体返回值的方法上`使用级联表达式时需要注意，级联表达式不能用于void类型上。

```
var result = StringBuffer()
                 .write('foo')
                 ..write('bar');
```
上述调用会报错：
`// Error: method 'write' isn't defined for 'void'.`

因为StringBuffer#write方法的返回值为void，你不能在void类型上使用级联表达式。

我们可以适当做下修改，就可以继续使用级联表达式了，如下所示：

```
main(){
  var result = StringBuffer()
                 ..write('foo')
                 ..write('bar');

  print('result:$result'); // result:foobar
}
```

#### 总结——适用场景

简而言之，当我们需要对同一个对象进行多次操作时，我们可以考虑使用级联表达式来简化我们的操作，以此对同一个对象进行连续调用。


#### 参考：

[cascade-notation](https://www.dartlang.org/guides/language/language-tour#cascade-notation-)


[Method Cascades in Dart](https://news.dartlang.org/2012/02/method-cascades-in-dart-posted-by-gilad.html)


[dart-programming-cascade-operator](https://www.tutorialspoint.com/programming_example/CDjWMF/dart-programming-cascade-operator)

