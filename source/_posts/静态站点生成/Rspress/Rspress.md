---
title: Rspress
categories: 静态站点生成
---

# 介绍

[`Rspress` ](https://rspress.dev/zh/index)是一个由字节跳动团队开发的基于 [Rsbuild](https://rsbuild.dev/) 的`静态站点生成器`，基于 React 框架进行渲染，内置了一套默认的文档主题，你可以通过 Rspress 来快速搭建一个文档站点，同时也可以自定义主题，来满足你的个性化静态站需求，比如博客站、产品主页等。当然，你也可以接入官方提供的相应插件来方便地搭建组件库文档。

对于这类静态站点生成器，常见的还有：

- [VuePress](https://vuepress.vuejs.org/zh/)

**Vue 驱动的静态网站生成器**

- [VitePress](https://vitepress.qzxdp.cn/)

**Vite 和 Vue 强力驱动的静态网站生成器,  VitePress 甚至比 VuePress 要更轻更快，但它在灵活性和可配置性上作出了一些让步，比如它不支持插件系统。**

- [Hexo](https://hexo.io/zh-cn/)

**快速、简洁且高效的博客框架**

- [Hugo](https://www.gohugo.org/)

**Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。**

- [docsify](https://docsify.js.org/#/zh-cn/)

**docsify 可以快速帮你生成文档网站。不同于 GitBook、Hexo 的地方是它不会生成静态的 `.html` 文件，所有转换工作都是在运行时, 所以对SEO不太友好，更合适用于公司内部的技术文档等。可直接[部署在 GitHub Pages](https://docsify.js.org/#/zh-cn/deploy)。**

- [Gitbook](https://www.gitbook.com/)

**Forget building your own custom docs platform. With GitBook you get beautiful documentation for your users, and a branch-based Git workflow for your team.**

## 静态站点生成器

**静态网站生成器**是一种工具软件。它能够在构建阶段读取诸如文本文件（常见的是 Markdown 格式）、模板文件（如 HTML 模板）以及其他资源文件（像 CSS 样式表、JavaScript 脚本）等内容，然后按照特定的规则和逻辑，将这些内容组合、转换并生成一系列静态的 HTML、CSS 和 JavaScript 文件。简单来说，你可以不用编写过多代码，只要有资源文件，就可以基于这套工具快速的生产出一套网站，常见的产品有各种框架和工具的官方文档。



# 快速开始

## 初始化项目

```shell
npm create rspress@latest
```

## 启动 Dev Server

```shell
npm run dev
```

> 对于 dev 命令，你可以通过 `--port` 或 `--host` 参数来指定开发服务的端口号或 host，例如 `rspress dev --port 8080 --host 0.0.0.0`。方便起见，可以直接加入到 package.json  的script命令中

## 构建打包

```shel
npm run preview
```



默认情况下，Rspress 将会把产物打包到 `doc_build` 目录。

## 预览线上效果

``` shell
npm run preview
```



# 具体使用

​	上面介绍了快速启动部署一套默认配置的Rspress网站，接下来我们讲讲如何定制化修改网站内容。

## 路由系统

### 约定式路由

Rspress 使用的是文件系统路由，页面的文件路径会简单的映射为路由路径，这样会让整个项目的路由非常直观。

例如，如果在 `docs` 目录中有一个名为 `foo.md` 的文件，则该文件的路由路径将是 `/foo`。

Rspress 会自动扫描根目录和所有子目录，并将文件路径映射到路由路径。



- `rspress.config.js`中不能配置 `nav` 或者 `sidebar`，不然`_meta.json`的配置会不生效

## 导航栏与侧边栏

## 自定义编写组件

## 主题

- 配置了`locales`配置项后，文本文案要在相应的语言文件下单独配置

## 插件系统

### 官方插件

- @rspress/plugin-last-updated

**在文章中显示最后更新时间，当你在默认主题中配置了 `lastUpdated: true` 时，该插件会自动生效，不需要你去安装和注册插件。**

## 国际化



