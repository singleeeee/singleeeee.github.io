---
title: webpack入门
categories: 工程化
---

# webpack入门

## 前言

​	如果你想要编写一个hello world 的网页，那很简单，只需要创建一个文本文件，写下html的内容就能在网页打开，如果你还想给`hello world`加上颜色，那只需要写一个css文件即可，如果你还想加一个弹窗提示，引入一个js就能解决。但是开发一个完整的项目往往还需要

- css 预处理器，scss / less / tailwindcss 等等
- 框架， vue / react 
- 兼容性处理, babel
- 热更新
- 代码合并、代码压缩
- ....

如果说这些都需要导入相关的库去解决，那有没有更好的办法去统一管理和配置他们呢？webpack就能解决这些问题，接下来让我们简单用一下webpack吧

## 编写文件

现在我们想实现一个简单的网页如下，包括html、css 和 js：

![image-20241210211021302](C:/Users/%E9%98%AE%E5%BF%97%E8%8D%A3/AppData/Roaming/Typora/typora-user-images/image-20241210211021302.png)

## 代码合并打包

这时候我们又在index.js里面引入了其它的js文件，在浏览器打开的时候，它会并发加载这些js文件。大量的网络io操作会导致页面不流畅，这时候我们需要webpack来将js文件合并。



## 热更新（开发服务器）

当我们打开webpack为我们处理后的index.js文件，发现

## 兼容性处理