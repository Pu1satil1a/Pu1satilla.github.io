---
title: Mybatis(一)
description: Mybatis入门案例
categories:
 - Web
 - Java
 - Mybatis
tags: [Web, Java, Mybatis]
---

# Mybatis入门

## Mybatis下载与安装

### Mybatis下载
[Mybatis下载](https://github.com/mybatis/mybatis-3/releases)

### Mybatis导入
- 导入Mybatis相关包
![](/assets/images/Mybatis/Mybatis_download.PNG)

- 导入mysql驱动

## Mybatis简介
Mybatis是一个优秀的基于Java的持久层框架，它内部封装了JDBC，是开发者只需关注SQL语句本身，而不需要花费精力去处理注入注册驱动、创建Connection、配置Statement等复杂过程。

Mybatis通过xml或注解的方式将要执行各种statement(statement、preparedStatement等)配置起来，并通过Java对象和Statement中Sql的动态参数进行映射生成最终执行的SQL语句，最后由MyBatis框架执行Sql并将结果映射到Java对象并返回。

## Mybatis与Hibernate区别

Hibernate框架是提供了全面的数据库封装机制的“全自动”ORM，即实现了POJO和数据库表之间的映射，以及SQL的自动生成和执行。

相对于此，MyBatis只能算作是“半自动”ORM。其着力点，是在POJO类与SQL语句之间的映射关系。也就是说，MyBatis并不会为程序员自动生成SQL 语句。具体的SQL 需要程序员自己编写，然后通过SQL语句映射文件，将SQL所需的参数，以及返回的结果字段映射到指定POJO。因此，MyBatis成为了“全自动”ORM的一种有益补充。

MyBatis与Hibernate相比具有以下几个特点：
- 在xml文件中配置SQL语句，实现了SQL语句与代码分离，给程序的维护带来了很大的便利。
- 因为需要程序员自己去编写SQL语句，程序员可以结合数据库自身的特点灵活控制SQL语句，因此能够实现比Hibernate等全自动ORM框架更高的查询效率，能够完成复杂查询。
- 简单，易于学习，易于使用，上手快。

## MyBatis体系结构
![](/assets/images/Mybatis/Mybatis_structure.PNG)

## Mybatis工作原理

由应用程序执行Mybatis框架Api，而API封装了JDBC代码，框架执行相应代码从xml文件中获取sql语句，从而执行进行操作数据库。
![](/assets/images/Mybatis/Mybatis原理.PNG)

对象关系映射：
object-RelationShip-Mapping

## 入门
完成一次通过Mybatis操作数据库过程

### 导入jar包
```
ant-1.9.6.jar
ant-launcher-1.9.6.jar
asm-5.2.jar
cglib-3.2.5.jar
commons-logging-1.2.jar
javassist-3.22.0-GA.jar
log4j-1.2.17.jar
log4j-api-2.3.jar
log4j-core-2.3.jar
mybatis-3.4.6.jar
mysql-connector-java-5.1.37-bin.jar
ognl-3.1.16.jar
slf4j-api-1.7.25.jar
slf4j-log4j12-1.7.25.jar 
```
### domain包

``` java
package cn.Pu1satilla.domain;

public class Student {
    private Integer id;
    private String name;
    private int age;
    private double score;

    public Student() {

    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public double getScore() {
        return score;
    }

    public void setScore(double score) {
        this.score = score;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", score=" + score +
                '}';
    }
}
```
### 建立数据库表
``` sql
# 建立数据库
CREATE DATABASE mybatis CHARACTER SET 'utf8';
# 切换数据库
USE mybatis;
# 建对应Student类数据库表
DROP TABLE IF EXISTS `student` ;
CREATE TABLE student (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255),
  age INT ,
  score DOUBLE
)CHARSET 'utf8';
```

### 主配置文件

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--导入配置文件-->
    <properties resource="DruidUtils.properties"/>

    <!--配置log4j日志-->
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <!--配置运行环境-->
    <environments default="MySQLEM">
        <environment id="MySQLEM">

            <!-- 事务管理
                属性type - 表示使用JDBC默认管理
            -->
            <transactionManager type="JDBC"/>

            <dataSource type="POOLED">
            <!-- 数据库连接池
                属性type - 表示使用MyBatis自带连接池POOLED
            -->
                <property name="password" value="${jdbc.password}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="url" value="${jdbc.jdbcUrl}"/>
                <property name="driver" value="${jdbc.driverClassName}"/>
            </dataSource>
        </environment>
    </environments>

    <!--注册映射文件-->
    <mappers>
        <mapper resource="cn/Pu1satilla/domain/mapper.xml"/>
    </mappers>


</configuration>
```

### 配置mapping
在dao包下名为mapper.xml
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="test">
    <insert id="insertStudent" parameterType="cn.Pu1satilla.domain.Student">
        INSERT INTO mybatis.student (name, age, score)
        VALUES (#{name}, #{age}, #{score})
    </insert>
</mapper>
```

### 建立Dao包接口与类
``` java
//接口

public interface IStudentDao {
    void insertStudent(Student student);
}

//接口实现类

public class IStudentDaoImpl implements IStudentDao {
    private SqlSession sqlSession;
    @Override
    public void insertStudent(Student student) {

        try {
            //        1.加载主配置文件
            InputStream inputStream = Resources.getResourceAsStream("mybatis.xml");

            //        2.创建SqlSessionFactory对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

            //        3.创建SqlSession对象
            sqlSession = sqlSessionFactory.openSession();

            //        4.相关操作
            sqlSession.insert("insertStudent", student);

            //        5.提交SqlSession
            sqlSession.commit();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            sqlSession.close();
        }
    }
}

```

### 测试类
``` xml
public class Demo {
    private IStudentDaoImpl studentDao = new IStudentDaoImpl();

    @Test
    public void insertDemo() {
        Student zhangsan = new Student(null, "张三", 10, 50);
        studentDao.insertStudent(zhangsan);
    }
}
```

# MyBatis配置


## mapper配置

### 配置头文件
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

### 命名空间namespace
命名空间作用类似于java中的package，为了使不同包中的同名类作为区别。

在mapper中配置namespace属性用于区别不同xml文件中相同id。


### select元素
**select元素用于映射查询操作**

所有属性如下：

属性 	|描述
---- | ----
id 	|在命名空间中唯一的标识符，可以被用来引用这条语句。
parameterType 	|将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。
_parameterMap_	| _这是引用外部parameterMap的已经被废弃的方法。使用内联参数映射和parameterType属性。_
resultType 	|从这条语句中返回的期望类型的类的完全限定名或别名。注意如果是集合情形，那应该是集合可以包含的类型，而不能是集合本身。使用 resultType 或 resultMap，但不能同时使用。
resultMap 	|外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，对其有一个很好的理解的话，许多复杂映射的情形都能迎刃而解。使用 resultMap 或 resultType，但不能同时使用。
flushCache 	|将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false。
useCache 	|将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true。
timeout 	|这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。
fetchSize 	|这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动）。
statementType 	|STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。
resultSetType 	|FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）。
databaseId 	|如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。
resultOrdered 	|这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：false。
resultSets 	|这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。 

诸如insert、update和delete标签属性类似


## 主配置











































































































