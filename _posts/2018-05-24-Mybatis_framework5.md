---
title: Mybatis(五)
description: 查询缓存、MyBatis注解式开发与总结
categories:
 - Web
 - Java
 - Mybatis
tags: [Web, Java, Mybatis]
---


# 查询缓存
查询缓存的使用，主要是为了提高查询访问速度。将用户对同一数据的重复查询过程简化，不再每次均从数据库查询获取结果数据，从而提高访问速度。MyBatis的查询缓存机制，根据缓存区的作用域（生命周期）可划分为两种：一级查询缓存与二级查询缓存。

## 查询缓存机制

## 一级缓存
![](/assets/images/Mybatis/1levelCache.png)

### 读取数据的依据与实现
依据：Mybatis的查询依据是Sql的id + Sql语句，而hibernate查询的是结果对象的id

实现：缓存的底层实现是一个Map，Map的value是查询结果

### 缓存的刷新
增删改操作都会清空一级缓存

### 作用域以及生命周期
作用域是根据映射文件`mapper`的`namespace`划分的，相同的命名空间的`mapper`查询数据存放在同一个缓存区域。不同namespace下的数据互不干扰。无论是一级缓存还是二级缓存，都是按照namespace分别存放的。

生命周期是SqlSession的生命周期。

## 内置二级缓存
使用二级缓存的目的不是共享数据，因为Mybatis从缓存中读取数据的依据是SQL的id，而非查询出的对象。所以，**二级缓存中的数据不是为了在多个查询之间查询**，而是为了**延长查询结果的保存时间**，提高系统性能。

`myBatis`内置的二级缓存为`org.apache.ibatis.cache.impl.PerpetualCache`。

### 二级缓存的用法
1. 查询结果设计的实体类要实现`java.io.Serializable`接口，若该实体类存在父类，或其具有域属性，则父类与域属性也要实现序列化接口。
2. mapper映射中添加<cache/>标签

### 二级缓存的属性

1. eviction：逐出策略。当二级缓存中的对象达到最大值时，就需要通过逐出策略将缓存中的对象移出缓存。默认为LRU。常用的策略有：
- FIFO：First In First Out，先进先出
- LRU：Least Recently Used，未被使用时间最长的

2. flushInterval：刷新缓存的时间间隔，单位毫秒。这里的刷新缓存即清空缓存。一般不指定，即当执行增删改时刷新缓存。

3. readOnly：设置缓存中数据是否只读。只读的缓存会给所有调用者返回缓存对象的相同实例，因此这些对象不能被修改，这提供了很重要的性能优势。但读写的缓存会返回缓存对象的拷贝。这会慢一些，但是安全，因此默认是false。

4. size：二级缓存中可以存放的最多对象个数。默认为1024个。

### 二级缓存的刷新
增删改操作也会清空二级缓存，但是对所查找key对应的value置为null，而非将<key，value>对删除，从数据库进行select查询的条件是
1. 缓存中根本不存在这个key
2. 缓存中存在该key所对应的Entry对象，但其value为null

### 二级缓存的关闭
全局关闭：开关设置在主配置文件的全局设置<settings/>中，该属性为cacheEnabled，设置为false，则关闭；设置为true，则开启，默认值为true。即二级缓存默认是开启的。

局部关闭：所谓局部关闭是指，整个应用的二级缓存是开启的，但只是针对于某个<select/>查询，不使用二级缓存。此时可以单独只关闭该<select/>标签的二级缓存。在该要关闭二级缓存的<select/>标签中，将其属性useCache设置为false，即可关闭该查询的二级缓存。该属性默认为true，即每个<select/>查询的二级缓存默认是开启的。

### 二级缓存的使用原则
1. 只能在一个命名空间下使用二级缓存
```
由于二级缓存中的数据是基于namespace的，即不同namespace中的数据互不干扰。
在多个namespace中若均存在对同一个表的操作，那么这多个namespace中的数据可能就会出现不一致现象。
```

2. 在单表上使用二级缓存
```
如果一个表与其它表有关联关系，那么就非常有可能存在多个namespace对同一数据的操作。
而不同namespace中的数据互不干扰，所以有可能出现这多个namespace中的数据不一致现象。
```

3. 查询远多于修改时使用二级缓存
```
在查询操作远远多于增删改操作的情况下可以使用二级缓存。
因为任何增删改操作都将刷新二级缓存，对二级缓存的频繁刷新将降低系统性能。
```

## ehcache二级查询缓存

