---
layout: post
title: SpringBoot中统一格式输出、全局异常捕获和自定义异常（二）
categories: SpringBoot
description: springboot中统一格式输出、全局异常捕获和自定义异常（二）
keywords: Java, 统一格式输出,全局异常处理,自定义异常
---

**二、自定义异常捕获和处理**
***2.1前言***
本篇介绍自定义异常捕获，上一篇有介绍到json返回消息的[统一格式的输出](https://blog.csdn.net/weixin_40550118/article/details/106700406)，针对后面提到的问题：
`在代码中使用try…catch捕获异常并处理，虽然可以实现异常的捕获和处理，但是在结构上容易混乱，那有没有可以全局捕获异常，或者使用自定义的异常信息捕获和处理方式呢`
上文中提到的代码如下：

```java
/**
 * test2测试使用统一返回结果的形式
 * @return
 */
@RequestMapping("/test02")
public ResponseInfo test2() {

    try {
        // 模拟异常业务代码
      int i = 1/0 ;
        Map<String , Object> datamap  = new HashMap<>();
        datamap.put("username", "wshy");
        datamap.put("password", "12345678");
        return ResponseUtils.success(datamap);
    } catch (Exception e) {
        return ResponseUtils.failure();
    }
}
```
代码中模拟异常信息` int i = 1/0 `，使用try...catch捕获异常，然后使用上一篇封装的ResponseUtils.failure（）输出错误信息。下面介绍使用自定义异常类来进行异常捕获和处理。

***2.2使用自定义异常类捕获和处理***
**2.2.1**新建ResponseMsgEnum枚举类，该类与上一篇的统一格式输出的枚举类相同，因为这里将异常的类型和返回码、返回值的类型放到了同一个枚举类中，代码如下：

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

**2.2.2**新建CenterErrorException类，该类用于拦截自定义的异常，代码如下：

```java
package com.wshy.frontweb.Exception;

import org.springframework.http.HttpStatus;
import sun.rmi.transport.ObjectTable;

/**
 * 拦截自定义异常
 */
public class CenterErrorException extends RuntimeException {

        private static final long serialVersionUID = -7480022450501760611L;

    public HttpStatus getStatus() {
        return status;
    }

    public void setStatus(HttpStatus status) {
        this.status = status;
    }

    private HttpStatus status;

        /**
         * 异常码
         */
        private Object code;
        /**
         * 异常提示信息
         */
        private String msg;

    /**
     * 自定义异常，使用枚举类中的异常码和异常信息
     * @param responseMsgEnum
     */
        public CenterErrorException(ResponseMsgEnum responseMsgEnum) {
            this.code = responseMsgEnum.getCode();
            this.msg = responseMsgEnum.getMsg();
        }

    /**
     * 用户自定义异常消息内容和异常消息内容
     * @param responseMsgEnum
     * @param msg，用户自定义消息内容
     */
        public CenterErrorException(ResponseMsgEnum responseMsgEnum, String msg) {
            this.code = responseMsgEnum.getCode();
            this.msg = msg;
        }

    /**
     * 传入系统异常代码和自定义异常内容msg
     * @param httpStatus
     */
    public CenterErrorException(HttpStatus httpStatus, String msg) {
        this.status = httpStatus;
        this.code = httpStatus.value();
        this.msg = msg;
    }

    /**
     * 传入系统异常代码
     * @param httpStatus
     */
    public CenterErrorException(HttpStatus httpStatus) {
        this.status = httpStatus;
        this.code = httpStatus.value();
    }



    // get set方法

        public Object getCode() {
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
**2.2.3**新建ExceptionHandlerAdvice类，用于捕获自定义的异常，使用`@RestControllerAdvice`注解，返回json格式数据，使用`@ExceptionHandler(CenterErrorException.class)`用于捕获自定义异常，代码如下：

```java
package com.wshy.frontweb.Exception;

import com.wshy.frontweb.Utils.ResponseUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice

public class ExceptionHandlerAdvice {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    /**param
     * @return
     */
    @ExceptionHandler(CenterErrorException.class)
    //@ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR)
    public ResponseInfo handleBusinessError(CenterErrorException businessErrorException) {

        Object code = businessErrorException.getCode();
        String msg = businessErrorException.getMsg();
        HttpStatus status = businessErrorException.getStatus();

        return ResponseUtils.Exception(code,msg,status);
    }

}

```
**2.2.4**新建Controller类，代码如下：

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
**2.2.5**测试结果显示
异常未发生时：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612122816333.png)
异常发生：（`ResponseMsgEnum.UNEXPECTED_EXCEPTION`，枚举类中定义的参数为：`UNEXPECTED_EXCEPTION("501", "系统发生异常，请联系管理员！")`），这里返回的returnInfo为controller中自定义的msg，当然，也可以不传这个msg，使用枚举类中的` "系统发生异常，请联系管理员！"`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612122900388.png)