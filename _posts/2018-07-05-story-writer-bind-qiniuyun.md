---
layout: post
title:  小书匠绑定七牛云图床
date:   2018-07-05 17:32:57 +0800
categories: 图床
tag: [小书匠, 七牛云]
---

* content
{:toc}



### 小书匠绑定七牛云图床

本人一直使用印象笔记，但是其并不是完全的支持导入导出md格式，所以就使用小书匠这款软件来进行md文档编写，并直接发布到印象笔记中。
下面说下小书匠如何关联七牛云存储，这里是用七牛云作为图床。这样做的话，写好的md文件中的图片都是以外链形式导入的，可以在任何地方打开使用。

![小书匠绑定七牛云](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/xiaoshujiang.png)

需要的信息如上图所示，这里挨个解释下如何填写：
①第一个自定义名称可随便填写。

②接下来的上传入口和下方的链接填写，请参考下图：
![上传入口](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/xiaoshujiang2.png)
如上图中的1所示，在七牛云中，我的存储空间是入口是华北，所以这里的上传入口我选择华北。
上传入口的域名，请查看上图中的2处，填写这个域名即可。需要注意的是，这里提示的格式是"http://xxx.xxx.xxx" ,我们要严格按照这个格式填写。
这里有个小技巧，可以找一个已经上传成功的图片，复制它的外链并在浏览器中打开，然后将外链中的域名复制出来即可。

③AccessKey和SecretKey的值，具体请参照参照七牛云平台中的 个人面板 --> 个人中心 --> 秘钥管理中的数据，如下所示：
![秘钥管理](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/xiaoshujiang3.png)

④空间名称(Bucket Name)的填写，在七牛云中选择一个你的空间名称即可，如下所示：
![空间名称](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/xiaoshujiang4.png)
这里我填写"tiny-vampire"即可。

⑤文件名称生成规则的填写：
这里可以按需更改，需要注意的是最好把汉字替换掉，换成英文即可。

⑥图片url前缀填写：请参照步骤②中的入口url的填写，唯一区别是这里在的格式是以"/"结尾。

好了，配置到了这里就告一段落了，如果对你有用的话记得点赞。

