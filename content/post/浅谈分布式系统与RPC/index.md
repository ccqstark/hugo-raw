---
title: "浅谈分布式系统与RPC"
date: 2021-02-24T23:01:00+08:00
draft: true
image: dis-rpc.png
slug: "distributed_rpc"
tags:
    - RPC
categories:
    - 分布式
--- 

随着互联网的发展，Web应用与服务的规模不断扩大，才能满足不断增加的用户和需求。而原来只用一台服务器来部署的单机应用的方式已经满足不了如此大的需求。为了提高性能，单机发展为分布式架构，简单来说就是通过增加服务器的数量来弥补性能上的不足，当然同时也带来了一些问题。

### 分布式系统

**`分布式系统（distributed system）`**是由一组网络进行通信，为了完成共同的任务而协调工作的计算机节点组成的系统。分布式系统是为了用单体性能普通的机器完成单个计算机无法完成的计算、存储任务，通过**合理调度**各台计算机**共同合作**来提高性能，完成数量更多、体量更大的任务。

![%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled.png](%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled.png)

比如百度或淘宝这样体量规模都十分大的应用，为了满足如此大的用户数量和请求压力，背后肯定不止一台计算机在提供服务，而是部署了**计算机集群**，通过提升计算机的数量来提高处理数据的能力。在流量高峰时，大量的请求可以被均匀地分配给各台服务器去处理，从而避免其中一台服务器由于压力过大而宕机，这就是`负载均衡`，也就是`nginx`可以做的事情。

![%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%201.png](%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%201.png)

同时应用的各个模块也可以分别部署在不同的服务器上，比如淘宝这样的电商项目可以把**服务拆分**成商品、支付、订单、物流等不同的模块，不同模块可以部署在不同的服务器上。当一个模块部署到多台服务器上时，其中一台崩了，还有其他的服务器可以提供服务，提高了服务的稳定性与高可用。

![%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%202.png](%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%202.png)

而对于用户来说，他们都是访问一个域名来获取服务的，所以对于用户来说服务还是一个整体，而不需要知道是哪台服务器为他们提供了服务。

### RPC

下图是**Dubbo**官方文档中的一张图，说明了网站应用的架构演变

![%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%203.png](%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%203.png)

原来应用是单体应用，程序如果要调用一个函数或方法就直接调用就行了，因为都是在本地。

现在应用采用分布式架构，服务被分散到不同的服务器上，一台服务器上的程序就会遇到需要调用另一台服务器上的某个方法的情况，这个时候就叫`RPC(远程过程调用)(Remote Procedure call)`

RPC的调用过程如下图：

![%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%204.png](%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%204.png)

通过网络发送调用消息就需要先序列化，到目标服务器后成功调用对应的服务后返回，反序列化得到结果。

`Dubbo`就是一个流行的RPC框架，提供面向接口代理的高性能RPC调用、智能负载均衡、服务自动注册与发现、高度可扩展能力、运行期流量调度（灰度发布）、可视化工具等功能。

### 流动计算架构

当服务增多时，服务的管理与资源的分配成为亟待解决的问题，此时，需要增加一个调度中心基于访问压力实时管理集群容量，提高集群的利用率。用于提高机器利用率的资源调度和治理中心(SOA)(Service Oriented Architecture)就十分重要。

服务通过在注册中心进行注册，被统一的管理起来并可被发现，并对用户开放。当用户需要用到某一服务时，就去注册中心拿，注册中心就会将对应的服务提供给他。

![%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%205.png](%E6%B5%85%E8%B0%88%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E4%B8%8ERPC%20c2afcf99bd164cd7bf6ce7ec6e5de10a/Untitled%205.png)

注册中心用的比较多的是`Zookeeper`