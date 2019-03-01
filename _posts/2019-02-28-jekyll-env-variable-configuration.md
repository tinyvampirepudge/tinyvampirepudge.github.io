---
layout: post
title:  jekyll 配置环境变量——zsh
date:   2019-02-28 23:22:29 +0800
categories: 个人主页搭建
tag: [jekyll]
---

* content
{:toc}



### jekyll 配置环境变量——zsh

在尝试使用github搭建个人主页的过程中，遇到了一些问题，这里记录下。

在安装ruby、gem之后，通过gem安装jekyll后，执行`jekyll -v`命令时遇到`zsh: command not found: jekyll`错误，很明显这个是环境变量的问题。

这里我的终端使用的zsh，不是mac自带的。

最后通过gem重新安装jekyll，然后根据终端中的提示来配置环境变量。终端输出如下：

```
bogon% sudo gem install --user-install bundler jekyll
Password:
Fetching bundler-2.0.1.gem
/usr/local/Cellar/ruby/2.6.0_1/lib/ruby/2.6.0/rubygems/installer.rb:704: warning: Insecure world writable dir /Users/tinytongtong/Documents in PATH, mode 040777
WARNING:  You don't have /Users/tinytongtong/.gem/ruby/2.6.0/bin in your PATH,
	  gem executables will not run.
Successfully installed bundler-2.0.1
Parsing documentation for bundler-2.0.1
Installing ri documentation for bundler-2.0.1
Done installing documentation for bundler after 3 seconds
Successfully installed jekyll-3.8.5
Parsing documentation for jekyll-3.8.5
Installing ri documentation for jekyll-3.8.5
Done installing documentation for jekyll after 1 seconds
2 gems installed
bogon%
```

提示中很明确的告诉我们，如果不把`/Users/tinytongtong/.gem/ruby/2.6.0/bin`这个路径添加进Path中，gem的可执行文件就不会执行，也就是我们新安装的jekyll不会执行，所以我们把这个路劲添加进Path中。

```
#jekyll
export PATH=${PATH}:/Users/tinytongtong/.gem/ruby/2.6.0/bin
```

接着重启zsh，输入`jekyll -v`，输出如下：

```
bogon% jekyll -v
jekyll 3.8.5
bogon%
```

说明一切OK了。

参考：
[如何快速搭建自己的github.io博客](https://blog.csdn.net/Walkerhau/article/details/77394659)
[安装jekyll——官网](https://jekyllrb.com/docs/installation/macos/)
