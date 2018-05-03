---
title: Hibernate(一)
description: Hibernate框架的搭建
categories:
 - Web
 - Java
tags: [Web, Java]
---


# 什么是hibernate

## 什么是框架
- 框架是用来提高开发效率的
- 封装好了一些功能，我们需要使用这些功能时，即用即可，不需要再手动实现
- 框架可以理解成为一个半成品项目，只需要懂得如何使用

## 什么是hibernate框架
与DBUtils功能类似，帮我们完成数据库操作

## hibernate好处
操作数据库时候，可以以面向对象的方法来完成，不需要书写sql语句

## hibernate是一款orm框架
对象关系映射
```orm:object relation mapping```

通过配置，将对象的信息与数据库中的表进行对应，完成操作

# 导包
## hibernate依赖

![](/assets/images/hibernate/hibernate_lib.png)
[下载地址](/source/hibernate_lib)

## 驱动包
![](/assets/images/hibernate/mysql_driver_jar.png)
[下载地址](/source/mysql_driver/mysql-connector-java-5.1.7-bin.jar)

# 使用Idea搭建环境
1. 在当前project中创建一个module
![](/assets/images/hibernate/idea&hibernate1.png)
2. 选择web Application以及Hibernate项目，选择`Create default hibernate configuration and main class`，是直接创建项目的配置文件，并且选择下载或者使用本地的hibernate library
![](/assets/images/hibernate/idea&hibernate2.png)
3. 选择下载选项可以将下载的jar包copy一份保存到本地，以便创建其他项目使用，版本我选择的不是最新的，最新的hibernate可能会与其他依赖库不能兼容
![](/assets/images/hibernate/idea&hibernate3.png)
4. 创建之后在src文件下自动创建了一个配置文件`hibernate.cfg.xml` ,进行项目其他文件设置，打开project structure，给web项目创建WEB-INF和MATE-INF，设置目录
![](/assets/images/hibernate/idea&hibernate4.png)
![](/assets/images/hibernate/idea&hibernate5.png)
![](/assets/images/hibernate/idea&hibernate6.png)
5. 设置输出目录
![](/assets/images/hibernate/idea&hibernate7.png)
![](/assets/images/hibernate/idea&hibernate8.png)
6. 设置lib，在WEB-INF下新建lib目录，在module下添加lib为module目录
![](/assets/images/hibernate/idea&hibernate9.png)
7. 建立与数据库表一致的domain包下的JavaBean类，并且生成配置文件，选择对应自己选择数据库表
![](/assets/images/hibernate/idea&hibernate10.png)
![](/assets/images/hibernate/idea&hibernate11.png)
![](/assets/images/hibernate/idea&hibernate12.png)

# 主配置文件
## 必选属性配置（5个）
数据库驱动
``` xml 
<!--数据库驱动-->
<property name="connection.driver_class">com.mysql.jdbc.Driver</property>
```
数据库url
``` xml 
<!--数据库url-->
<property name="connection.url">jdbc:mysql://localhost:3306</property>
```
数据库连接用户名
``` xml 
<!--数据库连接用户名-->
<property name="connection.username">root</property>
```
数据库连接密码
``` xml 
<!--数据库连接密码-->
<property name="connection.password">feng8375</property>
```
数据库方言，不同数据库方言有所不同 ，sql语法略有区别，指定方言可以让hibernate框架针对指定数据库生成sql语句，**MySQL在选择方言时选择最短的**
``` xml 
<!--数据库方言-->
<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
```

