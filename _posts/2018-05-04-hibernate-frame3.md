---
title: Hibernate(三)
description: hibernate实体创建规则，hibernate对象状态，hibernate一级缓存
categories:
 - Web
 - Java
tags: [Web, Java, Hibernate]
---


# hibernate中的实体规则

## 什么是持久化类
Hibernate是持久层的ORM映射框架，专注于数据的持久化工作，持久化是将内存中的数据永久存储到关系型数据库中。持久化类另外一种理解就是Java类有一个映射文件与数据库的表建立关系。

## 实体类创建事项

### 持久化类提供无参构造

持久化类需要提供无参数的构造方法，因为在Hibernate的底层需要使用反射生成类的实例。

反射获取构造方法并运行的快速方式：
1. 前提：被反射的类，必须具有空参数构造方法
2. 构造方法权限必须public
3. T newInstance()  直接创建被反射类对象实例

### 成员变量私有

**成员变量私有，并且提供get/set方法访问，需提供属性**

属性可以理解为get和set方法，属性只局限于类中方法的声明，拥有get以及set方法可以说改类拥有可读写的属性，去掉了set可以说是可读属性

### 包装类型

**持久化类中的属性，应尽量使用包装类型**  
好处：
1. 包装类型比基本数据类型在表达值时可以多表达一个null
2. 当数据库中的值为空，但是实体相应变量类型为基本数据类型，那么不能存储null，会报错

### 需提供oid
**持久化类需要提供oid，与数据库中的主键列对应**，没有主键的表无法映射到Hibernate

### 不能用final修饰class
Hibernate使用cglib代理生成对象，代理对象是继承被代理对象，如果被final修饰，将无法生成代理

## 主键类型

### 自然主键（少见）
表的业务列中，有某业务符合，不许有，并且不重复的特征时，该列可以作为主键使用

### 代理主键（常见）
表的业务列中，没有某业务符合，不许有，并且不重复的特征时，需要创建一个没有业务意义的列作为主键

``` xml
<id name="cid" column="cid">

	<!--generator：主键生成策略，就是每条记录录入时，主键的生成规则（7个），
		class>>>>规则（7个）：
		identity：主键自增，有数据库来维护主键值，录入时不需要指定主键
		increment（了解）：主键自增，由hibernate来维护，每次插入前会先查询表中id最大值+1作为新主键值（类似identity，不推荐，线程安全问题）
		sequence：Oracle中的主键生成策略
		hilo：（了解）高低位算法，由hibernate来维护
		native：hilo+sequence+identity 自动三选一策略（推荐）
		uuid：产生随机字符串作为主键（String）
		assigned：自然主键生成策略，hibernate不会管理主键，由开发人员自己录入
		-->

	<generator class="native"/>
</id>
```

# hibernate中对象状态


## 瞬时状态

瞬时状态的实例由new命令创建、开辟内存空间的对象，不存在持久化标志OID（相当于主键值），尚未与Hibernate Session关联，在数据库中也没有记录，失去引用后被JVM回收。瞬时状态的对象在内存中是孤立存在的，与数据库中的数据无任何关联，仅仅是一个信息携带的载体。

## 持久化状态

持久化对象其实有一个非常重要的特性：持久态对象可以自动更新数据库。持久态对象之所以有这样的功能其实都依赖Hibernate的以及缓存

持久化的对象存在持久化标志OID，加入到了Session缓存中，并且相关联的Session没有关闭，在数据库中有对应的记录，每条记录值对应唯一的持久化对象，持久化对象是在事务还未提交前变成持久态的。

## 游离|脱管状态

脱管状态称为离线态或者游离态，当某个持久化状态的实例与Session的关联被关闭时就变成了脱管态。脱管态对象存在持久化标志OID，并且仍然与数据库中的数据存在关联，只是失去了与当前Session的关联，脱管状态对象发生改变时，Hibernate不能检测到。

## 状态之间的转换
![](/assets/images/hibernate/status.png)

# Hibernate一级缓存

## 什么是一级缓存
Hibernate的一级缓存是指Session缓存，Session缓存是一块内存空间，用来存放互相管理的Java对象，在使用Hibernate查询对象的时候，首先会使用对象属性的OID值在Hibernate的以及缓存中进行查找，如果找到匹配OID值的对象，就直接将改对象从以及缓存中取出使用，不会再查询数据库；如果没有找到相同OID值的对象，则会去数据库中查找响应数据。当从数据库中查询到所需数据时，该数据信息也会放置到一级缓存中。

