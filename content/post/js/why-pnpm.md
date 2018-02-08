---
title: "为什么我们应该使用 pnpm（译）"
date: 2018-02-08T15:30:00+08:00
lastmod: 2018-02-08T15:30:00+08:00
keywords: ["pnpm", "pnpm", "node.js"]
description: "pnpm 是又一个 Node.js 的包管理工具。它可以替换 npm，而且 npm 更快更高效。"
tags: ["npm"]
categories: ["npm"]
author: "lxzxl"
---

[pnpm](https://github.com/pnpm/pnpm) 是又一个 Node.js 包管理工具。它可以替换 npm， 而且 npm 更快更高效。

<!--more-->

能有多快? _3 倍！_ 可以在[这里](https://github.com/pnpm/node-package-manager-benchmark)查看 benchmarks 。

为什么更高效？ 当你安装一个软件包，我们把它保存在你的机器上的一个全局存储目录中，然后我们创建一个硬链接而不是复制。 对于模块的每个版本，只会有一个副本保存在磁盘上。 例如，当使用 npm 或 yarn 时，如果有 100 个使用 lodash 的项目，你的磁盘上就会有有 100 份 lodash 的拷贝。pnpm 能帮助您节省千兆字节的磁盘空间！

## 为什么不用 **`Yarn`**

老实说，当 `Yarn` 开始公开时，我真的很失望。 几个月来我一直在为 `pnpm` 作出重大贡献，而且没有任何关于 Yarn 的消息。 有关其发展的信息是不公开的。

几天之后，我意识到 Yarn 只是 npm 的一个小小的改进。 尽管它使得安装速度更快，并且具有一些不错的新功能，但是它使用了与 npm 相同的扁平 node_modules 结构（从版本 3 开始）。

扁平化的依赖关系树带来了一系列的问题：

1. 模块可以访问他们不依赖的软件包
2. 扁平化依赖树的算法相当复杂
3. 一些软件包必须复制在一个项目的 node_modules 文件夹中

此外，还有一些 Yarn 不打算解决的问题，如磁盘空间使用问题。 所以我决定继续把时间花在 pnpm 上，并取得了巨大的成功。 截至目前（2017 年 3 月），pnpm 具有 Yarn 超过 npm 的所有附加功能：

1. ** 安全 **。 像 Yarn 一样，pnpm 有一个特殊的文件，其中包含所有安装包的校验码，以在代码执行之前验证每个已安装包的完整性。
2. ** 离线模式 **。 pnpm 将所有下载的软件包 tar 包保存在本地镜像仓库中。 当一个包在本地可用时，它从不发出请求。 使用 --offline 参数，HTTP 请求可以被完全禁止。
3. ** 速度 **。 pnpm 不仅比 npm 快，而且比 Yarn 还要快。 它比 cold 和 hot 缓存 Yarn 都快。 Yarn 从缓存拷贝文件，而 pnpm 只是从全局存储目录链接它们。

## 这怎么可能？

正如我前面提到的，pnpm 不会拍平依赖关系树。 因此，pnpm 使用的算法可以更容易！ 这就是为什么只有一名开发人员能够跟上 Yarn 的几十位贡献者的步伐。

那么，如果不依靠拍平的方式，pnpm 如何构造 node_modules 目录呢？ 为了理解它，我们应该回忆在 npm 版本 3 之前 node_modules 文件夹看起来是怎样的。在 npm@3 之前，node_modules 结构是可预测和干净的，因为 node_modules 中的每个依赖项都有自己的 node_modules 以及所包含依赖关系的 package.json 文件。

```shell
node_modules
└─ foo
   ├─ index.js
   ├─ package.json
   └─ node_modules
      └─ bar
         ├─ index.js
         └─ package.json
```

这种方法有两个严重的问题：

* 频繁使用的代码包创建了太深的依赖关系树，导致 Windows 上很长的目录路径问题
* 当被不同的依赖关系需要时，代码包会被复制粘贴多次

为了解决这些问题，npm 重新思考了 node_modules 结构，并提出了扁平化。 在 npm@3 中，node_modules 的结构现在看起来像这样：

```Shell
node_modules
├─ foo
|  ├─ index.js
|  └─ package.json
└─ bar
   ├─ index.js
   └─ package.json
```

有关 npm v3 依赖关系解析的更多信息，请参见 [npm v3 依赖关系解析](https://docs.npmjs.com/how-npm-works/npm3)。

与 npm@3 不同，pnpm 试图解决 npm@2 所具有的问题，而不是将依赖关系树展平。 在由 pnpm 创建的 node_modules 文件夹中，所有的软件包都有自己的依赖关系，但是目录树永远不会像 npm@2 那么深。 pnpm 保持所有依赖关系平坦，但使用符号链接将它们组合在一起。

```shell
-> - a symlink (or junction on Windows)

node_modules
├─ foo -> .registry.npmjs.org/foo/1.0.0/node_modules/foo
└─ .registry.npmjs.org
   ├─ foo/1.0.0/node_modules
   |  ├─ bar -> ../../bar/2.0.0/node_modules/bar
   |  └─ foo
   |     ├─ index.js
   |     └─ package.json
   └─ bar/2.0.0/node_modules
      └─ bar
         ├─ index.js
         └─ package.json
```

要查看实例，请访问 [示例 pnpm 项目](https://github.com/pnpm/sample-project)。

虽然这个例子对于一个小项目似乎太复杂了，但是对于更大的项目来说，结构看起来比 npm / yarn 创建的结构要好。 让我们看看为什么它不错。

首先，您可能已经注意到，node_modules 根目录下的包只是一个符号链接。 这很好，因为 Node.js 忽略符号链接并执行实际路径。 所以 require（'foo'）会执行文件 `node_modules/.registry.npmjs.org/foo/1.0.0/node_modules/foo/index.js` 而不是 `node_modules/foo/index.js`。

其次，没有一个安装的软件包在其目录中有自己的 node_modules 文件夹。 那么 `foo` 怎么引入 `bar`？ 让我们看看包含 `foo` 包的文件夹：

```shell
node_modules/.registry.npmjs.org/foo/1.0.0/node_modules
├─ bar -> ../../bar/2.0.0/node_modules/bar
└─ foo
   ├─ index.js
   └─ package.json
```

如您所看到的

1. `foo` 的依赖关系（只有 `bar`）被安装，但在目录结构中的一级。
2. 这两个软件包都位于名为 node_modules 的文件夹内。

`foo` 可以引入 `bar`，因为 Node.js 在目录结构中查找模块直到磁盘的根目录。 而且 `foo` 也可以引入 `foo`，因为它在一个名为 node_modules 的文件夹中（没错，这正是一些软件包所做的）。

## 你相信吗？

只需通过 npm 安装 pnpm：npm install -g pnpm。 只要你想安装一些东西，就用它来代替 npm：pnpm i foo。

您也可以在 [pnpm GitHub repo](https://github.com/pnpm/pnpm) 或 [pnpm.js.org](https://pnpm.js.org/) 中阅读更多信息。 您可以关注 [pnpm on Twitter](https://twitter.com/pnpmjs)，或者在 [pnpm Gitter Chat Room](https://gitter.im/pnpm/pnpm) 寻求帮助。

---

原文：[Why should we use pnpm?](https://www.kochan.io/nodejs/why-should-we-use-pnpm.html)
