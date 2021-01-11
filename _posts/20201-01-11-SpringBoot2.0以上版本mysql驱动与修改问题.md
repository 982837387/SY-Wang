---
layout: post
title: SpringBoot2.0以上版本mysql驱动与修改问题
categories: SpringBoot
description: SpringBoot2.0以上版本mysql驱动与修改问题
keywords: Java, SpringBoot，MySQL
---

最近在部署项目到服务器的时候，出现报错`Cannot load driver class: com.mysql.jdbc.Driver`，意为无法加载mysql驱动。
解决：更改项目mysql驱动依赖版本。

`在网上查阅资料知道，springboot2.0以上版本，mysql-connector-java默认使用的是8.0以上版本`，查看服务器项目的mysql版本为5.1.41，因此，需要将驱动版本改为5.1.41。

默认版本：

```java
    <dependency>
     <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <scope>runtime</scope>
   </dependency>  
```
修改后版本：

```java
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.41</version>
</dependency>
```
修改后版本![在这里插入图片描述](https://img-blog.csdnimg.cn/20200624093853204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
对应修改配置yml 文件：`driver-class-name: com.mysql.jdbc.Driver`（mysql5及以下版本）

修改前版本
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200624094205529.png)
对应修改配置yml 文件：`driver-class-name: com.mysql.cj.jdbc.Driver`（mysql6及以上版本）