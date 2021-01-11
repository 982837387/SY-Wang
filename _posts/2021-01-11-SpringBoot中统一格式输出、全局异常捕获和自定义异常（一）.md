---
layout: post
title: SpringBoot中统一格式输出、全局异常捕获和自定义异常（一）
categories: SpringBoot
description: springboot中统一格式输出、全局异常捕获和自定义异常（一）
keywords: Java, 统一格式输出,全局异常处理,自定义异常
---

**一、统一格式输出**
***1.1使用统一格式前***
在工作中，经常会要求统一输出格式，例如以返回为json格式数据为例，有以下输出格式：

```c
{    
	"returnCode": "0000",    
	"returnInfo": "信息更新成功"，
	"data": {
		"username"： "wshy",
		"password": "12345678",
		"telnum": "18361393088",
		"address": "徐州市贾汪区"
	}
}
```
或者有错误信息如下：

```c
{    
	 "returnCode": "9999",    
	 "returnInfo": "信息更新失败"，
	 "data": null
}
```
你的代码可能是这样的

```java
@RestController
@RequestMapping(value = "/lay", method = RequestMethod.GET)
public class TestController {
    /**
     * test1测试无统一返回结果时返回数据的形式和try...catch捕获异常
     * @return
     */
    @RequestMapping("/test01")
    public JSONObject test1() {
        JSONObject result = new JSONObject();
        Map<String,Object> datamap = new HashMap<>();
        try {
            // 业务逻辑代码
            result.put("returnCode", "0000");
            result.put("returnInfo", "信息更新成功！");

            datamap.put("username", "wshy");
            datamap.put("password", "12345678");

            result.put("data", datamap);
        } catch (Exception e) {
            result.put("ReturnCode", "500");
            result.put("ReturnInfo", "系统异常，请联系管理员！");
        }
        return result;
    }
}
```
这种格式输出在结果上是没有错的，但是试想下，如果一个controller中有很多个请求，每个请求的参数都要写这种格式返回结果，无疑是增加无用的工作量，而且很容易会导致出错，在代码整洁性上也欠佳。所以，使用统一的格式输出可以很好的解决以上问题。

***1.2使用统一格式输出***
1.2.1新建ResponseInfo类，用于定义ReturnCode、ReturnInfo和data，代码如下：

```java
package com.wshy.frontweb.Exception;

import org.springframework.http.HttpStatus;

public class ResponseInfo<T> {

    /**
     * 返回码
     */
    protected Object returnCode;
    /**
     * 返回码描述
     */

    protected Object returnInfo;
    /**
     * 返回数据,不/知道返回结果具体类型，这里使用泛型
     */
    private T data;

    /**
     * 若没有数据返回，默认状态码为 0，提示信息为“操作成功！”
     */
    public ResponseInfo() {
        this.returnCode = "0";
        this.returnInfo = "操作成功！";
    }

    /**
     * 若没有数据返回，可以人为指定状态码和提示信息
     */
    public ResponseInfo(Object code, String msg) {
        this.returnCode = code;
        this.returnInfo = msg;
    }

    /**
     * 操作成功，返回枚举类中normal_return的状态码和状态信息，data为数据部分
     * @param data
     */
    public ResponseInfo(T data) {
        this.data = data;
        this.returnCode = "0";
        this.returnInfo = "1";
    }

    //get 和 set 方法
    public Object getReturnCode() {
        return returnCode;
    }

    public void setReturnCode(String returnCode) {
        this.returnCode = returnCode;
    }

    public Object getReturnInfo() {
        return returnInfo;
    }

    public void setReturnInfo(String returnInfo) {
        this.returnInfo = returnInfo;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public ResponseInfo(Object code, Object msg, T data) {
        this.returnCode = code;
        this.returnInfo = msg;
        this.data = data;
    }


}


```
1.2.2新建ResponseMsgEnum枚举类，用于定义返回码和返回信息，代码如下：

```java
package com.wshy.frontweb.Exception;

public enum ResponseMsgEnum {
    /**
     * 正常返回
     */
    NORMAL_RETURN("0000","请求成功！"),
    /**
     * 请求失败
     */
    INNORMAL_RETURN("9999","请求失败！"),
    /**
     * 参数异常
     */
    PARMETER_EXCEPTION("101", "参数异常!"),
    /**
     * 等待超时
     */

    SERVICE_TIME_OUT("102", "服务超时！"),

    /**
     * 参数过大
     */
    PARMETER_BIG_EXCEPTION("903", "内容不能超过2字，请重试!"),
    /**
     * 数据库操作失败
     */
    DATABASE_EXCEPTION("400", "数据库操作异常，请联系管理员！"),
    /**
     * 500 : 一劳永逸的提示也可以在这定义
     */
    UNEXPECTED_EXCEPTION("501", "系统发生异常，请联系管理员！");
    // 还可以定义更多的业务异常

    /**
     * 消息码
     */
    private String code;
    /**
     * 消息内容
     */
    private String msg;

    private ResponseMsgEnum(String code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

```
1.2.3新建ResponseUtils类，为了让返回内容更加简洁，代码如下：

```java
package com.wshy.frontweb.Utils;

import com.wshy.frontweb.Exception.ResponseInfo;
import com.wshy.frontweb.Exception.ResponseMsgEnum;
import org.springframework.http.HttpStatus;


public class ResponseUtils {
    /**
     * 成功时返回
     * @param object为data部分的内容
     * @return
     */
    public static ResponseInfo success(Object object) {

        return new ResponseInfo(ResponseMsgEnum.NORMAL_RETURN.getCode(),ResponseMsgEnum.NORMAL_RETURN.getMsg(),object);
    }

    /**
     * 失败时返回
     * @param
     * @return
     */
    public  static  ResponseInfo failure() {

        return  new ResponseInfo(ResponseMsgEnum.INNORMAL_RETURN.getCode(),ResponseMsgEnum.INNORMAL_RETURN.getMsg());
    }

    /**
     * 异常时返回
     * @param
     * @return
     */
    public  static  ResponseInfo Exception(Object code, String msg, HttpStatus status) {

        return  new ResponseInfo(code,msg,status);
    }

}

```
1.2.4controller中使用ResponseUtils返回信息，代码如下：

```java
/**
 * test2测试使用统一返回结果的形式
 * @return
 */
@RequestMapping("/test02")
public ResponseInfo test2() {

    try {
        // 模拟异常业务代码，datamap为data输出内容
        Map<String , Object> datamap  = new HashMap<>();
        datamap.put("username", "wshy");
        datamap.put("password", "12345678");
        return ResponseUtils.success(datamap);
    } catch (Exception e) {
        return ResponseUtils.failure();
    }
}
```
1.2.5结果显示
成功结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612013233690.png)
失败结果：
在代码try...catch中增加一个异常，`int i = 1/0 ;`，捕获异常时，返回ResponseUtils.failure();
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612013456214.png)
1.2.6问题
在代码中使用try...catch捕获异常并处理，虽然可以实现异常的捕获和处理，但是在结构上容易混乱，那有没有可以全局捕获异常，或者使用自定义的异常信息捕获和处理方式呢？见下一篇介绍。