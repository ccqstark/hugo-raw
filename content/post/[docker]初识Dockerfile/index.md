---
title: "[docker]初识Dockerfile"
date: 2021-01-06T21:13:00+08:00
draft: true
image: dockerfile.png
slug: "dockerfile"
tags:
    - Docker
categories:
    - 环境搭建
---

## Dockerfile介绍

dockfile是用来构建docker镜像的文件，命令参数脚本

💡构建步骤：

1. 编写`dockerfile`脚本
2. 用`docker build`命令构建一个镜像
3. 用`docker run`运行镜像
4. 用`docker push`发布镜像（DockerHub、阿里云仓库）

在官网点击镜像会跳转到github对应的dockerfile

可以发现这些镜像也是通过dockerfile来构建的

![Dockerfile%20301ab8b18fee4cdb9b1ffde8b96fcc3d/Untitled.png](Dockerfile%20301ab8b18fee4cdb9b1ffde8b96fcc3d/Untitled.png)

上图是centos的dockerfile，其中`scratch`是最基本的，90%都是基于这个镜像。

然后`ADD` 就是添加来一层centos相关的镜像文件

官方很多镜像都是基础包，功能很少，很多我们需要的都没有，所以我们通常都会构建自己的镜像。

比如我们可以直接构建一个`centos+jdk+tomcat+mysql`的镜像，不就直接有来一个可以运行javaweb项目的环境镜像了吗？

## Dockerfile构建过程

### 基本规则

1. 每个关键字（保留字）都是大写的
2. 执行顺序是从上到下的
3. "#" 表示注释
4. 每一个指令都会创建一个新的镜像层，并提交

![Dockerfile%20301ab8b18fee4cdb9b1ffde8b96fcc3d/Untitled%201.png](Dockerfile%20301ab8b18fee4cdb9b1ffde8b96fcc3d/Untitled%201.png)

以前开发交付都是用jar包或war包，现在**云原生**时代交付的就是docker镜像，docker镜像也逐渐成为企业交付标准，而构建docker镜像就需要学会编写dockerfile

[什么是云原生？聊聊云原生的今生_阿里云开发者-CSDN博客](https://blog.csdn.net/alitech2017/article/details/104606956)

## Dockerfile常用指令


| 指令关键字      | 作用                                    |
|------------|---------------------------------------|
| FROM       | 构建镜像所用的基础镜像                           |
| MAINTAINER | 镜像作者，一般是姓名+邮箱                         |
| RUN        | 镜像构建时运行的命令                            |
| ADD        | 为镜像添加内容                               |
| WORKDIR    | 镜像的工作目录                               |
| VOLUME     | 挂载目录                                  |
| EXPOSE     | 暴露的端口                                 |
| CMD        | 容器启动时需要运行的命令，只有最后一个会生效，可被替代           |
| ENTRYPOINT | 也是指定启动时需要运行的命令，但是可以追加                 |
| ONBUILD    | 构建一个被继承的dockerfile时会运行ONBUILD的指令。触发指令 |
| COPY       | 类似ADD，将文件拷贝到镜像中                       |
| ENV        | 构建时设置的环境变量                            |


## 实践：构建自己的centos

举个例子：

```docker
FROM centos  # centos为基础镜像
MAINTAINER ccqstark<xxxxxx@qq.com>  # 作者名和邮箱

ENV MYPATH /usr/local  # 环境变量
WORKDIR $MYPATH  # 工作目录

# 安装vim和ifconfig命令
RUN yum -y install vim      
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "---end---"
CMD /bin/bash
```

`docker build` 之后 `run` 起来就可以使用了！

还可以使用下面命令查看镜像构建的过程

```bash
docker history [镜像id]
```

参考自[狂神的docker教程](https://www.bilibili.com/video/BV1og4y1q7M4?p=29)