---
title: Hibernate(四)
description: hibernate事务以及Hibernate的批量查询（概述）
categories:
 - Web
 - Java
 - Hibernate
tags: [Web, Java, Hibernate, MySQL事务]
---

# hibernate事务

## 事务特性（ACID）
1. 原子性：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生
2. 一致性：事务前后数据的完整必须保持一致（A-100同时B+100）
3. 隔离性：多个用户并发访问数据库时，一个用户的事务不能被其它用户的事务所干扰，多个并发事务之间数据要相互隔离
4. 持久性：一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

## 事务并发问题
1. 脏读：B事务读取到A事务尚未提交的数据
2. 不可重复读：一个事务读到了另一个事务已经提交(update)的数据。引发另外一个事务在事务中的多次查询结果不一致
3. 虚读/幻读：一个事务读到了另一个事务已经提交(insert,delect)的数据。导致另一个事务在事务中多次查询的结果不一致

## 隔离级别
数据库规范规定4中隔离级别，分别用于描述两个事务并发的所有情况。
1. **read uncommitted**：读取尚未提交的数据，一个事务读到另一个事务没有提交的数据
- 存在：3个问题
- 解决：0个问题
2. **read commited**：读取提交的数据，一个事务读取另外一个提交的事务(Oracle默认)
- 存在：2个问题
- 解决：1个问题(脏读)
3. **repeatable read**：可重复读，在一个事务中读到的数据始终保持一致，无论另一个事务是否提交(MySQL默认)
- 存在：1个问题(虚读/幻读)
- 解决：2个问题
4. **serializable**：串行化，同时只能执行一个事务，相当于事务中的单线程
- 存在：0个问题
- 解决：3个问题

- 安全与性能成反比(能解决更多的问题，性能越低)
- 常见数据库隔离级别(Oracle:read commited, MySQL:repeatable read)

## 指定hibernate隔离级别
``` xml
<!--指定数据库的隔离级别
	级别分类：1|2|4|8
	0001    1   读未提交
	0010    2   读已提交
	0100    4   可重复读
	1000    8   串行化
-->
<property name="hibernate.connection.isolation">4</property>
```

## 在hibernate中管理事务
为了防止层与层之间进行污染，要求在业务层开启事务，采用类似于TreadLocal绑定线程的方式将对象绑定到当前线程，HIbernate提供了方便的接口实现了这个功能（getCurrentSession()）。

### 配置
``` xml
<!--指定session与当前线程绑定-->
<property name="hibernate.current_session_context_class">thread</property>
```
注意：调用getCurrentSession()方法获得的session对象，当事务提交时，session会自动关闭，不需要手动关闭

### 完善HibernateUtils
``` java
package com.Pu1satilla.utils;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.service.ServiceRegistryBuilder;

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

    /**
     * @return 获取与当前线程绑定的session对象
     */
    public static Session getCurrentSession() {
        return sessionFactory.getCurrentSession();
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





































