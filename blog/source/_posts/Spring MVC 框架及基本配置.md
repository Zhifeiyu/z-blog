---
title: Spring MVC 框架及基本配置
date: 2016-07-21 10:08:09
tags: [spring,springmvc,java]
categories: [spring]
---
## Web开发-请求响应模型
<center>![请求响应模型](http://oflrm5g9z.bkt.clouddn.com/17-2-15/6043565-file_1487163503878_11a8c.png)</center>
1. web客户端（如：浏览器）发起请求，如访问www.baidu.com
2. web服务器端（如：tomcat）接收请求，处理请求，最后产生响应
3. web服务器端处理完成后，返回内容给客户端，客户端对接收的内容进行处理（如web浏览器对接收到的html内容进行渲染展示）
## Web MVC概述
<center>![ MVC概述](http://oflrm5g9z.bkt.clouddn.com/17-2-15/28677677-file_1487163208601_13fe6.png)</center>
**Model（模型）**：数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或 JavaBean 组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据） 和 服务层（行为）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。
 **View（视图）**：负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。
 **Controller（控制器）**：接收用户请求，委托给模型进行处理（状态改变） ，处理完毕后把返回的模型数据返回给视图，由视图负责展示。 也就是说控制器做了个调度员的工作。
## Spring MVC 架构
### 请求处理过程
<center>![ 请求处理过程](http://oflrm5g9z.bkt.clouddn.com/17-2-15/89018877-file_1487163563329_16205.png)</center>
  执行步骤如下：
1. 首先用户发送请求————>前端控制器，前端控制器根据请求信息（如 URL）来决定选择哪一个页面控制器进行处理并把请求委托给它；上图中的 1、2 步骤；
2. 页面控制器接收到请求后，进行功能处理，首先需要收集和绑定请求参数到一个对象，并进行验证，然后将该对象对象委托给业务对象进行处理；处理完毕后返回一个 ModelAndView（模型数据和逻辑视图名） ；上图中的 3、4、5 步骤；
3. 前端控制器收回控制权，然后根据返回的逻辑视图名，选择相应的视图进行渲染，并把模型数据传入以便视图渲染；上图 中的步骤 6、7；
4. 前端控制器再次收回控制权，将响应返回给用户，图 2-1 中的步骤 8；至此整个结束。
### 核心架构
<center>![ 核心架构](http://oflrm5g9z.bkt.clouddn.com/17-2-15/55267022-file_1487163626154_10138.png)</center>
具体流程步骤：
1. 首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；
2. DispatcherServlet——>Handlermapping（请求到处理器的映射），HandlerMapping 将会把请求映射为 HandlerExecutionChain 对象（包含一个 Handler 处理器（页面控制器）对象、多个 HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
3. DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4. HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个 ModelAndView 对象（包含模型数据、逻辑视图名）；
5. ModelAndView 的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的 View，通过这种策略模式，很容易更换其他视图技术；
6. View——>渲染，View 会根据传进来的 Model 模型数据进行渲染，此处的 Model 实际是一个 Map 数据结构，因此很容易支持其他视图技术；
7. 返回控制权给 DispatcherServlet，由 DispatcherServlet 返回响应给用户，到此一个流程结束。
## 入门（Hello）
### 前端控制器（DispatcherServlet）配置
web.xml 配置
```
    <!--前端控制器-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <!--参数定义了要装入的 Spring 配置文件。-->
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <!--
            当值为0或者大于0时，表示容器在应用启动时就加载这个servlet；
            当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载。
            正数的值越小，启动该servlet的优先级越高。
        -->
        <load-on-startup>1</load-on-startup>
    </servlet>
```
### handlerMapping、handlerAdapter配置
resources/spring-mvc.xml中配置handlerMapping、handlerAdapter
```
<!-- 非注解式控制器 -->
    <!-- 处理器映射解析器HandlerMapping -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    <!-- 处理器适配器 -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
    <!-- 处理器 -->
    <bean name="/hello" class="com.zfy.demo.springmvc.controller.HelloController"/>


    <!-- 注解式控制器 -->
    <!-- 开启注解式处理器支持(启用注解) 方式1 -->
    <!--bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>-->
    <!--<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>-->
    <!-- 开启注解式处理器支持(启用注解) 方式2 -->
    <!--<mvc:annotation-driven />-->
```
### viewResolver
resources/spring-mvc.xml中配置viewResolver
```
    <!-- 视图分解器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```
### 开发处理器/页面处理器
非注解式controller
```
public class HelloController implements Controller {

    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        //1、收集参数
        //2、绑定参数到命令对象
        //3、调用业务对象
        //4、选择下一个页面
        ModelAndView mv = new ModelAndView();
        //添加模型数据 可以是任意的POJO对象
        mv.addObject("message", "Hello World!");
        //设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
        mv.setViewName("/hello");
        return mv;
    }
```
 **ModelAndView**：包含了视图要实现的模型数据和逻辑视图名；“mv.addObject("message", "Hello World!");”表示添加模型数据，此处可以是任意 POJO 对象；“mv.setViewName("/hello");”表示设置逻辑视图名为“hello”，视图解析器会将其解析为具体的视图，如前边的视图解析器InternalResourceVi。wResolver 会将其解析为“WEB-INF/jsp/hello.jsp”。
注解式controller
```
@Controller
public class HelloController2 {
    @RequestMapping(value = "/hello2")
    public ModelAndView hello(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) {
        Map<String, Object> data = new HashMap<>();
        data.put("message", "hello world 2");
        return new ModelAndView("/hello", data);
    }

}
```
### 开发视图页面
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Hello World</title>
</head>
<body>
${message}
</body>
</html>
```
### 运行流程分析
<center>![ 运行流程分析](http://oflrm5g9z.bkt.clouddn.com/17-2-15/53622352-file_1487163689720_8c8.png)</center>
运行步骤如下：
1. 首先用户发送请求 http://localhost:8080/hello——>web 容器，web 容器根据“/hello”路径映射到DispatcherServlet（url-pattern 为/）进行处理；
2.  DispatcherServlet——>BeanNameUrlHandlerMapping 进行请求到处理的映射，BeanNameUrlHandlerMapping 将“/hello”路径直接映射到名字为“/hello”的 Bean 进行处理，即 HelloWorldController，BeanNameUrlHandlerMapping将其包装为HandlerExecutionChain（只包括 HelloWorldController 处理器，没有拦截器） ；
3.  DispatcherServlet——> SimpleControllerHandlerAdapter，SimpleControllerHandlerAdapter 将 HandlerExecutionChain中的处理器（HelloWorldController）适配为 SimpleControllerHandlerAdapter；
4. SimpleControllerHandlerAdapter — — > HelloWorldController 处 理 器 功 能 处 理 方 法 的 调 用 ，SimpleControllerHandlerAdapter 将会调用处理器的 handleRequest 方法进行功能处理，该处理方法返回一个 ModelAndView 给 DispatcherServlet；
5. hello（ModelAndView 的逻辑视图名）——>InternalResourceViewResolver， InternalResourceViewResolver 使用JstlView，具体视图页面在/WEB-INF/jsp/hello.jsp；
6. JstlView（/WEB-INF/jsp/hello.jsp）——>渲染，将在处理器传入的模型数据(message=HelloWorld！)在视图中展示出来；
7. 返回控制权给 DispatcherServlet，由 DispatcherServlet 返回响应给用户，到此一个流程结束。
## post中文乱码解决方案
spring Web MVC 框架提供了 org.springframework.web.filter.CharacterEncodingFilter 用于解决 POST 方式造成的中文乱码问题，具体配置如下：
```
    <!-- 指定UTF-8编码 -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```


