---
title: "Dubbo + ZooKeeper 基础入门"
date: 2021-02-24T23:01:00+08:00
draft: true
image: dubbo-zookeeper.png
slug: "dubbo_zookeeper"
tags:
    - Dubbo
    - ZooKeeper
categories:
    - 分布式
--- 


## 简介

`Dubbo`原本是阿里的开源框架，有很多著名厂商都在用。但在14年停更，之后Spring Cloud大红大紫，Dubbo终于在17年再度更新，并在18年合并当当网的基于它开发出的DubboX推出了2.6版本。之后在18年除夕夜，阿里正式将Dubbo捐献给了著名开源组织Apache，成为Apache众多开源项目之一。

[Apache Dubbo](https://dubbo.apache.org/zh/)

`ZooKeeper` 也是 Apache 软件基金会的一个软件项目，它为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。

ZooKeeper 的架构通过冗余服务实现高可用性。

Zookeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

一个典型的分布式数据一致性的解决方案，分布式应用程序可以基于它实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

[Apache ZooKeeper](https://zookeeper.apache.org/)

## 安装

使用Dubbo引入相关依赖即可，下面具体实践会涉及。

ZooKeeper可以去官网下载安装，这里我还是用`docker`在Linux服务器上安装。

```bash
# 拉取镜像
docker pull zookeeper

# 启动
docker run -d \
-p 2181:2181 \
-v /home/zookeeper/data/:/data/ \
--name=zookeeper  \
--privileged zookeeper

# 如果想运行自带的客户端可以：
docker exec -it zookeeper bash
cd bin
./zkCli.sh
# 之后就可以使用相关命令了
```

安装ZooInspector来可视化查看zookeeper

[](https://issues.apache.org/jira/secure/attachment/12436620/ZooInspector.zip；)

```bash
# 下载后进入build目录运行jar包，输入zookeeper的地址即可连接
java -jar zookeeper-dev-ZooInspector.jar
```

![Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled.png](Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled.png)

## 使用dubbo-admin可视化监控服务

`dubbo-admin`是一个Springboot项目，可以监控我们注册到注册中心到服务。

到github上下载

[apache/dubbo-admin](https://github.com/apache/dubbo-admin)

解压后在application.properties中修改zookeeper地址

```java
dubbo.registry.address=zookeeper://[ip]:2181
```

之后用mvn命令打包再运行jar包即可，也可以直接在idea里打包

运行后浏览器 打开`localhost:7001` ，输入账号密码默认都为root，来到主页

![Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled%201.png](Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled%201.png)

中间的搜索框可以搜索服务、应用、ip，菜单栏上也有各种监控服务的方式。

## SpringBoot Demo

用idea建立一个空项目，然后添加两个Modules，都为Springboot项目。

一个作为服务提供者provider，一个作为服务消费者consumer

两者都添加以下maven依赖

```xml
<!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.7.8</version>
</dependency>

<!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
    <exclusions>
        <exclusion>
            <artifactId>zookeeper</artifactId>
            <groupId>org.apache.zookeeper</groupId>
        </exclusion>
    </exclusions>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.12.0</version>
    <exclusions>
        <exclusion>
            <artifactId>zookeeper</artifactId>
            <groupId>org.apache.zookeeper</groupId>
        </exclusion>
    </exclusions>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.12.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.2</version>
    <exclusions>
        <exclusion>
            <artifactId>log4j</artifactId>
            <groupId>log4j</groupId>
        </exclusion>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

其中`Curator`是zookeeper分布式协调服务的java客户端库，它包装了一系列操作zk的高级API和实用库，是的操作zk变得更加容易和可靠。

### 在provider项目中创建并注册服务

包目录都是就直接com.xxx，不要多一级，然后在provider项目的com.xxx路径下创建一个service目录，写一个简单的服务接口

```java
package com.ccqstark.service;

public interface TicketService {
    String getTicket();
}
```

然后是其实现

```java
import com.ccqstark.service.TicketService;
import org.apache.dubbo.config.annotation.DubboService;
import org.springframework.stereotype.Component;

@DubboService // 服务注册注解
@Component
public class TicketServiceImpl implements TicketService {

    @Override
    public String getTicket(){
        return "dubbo+zookeeper!";
    }

}
```

其中注解`@DubboService` 用来把该服务注册到zookeeper中

修改provider项目的application.properties

```java
# 服务应用名称
dubbo.application.name=provider-server
# 注册中心地址
dubbo.registry.address=zookeeper://[ip]:2181
# 被注册的服务
dubbo.scan.base-packages=com.ccqstark.service
```

然后启动项目，就可以在之前安装的dubbo中发现该服务了

![Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled%202.png](Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled%202.png)

### 在consumer项目中调用服务

配置文件中增加配置

```java
# 消费者暴露的服务应用名称
dubbo.application.name=consumer-server
# 注册中心地址
dubbo.registry.address=zookeeper://[ip]:2181
```

同样创建com.xxx.service包，把provider里的TicketService的接口文件复制过来

创建一个调用此服务的例子

```java
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @DubboReference // 获取注册中心中的服务
    TicketService ticketService;

    public void buyTicket() {
        String ticket = ticketService.getTicket();
        System.out.println("调用provider的服务：" + ticket);
    }
    
}
```

注解`@DubboReference` 用于获取注册中心的服务

写个单元测试

```java
@SpringBootTest
class ConsumerServerApplicationTests {

    @Autowired
    UserService userService;

    @Test
    void contextLoads() {
        userService.buyTicket();
    }

}
```

运行之后就可以发现consumer项目调用provider的服务成功！

![Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled%203.png](Dubbo%20+%20ZooKeeper%20%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%20bdbb946b7fbd405f84bb1486e4365746/Untitled%203.png)

最基本的服务注册发现与rpc调用大概是demo这样。

参考自：
[bilibili](https://www.bilibili.com/video/BV1PE411i7CV?p=60)