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
类似可以将Service层注入Spring容器

# spring中aop事务


































































































