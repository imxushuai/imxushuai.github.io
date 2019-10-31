---
title: 使用Java操作MongoDB
tags:
  - MongoDB
  - SpringDataMongoDB
categories:
  - MongoDB
abbrlink: 4883
date: 2019-04-20 16:49:11
---

<center><i>使用原生API和 SpringBootDataMongodb 操作 MongoDB</i></center>

![](https://www.imxushuai.com/img/asset/Mongodb.png)

<!-- more -->

# MongoDB原生API

## 准备工作

1. 引入依赖

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.imxushuai</groupId>
       <artifactId>mongodb-demo</artifactId>
       <version>1.0-SNAPSHOT</version>
   
       <dependencies>
           <!-- MongoDB驱动 -->
           <dependency>
               <groupId>org.mongodb</groupId>
               <artifactId>mongodb-driver</artifactId>
               <version>3.8.2</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <version>2.0.1.RELEASE</version>
           </dependency>
       </dependencies>
   
   </project>
   ```

   > 引入MongoDB驱动以及测试依赖

2. 创建测试类

   ```java
   package com.imxushua.mongodb.nativeapi;
   
   import org.junit.Test;
   
   /**
    * MongoDB 原生JAVA API
    */
   public class NativeMongoDB {
       
   }
   
   ```

## API

### 连接数据库

1. 代码

   ```java
       private final String HOST = "192.168.136.104";
       private final int PORT = 27017;
   
       private MongoDatabase mongoDatabase;
   
       @Test
       public void connectMongoDB() {
           try {
               // 创建Java Client，如果端口是27017可以省略第二个参数
               MongoClient client = new MongoClient(HOST, PORT);
               // 获取数据库，这个数据库是我提前在mongodb中创建好的
               mongoDatabase = client.getDatabase("imxushuai");
               log.info("Connect to mongoDB successfully, connect database name is [{}]", mongoDatabase.getName());
           } catch (Exception e) {
               log.error("Connect to mongoDB failure", e);
           }
       }
   ```

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617210451.png)

> 连接成功后，将@Test注解更换为@Before注解，这样运行其他测试方法的时候，就会先连接到数据库了。

### 创建集合（collection）

1. 代码

   ```java
       @Test
       public void createCollection() {
           // 创建Collection
           mongoDatabase.createCollection("imxushuai collection 1");
           mongoDatabase.createCollection("imxushuai collection 2");
       }
   ```

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617211001.png)

3. 使用GUI查看

   ![](https://www.imxushuai.com/img/asset/20190617213506.png)

   > 这里使用的是`Navicat Premium 12`，如果需要下载安装的小伙伴可以参考：[Navicat Premium 12破解]([https://www.imxushuai.com/2018/12/13/navicat-premium-12%E7%A0%B4%E8%A7%A3/](https://www.imxushuai.com/2018/12/13/navicat-premium-12破解/))

### 插入文档

1. 代码

   ```java
       @Test
       public void createDocument() {
           // 获取集合
           MongoCollection<Document> collection = mongoDatabase.getCollection("imxushuai collection 1");
           /*
            * 创建文档
            *  方式1：document.append(String key, Object value);
            *  方式2：new Document(Map<String, Object> map);
            */
           Document document = new Document();
           document.append("name", "imxushuai")
                   .append("age", "20")
                   .append("address", "China");
           // 插入多个, 插入单个 collection.insertOne(document);
           collection.insertMany(Collections.singletonList(document));
       }
   ```

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617213220.png)

3. 使用GUI查看

   ![](https://www.imxushuai.com/img/asset/20190617213407.png)

### 查询集合所有数据

1. 代码

   ```java
       @Test
       public void findAllByDocument() {
           MongoCollection<Document> collection = mongoDatabase.getCollection("imxushuai collection 1");
           FindIterable<Document> documents = collection.find();
           for (Document document : documents) {
               System.out.println(document.toString());
           }
       }
   ```

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617214051.png)

### 按条件查询数据

1. 代码

   ```java
       @Test
       public void findByParams() {
           MongoCollection<Document> collection = mongoDatabase.getCollection("imxushuai collection 1");
           // 构建查询条件按
           BasicDBObject params = new BasicDBObject("name", "imxushuai");
           // 查询
           FindIterable<Document> documents = collection.find(params);
           for (Document document : documents) {
               System.out.println(document.toString());
           }
       }
   ```

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617214555.png)

### 更新文档

1. 代码

   ```java
       @Test
       public void update() {
           MongoCollection<Document> collection = mongoDatabase.getCollection("imxushuai collection 1");
           // 修改age为25的地址为 China ChengDu
           collection.updateMany(
                   Filters.eq("age", "25"), 
                   new Document("$set", new Document("address", "China ChengDu"))
           );
       }
   ```

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617215229.png)

3. 使用GUI查看

   ![](https://www.imxushuai.com/img/asset/20190617215326.png)

### 删除文档

1. 代码

   ```java
       @Test
       public void delete() {
           MongoCollection<Document> collection = mongoDatabase.getCollection("imxushuai collection 1");
           // 删除所有符合条件的文档
           // 我这里删除address中的值包含China的数据,为了测试我又新增了一条数据
           collection.deleteMany(Filters.regex("address", Pattern.compile("^.*China.*$", Pattern.CASE_INSENSITIVE)));
       }
   ```

2. 使用GUI查看

   ![](https://www.imxushuai.com/img/asset/20190617221105.png)

   > 另外两条数据成功删除。

# Spring Data MongoDB

## 准备工作

1. 引入依赖

   ```xml
   <!-- Spring Data MongoDB -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-mongodb</artifactId>
       <version>2.0.1.RELEASE</version>
   </dependency>
   ```

2. 创建application.yml配置文件

   ```yaml
   spring:
     data:
       mongodb:
         # 主机地址
         host: 192.168.136.104
         # 端口号, 默认为27017 可省略
         port: 27017
         # 数据库名
         database: imxushuai
   ```

3. 创建测试类

   ```java
   package com.imxushua.mongodb.springdata;
   
   import org.junit.runner.RunWith;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.test.context.junit4.SpringRunner;
   
   /**
    * 使用Spring Data MongoDB操作MongoDB
    */
   @SpringBootTest
   @RunWith(SpringRunner.class)
   public class SpringDataMongoDB {
   
   }
   ```

4. 创建User实体

   ```java
   package com.imxuhsuai.mongodb.pojo;
   
   import lombok.Data;
   import org.springframework.data.annotation.Id;
   import org.springframework.data.mongodb.core.mapping.Document;
   
   @Data
   @Document(collection = "imxushuai collection 1")
   public class User {
   
       @Id
       private String _id;
       private String name;
       private String age;
       private String address;
   }
   ```

5. 创建UserRepository

   ```java
   package com.imxuhsuai.mongodb.repository;
   
   import com.imxuhsuai.mongodb.pojo.User;
   import org.springframework.data.mongodb.repository.MongoRepository;
   
   public interface UserRepository extends MongoRepository<User, String> {
   }
   ```

> 不要问为什么创建UserRepository，用过Spring Data的小伙伴都知道！！！

## Sping Boot Data MongoDB

### 创建集合

1. 代码

   ```java
       @Autowired
       private MongoTemplate mongoTemplate;
   
       @Test
       public void createCollection() {
           mongoTemplate.createCollection(User.class);
       }
   ```

2. GUI查看结果

   ![](https://www.imxushuai.com/img/asset/20190617223927.png)

### 插入文档

1. 代码

   ```java
       @Test
       public void createDocument() {
           User user = new User(null, "xushuai", "22", "SiChuan ChengDu");
           userRepository.insert(user);
       }
   ```

2. GUI查看结果

   ![](https://www.imxushuai.com/img/asset/20190617224223.png)

   > 有趣的是，保存后，多了一个`_class`的字段，这个字段是`Spring Data`帮我们生成的，用于记录实体的全类名。

### 查询集合所有数据

1. 代码

   ```java
       @Test
       public void findAllByDocument() {
           List<User> userList = userRepository.findAll();
           userList.forEach(user -> System.out.println(user.toString()));
       }
   ```

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617224853.png)

### 按条件查询数据

1. 代码

   ```java
       @Test
       public void findByParams() {
           List<User> byAddressLike = userRepository.findByAddressLike("Bei");
           byAddressLike.forEach(user -> System.out.println(user.toString()));
       }
   ```

   > 需要在`UserRepository`中定义方法：
   >
   > ​	List<User> findByAddressLike(String address);

2. 运行结果

   ![](https://www.imxushuai.com/img/asset/20190617230306.png)

### 更新文档

1. 代码

   ```java
       @Test
       public void update() {
           Optional<User> optionalUser = userRepository.findById("5d07a7330a6c462c5822adf0");
           if (optionalUser.isPresent()) {
               User user = optionalUser.get();
               user.setName("Jerry");
               userRepository.save(user);
           }
       }
   ```

2. GUI查看结果

   ![](https://www.imxushuai.com/img/asset/20190617231011.png)

### 删除文档

1. 代码

   ```java
       @Test
       public void delete() {
           userRepository.deleteById("5d07a7330a6c462c5822adf0");
       }
   ```



# 代码获取

GitHub：[Demo](<https://github.com/imxushuai/Demo-Repository>)

