---
layout: post
title: Java实现银联商务公众号+服务窗对接----支付下单
categories: 银联商务支付
description: Java实现银联商务公众号+服务窗对接----支付下单
keywords: Java, 银联商务支付,公众号支付,支付下单
---

Java实现银联商务公众号+服务窗对接----支付下单

GitLab地址：https://gitlab.com/982837387/UnionPayGetWay.git


本文对接银联商务公众号+服务窗支付，实现支付下单、[订单查询](https://blog.csdn.net/weixin_40550118/article/details/103972599)、[订单退款](https://blog.csdn.net/weixin_40550118/article/details/103974117)、[退款查询](https://blog.csdn.net/weixin_40550118/article/details/103974578)和[订单关闭](https://blog.csdn.net/weixin_40550118/article/details/103974978)几个功能，使用到银联商务的公众+服务窗支付接口规范，请自行百度下载。
## 一、接入前准备
创建maven项目，项目目录如下，各目录功能不再详细介绍，直接看接口和功能。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200114135034121.png)
## 二、接口及代码实现

### 1.支付下单接口

#### 1.1接口规范

	接口规范请查看下载的银联商务公众号+服务窗接口规范**下单接口**部分，这里不做展示。
#### 1.2 代码实现

##### 1.2.1 UnionPayOnlineController代码



