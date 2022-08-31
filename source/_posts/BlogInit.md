---
title : 关于博客的创建
date : 2022-01-10 00:34:38
tag : [nodejs,hexo]
categories : 关于小站
description : 有关个人博客的创建记录 
---

# 博客

个人博客基于Hexo搭建，以下记录博客搭建过程的相关纪录

# 一、安装

## 1.摘要

考虑到github pages 服务器在国外，整体访问不稳定的问题。凑巧阿里云搞活动，白嫖了一台轻量级服务器，所以采用阿里云+宝塔界面配置的操作。

服务器：CentOS8

参考链接
📁[Hexo官方文档](https://hexo.io/zh-cn/docs/)

🌐[Hexo搭建](https://blog.csdn.net/sinat_37781304/article/details/82729029)

🏠[Fluid-Theme](https://hexo.fluid-dev.com/)

🔧[Fluid配置文档](https://hexo.fluid-dev.com/docs/guide/)

💬[Twiko评论系统文档](https://twikoo.js.org/#%E7%89%B9%E8%89%B2)

🐂 [七牛云图床设置教程](https://blog.csdn.net/qq_29086527/article/details/117688495)


## 2.安装步骤

- [ ] 1.安装git
	`sudo yum install git-core`
- [ ] 2.安装node.js
	直接在宝塔面板上配置，选择稳定高版本
	打开宝塔界面→网站→Node项目→按提示添加Node.JS
- [ ] 3.安装Hexo
	- 详细步骤
		前面git和nodejs安装好后，就可以安装hexo了，你可以先创建一个文件夹blog，然后cd到这个文件夹下（或者在这个文件夹下直接右键git bash打开）。
		输入命令
		```Bash
		npm install -g hexo-cli
		```
		
		依旧用`hexo -v`查看一下版本
		至此就全部安装完了。
		接下来初始化一下hexo
		```Bash
		hexo init myblog
		```
		
		这个myblog可以自己取什么名字都行，然后
		```Bash
		cd myblog //进入这个myblog文件夹
		npm install
		```
		
		新建完成后，指定文件夹目录下有：
- node_modules: 依赖包
- public：存放生成的页面
- scaffolds：生成文章的一些模板
- source：用来存放你的文章
- themes：主题
-  _config.yml: 博客的配置文件**
```Bash
hexo g
hexo server
```

打开hexo的服务，在浏览器输入localhost:4000就可以看到你生成的博客了。
大概长这样：
![](https://img-blog.csdnimg.cn/img_convert/84e7c09ee0395f0faac60fdc381cd526.png)
使用ctrl+c可以把服务关掉。
原文链接：[https://blog.csdn.net/sinat_37781304/article/details/82729029](https://blog.csdn.net/sinat_37781304/article/details/82729029)

## 3.基本配置
1. 打开宝塔界面→网页→Node项目→添加Node项目
	![](https://oneindex.yyin.top/images/2022/01/09/FgHmZt6LY0/image.png)
2. 如图修改项目目录、项目名称
3. 修改端口`hexo s -p [port]`
4. 开放端口,阿里云和宝塔面板的都打开对应的port
5. 填写端口和域名
6. 启动项修改
	![](https://oneindex.yyin.top/images/2022/01/09/XsfWIfYFqd/image_1.png)
7. 其他默认，提交


