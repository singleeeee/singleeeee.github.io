---
title: npm安装依赖过程
categories: 工程化
---

# npm安装依赖过程

## 查找npm配置信息

首先检查根目录下的`.npmrc`文件，如果没有再检查全局的`.npmrc`文件，最后检查npm的默认配置信息。

## 构建依赖树

- 首先查找`package-lock.json`文件，如果有就直接使用它的依赖树
- 如果没有则递归查找每个依赖需要的依赖
- 2.x 是全部循环遍历递归下载所有依赖，可能会出现**node_modules 黑洞**现象
-  3.x 是进行扁平化处理，碰到新的依赖会尝试放到第一层，如果后续碰到版本不一的依赖，则下载到相应的依赖下，问题是：`不同的依赖遍历顺序会出现不一样的依赖安装结果`
- 5.x **新增了 `package-lock.json` 锁文件**

## 根据依赖树到缓存或者远程仓库去下载依赖资源

过程中会有完整性校验

## 生成 packge-lock.json 文件



# npm的缓存机制

## 在用户本地会有缓存

- `index-v5`
- `content-v2`

>  **content-v2 文件是用来存在缓存包的具体内容，index-v5 是用来存储依赖包的索引，根据 index-v5 中的索引去 content-v2 中查找具体的源文件 。**

​         npm 在安装依赖的过程中，会根据 `lock`文件中具体的包信息，用 `pacote:range-manifest:{url}:{integrity}` 规则生成出唯一的 key，然后用这个 key 通过 SHA256 加密算法得到一个 hash。这个 hash 对应的就是 `_cache/index-v5` 中的文件路径。



- 用 `pacote:range-manifest:{url}:{integrity}` 规则生成Key，通过 SHA256 加密算法得到一个 hash
- 该hash用于缓存文件的索引文件，即index-v5下面，我们根据哈希值的前 4 位是来区分路径

如 `0020`表示00文件夹下的20文件

![image-20241211105101857](https://raw.githubusercontent.com/singleeeee/imgStorage/main/img/202412111051981.png)

- 该文件下的依赖会有一个 `_shasum`属性，充当了源文件包索引的作用
- 根据相同的查找机制，可以拿到依赖的压缩包，将它解压后就是我们需要的依赖源文件

# npm如何进行完整性校验

每个版本资源包都会有一个固定的hash值，存储在依赖信息的`shasum`字段

在下载完成之后在本地用相同的算法再次计算得出一个 `shasum`，对比两个 `shasum`：如果相同，就代表下载的依赖是完整的；反之，则说明下载过程中出现了问题，会再次重新下载。



# 开发中的最佳实践

以下建议均为个人在开发中根据所遇到的问题而产生的最佳实践，大家可以参考下。

- 不同的 npm 版本的安装流程会有所不同，我们尽量使用 npm v5.7 以上版本（因为 `package-lock.json` 生成逻辑 在 5.0 - 5.6 版本中有过几次更新，npm 5.6 以上才逐渐稳定)。
- 建议在项目中使用 `package-lock.json` 文件，并将其提交到远端仓库，从而提升依赖安装时间以及安装包的稳定性。
- 创建项目的时候使用 `npm install` 安装依赖，在提交代码的时候将 `package.json`、`package-lock.json` 提交到远端仓库，同时 ignore掉 `node_modules`文件。
- 其他成员在 `clone` 过代码之后执行 `npm install` 安装依赖，npm 会根据 `package.json` 和 `package-lock.json` 文件一起确定 `node_modules` 文件。
- 升级依赖使用 `npm` 命令安装\更新\删除依赖，此时 `package-lock.json` 文件会一同修改。然后将 `package.json` 文件和 `package-lock.json` 文件一起提交到远端仓库。
- 在远端的 `package.json` 文件和 `package-lock.json` 文件更新后，其他成员应该拉取最新的代码并执行 `npm install` 更新依赖。
- 如果 `package-lock.json` 文件冲突，应该先手动解决 `package.json` 的冲突，然后执行 `npm install --package-lock-only`，让 npm 自动帮你解冲突。可以参考 [官方推荐做法](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.npmjs.com%2Fcli%2Fv6%2Fconfiguring-npm%2Fpackage-locks%23resolving-lockfile-conflicts)。