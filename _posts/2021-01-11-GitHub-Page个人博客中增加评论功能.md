---
layout: post
title: GitHub Page个人博客中增加评论功能
categories: GitHub 
description: GitHub Page个人博客中评论功能
keywords: GitHub, Git
---

最近通过fork大佬的github和jekyll模板搭了一个博客，但在使用过程中，发现博文的评论功能不能使用，这里记录下个人博客中评论功能得我使用流程及问题。

#### 1.出现问题
首先看一下初始博客搭建好之后的评论功能，显示“Error : Not Found”，这个问题主要是由于未能正确找到仓库 repo，检查一下你的仓库是否配置正确。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111174701994.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
#### 2.解决办法
#####  2.1新建评论仓库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111175410362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
##### 2.2 Setting中设置开启issues
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111175511694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
##### 2.3设置Auth权限

```csharp
https://github.com/settings/applications/new
```
访问上面链接，填写如下信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111180211971.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
以下是我填写的信息，其中SY-Wang是我的仓库名称
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111180318607.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)
##### 2.4获取id和secret
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111180502403.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)

##### 2.5填写到_config.yml
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210112092157279.png)
owner：填写你登录到github的账号
repo：填写之前用于记录评论的日志仓库名称
clientID：填写上面申请的clientID
clientSercret：填写上面申请的clientSecret

##### 2.6测试评论
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111180926879.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)

log-comments中查看评论信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111180840673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70)







