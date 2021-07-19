---
title: "[SpringBoot]整合SLF4J-log4j"
date: 2020-10-17T10:54:00+08:00
draft: true
slug: "springboot_slf4j-log4j"
tags:
    - SpringBoot
categories:
    - JavaEE
---

### 导入依赖

```xml
<!-- SLF4j - log4j -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.8.0-alpha2</version>
</dependency>
```

然后要在IDEA下载插件Maven Helper中把logback相关的包给Exclude，否则会出现冲突



### 配置

log4j.properties中配置

```properties
# rootLogger参数分别为：根Logger级别，输出器stdout，输出器log
log4j.rootLogger = info,stdout,log

# 输出信息到控制台
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = %d [%-5p] %l %rms: %m%n

# 输出DEBUG级别以上的日志到D://log/debug.log，这个是日志文件存放的路径，根据时间情况进行设置
log4j.appender.log = org.apache.log4j.DailyRollingFileAppender
log4j.appender.log.DatePattern = '.'yyyy-MM-dd
log4j.appender.log.File = D://log/debug.log
log4j.appender.log.Encoding = UTF-8
#log4j.appender.log.Threshold = INFO
log4j.appender.log.layout = org.apache.log4j.PatternLayout
log4j.appender.log.layout.ConversionPattern = %d [%-5p] (%c.%t): %m%n
```



### 测试

编写测试类，使用`@Slf4j`注解之前确保使用了lombok

```java
package com.ccqstark.springbootquick;

import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

@Slf4j
public class LoggerTest {

//    private static final Logger log = LoggerFactory.getLogger(LoggerTest.class);

    @Test
    public void TestSLF4j(){
        log.info("Current Time: {}", System.currentTimeMillis());
        log.info("Current Time: " + System.currentTimeMillis());
        log.info("Current Time: {}", System.currentTimeMillis());
        log.trace("trace log");
        log.warn("warn log");
        log.debug("debug log");
        log.info("info log");
        log.error("error log");
    }
}
```

运行后输出以下说明成功

```
2020-10-15 16:36:45,459 [INFO ] com.ccqstark.springbootquick.LoggerTest.TestSLF4j(LoggerTest.java:13) 0ms: Current Time: 1602751005450
2020-10-15 16:36:45,464 [INFO ] com.ccqstark.springbootquick.LoggerTest.TestSLF4j(LoggerTest.java:14) 5ms: Current Time: 1602751005464
2020-10-15 16:36:45,465 [INFO ] com.ccqstark.springbootquick.LoggerTest.TestSLF4j(LoggerTest.java:15) 6ms: Current Time: 1602751005465
2020-10-15 16:36:45,466 [WARN ] com.ccqstark.springbootquick.LoggerTest.TestSLF4j(LoggerTest.java:17) 7ms: warn log
2020-10-15 16:36:45,466 [INFO ] com.ccqstark.springbootquick.LoggerTest.TestSLF4j(LoggerTest.java:19) 7ms: info log
2020-10-15 16:36:45,468 [ERROR] com.ccqstark.springbootquick.LoggerTest.TestSLF4j(LoggerTest.java:20) 9ms: error log
```



### 用法

添加注解`@Slf4j`（确保使用了lombok）

然后如测试类中`log.info`或其他类型的日志便可以使用了