### 导入jar包
这里需要两个Jar包：一个为ehcache的核心Jar包位于压缩文件lib目录下，一个是myBatis与ehcache整合的插件Jar包。  
[下载](https://github.com/mybatis/ehcache-cache/releases)

### 添加ehcache.xml
解压`EHCache`的核心Jar包`ehcache-core-2.6.8.jar`，将其中的一个配置文件`ehcache-failsafe.xml`直接放到项目的src目录下，并更名为`ehcache.xml`。

### IDEA添加约束
![](/assets/images/Mybatis/ehcache.png)

### 配置ehcache.xml标签
- <diskStore/>标签
指定一个文件目录，当内存空间不够，需要将二级缓存中数据写到硬盘上时，会写到这个指定目录中。其值一般为`java.io.tmpdir`，表示当前系统的默认文件临时目录。当前系统的默认文件临时目录，可以通过System.property()方法查看：
``` java
@Test
public void test(){
	String path = System.getProperty("java.io.tmpdir");
	System.out.println(path);
}
```

**`<defaultCache/>`标签**

设定缓存的默认属性数据：

- maxElementsInMemory：指定该内存缓存区可以存放缓存对象的最多个数
- eternal：设定缓存对象是否不会过期。若设为true，表示对象永远不会过期，此时会忽略timeToIdleSeconds与timeToLiveSeconds属性。默认值为false
- timeToIdleSeconds：设定允许对象处于空闲状态的最长时间，以秒为单位。当对象自从最近一次被访问后，若处于空闲状态的时间超过了timeToIdleSeconds设定的值，这个对象就会过期。当对象过期，EHCache就会将它从缓存中清除。设置值为0，则对象可以无限期地处于空闲状态
- timeToLiveSeconds：设定对象允许存在于缓存中的最长时间，以秒为单位。当对象自从被存放到缓存后，若处于缓存中的时间超过了timeToLiveSeconds设定的值，这个对象就会过期。当对象过期，EHCache就会将它从缓存中清除。设置值为0，则对象可以无限期地存在于缓存中。注意，只有timeToLiveSeconds≥ timeToIdleSeconds，才有意义
- overflowToDisk：设定为true，表示当缓存对象达到了maxElementsInMemory界限，会将溢出的对象写到`<diskStore>`元素指定的硬盘目录缓存中
- maxElementsOnDisk：指定硬盘缓存区可以存放缓存对象的最多个数
- diskPersistent：指定当程序结束时，硬盘缓存区中的缓存对象是否做持久化
- diskExpiryThreadIntervalSeconds：指定硬盘中缓存对象的失效时间间隔
- memoryStoreEvictionPolicy：如果内存缓存区超过限制，选择移向硬盘缓存区中的对象时使用的策略。支持三种策略：
1. FIFO：First In First Out，先进先出
2. LFU：Less Frequently Used，最少使用
3. LRU：Least Recently Used，最近最少使用

### 启用ehcache缓存机制
在映射文件的mapper中的`<cache/>`中通过type指定缓存机制为Ehcache缓存。默认为myBatis内置的二级缓存`org.apache.ibatis.cache.impl.PerpetualCache`。

``` xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

### 个性化设置
在ehcache.xml中设置的属性值，会对该项目中所有使用ehcache缓存机制的缓存区域起作用。一个项目中可以有多个mapper，不同的mapper有不同的缓存区域。对于不同缓存区域也可进行专门针对于当前区域的个性化设置，可通过指定不同mapper的`<cache>`属性值来设置。`<cache>`属性值的优先级高于ehcache.xml中的属性值。

``` xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache">
	<property name="maxElementInMemory" value="5000"/>
	<property name="timeToIdleSeconds" value="240"/>
</cache>
```

# MyBatis注解式开发
mybatis的注解，主要是用于替换映射文件。而映射文件中无非存放着增、删、改、查的SQL映射标签。所以，mybatis注解，就是要替换映射文件中的SQL标签。mybatis官方文档中指出，若要真正想发挥mybatis功能，还是要用映射文件。**即mybatis官方并不建议通过注解方式来使用mybatis**。

## 注解的基础知识
以下注解知识的讲解，均使用使用`@Overide`、`@Deprecated（过时）`、`@SuppressWarnings`举例。

## 注解的基础语法
A、注解后是没有分号的。  
B、注解首字母是大写的，因为注解与类、接口是同一级别的。一个注解，后台对应着一个`@interface`类  
C、在同一语法单元上，同一注解只能使用一次。  
D、在注解与语法单元之间可以隔若干空行、注释等非代码内容。  

## 修改主配置文件
``` xml
    <!--注册映射文件-->
    <mappers>
        <!--<mapper resource="cn/Pu1satilla/domain/mapper.xml"/>-->
        <!--分类映射文件-->
        <!--<mapper resource="cn/Pu1satilla/domain/category.xml"/>-->
        <!--<mapper resource="cn/Pu1satilla/dao/many2many.xml"/>-->
        <package name="cn.Pu1satilla.dao"/>
    </mappers>
```

## insert
``` java
    //    根据对象添加
    @Insert(value = {"INSERT INTO student_annotation (sname,sage,score) VALUES(#{sname},#{sage},#{score})"})
    void insertStudent(Student student);
```

将插入的对象返回插入id值并且存入当前对象
``` java
    //    根据对象添加
    @Insert(value = {"INSERT INTO student_annotation (sname,sage,score) VALUES(#{sname},#{sage},#{score})"})
    @SelectKey(statement = "select @@identity", keyProperty = "sID", before = false, resultType = int.class)
    void insertStudent(Student student);
```

## delete
``` java
    //    根据id删除条目，参数只有一个不需要加大括号
    @Delete(value = "DELETE FROM student_annotation WHERE sId = #{id}")
    void deleteStudent(int id);
```

## update
``` java
    //    根据对象修改数据库条目
    @Update(value = "        UPDATE student_annotation" +
            "        SET sname = #{sname}, sage = #{sage}, score = #{score}" +
            "        WHERE sId = #{sID}")
    void updateStudent(Student student);
```

## select
**通过id查询对象**
``` java
    //    根据id查询
    @Select(value = "SELECT * FROM student_annotation WHERE sID=#{?}")
    Student selectStudent(int id);
```

**查询所有数据库中条目**
``` java
    //    查询所有数据库中条目
    @Select(value = "SELECT * FROM student_annotation")
    List<Student> selectAllStudent();
```

**模糊查询**
``` java
    //    模糊查询
    @Select(value = "SELECT * FROM student_annotation WHERE sname LIKE '%' #{?} '%'")
    List<Student> selectStudentByName(String name);
```

# 总结
**重点**
- 独立完成MyBatis的demo程序 
- 源码分析
- Mapper动态代理
- 动态SQL
- 关联查询
- 延迟加载
- 一级缓存























































































































































