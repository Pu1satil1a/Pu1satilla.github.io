---
title: Spring(三)
description: 使用注解配置spring和spring中的aop
categories:
 - Web
 - Java
 - Spring
tags: [Web, Java, Spring]
---


# 注解配置spring

## xml文件约束
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--
    扫描cn.Pu1satilla.domain文件下所有类的注解
    注意：扫描包时，会扫描包下的所有子孙包
    -->
    <context:component-scan base-package="cn.Pu1satilla.domain"/>
</beans>
```

## 将对象注册到容器
在xml配置下原型为：
``` xml
<bean name="user" class="cn.Pu1satilla.domain.User">
```

### 注解位置
位于类上方

### 普通注解
``` java
@Component("user")
```

### 分层注解
``` java
//		service层
@Service("user")

//		web层
@Controller("user")

//		dao层
@Repository("user")
```

## 修改对象的作用范围（scope）
在xml配置下原型为：
``` xml
<bean name="user0" class="cn.Pu1satilla.domain.User" scope="prototype"/>
```

### 注解位置
位于类上方

### 注解格式
``` java
@Scope("prototype")
```

##  值类型注入

### 反射注入
通过反射的Field赋值，破坏了封装性
``` java
@Value("tom")
private String name;
@Value("18")
private Integer age;
```

### set注解注入
通过set方法赋值，推荐使用
``` java
@Value("Lucy")
public void setName(String name) {
	this.name = name;
}

@Value("18")
public void setAge(Integer age) {
	this.age = age;
}
```

## 引用类型注入

### 自动装配
``` java
@Autowired
private Cat cat;
```
但是如果容器中出现了多个cat，那么无法选择具体注入是哪一个cat，这时就需要指定装配

### 指定装配
``` java
@Autowired
@Qualifier("cat1")
private Cat cat;
```

### 手动注入（建议）
直接告诉spring装配给定对象
``` java
@Resource(name = "cat1")
private Cat cat;
```

## 初始化|销毁方法
在xml配置下原型为：
``` xml
<bean name="user" class="cn.Pu1satilla.domain.User" destroy-method="destroy" init-method="init"/>
```

注解配置：
``` java
@PostConstruct  //在对象在构造后调用
public void init(){
	System.out.println("user被初始化");
}

@PreDestroy  //对象销毁之前调用（在多例模式下无法调用）
public void destroy(){
	System.out.println("user被销毁");
}
```

# spring整合junit

配置注解
``` java
//由注解创建容器
@RunWith(SpringJUnit4ClassRunner.class)
//指定创建容器时使用配置文件
@ContextConfiguration("classpath:applicationContext.xml")
```

测试用例
``` java
package cn.Pu1satilla.demo;

import cn.Pu1satilla.domain.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;

//由注解创建容器
@RunWith(SpringJUnit4ClassRunner.class)
//指定创建容器时使用配置文件
@ContextConfiguration("classpath:applicationContext.xml")

public class JunitSpringDemo {

    @Resource(name = "user")
    private User user;

    @Test
    public void demo1() {
        System.out.println(user);
    }
}
```

# Spring的AOP

## AOP思想
`AOP（Aspect-OrientedProgramming，面向方面编程）`，可以说是OOP`（Object-Oriented Programing，面向对象编程）`的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当我们需要为分散的对象引入公共行为的时候，OOP则显得无能为力。也就是说，OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如日志功能。日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。对于其他类型的代码，如安全性、异常处理和透明的持续性也是如此。这种散布在各处的无关的代码被称为横切`（cross-cutting）`代码，在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

而AOP技术则恰恰相反，它利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手将这些剖开的切面复原，不留痕迹。



















































































































































