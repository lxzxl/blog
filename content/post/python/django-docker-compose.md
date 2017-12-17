---
title: "使用Docker Compose部署Django和Vue.js应用"
date: 2017-07-18T22:09:00+08:00
lastmod: 2017-07-18T22:09:00+08:00
keywords: ["django", "docker compose", "vue"]
description: "使用Docker Compose部署Django和Vue.js应用"
tags: ["django", "docker", "python"]
categories: ["Django", "Docker"]
author: "lxzxl"
---

# 前言

本文主要内容关于使用 docker-compose 实践部署后端 django-rest-framework 和前端 vue.js 应用。记录其中遇到的一些坑以及解决办法。

# 准备 Docker-compose 环境

系统：Ubuntu 16.04(阿里云)
代码中用户名：test

## 安装 Docker

```bash
# install docker
## prepare
echo 'Preparing...'
sudo apt update
sudo apt upgrade -y
sudo apt install -y linux-image-extra-$(uname -r) linux-image-extra-virtual
## docker
echo 'Installing docker...'
sudo apt remove -y docker-ce docker-engine docker.io
wget -qO- http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh
sudo apt autoremove -y

sudo usermod -aG docker ${USER}

## docker Aliyun accelerator
## https://cr.console.aliyun.com/#/accelerator
echo 'Configuring docker registry mirrors...'
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://<your-own>.mirror.aliyuncs.com"]
}
EOF

exists(){
  command -v "$1" >/dev/null 2>&1
}

echo 'Installing docker-compose...'
if ! exists pip; then
    sudo apt install python-pip
fi
sudo python `which pip` install docker-compose

echo 'Restarting docker...'
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 部署

### 目录结构

```
.
├── .env // 环境变量
├── docker-compose.yml
├── backend // 放置后台django文件
├── frontend // 放置前端vue编译后代码
└── nginx // nginx相关配置
    ├── backend.conf
    ├── Dockerfile
    └── frontend.conf
```

### 具体配置

#### `docker-compose.yml`

```yaml
version: '3'

services:
  web:
    restart: always
    build: ./backend
    expose:
      - "8000"
    volumes:
      - ./backend:/code
    env_file: .env
    links:
      - db
    depends_on:
      - db
    command: ["/code/wait-for-it.sh", "db:3306", "--", "bash","startup.sh"]
  nginx:
    restart: always
    build: ./nginx
    ports:
      - "80:80"
    volumes:
      - ./frontend:/usr/share/nginx/html/frontend:ro
      - ./backend/public:/usr/share/nginx//html/backend/public:ro
    links:
      - web
    depends_on:
      - web
  db:
    restart: always
    image: mysql:latest
    env_file: .env
    volumes:
      - ./data/initsql:/docker-entrypoint-initdb.d
      - ./data/db:/var/lib/mysql
    command: [mysqld, --character-set-server=utf8, --collation-server=utf8_unicode_ci]
