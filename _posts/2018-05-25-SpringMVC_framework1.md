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

### web.xml文件
配置web下web.xml文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!--注册中央调度器-->
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

#### 全限定性类名
中央调度器为一个Servlet，需要注册才能够使用，其全限定类名为  
`org.springframework.web.servlet.DispatcherServlet`。

#### load-on-startup标签
作用是标记是否在Web服务器启动时会创建这个`Servlet`实例，是否会在Web服务器启动时调用执行该Servlet的init()方法，而不是真正访问才创建。

值的要求：
- 必须是整数
- 值>=0时，表示容器在启动时就加载并初始化这个servlet，数值越小，该servlet的优先级越高，被创建得越早。
- 当值小于0或者没有指定时，则表示该servlet在真正被使用时才会去创建。
- 当值相同时，容器会自己选择创建顺序

#### url-pattern标签

**建议写为*.xxx形式**  
在没有特殊要求的情况下，`SpringMVC`的中央调度器`DispatcherServlet`的`<url-pattern/>`常使用后辍匹配方式，如写为`*.do`。

**不能写为`/*`**  
`/*`会覆盖所有其他的servlet，无论发送什么请求都会通过这个servlet  
这里的url-pattern不能写为/*，所有请求经过DispatcherServlet中央调度器进行分配给指定处理器映射器，处理器映射器发送给对应Controller，处理器Controller经过一系列处理之后将信息转发给jsp动态页面时中央调度器再次进行调用处理器映射器，jsp对应不上相应处理器会报404错误。

**最好也不要写为`/`**  
`/`不覆盖其他servlet，用于所有不匹配任何其他注册servlet的请求。  
最好也不要写为/，当客户端发送静态页面获取请求时，中央处理器会当做是一个普通的Controller请求。中央处理器发送请求给处理器适配器，处理器适配器寻找不到相应的处理器就会报404.

#### 配置文件位置与名称
不进行配置时默认在WEB-INF下寻找对应xxx-servlet.xml配置文件。  

配置指定位置：
``` xml
    <!--中央处理器-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!--设置中央处理器配置位置-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
			<!--classpath表示配置在src文件夹下-->
            <param-value>classpath:dispatcher-servlet.xml</param-value>
        </init-param>

        <!--tomcat启动时servlet创建优先级-->
        <load-on-startup>1</load-on-startup>
    </servlet>
```

### servlet_name-servlet.xml

#### 注册处理器
配置web下dispatcher-servlet.xml文件用于注册处理器，id属性值为一个请求URI，表示客户端提交该请求时，会访问class指定的这个处理器。
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--注册处理器-->
    <bean id="/x.form" class="cn.Pu1satilla.handlers.MyController"/>
</beans>
```

#### 配置视图解析器
springMVC为了避免请求资源路径与扩展名上的冗余，在视图解析器InternalResouceViewResolver中引入请求的前缀和后缀，在控制器中只需要添加文件名即可，视图解析器完成自动拼接。
``` xml
    <!--注册视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/demo/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```

## 书写controller代码
``` java
public class MyController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv = new ModelAndView();

        //        底层执行的是request.setAttribute()方法
        mv.addObject("message", "Hello springmvc world");

        //        只需要输入文件名，视图解析器能够完成全名拼接
        mv.setViewName("welcome");
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
3. 处理器映射器根据请求，找到处理该请求的处理器，并将其封装为处理器执行链（request请求对应的处理器以及拦截器）后返回给中央调度器
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
# 处理器映射器
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

# 处理器适配器
org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

# 视图解析器
org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver
```

# sprinMVC配置文件全约束
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
</beans>
```

# 静态资源访问
`<url-pattern/>`的值并不是说写为`/`后，静态资源就无法访问了。经过一些配置后，该问题也是可以解决的。

## default的Servlet
在Tomcat中，有一个专门用于处理静态资源访问的Servlet-DefaultServlet，其`<Servlet-name/>`为default，可以处理各种静态资源访问请求。其注册在Tomcat主配置文件web.xml文件中，位于config目录下web.xml。
![](/assets/images/springMVC/tomcat_xml.png)

因为在xml文件中以及配置注册，我们只需要直接使用即可，在web.xml文件下直接进行配置。
``` xml
    <!--处理静态资源-->
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.jpg</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.png</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.js</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.css</url-pattern>
    </servlet-mapping>
```
通过tomcat直接匹配相应处理静态文件的servlet，不需要经过spring，可能效率更好？

## mvc:default-servlet-handler
导入springMVC约束，并且进行配置
``` xml
<mvc:default-servlet-handler/>
```
会将对静态资源的访问请求添加到`SimpleUrlHandlerMapping`的`urlMap`中，key就是请求的URI，而value则为默认Servlet请求处理器`DefaultServletHttpRequestHandler`对象。而该处理器调用了Tomcat的`DefaultServlet`来处理静态资源的访问请求。

## mvc:resources标签
在Spring3.0.4版本后，Spring中定义了专门用于处理静态资源访问请求的处理器`ResourceHttpRequestHandler`。并且添加了`<mvc:resources/>`标签，专门用于解决静态资源无法访问问题。需要在`springmvc.xml`中添加如下形式的配置：
``` xml
    <!--
        处理静态资源请求
            mapping: 访问地址
            location:工程地址
    -->
    <mvc:resources mapping="/img/**" location="/img/"/>
```
- location是工程路径地址
- mapping是映射后的访问地址

# 相对路径

## 后台相对路径
以斜杠开头的后台相对路径：web应用下的根（IDEA目录下web文件夹作为根）。  
**例如：**
``` xml
    <!--注册处理器-->
    <bean id="/x.form" class="cn.Pu1satilla.handlers.MyController"/>
```
匹配的路径为：xxx.xxx.xxx:端口号/applicationContext-name/x.form

## 前台相对路径
以斜杠开头的前台相对路径：路径=服务器网址+端口号+前台相对路径  
**例如：**  
```
//		前台相对路径
"href : '/action' "

127.0.0.1:8080/action
```

不以斜杠开头的前台相对路径：路径=当前访问路径+前台相对路径  
**例如：**  
```
//		当前访问路径
127.0.0.1:8080/abc.jsp

//		前台相对路径
"href : 'example/action' "

//		完整路径
127.0.0:8080/example/action
```

## 特例
当进行response的重定向时相当于浏览器进行再次访问，规则等同于前台相对路径。    
**例如：**  
``` java
//		路径 = 127.0.0.1+/index.jsp
response.sendRedirect("/index.jsp");
```














































































