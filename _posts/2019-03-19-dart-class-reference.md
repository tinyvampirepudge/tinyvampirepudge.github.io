---
layout: post
title:  Dart中的类——初始化列表、命名构造器、factory构造器、常量构造器、构造器私有化、get和set方法、枚举
date:   2019-03-19 20:35:26 +0800
categories: Dart
tag: [Dart, 初始化列表, 命名构造器, factory构造器]
---

* content
{:toc}



### Dart中的类——初始化列表、命名构造器、factory构造器、常量构造器、构造器私有化、get和set方法、枚举

### 1、调用成员变量——使用"."来调用成员变量或方法

```
var p = Point(2, 2);

// Set the value of the instance variable y.
p.y = 3;

// Get the value of y.
assert(p.y == 3);

// Invoke distanceTo() on p.
num distance = p.distanceTo(Point(4, 4));
```

使用 "?." 来避免对象为空导致的异常，参照kotlin中的类型安全。

```
// If p is non-null, set its y value to 4.
p?.y = 4;
```

### 2、使用构造器

构造器名字既可以有两种形式——`ClassName`、`ClassName.inentifier`。如下所示：

```
var p1 = Point(2, 2);
var p2 = Point.fromJson({'x': 1, 'y': 2});
```

当然了，这里我们是省略了`new`关键字的，如下所示：

```
var p1 = new Point(2, 2);
var p2 = new Point.fromJson({'x': 1, 'y': 2});
```

#### constant constructors

