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

# hibernate批量查询

## HQL查询
### 概念以及适用情况
Hibernate独家查询语言，属于面向对象的查询语言，适用于多表查询，但是不复杂时查询

### 基本查询
``` java
//        1.书写HQL语句，当源码包中类名唯一，可以只输类名
//        String Hql = "FROM cn.Pu1satilla.domain.Customer";
String hql = "FROM Customer";

//        2.根据Hql语句创建查询对象
Query query = session.createQuery(hql);

//        3.根据查询对象获取查询结果（接收唯一的查询结果：query.uniqueResult();）
List list = query.list();
```
### 条件查询

#### ?号占位符
类似于sql语句中的？号占位符
``` java
//        1.书写HQL语句
String hql = "FROM Customer WHERE cid=?";

//        2.创建查询对象
Query query = session.createQuery(hql);

//        3.设置占位符内容
query.setParameter(0, 1);

//        4.根据查询对象获取查询结果
Customer customer = (Customer) query.uniqueResult();
```

#### 命名占位符
将问号代替成:name，那么设置参数的时候对应的就是hql语句在设定的那个名称
``` java
//        1.书写HQL语句
String hql = "FROM Customer WHERE cust_name = :second_name";

//        2.创建查询对象
Query query = session.createQuery(hql);

//        3.设置占位符内容
query.setParameter("second_name", "村上");
Customer customer = (Customer) query.uniqueResult();
```

### 分页查询
类似于sql语句中limit ?,?  

	第一个?对应query.setFirstResult(); 第二个?对应query.setMaxResults();
	
``` java
//        1.书写HQL语句
String hql = "FROM Customer";

//        2.创建查询对象
Query query = session.createQuery(hql);

//        3.设置分页情况
query.setFirstResult(0);
query.setMaxResults(2);

//        4.获取查询内容
List list = query.list();
System.out.println(Arrays.toString(list.toArray()));
```

## Criteria查询

### 概念以及适用情况
Hibernate自创的无语局面向对象查询，适用于单表条件查询

### 基本查询
``` java
//        查询所有的Customer对象
Criteria criteria = session.createCriteria(Customer.class);
List list = criteria.list();
```

### 条件查询
条件查询对应方法
```
>	gt 
>=	ge
<	lt
<=	le
==	eq
!=	ne
in	in
between and	between
like		like
is not null	isNotNull
is null		isNull
or		or
and		and
```

``` java
//        1.创建Criteria查询对象
Criteria criteria = session.createCriteria(Customer.class);

//        2.添加查询参数 => 查询cid>1的Customer对象
criteria.add(Restrictions.gt("cid", 1));

//        3.执行查询获得结果
Customer customer = (Customer) criteria.uniqueResult();
System.out.println(customer);
```

### 分页查询
``` java
//        1.创建Criteria查询对象
Criteria criteria = session.createCriteria(Customer.class);

//        2.分页设置
criteria.setFirstResult(0);
criteria.setMaxResults(2);

//        3.获取查询结果
List list = criteria.list();
```

## Sql语句查询

### 适用情况
适用于复杂的业务查询

### 基本查询

#### 返回数组List
``` java
//        1.书写sql语句
String sql = "SELECT * FROM Customer";

//        2.创建sql查询对象
SQLQuery sqlQuery = session.createSQLQuery(sql);

//        3.执行查询获得结果 => 获得数组list集合
List<Object[]> list = sqlQuery.list();
for (Object[] objects : list) {
	System.out.println(Arrays.toString(objects));
}
```
查询结果：  
```
Hibernate: 
    SELECT
        * 
    FROM
        Customer
[1, 村上, vip, 百度, 546, 4848, 56464545646]
[2, 马彦宏, 农民, 澡堂, 1001, 4848, 56]
```
#### 返回对象List
``` java
//        1.书写sql语句
String sql = "SELECT * FROM Customer";

//        2.获取sql查询对象
SQLQuery sqlQuery = session.createSQLQuery(sql);

//        指定结果集封装到某个对象
sqlQuery.addEntity(Customer.class);

//        3.执行查询获得结果 => 获得对象集合
List list = sqlQuery.list();
```
查询结果：  
```
Hibernate: 
    SELECT
        * 
    FROM
        Customer
[Customer{cid=1, cust_name='村上', cust_level='vip', cust_source='百度', cust_linkman='546', cust_phone=4848, cust_mobile=56464545646}, Customer{cid=2, cust_name='马彦宏', cust_level='农民', cust_source='澡堂', cust_linkman='1001', cust_phone=4848, cust_mobile=56}]
```

### 条件查询
``` java
//        1.书写sql语句
String sql = "SELECT * FROM Customer WHERE cid=?";

//        2.创建sql查询对象
SQLQuery sqlQuery = session.createSQLQuery(sql);

//        3.给sql语句添加占位符信息
sqlQuery.setParameter(0, 1);

//        指定结果集封装到某个对象
sqlQuery.addEntity(Customer.class);

//        4.执行查询获得结果
Customer customer = (Customer) sqlQuery.uniqueResult();
```

### 分页查询
``` java
//        1.书写sql语句
String sql = "SELECT * FROM Customer LIMIT ?,?";

//        2.创建sql查询对象
SQLQuery sqlQuery = session.createSQLQuery(sql);

//        3.给sql语句添加占位符信息
sqlQuery.setParameter(0, 1);
sqlQuery.setParameter(1, 1);

//        指定将结果集封装到某个对象
sqlQuery.addEntity(Customer.class);

//        4.执行查询获得查询结果
Customer customer = (Customer) sqlQuery.uniqueResult();
```





































