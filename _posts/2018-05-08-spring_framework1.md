---
title: Spring(一)
description:  Spring介绍，Spring环境搭建，Spring概念以及Spring配置详解
categories:
 - Web
 - Java
 - Spring
tags: [Web, Java, Spring]
---

# Spring介绍

# Spring环境搭建
与使用IDEA搭建Hibernate环境类似，[参考](https://pu1satilla.github.io/web/java/2018/05/02/hibernate-frame/) 

Spring4.2.4所需jar、源码以及文档，[下载](https://pan.baidu.com/s/1Bi5zYy9AFGe2O-qUkbzVDA)  

环境搭建完成，将在WEB-INF文件夹下自动生成的applicationContext.xml文件移动到src下（可选）

项目结构如图：
![](/assets/images/Spring/idea.png)

# Spring概念

## IOC

## DI 

# Spring配置详解

在applicationContext.xml文件下配置

## Bean元素
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
	<!--
    将JavaBean对象交给spring容器管理
    Bean元素：使用该元素描述需要spring容器管理的对象
        class属性：被管理对象的完整类名
        name属性：给被管理的对象起个名字，获得对象时根据该名称获得对象，
            名称可以重复，可以使用特殊字符
        id属性：与name属性一模一样，
            名称不可重复，不能使用特殊字符
        尽量使用name属性
    -->
    <bean name="user" class="cn.Pu1satilla.domain.User"/>
</beans>
```

## Bean元素进阶

## Spring创建元素的方式

### 空参构造方式
``` xml

```

## Spring的分模块配置






























