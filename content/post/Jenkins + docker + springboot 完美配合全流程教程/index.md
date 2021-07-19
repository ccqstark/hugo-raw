---
title: "Jenkins + docker + springboot 完美配合全流程教程"
date: 2021-01-09T23:01:00+08:00
draft: true
image: jenkins-cicd.png
slug: "jenkins_docker_springboot"
tags:
    - CI/CD
categories:
    - 环境搭建
--- 

DevOps现在非常流行，CI/CD持续集成、持续部署也大火，而`Jenkins`就是自动化部署主要的工具之一。

这篇博客就来详细介绍用jenkins来实现自动化部署springboot项目的docker容器，堪称保姆级教学了。

## 用docker拉取jenkins镜像，启动Jenkins容器

这里采用的jenkins本身也是用docker容器部署的，不得不说docker确实好用，当然也可以直接运行在主机上

### 首先拉取Jenkins镜像

```bash
docker pull jenkins/jenkins
```

⚠️注意：切勿docker pull jenkins，已经废弃

### 启动Jenkins容器

```bash
docker run -u root -itd --name jenkins \
-p 6001:8080 \
-v $(which docker):/usr/bin/docker \
-v /var/run/docker.sock:/var/run/docker.sock -e TZ="Asia/Shanghai" \
-v /etc/localtime:/etc/localtime:ro \
-v /volume1/docker/jenkins:/var/jenkins_home \
jenkins/jenkins
```

- `-p 6001:8080`Jenkins默认网页访问端口为8080，将端口映射到外部主机6001端口
- `-v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock`使Jenkins内部可以使用docker命令
- `-e TZ="Asia/Shanghai" -v /etc/localtime:/etc/localtime:ro`配置Jenkins容器的时区
- `-v /volume1/docker/jenkins:/var/jenkins_home` 将Jenkins的配置映射到外部主机卷，容器删除仍可保留配置

### 测试Jenkins容器内部

```bash
# 进入Jenkins的容器内部
docker exec -it jenkins bash

# 判断docker命令是否正常执行
docker info
```

### 访问Jenkins网页端

用`http://主机IP:6001` 就可以访问Jenkins的网页端了

## Jenkins初始化

访问页面后需要输入初始密码，用`cat` 命令查看一下页面上给出的路径就可以获得初始密码，复制进去后就可以成功进入

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled.png)

到插件这里就选择安装推荐插件即可，等待安装完毕，速度稍慢。如果很多插件一直安装失败可以等下一步配置国内源之后再安装

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%201.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%201.png)

然后按提示创建一个自己的管理员账户

**如果想重启Jenkins：**

在jenkins主页网址后加上`/restart` 后回车，点击确定即可

## 安装插件和必要配置

### 修改插件国内源并安装其它插件

点击侧边栏`系统管理`→`插件管理`→`高级`

![https://static01.imgkr.com/temp/06e2f322cdd84009b2c0359631f7b899.jpg](https://static01.imgkr.com/temp/06e2f322cdd84009b2c0359631f7b899.jpg)

将图中所示URL的输入中的链接改为阿里的：

```bash
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

### 安装一些插件

在`插件管理`中选择`可选插件`

安装`Maven Integration`和`Docker`的插件，安装完重启

### 全局工具配置

点击`系统管理`→`全局工具配置`

JDK

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%202.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%202.png)

用`docker inspect jenkins` 查看`JAVA_HOME` 路径后填入即可，这是个openjdk1.8，刚好用于项目，因为jenkins就是Java开发的

**Git**

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%203.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%203.png)

用默认即可

**Maven**

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%204.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%204.png)

可以用外部安装的，这里因为下了插件就用Jenkins里的插件就行，点击自动安装

**Docker**

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%205.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%205.png)

都好了之后点`应用`在点`保存`

### Jenkins容器内使用vim

进入Jenkins容器内部后想使用vim的话还需要额外安装，后面需要用到vim

```bash
# 更新一下软件源
apt-get update

# 安装vim
apt-get install vim
```

这里可能比较慢，就等一下，但只需用一次就不额外换源了

### 修改maven插件的镜像源

因为jenkins在容器内，所以要进入容器内

`cd`到目录`~/.m2` 下，`ls` 一下发现只有一个`repository` 目录，这个就是默认的maven仓库目录，然后就`vim settings.xml` 新建一个配置文件

在命令模式下用`:set paste` 开启粘贴模式，然后把下面内容粘贴进去，记得检查一下内容和编码是不是`utf-8`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <localRepository>~/.m2/repository</localRepository>
  
  <pluginGroups>
    
  </pluginGroups>

  <proxies>

  </proxies>

  <servers>
    
  </servers>

  
  <mirrors>
     <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>

  <profiles>
    
  </profiles>

</settings>
```

重启jenkins，之后下载包时发现已经更改为阿里源了

## 创建项目准备自动化部署

左侧边栏新建一个任务，选择maven项目

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/451610197149_.pic.jpg](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/451610197149_.pic.jpg)

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/461610197185_.pic_hd.jpg](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/461610197185_.pic_hd.jpg)

### 添加GitHub仓库源码

这里用GitHub做代码仓库，也可以gitlab

