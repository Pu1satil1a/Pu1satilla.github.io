---
title: Mybatis(四)
description: Mybatis关联关系查询，Mybatis延迟加载
categories:
 - Web
 - Java
 - Mybatis
tags: [Web, Java, Mybatis]
---


# 一对多关联查询
一对多关联查询是指，在查询一方对象的时候，同时将其所关联的多方对象也都查询出来。

## Java实体
``` java
//		类别实体
public class Category {
    private Integer cid;
    private String cname;
    private Set<Animal> animals;

    public Integer getCid() {
        return cid;
    }

    public void setCid(Integer cid) {
        this.cid = cid;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }

    public Set<Animal> getAnimals() {
        return animals;
    }

    public void setAnimals(Set<Animal> animals) {
        this.animals = animals;
    }

    @Override
    public String toString() {
        return "Category{" +
                "cid=" + cid +
                ", cname='" + cname + '\'' +
                ", animals=" + animals +
                '}';
    }
}

//		动物实体
public class Animal {
    private String aname;
    private Integer aid;
    private Category category;

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    public String getAname() {
        return aname;
    }

    public void setAname(String aname) {
        this.aname = aname;
    }

    public Integer getAid() {
        return aid;
    }

    public void setAid(Integer aid) {
        this.aid = aid;
    }

    @Override
    public String toString() {
        return "Animal{" +
                "aname='" + aname + '\'' +
                ", aid=" + aid +
                ", category=" + category +
                '}';
    }
}
```

## 新建关联表

``` sql
CREATE TABLE category (
  cid  INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255)
)
  CHARSET 'utf8';

CREATE TABLE animal (
  aid  INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255),
  cid  INT,
  FOREIGN KEY (cid) REFERENCES category (cid)
)
  CHARSET 'utf8';
```
插入数据库条目
![](/assets/images/Mybatis/animals.png)
![](/assets/images/Mybatis/categories.png)

## 配置xml文件
当双表对应外键名称相同（双关）时，只能通过映射的方式配置，否则mybatis不能识别查询的键对应为哪个表。
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--确定命名空间-->
<mapper namespace="cn.Pu1satilla.dao.CategoryDao">

    <!--
        映射标签
            column标签对应数据库表中字段，property对应类属性
            collection属性对应实体集合属性（存储从表多个条目）
                ofType属性存储从表条目对象
                select属性通过mybatis进行查询操作,将查询到的结果存储到Animal实体中
    -->
    <select id="selectAnimalByCategory" resultType="Animal">
        SELECT *
        FROM mybatis.animal WHERE cid = #{?}
    </select>

    <resultMap id="CategoryMapper" type="Category">
        <id column="cid" property="cid"/>
        <result column="name" property="cname"/>

        <collection property="animals" ofType="Animal"
                    select="selectAnimalByCategory"
                    column="cid"
        />

    </resultMap>

    <!--
        查询主表信息:
            resultMap指定映射标签
    -->
    <select id="selectCategoryById" resultMap="CategoryMapper">
        SELECT *
        FROM mybatis.category WHERE cid=#{?}
    </select>
</mapper>
```
![](/assets/images/Mybatis/one2more.png)

## 接口类以及测试
``` java
public interface CategoryDao {

    //    根据id获取分类信息
    Category selectCategoryById(int id);
}

public class Demo2 {
    private SqlSession sqlSession = MybatisUtils.getSqlSession();
    private CategoryDao categoryDao = sqlSession.getMapper(CategoryDao.class);

    //    根据id进行联表查询
    @Test
    public void demo1(){
        Category category = categoryDao.selectCategoryById(1);
        System.out.println(category);
    }
}
```

# 多对一关联查询

## 配置文件
``` xml
    <!--查询从表信息-->
    <select id="selectAnimalById" resultMap="AnimalMapper">
        SELECT
            cid,
            aname,
            aid
        FROM mybatis.animal
        WHERE aid = #{?}
    </select>

    <resultMap id="AnimalMapper" type="Animal">
        <id column="aid" property="aid"/>
        <result column="aname" property="aname"/>
        <association property="category" column="cid"
                     select="selectCategoryByAnimal"
                     javaType="Category"/>
    </resultMap>

    <select id="selectCategoryByAnimal" resultType="Category">
        SELECT
            cid,
            name cname
        FROM mybatis.category
        WHERE cid = #{?};
    </select>
```

## 接口以及测试类
``` java
    //    根据id获取animal信息
    Animal selectAnimalById(int id);
	
	    //    根据id查询Category
    @Test
    public void demo2(){
        Animal animal = categoryDao.selectAnimalById(1);
        System.out.println(animal);
    }
```

# 多对多关联查询
通过学生具有多门课程，课程具有多名学生作为案例为演示

## 新建实体类
``` java 
//		学生实体类
public class Course {
    private int cid;
    private String cname;
    private Set<Student3> students;

    public Set<Student3> getStudents() {
        return students;
    }

    public void setStudents(Set<Student3> students) {
        this.students = students;
    }

    public int getCid() {
        return cid;
    }

    public void setCid(int cid) {
        this.cid = cid;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }

    @Override
    public String toString() {
        return "Course{" +
                "cid=" + cid +
                ", cname='" + cname + '\'' +
                '}';
    }
}

