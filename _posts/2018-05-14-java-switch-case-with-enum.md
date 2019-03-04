---
layout: post
title:  enum和switch case结合使用
date:   2018-05-14 15:02:29 +0800
categories: Java
tag: [switch-case, enum]
---

* content
{:toc}



### enum和switch case结合使用

在将enum和switch case结合使用的过程中，遇到了这个错误：“An enum switch case label must be the unqualified name of an enumeration constant”，代码如下所示：

```javascript
public enum EnumType {
    type1("type1"), type2("type2"), type3("type3");
    private String type;

    EnumType(String type) {
        this.type = type;
    }

    public String getType() {
        return type;
    }
}

@OnClick(R.id.btn_test_enum_with_switchcase)
public void onViewEnumWithSwitchCaseClicked() {
    EnumType enumType = EnumType.type1;
    testEnum(enumType);
}

private void testEnum(EnumType type) {
    switch (type) {
        case EnumType.type1:
            Log.e("type1:", type.getType());
            break;
        case EnumType.type2:
            Log.e("type2:", type.getType());
            break;
        case EnumType.type3:
            Log.e("type3:", type.getType());
            break;
        default:
            break;
    }
}
```

错误提示如下所示：An enum switch case label must be the unqualified name of an enumeration constant
![这里写图片描述](https://img-blog.csdn.net/20180514150029774?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

根据错误提示的意思，枚举类型和switch case一起使用时一定不要限定枚举常量值的常量，也就是它的类型。
对代码做下修改：

```javascript
private void testEnum(EnumType type) {
    switch (type) {
        case type1:
            Log.e("type1:", type.getType());
            break;
        case type2:
            Log.e("type2:", type.getType());
            break;
        case type3:
            Log.e("type3:", type.getType());
            break;
        default:
            break;
    }
}
```
好了，修改完成。