有2种方式：`https`和`ssh`

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%206.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%206.png)

如果是https的话添加`Credentials` 的时候就直接配置自己的GitHub用户名和密码，仓库的URL填写github仓库的url就行

如果是ssh的话要配置密钥，点击`系统管理`→`Manage Credentials` →`全局` ,点击左侧边栏`添加凭据`

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%207.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%207.png)

如果之前生成过，在自己机器上的`~/.ssh` 下cat一下就行，没有的话就`ssh-keygen` 生成

注意此时url就要用`ssh://git@github.com/[用户名]/[项目名].git` 的格式

### 加快代码拉取速度

为了加快构建速度，勾选此选项可以使jenkins不拉取代码的历史版本，从而加快构建速度

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%208.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%208.png)

勾选浅克隆

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%209.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%209.png)

初次之外，不要在本地打出jar包，不然这么大一个文件上传到仓库再被拉取很费时间

### 构建触发器

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2010.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2010.png)

### 添加maven构建步骤
![clean_package](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/clean_package.png)
### 构建脚本

这里是运行一些shell脚本来构建docker镜像和运行容器的

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2011.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2011.png)

脚本如下，记得在项目目录下写好一个Dockerfile

```bash
# 进入项目目录
cd /var/jenkins_home/workspace/[项目名]

# 执行构建Dockerfile命令
docker build -f Dockerfile -t [镜像名]:[tag] .

# 停止之前的容器运行
docker stop [容器名]
# 删除之前的容器
docker rm [容器名]

#运行刚刚创建的容器
docker run -d --name [容器名] -p [映射端口]:8080 [镜像名]:[tag]

echo "构建完成"
```

Dockerfile参考：

```docker
FROM openjdk:8

MAINTAINER [作者]

ADD /target/[项目名]-0.0.1-SNAPSHOT.jar [项目名].jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/[项目名].jar"]
```

### push触发构建

为了实现只要我们一向代码仓库push就可以自动进行构建，我们需要配置`webhook`

点击`系统管理`→`系统配置` ，找到`GitHub`选项，点击`高级` 

按下图操作：

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2012.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2012.png)

打开自己GitHub项目页面

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2013.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2013.png)

粘贴刚刚复制的地址

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2014.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2014.png)

下面勾选`Pushes`和`Active` ，最后点击添加即可

点击项目内左侧栏的`立即构建` 可以手动开始构建

在进程中点击`控制台输出` 看到构建过程中的日志信息，这些信息很重要我们经常要看，用来发现构建过程中的错误

### 清除无用的镜像

```bash
docker image prune
```

在多次构建之后可能会发现一些为none的镜像，用此命令清除

## 配置邮箱通知

在这之前保证安装了相关插件，如果一开始是选安装推荐插件那应该都安装了

点击`系统管理`→`系统配置`

先配一下管理员邮箱

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2015.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2015.png)

然后拉到下面，按图中配置，这个邮箱要填刚刚上面的管理员邮箱

注意里面的密码是开启SMTP的密码，不是邮箱的密码

然后可以点击发送测试邮件试试

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2016.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2016.png)

上面还有一个`Extended E-mail Notification` 也配一下，也按这些信息填写

`Default Recipients` 是默认接收通知的邮箱，可以填写多个，用英文半角逗号隔开

`Default Triggers` 也可以配置一下，是触发邮件的事件

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2017.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2017.png)

这里有个坑，在系统设置配置完之后，在项目里面的设置记得也要配置完整

进入要发送邮箱功能的项目的`配置`

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2018.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2018.png)

再点击`增加构建后操作步骤`

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2019.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2019.png)

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2020.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2020.png)

点击`高级设置`后设置触发条件

![Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2021.png](Jenkins%20+%20docker%20+%20springboot%20%E5%AE%8C%E7%BE%8E%E9%85%8D%E5%90%88%E5%85%A8%E6%B5%81%E7%A8%8B%E6%95%99%E7%A8%8B%20e510c92879ca422aafb008869a20a133/Untitled%2021.png)

其他的诸如`Content Type`和`Default Content` 之类的就按需求配就行

之后在控制台输出就可以看到构建项目到最后有发邮件的步骤日志

### 主题美化

可以参考下面这篇文章，个人觉得还是习惯于原版

[Jenkins自定义主题教程_FlyWine的博客-CSDN博客_jenkins自定义界面](https://blog.csdn.net/wf19930209/article/details/80386902)

**参考文章：**

[最优雅的Docker+Jenkins pipeline部署Spring boot项目](https://juejin.cn/post/6844903955600769031#heading-0)

[Jenkins+Docker+github+Spring Boot自动化部署_linfen1520的博客-CSDN博客](https://blog.csdn.net/linfen1520/article/details/109045063)

[jenkins+git+maven+docker持续集成部署_自动化_运维开发网_运维开发技术经验分享](https://www.qedev.com/auto/170005.html)

[Jenkins - SSH认证方式拉取Git代码](https://www.cnblogs.com/xiaoxi-3-/p/9680205.html)

[centos7的Jenkins的maven插件的settings.xml配置文件路径在哪里_festone000的专栏-CSDN博客](https://codejam.blog.csdn.net/article/details/106043546?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3.control)