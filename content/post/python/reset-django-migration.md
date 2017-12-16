---
title: "重置Django migration的常见方式"
date: 2017-12-13T19:05:35+08:00
lastmod: 2017-12-13T19:05:35+08:00
weight: 50
keywords: ["django", "migration", "python"]
description: "重置Django migration的常见方式"
tags: ["django", "python"]
categories: ["Django"]
author: "lxzxl"
---

根据 django 官方文档建议，开发过程中会把生成的 migrations 提交到 git 中。由于各种原因，会有一些场景需要重置 migrations，故总结一些常用场景及解决办法。

## 场景一

> 不考虑数据库数据，可以完全清空数据库。

步骤：

1. 删除所有 migrations

   ```shell
   find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
   find . -path "*/migrations/*.pyc"  -delete
   ```

2. 删除数据库

3. 重新生成 migrations

   ```shell
   python manage.py makemigrations
   python manage.py migrate
   ```

## 场景二

> 有时候我们会直接导入完整的数据库，包括数据，这种情况下就不能简单的清空数据库。这时我们的目的就是：清空数据库的 migration history，保证以后的 migrate 能正常使用，但要保留其他数据。

步骤：

1. 从数据库中删除所有非`0001_initial`的 migration history

   ```mysql
   DELETE FROM django_migrations WHERE app IN ('your','app','labels') AND name != '0001_initial'
   ```

2. 使用 migrate 命令回滚`0001_initial`的 migration history

   ```shell
   python manage.py migrate --fake your zero
   python manage.py migrate --fake app zero
   python manage.py migrate --fake labels zero
   ```

3. 重新生成`0001_initial`，如果能保证已有`0001_initial`已是最新的，可跳过此步

   ```shell
   find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
   find . -path "*/migrations/*.pyc"  -delete

   python manage.py makemigrations
   ```

4. 在数据库中生成新的`0001_initial`记录

   ```shell
   python migrate --fake-initial
   ```

   ​
