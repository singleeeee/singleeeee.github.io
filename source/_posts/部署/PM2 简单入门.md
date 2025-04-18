---
title: PM2 简单入门
categories: 部署
---

# PM2 简单入门

## 介绍

PM2 是一个守护进程管理器，它将帮助您管理并保持您的应用程序。

简单来说有以下场景：

1. 当**进程崩溃或异常退出**时，默认情况下，PM2 会无限次重启崩溃的进程，除非你设置了重启限制。
2. 当**进程内存或 CPU 占用过高**时，你可以为进程配置内存或 CPU 使用的上限。当超过设定的阈值时，PM2 会自动重启该进程，以避免系统资源被消耗殆尽。
3. 当**服务器重启或宕机**时，PM2可以在服务器重启时自动启动并恢复之前运行的进程
4. 当**应用代码更新后**自动重启，可以使用 `watch` 选项让 PM2 自动监控文件变化，并在检测到代码更新时自动重启进程。
5. **日志管理**，PM2 提供了强大的日志管理功能，可以通过命令查看、清除日志，也可以将日志存储在文件中，方便日后分析。
6. **负载均衡**，PM2 提供了内置的负载均衡（cluster mode），可以在多核服务器上启动多个实例，并自动分配流量给不同的实例。
7. **持久性配置与状态恢复**，通过 `pm2 save` 保存进程列表，`pm2 resurrect` 恢复上一次保存的进程状态。

## 操作

### 启动

```shell	
pm2 start index.js
```

### 重启

`pm2 restart` 是简单的重启进程，无论进程是单实例还是多实例，都会直接杀掉旧的进程并启动新的进程。

``` shell
pm2 restart app_name
```

### 重新加载(平滑重启/热重启)

`pm2 reload` 是一种**无停机时间的重启方式**（特别是在 **cluster mode** 即集群模式下），它会逐个重启应用的实例，确保服务在整个过程中保持可用。

```shell
pm2 reload app_name
```

### 停止

```shell
pm2 stop app_name
```

### 删除

``` shell
pm2 delete app_name
```

除了`app_name` 还可以使用

- `all` 操作所有进程
- `id` 使用具体的进程id

### 查看所有进程状态

```shell
pm2 [list|ls|status]
```

![image-20241030151514009](C:/Users/%E9%98%AE%E5%BF%97%E8%8D%A3/AppData/Roaming/Typora/typora-user-images/image-20241030151514009.png)

### 查看日志

```shell
pm2 logs --lines 200
```

### 监控面板

```shell
pm2 monit
```

### 配置文件

创建 `ecosystem.config.js` 文件

```
pm2 ecosystem
```

默认配置如下：
``` js
module.exports = {
  apps : [{
    name: "app",
    script: "./app.js",
    env: {
      NODE_ENV: "development",
    },
    env_production: {
      NODE_ENV: "production",
    }
  }, {
     name: 'worker',
     script: 'worker.js'
  }]
}
```

然后可以轻松启动该文件

``` shell
pm2 start ecosystem.config.js
```

### 设置启动脚本

当我们的服务器启动或者重启等情况下，我们需要重新启动PM2。想解决这个问题，实现PM2自动重启，只需要使用下面的命令

```shel
pm2 startup
```

这个命令这会让我们的PM2重新启动，但是之前PM2管理的进程是不会恢复的，所以我们需要加上如下命令，给当前正在运行的进程列表保存到一个快照文件。

```shell
pm2 save
```

### 监听文件变化，重启程序

```shell
pm2 start env.js --watch --ignore-watch="node_modules"
```

这将监视并重新启动当前目录+所有子文件夹中的任何文件更改，并且它将忽略 node_modules 文件夹中的任何更改。

### 更新PM2

``` shell
npm install pm2@latest -g
```

然后更新内存中的 PM2 :

```shell
pm2 update
```

## 配置项

常用的一些选项

```shell
# 指定进程名称
--name <app_name>

# 监听文件，当文件变动时重启进程
--watch

# 设置内存阈值，在内存使用超过一定限度时自动重启
--max-memory-restart <200MB>

# 指定日志文件
--log <log_path>

# 传递额外的参数给脚本
-- arg1 arg2 arg3

# 自动重启之间的延迟
--restart-delay <delay in ms>

# 在日志前添加时间前缀
--time

# 禁用自动重启功能。如果应用崩溃或退出，PM2 不会尝试重新启动该应用
--no-autorestart

# 指定强制重启的定时任务
# 使用 Cron 表达式来指定强制重启应用的时间。例如：--cron "0 0 * * *" 会每天午夜 12 点强制重启应用。
--cron <cron_pattern>

# 附加到应用程序日志（前台运行）
# 以非守护进程模式运行应用，这意味着 PM2 会保持在前台并显示应用的日志信息。
--no-daemon
```