一些类提供了[常量构造器](https://www.dartlang.org/guides/language/language-tour#constant-constructors)。通过在构造器前面添加const关键字，可以创建一个编译时常量。

```
class ImmutablePoint {
  static final ImmutablePoint origin =
      const ImmutablePoint(0, 0);

  final num x, y;

  const ImmutablePoint(this.x, this.y);
}
```

```
var p = const ImmutablePoint(2, 2);
```

##### 使用constant-constructors生成同一个实例

使用构造器创建两个相等的编译时常量，它们会生成同一个实例，如下所示：

```
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);

assert(identical(a, b)); // They are the same instance!
```

##### 省略const关键字
在常量上下文内部，你可以省略const关键字。如下所示：

```
省略前：
// Lots of const keywords here.
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};
```

除了第一个const关键字，后面的const你都可以省略：

```
省略后：
// Only one const, which establishes the constant context.
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

##### constant-constructors也可以生成非常量的对象
如果一个常量构造器，在常量上下文外部被调用，并且没有const关键字，此时它会生成一个非常量的对象。

```
var a = const ImmutablePoint(1, 1); // Creates a constant
var b = ImmutablePoint(1, 1); // Does NOT create a constant

assert(!identical(a, b)); // NOT the same instance!
```

### 3、获取对象的type
如果想要获取某个对象在运行时的type，就可以使用对象的runtimeType属性，该属性返回一个[Type](https://api.dartlang.org/stable/2.2.0/dart-core/Type-class.html)对象。

```
print('The type of a is ${a.runtimeType}');
```

### 4、初始化实例变量（类似Java中的成员变量）
如下声明变量：

```
class Point {
  num x; // Declare instance variable x, initially null.
  num y; // Declare y, initially null.
  num z = 0; // Declare z, initially 0.
}
```

#### 所有未初始化的实例变量的默认值都为`null`。

#### 所有实例变量都会生成一个隐式的`getter`方法。`非final`类型的实例变量还会生成一个隐式的setter方法。细节请参考[Getters and setters](https://www.dartlang.org/guides/language/language-tour#getters-and-setters)。

```
class Point {
  num x;
  num y;
}

void main() {
  var point = Point();
  point.x = 4; // Use the setter method for x.
  assert(point.x == 4); // Use the getter method for x.
  assert(point.y == null); // Values default to null.
}
```

你也可以通过`get`和`set`关键字去重写属性默认的`getters`和`setters`方法。

```
class Rectangle {
  num left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // Define two calculated properties: right and bottom.
  num get right => left + width;
  set right(num value) => left = value - width;
  num get bottom => top + height;
  set bottom(num value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

#### 实例变量初始化时机

如果你的实例变量是通过声明的方式创建的（不是在构造器或者方法中），那么当实例对象一创建它就会赋值，这个时机在`构造器`和`初始化列表`执行之前。

### 5、构造器
创建一个方法名称跟类名相同的方法，即可声明一个构造器。当然，方法名也可以添加额外的标识符，例如[Named constructors](https://www.dartlang.org/guides/language/language-tour#named-constructors)。

自动生成的构造器是最常用的形式，如下所示：

```
class Point {
  num x, y;

  Point(num x, num y) {
    // There's a better way to do this, stay tuned.
    this.x = x;
    this.y = y;
  }
}
```

这里的`this`关键字表示当前实例。

```
注意：默认情况下Dart中会省略this关键字，只有在命名有歧义时才会使用this标识。
```

#### 构造器的语法糖
将构造器参数的值赋值给实例变量模式是很普遍的，Dart为此提供了语法糖，让这个过程简化：

```
class Point {
  num x, y;

  // Syntactic sugar for setting x and y
  // before the constructor body runs.
  Point(this.x, this.y);
}
```

#### 默认构造器
如果没有声明构造器，会提供一个默认的构造器。默认构造器是无参的，并且会调用父类的无参构造器。

#### 构造器不会被继承
子类不会从父类继承构造器。子类如果不声明构造器的话，那么就只会有默认的无参构造器。

#### 命名构造器——`Named Constructors`
在类中使用命名构造器实现多个构造器，可以提供额外的清晰度：

```
class Point {
  num x, y;

  Point(this.x, this.y);

  // Named constructor
  Point.origin() {
    x = 0;
    y = 0;
  }
}
```

**牢记构造器不能继承。** 这意味着父类的具名构造器不会被子类继承。如果你想使用父类中定义的具名构造器来创建子类实例，那么你不需在子类实现那个具名构造器。

#### 调用非默认的父类构造器——构造器、初始化列表执行顺序
默认情况下，子类中的构造器调用父类中默认无参构造器。父类构造器是在子类构造器的方法体第一行调用的。

如果此时还是用了[initializer list](https://www.dartlang.org/guides/language/language-tour#initializer-list)，那么它会在父类被调用之前执行。

总而言之，执行顺序如下：

```
1、initializer list
2、superclass’s no-arg constructor
3、main class’s no-arg constructor
```

如果父类没有无名无参构造器，那你必须手工调用父类的某个构造器。指定父类构造器是在冒号":"后面，构造器的方法体前面（如果有的话）。

下面这个例子中，子类Employee的构造器调用了父类的命名构造器。

```
class Person {
  String firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person does not have a default constructor;
  // you must call super.fromJson(data).
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
}

main() {
  var emp = new Employee.fromJson({});

  // Prints:
  // in Person
  // in Employee
  if (emp is Person) {
    // Type check
    emp.firstName = 'Bob';
  }
  (emp as Person).firstName = 'Bob';
}
```

##### 父类构造器传参规则
由于父类构造器的参数是在子类构造器调用之前被赋值的，并且参数可以是表达式——比如一个方法，所以可以有如下代码：

```
class Employee extends Person {
  Employee() : super.fromJson(getDefaultData());
  // ···
}
```

```
需要注意的是，父类构造器的参数是无法获取this的。例如，父类构造器的参数可以调用静态方法，但是不能调用实例方法。
```

#### 初始化列表——Initializer list
除了调用父类构造器，在子类构造器的方法体执行之前，你也可以初始化实例变量。不同的初始化变量之间用逗号分隔开。

```
// Initializer list sets instance variables before
// the constructor body runs.
Point.fromJson(Map<String, num> json)
    : x = json['x'],
      y = json['y'] {
  print('In Point.fromJson(): ($x, $y)');
}
```

```
需要注意的是，初始化变量的右侧不能访问this。
```

##### assert校验输入的值
在开发过程中，你可以在初始化列表中，通过assert来校验输入的值。

```
Point.withAssert(this.x, this.y) : assert(x >= 0) {
  print('In Point.withAssert(): ($x, $y)');
}
```

##### 给final实例变量赋值。
在初始化列表中给final修饰的实例变量赋值是很方便的。如下所示：

```
import 'dart:math';

class Point {
  final num x;
  final num y;
  final num distanceFromOrigin;

  Point(x, y)
      : x = x,
        y = y,
        distanceFromOrigin = sqrt(x * x + y * y);
}

main() {
  var p = new Point(2, 3);
  print(p.distanceFromOrigin);
}

output: 
3.605551275463989
```

#### 重定向构造器——Redirecting constructors
某些构造器的目的只是为了调用同一个类中的另一个构造器，我们称之为重定向构造器。

重定向构造器的方法体为空，冒号后面跟着被调用的构造器。

```
class Point {
  num x, y;

  // The main constructor for this class.
  Point(this.x, this.y);

  // Delegates to the main constructor.
  Point.alongXAxis(num x) : this(x, 0);
}
```

#### 常量构造器——Constant constructors
如果你的类生成的对象不会改变，你可以让这些对象成为编译时常量。

你需要使用const来定义构造器，并且确保所有的实例变量都用final修饰。

```
class ImmutablePoint {
  static final ImmutablePoint origin =
      const ImmutablePoint(0, 0);

  final num x, y;

  const ImmutablePoint(this.x, this.y);
}
```

常量构造器也会生成非常量类型。更多细节请看[using constructors](https://www.dartlang.org/guides/language/language-tour#using-constructors)部分。

#### 工厂构造器——Factory constructors
使用factory关键字来修饰构造器，不一定每次都会生成新的类的实例。比如，工厂构造器可能会返回缓存中的某个实例，或者它返回子类的一个实例。

下面这个例子展示了工厂构造器从内存中返回对象：

```
class Logger {
  final String name;
  bool mute = false;

  // _cache is library-private, thanks to
  // the _ in front of its name.
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    if (_cache.containsKey(name)) {
      return _cache[name];
    } else {
      final logger = Logger._internal(name);
      _cache[name] = logger;
      return logger;
    }
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```

```
注意：工厂构造器不能访问this。
```

调用工厂构造器跟使用别的构造器是一样的：

```
var logger = Logger('UI');
logger.log('Button clicked');
```

### 6、方法

方法用来描述对象的行为。

#### 实例方法

对象的实例方法可以访问实例变量和this。如下所示：

```
import 'dart:math';

class Point {
  num x, y;

  Point(this.x, this.y);

  num distanceTo(Point other) {
    var dx = x - other.x;
    var dy = y - other.y;
    return sqrt(dx * dx + dy * dy);
  }
}
```

#### Getters and setters

get和set是比较特殊的方法，通过它们可以对对象的属性进行读写操作。所有实例变量都会生成一个隐式的`getter`方法，`非final`类型的实例变量还会生成一个隐式的setter方法。

通过`get`和`set`关键字，你可以重写`get`和`set`方法，以添加一些额外的属性。

```
class Rectangle {
  num left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // Define two calculated properties: right and bottom.
  num get right => left + width;
  set right(num value) => left = value - width;
  num get bottom => top + height;
  set bottom(num value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

tips:

```
Note：无论get方法是否显式定义，操作符（例如 ++ ）都会按照期望运行。为了避免不可预知的错误，操作符只会调用get方法一次，将变量保存在临时变量中。
```


#### 抽象方法
实例的`get`和`set`方法都可以定义为抽象的，定义一个具体实现取决于其他类的一个接口。

抽象方法只能存在于抽象类中。

抽象方法使用分号";"来代替方法体。 如下所示：

```
abstract class Doer {
  // Define instance variables and methods...

  void doSomething(); // Define an abstract method.
}

class EffectiveDoer extends Doer {
  void doSomething() {
    // Provide an implementation, so the method is not abstract here...
  }
}
```

### 7、抽象类——abstract
抽象类是不能被实例化的类，它使用`abstract`修饰符定义。在定义接口时抽象类很有用，会包含一些实现。

如果你想实例化抽象类，可以通过[工厂构造器](https://www.dartlang.org/guides/language/language-tour#factory-constructors)实现。

抽象类经常包含抽象方法。下面的例子中声明了包含了一个抽象方法的抽象类。

```
// This class is declared abstract and thus
// can't be instantiated.
abstract class AbstractContainer {
  // Define constructors, fields, methods...

  void updateChildren(); // Abstract method.
}
```

### 8、隐式接口——[Implicit interfaces](https://www.dartlang.org/guides/language/language-tour#implicit-interfaces)

每个类的内部都隐式的定义了一个接口，这个接口包含类的成员的所有实例，以及类实现的所有接口。

如果你想让A类支持B类所有的API，并且不通过继承B类实现的方式，那么A类应该实现B接口。

一个类可以实现一个或多个接口，通过`implements`关键字。

```
// A person. The implicit interface contains greet().
class Person {
  // In the interface, but visible only in this library.
  final _name;

  // Not in the interface, since this is a constructor.
  Person(this._name);

  // In the interface.
  String greet(String who) => 'Hello, $who. I am $_name.';
}

// An implementation of the Person interface.
class Impostor implements Person {
  get _name => '';

  String greet(String who) => 'Hi $who. Do you know who I am?';
}

String greetBob(Person person) => person.greet('Bob');

void main() {
  print(greetBob(Person('Kathy')));
  print(greetBob(Impostor()));
}
```

下面这段代码中的class实现了多个接口：

```
class Point implements Comparable, Location {...}
```

### 9、类的继承——Extending a class
使用`extends`关键字创建一个子类，使用super关键字指向父类。

```
class Television {
  void turnOn() {
    _illuminateDisplay();
    _activateIrSensor();
  }
  // ···
}

class SmartTelevision extends Television {
  void turnOn() {
    super.turnOn();
    _bootNetworkInterface();
    _initializeMemory();
    _upgradeApps();
  }
  // ···
}
```

#### 方法的重写——Overriding members
子类可以覆盖实例方法、get和set方法。你可以使用`@override`注解表明你想覆盖一个成员。

```
class SmartTelevision extends Television {
  @override
  void turnOn() {...}
  // ···
}
```

为了保证方法参数和实例变量的[类型安全](https://www.dartlang.org/guides/language/sound-dart)，我们需要使用[`covariant——协变`](https://www.dartlang.org/guides/language/sound-problems#the-covariant-keyword)关键字。


#### 可重写的操作符——Overridable operators
你可以重写下表中的操作符。

![](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20190319-170222.png)

```
Note： “!=” 不是可以重写的操作符。表达式 e1 != e2 相对于 !(e1 == e2) 来说，仅仅是个语法糖。
```

下面这个例子中重写了 `+` 和 `-` 操作符:

```
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  // Operator == and hashCode not shown. For details, see note below.
  // ···
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}
```

如果你重写了`==`操作符，那么你也应该重写对象的`hashCode`的`get`方法。重写`==`和`hashCode`的示例，请看[Implementing map keys](https://www.dartlang.org/guides/libraries/library-tour#implementing-map-keys).

#### noSuchMethod()
你可以重写`noSuchMethod()`方法，该方法可以检测到对不存在的方法或者实例变量的使用。

```
class A {
  // Unless you override noSuchMethod, using a
  // non-existent member results in a NoSuchMethodError.
  @override
  void noSuchMethod(Invocation invocation) {
    print('You tried to use a non-existent member: ' +
        '${invocation.memberName}');
  }
}
```

一般情况下你是不能调用未实现的方法的，符合以下情况之一的除外：
1、接收者的静态类型是dynamic。
2、接收者的静态类型中定义了未实现的方法（抽象类），并且接收者的动态类型有一个`noSuchMethod()`方法的实现，该实现与Object类的不同。


更多细节请查看[noSuchMethod forwarding specification](https://github.com/dart-lang/sdk/blob/master/docs/language/informal/nosuchmethod-forwarding.md).

### 10、枚举类型
枚举类型经常被称为`enumerations`或者`enums`，它是一种特殊的类，经常用来表示固定的数字或者常量值。

#### 使用枚举
使用`enum`关键字声明枚举类型：

```
enum Color { red, green, blue }
```

枚举中的每个值都有一个`index`属性及对应的get方法，index的值从0开始。

```
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);
```

通过枚举的`values`属性，可以得到一个包含枚举中所有数值的列表。

```
List<Color> colors = Color.values;
assert(colors[2] == Color.blue);
```

当然了，你可以在switch声明中使用枚举：

```
var aColor = Color.blue;

switch (aColor) {
  case Color.red:
    print('Red as roses!');
    break;
  case Color.green:
    print('Green as grass!');
    break;
  default: // Without this, you see a WARNING.
    print(aColor); // 'Color.blue'
}
```

枚举类型有如下限制：
* You can’t subclass, mix in, or implement an enum.
* You can’t explicitly instantiate an enum.

更多细节请查看[Dart language specification](https://www.dartlang.org/guides/language/spec).

### 11、Adding features to a class: mixins
`mixins`是一种实现多重继承的方式。

在`with`关键字后面可以跟随一个或多个mixin的名称。如下所示：

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

### 12、静态变量和方法
使用static关键字可以实现class级别的变量和方法。

#### 静态变量
类的静态变量常用来表示类级别的状态和常量：

```
class Queue {
  static const initialCapacity = 16;
  // ···
}

void main() {
  assert(Queue.initialCapacity == 16);
}
```

```
注意：静态变量在使用时才会初始化。
```

#### 静态方法
静态方法不能操作实例对象，因此也无法访问`this`。

```
import 'dart:math';

class Point {
  num x, y;
  Point(this.x, this.y);

  static num distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

void main() {
  var a = Point(2, 2);
  var b = Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(2.8 < distance && distance < 2.9);
  print(distance);
}
```

```
注意：对于一些公用或者广泛使用的工具类和函数，可以考虑使用顶级函数来代替静态方法。
```

你可以将静态方法视为编译时常量。比方说你可以将静态方法当做参数传递给常量构造器。

### 参考：

[https://www.dartlang.org/guides/language/language-tour#classes](https://www.dartlang.org/guides/language/language-tour#classes)

