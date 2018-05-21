---
title: Mybatis(三)
description: 单表查询扩展，包含属性名与查询字段名不相同，Mapper动态代理，动态SQL
categories:
- Web
- Java
- Mybatis
tags: [Web, Java, Mybatis]
---



# 属性名与查询字段不一致

开发时有可能分工不同，例如A负责数据库表设计，B负责后端代码，这样可能导致数据库字段与类实体属性名与查询字段不一致，Mybatis无法将数据封装到实体中。

## 使用别名
``` xml
    <!--
        查询所有条目
        前面是数据库中字段，后面是实体属性
    -->
    <select id="selectAllStudent" resultType="Student1">
        SELECT
            id   Sid,
            age  Sage,
            name Sname,
            score
        FROM mybatis.student
    </select>
```

## 通过映射配置resultMap
``` xml
    <!--
        配置结果映射
            id作为当前resultMap的id，为其他元素调用做使用
            type为结果类型
            result column对应数据库条目
            result property对应实体属性
    -->
    <resultMap id="StudentMapper" type="Student1">
        <id column="id" property="Sid"/>
        <result column="name" property="Sname"/>
        <result column="age" property="Sage"/>
    </resultMap>

    <!--
        查询所有条目
        前面是数据库中字段，后面是实体属性
        resultMap属性为映射方式
    -->
    <select id="selectAllStudent" resultMap="StudentMapper">
        SELECT *
        FROM mybatis.student
    </select>
```

# Mapper动态代理

通过书写上方代码发现一个问题：Dao的实现类其实并没有干什么实质性的工作，它仅仅是通过SqlSession的相关API定位到映射文件mapper中的相应SQL语句，对DB操作。这种对Dao的实现方式称为Mapper的动态代理方式。

## mapper配置
想要实现Mapper动态代理，需要进行xml文件配置

### mapper命名空间
一般情况下，一个Dao接口的实现类方法使用的是同一个SQL映射文件中的SQL映射id。所以，Mybatis框架要求，将映射文件中<mapper/>标签的namespace属性设为Dao接口的全雷鸣，则系统会根据方法所属Dao接口，自动到相应namespace的映射文件中查找相关SQL映射。
``` xml
<mapper namespace="cn.Pu1satilla.dao.IStudentDao">
```

**注意：接口名必须与mapper文件中id值对应**

### mapper配置
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--属性namespace：
        类似于Java中的包名,通常指定对应表名
-->
<mapper namespace="cn.Pu1satilla.dao.IStudentDao">

    <!--
     insert元素：
               id：SqlSession对象调用insert方法传入id作为参数用以调用该sql语句
    -->
    <insert id="insertStudent">
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

    <!--删除条目-->
    <delete id="deleteStudent">
        <!-- 这里的id是占位符，也可以是任意内容 -->
        DELETE FROM mybatis.student WHERE id=#{id}
    </delete>

    <!--更新条目-->
    <update id="updateStudent">
        UPDATE mybatis.student
        SET name = #{name}, age = #{age}, score = #{score}
        WHERE id = #{id}
    </update>

    <!--
        配置结果映射
            id作为当前resultMap的id，为其他元素调用做使用
            type为结果类型
            result column对应数据库条目
            result property对应实体属性
    -->
    <resultMap id="StudentMapper" type="Student1">
        <id column="id" property="Sid"/>
        <result column="name" property="Sname"/>
        <result column="age" property="Sage"/>
    </resultMap>

    <!--
        查询所有条目
        前面是数据库中字段，后面是实体属性
        resultMap属性为映射方式
    -->
    <select id="selectAllStudent" resultMap="StudentMapper">
        SELECT *
        FROM mybatis.student
    </select>

    <!--根据id查询单个条目-->
    <select id="selectStudent" resultType="Student">
        <!-- 这里的?是占位符，可以是任意内容 -->
        SELECT * FROM mybatis.student WHERE id=#{?}
    </select>

    <!--模糊查询-->
    <select id="selectStudentByName" resultType="Student">
        SELECT *
        FROM mybatis.student
        WHERE name LIKE '%${value}%'
    </select>
</mapper>
```

### 接口类
``` java
public interface IStudentDao {

    //    根据对象添加
    void insertStudent(Student student);

    //    根据id删除条目
    void deleteStudent(int id);

    //    根据对象修改数据库条目
    void updateStudent(Student student);

    //    根据id查询
    Student selectStudent(int id);

    //    查询所有数据库中条目
    List<Student> selectAllStudent();

    //    根据年龄查询
    List<Student> selectStudentByName(String name);
}

```

### 测试类
``` xml
public class Demo1 {
    private SqlSession sqlSession = MybatisUtils.getSqlSession();
    private IStudentDao studentDao = sqlSession.getMapper(IStudentDao.class);

    @After
    public void after() {
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void demo1() {
        Student student = new Student(null, "白柏武", 50, 50);
        studentDao.insertStudent(student);
    }

    @Test
    public void demo2() {
        studentDao.deleteStudent(34);
    }

    @Test
    public void demo3() {
        Student student = new Student(33, "赛利亚", 18, 99.9);
        studentDao.updateStudent(student);
    }

    @Test
    public void demo4() {
        Student student = studentDao.selectStudent(33);
        System.out.println(student);
    }

    @Test
    public void demo5() {
        List<Student> students = studentDao.selectAllStudent();
        System.out.println(students);
    }

    @Test
    public void demo6() {
        List<Student> students = studentDao.selectStudentByName("赛");
        System.out.println(students);
    }
}
```














































































































