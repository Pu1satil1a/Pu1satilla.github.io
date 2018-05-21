---
title: Mybatis(二)
description: Mybatis源码分析以及Mybatis增删改查
categories:
 - Web
 - Java
 - Mybatis
tags: [Web, Java, Mybatis]
---

# 源码分析

# 使用Mybatis步骤

1. 书写Dao接口以及实现类
2. 配置mapper文件
3. 配置主配置文件

## 主配置文件
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

    <!--
        typeAliases自定义别名元素:
            type：需要被命名对象
           alias：命名名称
    -->
    <typeAliases>
        <!--将指定包中所有类的简单类名当做别名-->
        <package name="cn.Pu1satilla.domain"/>
    </typeAliases>

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

    <!--注册映射文件-->
    <mappers>
        <mapper resource="cn/Pu1satilla/domain/mapper.xml"/>
    </mappers>

</configuration>
```

# 单表CURD

## select
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

### 查询所有条目

#### mapper配置
``` xml
    <!--查询所有条目-->
    <select id="selectAllStudent" resultType="Student">
        SELECT * FROM mybatis.student
    </select>
```

#### dao实现类方法
``` java
    @Override
    public List<Student> selectAllStudent() {
        try {
            return sqlSession.selectList("selectAllStudent");
        } finally {
            sqlSession.commit();
            sqlSession.close();
        }
    }
```

### 根据id查询单个条目

#### mapper配置
``` xml
    <!--根据id查询单个条目-->
    <select id="selectStudentById" resultType="Student">
        <!-- 这里的?是占位符，可以是任意内容 -->
        SELECT * FROM mybatis.student WHERE id=#{?}
    </select>
```

#### dao实现类方法
``` java
    @Override
    public Student selectStudent(int id) {
        try {
            return sqlSession.selectOne("selectStudentById",id);
        }finally {
            sqlSession.commit();
            sqlSession.close();
        }
    }
```

### 模糊查询
模糊查询需要进行字符串拼接，为了防止sql注入，需要以动态参数的形式出现在SQL语句中。

#### Statement查询（不推荐）
这种方式是纯粹的字符串拼接，直接将参数拼接到了sql语句中。这种方式可能会发生sql注入。
固定格式：${value}

##### mapper配置
``` xml
    <select id="selectStudentByLike" resultType="Student">
        SELECT *
        FROM mybatis.student
        WHERE name LIKE '%${value}%'
    </select>
```

#### 防止sql注入（PreparedStatement）
使用PreparedStatement更快，并能提供参数化设置，通过占位符指定，能够防止sql注入。

##### mapper配置
``` xml
    <!--模糊查询-->
    <select id="selectStudentByLike" resultType="Student">
        SELECT *
        FROM mybatis.student
        WHERE name LIKE '%' #{name} '%'
    </select>
```

#### dao实现类方法
``` xml
    public List<Student> selectStudentByName(String name) {
        try {
            return sqlSession.selectList("selectStudentByLike", name);
        } finally {
            sqlSession.commit();
            sqlSession.close();
        }
    }
```

## insert
与select元素不一致且重要的属性:

属性 |	描述
---- | ----
keyProperty 	|（仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 
useGeneratedKeys	|（仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。

### 将插入id返回给参数对象
mysql中sql语句中有一个方法能够获取最近插入id值
``` sql
SELECT last_insert_id();
```

mybatis完成将插入的对象元素id填充可以实际上是执行了sql查询最近插入id值的方法。

#### 方式一
mapper配置
``` xml
    <!--
     insert元素：
               id：SqlSession对象调用insert方法传入id作为参数用以调用该sql语句
    parameterType：
                方法传入参数类型，在主配置文件中通过typeAliases自定义别名元素
                | 指定package时Mybatis将扫描相应包
 useGeneratedKeys:
				获取数据库内部生成的主键
		 keyProperty：
                插入对象生成的id值
				-->
    <insert id="insertStudent" keyProperty="id" useGeneratedKeys="true">
        INSERT INTO mybatis.student (name, age, score)
        VALUES (#{name}, #{age}, #{score})
    </insert>
```

#### 方式二
通过selectKey元素来设置id
``` xml
    <!--
     insert元素：
               id：SqlSession对象调用insert方法传入id作为参数用以调用该sql语句
    -->
    <insert id="insertStudent" >
        INSERT INTO mybatis.student (name, age, score)
        VALUES (#{name}, #{age}, #{score})
        <selectKey order="AFTER" keyProperty="id" resultType="int">
            <!-- 
                selectKey属性：
                    order：查询语句执行在insert语句前或后
              keyProperty：插入对象属性
               resultType：返回值类型
             -->
            SELECT last_insert_id()
        </selectKey>
    </insert>
```

### dao实现类方法
``` java
    @Override
    public void insertStudent(Student student) {
        sqlSession.insert("insertStudent", student);

        System.out.println(student);
        sqlSession.commit();
        sqlSession.close();
    }
```

## delete

### mapper配置
``` xml
    <delete id="deleteStudent">
        <!-- 这里的id是占位符，也可以是任意内容 -->
        DELETE FROM mybatis.student WHERE id=#{id}
    </delete>
```

### dao实现类方法
``` java
    @Override
    public void deleteStudent(int id) {
        sqlSession.delete("deleteStudent",id);

        sqlSession.commit();
        sqlSession.close();
    }
```

## update

### mapper配置
``` xml
    <!--更新条目-->
    <update id="updateStudent">
        UPDATE mybatis.student
        SET name = #{name}, age = #{age}, score = #{score}
        WHERE id = #{id}
    </update>
```

### dao实现类方法
``` java
    @Override
    public void updateStudent(Student student) {
        sqlSession.update("updateStudent",student);

        sqlSession.commit();
        sqlSession.close();
    }
```













































































































