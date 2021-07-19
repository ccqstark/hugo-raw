---
title: "[SpringBoot]日志与异常捕获"
date: 2021-03-18T23:01:00+08:00
draft: true
slug: "log_catch_error"
tags:
    - SpringBoot
categories:
    - JavaEE
--- 

项目中用到了之前说的日志门面slf4j+log4j，但是之后遇到了一些问题。比如程序报错没有记录在日志，记录的时间也和服务器的不一致（服务器是东八区时间），或者记录一些不需要的信息，此篇就来解决这些问题。

### slf4j与log4j

之前有一篇文章介绍了slf4j怎么整合进Springboot，slf4j是一个日志门面，和我们所用的logback、log4j这些日志框架不同，它是为这些日志框架统一调用的API，通过api来调用具体的日志实现，简化了日志的配置与使用。slf4j要与具体的日志框架搭配，我用的是log4j。

```xml
<!-- SLF4j - log4j -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.8.0-alpha2</version>
</dependency>
```

使用时只要用注解`@Slf4j` ，然后直接用`log.info()` 方法就可以记录日志了。

### 日志等级

日志分为以下几个等级：

`OFF`：最高等级的，用于关闭所有日志记录。

`FATAL`：会导致应用程序推出的严重错误。

`ERROR`：虽然发生错误事件，但仍然不影响系统的继续运行，一般也是程序的各种**Exception**，但要注意的是并不是所有异常都会导致Error，这就是下面的要说的异常捕获。打印错误和异常信息，如果不想输出太多的日志，可以使用这个级别。

`WARN`：**警告**，表明会出现潜在错误的情形，有些信息不是错误信息，只是一些提示。

`INFO`：消息在粗粒度级别上突出强调应用程序的运行过程。打印一些你感兴趣的或者重要的信息，这个可以用于生产环境中输出程序运行的一些重要信息，但是不能滥用，避免打印过多的日志。

`DEBUG`：主要用于开发过程中打印一些运行信息。但是打印但信息量过多，项目上线后不要用。

`TRACE`：跟踪日志，日志消息的粒度太细，很低的日志级别，一般不会使用。

`ALL`：最低等级的，用于打开所有日志记录。

通过修改日志配置文件`log4j.properties`来改变日志等级：

```java
log4j.rootLogger = ERROR,stdout,log //第一个参数是日志等级
```

### 错误捕获

在SpringBoot我们希望有统一的操作来捕获系统运行过程中参数的所有错误，对未预测到对错误设置友好的返回值给用户，避免返回`500`状态码。甚至可以将系统产生的报错通过邮件发送给开发者，让生产环境中的错误能得到快速直接的监测和解决。

我们用到`@RestControllerAdvice`和`@ExceptionHandler` 这两个注解

```java
@Slf4j
@RestControllerAdvice // 用于拦截异常的注解
public class ExceptionProcesser extends ResponseEntityExceptionHandler {

	@Autowired
  private MailService mailService;

	/**
	 * 全局异常捕获入日志
	 */
	// 此注解用来标示处理哪个类的异常
	@ExceptionHandler(value = Exception.class) // 表示所有的异常都会处理
	public CommonResult<String> defaultErrorHandler(Exception e) {
	
			// slf4j下的日志用法，简洁易用
	    log.error("defaultErrorHandler:", e);
			
			// 将报错栈的信息转为字符串
	    StringWriter errors = new StringWriter();
	    e.printStackTrace(new PrintWriter(errors));
			// 发送邮件给开发者
	    mailService.sendSimpleMail("xxxxx@qq.com", "项目报错", errors.toString());
	
	    return CommonResult.failed("这里可能有bug，报错信息发给ccq了，找他改bug去");
	}

	/**
   * 对一些无法try/catch的具体错误还可以专门处理
   */
  @ExceptionHandler(MultipartException.class) // 表示只处理MultipartException这个类相关的异常
  public CommonResult<String> handleUploadFileTooLargeError() {
      return CommonResult.failed(ResultCode.FILE_TOO_BIG);
  }

}
```

### 日志时差

应用部署到线上的时候可能会遇到日志的记录时间和我们的东八区时间有时差，那就是经典时区问题了，可以通过启动jar包时设置参数来解决

```java
java -jar -Duser.timezone=GMT+08 xxx.jar
```