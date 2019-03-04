---
layout: post
title:  maven安装与zsh环境变量配置
date:   2018-09-18 09:46:58 +0800
categories: Maven
tag: [Maven]
---

* content
{:toc}




### maven安装与zsh环境变量配置

1、官网下载maven文件
[maven download](https://maven.apache.org/download.cgi)

![下载这两个文件中的任意一个即可](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20180918-092916.png)

按照安装提示进行操作，先解压，然后配置环境变量：
[Installing Apache Maven](https://maven.apache.org/install.html)

![解压完后的文件目录](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20180918-093811%402x.png)

接下来配置环境变量:
① vi ~/.zshrc ,  然后进入编辑模式。
②定义环境变量M2_HOME，其值为刚才解压好的文件路径；接着将M2_HOME添加进PATH环境变量。如下所示：

```javascript
export M2_HOME=/Users/xxx/Documents/software/apache-maven-3.5.4
export PATH=${PATH}:${M2_HOME}/bin
```

③关闭并重启Terminal窗口，输入 mvn -v 命令查看maven环境变量是否配置成功，如果成功的话结果就如下所示：

![mvn -v 结果](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20180918-094447.png)
