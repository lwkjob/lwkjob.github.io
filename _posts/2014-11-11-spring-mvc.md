---
layout: post
title: spring mvc使用总结
description: spring mvc 使用流程
keywords: ""
category: 
tags: [笔记]
---
# spring mvc的运行流程：
- MVC框架的出现是为了将URL从HTTP的世界中映射到JAVA世界中，这是MVC框架的核心功能
- 首先整个M-V-C的流程都依赖于Spring容器完成对象创建(IOC)和对象依赖注入(DI),
- 并充分利用注解减少xml配置,本着约定优于配置的原则.
- 利用一个servlet作为中央控制器,接收客户端请求,
- 并转发到相应的控制器(controller)做实际的数据采集,数据类型转化,校验,组装成model,
- 调用service业务逻辑,然后将处理结果封装model(map键值对类型)传给view试图,
- 根据视图解析器根据相应的试图渲染结果返回给客户端.
- 控制器根据`<servletName>-servlet.xml`配置选择相应的请求映射器

# spring mvc的优点：
- SpringMVC设计思路：将整个处理流程规范化，并把每一个处理步骤分派到不同的组件中进行处理。
- 这个方案实际上涉及到两个方面：
- 处理流程规范化 —— 将处理流程划分为若干个步骤（任务），并使用一条明确的逻辑主线将所有的步骤串联起来
- 处理流程组件化 —— 将处理流程中的每一个步骤（任务）都定义为接口，并为每个接口赋予不同的实现模式
- 处理流程规范化是目的，对于处理过程的步骤划分和流程定义则是手段。因而处理流程规范化的首要内容就是考虑一个通用的Servlet响应程序大致应该包含的逻辑步骤：
- 步骤1—— 对Http请求进行初步处理，查找与之对应的Controller处理类（方法）   ——HandlerMapping
- 步骤2—— 调用相应的Controller处理类（方法）完成业务逻辑                    ——HandlerAdapter
- 步骤3—— 对Controller处理类（方法）调用时可能发生的异常进行处理            ——HandlerExceptionResolver
- 步骤4—— 根据Controller处理类（方法）的调用结果，进行Http响应处理       ——ViewResolver
- 正是这基于组件、接口的设计，支持了SpringMVC的另一个特性：行为的可扩展性。

#  Spring MVC对比 Struts2：
1. 实现机制
	- struts2框架是类级别的拦截，每次来了请求就创建一个controller中对应的Action，然后调用setter getter方法把request中的数据注入 。
	- struts2实际上是通过setter getter方法与request打交道的。struts2中，一个Action对象对应一个request上下文。
	- spring3 mvc不同，spring3mvc是方法级别的拦截，拦截到方法后根据参数上的注解，把request数据注入进去。
	- 在spring3mvc中，一个方法对应一个request上下文，而方法同时又跟一个url对应。
	- 所以说从架构本身上 spring3 mvc就容易实现restful url。
	- 而struts2的架构实现起来要费劲，因为struts2 action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法。

2. Request数据共享
	- spring3mvc的方法之间基本上独立的，独享request response数据。请求数据通过参数获取，处理结果通过ModelMap交回给框架，方法之间不共享变量。
	- 而struts2搞的就比较乱，虽然方法之间也是独立的，但其所有Action变量是共享的。这不会影响程序运行，却给我们编码、读程序时带来麻烦 。

#  spring mvc的缺点：
- 项目全用注解如果事先不定好规范，后期不利于维护，配置文件可读性强，但是如果定好规范比如:
- 控制器的包路径=请求路径=映射文件jsp路径，这样也会是一目了然了，后期维护人员找类也好找。

#  使用spring mvc：
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


