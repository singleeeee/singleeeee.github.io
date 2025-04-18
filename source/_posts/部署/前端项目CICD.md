---
title: 基于Github Actions 和 docker 的前端项目CI/CD
category: 部署
---

# 基于Github Actions 和 docker 的前端项目CI/CD

## 项目背景

​	最近在写一个管理系统项目的时候出现了一些比较麻烦的问题，由于进度比较赶，项目虽然上线了，但是还需要继续开发，这就涉及到频繁的构建和部署。

​	首先讲一下没使用CI/CD 之前的部署方式:

1. 首先使用`pnpm build`，拿到项目的打包产物`.output`文件夹
2. 将`.output`文件夹压缩，并通过服务器可视化工具宝塔，将压缩包上传到指定文件夹下
3.  第一次部署时使用pm2命令`pm2 start ./.output/server/index.mjs -name 'xxx'`启动项目

4. 以后的部署都直接使用 `pm2 restart [name]` 即可

5. 后面重构的时候了解到pm2还有文件监听功能, 启动命令就换成了`pm2 start ./.output/server/index.mjs -name 'xxx' --watch`, 这样就不用每次都打开终端重启pm2 进程了

​	但是这种部署方式实在繁琐，我需要通过：1. 手动打包项目， 2. 将打包产物压缩， 3. 通过宝塔或者其它ssh工具，连接到我的服务器上，4. 解压打包产物 （5. 重启pm2进程） （6. 将代码提交到github) ， 如果是一两天更新一次线上环境还好，但是由于需要继续开发，这个步骤就需要频繁的执行，如果能去掉这些机械化的操作能够节省出大量的时间。

## 具体细节

技术方案： 基于`Github Actions` + `docker` + `Watch Tower` 的CI/CD 方案

实现效果： 本地提交代码以后，线上环境在等待一定时间后自动更新

工作原理： 本地提交代码`git push` 触发github仓库工作流的hook，执行相应的工作流；在工作流中，我们通过`dockerFile`构建我们的项目，并推送到DocketHub仓库；在我们的服务器上已经安装好的`WtachTower`工具会定时轮询DockerHub，发现镜像改动以后，会自动拉取到本地，并平滑停止并重启设置的容器，这时通过Docker容器部署的线上环境就会在重启后同步更新。



## 实现步骤

### 注册一个dockerhub账号

