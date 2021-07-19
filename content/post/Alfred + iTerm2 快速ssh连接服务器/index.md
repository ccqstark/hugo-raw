---
title: "Alfred + iTerm2 快速ssh连接服务器"
date: 2021-01-10T16:35:00+08:00
draft: true
image: ssh.png
slug: "alfred_iterm"
tags:
    - MacOS
categories:
    - 其它技术
---

为了提高在mac下连接ssh的效率，我们可以用alfred和iTerm配合，达到只要在输入框中输入`ssh [主机名]` 就可以快速连上了，效果如下图：

![Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled.png](Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled.png)

### 使用ssh config

在`~/.ssh/config`文件里添加服务器信息，没有的话就新建一个

```docker
vim ~/.ssh/config
```

然后在文件中输入主机的信息，有多个主机就追加在后面就行

```docker
Host [主机名]
  HostName [ip]
  User root
  Port [端口]
```

### 使用密钥登陆

如果本地的`~/.ssh` 目录下没有`id_rsa` 私钥文件，可以是使用下面这个目录生成，一路回车即可，如果已经有了就可以跳过这步

```docker
ssh-keygen
```

然后将私钥复制到远程服务器

```docker
ssh-copy-id -i -p[端口号] root@ip
```

按提示输入一次密码，就会自动将刚才生成的公钥`id_rsa.pub`追加到远程主机的`~/.ssh/authorized_keys`后面了，这样以后的 ssh 连接都不用输入密码了

### 安装alfred-ssh插件

[https://github.com/deanishe/alfred-ssh](https://github.com/deanishe/alfred-ssh) 

到上面github链接下载最新版：Secure-SHell的alfredworkflow，双击自动添加到alfred的workflow

添加后打开alfred的偏好设置可以看到效果如下：

![Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled%201.png](Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled%201.png)

测试用alfred输入ssh+主机名就可以连上服务器了，但是默认是用mac自带但终端，想用好看的iTrem2还需要进一步操作

### 安装alfred集成iTerm2配置

如下图，打开iTrem2的偏好设置，如下图设置默认方式为ssh

![Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled%202.png](Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled%202.png)

进入下面github链接，按说明操作

[https://github.com/vitorgalvao/custom-alfred-iterm-scripts](https://github.com/vitorgalvao/custom-alfred-iterm-scripts)

![Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled%203.png](Alfred%20+%20iTerm2%20%E5%BF%AB%E9%80%9Fssh%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8%20f43ec088887841b79f3c5019d51d1016/Untitled%203.png)

按上面要求运行命令并粘贴到对应地方就完成了！  

参考文章：
[开发效率神器之alfred集成ssh+iTerm2实现一步登录服务器](https://yaoyuanyy.github.io/2019/05/13/%E5%BC%80%E5%8F%91%E6%95%88%E7%8E%87%E7%A5%9E%E6%8F%90%E5%8D%87%E4%B9%8Balfred%E9%9B%86%E6%88%90ssh+iterm/)