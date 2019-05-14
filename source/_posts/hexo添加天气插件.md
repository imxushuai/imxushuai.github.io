---
title: hexo添加天气插件
photo:
  - 'https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxq63qyv3xj30yx0a7t99.jpg'
date: 2019-03-29 17:27:30
tags:
  - hexo
  - 天气插件
categories:
  - 其他
description: 为自己的hexo博客添加一个实时的天气插件吧
---

<center><i>为自己的hexo博客添加一个实时的天气插件吧</i></center>

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxq63qyv3xj30yx0a7t99.jpg)

<!-- more -->

<center><font size="6px">hexo添加天气显示</font></center>

---

网上有许多天气的插件，这里我选择的是心知天气，因为免费，而且外观也还比较不错。

[心知天气官网](https://www.seniverse.com)

### 生成插件代码

#### 注册心知天气账号

点击注册，填写相关信息即可注册。

> **注意：**
>
> ​	填写组织域名的时候，有域名就填写域名，没有域名就填写ip地址，填写域名时，最好和你博客使用的域名一致。

#### 获取天气插件代码

- 点击产品，进入weather Widget天气网页插件

  ![](https://images.xushuai.fun/20190329171023.png)

- 点击立即创建

  ![](https://images.xushuai.fun/20190329171359.png)

- 配置插件样式

  ![](https://images.xushuai.fun/20190329171755.png)

- 复制下方的安装代码

  ![](https://images.xushuai.fun/20190329171938.png)

### 使用插件代码

#### 修改layout.swig文件

> 这里我的主题为：next。所以路径是：`/themes/next/layout/_layout.swig`，将复制的插件代码放在`</body>`标签之前。

![](https://images.xushuai.fun/20190329172323.png)

配置好后，重启自己的hexo即可前往自己的页面查看效果。

### 通知

心知官方已经关闭了心知天气插件的提供，所以这篇文章废了，但是我不想删，就这样吧，如果想要天气插件，可以Google一下，使用其他的天气插件。
