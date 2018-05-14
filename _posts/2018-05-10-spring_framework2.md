---
title: Spring(二)
description:  Spring配置详解、Spring属性注入
categories:
 - Web
 - Java
 - Spring
tags: [Web, Java, Spring]
---

# Spring配置详解

在applicationContext.xml文件下配置

## Bean元素
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
	<!--
    将JavaBean对象交给spring容器管理
    Bean元素：使用该元素描述需要spring容器管理的对象
        class属性：被管理对象的完整类名
        name属性：给被管理的对象起个名字，获得对象时根据该名称获得对象，
            名称可以重复，可以使用特殊字符
        id属性：与name属性一模一样，
            名称不可重复，不能使用特殊字符
        尽量使用name属性
    -->
    <bean name="user" class="cn.Pu1satilla.domain.User"/>
</beans>
```

## Bean元素进阶

### scope属性

#### singleton（重点|默认值）
单例对象，被标识为单例的对象在spring容器中只会存在一个实例

测试用例
``` java
@Test
public void demo4() {

	//        1.创建spring容器对象
	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	//        2.获取容器中实例对象user3
	User user0 = (User) context.getBean("user3");
	User user1 = (User) context.getBean("user3");
	System.out.println(user0 == user1);//结果为true
}
```

#### prototype（重点）
多例原型，被标识为多例的对象，每次再获得才会创建，每次创建都是新的对象 

测试用例
``` java
@Test
public void demo5() {

	//        1.创建spring容器对象
	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	//        2.获取容器中实例对象user3
	User user0 = (User) context.getBean("user3");
	User user1 = (User) context.getBean("user3");
	System.out.println(user0 == user1);//结果为false
}
```

#### request（了解）
在web环境下，对象与reqeust生命周期一致

#### session（了解）
在web环境下，对象与session生命周期一致

### 生命周期属性（了解）
配置一个方法作为生命周期初始化方法 ，spring会在对象创建之后立即调用（init-method）

配置一个方法作为生命周期的销毁方法，spring容器在关闭并销毁所有容器中的对象之前调用（destory-method）

## Spring创建元素的方式

### 构造函数方式（重要）

Bean类
``` java
package cn.Pu1satilla.domain;

public class User {
    private String name;
    private Integer age;

    public User() {
        System.out.println("空参构造函数方式创建spring容器对象");
    }

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
        System.out.println("有参构造函数方式创建spring容器对象User{name="+name+";age="+age+"}");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

测试类
``` java
@Test
public void demo1(){

	//        创建spring容器对象
	new ClassPathXmlApplicationContext("applicationContext.xml");
}
```

#### 无参构造函数
配置xml文件
``` xml
<!--1.默认无参数构造函数-->
<bean name="user0" class="cn.Pu1satilla.domain.User"/>
```

测试结果
```
空参构造函数方式创建spring容器对象 
```

#### 有参构造函数
配置xml文件
``` xml
<!--2.带参数构造器-->
<bean name="user1" class="cn.Pu1satilla.domain.User">
	<constructor-arg index="0" type="java.lang.String" value="小明"/>
	<constructor-arg index="1" type="java.lang.Integer" value="10"/>
</bean>
```

测试结果

```
有参构造函数方式创建spring容器对象User{name=小明;age=10} 
```

### 静态工厂（了解）

#### 书写静态工厂
``` java
package cn.Pu1satilla.domain;

public class UserFactory {

    public static User creatUser(){

        System.out.println("静态工厂创建User");
        return new User();
    }
}

```

#### 书写xml配置
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--
    将JavaBean对象交给spring容器管理
    Bean元素：使用该元素描述需要spring容器管理的对象
        class属性：被管理对象的完整类名
        name属性：给被管理的对象起个名字，获得对象时根据该名称获得对象，
            名称不可以重复，可以使用特殊字符
        id属性：与name属性一模一样，
            名称不可重复，不能使用特殊字符
        尽量使用name属性
    -->

    <!--1.默认无参数构造函数-->
    <!--<bean name="user0" class="cn.Pu1satilla.domain.User"/>-->

