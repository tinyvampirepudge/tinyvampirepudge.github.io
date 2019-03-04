---
layout: post
title:  如何通过Android Studio查看不同版本的Android源码？
date:   2018-03-23 17:58:43 +0800
categories: Android
tag: [Android源码]
---

* content
{:toc}



# 如何通过Android Studio查看不同版本的Android源码？
1、如何通过Android Studio查看不同版本的Android源码？
①通过Android SDK将你需要版本的源码下载下来。如果不会请自行google
![这里写图片描述](https://img-blog.csdn.net/2018032317550133?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

②新建一个项目，修改build.gradle中的文件。
![这里写图片描述](https://img-blog.csdn.net/20180323175514139?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
解释说明：

1处我给为自己想看的版本。这里我改成了21。

2处需要修改v7支持包的版本，具体的版本号可以去这里查看，[https://developer.android.com/topic/libraries/support-library/rev-archive.html](https://developer.android.com/topic/libraries/support-library/rev-archive.html)

3处的代码是我注释掉了，因为编译的时候有冲突

还需要注意的地方有：高版本特有的属性，比方说roundIcon，直接删除即可。

③sync下，接下来就可以查看21的api了。
![这里写图片描述](https://img-blog.csdn.net/20180323175553818?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2Mjg3NDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

