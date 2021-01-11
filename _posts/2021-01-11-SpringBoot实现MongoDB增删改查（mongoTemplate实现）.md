---
layout: post
title: SpringBoot实现MongoDB增删改查（mongoTemplate实现）
categories: SpringBoot
description: SpringBoot实现MongoDB增删改查（mongoTemplate实现）
keywords: Java, SpringBoot,MongoDB
---

这里介绍基使用SpringBoot实现MongoDB增删改查，基于mongoTemplate实现。



1.首先创建项目，目录如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401230828550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)

application.yml配置
2.pom.xml引入mongodb依赖和application.yml配置：

```java
<!-- mongodb依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401232716431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
3.创建实体类User，如下所示，注意@Document(collection = "user")注解，用于标记本实体类对应的mongodb集合，本实例为user集合。

```java
package com.ggeit.pay.entity;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "user")
public class User {
 private String name;
 private String age;
 public String getName() {
  return name;
 }
 public void setName(String name) {
  this.name = name;
 }
 public String getAge() {
  return age;
 }
 public void setAge(String age) {
  this.age = age;
 }
}
```
4.创建UserServiceInf接口

```java
package com.ggeit.pay.inf;
import java.util.Collection;
import java.util.List;
import java.util.Map;
import com.ggeit.pay.entity.User;
public interface UserServiceInf {
 void updateUserbyname(String name, String age);
 void insert(Map<String, Object> reqmap, String collectionName);
 String find();
}
```
5.创建接口实现类，使用MongoTemplate 对象中的增删改查操作

```java
@Service
public class UserServiceImpl implements UserServiceInf {
 private final static Logger logger = LoggerFactory.getLogger(UserServiceImpl.class);
   @Autowired
   private MongoTemplate mongotemplate;
 /**
  * 封装mongotemplate中的insert方法
  */
 @Override
 public void insert(Map<String,Object> reqmap,String collectionName) {
  // TODO Auto-generated method stub
//  User user = new User();
//  user.setName(name);
//  user.setAge(age);
//  logger.info("user = " + user);
  mongotemplate.insert(reqmap, collectionName);
 }
 /**
  * upsert更新，上传参数为query查询条件，update更新内容，collectionName为String类型
  */
 @Override
 public void updateUserbyname(String name,String age) {
  // TODO Auto-generated method stub
  Query query = new Query();
     query.addCriteria(Criteria.where("name").is(name));
     Update update = Update.update("age", age);
     mongotemplate.upsert(query, update, "user");
 }
 @Override
 public String find() {
  // TODO Auto-generated method stub
  //Query query = new Query(); 
  //query.addCriteria(Criteria.where("name").is("wshy"));  
  //query.addCriteria(Criteria.where("age").is("12222222222"));  
  Collection<User> findList = mongotemplate.findAll( User.class,"user");
  
  return new Gson().toJson(findList);
 }
}
```
6.创建Controller代码

```java
@RestController
@RequestMapping("/user")
public class UserController {
 private final static Logger logger = LoggerFactory.getLogger(UserController.class);
    @Autowired
    private UserServiceImpl userserviceimpl;
 /**
  * MongoDB insert
  * @param request
  * @param response
  * @throws Exception
  */
 @RequestMapping(value = "/insertuser", method = RequestMethod.GET)
 public void insert(HttpServletRequest request, HttpServletResponse response,String name,String age) throws Exception {
  
  Map<String ,Object> reqmap = new HashMap<String,Object>();
  reqmap.put("name", name);
  reqmap.put("age", age);
  String collectionName = "user";//插入哪个集合（表）
  //调用insert接口
  userserviceimpl.insert(reqmap,collectionName);
  logger.info("插入成功！");
     }
 /**
  * MongoDB update
  * @param req
  * @param resp
  */
 @RequestMapping(value = "/update", method = RequestMethod.GET)
 public void update(HttpServletRequest req,HttpResponse resp,String name,String age) {
  userserviceimpl.updateUserbyname(name,age);
  logger.info("更新成功！");
 }
 /**
  * MongoDB findall
  * @param req
  * @param resp
  */
 @RequestMapping(value = "/findall", method = RequestMethod.GET)
 public String findall(HttpServletRequest req,HttpResponse resp) {
  String str = userserviceimpl.find();
  //查出来的数据要转成json格式，不然会显示错误
  logger.info("查询成功：" + str);
  return str;
 } 
}
```
insert和update在调试时未出现错误，在调试查询时，返回的数据为list，内容总是显示为class对象，需利用`new Gson().toJson( )`进行转化。
7.测试时结果insert和update

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401232210180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
find操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401232451323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
最近刚开始玩Git版本工具，并在GitHub上传了源码，有兴趣可以下载：

[https://github.com/wshy0924/MongoDB_demo](https://github.com/wshy0924/MongoDB_demo)
