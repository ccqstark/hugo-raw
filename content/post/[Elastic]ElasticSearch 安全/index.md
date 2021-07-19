---
title: "[Elastic]ElasticSearch 安全"
date: 2021-01-29T21:26:00+08:00
draft: true
image: x-pack.png
slug: "x_pack"
tags:
    - ElasticSearch
categories:
    - 环境搭建
--- 

ElacticSearch索引中有大量的数据，如果没有一些安全措施的话会让系统处于一个十分危险的处境，引发的相关安全事件可以看看这篇文章。

[你的Elasticsearch在"裸奔"吗？](https://juejin.cn/post/6844903780895424526#heading-14)

而ElaticSearch官方的高级安全服务是收费的，主要给企业提供。但是从6.8和7.1版本开始，基础安全功能就免费了，而且已经集成在里面不用额外安装。

除此之外诸如`Search Guard`、`ReadonlyREST`、`Nginx` 等开源免费等方法来达到安全的目的，这里介绍的是使用官方的`x-pack`的基础安全功能，对于小项目来说够用了。

本文版本为`7.10.1`

### 修改配置文件

在`elasticsearch.yml`里新增

```yaml
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

之后`重启` es

### 在es目录下执行

```bash
elasticsearch-setup-passwords interactive
```

然后输入多个用户的密码

```bash
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y

Enter password for [elastic]:
Reenter password for [elastic]:
Passwords do not match.
Try again.
Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [apm_system]:
Reenter password for [apm_system]:
Enter password for [kibana]:
Reenter password for [kibana]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Enter password for [beats_system]:
Reenter password for [beats_system]:
Enter password for [remote_monitoring_user]:
Reenter password for [remote_monitoring_user]:
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

其中`elastic`用户相当与es的root用户，之后使用es和kibana需要这个用户的密码

设置完重启一下es

### 测试

```bash
curl -GET -u elastic http://[ip]:9200/
```

发现提示输入elastic用户的密码

```bash
Enter host password for user 'elastic':
```

基本的安全就实现了，之后进一步防止暴力破解密码可以再使用`iptables`

### Kibana设置

修改`kibana.yml`

```yaml
elasticsearch.username: "elastic"
elasticsearch.password: "[密码]"
xpack:
  apm.ui.enabled: false
  graph.enabled: false
  ml.enabled: false
  monitoring.enabled: false
  reporting.enabled: false
  security.enabled: true   # 这里要打开
  grokdebugger.enabled: false
  searchprofiler.enabled: false
```

之后进入kibana进入登陆界面

![ElasticSearch%20%E5%AE%89%E5%85%A8%20d5ed4d3e7edc4879bf7a5a55b5c400dc/Untitled.png](ElasticSearch%20%E5%AE%89%E5%85%A8%20d5ed4d3e7edc4879bf7a5a55b5c400dc/Untitled.png)

用elastic用户和密码登陆即可

### 代码中配置

`Java High Level REST Client`中配置账户和密码

```java
final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
    credentialsProvider.setCredentials(AuthScope.ANY,
            new UsernamePasswordCredentials("elastic", "123456"));  //es账号密码（默认用户名为elastic）
    RestHighLevelClient client = new RestHighLevelClient(
            RestClient.builder(
                    new HttpHost("localhost", 9200, "http"))
                    .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
                        public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                            httpClientBuilder.disableAuthCaching();
                            return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
                        }
                    }));
```

`SpringBoot`的配置文件

```yaml
spring.elasticsearch.rest.username=elastic
spring.elasticsearch.rest.password=123456
```

### 修改密码

```bash
curl -H "Content-Type:application/json" -XPOST -u elastic 'http://127.0.0.1:9200/_xpack/security/user/elastic/_password' -d '{ "password" : "123456" }'
```

### 结尾

这里只是单节点示例，集群以及证书相关可以参看官方文档

[通过 TLS 加密和基于角色的访问控制确保 Elasticsearch 的安全](https://www.elastic.co/cn/blog/getting-started-with-elasticsearch-security)