```

第一次使用 docker-compose 部署，从网上参考了许多例子。但是由于 docker-compose 个版本的语法改动不小，遇到很多坑：

#### 不同容器共享数据(host 主机上的数据)

> 有些例子使用`volumes_from`，但是 version 3 已经删除该设置项[更新变化](https://docs.docker.com/compose/compose-file/compose-versioning/#version-3)。官方推荐在根节点（与`services`同级）下配置`volumes`，但是没法映射到 host 主机的文件，有个插件[`local-persist`](https://github.com/CWSpear/local-persist)可以做到，但是不想依赖第三方插件。只能使用麻烦点的写法：在每个 service 的`volume`设置项里重复映射 host 主机的文件
>
> ```yaml
> web:
>   volumes:
>     - ./backend:/code
> nginx:
>   volumes:
>     - ./backend/public:/usr/share/nginx//html/backend/public:ro
> ```

#### 不同容器互相通信

> 使用 links 实现容器间通信。配置需要访问的容器于 links 配置项下，没什么毛病。

#### 容器启动顺序

> 实现容器按顺序启动，需要用到`depends_on`。
>
> 但是会有一个问题：`depends_on`只保证启动顺序，而我们的实际需求是：web 容器启动的 commands 必须等到 mysql 完全启动，web 容器的 command 才可以执行。因为我们加了`restart: always`，所以会导致 web 容器不断重启直到 db 容器启动完成。
>
> 我的解决办法是：给 web 容器的启动命令加上对 mysql 服务可用的查询，使用了一个查询脚本[wait-for-it](https://github.com/vishnubob/wait-for-it):
>
> ```dockerfile
> command: ["/code/wait-for-it.sh", "db:3306", "--", "bash","startup.sh"]
> ```

#### 环境变量

> 会有多个容器使用相同环境变量的情况，所以都放在`.env`文件里
>
> ```shell
> DEBUG=false
>
> MYSQL_HOST=db
> MYSQL_DATABASE=mydb
> MYSQL_ROOT_PASSWORD=mypass
> ```

#### 关于 Mysql 官方镜像配置

> * 可以定义 volume 持久化数据库文件：
>
>       ```yaml
>       volumes:
>         - ./data/db:/var/lib/mysql
>       ```
>
> * 如果有初始数据需要导入，可以定义 volume 映射到`/docker-entrypoint-initdb.d`：
>
>       ```yaml
>       volumes:
>         - ./data/initsql:/docker-entrypoint-initdb.d
>       ```
>
>       注意： 该配置只有在`/var/lib/mysql/`下的`mysql`目录不存在时才会生效。也就是说，一旦容器启动过一次，之后就在也不会导入`/docker-entrypoint-initdb.d`里的文件，除非手动清空`/var/lib/mysql/`（或 host 主机的`./data/db`目录）。

#### 关于 Nginx 容器配置

> `Dockerfile`
>
> ```dockerfile
> FROM nginx:alpine
>
> RUN rm /etc/nginx/conf.d/default.conf
>
> ADD frontend.conf /etc/nginx/conf.d/
> ADD backend.conf /etc/nginx/conf.d/
> ```
>
> `frontend.conf` - vue app build files
>
> ```nginx
> server {
>     listen 80 deferred;
>     server_name new.bylie.cn;
>
>     root /usr/share/nginx/html/frontend;
>
>     location / {
>         try_files $uri $uri/ /index.html =404;
>     }
>
>     # redirect server error pages to the static page /50x.html
>     #
>     error_page   500 502 503 504  /50x.html;
>     location = /50x.html {
>         root   /usr/share/nginx/html;
>     }
> }
> ```
>
> `backend.conf` - django app
>
> ```nginx
> server {
>     # use 'listen 80 deferred;' for Linux
>     # use 'listen 80 accept_filter=httpready;' for FreeBSD
>     listen 80 deferred;
>     client_max_body_size 5M;
>
>     # set the correct host(s) for your site
>     server_name service.bylie.cn;
>
>     keepalive_timeout 5;
>
>     location /public {
>         root /usr/share/nginx/html/backend;
>     }
>
>     location / {
>         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
>         # enable this if and only if you use HTTPS
>         # proxy_set_header X-Forwarded-Proto https;
>         proxy_set_header Host $http_host;
>         # we don't want nginx trying to do something clever with
>         # redirects, we set the Host: header above already.
>         proxy_redirect off;
>         proxy_pass http://web:8000;
>     }
>
>     error_page   500 502 503 504  /50x.html;
>     location = /50x.html {
>         root   html;
>     }
> }
> ```
>
> 备份数据库到 host 主机
>
> ```shell
> docker-compose exec db sh -c 'exec mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" --databases ${MYSQL_DATABASE}' > ./backup/database.sql
> ```

#### 关于 django web 容器配置

> 比较简单
>
> `Dockerfile`
>
> ```dockerfile
> FROM python:3
>
> ENV PYTHONUNBUFFERED 1
> RUN mkdir /code
> WORKDIR /code
> ADD requirements.txt /tmp/
> RUN pip install -r /tmp/requirements.txt
> ```
>
> 启动脚本`startup.sh`
>
> ```bash
> #!/usr/bin/env bash
> python manage.py collectstatic --noinput &&
> python manage.py migrate &&
> gunicorn django_web_app.wsgi:application -w 2 -b :8000
> ```
>
> 当容器运行起来之后，可能需要导入一些初始数据或者 fixtures，我写了一个 init 的 django custom command，手动执行该 command：
>
> ```bash
> docker-compose exec web python manage.py init
> ```
