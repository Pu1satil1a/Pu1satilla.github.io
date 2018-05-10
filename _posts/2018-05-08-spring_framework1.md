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
Spring是一个开源框架，Spring是2003年兴起的一个轻量级Java开发框架，是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一是分层架构，分层架构允许使用者选择使用哪一个组件，同时为J2EE应用程序开发提供集成的框架。Spring的用途不仅限于**服务器端**的开发。从简单性、可测试性和松耦合的角度而言，任何
Java应用都可以从Spring中受益。Spring的核心是**控制反转（IoC）**和**面向切面（AOP）**。简单来说，**Spring是一个分层的JavaSE/EEfull-stack(一站式) 轻量级开源框架。**


# Spring环境搭建

## 环境搭建以及导包
与使用IDEA搭建Hibernate环境类似，[参考](https://pu1satilla.github.io/web/java/2018/05/02/hibernate-frame/) 

Spring4.2.4所需jar、源码以及文档，[下载](https://pan.baidu.com/s/1Bi5zYy9AFGe2O-qUkbzVDA)  

## 项目结构
环境搭建完成，将在WEB-INF文件夹下自动生成的applicationContext.xml文件移动到src下（可选）

项目结构如图：
![](/assets/images/Spring/idea.png)

## 代码测试

### 建立JavaBean对象
``` java
public class User {
    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
```

### 简单配置xml文件
简单配置xml文件，使得JavaBean对象应用于Spring容器
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="user" class="cn.Pu1satilla.domain.User"/>
</beans>
```

### 测试类

``` java
@Test
public void demo(){

	//        1.创建Spring容器对象
	ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");

	//        2.向容器要user对象
	User user = (User) applicationContext.getBean("user");

	//        3.打印user对象
	System.out.println(user);
}
```

# Spring概念

## IOC
*Inversion of Control* 控制反转，指的是对象的创建权反转给Spring，对象的创建以及以来关系可以由Spring完成创建以及注入，反转控制就是反转了对象的创建方式，由开发人员创建反转给了Spring，作用是实现了程序的解耦合。

用大白话理解：所有的类都会在spring容器中登记，程序运行时告诉spring需要什么东西，然后spring会把需要的东西给当前。所有的类的创建、销毁都由spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。

## DI 
*Dependency Injection* 依赖注入，需要有IOC环境，spring创建这个类的过程中，spring将类的依赖属性设置进去。

实现IOC思想需要DI作为支持 

注入方式：
- set方法注入
- 构造方法注入
- 字段注入

注入类型：
- 值类型注入
- 引用类型注入

## BeanFactory&&ApplicationContext

### BeanFactory接口
BeanFactory接口子类关系图
![](/assets/images/Spring/structure.png)
作为spring原始接口，针对原始接口的实现类功能较为单一

BeanFactory接口实现类的容器特点：在每次获得对象时才会创建对象

### ApplicationContext接口
ApplicationContext接口的容器特点：每次容器启动就会创建容器中配置的所有对象

在运行配置较差时使用BeanFactory接口
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

### 构造函数方式（重要）

Bean类
``` java
package cn.Pu1satilla.domain;

public class User {
    private String name;
    private Integer age;

    public User() {
        System.out.println("空参构造函数方式创建spring容器对象");
    }

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
        System.out.println("有参构造函数方式创建spring容器对象User{name="+name+";age="+age+"}");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

测试类
``` java
@Test
public void demo1(){

	//        创建spring容器对象
	new ClassPathXmlApplicationContext("applicationContext.xml");
}
```

#### 无参构造函数
配置xml文件
``` xml
<!--1.默认无参数构造函数-->
<bean name="user0" class="cn.Pu1satilla.domain.User"/>
```

测试结果
```
空参构造函数方式创建spring容器对象 
```

#### 有参构造函数
配置xml文件
``` xml
<!--2.带参数构造器-->
<bean name="user1" class="cn.Pu1satilla.domain.User">
	<constructor-arg index="0" type="java.lang.String" value="小明"/>
	<constructor-arg index="1" type="java.lang.Integer" value="10"/>
</bean>
```

测试结果

```
有参构造函数方式创建spring容器对象User{name=小明;age=10} 
```

### 静态工厂（了解）



### 实例工厂（了解）

## Spring的分模块配置






























