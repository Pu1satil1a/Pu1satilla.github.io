---
title: Spring(五)
description: spring整合JDBC以及spring中aop事物
categories:
 - Web
 - Java
 - Spring
tags: [Web, Java, Spring]
---

# spring提供JDBC

## 导包
![](/assets/images/Spring/aop_configuration.png)

## 测试JDBC连接
``` java
@Test
public void func1() throws IOException {
	DataSource dateSource = DruidUtils.getDruidDateSource();

	JdbcTemplate jdbcTemplate = new JdbcTemplate(dateSource);

	String sql= "INSERT INTO demo VALUES(null,?)";

	jdbcTemplate.update(sql,"林伟祥");
}
```

## 注入DruidDataSource

### 在src文件下书写properties

``` properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.password=*******
jdbc.username=root
jdbc.jdbcUrl=jdbc:mysql://localhost:3306/spring?useUnicode=true&characterEncoding=utf8
jdbc.initialSize=5
jdbc.minIdle=5
jdbc.maxActive=20
```

### xml配置

注入阿里的DruidDataSource连接池dataSource，并且将其作为JdbcTemplate属性
``` xml
    <!--load properties-->
    <context:property-placeholder location="DruidUtils.properties"/>

    <bean class="com.alibaba.druid.pool.DruidDataSource" name="dataSource" destroy-method="close">
        <property name="url" value="${jdbc.jdbcUrl}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

## JDBC常用API
与Dbutils类似，Dbutils的QueryRunner类似于SpringJDBC的JdbcTemplate，学习如何使用SpringJDBC，根据下列常用API

1. execute方法：可以用于执行任何sql语句，一般用于执行DDL语句
2. update方法|batchUpdate方法：update方法用于执行新增、修改 、删除等语句；batchUpdate方法用于执行批处理相关语句
3. query方法及queryForXXX方法：用于执行查询相关语句
4. call方法：用于执行存储过程、函数相关语句

### 增
``` java
		//        获取JDBCTemplate
        JdbcTemplate jt = new JdbcTemplate(dataSource);

        //        增加条目
        jt.update("INSERT INTO demo VALUES (NULL ,'Pulsatilla')");

        /*
         *        批量增加条目
         *        最后一个参数是 Object[] 的 List 类型：因为修改一条记录需要一个 Object 数组，
         *        修改多条记录就需要一个 List 来存放多个数组
         */
        List<Object[]> list = new ArrayList<>();
        list.add(new Object[]{10,"张三"});
        list.add(new Object[]{11,"李四"});
        list.add(new Object[]{12,"王五"});
        list.add(new Object[]{13,"赵柳"});
        list.add(new Object[]{14,"孙八"});
        jt.batchUpdate("INSERT INTO demo VALUES (?,?)",list);
```

### 删
``` java
        //        删除条目
        jt.update("DELETE FROM demo WHERE id=1");
```

### 改
``` java
        //        修改条目
        jt.update("UPDATE demo SET 'name'='霞洛' WHERE id=5");
```

### 查询
``` java
        /*
          查询条目
          从数据库中获取一条记录，实际得到对应的一个对象
          注意：不是调用 queryForObject(String sql, Class<Employee> requiredType, Object... args) 方法!
          而需要调用    queryForObject(String sql, RowMapper<Employee> rowMapper, Object... args)
          1、其中的 RowMapper 指定如何去映射结果集的行，常用的实现类为 BeanPropertyRowMapper
          2、使用 SQL中的列的别名完成列名和类的属性名的映射，例如 last_name lastName
          3、不支持级联属性。 JdbcTemplate 只能作为一个 JDBC 的小工具, 而不是 ORM 框架
         */
		 //	  查询单个条目
        User user = jt.queryForObject("SELECT * FROM demo WHERE id=?", new BeanPropertyRowMapper<>(User.class), 1);

        //        查询多个条目(注意不是调用的queryForList)
        List<User> query = jt.query("SELECT * FROM demo", new BeanPropertyRowMapper<>(User.class));

        //        查询统计条目
        Integer aLong = jt.queryForObject("SELECT count(id) FROM demo", Integer.class);
        System.out.println(aLong);
```

## 注入JdbcTemplate
``` xml
    <!--注入druid阿里连接池-->
    <bean class="com.alibaba.druid.pool.DruidDataSource" name="dataSource" destroy-method="close">
        <property name="url" value="${jdbc.jdbcUrl}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--将DruidDataSource作为属性注入JdbcTemplate-->
    <bean class="org.springframework.jdbc.core.JdbcTemplate" name="jdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

## 注入Dao实现类
``` xml
    <!--将JdbcTemplate注入DEmoDaoImpl-->
    <bean class="cn.Pu1satilla.Dao.DemoDaoImpl" name="dao">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>
```
类似可以将被注入dao层的Service层注入Spring容器

# spring中aop事务

## spring事务操作对象
因为不同平台（hibernate、mybatis、springJdbc）对事务操作的代码不同，spring想操作不同平台的事务，提供PlatforTransactiionManager接口，并提供相应平台的实现类。
- （springJdbc）DataSourceTransactionManager
- （hibernate）HibernateTransactionManager

