---
layout: post
title:  dart中json和对象互转
date:   2019-04-01 21:06:25 +0800
categories: Dart
tag: [Json]
---

* content
{:toc}



### dart中json和对象互转

开发过程中，json是必不可少的基础技能之一。这里记录下，在Dart语言中，如何将json解析成实例对象，以及如何将实例对象转化成json字符串。

这里使用的工具是`dart:convert`包。

我们的目的很简单，待解析的json字符串格式如下：

```
{
"key":"wangdandan",
"value":"王蛋蛋的father"
}
```

#### json字符串解析成实例对象

##### 1、创建model对象

```
class JsonModelDemo {
  String key;
  String value;
}
```

##### 2、将实体类对象解析成json字符串。

我们创建一个实例对象，然后给这个实例对象赋值，接着使用`jsonDecode`方法解析实例对象。代码如下，

```
import 'dart:convert';

import 'package:dart_demo1/json/json_model.dart';

///  将实体类对象解析成json字符串
String generatePlatformJson({String key, String value}) {
  JsonModelDemo jsonModelDemo = new JsonModelDemo();
  jsonModelDemo.key = key;
  jsonModelDemo.value = value;
  String jsonStr = jsonEncode(jsonModelDemo);
  return jsonStr;
}

/// 这里写测试方法
main() {
  String result1 = generatePlatformJson(key: "result1", value: "result1Value");
  print('result1:$result1');
}
```

执行代码，报错如下：

```

lib/json/json_parse_util.dart:1: Warning: Interpreting this as package URI, 'package:dart_demo1/json/json_parse_util.dart'.
Unhandled exception:
Converting object to an encodable object failed: Instance of 'JsonModelDemo'
#0      _JsonStringifier.writeObject (dart:convert/json.dart:645:7)
#1      _JsonStringStringifier.printOn (dart:convert/json.dart:832:17)
#2      _JsonStringStringifier.stringify (dart:convert/json.dart:817:5)
#3      JsonEncoder.convert (dart:convert/json.dart:253:30)
#4      JsonCodec.encode (dart:convert/json.dart:164:45)
#5      jsonEncode (dart:convert/json.dart:76:10)
#6      generatePlatformJson (package:dart_demo1/json/json_parse_util.dart:10:20)
#7      main (package:dart_demo1/json/json_parse_util.dart:16:20)
#8      _startIsolate.<anonymous closure> (dart:isolate/runtime/libisolate_patch.dart:300:19)
#9      _RawReceivePortImpl._handleMessage (dart:isolate/runtime/libisolate_patch.dart:171:12)

Process finished with exit code 255
```

查找`Converting object to an encodable object failed: Instance of 'xxx'`这个错误，在stackoverflow上找到答案：[https://stackoverflow.com/questions/27739434/dart-object-json-string-failing-to-convert-to-json](https://stackoverflow.com/questions/27739434/dart-object-json-string-failing-to-convert-to-json)

我们给model实体类添加toJson方法：

```
class JsonModelDemo {
  String key;
  String value;

  /// jsonDecode(jsonStr) 方法中会调用实体类的这个方法。如果实体类中没有这个方法，会报错。
  Map toJson() {
    Map map = new Map();
    map["key"] = this.key;
    map["value"] = this.value;
    return map;
  }
}
```

这次再运行代码，解析成功，输出如下：

```
result1:{"key":"result1","value":"result1Value"}
```

#### 实例对象转化成json字符串
解析代码如下：

```
/// 将json字符串解析成实体类对象
JsonModelDemo parsePlatformJson(String jsonStr) {
  JsonModelDemo result = jsonDecode(jsonStr);
  return result;
}
```

测试代码如下：

```
JsonModelDemo modelDemo = parsePlatformJson(result1);
  print('parsePlatformJson:$modelDemo');
```

为了方便测试，在JsonModelDemo中重写toString方法;

```
@override
String toString() {
return 'JsonModelDemo{key: $key, value: $value}';
}
```

运行代码，报错如下：

```
Unhandled exception:
type '_InternalLinkedHashMap<String, dynamic>' is not a subtype of type 'JsonModelDemo'
#0      parsePlatformJson (package:dart_demo1/json/json_parse_util.dart:16:17)
#1      main (package:dart_demo1/json/json_parse_util.dart:25:29)
#2      _startIsolate.<anonymous closure> (dart:isolate/runtime/libisolate_patch.dart:300:19)
#3      _RawReceivePortImpl._handleMessage (dart:isolate/runtime/libisolate_patch.dart:171:12)
```

仔细观察报错，发现是类型不匹配，具体原因是jsonDecode方法返回的是Map<String, dynamic>，不是我们期望的实例对象。所以，我们还需要将Map<String, dynamic>转化为我们想要的实例对象。

在model中添加转化方法：

```
/// jsonDecode(jsonStr)方法返回的是Map<String, dynamic>类型，需要这里将map转换成实体类
static JsonModelDemo fromMap(Map<String, dynamic> map) {
    JsonModelDemo jsonModelDemo = new JsonModelDemo();
    jsonModelDemo.key = map['key'];
    jsonModelDemo.value = map['value'];
    return jsonModelDemo;
}
```

接着修改解析方法：

```
/// 将json字符串解析成实体类对象
JsonModelDemo parsePlatformJson(String jsonStr) {
  JsonModelDemo result = JsonModelDemo.fromMap(jsonDecode(jsonStr));
  return result;
}
```

运行代码，解析成功，输出如下：

```
result1:{"key":"result1","value":"result1Value"}
parsePlatformJson:JsonModelDemo{key: result1, value: result1Value}
```

参考：

[https://stackoverflow.com/questions/27739434/dart-object-json-string-failing-to-convert-to-json](https://stackoverflow.com/questions/27739434/dart-object-json-string-failing-to-convert-to-json)

[https://www.dartlang.org/guides/libraries/library-tour#dartconvert---decoding-and-encoding-json-utf-8-and-more](https://www.dartlang.org/guides/libraries/library-tour#dartconvert---decoding-and-encoding-json-utf-8-and-more)

[https://medium.com/flutter-community/parsing-complex-json-in-flutter-747c46655f51](https://medium.com/flutter-community/parsing-complex-json-in-flutter-747c46655f51)