## 一级缓存的特点
- 当应用程序调用Session接口的save()、update()、saveOrUpdate()时，如果Session缓存中没有相应的对象，Hibernate就会自动把从数据库中查询到的相应对象信息加入到以及缓存中去。
- 当调用Session接口的load()、get()，以及Query接口的list()、iterator()方法时，会判断缓存中是否存在该对象，有则返回，不会查询数据库，如果缓存中没有要查询对象，再去数据库中查询相应对象，并添加到以及缓存中。
- 当调用Session的close()方法时，Session缓存就会被清空。

## 一级缓存的快照区
Hibernate的持久态对象能够自动更新数据库，其实就是依赖一级缓存，而一级缓存中有块特殊的区域**快照区**。

运行下面代码：
``` java
@Test
public void demo1(){
	Session session = HibernateUtils.getSession();
	Transaction transaction = session.beginTransaction();

	//        获得持久化对象
	Customer customer = (Customer) session.get(Customer.class,1);
	customer.setCust_name("树下");

	//        不通过update方法就可以更新
	transaction.commit();
	session.close();
}
```

运行结果：
![](/assets/images/hibernate/cache01.png)

在上面代码运行中，并没有调用update或者save方法，在数据库中的结果却完成了更新，这是为什么？

Hibernate向以及缓存放入数据时，同时复制一份数据放入到Hibernate快照中，当使用commit()方法提交事务时，同时会清理Session的以及缓存，**这是会使用OID判断以及缓存中的对象和快照中的对象是否一致，如果两个对象中的属性发生变化，则执行update语句，将缓存的内容同步到数据库，并更新快照；如果一致，则不执行update语句。**Hibernate快照的作用是检测到缓存区中属性变化的对象并且更新到数据库。

## 一级缓存的作用

### 提高查询效率
``` java
@Test
public void demo2() {
	Session session = HibernateUtils.getSession();
	Transaction transaction = session.beginTransaction();

	//        重复获得持久化状态对象
	Customer customer1 = (Customer) session.get(Customer.class, 1);
	Customer customer2 = (Customer) session.get(Customer.class, 1);
	System.out.println(customer1 == customer2);

	transaction.commit();
	session.close();
}
```
**结果：**  
```
Hibernate: 
    select
        customer0_.cid as cid0_0_,
        customer0_.cust_name as cust2_0_0_,
        customer0_.cust_level as cust3_0_0_,
        customer0_.cust_source as cust4_0_0_,
        customer0_.cust_linkman as cust5_0_0_,
        customer0_.cust_phone as cust6_0_0_,
        customer0_.cust_mobile as cust7_0_0_ 
    from
        customer.customer customer0_ 
    where
        customer0_.cid=?
true 
```
**过程：**  
![](/assets/images/hibernate/cache02.png)

### 减少修改语句发送

**试验一：不改变对象属性，查看控制台**  
``` java
@Test
public void demo3() {
	Session session = HibernateUtils.getSession();
	Transaction transaction = session.beginTransaction();

	//        获得持久化状态对象
	Customer customer = (Customer) session.get(Customer.class,1);

	//        不改变对象属性存储
	session.save(customer);
	transaction.commit();
	session.close();
}
```
**结果：只进行查询，没有将语句进行发送**  
```
Hibernate: 
    select
        customer0_.cid as cid0_0_,
        customer0_.cust_name as cust2_0_0_,
        customer0_.cust_level as cust3_0_0_,
        customer0_.cust_source as cust4_0_0_,
        customer0_.cust_linkman as cust5_0_0_,
        customer0_.cust_phone as cust6_0_0_,
        customer0_.cust_mobile as cust7_0_0_ 
    from
        customer.customer customer0_ 
    where
        customer0_.cid=?
```

**试验二：改变对象属性，但是不进行sava()操作**  
``` java
@Test
public void demo1() {
	Session session = HibernateUtils.getSession();
	Transaction transaction = session.beginTransaction();

	//        获得持久化状态对象
	Customer customer = (Customer) session.get(Customer.class, 1);
	customer.setCust_name("村上");

	//        不通过update方法就可以更新
	transaction.commit();
	session.close();
}
```
**结果：进行语句查询，并且发送更新sql语句**   
```
Hibernate: 
    select
        customer0_.cid as cid0_0_,
        customer0_.cust_name as cust2_0_0_,
        customer0_.cust_level as cust3_0_0_,
        customer0_.cust_source as cust4_0_0_,
        customer0_.cust_linkman as cust5_0_0_,
        customer0_.cust_phone as cust6_0_0_,
        customer0_.cust_mobile as cust7_0_0_ 
    from
        customer.customer customer0_ 
    where
        customer0_.cid=?
Hibernate: 
    update
        customer.customer 
    set
        cust_name=?,
        cust_level=?,
        cust_source=?,
        cust_linkman=?,
        cust_phone=?,
        cust_mobile=? 
    where
        cid=?
```
**过程：**  
![](/assets/images/hibernate/cache03.png)



































