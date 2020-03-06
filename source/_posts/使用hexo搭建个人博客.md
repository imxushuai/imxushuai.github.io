---
title: 使用hexo搭建个人博客
tags: 
  - hexo
  - blog 
  - 个人博客
categories: 其他
description: 使用hexo搭建炫酷的个人博客
photos:
  - >-
    https://imxushuai-01.coding.net/p/pic/d/pic/git/raw/master/006ifTg0gy1fxq63qyv3xj30yx0a7t99.jpg
abbrlink: 43303
date: 2018-11-25 17:22:21
---

<center><i>使用hexo搭建炫酷的个人博客</i></center>

![](https://imxushuai-01.coding.net/p/pic/d/pic/git/raw/master/006ifTg0gy1fxq63qyv3xj30yx0a7t99.jpg)

<!-- more -->

<center><font size="6px">Hexo搭建个人博客</font></center>

---
### Hexo介绍
hexo是一款基于Node.js的静态博客框架。   

[hexo中文网](https://hexo.io/zh-cn/index.html)

### 搭建Hexo博客环境
> 系统：centos 7   

1. 安装 Hexo 需要的环境
由于Hexo是基于`Node.js`的，所以NodeJS环境肯定是必须的。

```shell
yum install nodejs -y
```
也可以参考：[node.js安装教程](http://www.runoob.com/nodejs/nodejs-install-setup.html)

2. 安装 git
```shell
yum install git -y
# 设置github的账户信息
git config --global user.email "你的github邮箱"
git config --global user.name  "自定义名称"
```
3. 创建github仓库
> 我这里采用github作为博客文章的仓库，如果还没有github，请自行注册并创建仓库。

仓库名称为`custom.github.io`，其中`custom`为自定义部分，如：`mayun.github.io`

4. 安装 Hexo
```shell
npm install hexo-cli -g

```
5. 初始化 Hexo
```shell
## custom以你创建的仓库名为准
hexo init custom.github.io
```
> 此时在当前目录会出现`custom.github.io`文件夹

### 配置 Hexo
1. 更换一个好看的主题
```shell
## 进入customer.github.io文件夹
cd custom.github.io
## 使用git下载一款喜欢的主题,我这里下载的是 next 主题
git clone https://github.com/iissnan/hexo-theme-next.git themes/next
```
[附: next主题github](https://github.com/iissnan/hexo-theme-next)

2. 博客配置
```shell
## 修改custom.github.io中的_config.yml文件
vim _config.yml
```
> ##### 主要需要修改的配置：   
```text
title: title #网站名称   
Subtitle: Subtitle #二级标题   
description: description #网页描述   
author: author #作者   
language: zh-Hans #不同的主题，这里的值可能不一样   
timezone: Asia/Shanghai #时区   
url: http://xxxxx  #可以是ip或域名 
theme: next #修改主题   
port: 4000 #若没有，可自行添加该配置项
server_ip: localhost #若没有，可自行添加该配置项
deploy:
  type: git
  repo: https://github.com/xs1031893936/xushuai.github.io.git #你的github仓库地址
  branch: master
```
[其他配置项参考](https://hexo.io/zh-cn/docs/configuration.html)
> **注意的坑**：   配置文件为yml格式, 所以每个配置项 `:`必须有一个空格,配置文件中还有很多地方配置的值为`:year`这种形式,此`:`后不能有空格

3. 编写测试博客
```shell
## 进入`customer.github.io/source/_post/`目录中新建文件
touch test-blog.md
## 编辑，若没有vim，使用vi编辑器
vim test-blog.md
## 内容---------------------------------
---
title: test-blog
---

# hello hexo!
## 内容---------------------------------
```

4. 运行hexo
```shell
hexo server
```
> 若未正常运行，注意查看报错

5. 访问博客

6. 安装自动部署发布工具
```shell
npm install hexo-deployer-git --save
```

7. 发布到github
```shell
hexo clean && hexo g && hexo d
```
> 提交需要输入github账号和密码

基本上hexo博客的搭建已经完成。

[附:hexo主题自定义](https://juejin.im/entry/59d2da336fb9a00a5015e16b)
