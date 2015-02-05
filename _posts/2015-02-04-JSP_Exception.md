---
layout: post
title:  "jsp统一异常处理"
keywords: "jspError, jsp异常,tomcat 500"
description: "给jsp统一异常处理"
category: demo
tags: [jsp]
---

## jsp统一异常捕获
> 实现所有jsp抛出的异常，在一个页面统一捕获

1 在需要捕获异常的jsp添加一条指令` <%@ page errorPage="/common/error.jsp" %>  ` 

2 在error.jsp中添加 ` isErrorPage="true" `指令，然后就可以再本页使用exception内置对象了

``` jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"    pageEncoding="UTF-8"%>
<%@ page isErrorPage="true" %><!-- 这个指令是关键 -->
<%  
	/*拿到exception可以做自己的异常逻辑了，是直接打印或者是写文件。*/
	String errorMessage=exception.getMessage();
	//log.error("0001业务报错",exception);//用log4j记录日志
  %>
<html>
<head>
<title>错误页</title>
</head>
<body>
	系统异常请联系管理员:<%=errorMessage%>
</body>
</html>
```

# tomcat配置统一异常拦截
> 如果jsp页面没有单独设置错误页，可以配置项目`web.xml`文件tomcat做统一处理

1 **web.xml** 文件中加入

``` xml
<error-page>
    <error-code>400</error-code>
    <location>/common/400.jsp</location>
  </error-page>
  <error-page>
    <error-code>404</error-code>
    <location>/common/404.jsp</location>
  </error-page>
  <error-page>
    <error-code>500</error-code>
    <location>/common/500.jsp</location>
  </error-page>  
```
2 然后建立相应的jsp页面，其中500页面，同上面的错误页面加上 ` isErrorPage="true" `指令可以统一处理异常
 
# spring mvc 统一异常拦截
> 注解 @ExceptionHandler(Exception.class) 作统一异常管理

``` java  
@ExceptionHandler(Exception.class)
public ModelAndView handleException(Exception e, HttpServletRequest request, HttpServletResponse response) {

	StringWriter sw = new StringWriter();
	e.printStackTrace(new PrintWriter(sw, true)); //将错误堆栈输出流转换为String
	String errorMsg = sw.toString();
	logger.error("[handleException]进行Controller处理时出现错误：{}", errorMsg);

	ModelAndView mav = new ModelAndView(ERROR_PAGE);
	if(e instanceof ReportException){
		mav.addObject("errCode", ((ReportException) e).getErrCode());
		mav.addObject("errMsg", ((ReportException) e).getErrMsg());
	}else{
		mav.addObject("errMsg", errorMsg);
	}

	return mav;
}
``` 
此方法可以放在一个抽象类里面，所有相同模块的Controller都统一公用这个异常处理
