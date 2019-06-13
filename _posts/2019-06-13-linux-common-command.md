---
layout: post
title:  linux下常用命令—个人总结
date:   2019-06-13 15:13:22 +0800
categories: linux
tag: [linux]
---

* content
{:toc}




### linux下常用命令—个人总结
##### 1、转换目录：cd
进入子目录:

```
cd xxx
```

返回上一级目录：

```
cd ..
```

进入平级的某个目录：

```
cd ../xxx
```

##### 2、创建文件夹：

```
mkdir xxx
```

##### 3、查看当前目录下的文件

查看当前目录下的所有文件及读写权限：

```
ls -al
```

##### 4、显示当前目录路径

```
pwd
```

##### 5、copy文件

```
cp 参数 源文件 目标文件
```
例：想把桌面的Natit.kext 拷贝到驱动目录中 

```
cp -R /User/用户名/Desktop/Natit.kext /System/Library/Extensions;
```
参数R表示对目录进行递归操作，kext在图形界面下看起来是个文件，实际上是个文件夹。

把驱动目录下的所有文件备份到桌面backup:

```
cp -R /System/Library/Extensions/* /User/用户名/Desktop/backup;
```

##### 6、删除文件及文件夹：

```
rm 参数 文件
```
例：想删除驱动的缓存

```
rm -rf /System/Library/Extensions.kextcache 
```

```
rm -rf /System/Library/Extensions.mkext
```
参数－rf表示递归和强制，执行了 `rm -rf /` 你的系统就瘫痪了；

##### 7、移动文件：

```
mv 文件
```
例：想把AppleHDA.Kext 移到桌面:

```
mv /System/Library/Extensions/AppleHDA.kext /User/用户名/Desktop
```

想把AppleHDA.Kext 移到备份目录中:

```
mv /System/Library/Extensions/AppleHDA.kext /System/Library/Extensions/backup
```

##### 8、创建文件：

```
touch abc.txt
```
修改文件内容：

```
vi abc.txt
```
或者使用sublime打开。

还可以使用 echo命令：

```
echo dandan > a.txt
```
将"dandan"写入a.txt文件。

##### 9、curl url
查看网页源码:

```
curl www.sina.com
```
显示头信息：

```
curl -i www.sina.com
```
显示通信过程：

```
curl -v www.sina.com
```
具体请看：[http://www.ruanyifeng.com/blog/2011/09/curl.html](http://www.ruanyifeng.com/blog/2011/09/curl.html)

##### 10、env
env命令用于显示系统中已存在的环境变量，以及在定义的环境中执行指令。该命令只使用"-"作为参数选项时，隐藏了选项"-i"的功能。
若没有设置任何选项和参数时，则直接显示当前的环境变量。

##### 11、将终端的输出写入到文件：覆盖操作

```
 xxx > file
```

例如：
```
tinytongtongdeMacBook-Pro% adb shell ps > log.txt
```

##### 12、拷贝文件中的内容到剪切板：

```
pbcopy < ~/.ssh/id_rsa.pub
```

也可以这样写：

```
cat acttop.txt | pbcopy
```

##### 13、将字符串数据写入剪切板：

```
echo xxx | pbcopy
```

实例
```
tinytongtongdeMacBook-Pro% echo maolegemi | pbcopy
```

##### 14、将数据写入文件：echo

```
echo dandan > a.txt
```
将"dandan"写入a.txt文件，覆盖操作。

##### 15、输出文件内容：cat

可以把一批命令文件串联后输出到标准输出上。cat 可用来在屏幕上打印文件。

```
tinytongtongdeMacBook-Pro% cat a.txt b.txt
dandan
heheda
tinytongtongdeMacBook-Pro%
```

##### 16、追加写入文件：

```
>>
```

示例：

```
tinytongtongdeMacBook-Pro% echo asdfasdf >> a.txt
tinytongtongdeMacBook-Pro% cat a.txt
dandan
asdfasdf
tinytongtongdeMacBook-Pro%
```

未完待续。