    <!--2.带参数构造器-->
    <!--<bean name="user1" class="cn.Pu1satilla.domain.User">-->
        <!--<constructor-arg index="0" type="java.lang.String" value="小明"/>-->
        <!--<constructor-arg index="1" type="java.lang.Integer" value="10"/>-->
    <!--</bean>-->
    <!--3.静态工厂创建
        调用UserFactory的creatUser方法创建名为user2的对象，放入spring容器
    -->
    <bean name="user2" class="cn.Pu1satilla.domain.UserFactory"
          factory-method="creatUser"/>
</beans>
```

#### 测试结果
```
静态工厂创建User
空参构造函数方式创建spring容器对象 
```

### 实例工厂（了解）

#### 书写实例工厂
``` java
package cn.Pu1satilla.domain;

public class UserFactory {

    public static User creatUser(){

        System.out.println("静态工厂创建User");

        return new User();
    }

    public User createUser2(){

        System.out.println("实例工厂创建User");
        return new User();
    }
}
```

#### 书写xml配置文件
``` xml
<!--4.实例工厂创建
	调用UserFactory的createUser2创建名为user3的对象，放入spring容器
-->
<bean name="user3"
	  factory-bean="userFactory"
	  factory-method="createUser2"/>
<bean name="userFactory" class="cn.Pu1satilla.domain.UserFactory"/>
```

#### 测试结果
```
实例工厂创建User
空参构造函数方式创建spring容器对象
```

## Spring的分模块配置
``` xml
<!--导入其他spring配置文件
	resource为其他spring配置文件路径
-->
<import resource="applicationContext.xml"/>
```

# spring属性注入

## set方法注入（重点）

### 值类型注入
xml配置
``` xml
<!--为user对象注入属性值-->
<bean name="user0" class="cn.Pu1satilla.domain.User">
	<property name="name" value="lucy"/>
	<property name="age" value="18"/>
</bean>
```

测试用例
``` java
@Test
public void demo6(){

	//        1.创建spring容器对象
	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	//        2.获取容器对象
	User user = (User) context.getBean("user0");

	//        3.打印
	System.out.println(user);
}
```

测试结果
```
User{name='lucy', age=18} 
```

### 引用类型注入
xml配置
``` xml
<!--为user对象注入引用类型-->
<bean name="user" class="cn.Pu1satilla.domain.User">
	<property name="age" value="10"/>
	<property name="name" value="Marry"/>
	<property name="cat" ref="cat"/>
</bean>

<!--为cat对象注入属性值-->
<bean name="cat" class="cn.Pu1satilla.domain.Cat">
	<property name="name" value="Tom"/>
	<property name="age" value="3"/>
</bean>
```

更新Bean类
``` java
package cn.Pu1satilla.domain;

public class User {
    private String name;
    private Integer age;
    private Cat cat;

    public User() {
    }

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", cat=" + cat +
                '}';
    }
}
```

测试用例
``` java
/**
 * 演示引用类型注入方式
 */
@Test
public void demo7(){
	//        1.创建spring容器对象
	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	//        2.获取容器对象
	User user = (User) context.getBean("user");

	//        3.打印
	System.out.println(user);
}
```

演示结果
``` java
User{name='Marry', age=10, cat=Cat{name='Tom', age=3}}
```
成功注入！

## 构造函数注入（掌握）
xml配置
``` xml
<!--为cat对象注入属性值-->
<bean name="cat" class="cn.Pu1satilla.domain.Cat">
	<property name="name" value="Tom"/>
	<property name="age" value="3"/>
</bean>

<!--构造函数注入-->
<bean name="user" class="cn.Pu1satilla.domain.User">
	<!--参数
		name属性：构造函数的参数名
		index属性：构造函数的参数索引
		type属性：构造函数的参数类型
		value：构造函数参数值
	-->
	<constructor-arg name="age" value="10" type="java.lang.Integer" index="1"/>
	<constructor-arg name="name" value="笑潘慧" type="java.lang.String" />
	<constructor-arg name="cat" ref="cat" type="cn.Pu1satilla.domain.Cat" />
</bean>
```
Bean类
``` java
package cn.Pu1satilla.domain;

public class User {
    private String name;
    private Integer age;
    private Cat cat;

