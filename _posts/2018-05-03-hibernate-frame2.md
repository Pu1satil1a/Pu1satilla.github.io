---
title: Hibernate(二)
description: hibernate常见Api
categories:
 - Web
 - Java
tags: [Web, Java, Hibernate]
---

# Configuration

## 功能
配置加载类，用于加载主配置，orm元数据加载

## 创建
``` java
//创建，调用空参构造
Configuration conf = new Configuration();
```
## 加载主配置
``` java
//读取指定主配置文件=>空参加载方法，加载src下的hibernate
conf.configure();
```
## 加载orm元数据（了解）
``` java
//读取指定orm元数据，如果主配置中已经引入映射配置，不需要手动加载
conf.addResource(ResourceName);
conf.addClass(persistentClass);
```
## 创建sessionFactory
``` java
//根据配置信息，创建SessionFactory对象
SessionFactory sf = con.buildSessionFactory();
```

# SessionFactory

## 功能

用于创建操作数据库核心对象session对象的工厂  
**注意**  
1. sessionFactory 负责保存和使用所有配置信息，消耗内存资源非常大
2. sessionFactory 属于线程安全的对象设计

**结论**  
保证在web项目中，只创建一个sessionFactory()

## 获得session对象
``` java
Session session = sf.openSession();
```
## 获得一个与线程绑定的session对象
``` java
sf.getCurrentSession();
```

# Session（Hibernate框架与数据库之间的连接）

## 功能
类似于JDBC的connection对象，还可以完成对数据库中数据的增删改查操作，Session是Hibernate操作数据库的核心对象

## 操作事务 
``` java
//获得操作事务的对象
Transaction transaction = session.getTransaction();
```
## 获取操作事务的对象，并且开启事务
``` java
Transaction transaction = session.beginTransaction();
```

## session的操作
``` java
//1. 添加数据库列

//1.1 创建user对象
User user = new User();
//1.2 完善user对象
user.setUsername("小明");
//1.3 将对象映射插入到数据库表中
session.save(user);

//2. 删除数据库列（id）

//2.1 获得要删除的对象
User user = session,get(User.class, 1l);
//2.2 调用delete删除对象
session.delete(user);

User user = session.get(User.class,)

//3. 修改数据库列（id）

//3.1 获得要修改的对象
User user = new User();
//3.2 修改对象属性
user.setUsername("小样");
//3.3 执行update方法进行提交
session.update(user);

//4. 查询数据库列（id）

//4.1 获取列对应的对象（id,1l对应，1表示id为1的列，l表示id的类型为long型）
User user = session.get(Customer.class,1l);

```

## 事务提交与回滚
``` java
transaction.commit();
transaction.rollback();
```
## 释放资源
``` java
//释放session资源
session.close();
//释放sessionFactory资源
sessionFactory.close();
```

# 代码整合
``` java
package com.Pu1satilla.demo;

import com.Pu1satilla.domain.UserEntity;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class Demo {
    public static void main(String[] args) {

        //        1.加载主配置
        Configuration configuration = new Configuration();
        configuration.configure();

        //        2.创建sessionFactory
        SessionFactory sessionFactory = configuration.buildSessionFactory();

        //        3.获取session连接数据库会话对象
        Session session = sessionFactory.openSession();

        //        4.获取操作事务对象并开启
        Transaction transaction = session.beginTransaction();

        //        5.hibernate操作数据库
        UserEntity userEntity = new UserEntity();
        userEntity.setUsername("小明");
        userEntity.setPassword(":");
        userEntity.setUid(1);
        session.save(userEntity);
        Object o = session.get(UserEntity.class, 1);
        System.out.println(o);

        //        6.提交事务，释放资源
        transaction.commit();
        session.close();
        sessionFactory.close();
    }
}
```

# 抽取代码（HibernateUtils）

``` java
package com.Pu1satilla.utils;

import com.Pu1satilla.domain.UserEntity;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

/**
 * 工具类，抽取hibernate常用Api，简化编写代码
 */
public class HibernateUtils {

    private static SessionFactory sessionFactory;

    static {
        Configuration configure = new Configuration().configure();
        ServiceRegistry registry = new ServiceRegistryBuilder().applySettings(configure.getProperties()).buildServiceRegistry();
        sessionFactory = configure.buildSessionFactory(registry);
    }

    /**
     * @return 连接数据库会话对象
     */
    public static Session getSession() {
        return sessionFactory.openSession();
    }

    /*demo
    public static void main(String[] args) {

        //        1.建立与数据库表映射对象
        UserEntity userEntity = new UserEntity();
        userEntity.setUsername("小花");
        userEntity.setUid(1);

        //        2.通过工具类获取session对象
        Session session = HibernateUtils.getSession();

        //        3.获取事务并提交
        Transaction transaction = session.beginTransaction();

        //        4.执行hibernate语句
        session.save(userEntity);
        System.out.println(session.get(UserEntity.class, 1));

        //        5.事务提交
        transaction.commit();

        //        6.释放资源
        session.close();
        sessionFactory.close();
    }
    */
}
```














































