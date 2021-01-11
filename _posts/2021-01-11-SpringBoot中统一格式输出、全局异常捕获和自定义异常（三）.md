---
layout: post
title: SpringBoot中统一格式输出、全局异常捕获和自定义异常（三）
categories: SpringBoot
description: springboot中统一格式输出、全局异常捕获和自定义异常（三）
keywords: Java, 统一格式输出,全局异常处理,自定义异常
---
这里总结下SpringBoot中统一格式输出、全局异常捕获和自定义异常第一部分。

**三、全局异常捕获和处理**
**3.1前言**
本篇介绍全局异常捕获和处理方式，上两篇有介绍使用统一格式进行json格式输出和自定义异常捕获和处理，若不想使用自定义异常枚举类的方式，可以考虑全局异常捕获的方式进行异常的捕获和处理。

```java
 /**
   * test4测试使用自定义异常处理
   * @return
   */
  @RequestMapping(value = "/test04")
  public ResponseInfo test04(@RequestParam String user){

     if ("wshy".equals(user)){

          return ResponseUtils.success(user);
     }else {
         throw new CenterErrorException(ResponseMsgEnum.UNEXPECTED_EXCEPTION,"名称不为wshy!");
     }

}
```
上 一篇中，controller中有代码如上所示，在else语句中，使用throw抛出一个自定义异常，CenterErrorException()为异常处理类的构造方法，其中有两个参数，一个是自定义异常枚举类中的UNEXPECTED_EXCEPTION类型的异常，第二个为需要显示的异常信息，这里使用的是自定义的名称部位wshy！的异常信息，没有使用枚举类中定义的异常信息。
**3.2使用全局异常捕获异常并进行处理**
**3.2.1**在ExceptionHandlerAdvice异常拦截类中添加测试参数不全的异常，代码如下：

```java
/**
 * 全局异常捕获
 * @param exception
 * @return
 */
@ExceptionHandler(Exception.class)
//@ResponseStatus(HttpStatus.BAD_REQUEST)
public ResponseInfo globalRequestException(Exception exception) {
    return new ResponseInfo(ParamUtils.getErrorCode(exception), ParamUtils.getError(exception));
}
```
**3.2.2在controller中添加如下代码测试**

```java
/**
 * test6测试使用全局异常
 * pg1-如果浏览器输入内容为：http://localhost:8080/lay/test06?username=wshy,缺少password
 *      此时系统会抛出异常“Required String parameter 'password' is not present”，对捕获后的全局异常进行显示
 *      {"returnCode":"599","returnInfo":"Required String parameter 'password' is not present","data":null}
 *pg2-如果浏览器输入内容为：http://localhost:8080/lay/test06?username=wshy&password=111,用户名密码错误
 *      此时系统会抛出我们自定义的异常，“用户名错误或密码错误，请核实后登录！”
 *      {"returnCode":500,"returnInfo":"用户名错误或密码错误，请核实后登录！","data":null}
 * pg3-如果浏览器输入内容为：http://localhost:8080/lay/test06?username=wshy&password=123，正常状态
 *      此时系统相应正常
 *      {"returnCode":"0000","returnInfo":"请求成功！","data":{"password":"123","username":"wshy"}}
 * @return
 */
@RequestMapping(value = "/test06")
public ResponseInfo test06(@RequestParam String username, @RequestParam String password) throws CenterErrorException {
    if ("wshy".equals(username) && "123".equals(password)) {

        Map<String, Object> datamap = new HashMap<>();
        datamap.put("username", username);
        datamap.put("password", password);
        return ResponseUtils.success(datamap);
    }else {
        throw new CenterErrorException(HttpStatus.INTERNAL_SERVER_ERROR,"用户名错误或密码错误，请核实后登录！");
    }
}

```
test6测试使用全局异常
  **pg1-**`如果浏览器输入内容为：http://localhost:8080/lay/test06?username=wshy,缺少password（全局异常捕获）`
       此时系统会抛出异常“Required String parameter 'password' is not present”，对捕获后的全局异常进行显示
      {"returnCode":"599","returnInfo":"Required String parameter 'password' is not present","data":null}
 **pg2-**`如果浏览器输入内容为：http://localhost:8080/lay/test06?username=wshy&password=111,用户名密码错误（自定义异常捕获）`
       此时系统会抛出我们自定义的异常信息，“用户名错误或密码错误，请核实后登录！”
       {"returnCode":500,"returnInfo":"用户名错误或密码错误，请核实后登录！","data":null}
  **pg3-**`如果浏览器输入内容为：http://localhost:8080/lay/test06?username=wshy&password=123，正常状态`
     此时系统相应正常
       {"returnCode":"0000","returnInfo":"请求成功！","data":{"password":"123","username":"wshy"}}

**3.2.3测试结果**
![正常状态](https://img-blog.csdnimg.cn/20200617145429886.png)
![自定义异常捕获](https://img-blog.csdnimg.cn/20200617145451605.png)
![全局异常捕获](https://img-blog.csdnimg.cn/20200617145515613.png)
测试地址：[http://139.196.227.77:8080/ExceptionDemo/lay/test06?username=wshy&password=123](http://139.196.227.77:8080/ExceptionDemo/lay/test06?username=wshy&password=123)
