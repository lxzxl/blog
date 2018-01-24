---
title: "使用 Hugo 搭建博客"
date: 2017-12-17T00:07:53+08:00
lastmod: 2017-12-17T00:38:53+08:00
weight: 20
keywords: ["hugo", "pages", "github"]
description: "使用 Hugo 从零开始搭建 Github Pages Blog"
tags: ["hugo"]
categories: ["Other"]
author: "lxzxl"
---

以前写的一些文章笔记都托管在简书和 Segmenfault 上，但由于简书内容越来越垃圾，一直都打算转移到 github pages 上，但由于个人原因拖到现在。最近刚好有些时间，比较了一些静态博客生成工具，最后选择用 Hugo 来生成和管理自己的博客。

<!--more-->

# Hugo

[Hugo](https://gohugo.io/) 是由 Go 语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

---

## 安装

环境：macOS

```bash
brew install hugo

# 检查安装成功
hugo version # Hugo Static Site Generator v0.30.2 darwin/amd64 BuildDate: 2017-12-13T17:35:33+08:00
```

---

## 开始使用

### 生成 site 目录

```bash
hugo new site blog
cd blog
git init
#Congratulations! Your new Hugo site is created in /Users/steven/MyProjects/Demo/blog.

#Just a few more steps and you're ready to go:
#
#1. Download a theme into the same-named folder.
#   Choose a theme from https://themes.gohugo.io/, or
#   create your own with the "hugo new theme <THEMENAME>" command.
#2. Perhaps you want to add some content. You can add single files
#   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
#3. Start the built-in live server via "hugo server".
#
#Visit https://gohugo.io/ for quickstart guide and full documentation.

# 目录结构
tree blog
#blog
#├── archetypes
#│   └── default.md
#├── config.toml
#├── content
#├── data
#├── layouts
#├── static
#└── themes
```

`config.toml` 是配置文件，在里面可以定义博客地址、构建配置、标题、导航栏等等。

`themes` 是主题目录，可以去 [themes.gohugo.io](http://themes.gohugo.io/) 下载喜欢的主题。

`content` 是博客文章的目录。

### 安装主题

去 [themes.gohugo.io](http://themes.gohugo.io/) 选择喜欢的主题，下载到 themes 目录中，然后在 config.toml 中配置 `theme = "even"` 即可。其他配置可见 theme 说明

下面以我比较喜欢 [Even 主题](https://themes.gohugo.io/hugo-theme-even/) 举个例子

#### 1. 下载

* 可以直接 `clone` 到 `themes` 目录下，优点是如果对主题有调整需求可以同时提交到 git 控制中。

    ```bash
    git clone https://github.com/olOwOlo/hugo-theme-even themes/even
    ```

* 也可以添加到 git 的 submodule 中，优点是后面讲到用 travis 自动部署时比较方便。如果需要对主题做更改，最好 fork 主题再做改动。

    ```bash
    git submodule add https://github.com/olOwOlo/hugo-theme-even.git themes/even
    ```

#### 2. 使用

如果需要调整更改主题，需要在 themes/even 目录下重新 build

`cd themes/even && npm i && npm start`

### 第一篇文章

```bash
hugo new post/my-first-post.md
```

文章顶部可以设置一些 meta 信息，例如：

```markdown
---
title: "My First Post"
date: 2017-12-14T11:18:15+08:00
weight: 70
keywords: ["hugo"]
description: "第一篇文章"
tags: ["hugo", "pages"]
categories: ["pages"]
author: ""
---

这里是文章内容
```

### 预览

执行命令，使用 Hugo 生成静态内容并在启动本地 HTTP Server。然后即可访问 http://localhost:1313/ 查看效果。

```bash
hugo server -D
#...
#Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

![Hugo Server](/images/start-hugo/server.png)

> Hugo server 会检测文件变化，自动刷新浏览器。

---

## 部署到 GitHub Pages

最终我们需要把博客部署到一个托管服务，免费稳定的 Github Pages 是个很好的选择。再结合 Travis 自动部署，发布文章会变得很简单。

1. 先把源码提交到 GitHub 的一个 repo (源码 repo)

    ```bash
    git add -A
    git commint -m "initial all files"
    git remote add origin https://github.com/<username>/blog
    git push -u origin master
    ```

2. 准备发布博客使用的 pages repo

    Github Pages 有多种类型：个人、组织、个人的某个项目、组织的某个项目。具体细节官方文档可见 [GitHub Pages](https://help.github.com/articles/user-organization-and-project-pages/)。本文使用的是 `<username>.github.io`。

3. 首先在 Github 上创建 `<username>.github.io`repo，同时 config.toml 的 `baseURL` 要设置成 `https://<username>.github.io`

4. 生成 [Github Access Token](https://github.com/settings/tokens/new)，至少要有 public_repo 的权限。![Access Token](/images/start-hugo/access-token.png)

5. 配置 Travis

    去 [Travis CI](https://travis-ci.org/) 注册关联 Github 的账号，然后同步账户并激活 blog repo。

    ![Travis Account](/images/start-hugo/travis-account.png)

    接着进入 blog 的设置页面，选择自动部署触发条件，并把刚刚生成的 GitHub Access Token 添加到环境变量里。

    ![Travis Settings](/images/start-hugo/travis-settings.png)

6. 在 blog repo 中添加 .travis.yml

    ```yaml
    sudo: false
    language: go
    git:
        depth: 1
    install: go get -v github.com/gohugoio/hugo
    script:
        - hugo
    deploy:
        provider: pages
        skip_cleanup: true
        github_token: $GITHUB_TOKEN
        on:
            branch: master
        local_dir: public
        repo: <username>/<username>.github.io
        fqdn: <custom-domain-if-needed>
        target_branch: master
        email: <github-email>
        name: <github-username>
    ```

    部分参数解释：

    > * 默认情况下，travis 会自动下载 git submodules
    > * github_token: $GITHUB_TOKEN 要和 travis 设置的环境变量名一致
    > * fqdn: 如果需要设置自定义域名，可以在这里设置，travis 会自动生成 CNAME 文件提交，同时要设置 config.toml 中的相应的 `baseURL`

7. 最后，可以手动去 travis 触发一次 build 检查效果。如果设置了提交触发 build，之后每次 blog repo 有提交都会自动 build，不再需要关心 travis 状态。
