---
title: "使用Hugo遇到的坑"
date: 2017-12-17T00:07:53+08:00
lastmod: 2017-12-17T00:38:53+08:00
weight: 20
keywords: ["hugo", "bugs"]
description: "使用Hugo遇到的坑搜集"
tags: ["hugo"]
categories: ["Other"]
author: "lxzxl"
---

上一篇介绍了 Hugo 入门，但实际使用中还是遇到不少坑，搜集在这边文章里帮助其他童鞋少走弯路。

# Hugo server 无法正确监测到文件变化

本来用家里的电脑写文章时还是正常的，每次改动都能自动检测到，会触发重新编译。但之后用公司电脑就不行了，会出现下面图中的错误：
`hugo server`竟然会监听到`.git`中的变化。即便我删掉我的 blog 目录重新 clone 下来，还是会有一样的问题。

![Hugo Server Error](/images/hugo-bugs-collection/hugo-server-watch.jpg)

去 Hugo 的 Github Issues 里能找到一样的问题[#3468](https://github.com/gohugoio/hugo/issues/3468)，但查看提交版本发现我的版本已经包含修复。

```bash
$ hugo version
Hugo Static Site Generator v0.30.2 darwin/amd64 BuildDate: 2017-12-13T17:35:33+08:00
```

实在没有办法，我发现 0.31 有一堆大改动，我就尝试更新到最新版本，试了一下发现问题没有了。

```bash
$ brew upgrade hugo
$ hugo version
Hugo Static Site Generator v0.31.1 darwin/amd64 BuildDate: 2017-12-20T13:32:10+08:00
```

---

# 嵌套列表中的代码块中的短横线识别错误

我需要在某个列表下显示代码块，我在其他 markdown 编辑器（Typora）中是能正常显示，但 hugo 渲染的不是我想要的：

````markdown
1. item1

    ```yaml
    sudo: false
    language: go
    git:
    depth: 1
    install: go get -v github.com/gohugoio/hugo
    scripts:
    - build
    - publish
    ```

2. item2
````

最后的效果见下图，代码块中的 YAML 数组，被错误的渲染成了 markdown 的列表，整个代码块都乱掉了。
![List in Code in List](/images/hugo-bugs-collection/list-code-list.jpg)

我去hugo提了issue[#4171](https://github.com/gohugoio/hugo/issues/4171)，但回复说不支持4个空格缩进和代码块同时使用 😭 。

经过多次试验，发现了一些workaround：

- 给代码块中的短横线前加缩进 - 4个空格。这个方法适合yaml，因为yaml对缩进的空格数量没有要求，对齐即可。这应该是最合适的方法。

    > 1. item1
    > 
    >     ```yaml
    >     sudo: false
    >     scripts:
    >         - build # 这里 - 前面有4个空格
    >         - publish
    >     ```
    > 
    > 2. item2

- 额外加一层缩进，这种方法甚至不需要在加```` ```yaml````标记，因为无法识别，这也是缺点。

    > 1. item1
    > 
    >         ```yaml 
    >         sudo: false # 前面一共有8个空格
    >         scripts:
    >           - build
    >           - publish
    >         ```
    > 
    > 2. item2


- 使用一个和短横线相似的符号暗度陈仓，但这不适合yaml代码，所以谨慎使用。[来源](https://github.com/gohugoio/hugo/issues/4171#issuecomment-352851263)

    > 1. item1
    > 
    >     ```yaml 
    >     sudo: false
    >     scripts:
    >       ‐ build # 这里的‐不是markdown中的-
    >       ‐ publish
    >     ```
    > 
    > 2. item2

---

（待搜集。。。）