---
layout: post
title: sql中单字段模糊查询多个匹配字段
categories: SQL
description: sql中单字段模糊查询多个匹配字段。
keywords: SQL,模糊查询
---
今天被同事问道SQL模糊查询多个匹配字段的问题，我给的答案是使用or进行匹配，这里总结下常用的几种模糊匹配多个字段的方法。

### 方式一
这里是查询post中包含u和r的所有记录
使用or
~~~sql
select * from
sys_post
where
post_code like '%u%' or post_code like '%r%' 
~~~
### 方式二
使用union
```sql
select * from
sys_post
where
post_code like '%u%' 
union
select * from
sys_post
where
post_code like '%r%' 
```
### 方式三
使用正则表达式，REGEXP

```sql
select * from
sys_post
where
post_code regexp 'u|r'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210104181825235.png)
### 方式四
当模糊查询的字段比较多时，可以选择将待匹配的字段存放在另一张表B中，在Sql server中可以使用以下语句进行查询
~~~sql
select * from A,B where charindex(B.N,A.N) >0 
~~~