## 可选属性配置（3个）
将hibernate生成的sql语句打印到控制台
``` xml 
<!--将hibernate生成的sql语句打印到控制台-->
<property name="hibernate.show_sql">true</property>
```
将hibernate生成的sql语句格式化
``` xml 
<!--将hibernate生成的sql语句格式化（自动缩进）-->
<property name="hibernate.format_sql">true</property>
```
自动导出表结构
``` xml 
<!--自动导出表结构
hibernate.hbm2ddl.auto create       自动建表，每次框架运行都会创建新的表，以前表将会被覆盖，表数据会丢失（开发环境中测试使用）
hibernate.hbm2ddl.auto create-drop  自动建表并且删除，每次框架运行都会创建新的表，运行结束都会讲所有表删除（开发环境中测试使用）
hibernate.hbm2ddl.auto update       自动生成表，如果已经存在，不会再生成，如果表变动，会自动更新表（不会删除任何数据，新的属性会创建新列）
hibernate.hbm2ddl.auto validate     校验，不自动生成表，每次启动会校验数据库表是否正确，校验失败（抛出SchemaManagementException异常）
-->
<property name="hibernate.hbm2ddl.auto">update</property>
```
## 引入orm元数据
``` xml 
<!--引入orm元数据
	路径:src下的路径
-->
<mapping resource="com/Pu1satilla/domain/UserEntity.hbm.xml"/>
<mapping class="com.Pu1satilla.domain.UserEntity"/>
```
## 整合
``` xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--必选-->
        <!--数据库url-->
        <property name="connection.url">jdbc:mysql://localhost:3306</property>
        <!--数据库驱动-->
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <!--数据库连接用户名-->
        <property name="connection.username">root</property>
        <!--数据库连接密码-->
        <property name="connection.password">feng8375</property>
        <!--数据库方言-->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <!--可选-->
        <!--将hibernate生成的sql语句打印到控制台-->
        <property name="hibernate.show_sql">true</property>
        <!--将hibernate生成的sql语句格式化（自动缩进）-->
        <property name="hibernate.format_sql">true</property>
        <!--自动导出表结构
        hibernate.hbm2ddl.auto create       自动建表，每次框架运行都会创建新的表，以前表将会被覆盖，表数据会丢失（开发环境中测试使用）
        hibernate.hbm2ddl.auto create-drop  自动建表并且删除，每次框架运行都会创建新的表，运行结束都会讲所有表删除（开发环境中测试使用）
        hibernate.hbm2ddl.auto update       自动生成表，如果已经存在，不会再生成，如果表变动，会自动更新表（不会删除任何数据，新的属性会创建新列）
        hibernate.hbm2ddl.auto validate     校验，不自动生成表，每次启动会校验数据库表是否正确，校验失败（抛出SchemaManagementException异常）
        -->
        <property name="hibernate.hbm2ddl.auto">update</property>
        <!--引入orm元数据
            路径:src下的路径
        -->
        <mapping resource="com/Pu1satilla/domain/UserEntity.hbm.xml"/>
        <mapping class="com.Pu1satilla.domain.UserEntity"/>

    </session-factory>
</hibernate-configuration>
```
# orm元数据