前往[dockerHub](https://hub.docker.com/)注册一个账号

### 在部署服务器上下载docker

``` bash
sudo apt-get update
sudo apt-get install -y docker.io
```

下载完docker以后，还需要连接到自己的dockerHub仓库，但是由于dockerHub是国外的网站，需要自己额外配置代理，才能够登上dockerHub，如果发现还有问题可以考虑一下使用其它镜像源，读者可自行查阅相关资料。

### 编写dockerfile文件和ecosystem.config.js

我的项目是**nuxt项目**，如果读者的项目是其它框架的，按需更换打包和启动命令配置即可。

``` dockerfile
# 使用 Node.js 18 的 Alpine Linux 版本作为基础镜像，体积小且适合生产环境
FROM node:18-alpine AS builder

# 设置工作目录为 /app
WORKDIR /app

# 全局安装 pnpm 包管理工具
RUN npm install -g pnpm

# 将 package.json 和 pnpm-lock.yaml 文件复制到工作目录中
COPY package.json pnpm-lock.yaml ./

# 使用 pnpm 安装项目依赖，并确保使用的是锁定的依赖版本
RUN pnpm install --frozen-lockfile

# 将项目的所有文件复制到工作目录中
COPY . .

# 执行项目的构建过程（通常是针对 Nuxt.js 等框架生成静态文件或打包项目）
RUN pnpm build

# 第二阶段，使用 Node.js 18 的 Alpine Linux 版本作为基础镜像，适合生产环境
FROM node:18-alpine

# 设置工作目录为 /app
WORKDIR /app

# 全局安装 PM2，用于管理和运行 Node.js 应用程序
RUN npm install -g pm2

# 将第一阶段构建的 .output 目录（Nuxt.js 编译结果）复制到当前容器中
COPY --from=builder /app/.output ./.output

# 将第一阶段的 ecosystem.config.js（PM2 配置文件）复制到当前容器中
COPY --from=builder /app/ecosystem.config.js ./ecosystem.config.js

# 暴露 3000 端口，用于访问应用程序
EXPOSE 3000

# 使用 pm2-runtime 启动应用程序，根据 ecosystem.config.js 文件运行
CMD ["pm2-runtime", "start", "ecosystem.config.js"]
```

- ecosystem.config.js 文件是pm2的启动脚本文件

``` js
module.exports = {
  apps: [
    {
      name: 'demo',
      script: './.output/server/index.mjs',
      args: 'start',
      instances: 'max',
      exec_mode: 'cluster',
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
      env: {
        NODE_ENV: 'production',
        HOST: '0.0.0.0',
        PORT: 3000,
      },
      error_file: './logs/err.log',
      out_file: './logs/out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss',
      merge_logs: true,
      time: true,
    },
  ],
}

```

### 编写github workflow文件

在项目的根目录中，创建`.giuthub/workflows/docker-image.yml` 

``` yaml
name: Docker Image CICD

on:
  push:
    branches:
      - test

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Build the Docker image
        run: |
          docker login -u ${{ secrets.DOCKER_HUB_USERNAME }} -p ${{ secrets.DOCKER_HUB_PASSWORD }}
          docker buildx create --use
          docker buildx build  . --push --tag [dockerhub用户名/镜像名称:tag名称]

```

- 在 GitHub 仓库的 `Settings > Secrets` 中添加 `DOCKER_USERNAME` 和 `DOCKER_PASSWORD`。

![image-20241030114107777](https://raw.githubusercontent.com/singleeeee/imgStorage/main/img/202410301141987.png)

### 将github actions构建的镜像拉下来启动

```shell
docker run -d --name your-container-name -p 3000:3000 your-dockerhub-username/your-image-name:yourTagName
```

这里的` -p 3000:3000 `前面端口对应的是你的容器内项目启动的端口，我们上面ecosystem配置了3000端口，后面的3000端口是映射到你服务器上的端口，部署成功后用户可通过后面的端口访问你的网站 【服务器IP:端口】。

- 请确保防火墙开放了这个端口
- 请确保这里的tagName和workflow推送的tag一致

### 运行watchTower镜像并指定项目容器

更多配置项，请查看[watchTower](https://containrrr.dev/watchtower/)

```shell
docker run -d \
  --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower your-container-name --interval 300 --cleanup 
```

这里需要注意`--interval`和`--cleanup`是放在最后的，位置错误会导致设置失败，而默认的时间间隔是一天，这就导致了我当时配置的时候觉得watchTower一直没有监听到我的镜像改变，后面通过查看日志才发现问题。这里时间间隔的单位是s（秒）

- 频繁的轮询会消耗更多的服务器资源，建议根据更新频次，相应设置轮询时间
- 如果想要监听多个容器直接在your-container-name 后添加即可
- watchTower还支持邮件报警，有需要的读者可自行查阅资料
- watchTower每次更新默认不删除上一次的镜像，如果不加上`--cleanup `会占用大量的磁盘空间



## 局限性

该方案更推荐在**测试环境**下使用，不推荐在**生产环境**使用，以下是它的一些局限性：

### **网络依赖**：

如果使用dockerHub作为镜像仓库，会很依赖 Docker Hub 的网络稳定性， 如果 Docker Hub 出现故障或网络不佳，可能导致构建或拉取镜像失败。此外还需要进行额外网络代理的配置，如果代理服务器不稳定项目也会受到影响。

### 镜像体积

随着项目的开发，体积会不断增长，如果构建的镜像过大，可能导致上传和下载速度慢，影响部署效率。

### 资源管理

Watch Tower 会自动重启容器，但如果应用在高负载期间频繁更新，可能导致`服务不可用`的短暂时刻。尤其对于一些数据库的敏感操作，该缺陷会造成严重的后果，这也是不推荐生产环境使用的主要原因。

### 调试难度

容器化环境的调试可能比传统环境更复杂，特别是在出现问题时，定位问题可能需要额外的工具和步骤。



---

如果读者大佬有更好的方案或者建议，欢迎在评论区留言。笔者才疏学浅QAQ，还在学习成长中，后续会不断订正自己的文章内容。

