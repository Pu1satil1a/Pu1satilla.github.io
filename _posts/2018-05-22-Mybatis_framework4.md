---
title: Mybatis(四)
description: Mybatis关联关系查询
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



















































































































