---
title: hexo 常用命令
tags: 建站
categories: 个人博客搭建
---


## 快速开始

### 项目初始化

``` bash
$ hexo init "根目录名称"
```
> 新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。


More info: [Commands](https://hexo.io/zh-cn/docs/commands)

### 清除缓存

``` bash
$ hexo clean
```

> 清除缓存文件 (db.json) 和已生成的静态文件 (public)。

More info: [Clearing Cache](https://hexo.io/docs/clearing-cache.html)


### 生成静态文件

``` bash
$ hexo generate || hexo g 
```
> 生成静态文件。

More info: [Generating](https://hexo.io/docs/generating.html)




### 启动本地服务器

``` bash
$ hexo server || hexo s
```

> 启动服务器。默认情况下，访问网址为： http://localhost:4000/。

More info: [Server](https://hexo.io/docs/server.html)



### 部署

``` bash
$ hexo deploy || hexo d
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

### 新建一篇文章

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### 发布草稿

``` bash
$ hexo publish [layout] <filename>
```
  
> 发布草稿。

More info: [Publishing](https://hexo.io/docs/publishing.html)







### 显示草稿

``` bash
$ hexo draft
```

> 显示草稿。

More info: [Draft](https://hexo.io/docs/draft.html)

