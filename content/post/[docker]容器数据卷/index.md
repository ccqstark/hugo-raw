---
title: "[docker]容器数据卷"
date: 2021-01-06T16:51:00+08:00
draft: true
image: docker.png
slug: "docker_volumes"
tags:
    - Docker
categories:
    - 环境搭建
---

把容器内的目录挂载到宿主机的某一个目录下，实现双向同步。

也就是说两者都指向了同一文件目录下，在其中一端所做的修改都会同步。

好处：

1. MySQL数据持久化，不会因为删了容器就没了
2. 方便修改文件，比如nginx的配置文件

## 基本使用 bind mounts

以启动一个centos容器为例

```bash
docker run -it -v [宿主机目录]:[容器内目录] centos /bin/bash
```

`-it` ：`-t`选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开，通常写成`-it`

`-v` ：挂载卷所需参数，后面的映射是`[宿主机目录]:[容器内目录]`

用此命令查看容器参数

```bash
docker inspect [容器id]
```

![%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled.png](%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled.png)

如上图，在`Mounts` 字段中可以看到：

`Source` 表示宿主机中被映射的目录

`Destination` 表示容器内要映射的目录

这种挂载方式称为`bind mounts`

## **实践：MySQL挂载**

### **拉取mysql镜像**

```bash
docker search mysql
docker pull mysql:5.7
```

### **启动容器**

`-d` 后台运行

`-p` 端口映射

`-v` 数据卷挂载

`—name` 容器名字

```bash
docker run \
-d \
-p 3310:3306 \
-v /home/mysql/conf:/etc/mysql/conf.d \
-v /home/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=[你配置的mysql密码] \
--name [容器名] \
mysql:5.7
```
`-v`可以一次写多个来多次挂载

### **连接测试**

运行成功后用navicat连接下试试

- 主机地址还是服务器公网ip
- 端口是映射出来的暴露端口，比如上面命令中的`3310`
- 密码就是`-e MYSQL_ROOT_PASSWORD`设置的密码

 可以创建新的数据库看看宿主机对应映射目录下有没有同步出现新数据库的文件

### **删除测试**

```bash
docker rm -f [mysql容器名]
```

运行上面的指令删除掉容器，再在主机下查看`/home/mysql/data` 目录发现数据依旧都还在

如果再次启动一个容器数据就还是和删除前一样，从而保证了数据安全，这就是MySQL数据卷挂载

## 匿名挂载和具名挂载 volumes

### **匿名挂载**

`-P` 随机映射端口

```bash
docker run -d -P --name nginx01 -v /etc/nginx nginx
```

如上图的命令，`-v` 是没有指定外部目录的，只写了内部目录，所以是匿名挂载，使用下面命令查看挂载的卷

```bash
docker volume ls
```

发现卷名是随机生成的字符串，所以是匿名的

![%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled%201.png](%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled%201.png)

### **具名挂载**

```bash
docker run -d -P --name nginx01 -v [自己起的卷名]:/etc/nginx nginx
```

`[卷名]:[目录名]` 这样指定了卷名的形式就是具名挂载

这样再用`docker volume ls` 看到的卷名就是自己指定的了

### **查看挂载的目录**

```bash
docker volume inspect [卷名]
```

用这条命令就可以查看卷的一些信息，其中`Mountpoint` 就是所挂载的外部目录

![%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled%202.png](%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled%202.png)

所以这种在没有指定目录的情况下（具名或匿名）都是挂载在`/var/lib/docker/volumes/[卷名]/_data`这个目录的

大多数情况下都是用`具名挂载`

这两种挂载方式统称`volumes`

## 总结

`-v [内路径]`        匿名挂载                                                                                                                            `-v 卷名:内路径`   具名挂载                                                                                                                   `-v 宿主机路径:容器内路径`  指定路径挂载

## 扩展

可以用参数改变读写权限

`ro` 只读，容器内不可修改，容器外可以

`re` 可读可写 

```bash
docker run -d -P --name nginx02 -v ccq-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v ccq-nginx:/etc/nginx:rw nginx
```

## 使用Dockerfile来构建和挂载

Dockerfile是用来构建docker镜像的构建文件，里面就是构建的脚本

这个文件可以用来生成镜像，由于镜像是一层一层的，所以脚本命令也是一句句对应一层层的

以centos来举个例子，在dockerfile里写下下面这些内容：

```docker
FROM centos  # 由哪个原始镜像构建

VOLUME ["volume01","volume02"]  # 挂载，此处匿名

CMD echo "---end---"

CMD /bin/bash
```

运行构建命令如下：

```bash
docker build -f [dockerfile路径] -t [镜像名]:[版本tag] [生成目录]
```

然后用命令`docker images` 就可以看到自己刚刚构建的镜像了，`docker run` 就可以跑起来

之后也可以`docker inspect` 查看挂载情况，挂载同样是数据内外目录同步的

## 数据卷容器

之前是容器内的目录挂载到容器外到到目录，现在是一个容器挂载到另一个容器

这样就实现了容器之间到数据同步

`—-volumes-from` 使用此参数开启一个容器挂载到另一个容器

被挂载的称为`父容器`

```bash
docker run -it --name [名字] --volumes-from [父容器名] [镜像名]:[版本tag] 
```

测试一下，开两个容器，进入挂载到一起的目录创建文件试试，发现数据是同步的

### 多重挂载

可以开启第三个容器挂载到第一或第二个容器，发现现在这3个容器对应到目录的数据都是同步的

所以挂载其实是可以`套娃`的

### 卷的挂载机制

不同容器的挂载机制并不是映射到单一到文件夹下的，如果这样的话其中一个容器被删除的话，其它所有容器对应都数据都会消失。

实际上挂载是一种`拷贝`的机制，数据是有多份相同的备份的，删除一种一份其它的都还在，不会消失的，只是会占用更多的存储空间

![%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled%203.png](%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%20b1baaa6db0a64ef88290e380e7cc610d/Untitled%203.png)

只有把挂载这一卷的所有的容器都删除，这个卷才会消失

当然，如果是bind到本地的目录那就删除全部容器数据也仍然在本地


参考自[狂神的docker教程](https://www.bilibili.com/video/BV1og4y1q7M4?p=21)