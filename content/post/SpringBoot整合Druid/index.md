---
title: "[SpringBoot]整合Druid数据源"
date: 2020-10-17T10:54:00+08:00
draft: true
slug: "springboot_druid"
tags:
    - SpringBoot
categories:
    - JavaEE
---

### 添加依赖

```xml
<!-- druid数据库连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.21</version>
</dependency>

<!-- MySql数据库驱动 -->
    <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

 <!--分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>

<!-- log4j日志 -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```



### 添加配置

```yaml
spring:
  datasource:
    username: root
    password: root
    #serverTimezone=UTC解决时区的报错
    url: jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true

    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```



### 测试

编写测试类

```java
@SpringBootTest
class SpringbootQuickApplicationTests {

    @Autowired
    DataSource dataSource;

    @Test
    void contextLoads() throws SQLException {

        System.out.println(dataSource.getClass());

        Connection connection = dataSource.getConnection();
        System.out.println(connection);

        connection.close();
    }

}
```

运行后控制台出现如下druid连接池相关字样说明成功

```
2020-10-15 10:40:34.179  INFO 11352 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
DEBUG [main] - {conn-10005} pool-connect
com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@31db34da
DEBUG [main] - {conn-10005} pool-recycle
```



### 添加后台监控

项目应用目录下的cofig目录添加

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    // 后台监控
    public ServletRegistrationBean statViewServlet(){

        ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>(new StatViewServlet(),"/druid/*");

        // 存储账号密码
        HashMap<String,String> initParameters = new HashMap<>();

        // 设置后台登录账号密码
        initParameters.put("loginUsername","admin");
        initParameters.put("loginPassword","123456");

        // 谁都可以访问
        initParameters.put("allow","");

        bean.setInitParameters(initParameters); // 设置初始化参数
        return bean;

    }

    // filter
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        // 被过滤的请求
        Map<String,String> initParameters = new HashMap<>();

        initParameters.put("exclusions","*.js,*.css,/druid/*");
        bean.setInitParameters(initParameters);

        return bean;
    }

}
```

运行项目后打开`localhost:8080/druid/`出现后台监控登录页面说明成功