```java
package com.unionpay.controller;

import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;



@Controller
@CrossOrigin
@RequestMapping("/lay")	
public class UnionPayOnlineController {
	private final static Logger logger = LoggerFactory.getLogger(UnionPayOnlineController.class);
	
	@Autowired
	private UnifiedOrderServiceimpl unionpayserviceimpl;
	
	/**
	 * 公众号支付下单接口
	 * 调用该接口，跳转到html支付页面
	 * @param request
	 * @param response
	 * @param jsonreq
	 * @return
	 * @throws UnsupportedEncodingException 
	 */
	@RequestMapping(value = "/UnifiedPay", method = RequestMethod.GET)
	public String unionpay(HttpServletRequest request, HttpServletResponse response,Map resultdatamap) throws UnsupportedEncodingException {
		Map datamap = new HashMap();	//客户端请求数据
		String key = UnionPayConstants.GGMD5KEY;					//国光MD5密钥
		
		//---------------------取到URL请求数据----------------------------------------------------
		logger.info("getQueryString:" + request.getQueryString());   //请求url的QueryString
		
			String mid = request.getParameter("mid");
			String tid = request.getParameter("tid");
			String instMid = request.getParameter("instMid");
			String msgSrc = request.getParameter("msgSrc");
			String totalAmount = request.getParameter("totalAmount");
			String payType = request.getParameter("payType");
			String notifyUrl = request.getParameter("notifyUrl");
			String returnUrl = request.getParameter("returnUrl");
			String sign = request.getParameter("sign");
			
		//--------------------step1 对请求字段值进行URLDecoder转码----------------------------
			datamap.put("mid",URLDecoder.decode(mid, "UTF-8"));
			datamap.put("tid",URLDecoder.decode(tid, "UTF-8"));
			datamap.put("instMid",URLDecoder.decode(instMid, "UTF-8"));
			datamap.put("msgSrc",URLDecoder.decode(msgSrc, "UTF-8"));
			datamap.put("totalAmount",URLDecoder.decode(totalAmount, "UTF-8"));
			datamap.put("msgType",URLDecoder.decode(payType, "UTF-8"));
			datamap.put("notifyUrl",URLDecoder.decode(notifyUrl, "UTF-8"));
			datamap.put("returnUrl",URLDecoder.decode(returnUrl, "UTF-8"));
			logger.info("URLDecoder转码后datamap = " + datamap);
		
		//-------------------------step2验证签名------------------------------------------------------
		try {
			if (!PayUtil.verifySign(datamap,key,sign)) {
				resultdatamap.put("returnInfo", "签名错误");
				resultdatamap.put("returnCode", "Bad_Sign");
				//String failureurl = unionpayserviceimpl.CreatefailureUrl(resultmap);
				//logger.info("url = " + failureurl);
				return "failure";	
			}
		//-----------------------step3  验证传参完整性-------------------------- 
			//验证公共参数完整性
			if(!PayUtil.verifyParameter(datamap)) {
				resultdatamap.put("returnCode", "Common_Value_Error");
				resultdatamap.put("returnInfo", "缺少必要公共参数");
				return "failure";
			}
			//验证接口参数完整性
			if(datamap.get("totalAmount").equals("") || datamap.get("msgType").equals("") 
					|| datamap.get("notifyUrl").equals("") || datamap.get("returnUrl").equals("") ) {
				resultdatamap.put("returnCode", "Value_Error");
				resultdatamap.put("returnInfo", "缺少必要接口参数,必传接口参数不允许为空");
				return "failure";
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			resultdatamap.put("returnCode", "System_Error");
			resultdatamap.put("returnInfo", "系统异常");
			return "failure";
		}
		//-----------------------step4 传值并生成链接跳转到支付页面--------------------------------
		//Map resultdatamap = new HashMap();
		resultdatamap.put("mid", mid);					    //商户号
		resultdatamap.put("tid", tid);						//终端号
		resultdatamap.put("instMid", instMid);				//机构商户号
		resultdatamap.put("msgSrc", msgSrc);				//消息来源
		resultdatamap.put("totalAmount", totalAmount); 		//总金额
		resultdatamap.put("msgType", payType);				//支付类型
		resultdatamap.put("notifyUrl", notifyUrl);			//支付结果通知
		resultdatamap.put("returnUrl", returnUrl);			//网页跳转地址
		resultdatamap.put("merOrderId", unionpayserviceimpl.CreateOrderID());	//生成商户订单号
		logger.info("resultdatamap = " + resultdatamap);
		//根据payType判断回调到哪个支付页面
		if(resultdatamap.get("msgType").equals("WXPay.jsPay")) {	//微信支付
			return "wxunionpay";
		}else {
			return "aliunionpay";									//支付宝支付
		}
	}
	
	/**
	 * a支付失败页面
	 * @param request
	 * @param response
	 * @param returnCode
	 * @param map
	 * @return
	 */
	@RequestMapping(value = "/failure", method = RequestMethod.GET)
	public String failure(HttpServletRequest request, HttpServletResponse response,String returnCode,Map map) {
		
		map.put("returnCode", returnCode); 
		
		return "failure";	
	}
	
	/**
	 * a根据支付页面请求，调起支付下单
	 * @param request
	 * @param response
	 * @param totalAmount
	 * @param msgType
	 * @return
	 */
	@RequestMapping(value = "/unifiedpay", method = RequestMethod.GET)
	public String index(HttpServletRequest request, HttpServletResponse response, Map map) {
		Map reqmap = new HashMap();
		//---------------------取到URL请求数据-----------------------------
		logger.info("unifiedpay getQueryString:" + request.getQueryString());    //请求url的QueryString
			String mid = request.getParameter("mid");
			String tid = request.getParameter("tid");
			String instMid = request.getParameter("instMid");
			String msgSrc = request.getParameter("msgSrc");
			String totalAmount = request.getParameter("totalAmount");
			String YtotalAmount = PayUtil.changeY2F(totalAmount);	//元转分
			logger.info("YtotalAmount = " + YtotalAmount);
			String msgType = request.getParameter("msgType");
			String notifyUrl = request.getParameter("notifyUrl");
			String returnUrl = request.getParameter("returnUrl");
			String merOrderId = request.getParameter("merOrderId");
			
			reqmap.put("mid", mid);
			reqmap.put("tid", tid);
			reqmap.put("instMid", instMid);
			reqmap.put("msgSrc", msgSrc);
			reqmap.put("totalAmount", YtotalAmount);
			reqmap.put("msgType", msgType);
			reqmap.put("notifyUrl", notifyUrl);
			reqmap.put("returnUrl", returnUrl);
			reqmap.put("merOrderId", merOrderId);
			logger.info("reqmap = " + reqmap);
		String url = "";
		try {
			url = unionpayserviceimpl.UnifiedOrder(reqmap);
			//logger.info("请求URL = " + url);
		} catch (UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
//		if(url.equals("缺少必要参数，请核实后再进行下单")) {
//			map.put("returnCode", "缺少必要参数，请核实后再进行下单");
//			return "failure";
//		}
		return "redirect:" + url; //重定向到下单页面	
	}
	
}

```
##### 1.2.2 UnionpayServiceimpl代码
本部分代码为银联商务支付下单接口服务。

