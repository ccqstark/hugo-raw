---
title: "记一次服务器被攻击"
date: 2020-11-22T22:10:00+08:00
draft: true
slug: "first_attack"
tags:
    - DDos
categories:
    - 网络安全
---

### 事情经过

11月17号这天早上上着课，突然有用户反馈应用卡顿，使用不了。我感觉上去看了果然是这样，有时数据加载很慢甚至加载不出来，我感到焦虑与害怕，害怕数据库和别人一样被黑了然后删光要钱，课都没心情听了。然后手机下了个Termius，先把服务关了。

中午回到宿舍后看了数据库发现数据完好无损，赶紧备份了一波，然后寻找问题。

内存和CPU和磁盘都挺正常，看了服务器的安全日志，那登录记录刷刷的，有人在暴力破解我的**root密码**！

由于下午还有体育课，就先直接关了服务器，然后上课去了。

下午来到实验室，重新打开服务器，又开始攻击了，气死了。用脚本封了攻击者一百多个肉鸡ip，以为可以了之后，把服务开启，通知用户可以用了。

结果用户又说太卡了，我又检查了一波，CPU、内存、磁盘正常，然后这个带宽就不太正常了，我学生机的1M/s都超了，应该是这个原因导致卡的。

用命令找了进程发现好像也没哪个占用很多呀，弄了很久还是很迷惑，攻击者的ip也明明被我加入黑名单了呀，之后还开了腾讯云的专业机阻断，还是很卡，卡到我ssh都连不太上。然后有个办法叫我改ssh的22端口，我怕操作失误连自己都直接连不上就GG了，然后就算了不这样搞了。

最后只能屁颠屁颠去找客服，他跟我说也是其他正常带宽跑满，叫我看有没有什么进程占用很多，主要是建议我临时升级带宽。

我想着：啊好家伙，开始了。但是也没啥其他办法，就去买了3天升级到3M/s的带宽，不贵，结果也是真香。

后来最多手动在安全组加了几个奇怪ip封掉，服务就完成稳定下来了，看来没有什么是加钱不能解决的。

后来过了两天攻击者还在继续冲，除了影响我带宽之外其实也没太大问题反正他进不来的，问了师兄建议说改22端口，反正重要使用期也过了，我就试试改下，还真有用，安全日志也没攻击者那些破解记录了，带宽也占用也再降了，说明攻击者也是冲22，这下直接完全被挡住，带宽也不占了。

第一次与黑客对线还是学到了很多的，也加强了我的安全防范意识



### 查看系统状态命令

下面有些命令工具需要额外安装的，直接`yum install xxx`安装就行

#### 查看服务器安全日志(动态实时)(CentOS)

```shell
tail -f /var/log/secure
```

#### 查看CPU等的使用情况(按进程)

```shell
top
```

#### 查看内存使用情况

```shell
free -h
```

#### 查看磁盘使用情况

```shell
df -hl
```

#### 查看网络带宽占用(按ip)

```shell
iftop -i eth0
jnettop
```

按Q退出

#### 查看网络带宽占用(按进程)

```shell
nethogs
```

还有防火墙工具**iptables**和**firewall-cmd**

[https://wangchujiang.com/linux-command/c/iptables.html](https://wangchujiang.com/linux-command/c/iptables.html)

[https://wangchujiang.com/linux-command/c/firewall-cmd.html](https://wangchujiang.com/linux-command/c/firewall-cmd.html)



### 编写自动化脚本封禁暴力破解登录的ip

从安全日志中读取记录，把那些多次登录失败的ip写进请求黑名单中（hosts.deny），但是这种办法只是拒绝连接，如果黑客继续攻击还是会占用带宽

先找个目录，vim建立一个脚本文件

```shell
vim /usr/local/secure_ssh.sh
```

然后编写脚本

```shell
#! /bin/bash
cat /var/log/secure|awk '/Failed/{print $(NF-3)}'|sort|uniq -c|awk '{print $2"="$1;}' > /usr/local/bin/black.txt
for i in `cat  /usr/local/bin/black.txt`
do
        IP=`echo $i |awk -F= '{print $1}'`
        NUM=`echo $i|awk -F= '{print $2}'`
        result=$(cat /etc/hosts.deny | grep $IP)
        if [[ $NUM -gt 10 ]];then
                if [[ $result = "" ]];then
                        echo "sshd: $IP" >> /etc/hosts.deny
                fi
        fi
done
```

设置定时任务

```shell
#首先打开定时任务列表
crontab -e

#添加下面这行，表示每十分钟
*/10 * * * * bash /usr/local/secure_ssh.sh

#保存退出后重启下
service crond restart 
```



### 修改sshd的22端口

这个方法可以完全把对22端口的攻击直接拒之门外，安全日志都没记录破解登录事件，带宽也不会影响，但操作过程要小心一点点

修改sshd配置文件

```shell
vim /etc/ssh/sshd_config

#Port 22     //这行去掉#号，然后在下面增加自己要新开的端口，保证新端口能用再去掉这行
```

下面新增端口

```shell
Port xxxxx
```

防火墙开启对应端口，安全组也记得开，或者先把防火墙关了，之后试了新端口能用再开

```
firewall-cmd --zone=public --add-port=xxxxx/tcp --permanent
```

重启sshd

```shell
/etc/init.d/sshd restart
#或者
service sshd restart 
```

然后重新连服务器终端试试，没问题之后可以把22直接禁掉，防火墙和安全组也封了22，对新端口放行就行



### 其他

使用强密码或者只用密钥登录

数据库多做备份

建立快照

关键数据加密



要增强安全意识呀！