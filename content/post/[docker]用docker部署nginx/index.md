---
title: "[docker]用docker部署nginx"
date: 2021-01-08T15:46:00+08:00
draft: true
image: nginx-docker.png
slug: "docker_nginx"
tags:
    - Docker
categories:
    - 环境搭建
---

如果是单体应用的话nginx用docker部署其实是更麻烦的，不过既然操作过就记录一下。

### 拉取nginx镜像

```bash
docker pull nginx
```

还是一样，默认是拉取latest版本，也可以选择想要的特定版本

### 启动并挂载html目录

```bash
docker container run \
  -d \
  -p 80:80 \
  --name mynginx \
  --v [本机挂载目录]:/usr/share/nginx/html \
  nginx
```

### 复制出配置文件

```bash
docker container cp mynginx:/etc/nginx .
```

将复制出来的文件夹改名并移动到你想要的目录下，然后把容器停止并删除

### 挂载配置文件目录

最后一步就是重新启动一个容器并把html和配置文件目录都挂载了

```bash
docker run \
  --name test-nginx \
  -v [本机挂载html目录]:/usr/share/nginx/html \
  -v [本机挂载nginx目录]:/etc/nginx \
  -p 80:80 \
  -d \
  nginx
```

访问一下试试就可以了！

参考：

[Nginx 容器教程](https://www.ruanyifeng.com/blog/2018/02/nginx-docker.html)