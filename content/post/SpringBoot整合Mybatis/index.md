---
title: "[SpringBoot]整合Mybatis"
date: 2020-10-17T10:54:00+08:00
draft: true
slug: "springboot_mybatis"
tags:
    - SpringBoot
categories:
    - JavaEE
---

以我的项目目录结构为例: `com.ccqstark.springbootquick`

### 导入依赖

```xml
<!-- springboot的mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```



### 配置

```yaml
#整合mybatis
mybatis:
  type-aliases-package: com.ccqstark.springbootquick.pojo
  mapper-locations: classpath:mybatis/mapper/*.xml
```



### 编写POJO(用了Lombok)

在com.ccqstark.springbootquick下新建目录pojo，然后新建类User.java，用于存储数据的对象（与数据库中的表对应）

```java
package com.ccqstark.springbootquick.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private int id;
    private String name;
    private String pwd;

}
```



### 编写Mapper

在com.ccqstark.springbootquick下新建目录mapper，然后新建UserMapper.java，用于写接口，CRUD函数

```java
package com.ccqstark.springbootquick.mapper;

import com.ccqstark.springbootquick.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;

import java.util.List;

// Mapper注解说明这是一个mybatis的mapper类
//@Mapper  //如果有扫描的话这里可以不用写这个注解了
@Repository
public interface UserMapper {

    List<User> queryUserList();

    User queryByUserId(int id);

    int addUser(User user);

    int updateUser(User user);

    int deleteUser(int id);
}

```

如果是以扫描的形式，就是在项目的app启动类加上注解**@MapperScan**

```java
@SpringBootApplication
// 也可以用着方式扫描，就不用一个个写@Mapper了
@MapperScan("com.ccqstark.springbootquick.mapper")
public class SpringbootQuickApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootQuickApplication.class, args);
    }

}
```



### 在XML里编写SQL语句

在resources目录下新建mybatis/mapper目录，再在下面新建xml文件，如这里的UserMapper.xml，在里面记得配好**namespace**，然后就可以写自定义SQL语句啦

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ccqstark.springbootquick.mapper.UserMapper">
<!-- namespace要配好，如上面的格式，对应源码目录中的mapper接口 -->
   <!-- 下面就是CRUD的SQL语句 -->
    <!-- id对应的就是接口定义的函数名 -->
    <select id="queryUserList" resultType="User"> // id对现有
        SELECT * FROM user
    </select>

    <!-- #{id}就是模板待填空位-->
    <select id="queryByUserId" resultType="User">
        SELECT * FROM user WHERE id = #{id}
    </select>

    <insert id="addUser" parameterType="User">
        INSERT into user (id,name,pwd) values (#{id},#{name},#{pwd})
    </insert>

    <update id="updateUser" parameterType="User">
        UPDATE user SET name=#{name},pwd=#{pwd} where id = #{id}
    </update>

    <delete id="deleteUser" parameterType="int">
        DELETE FROM user WHERE id = #{id}
    </delete>

</mapper>
```



### 调用来进行CRUD

在Controller或者Service里调用Mybatis进行CRUD，先@Autowired注入一个Mapper，然后利用这个接口类型指向的对象就可以调用接口里面的方法了。这些方法对应的sql语句操作就是在xml中定义的。

```java
@RestController
@RequestMapping("/mybatis")
public class UserController {

    @Autowired
    private UserMapper userMapper;

    @PostMapping("/query")
    public List<User> queryUserList(){
        List<User> userList = userMapper.queryUserList();

        return userList;
    }

    @PostMapping("/create")
    public String createUser(){
        userMapper.addUser(new User(4,"wuhu","5555"));

        return "ok";
    }


    @PostMapping("/update")
    public String updateUser(){
        userMapper.updateUser(new User(1,"qqq","33"));
        return "ok";
    }


    @PostMapping("/delete")
    public String deleteUser(){
        userMapper.deleteUser(4);
        return "ok";
    }
}
```

