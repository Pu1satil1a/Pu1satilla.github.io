---
title: Spring(三)
description: 使用注解配置spring以及junit整合spring
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
















































































