    public User() {
    }

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public User(String name, Integer age, Cat cat) {
        this.name = name;
        this.age = age;
        this.cat = cat;
    }

    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", cat=" + cat +
                '}';
    }
}
```
测试用例
``` java
/**
 * 演示引用类型注入方式
 */
@Test
public void demo7(){
	//        1.创建spring容器对象
	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	//        2.获取容器对象
	User user = (User) context.getBean("user");

	//        3.打印
	System.out.println(user);
}
```
测试结果
```
User{name='笑潘慧', age=10, cat=Cat{name='Tom', age=3}}
```

## p名称空间注入（了解）

``` xml
<!--为cat对象注入属性值-->
<bean name="cat" class="cn.Pu1satilla.domain.Cat">
	<property name="name" value="Tom"/>
	<property name="age" value="3"/>
</bean>
<!--p名称空间注入(set方法)
1.导入p名称空间 xmlns:p="http://www.springframework.org/schema/p"
2.使用p：属性完成注入
	 |-值类型：p：属性名="值"
	 |-对象类型：p：属性名-ref="bean名称"
-->
<bean name="user" class="cn.Pu1satilla.domain.User" p:name="张怀义" p:age="108" p:cat-ref="cat"/>
```

## spel注入（了解）

spel注入：Spring Expression Language spring表达式语言
语法 #{ SpEL }
``` xml
<bean name="cat" class="cn.Pu1satilla.domain.Cat">
	<property name="name" value="Tom"/>
	<property name="age" value="3"/>
</bean>

<!--spel注入：
	spring Expression Language spring表达式语言
-->
<bean name="user" class="cn.Pu1satilla.domain.User">
	<property name="name" value="#{cat.name}"/>
	<property name="age" value="#{10}"/>
	<property name="cat" value="#{cat}"/> 
</bean>
```

## 复杂类型注入
包含数组、集合、Map以及Properties注入

xml配置文件
``` xml
<!--复杂类型注入-->
<bean name="collection" class="cn.Pu1satilla.domain.Collection">
	<!--
	数组单个值注入
	如果数组值准备注入一个值（对象），直接使用value|ref即可
	-->
	<!--<property name="arr" value="onlyOne"/>-->

	<!--数组多个值注入-->
	<!--<property name="arr">-->
	<!--<array>-->
	<!--<value>aaa</value>-->
	<!--<value>bbb</value>-->
	<!--<value>ccc</value>-->
	<!--<ref bean="cat"/>-->
	<!--</array>-->
	<!--</property>-->

	<!--List单个值注入
		如果数组值准备注入一个值（对象），直接使用value|ref即可
	-->
	<!--<property name="list" value="onlyOne"/>-->

	<!--List多个值注入-->
	<!--<property name="list">-->
	<!--<list>-->
	<!--<value>aaa</value>-->
	<!--<value>bbb</value>-->
	<!--<value>ccc</value>-->
	<!--<ref bean="cat"/>-->
	<!--</list>-->
	<!--</property>-->

	<!--map类型注入-->
	<!--<property name="map">-->
		<!--<map>-->
			<!--<entry key="aaa" value="a-a-a"/>-->
			<!--<entry key="bbb" value="b-b-b"/>-->
			<!--<entry key-ref="cat" value-ref="user"/>-->
		<!--</map>-->
	<!--</property>-->

	<!--properties类型注入-->
	<property name="prop">
		<props>
			<prop key="aaa">a-a-a</prop>
			<prop key="bbb">b-b-b</prop>
		</props>
	</property>
</bean>
```

测试用例
``` java
/**
 * 演示复杂类型注入
 * 在数组中注入单个对象
 */
@Test
public void demo8(){

	//        1.创建spring容器对象
	ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");

	//        2.获取容器内对象
	Collection collection = (Collection) applicationContext.getBean("collection");

	//        3.打印容器对象数组属性
	//        System.out.println(Arrays.toString(collection.getArr()));

	//        4.打印容器中对象集合属性
	//        System.out.println(collection.getList());

	//        5.打印容器中对象Map属性
	//        System.out.println(collection.getMap());

	//        6.打印容器中对象Prop属性
	System.out.println(collection.getProp());
}
```
































