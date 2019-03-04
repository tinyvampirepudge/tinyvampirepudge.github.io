---
layout: post
title:  Jenkins构建bug
date:   2018-08-30 16:06:43 +0800
categories: Jenkins
tag: [Jenkins构建]
---

* content
{:toc}



### Jenkins构建bug
构建时报下面这个错误：

```javascript
* What went wrong:
A problem occurred configuring project ':app'.
The SDK directory '/Users/xxx/Documents/develop/sdk' does not exist.
```

环境变量我都配置过了，sdk在这个目录下也有。后来经过排查发现还是文件目录的权限问题，使用"chmod 777 dir-name" 对sdk的文件权限一层一层修改吧，然后再次构建会发现构建成功的。
这个原因大概是因为Jenkins构建时调用这个sdk，sdk所在的目录需要游客访问权限吧。

参考：

[https://groups.google.com/forum/#!topic/jenkinsci-users/Y2lYvFrpqa0](https://groups.google.com/forum/#!topic/jenkinsci-users/Y2lYvFrpqa0)

