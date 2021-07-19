---
title: "[docker]用docker部署SpringBoot项目"
date: 2021-01-07T21:39:00+08:00
draft: true
image: docker-springboot.jpg
slug: "docker_springboot"
tags:
    - Docker
categories:
    - 环境搭建
---

这篇文章介绍的是把整个Springboot后端项目部署到docker容器中，当然包括mysql和redis

按下面步骤一步步来

### 本地打出jar包

以Maven的话直接就IDEA里打出jar包到target目录下，这一步和以前一样

### 编写Dockerfile

可以用IDEA里的插件来写，也可以自己写dockerfile

在项目文件夹下新建一个文件Dockerfile

```docker
FROM openjdk:8

MAINTAINER ccqstark

ADD /target/[项目jar包名].jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

⚠️注意：ADD后两个参数，第一个是项目jar包的相对路径，第二是把jar包在容器内重新命的名

### 构建镜像

在Dockerfile所在文件夹下运行`build` 命令，注意最后有一个`.`

```docker
docker build -f Dockerfile -t [镜像名]:[版本tag] .
```

构建之后用`docker images` 查看一下自己构架的镜像

构建完之后本地run一下容器测试下

### push上传到镜像仓库

其实也可以把jar包上传服务器后用服务器的docker来构建和运行

但这里采用的是把本地构建的镜像上传到repository，相当于镜像仓库，其他人想用这个镜像就可以从那拉取下来使用。

repository可以是官方的`Docker Hub`，但是比较慢，也可以花钱上传到阿里云的容器镜像服务就会快很多

这里是上传到docker hub，首先要登陆自己到docker账号，没有的话可以去官网注册一个

```bash
docker login -u [账户名]
```

输入密码成功后登陆

在push之前要给镜像打个tag，这样才能上传到自己账号对应的仓库下

```bash
docker tag [镜像名] [账户名]/[镜像仓库名]:latest
```

之后就可以上传了

```bash
docker push [账户名]/[镜像仓库名]:latest
```

### pull拉取镜像

```bash
docker pull [账户名]/[镜像仓库名]:[tag]
```

在服务器上拉取到镜像后就可以启动容器了

```bash
docker run -it -d -p [对外暴露端口]:8080 app:[tag]
```

### 部署MySQL容器

```bash
# 拉取mysql镜像
docker pull mysql:5.7

# 跑起来
docker run \
-d \
-p 3306:3306 \
-v /home/mysql/conf:/etc/mysql/conf.d \
-v /home/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=[设置mysql到root密码] \ 
--name [容器名] \
mysql:5.7
```

mysql容器到暴露端口要和代码中配到一样就行

这里还把配置目录和数据目录挂载了出来，避免容器停止后数据丢失

## 部署redis容器

```bash
# 拉取redis镜像
docker pull redis

# 运行，这个时候指定密码，不指定默认为空
docker run -d --name myredis -p 6379:6379 redis --requirepass "mypassword"
```

⚠️注意：建议先把mysql和redis都部署好后再去启动jar包都镜像，防止应用启动时连不到它们而报错

所有容器都成功启动起来之后就把整个后端部署到docker完毕了