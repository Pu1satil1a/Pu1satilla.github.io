---
title: SpringMVC(三)
description: SpringMVC具体实现流程
categories:
 - Web
 - Java
 - SpringMVC
tags: [Web, Java, SpringMVC]
---


1. tomcat接收客户端发送的请求，将请求封装成一个request对象，通过web.xml找到对应Servlet，当请求路径与DispatherServlet匹配路径一致时，将reqeust发送给DispatcherServlet处理。
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

    <!--匹配路径-->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

2. 中央调度器接收到request对象，











































































































































































