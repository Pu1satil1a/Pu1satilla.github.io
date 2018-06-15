---
title: ssm(一)
description: ssm框架整合
categories:
 - Web
 - Java
 - SSM
tags: [Web, Java, SSM]
---


# 前端文件


# 配置web.xml文件



## ContextLoaderListener监听器
想要使用spring容器，那么必须创建spring容器，但是spring容器在何时创建最好呢？若是Servlet类会导致每次访问都会创建spring容器，显然不可行。**使用的是spring包中的ServletContext监听器，在服务器创建时监听器就完成了spring容器创建的初始化动作。**
``` xml
<!--
	1.1 注册spring容器创建监听器ContextLoaderListener
-->
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<!--
	1.2 设置spring配置文件路径
	值为所有符合param-value的配置文件里面配置bean在容器创建时都会创建
-->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:cn/pu1satilla/source/spring-*.xml</param-value>
</context-param>
```

# 配置字符集编码过滤器
解决请求参数携带参数乱码问题
``` xml
<!--2.注册字符编码过滤器-->
<filter>
	<filter-name>encoding</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

	<init-param>
		<!--encoding：设置编码字符集-->
		<param-name>encoding</param-name>
		<param-value>utf-8</param-value>
	</init-param>

	<!--设置强制使用指定的字符集：
			true：代码中指定的字符集不起作用，
			false：代码中若指定了字符集，就使用代码指定字符集，没指定就使用xml配置字符集
	-->
	<init-param>
		<param-name>forceEncoding</param-name>
		<param-value>true</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>encoding</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

# 注册中央调度器
当客户端访问指定路径，都会经过springmvc处理，将请求给中央调度器进行再一步处理。
``` xml
<!--
	3.注册spring中央调度器
		contextConfigLocation：  springmvc配置文件路径
		load-on-startup：        容器创建时Servlet创建优先级，默认越小越先
-->
<servlet>
	<servlet-name>DispatcherServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:cn/pu1satilla/source/spring-mvc.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>DispatcherServlet</servlet-name>
	<url-pattern>*.do</url-pattern>
</servlet-mapping>
```

# 数据库连接池对象
使用的数据库连接池为阿里Druid连接池

创建配置文件Druid.properties
```
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.password=****
jdbc.username=root
jdbc.jdbcUrl=jdbc:mysql://localhost:3306/ssmDemo?useUnicode=true&characterEncoding=utf8
jdbc.initialSize=5
jdbc.minIdle=5
jdbc.maxActive=20 
```

创建spring-db.xml文件
``` xml
<!--引入dbcp配置文件-->
<context:property-placeholder location="classpath:cn/pu1satilla/source/Druid.properties"/>
<!--注入dataSource对象-->
<bean id="projectDataSource" class="com.alibaba.druid.pool.DruidDataSource">
	<property name="password" value="${jdbc.password}"/>
	<property name="name" value="${jdbc.username}"/>
	<property name="url" value="${jdbc.jdbcUrl}"/>
</bean>
```
# dao层

## 接口
``` java
public interface ProjectDao {
    void insert(Student student);
}
```

## 配置文件

### spring-mybatis
该配置文件主要用于注册SqlSessionFactory对象给spring容器，并且注入mybatis通过Mapper动态代理生成dao对象。
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
    注入sqlSessionFactory对象
        dataSource：     数据库连接池对象
        configlocation： mybatis主配置文件位置
    -->
    <bean id="projectSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:cn/pu1satilla/source/mybatis.xml"/>
    </bean>

    <!--指定Mapper动态代理生成的扫描位置-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--指定SqlSessionFactory对象-->
        <property name="sqlSessionFactoryBeanName" value="projectSqlSessionFactory"/>
        <!--指定扫描包位置，生成接口的实现类对象-->
        <property name="basePackage" value="cn.pu1satilla.dao"/>
    </bean>
</beans>
```

### mybatis主配置文件
主配置文件无需配置运行环境，在spring-db.xml文件中已经设置好了数据库连接池对象，并且在注入sqlSessionFactory对象给spring容器时就已经配置好了dataSource属性。
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--别名设置-->
    <typeAliases>
        <package name="cn.pu1satilla.domain"/>
    </typeAliases>

    <!--指定mapper位置-->
    <mappers>
        <package name="cn.pu1satilla.dao"/>
    </mappers>
</configuration>
```

### mapper配置文件
``` xml
<!--指定动态代理生成对象接口类-->
<mapper namespace="cn.pu1satilla.dao.ProjectDao">
    <!--id指定类方法，其实现功能-->
    <insert id="insert" parameterType="Student">
        INSERT INTO ssmdemo.register VALUES (
            NULL, #{name}, #{age}
        )
    </insert>
</mapper>
```

# service层

## service接口以及实现类

``` java
public interface ProjectService {
    void insert(Student student);
}


public class ProjectServiceImpl implements ProjectService {
    private ProjectDao projectDao;

    @Override
    public void insert(Student student) {
        projectDao.insert(student);
    }

    public void setProjectDao(ProjectDao projectDao) {
        this.projectDao = projectDao;
    }
}
```

## service配置文件
在src文件夹下source文件夹内创建spring-service.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!--注入service层实现类bean-->
    <bean id="projectService" class="cn.pu1satilla.service.ProjectServiceImpl">
        <property name="projectDao" ref="projectDao"/>
    </bean>
</beans>
```

# controller层
接收来自前台的数据，并且封装成实体
``` java
public class RegisterController implements Controller {

    private ProjectService service;

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {

        //        1.创建模板视图对象
        ModelAndView modelAndView = new ModelAndView();

        //        2.获取请求参数,并且封装成实体
        Student student = new Student();
        student.setName(request.getParameter("name"));
        student.setAge(Integer.valueOf(request.getParameter("age")));

        //        3.将实体对象传递给service层进行插入操作
        service.insert(student);

        //        4.设置跳转页面
        modelAndView.setViewName("/success.jsp");
        return modelAndView;
    }

    public void setService(ProjectService service) {
        this.service = service;
    }
}
```

## springmvc配置文件
在src下source文件夹下创建spring-mvc.xml文件用于注入spring控制器。
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--注入注册控制器-->
    <bean id="/register.do" class="cn.pu1satilla.controller.RegisterController">
        <property name="service" ref="projectService"/>
    </bean>
</beans>
```













































































































































































































































































































