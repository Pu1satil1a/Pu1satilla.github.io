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

# 动态SQL
动态SQL主要用于解决查询条件不确定的情况：在程序运行期间没根据用户提交的查询条件进行查询，提交的查询条件不同，执行的SQL语句不同。若将每种可能的情况均逐一列出，对所有条件进行排列组合，将会出现大量SQL语句。此时，可使用动态SQL来解决这样的问题。

动态SQL，即通过Mybatis提供的各种标签对条件作出判断以实现动态拼接SQl语句。

## if标签
if标签test属性用于判断传入值是否符合，符合则执行标签内内容，不符合则跳过。
``` xml
    <!--动态SQL if标签-->
    <select id="selectStudentByCondition" resultType="Student">
        SELECT * FROM mybatis.student WHERE
        <if test=" name != null and name !=''">
            name LIKE '%' #{name} '%'
        </if>
        <if test="age > 0">
            AND age &gt; #{age}
        </if>
    </select>
```
这样又可能会导致首个条件不符合，直接判断第二个条件，sql语句报错：
``` sql
SELECT * FROM mybatis.student WHERE AND age &gt; #{age}
```

可以通过添加条件`1=1`
``` sql
    <!--动态SQL if标签-->
    <select id="selectStudentByCondition" resultType="Student">
        SELECT * FROM mybatis.student WHERE 1=1
        <if test=" name != null and name !=''">
            AND name LIKE '%' #{name} '%'
        </if>
        <if test="age > 0">
            AND age &gt; #{age}
        </if>
    </select>
```
这样又会导致执行没必要sql语句，浪费资源，这时where标签就派上用场了

## where标签

MyBatis 有一个简单的处理，这在 90% 的情况下都会有用。而在不能使用的地方，你可以自定义处理方式来令其正常工作。一处简单的修改就能达到目的：
``` xml
    <!--动态SQL where标签-->
    <select id="selectStudentByCondition" resultType="Student">
        SELECT * FROM mybatis.student
        <where>
            <if test=" name != null and name !=''">
                AND name LIKE '%' #{name} '%'
            </if>
            <if test="age > 0">
                AND age &gt; #{age}
            </if>
        </where>
    </select>
```
where 元素只会在至少有一个子元素的条件返回 SQL 子句的情况下才去插入`“WHERE”`子句。而且，若语句的开头为`“AND”`或`“OR”`，where 元素也会将它们去除。 

## choose标签
有时我们不想应用到所有的条件语句，而只想从中择其一项。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。
``` xml
    <!--动态SQL查询 Choose标签-->
    <select id="selectStudentByChoose" resultType="Student">
        SELECT * FROM mybatis.student
        <where>
            <choose>
                <when test="arg0 != null and arg0 != ''">
                    AND name like '%' #{arg0} '%'
                </when>
                <when test="arg1 > 0">
                    AND age > #{arg1}
                </when>
                <otherwise>
                    1=2
                </otherwise>
            </choose>
        </where>
    </select>
```

接口以及测试类
``` java
    //    动态SQL查询，choose标签
    List<Student> selectStudentByChoose(String name, int age);
	
	//	   测试方法
	    @Test
    public void demo9() {

        List<Student> students = studentDao.selectStudentByChoose("", 0);
        System.out.println(students);
    }
```

## foreach标签

动态 SQL 的另外一个常用的操作需求是对一个集合进行遍历，通常是在构建 IN 条件语句的时候。

foreach 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及在迭代结果之间放置分隔符。这个元素是很智能的，因此它不会偶然地附加多余的分隔符。

注意：** 你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象传递给 foreach 作为集合参数。当使用可迭代对象或者数组时，index 是当前迭代的次数，item 的值是本次迭代获取的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。**

### 参数为数组
配置文件
``` xml
    <!--动态SQL查询 foreach标签 数组作为参数-->
    <select id="selectStudentByForeachArray" resultType="Student">
        SELECT * FROM mybatis.student WHERE id IN

        <!--
            遍历数组,类似于Java中for循环,遍历一个数组并取出相应对象用于组成SQL语句
            SELECT * FROM mybatis.student WHERE id IN ( ? , ? , ? , ? )
                collection：指定array表明传入参数为数组
                      item：每次遍历获取的对象
                     index：当前遍历索引
        -->
            <foreach collection="array" item="item" index="index" open="(" separator="," close=")">
                #{item}
            </foreach>
    </select>
```
接口及测试类
``` java
	接口
    //    动态SQL查询，foreach标签，数组作为参数
    List<Student> selectStudentByForeachArray(int[] id);

	//    测试类
	@Test
    public void demo10() {

        int[] id = {24, 25, 26, 27};

        List<Student> students = studentDao.selectStudentByForeachArray(id);
        System.out.println(students);
    }
```

### 参数为集合（基本类型）

配置文件
``` xml
    <!--动态SQL查询 foreach标签 集合作为参数-->
    <select id="selectStudentByForeachList" resultType="Student">
        SELECT * FROM mybatis.student WHERE id IN

        <!--
            遍历集合,类似于Java中for循环,遍历一个数组并取出相应对象用于组成SQL语句
            SELECT * FROM mybatis.student WHERE id IN ( ? , ? , ? , ? )
                collection：指定list表明传入参数为集合
                      item：每次遍历获取的对象
                     index：当前遍历索引
        -->
            <foreach collection="list" item="item" index="index" open="(" separator="," close=")">
                #{item}
            </foreach>
    </select>
```
接口及测试类
``` java
    //    动态SQL查询，foreach标签，集合作为参数
    List<Student> selectStudentByForeachList(List<Integer> id);

	//	   接口
    @Test
    public void demo11() {

        List<Integer> id = new ArrayList<>();

        id.add(35);
        id.add(37);
        id.add(38);

        List<Student> students = studentDao.selectStudentByForeachList(id);
        System.out.println(students);
    }
```

### 参数为集合（引用类型）

配置文件
``` xml
    <!--动态SQL查询 foreach标签 集合作为参数（引用类型）-->
    <select id="selectStudentByForeachListStudent" resultType="Student">
        SELECT * FROM mybatis.student WHERE id IN

        <!--
            遍历集合,类似于Java中for循环,遍历一个数组并取出相应对象用于组成SQL语句
            SELECT * FROM mybatis.student WHERE id IN ( ? , ? , ? , ? )
                collection：指定list表明传入参数为集合
                      item：每次遍历获取的对象
                     index：当前遍历索引
        -->
        <foreach collection="list" item="item" index="index" open="(" separator="," close=")">
            #{item.id}
        </foreach>
    </select>
```
接口及测试类
``` java
    //    动态SQL查询，foreach标签，遍历引用类型的集合
    List<Student> selectStudentByForeachListStudent(List<Student> students);

	//	   测试类
	@Test
    public void demo12() {

        List<Student> id = new ArrayList<>();

        id.add(new Student(33, "松江", 55, 50));
        id.add(new Student(35, "孔明", 56, 77));

        List<Student> students = studentDao.selectStudentByForeachListStudent(id);
        System.out.println(students);
    }
```

## sql标签
将一部分sql语句存放到另外一个标签，在其它标签可以引用

``` xml
   <sql id="select">
        SELECT * FROM mybatis
    </sql>

    <!--动态SQL查询 通过引入其他sql语句-->
    <select id="selectStudentBySql" resultType="Student">
        <include refid="select"/>.student
    </select>
```
















































































