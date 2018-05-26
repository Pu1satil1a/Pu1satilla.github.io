---
title: SpringMVC(一)
description: SpringMVC简介
categories:
 - Web
 - Java
 - SpringMVC
tags: [Web, Java, SpringMVC]
---

# SpringMVC简介
`SpringMVC`也叫`Spring  web  mvc`，属于表现层的框架。`SpringMVC`是Spring框架的一部分，是在Spring3.0后发布的。

![](/assets/images/springMVC/jianjie.png)
spring由四大部分组成：Dao部分（Dao与ORM）、AOP部分、Web部分（JEE与Web）及Ioc容器部分（core）。

# 第一个HelloWorld
功能：访问指定网站，服务器端处理器在接受到这个请求后，显示hello，world，在相应页面显示该信息。

## 通过IDEA搭建工程环境
如图通过IDEA搭建工程环境：
![](/assets/images/springMVC/create_project.png)

项目结构简单配置会出现下面目录结构：
![](/assets/images/springMVC/structure.png)

## 导入jar包
![](/assets/images/springMVC/jar.png)

## 配置xml文件
配置web下web.xml文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!--注册处理器-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.form</url-pattern>
    </servlet-mapping>
</web-app>
```

配置web下dispatcher-servlet.xml文件用于注册处理器
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--注册处理器-->
    <bean id="/x.form" class="cn.Pu1satilla.handlers.MyController"/>
</beans>
```

## 书写controller代码
``` java
public class MyController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv = new ModelAndView();

        mv.addObject("message", "Hello springmvc world");
        mv.setViewName("WEB-INF/demo/welcome.jsp");
        return mv;
    }
}
```

## jsp文件
在WEB-INF下面建立demo文件夹，demo文件夹下建立welcome.jsp文件
``` jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Welcome page</title>
</head>
<body>
    ${message}
</body>
</html>
```

最后测试，在浏览器上访问`http://localhost:8080/x.form`网址会出现`Hello springmvc world`。


# springMVC执行流程
![](/assets/images/springMVC/springMVC_zhixing.png)

执行流程简单分析：  
1. 浏览器发送请求给中央调度器（DispatcherServlet）
2. 中央调度器发送请求给处理器映射器
3. 处理器映射器根据请求，找到处理该请求的处理器，并将其封装为处理器执行链后返回给中央调度器
4. 中央调度器根据处理器执行链中的处理器，找到能够执行该处理器的处理器适配器。
5. 处理器适配器调用执行处理器。
6. 处理器将处理结果及要跳转的视图封装到一个对象ModelAndView中，并将其返回给处理器适配器。
7. 处理器适配器直接将结果返回给中央调度器。
8. 中央调度器调用视图解析器，将ModelAndView中的视图名称封装为视图对象。
9. 视图解析器将封装了的视图对象返回给中央调度器。
10. 中央调度器调用视图对象，让其自己进行渲染，即进行数据填充，形成响应对象。
11. 中央调度器响应浏览器。

# DispatcherServlet的默认配置
第一个SpringMVC的程序已经可以正确运行了。但发现一个问题：在流程介绍中所述的重要的处理器映射器、处理器适配器、视图解析器等，都在哪里，不用做配置吗？

这些内容均在`DispatcherServlet`的默认配置`DispatcherServlet.properties`文件中被定义。这个文件与`DispatcherServlet`类在一个包下，而且是当Spring配置文件中没有指定配置时使用的默认情况。
![](/assets/images/springMVC/DispatcherServlet_properties.png)

``` properties
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter
	
org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver
```

# url-pattern标签

## 建议写为*.xxx形式
在没有特殊要求的情况下，`SpringMVC`的中央调度器`DispatcherServlet`的`<url-pattern/>`常使用后辍匹配方式，如写为`*.do`。

## 不能写为/*
`/*`会覆盖所有其他的servlet，无论发送什么请求都会通过这个servlet  
这里的url-pattern不能写为/*，所有请求经过DispatcherServlet中央调度器进行分配给指定处理器映射器，处理器映射器发送给对应Controller，处理器Controller经过一系列处理之后将信息转发给jsp动态页面时中央调度器再次进行调用处理器映射器，jsp对应不上相应处理器会报404错误。

## 最好也不要写为/
`/`不覆盖其他servlet，用于所有不匹配任何其他注册servlet的请求。  
最好也不要写为/，当客户端发送静态页面获取请求时，中央处理器会当做是一个普通的Controller请求。中央处理器发送请求给处理器适配器，处理器适配器寻找不到相应的处理器就会报404.































































































