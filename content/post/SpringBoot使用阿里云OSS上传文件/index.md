---
title: "[SpringBoot]使用阿里云OSS上传文件"
date: 2020-10-17T10:54:00+08:00
draft: true
slug: "springboot_oss"
tags:
    - SpringBoot
categories:
    - JavaEE
---

### 开通服务

登录阿里云，开通OSS服务，默认按量计费，为了业务稳定可以购买包月包年的资源包。



### 准备工作

创建Bucket，如果是为了作为网站的静态资源存储供用户访问的话把权限设为`公共读`，填写信息后创建成功，可以在Bucket下新建目录什么的。

单独创建一个RAM子用户用来调用API，选择编程访问，创建成功后一定要把`AccessKeyID`和`AccessKeySecret`等重要信息记下来，后面配置文件要用到。

然后要给这个子用户添加权限`AliyunOSSFullAccess`



### Maven依赖

```xml
<!-- OSS -->
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.4.2</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.4.1</version>
</dependency>
```



### 配置文件

`endpoint`就是在存储桶的概览里地域节点，填外网访问那个就行

`url`填资源访问的URL的前面部分（填到.com/）

`accessKeyId`和`accessKeySecret`就是创建子用户时那个

`bucketName`就是存储桶的名字

```yaml
# 阿里云oss
oss:
  endpoint: *
  url: *
  accessKeyId: *
  accessKeySecret: *
  bucketName: *
```



### 配置类

项目的config目录下新建OSS的配置类

```java
package com.ccqstark.springbootquick.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;

import java.io.Serializable;

/**
 * @Description: 阿里云 OSS 配置信息
 * @Author: ccq
 * @Date: 2020/10/16
 */
@Component //注册bean
@Data
@Configuration
@ConfigurationProperties(prefix = "oss")
public class OSSConfig implements Serializable {

    private String endpoint;
    private String url;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;

}
```



### 上传文件工具类

项目的util目录下新建这上传工具类

```java
package com.ccqstark.springbootquick.util;

import com.aliyun.oss.ClientConfiguration;
import com.aliyun.oss.OSSClient;
import com.aliyun.oss.common.auth.DefaultCredentialProvider;
import com.ccqstark.springbootquick.config.OSSConfig;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.util.UUID;

/**
 * @Description: 阿里云 oss 上传工具类(高依赖版)
 * @Author: ccq
 * @Date: 2020/10/17
 */
public class OSSBootUtil {

    private OSSBootUtil(){}

    /**
     * oss 工具客户端
     */
    private volatile static OSSClient ossClient = null;

    /**
     * 上传文件至阿里云 OSS
     * 文件上传成功,返回文件完整访问路径
     * 文件上传失败,返回 null
     *
     * @param ossConfig oss 配置信息
     * @param file 待上传文件
     * @param fileDir 文件保存目录
     * @return oss 中的相对文件路径
     */
    public static String upload(OSSConfig ossConfig, MultipartFile file, String fileDir){
        // 初始化客户端
        initOSS(ossConfig);
        // 文件URL
        StringBuilder fileUrl = new StringBuilder();

        try {
            String suffix = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf('.'));
            String fileName = System.currentTimeMillis() + "-" + UUID.randomUUID().toString().substring(0,18) + suffix;
            if (!fileDir.endsWith("/")) {
                fileDir = fileDir.concat("/");
            }
            fileUrl = fileUrl.append(fileDir + fileName);

            // 上传文件到指定的存储空间，并将其保存为指定的文件名称
            ossClient.putObject(ossConfig.getBucketName(), fileUrl.toString(), file.getInputStream());

        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
        fileUrl = fileUrl.insert(0,ossConfig.getUrl());
        return fileUrl.toString();
    }

    /**
     * 初始化 oss 客户端
     * @param ossConfig
     * @return
     */
    private static OSSClient initOSS(OSSConfig ossConfig) {
        if (ossClient == null ) {
            synchronized (OSSBootUtil.class) {
                if (ossClient == null) {
                    ossClient = new OSSClient(ossConfig.getEndpoint(),
                            new DefaultCredentialProvider(ossConfig.getAccessKeyId(), ossConfig.getAccessKeySecret()),
                            new ClientConfiguration());
                }
            }
        }
        return ossClient;
    }
}
```



### 服务层

在项目的service目录下新建服务层接口

```java
package com.ccqstark.springbootquick.service;

import com.ccqstark.springbootquick.model.ApiResult;
import org.springframework.web.multipart.MultipartFile;

/**
 * @Description: 公共业务
 * @Author: ccq
 * @Date: 2020/10/17
 */
public interface CommonService {

    /**
     * 上传文件至阿里云 oss
     *
     * @param file
     * @param uploadKey
     * @return
     * @throws Exception
     */
    ApiResult uploadOSS(MultipartFile file, String uploadKey) throws Exception;

}
```

新建实现类

```java
package com.ccqstark.springbootquick.service;

import com.ccqstark.springbootquick.config.OSSConfig;
import com.ccqstark.springbootquick.model.ApiResult;
import com.ccqstark.springbootquick.util.OSSBootUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.util.HashMap;
import java.util.Map;

/**
 * @Description: 公共业务具体实现类
 * @Author: ccq
 * @Date: 2020/10/17
 */
@Service("commonService")
public class CommonServiceImpl implements CommonService {

    @Autowired
    private OSSConfig ossConfig;

    /**
     * 上传文件至阿里云 oss
     *
     * @param file
     * @param uploadKey
     * @return
     * @throws Exception
     */
    @Override
    public ApiResult uploadOSS(MultipartFile file, String uploadKey) throws Exception {

        // 高依赖版本 oss 上传工具
        String ossFileUrlBoot = null;
        ossFileUrlBoot = OSSBootUtil.upload(ossConfig, file, "image/"); // 注意这里填写的是存储桶中你要存放文件的目录

        Map<String, Object> resultMap = new HashMap<>(16);
        resultMap.put("ossFileUrlBoot", ossFileUrlBoot);

        return new ApiResult(200, resultMap);
    }
}
```



### Controller上传测试

```java
package com.ccqstark.springbootquick.controller;

import com.ccqstark.springbootquick.model.ApiResult;
import com.ccqstark.springbootquick.service.CommonService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

/**
 * @Description: 上传文件
 * @Author: ccq
 * @Date: 2020/10/17
 */
@Slf4j
@RestController
@RequestMapping("/upload")
public class UploadController {

    @Autowired
    private CommonService commonService;


    /**
     * 上传文件至阿里云 oss
     *
     * @param file
     * @param uploadKey
     * @return
     * @throws Exception
     */
    @RequestMapping(value = "/oss", method = {RequestMethod.POST}, produces = {MediaType.APPLICATION_JSON_VALUE})
    public ResponseEntity<?> uploadOSS(@RequestParam(value = "file") MultipartFile file, String uploadKey) throws Exception {
        ApiResult apiResult = commonService.uploadOSS(file, uploadKey);

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        return new ResponseEntity<>(apiResult, headers, HttpStatus.CREATED);
    }


}
```

Postman工具，用POST请求，在form-data中用file字段对应图片或其他类型的文件，然后请求接口

返回的URL在浏览器中可以用公网访问说明成功了！