//		课程实体类
public class Student3 {
    private String sname;
    private int sid;
    private Set<Course> courses;

    public Set<Course> getCourses() {
        return courses;
    }

    public void setCourses(Set<Course> courses) {
        this.courses = courses;
    }

    public String getSname() {
        return sname;
    }

    public void setSname(String sname) {
        this.sname = sname;
    }

    public int getSid() {
        return sid;
    }

    public void setSid(int sid) {
        this.sid = sid;
    }

    @Override
    public String toString() {
        return "Student3{" +
                "sname='" + sname + '\'' +
                ", sid=" + sid +
                ", courses=" + courses +
                '}';
    }
}
```

## 配置xml文件
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.Pu1satilla.dao.Many2manyDao">
    <resultMap id="selectStudentMapper" type="Student3">
        <id column="sid" property="sid"/>
        <result column="sname" property="sname"/>

        <collection property="courses" ofType="Course">
            <id column="cid" property="cid"/>
            <result column="cname" property="cname"/>
        </collection>
    </resultMap>

    <!--通过student的id查询所有当前学生所属课程-->
    <select id="selectStudentById" resultMap="selectStudentMapper">
        SELECT *
        FROM mybatis.middle, mybatis.student, mybatis.course
        WHERE middle.studentId = student.sid AND course.cid = courseId AND student.sid = #{?}
    </select>

    <!--通过course的id查询课程以及所有课程所属学生-->
    <select id="selectCourseById" resultMap="selectCourseMapper">
        SELECT *
        FROM mybatis.middle, mybatis.student, mybatis.course
        WHERE middle.courseId = course.cid AND student.sid = studentId AND course.cid = #{?}
    </select>

    <resultMap id="selectCourseMapper" type="Course">
        <id column="cid" property="cid"/>
        <result column="cname" property="cname"/>

        <collection property="students" ofType="Student3">
            <id column="sid" property="sid"/>
            <result column="sid" property="sname"/>
        </collection>
    </resultMap>
</mapper>
```

## 定义Dao接口以及测试类
``` java
//		接口
public interface Many2manyDao {

    //    多对多查询，通过Student的id查询学生信息，Student包含课程信息
    Student3 selectStudentById(int sid);

    //    多对多查询，通过课程的id查询课程信息，Course包含课程信息
    Course selectCourseById(int cid);
}

//		测试类
public class Demo3 {
    private SqlSession sqlSession = MybatisUtils.getSqlSession();
    private Many2manyDao manyDao = sqlSession.getMapper(Many2manyDao.class);

    @Test
    //    通过id查询所有student信息，包含所属课程
    public void demo0() {
        Student3 student = manyDao.selectStudentById(1);
        System.out.println(student);
    }

    @Test
    //    通过id查询所有课程信息，包含所属学生
    public void demo1() {
        Course course = manyDao.selectCourseById(1);
        System.out.println(course);
    }
```

# 延迟加载

MyBatis根据对关联对象查询的select语句的执行时机，分为三种类型：直接加载、侵入式延迟加载与深度延迟加载。

加载策略 | lazyLoadingEnabled | aggressiveLazyLoading
---- | ---- | ----
直接加载 | false | false
深度延时加载 | true | false
侵入式延迟加载 | true | true

## 直接加载
直接加载：执行完对主加载对象的select语句，马上执行对关联对象的select查询。

**修改主配置文件**
在主配置文件的<properties/>与<typeAliases/>标签之间，添加<settings/>标签，用于完成全局参数设置。
``` xml
    <!--全局参数设置-->
    <settings>
        <!--设置延迟加载-->
        <setting name="lazyLoadingEnabled" value="false"/>
    </settings>
```
**全局属性`lazyLoadingEnabled`的值只要设置为`false`，那么，对于关联对象的查询，将采用直接加载。**即在查询过主加载对象后，会马上查询关联对象。

## 侵入式延迟
执行对主加载对象的查询时，不会执行对关联对象的查询。但当要访问主加载对象的详情时，就会马上执行关联对象的select查询。即对关联对象的查询执行，侵入到了主加载对象的详情访问中。也可以这样理解：将关联对象的详情侵入到了主加载对象的详情中，即将关联对象的详情作为主加载对象的详情的一部分出现了。

**修改主配置文件**
``` xml
    <settings>
        <!--配置log4j日志-->
        <setting name="logImpl" value="LOG4J"/>
        <!--延迟加载总开关-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!--侵入式延迟加载开关-->
        <setting name="aggressiveLazyLoading" value="true"/>
    </settings>
```

## 深度延迟
执行对主加载对象的查询时，不会执行对关联对象的查询。访问主加载对象的详情时也不会执行关联对象的select查询。只有当真正访问关联对象的详情时，才会执行对关联对象的select查询。

**修改主配置文件**
修改主配置文件的`<settings/>`，将延迟加载开关`lazyLoadingEnabled`开启（置为true），将侵入式延迟加载开关`aggressiveLazyLoading`关闭（置为false）。
``` xml
    <settings>
        <!--配置log4j日志-->
        <setting name="logImpl" value="LOG4J"/>
        <!--延迟加载总开关-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!--侵入式延迟加载开关-->
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
```







































































































