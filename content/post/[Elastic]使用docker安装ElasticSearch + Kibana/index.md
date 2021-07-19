---
title: "[Elastic]使用docker安装ElasticSearch + Kibana"
date: 2021-01-27T21:26:00+08:00
draft: true
image: elastic-docker.png
slug: "es_docker"
tags:
    - ElasticSearch
categories:
    - 环境搭建
--- 


## 安装ElasticSearch

### 拉取镜像

```bash
docker pull elasticsearch:7.10.1
```

### 启动容器

同时挂载目录（包括配置文件和data）（挂载出来的位置自己定义）

```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d \
-v /home/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /home/es/data:/usr/share/elasticsearch/data elasticsearch:7.10.1
```

注意这里还设置了`JVM的内存`大小，默认为2G，有点大，很可能会因为内存不够而无法正常启动。可以像我这里改为256m或者其他值。

### 可能出现的错误

查看容器日志

```bash
docker logs elasticsearch
```

如果出现以下错误

```bash
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

则要修改服务器配置

```bash
vim /etc/sysctl.conf
```

添加这行

```bash
vm.max_map_count=262144
```

立即生效, 执行：

```bash
/sbin/sysctl -p
```

对挂载的宿主机data目录可能出现权限不足问题

```bash
chmod 777 [宿主机data目录]
```

### 配置跨域

到挂载出来到位置编辑配置文件

```bash
vim elasticsearch.yml
```

添加以下几行

```yaml
network.host: 0.0.0.0
discovery.type: single-node

http.cors.enabled: true
http.cors.allow-origin: "*"
```

同时安全组和防火墙记得打开对应端口

### 记得每次修改完配置都要重启

```bash
docker restart elasticsearch
```

### 浏览器访问测试

```bash
http://[IP]:9200
```

看到类似以下的json就成功了

```json
{
	"name": "8c819d377714",
	"cluster_name": "elasticsearch",
	"cluster_uuid": "-AkgwTlbS1SsjvzNtG45nw",
	"version": {
	"number": "7.10.1",
	"build_flavor": "default",
	"build_type": "docker",
	"build_hash": "1c34507e66d7db1211f66f3513706fdf548736aa",
	"build_date": "2020-12-05T01:00:33.671820Z",
	"build_snapshot": false,
	"lucene_version": "8.7.0",
	"minimum_wire_compatibility_version": "6.8.0",
	"minimum_index_compatibility_version": "6.0.0-beta1"
	},
	"tagline": "You Know, for Search"
}
```

> 如果无法访问到

如果开了安全组和防火墙的话还是无法访问到的话，看看在容器启动时是否有如下警告：

```bash
WARNING: IPv4 forwarding is disabled. Networking will not work.
```

按如下步骤操作再访问即可

```bash
vim /etc/sysctl.conf

#添加如下代码：
net.ipv4.ip_forward=1

#重启network服务
systemctl restart network

#查看是否修改成功
sysctl net.ipv4.ip_forward

#如果返回为“net.ipv4.ip_forward = 1”则表示成功了
#这时，重启容器即可。
```



## 安装elasticsearch-head

### 拉取镜像

```bash
docker pull mobz/elasticsearch-head:5
```

### 启动容器

```bash
docker run -it -d --name head -p 9100:9100 mobz/elasticsearch-head:5
```

浏览器打开

```bash
http://[ip]:9100/
```

在上面集群连接处的输入框输入elasticsearch的地址

```bash
http://[IP]:9200
```

之后点击连接，右边的`集群健康值`字样出现绿色背景代表成功连接

## 安装Kibana

### 拉取镜像

```bash
docker pull kibana:7.10.1
```

注意版本和ES的要对应

### 配置文件kibana.yml

为了挂载配置文件，我们先在本机创建一个配置文件，这里以`/home/kibana/config/kibana.yml` 为例

配置文件中写入

```yaml
server.host: '0.0.0.0'
elasticsearch.hosts: ["http://[ip地址]:9200/"]
xpack:
  apm.ui.enabled: false
  graph.enabled: false
  ml.enabled: false
  monitoring.enabled: false
  reporting.enabled: false
  security.enabled: false
  grokdebugger.enabled: false
  searchprofiler.enabled: false
```

### 启动容器

```bash
docker run -d -it \
--name kibana -p 5601:5601 \
-v /home/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.10.1
```

浏览器打开

```bash
http://[ip]:5601/
```

### 安装ik分词器

首先进入es的容器内

```bash
docker exec -it elasticsearch /bin/bash
```

使用bin目录下的elasticsearch-plugin install安装ik分词器插件（注意版本要对应）

```bash
# github官方
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.10.1/elasticsearch-analysis-ik-7.10.1.zip
```

这里可能会很慢，可以用镜像加速

```bash
# 镜像加速
bin/elasticsearch-plugin install https://github.91chifun.workers.dev//https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.10.1/elasticsearch-analysis-ik-7.10.1.zip
```

也可以选择本地下载解压完再上传到服务器，再把它移动到容器内的plugins文件夹里

**然后重启容器**

```bash
docker restart elasticsearch
```

### 在kibana中测试

```bash
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "各地校车将享最高路权"
}
```

有`ik_smart` 和 `ik_max_word` 两种模式，分别是`最粗粒度的拆分`和`最细粒度的拆分`