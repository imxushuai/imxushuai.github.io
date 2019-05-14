---
title: 使用SpringMVC开发Restful API
photo:
  - 'https://images.xushuai.fun/restful.gif'
date: 2019-03-29 11:27:54
tags: 
  - Spring MVC
  - Restful API
categories:
  - 其他
description: Restful API的正确打开方式~
---

<center><i>Restful API的正确打开方式~</i></center>

![](https://images.xushuai.fun/restful.gif)

<!-- more -->

## 什么是Restful API

[参考：RESTful 架构详解](http://www.runoob.com/w3cnote/restful-architecture.html)



## 什么是Spring MVC

[参考：docs.spring.io](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)



## Demo

### 使用Spring mvc编写简单的Restful API

0. 依赖

   ```
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
   ```

   > 版本号使用spring io控制

1. 实体类

   ```java
   package com.xushuai.security.pojo;
   
   import lombok.Data;
   
   @Data
   public class User {
       private Integer id;
       private String username;
       private String password;
   }
   ```

2. 编写Controller

   ```java
   package com.xushuai.security.web.controller;
   
   import com.xushuai.security.pojo.User;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class UserController {
   
   
       /**
        * 获取user对象
        *
        * @param id ID并且使用正则表达式限定只能为数字
        * @return User对象
        */
       @GetMapping("/user/{id:\\d+}")
       public ResponseEntity<User> getUser(@PathVariable String id) {
           User user = new User();
           user.setId(1);
           user.setUsername("baker-1");
           user.setPassword("abc");
   
           return ResponseEntity.ok(user);
       }
   }
   ```

3. 使用MockMvc伪造浏览器进行测试

   ```java
   package com.xushuai.security.web.controller;
   
   import org.junit.Before;
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.context.annotation.Bean;
   import org.springframework.http.MediaType;
   import org.springframework.test.context.junit4.SpringRunner;
   import org.springframework.test.web.servlet.MockMvc;
   import org.springframework.test.web.servlet.MockMvcBuilder;
   import org.springframework.test.web.servlet.RequestBuilder;
   import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
   import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
   import org.springframework.test.web.servlet.setup.MockMvcBuilders;
   import org.springframework.web.context.WebApplicationContext;
   
   import static org.junit.Assert.*;
   
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class UserControllerTest {
   
       @Autowired
       private WebApplicationContext webApplicationContext;
   
       private MockMvc mockMvc;
   
       @Before
       public void setUp() throws Exception {
           mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
       }
   
       @Test
       public void getUserTest() throws Exception {
           mockMvc.perform(MockMvcRequestBuilders.get("/user/1") // 发送get请求，对应的有post,put,delete等
                   .contentType(MediaType.APPLICATION_JSON_UTF8)) // 设置contentType
                   .andExpect(MockMvcResultMatchers.status().isOk()) // 校验返回的状态码是否为200
                   .andExpect(MockMvcResultMatchers.jsonPath("$.username").value("baker-1")); // 校验返回的json对象中的username属性是否为baker-1
   
       }
   }
   ```

   > [GitHub：JsonPath](https://github.com/json-path/JsonPath)
   >
   > 可以快速定位和操作json的方法

4. 测试

![](https://images.xushuai.fun/19-1-7/17748187.jpg)

> 测试通过



### 使用JsonView控制返回的数据

1. 定义JsonView

   在实体类中自定义视图

   ```java
   package com.xushuai.security.pojo;
   
   import com.fasterxml.jackson.annotation.JsonView;
   import lombok.Data;
   
   @Data
   public class User {
   
       public interface UsernameView {
       }
   
       public interface UserView extends UsernameView {
       }
       @JsonView(UserView.class)
       privcate Integer id;
   
       @JsonView(UsernameView.class)
       private String username;
   
       @JsonView(UserView.class)
       private String password;
   }
   ```

   > 声明UsernameView接口，并在`username`字段上使用`@JsonView`注解标识该字段在UsernameView视图中展示。
   >
   > 声明UserView接口，并在`id`和`password`字段上使用`@JsonView`注解标识该字段在UserView视图中展示，因为UserView接口实现了UsernameView接口缘故，`username`字段也会在UserView视图中被展示。

2. 编写Controller，并修改之前的Restful Api

   ```java
   package com.xushuai.security.web.controller;
   
   import com.fasterxml.jackson.annotation.JsonView;
   import com.xushuai.security.pojo.User;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class UserController {
   
   
       /**
        * 获取user对象
        *
        * @param id ID并且使用正则表达式限定只能为数字
        * @return User对象
        */
       @GetMapping("/user/{id:\\d+}")
       @JsonView(User.UserView.class)
       public ResponseEntity<User> getUser(@PathVariable String id) {
           User user = new User();
           user.setId(1);
           user.setUsername("baker-1");
           user.setPassword("abc");
   
           return ResponseEntity.ok(user);
       }
   
       
       @GetMapping("/user/username/{id:\\d+}")
       @JsonView(User.UsernameView.class)
       public ResponseEntity<User> getUsername(@PathVariable String id) {
           User user = new User();
           user.setId(2);
           user.setUsername("baker-2");
           user.setPassword("abc2");
   
           return ResponseEntity.ok(user);
       }
   }
   ```

   > 使用`@JsonView`注解限制返回的数据

3. 编写测试用例

   ```java
   package com.xushuai.security.web.controller;
   
   import org.junit.Before;
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.context.annotation.Bean;
   import org.springframework.http.MediaType;
   import org.springframework.test.context.junit4.SpringRunner;
   import org.springframework.test.web.servlet.MockMvc;
   import org.springframework.test.web.servlet.MockMvcBuilder;
   import org.springframework.test.web.servlet.RequestBuilder;
   import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
   import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
   import org.springframework.test.web.servlet.setup.MockMvcBuilders;
   import org.springframework.web.context.WebApplicationContext;
   
   import static org.junit.Assert.*;
   
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class UserControllerTest {
   
       @Autowired
       private WebApplicationContext webApplicationContext;
   
       private MockMvc mockMvc;
   
       @Before
       public void setUp() throws Exception {
           mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
       }
   
       @Test
       public void getUserTest() throws Exception {
           String result = mockMvc.perform(MockMvcRequestBuilders.get("/user/1") // 发送get请求，对应的有post,put,delete等
                   .contentType(MediaType.APPLICATION_JSON_UTF8)) // 设置contentType
                   .andExpect(MockMvcResultMatchers.status().isOk()) // 校验返回的状态码是否为200
                   .andExpect(MockMvcResultMatchers.jsonPath("$.username").value("baker-1"))
                   .andReturn().getResponse().getContentAsString();// 校验返回的json对象中的username属性是否为baker-1
   
           System.out.println(result);
       }
   
       @Test
       public void getUsernameTest() throws Exception {
           String result = mockMvc.perform(MockMvcRequestBuilders.get("/user/username/1") // 发送get请求，对应的有post,put,delete等
                   .contentType(MediaType.APPLICATION_JSON_UTF8)) // 设置contentType
                   .andExpect(MockMvcResultMatchers.status().isOk()) // 校验返回的状态码是否为200
                   .andExpect(MockMvcResultMatchers.jsonPath("$.username").value("baker-2"))
                   .andReturn().getResponse().getContentAsString();// 校验返回的json对象中的username属性是否为baker-1
   
           System.out.println(result);
       }
   
   }
   ```

4. 运行两个测试方法

   - getUserTest()

     ![](https://images.xushuai.fun/19-1-7/1494678.jpg)

   - getUsernameTest()

     ![](https://images.xushuai.fun/19-1-7/47141981.jpg)

### 使用@Valid注解校验数据

1. Hibernate Validator注解介绍

| 注解                         | 含义                                                     |
| ---------------------------- | -------------------------------------------------------- |
| @Valid                       | 被注释的元素是一个对象，需要检查此对象的素有字段值       |
| @Null                        | 被注释的元素必须为null                                   |
| @NotNull                     | 被注释的元素必须不为null                                 |
| @AssertTrue                  | 被注释的元素必须为true                                   |
| @AssertFalse                 | 被注释的元素必须为false                                  |
| @Min（value）                | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max（value）                | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin（value）         | 与@Min作用相同，主要针对高精度的BigDecimal类型           |
| @DecimalMax（value）         | 与@Max作用相同，主要针对高精度的BigDecimal类型           |
| @Size（max，min）            | 被注释的元素的大小必须再指定的范围内                     |
| @digits（integer，fraction） | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| @Past                        | 被注释的元素必须是一个过去的日期                         |
| @Future                      | 被注释的元素必须是一个将来的日期                         |
| @Pattern（value）            | 被注释的元素必须符合指定的正则表达式                     |

   > `DecimalMin`和`DecimalMax`可以使用inclusive制定是否等于,为true则包含等于。
   >
   > 另见：[Hibernate Validator注解大全](https://blog.csdn.net/danielzhou888/article/details/74740817)

2. 使用Validator

   - 在实体类中加入相关注解

     ![](https://images.xushuai.fun/19-1-7/85052773.jpg)

   - 编写restful api

     ```java
         @PostMapping("/user")
         public ResponseEntity<User> addUser(@Valid @RequestBody User user, BindingResult bindingResult) {
             if (bindingResult.hasErrors()) {// 打印错误信息
                 bindingResult.getAllErrors().forEach(error -> System.out.println(error.getDefaultMessage()));
             }
             user.setId(3);
             // 创建成功返回的状态码为:201
             return ResponseEntity.status(HttpStatus.CREATED).body(user);
         }
     ```

   - 编写测试

     ```java
         @Test
         public void addUserTest() throws Exception {
             //language=JSON
             String param = "{\"username\":\"baker-3\",\"password\":null}";
             
             String result = mockMvc.perform(MockMvcRequestBuilders.post("/user")
                     .contentType(MediaType.APPLICATION_JSON_UTF8)
                     .content(param))
                     .andExpect(MockMvcResultMatchers.status().isCreated())
                     .andReturn().getResponse().getContentAsString();
     
             System.out.println(result);
         }
     ```

   - 运行

     ![](https://images.xushuai.fun/19-1-7/6534844.jpg)

     > 依然会执行创建工作，需要配合自定义异常来使用



   - 自定义异常消息

     ![](https://images.xushuai.fun/19-1-7/54703293.jpg)

   - 再次测试

     ![](https://images.xushuai.fun/19-1-7/71167951.jpg)



### 异常处理

#### 默认错误处理机制

> 运行项目

1. 访问不存在的Restful API

   - 浏览器访问

     ![](https://images.xushuai.fun/19-1-8/75771661.jpg)

   - Rest api client访问

     > 这里使用的是postman访问，一个比较好用的rest client客户端，当然也可以使用其他的，工具而已。

     ![1546962718119](https://images.xushuai.fun/1546962718119.png)

   > 可以看到这里通过浏览器和rest client访问得到的结果是不一样的。这里是Spring Boot默认的实现。这部分源码很容易理解，Spring Boot通过判断请求头中是否含有`text/html`，有的话就返回默认的错误页面，否则返回json信息。

#### 自定义异常处理

##### 自定义异常页面

- 在项目的resources目录中新增文件夹`static.error`

  ![](https://images.xushuai.fun/19-1-8/11460459.jpg)

- 在`error`文件夹中添加`404.html`文件

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>400</title>
  </head>
  <body>
      <h1>您访问的地址不存在!</h1>
  </body>
  </html>
  ```

- 重启并使用浏览器访问，跳转的页面为自定义的页面，文件命名为返回的错误状态码

  ![](https://images.xushuai.fun/19-1-9/28263142.jpg)

##### 自定义异常信息

1. 自定义异常枚举

   ```java
   package com.xushuai.security.exception;
   
   import lombok.AllArgsConstructor;
   import lombok.Getter;
   
   @Getter
   @AllArgsConstructor
   public enum ExceptionEnum {
   
       URL_NOT_FOUND(404, "您访问的地址不存在!");
   
       // 错误状态码
       private Integer code;
       // 错误消息
       private String message;
   }
   ```

2. 自定义异常

   ```java
   package com.xushuai.security.exception;
   
   import lombok.AllArgsConstructor;
   import lombok.Getter;
   import lombok.NoArgsConstructor;
   
   /**
    * 自定义异常
    */
   
   @Getter
   @AllArgsConstructor
   @NoArgsConstructor
   public class UserException extends RuntimeException {
   
       private ExceptionEnum exceptionEnum;
   }
   ```

3. 自定义异常返回对象

   ```java
   package com.xushuai.security.exception;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class ExceptionResult {
       private int status;
       private String message;
       private long timestamp;
   
       public ExceptionResult(ExceptionEnum exceptionEnum) {
           this.status = exceptionEnum.getCode();
           this.message = exceptionEnum.getMessage();
           this.timestamp = System.currentTimeMillis();
       }
   }
   ```

4. 定义Controller通知

   ```java
   package com.xushuai.security.exception;
   
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.ControllerAdvice;
   import org.springframework.web.bind.annotation.ExceptionHandler;
   
   /**
    * 定义controller通知
    */
   @ControllerAdvice
   public class ControllerExceptionHandler {
   
       /**
        * 当controller抛出UserException执行该方法
        *
        * @param e UserException
        * @return ExceptionResult
        */
       @ExceptionHandler(UserException.class)
       public ResponseEntity<ExceptionResult> commonExceptionHandler(UserException e) {
           return ResponseEntity.status(e.getExceptionEnum().getCode())
                   .body(new ExceptionResult(e.getExceptionEnum()));
       }
   }
   
   ```

5. 启动测试

   - 启动前，在UserController中新增一个controller方法，在方法中抛异常即可

     ```java
         @GetMapping("/notfound")
         public void notFound() {
             throw new UserException(ExceptionEnum.URL_NOT_FOUND);
         }
     ```

   - 测试

     ![](https://images.xushuai.fun/19-1-9/75470881.jpg)

   > 如果有新的异常信息，只需要在异常枚举中新增即可。