**注意：在spring中事务管理最为核心的对象就是TransactionManager对象**

通过这些类的方法完成spring对事务操作

## spring管理事务的属性

1. 事务隔离级别(isolation)
- 读未提交
- 读已提交
- 可重复读
- 串行化
2. 是否只读(read-only)
- true
- false
3. 事务的传播行为(propagation)
- **PROPACATION_REQUIED**：支持当前事务，如果不存在就新建一个(默认)
- PROPAGATION_SUPPORTS：支持当前事务，如果不存在，就不使用事务
- PROPAGATION_MANDATORY：支持当前事务，如果不存在，抛出异常
- **PROPAGATION_REQUIRES_NEW**：如果有事务存在，挂起当前事务，创建一个新的事务
- PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果有事务存在，挂起当前事务
- PROPAGATION_NEVER：以非事务方式运行，如果有事务存在，抛出异常
- **PROPAGATION_NESTED**：如果当前事务存在，则嵌套事务执行

## spring管理事务的方式
 
### 完成Dao层以及service层设计

#### JdbcDaoSupport
JdbcDaoSupport作用：此基类主要用于JdbcTemplate使用，但也可以在直接使用Connection或使用org.springframework.jdbc.object操作对象时使用 。**多用于Dao层类继承，可以简化Dao层类代码**
 
#### Dao类
``` java
package cn.Pu1satilla.dao;

import org.springframework.jdbc.core.support.JdbcDaoSupport;


/*
 * @Classname:      AccountDaoImpl
 * @Description:    Dao层完成转账操作，实现JdbcDaoSupport，该类简化Dao实现。声明了jdbcTemplate属性
 * @Author          fy [939902332feng@gmail.com]
 * @Date            2018/5/16 17:05
 *
 */
public class AccountDaoImpl extends JdbcDaoSupport implements AccountDao {

    /**
     * 收款人收账
     *
     * @param id    收账人id
     * @param money 收账金额
     */
    @Override
    public void addMoney(Integer id, Double money) {
        getJdbcTemplate().update("UPDATE account SET money=money+? WHERE id=?", money, id);
    }

    /**
     * 转账人转账
     *
     * @param id    转账人id
     * @param money 转账金额
     */
    @Override
    public void decreaseMoney(Integer id, Double money) {
        getJdbcTemplate().update("UPDATE account SET money=money-? WHERE id=?", money, id);
    }
}
```
#### Servcie层类
``` xml 
package cn.Pu1satilla.dao;



/*
 * @Classname:      AccountServiceImpl
 * @Description:    业务层转账操作，实现AccounService接口，演示spring中aop事务操作的service层
 * @Author          fy [939902332feng@gmail.com]
 * @Date            2018/5/16 17:27
 *
 */
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    /**
     * 通过spring aop配置完成事务方法切入
     *
     * @param from  转账人id
     * @param to    收款人id
     * @param money 转账金额
     */
    @Override
    public void transfer(int from, int to, double money) {

        //        转账人金额
        accountDao.decreaseMoney(from, money);

        //        测试出现异常，spring提供aop事务操作是否有效
        int i = 1 / 0;

        //        收款人金额
        accountDao.addMoney(to, money);
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
}
```
#### 测试类
``` java
package cn.Pu1satilla.demo;


import cn.Pu1satilla.dao.AccountService;
import cn.Pu1satilla.dao.AccountServiceImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/*
 * @Classname:      demo
 * @Description:    测试类，测试转账事务是否通过spring aop配置成功
 * @Author          fy [939902332feng@gmail.com]
 * @Date            2018/5/16 18:26
 *
 */
public class demo {
    public static void main(String[] args) {

        //        1.建立spring 容器
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");

        //        2.获取容器中service对象
        AccountService accountService = (AccountService) applicationContext.getBean("accountService");

        //        3.调用dao对象转账方法
        accountService.transfer(1,2,100);
    }
}
```
 
### xml配置（aop）

#### 导入约束
``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context
                            http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop
                            http://www.springframework.org/schema/aop/spring-aop.xsd
                            http://www.springframework.org/schema/tx
                            http://www.springframework.org/schema/tx/spring-tx.xsd">
```

#### DataSource导入配置文件内容
``` xml
    <!--从配置文件注入DruidDataSource-->
    <context:property-placeholder location="DruidUtils.properties"/>

    <bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url" value="${jdbc.jdbcUrl}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="initialSize" value="${jdbc.initialSize}"/>
        <property name="minIdle" value="${jdbc.minIdle}"/>
        <property name="maxActive" value="${jdbc.maxActive}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

#### 配置事务核心管理类
将连接池注入dao，dao类可以继承JdbcSupportDao类，具有方法getJdbcTemplate可直接获得JdbcTemplate，通过xml配置就能注入，无需再次声明
``` xml
    <!--将连接池注入Dao-->
    <bean name="accountDao" class="cn.Pu1satilla.dao.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

#### 将DataSource注入dao

#### 将dao类注入到service

#### 配置事务通知

#### 将通知织入切入点

### 注解配置（aop）























































































