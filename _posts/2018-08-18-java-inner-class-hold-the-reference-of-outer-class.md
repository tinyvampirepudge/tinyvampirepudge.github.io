---
layout: post
title:  普通内部类持有外部类引用的原理
date:   2018-08-18 15:54:21 +0800
categories: Java
tag: [普通内部类持有外部类引用的原理]
---

* content
{:toc}



### 普通内部类持有外部类引用的原理
内部类虽然和外部类写在同一个文件中， 但是编译完成后， 还是生成各自的class文件，内部类通过this访问外部类的成员。
		
1、编译器自动为内部类添加一个成员变量， 这个成员变量的类型和外部类的类型相同， 这个成员变量就是指向外部类对象(this)的引用；

2、编译器自动为内部类的构造方法添加一个参数， 参数的类型是外部类的类型， 在构造方法内部使用这个参数为内部类中添加的成员变量赋值；

3、在调用内部类的构造函数初始化内部类对象时，会默认传入外部类的引用。

示例演示：
1、先定义一个包含普通类的内部类

```javascript
public class Parcel1 {
	
	class Contents{
		private int i = 11;
		public int value() {
			return i;
		}
	}
	
	class Destination{
		private String label;
		public Destination(String whereTo) {
			// TODO Auto-generated constructor stub
			label = whereTo;
		}
		String readLabel(){
			return label;
		}
		
	}
	
	public void ship(String dest) {
		Contents c = new Contents();
		Destination d = new Destination(dest);
		System.out.println(d.readLabel());
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Parcel1 p = new Parcel1();
		p.ship("Tasmania");
	}

}
```

2、编译运行下，获取到.class文件

![生成的.class文件位置](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/screenshot-1.png)

3、查看.class文件
给Android Studio安装插件[Java Bytecode Editor]( https://blog.csdn.net/kwame211/article/details/77677662), 使用插件查看这生成的.class文件，结果如下：
![Parcen1.calss](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Parcel1.png)
![Parcel1_Contents.class](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Parcel1%24Contents.png)

![Parcel1_Destination.class](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Parcel1%24Destination.png)

如上所示，箭头所指的地方，说明普通内部类在编译之后，编译器会通过构造器传递外部类的引用进去。

参考：

[https://www.jianshu.com/p/7c3ad6e6a5b1](https://www.jianshu.com/p/7c3ad6e6a5b1)

[https://blog.csdn.net/kwame211/article/details/77677662](https://blog.csdn.net/kwame211/article/details/77677662)