## 根元素
``` xml
<!--配置表与实体对象的关系-->
<!--package属性：填写一个包名，在元素内部凡是需要书写完整类名的属性，可以直接写简单类名-->
<hibernate-mapping package="com.Pu1satilla.domain"></hibernate-mapping>
```
## class元素
``` xml
<!--配置实体与表的对应关系，name：完整类名，table：数据库表名，schema：数据库名-->
<class name="com.Pu1satilla.domain.UserEntity" table="user" schema="book_store"></class>
```
## id元素
``` xml
<!--id配置主键映射属性，
name填写主键对应属性，
column填写表中主键列名（可选），填写表中主键名，不填列名会默认使用属性名，
type（可选）：填写列（属性）的类型，hibernate会自动检测实体的属性类型，
	每个类型有三种填法（建议不填）：java类型|hibernate类型|数据库类型
not-null（可选）：配置该属性（列）是否不能为空，默认值为false，
length（可选）：配置数据库中列的长度，默认值：使用数据库类型的最大长度
-->
<id name="uid" column="uid"/>
```
## property元素
``` xml
<!--property配置除id外普通属性映射，
name填写属性，
column填写表中列名（可选），填写表中主键名，不填列名会默认使用属性名，
type（可选）：填写列（属性）的类型，hibernate会自动检测实体的属性类型，
	每个类型有三种填法（建议不填）：java类型|hibernate类型|数据库类型
not-null（可选）：配置该属性（列）是否不能为空，默认值为false，
length（可选）：配置数据库中列的长度，默认值：使用数据库类型的最大长度
-->
<property name="username" column="username"/>
<property name="password" column="password"/>
<property name="email" column="email"/>
<property name="code" column="code"/>
<property name="state" column="state"/>
```
## 整合
``` xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
		
<!--根元素：配置表与实体对象的关系-->
<!--package属性：填写一个包名，在元素内部凡是需要书写完整类名的属性，可以直接写简单类名-->
<hibernate-mapping package="com.Pu1satilla.domain">

    <!--class元素：配置实体与表的对应关系，name：完整类名，table：数据库表名，schema：数据库名-->
    <class name="com.Pu1satilla.domain.UserEntity" table="user" schema="book_store">
	
        <!--id配置主键映射属性，
        name填写主键对应属性，
        column填写表中主键列名（可选），填写表中主键名，不填列名会默认使用属性名，
        type（可选）：填写列（属性）的类型，hibernate会自动检测实体的属性类型，
            每个类型有三种填法（建议不填）：java类型|hibernate类型|数据库类型
        not-null（可选）：配置该属性（列）是否不能为空，默认值为false，
        length（可选）：配置数据库中列的长度，默认值：使用数据库类型的最大长度
        -->
        <id name="uid" column="uid"/>
		
        <!--property配置除id外普通属性映射，
        name填写属性，
        column填写表中列名（可选），填写表中主键名，不填列名会默认使用属性名，
        type（可选）：填写列（属性）的类型，hibernate会自动检测实体的属性类型，
            每个类型有三种填法（建议不填）：java类型|hibernate类型|数据库类型
        not-null（可选）：配置该属性（列）是否不能为空，默认值为false，
        length（可选）：配置数据库中列的长度，默认值：使用数据库类型的最大长度
        -->
        <property name="username" column="username"/>
        <property name="password" column="password"/>
        <property name="email" column="email"/>
        <property name="code" column="code"/>
        <property name="state" column="state"/>
    </class>
</hibernate-mapping>
```
# hibernate常见api

## Configuration

### 功能
配置加载类，用于加载主配置，orm元数据加载

### 创建
``` java
//创建，调用空参构造
Configuration conf = new Configuration();
```
### 加载主配置
``` java
//读取指定主配置文件=>空参加载方法，加载src下的hibernate
conf.configuration();
```
### 加载orm元数据（了解）
``` java
//读取指定orm元数据，如果主配置中已经引入映射配置，不需要手动加载
conf.addResource(ResourceName);
conf.addClass(persistentClass);
```
### 创建sessionFactory
``` java
//根据配置信息，创建SessionFactory对象
SessionFactory sf = con.buildSessionFactory();
```

## SessionFactory
### 功能
用于创建操作数据库核心对象session对象的工厂
**注意**  
1. sessionFactory 负责保存和使用所有配置信息，消耗内存资源非常大
2. sessionFactory 属于线程安全的对象设计

**结论**  
保证在web项目中，只创建一个sessionFactory()

### 获得session对象
``` java
Session session = sf.openSession();
```
### 获得一个与线程绑定的session对象
``` java
sf.getCurrentSession();
```

## Session（Hibernate框架与数据库之间的连接）

### 功能
类似于JDBC的connection对象，还可以完成对数据库中数据的增删改查操作，Session是Hibernate操作数据库的核心对象

### 操作事务 
``` java
//获得操作事务的对象
Transaction transaction = session.getTransaction();
```
### 获取操作事务的对象，并且开启事务
``` java
Transaction transaction = session.beginTransaction();
```

### session的操作
``` java
//1. 添加数据库列

//1.1 创建user对象
User user = new User();
//1.2 完善user对象
user.setUsername("小明");
//1.3 将对象映射插入到数据库表中
session.save(user);

//2. 删除数据库列（id）

User user = session.get(User.class,)

//3. 修改数据库列（id）

//4. 查询数据库列（id）

//4.1 获取列对应的对象（id,1l对应，1表示id为1的列，l表示id的类型为long型）
User user = session.get(Customer.class,1l);

```

### 事务提交与回滚
``` java
transaction.commit();
transaction.rollback();
```

### 释放资源
``` java
//释放session资源
session.close();
//释放sessionFactory资源
sessionFactory.close();
```











