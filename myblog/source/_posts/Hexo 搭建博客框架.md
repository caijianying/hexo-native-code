---
title: Hexo 搭建博客框架
categories:
  - 博客
  - 博客搭建
tags: 博客
abbrlink: eb92c116
date: 2021-02-28 22:52:21
---
[官网](https://hexo.io/zh-cn/docs/)
1.安装nodeJs，查看版本`node --version`，Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本
2.安装git
3.安装Hexo

安装Hexo1.安装命令
```java
npm install hexo -g
hexo init blog
cd blog
npm install
hexo s
```
这时博客就在本地生成了。访问`http://localhost:4000` 可以看效果。<br />可以说Hexo是很强大的，默认主题网站结构合理，适配手机，搜索栏（google）也有了。只需要优化（改一下失效的链接，添加评论，RSS等模块）就行了。<br />2.基本操作

- `hexo g` 生成/public 文件夹，里面是网站
- `hexo d` 把这个网站文件夹推送到服务器
- `hexo clean` 删除网站文件夹
- `hexo s` 本地查看效果

3.[Hexo官网一键部署](https://hexo.io/docs/one-command-deployment)<br />找到`_config.yml`文件编辑：
```java
deploy:
  type: git
  repo: <repository url> # https://bitbucket.org/JohnSmith/johnsmith.bitbucket.io
  branch: [branch]
  message: [message]
```
使用`hexo deploy`部署
