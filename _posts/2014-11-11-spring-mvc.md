---
layout: post
title: "spring mvc使用总结"
description: "spring mvc 使用流程"
keywords: ""
category: 
tags: [笔记]
---
####spring mvc的运行流程
首先整个M-V-C的流程都依赖于Spring容器完成对象创建(IOC)和对象依赖注入(DI),
并充分利用注解减少xml配置,本着约定优于配置的原则.
利用一个servlet作为中央控制器,接收客户端请求,
并转发到相应的控制器(controller)做实际的数据采集,转化,校验,组装成model,
调用service业务逻辑,然后将处理结果封装model(map键值对类型)传给view试图,
根据视图解析器根据相应的试图渲染结果返回给客户端.
控制器根据<servletName>-servlet.xml配置选择相应的请求映射器,然后

####spring mvc的优点
####spring mvc的缺点
####使用spring mvc

1. 配置
2. spring-mvc的注解
	 - 请求控制注解
	 - http请求头注解
3. spring-mvc拦截器
4. 请求表单页面
5. 自定义类型属性绑定
6. controller采集数据，校验数据，初始化参考数据，安装token防止表单提交
7. 清除token，控制器返回/重定向到视图
8. 国际化


