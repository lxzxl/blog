---
title: "npx命令介绍"
date: 2017-12-27T15:14:25+08:00
keywords: ["npm", "npx"]
description: "简单介绍npm新命令 - npx"
tags: ["javascript", "npm"]
categories: ["npm"]
author: "lxzxl"
---

# 什么是`npx`

第一次看到`npx`命令是在 babel 的文档里

> **Note:** If you do not have a `package.json`, create one before installing. This will ensure proper interaction with the `npx` command.

在自己的机器上试了下，真的有这个命令，于是去查了下 npm 官方信息，发现这个在 npm`v5.2.0`引入的一条命令（[链接](https://github.com/npm/npm/releases/tag/v5.2.0)）。引入这个命令的目的是为了提升开发者使用包内提供的命令行工具的体验。

# 为什么引入这个命令

举个例子，我们开发中要运行 parcel 命令来打包：`parcel index.html`，以前有这么几种方式：

1. 全局安装 parcel，但有时不同项目使用不同版本，不允许使用全局包，只能考虑下面一些方法

2. 使用 npm scripts，在 package.json 加一个 script

    ```json
    "scripts": {
        "start": "parcel index.html"
    }
    ```

3. 将 node_modules 的可执行目录加到 PATH 中，

    ```bash
    alias npmx=PATH=$(npm bin):$PATH
    npmx parcel index.html
    ```

4. 指定可执行命令路径

    ```bash
    ./node_modules/.bin/parcel index.html
    ```

现在我们有了 `npx` 命令，就不在需要考虑以上方法了（其实
 `npx` 是对方法 3 的封装）。当我们执行 `npx parcel index.html` 时，会自动去`./node_modules/.bin`目录下搜索。

`npx` 还允许我们单次执行命令而不需要安装，例如

```bash
npx create-react-app my-cool-new-app
```

这条命令会**临时**安装 create-react-app 包，命令完成后 create-react-app 会删掉，不会出现在 global 中。下次再执行，还是会重新临时安装。
