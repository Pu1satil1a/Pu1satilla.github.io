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

## Mybatis下载与导入

下载  
[Mybatis下载](https://github.com/mybatis/mybatis-3/releases)


导入以及mysql驱动
![](/assets/images/Mybatis/Mybatis_download.PNG)

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
![](/assets/images/Mybatis/Mybatis_principle.PNG)

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




# mapper配置

## 配置头文件
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

## 命名空间namespace
命名空间作用类似于java中的package，为了使不同包中的同名类作为区别。

在mapper中配置namespace属性用于区别不同xml文件中相同id。

## Crud包含属性

### parameterType
**parameterType可以省略不写！**

将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。

通过设置parameterType值，在sql语句中使用表达式${}来获取传入对象属性（底层使用getXxx()方法）。


# 主配置

主配置位置一般位于src目录下，所包含元素需要按顺序排列，否则会报错。（一般命名为mybatis.xml，不强行要求），在博客中将挑选出频繁使用的内容，其他可以参照[官网](http://www.mybatis.org/mybatis-3/zh/configuration.html#)

```
configuration 配置
	|-- properties 属性
	|-- settings 设置
	|-- typeAliases 类型别名
	|-- typeHandlers 类型处理器
	|-- objectFactory 对象工厂
	|-- plugins 插件
	|-- environments 环境
	|		|-- environment 环境变量
	|				|-- transactionManager 事务管理器
	|				|-- dataSource 数据源
	|
	|-- databaseIdProvider 数据库厂商标识
	|-- mappers 映射器
```

## properties属性
可以作为自定义键值对或者导入配置文件。一般用于导入配置文件。
``` xml
    <!--导入配置文件-->
    <properties resource="DruidUtils.properties"/>
```
导入完成后，在xml文件可以通过`${}`表达式来获取配置文件中内容。

例如：
``` xml
	<dataSource type="POOLED">
	<!-- 数据库连接池
		属性type - 表示使用MyBatis自带连接池POOLED
	-->
		<property name="password" value="${jdbc.password}"/>
		<property name="username" value="${jdbc.username}"/>
		<property name="url" value="${jdbc.jdbcUrl}"/>
		<property name="driver" value="${jdbc.driverClassName}"/>
	</dataSource>
```

## setting属性

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。下表描述了设置中各项的意图、默认值等。

设置参数	|描述	|有效值	|默认值
---- | ---- | ---- | ----
cacheEnabled	| 全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。	|true / false	|true
lazyLoadingEnabled	|延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态。	|true /false	|false
aggressiveLazyLoading	|当开启时，任何方法的调用都会加载该对象的所有属性。否则，每个属性会按需加载（参考lazyLoadTriggerMethods).	|true /false	|false (true in ≤3.4.1)
multipleResultSetsEnabled	|是否允许单一语句返回多结果集（需要兼容驱动）。	|true /false	|true
useColumnLabel	|使用列标签代替列名。不同的驱动在这方面会有不同的表现， 具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。	|true / false	|true
useGeneratedKeys	|允许 JDBC 支持自动生成主键，需要驱动兼容。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby）。	|true /false	|False
autoMappingBehavior	|指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。	|NONE, PARTIAL, FULL	|PARTIAL
autoMappingUnknownColumnBehavior	|指定发现自动映射目标未知列（或者未知属性类型）的行为。NONE: 不做任何反应WARNING: 输出提醒日志 ('org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN)FAILING: 映射失败 (抛出 SqlSessionException)|NONE, WARNING, FAILING| NONE
defaultExecutorType	|配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。	|SIMPLE REUSE BATCH	|SIMPLE
defaultStatementTimeout	|设置超时时间，它决定驱动等待数据库响应的秒数。	|任意正整数	|Not Set (null)
defaultFetchSize	|为驱动的结果集获取数量（fetchSize）设置一个提示值。此参数只可以在查询设置中被覆盖。	|任意正整数	|Not Set (null)
safeRowBoundsEnabled	|允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为false。	|true /false	|False
safeResultHandlerEnabled	|允许在嵌套语句中使用分页（ResultHandler）。如果允许使用则设置为false。	|true /false	|True
mapUnderscoreToCamelCase	|是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。|	true / false	|False
localCacheScope	|MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。	|SESSION / STATEMENT	|SESSION
jdbcTypeForNull	|当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。	|JdbcType 常量. 大多都为: NULL, VARCHAR and OTHER	|OTHER
lazyLoadTriggerMethods	|指定哪个对象的方法触发一次延迟加载。|	用逗号分隔的方法列表。	|equals,clone,hashCode,toString
defaultScriptingLanguage	|指定动态 SQL 生成的默认语言。	|一个类型别名或完全限定类名。|	org.apache.ibatis.scripting.xmltags.XMLLanguageDriver
defaultEnumTypeHandler	|指定 Enum 使用的默认 TypeHandler 。 (从3.4.5开始)	一|个类型别名或完全限定类名。	|org.apache.ibatis.type.EnumTypeHandler
callSettersOnNulls	|指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这对于有 Map.keySet() 依赖或 null 值初始化的时候是有用的。注意基本类型（int、boolean等）是不能设置成 null 的。|	true /false	|false
returnInstanceForEmptyRow	|当返回行的所有列都是空时，MyBatis默认返回null。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集 (i.e. collectioin and association)。（从3.4.2开始）|	true /false	|false
logPrefix	|指定 MyBatis 增加到日志名称的前缀。	|任何字符串|	Not set
logImpl	|指定 MyBatis 所用日志的具体实现，未指定时将自动查找。	|SLF4J/ LOG4J / LOG4J2 / JDK_LOGGING / COMMONS_LOGGING / STDOUT_LOGGING / NO_LOGGING	|Not set
proxyFactory	|指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具。	|CGLIB / JAVASSIST	|JAVASSIST (MyBatis 3.3 or above)
vfsImpl	|指定VFS的实现	|自定义VFS的实现的类全限定名，以逗号分隔。	|Not set
useActualParamName	|允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的工程必须采用Java 8编译，并且加上-parameters选项。（从3.4.1开始）	|true / false	|true
configurationFactory	|指定一个提供Configuration实例的类。 这个被返回的Configuration实例用来加载被反序列化对象的懒加载属性值。 |这个类必须包含一个签名方法static Configuration getConfiguration(). (从 3.2.3 版本开始)	类型别名或者全类名.	|Not set

## typeAliases元素
类型别名是为 Java 类型设置一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。

### typeAlias属性
typeAliases属性可以将需要对象命名成自己需要，简化类书写
``` xml
    <!--
        typeAliases自定义别名元素:
            type：需要被命名对象
           alias：命名名称
    -->
    <typeAliases>
        <!--将需要被命名对象命名为Student，在mapper直接使用Student-->
        <typeAlias type="cn.Pu1satilla.domain.Student" alias="Student"/>
    </typeAliases>
```
当在主配置文件中配置时，在mapper文件的parameterType可以使用简化后名称，如下：
``` xml
    <insert id="insertStudent" parameterType="Student" >
        INSERT INTO mybatis.student (name, age, score)
        VALUES (#{name}, #{age}, #{score})
    </insert>
```

### package
对于实体类的全限定性类名的别名指定方式，一般使用`<package/>`方式，这样做的好处是会将包中所有实体类的简单类型指定为别名，写法简单方便。
``` xml
    <typeAliases>
        <!--将指定包中所有类的简单类名当做别名-->
        <package name="cn.Pu1satilla.domain"/>
    </typeAliases>
```

相较于typeAlias属性指定，**优势是不需要逐个指定。**

## 配置环境（environments）

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者共享相同 Schema 的多个生产数据库。

**不过要记住：尽管可以配置多个环境，每个 SqlSessionFactory 实例只能选择其一。**

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

例子：
``` xml
    <!--
        配置运行环境
        default >>>> 通过id值指定某个默认环境
    -->
    <environments default="MySQL">

        <environment id="MySQL">

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

        <environment id="Oracle">
            <transactionManager type="JDBC"/>
            <dataSource type="Druid">
                
            </dataSource>
        </environment>
    </environments>
```
**配置environments关键点：**

- 默认的环境 ID（比如:default="development"）。
- 每个 environment 元素定义的环境 ID（比如:id="development"）。
- 事务管理器的配置（比如:type="JDBC"）。
- 数据源的配置（比如:type="POOLED"）。

## 指定映射文件
通过四种方式指定文件：
- **resource**
``` xml
    <!--注册单个映射文件-->
    <mappers>
        <mapper resource="cn/Pu1satilla/domain/mapper.xml"/>
    </mappers>
	
	<!--注册多个映射文件-->
    <mappers>
        <mapper resource="cn/Pu1satilla/domain/mapper.xml"/>
        <mapper resource="cn/Pu1satilla/domain/mapper2.xml"/>
    </mappers>
```
- url（了解）
- class
- package

# 工具类，简化代码
``` xml
/*
 * @Classname:      MybatisUtils
 * @Description:    Mybatis工具类，用于简化代码，避免代码过于冗余，简化了获取sql操作会话对象
 * @Author          fy [939902332feng@gmail.com]
 * @Date            2018/5/19 19:14
 *
 */
public class MybatisUtils {

    //    声明工厂用于存储会话连接对象
    private static SqlSessionFactory sessionFactory;

    //    类首次加载时，静态代码块用于建立会话工厂
    static {
        try {
            sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis.xml"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取数据库会话连接对象
     *
     * @return 数据库连接会话对象
     */
    public static SqlSession getSqlSession() {
        return sessionFactory.openSession();
    }
}
```



















































