```java
/**
	 * 	银联商务支付下单
	 * return map
	 * @throws UnsupportedEncodingException 
	 */
	@Override
	public String UnifiedOrder(Map map) throws UnsupportedEncodingException {
		// TODO Auto-generated method stub
		Map reqmap = new HashMap(); //请求银联商务map
			reqmap.put("mid", map.get("mid"));			//商户号
			reqmap.put("tid", map.get("tid"));			//终端号
			reqmap.put("instMid", map.get("instMid"));
			reqmap.put("msgSrc", map.get("msgSrc"));	//消息来源
			reqmap.put("msgId", "UnionPay_F001");		//自定义
			reqmap.put("msgType", map.get("msgType"));	//支付类型,前端传入
			
			//报文请求时间
			String aligetTime = PayUtil.aligetTime();
			logger.info("end_time = " + aligetTime);
			reqmap.put("requestTimestamp", aligetTime);	
			
			//商户订单号
			//reqmap.put("msgSrcId", this.msgSrcId);	//来源编号
//			String orderid = GGitUtil.createOrderID();
//			StringBuffer buff = new StringBuffer(); 
//			buff.append(this.msgSrcId);
//			buff.append(orderid);
			reqmap.put("merOrderId", map.get("merOrderId"));
			
			reqmap.put("originalAmount", map.get("totalAmount"));	//前端传入
			reqmap.put("totalAmount",map.get("totalAmount"));	//订单金额
			reqmap.put("notifyUrl", map.get("notifyUrl"));	//支付结果通知地址
			reqmap.put("returnUrl", map.get("returnUrl"));	//网页跳转地址
			
			//生成待签名字符串并进行MD5加密
			String builderSignStr = "";
			try {
				  builderSignStr = PayUtil.builderSignStr(reqmap,UnionPayConstants.MD5KEY);
				//signString = PayUtil.generateSignature(reqmap, UnionPayConstants.MD5KEY);
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			reqmap.put("sign", builderSignStr);
			logger.info("reqmap= " + reqmap);
			
			//拼接请求URL
			StringBuffer url = new StringBuffer();
			try {
				url.append("https://qr-test2.chinaums.com/netpay-portal/webpay/pay.do?");
				url.append("requestTimestamp=" + URLEncoder.encode((String) reqmap.get("requestTimestamp"), "UTF-8") +
						"&mid=" + URLEncoder.encode((String) reqmap.get("mid"), "UTF-8") + 
						"&tid="+ URLEncoder.encode((String) reqmap.get("tid"), "UTF-8") + 
						"&instMid=" + URLEncoder.encode((String) reqmap.get("instMid"), "UTF-8") + 
						"&msgSrc=" + URLEncoder.encode((String) reqmap.get("msgSrc"), "UTF-8") + 
						"&merOrderId=" + URLEncoder.encode((String) reqmap.get("merOrderId"), "UTF-8") + 
						"&totalAmount=" + URLEncoder.encode((String) reqmap.get("totalAmount"), "UTF-8") + 
						"&msgId=" + URLEncoder.encode((String) reqmap.get("msgId"), "UTF-8") + 
						"&msgType=" + URLEncoder.encode((String) reqmap.get("msgType"), "UTF-8") + 
						"&originalAmount=" + URLEncoder.encode((String) reqmap.get("originalAmount"), "UTF-8") + 
						"¬ifyUrl=" + URLEncoder.encode((String) reqmap.get("notifyUrl"), "UTF-8") + 
						"&returnUrl=" + URLEncoder.encode((String) reqmap.get("returnUrl"), "UTF-8") + 
						"&sign=" + URLEncoder.encode((String) reqmap.get("sign"), "UTF-8"));
				logger.info("银联商务下单url = " + url);
			} catch (Exception e) {
				// TODO: handle exception
				return "缺少必要参数，请核实后再进行下单";
			}
		return url.toString();
	}
```
#### 1.3 支付结果截图
在下单接口中，我先在写了一个支付的html页面，用于显示支付信息，用户点击“去支付”按钮调起图二支付页面，支付完成时，用户点击“完成”按钮，跳转到程序中指定的returnurl地址中，支付结果通知到notifyurl地址中。
支付下单参数由银联商务分配的参数填入。

访问地址：http://172.20.10.2:8080/UnionPay/lay/UnifiedPay(本地环境访问)
支付连接为：`http://172.20.10.2:8080/UnionPay/lay/UnifiedPay?mid=******&tid=******&instMid=******&msgSrc=******&payType=WXPay.jsPay&totalAmount=0.11¬ifyUrl=http%3A%2F%2F******&returnUrl=http%3A%2F%2F******&sign=B2AD890DD4636C9A9AEA03F265BE695A`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200114140501743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_0#pic_center =300x600)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200114141451777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU1MDExOA==,size_16,color_FFFFFF,t_70#pic_center =300x600)根据博友的需求，这里增加下银商验签部分代码

```c
 /**
   * 生成 MD5
   *
   * @param data
   *            待处理数据
   * @return MD5结果
   */
  public static String MD5(String data) throws Exception {
   java.security.MessageDigest md = MessageDigest.getInstance("MD5");
   byte[] array = md.digest(data.getBytes("UTF-8"));
   StringBuilder sb = new StringBuilder();
   for (byte item : array) {
    sb.append(Integer.toHexString((item & 0xFF) | 0x100).substring(1, 3));
   }
   return sb.toString().toUpperCase();
  }

 /**
  * 生成待签名数据
  * @param params
  * @return
  * @throws Exception 
  */
    public static String builderSignStr(Map<String, Object> params,String md5key) throws Exception {
         Set<String> keySet = params.keySet();
         List<String>keyList = new ArrayList<String>(keySet);
         Collections.sort(keyList);
         StringBuilder sb = new StringBuilder();
         for (String key : keyList) {
             sb.append(key);
             sb.append("=");
             sb.append(params.get(key));
             sb.append("&");
         }
         sb.deleteCharAt(sb.length() - 1); //去掉最后一个&
         sb.append(md5key);
         logger.info("builderSignStr= " + sb.toString());
         logger.info("验证sign:" + MD5(sb.toString()).toUpperCase());
         return MD5(sb.toString()).toUpperCase();
     }
```
