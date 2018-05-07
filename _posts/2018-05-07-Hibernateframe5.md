---
title: Hibernate(五)
description:  Hibernate表与表之间的关系，包含一对多，多对一以及多对多
categories:
 - Web
 - Java
 - Hibernate
tags: [Web, Java, Hibernate, MySQL, 多表]
---


# 一对多|多对一关系表

## 关系表达

### 数据库表的表达
建表原则：在多的一方创建外键指向少的一方的主键
![](/assets/images/hibernate/table01.png)

### 实体的表达
![](/assets/images/hibernate/table02.png)

### orm元数据中的表达
一对多关系：
``` xml 
<!--集合，一对多关系：主表配置-->
<!--
	name属性：set集合属性名
	colum属性：外键列名，自定义必须与从表外键列名相同
	class属性：与我关联的实体类名
-->
<set name="toucherSet">
	<key column="toucherLinkId" />
	<one-to-many class="Toucher"/>
</set>
```

多对一关系：
``` xml
<!--多对一关系-->
<!--
	name属性：集合属性名
	colum属性：外键列名
	class属性：与我关联的实体类名
-->
<many-to-one name="customer" class="Customer" column="toucherLinkId"/>
```

## Hibernate操作

### 添加主表跟从表条目
``` java
@Test
public void demo1() {

	//        1.获得session对象
	Session session = HibernateUtils.getSession();

	//        2.开启事务
	Transaction transaction = session.beginTransaction();

	//        3.操作

	//        3.1建立一个主表对象客户
	Customer customer = new Customer();
	customer.setCust_name("西平平");

	//        3.2建立三个从表对象联系人
	Toucher toucher0 = new Toucher();
	toucher0.setTname("虚构才");

	Toucher toucher1 = new Toucher();
	toucher1.setTname("另计化");

	Toucher toucher2 = new Toucher();
	toucher2.setTname("波来袭");

	//        3.3多对一
	toucher0.setCustomer(customer);
	toucher1.setCustomer(customer);
	toucher2.setCustomer(customer);

	//        3.4一对多
	customer.getToucherSet().add(toucher0);
	customer.getToucherSet().add(toucher1);
	customer.getToucherSet().add(toucher2);

	//        3.5持久化创建的对象
	session.save(toucher0);
	session.save(toucher1);
	session.save(toucher2);
	session.save(customer);

	//        4.提交事务
	transaction.commit();

	//        5.释放资源
	session.close();
}
```

### 为指定主表条目添加从表条目
``` java
@Test
public void demo2() {

	//        1.获得session对象
	Session session = HibernateUtils.getSession();

	//        2.开启事务
	Transaction transaction = session.beginTransaction();

	//        3.为指定主表条目添加从表条目操作

	//        3.1获取指定主表对象
	Customer customer = (Customer) session.get(Customer.class, 1);

	//        3.2添加一个从表对象
	Toucher toucher = new Toucher();
	toucher.setTname("张三");

	//        3.3设置表之间关系
	customer.getToucherSet().add(toucher);
	toucher.setCustomer(customer);

	//        3.4持久化对象
	session.save(toucher);

	//        4.提交事务
	transaction.commit();

	//        5.释放资源
	session.close();
}
```

### 为指定主表条目删除从表条目
``` java
@Test
public void demo3() {

	//        1.获得session对象
	Session session = HibernateUtils.getSession();

	//        2.开启事务
	Transaction transaction = session.beginTransaction();

	//        3.为指定主表条目删除从表条目操作

	//        3.1获取主表条目对象
	Customer customer = (Customer) session.get(Customer.class, 1);

	//        3.2获取从表条目对象
	Toucher toucher = (Toucher) session.get(Toucher.class, 2l);

	//        3.3删除主从表对象之间关系
	customer.getToucherSet().remove(toucher);
	toucher.setCustomer(null);

	//        4.提交事务
	transaction.commit();

	//        5.释放资源
	session.close();
}
```

## Hibernate进阶操作

### 级联操作

级联，就是对一个对象进行操作的时候，会把他相关联的对象也一并进行相应的操作，相关联的对象意思是指 比如前两节学的一对多关系中，班级跟学生，Student的实体类中，存在着Classes对象的引用变量，如果保存Classes对象的引用变量有值的话，则该值就是相关联的对象，并且在对student进行save时，如果保存Classes对象的引用变量有值，那么就会将Classes对象也进行save操作， 这个就是级联的作用。

#### 级联保存更新
主表配置
``` xml
<!--集合，一对多关系-->
<!--
	name属性：set集合属性名
	colum属性：外键列名，自定义
	class属性：与我关联的对象完整类名
	
	级联保存：（不需要代码操作，减少代码量）
	cascade
	save-update：级联保存更新
	delete：级联删除
	all：sava-update+delete
-->
<set name="toucherSet" cascade="save-update">
	<key column="toucherLinkId"/>
	<one-to-many class="Toucher"/>
</set>
```

`cascade="save-update"`在相关联的属性设置级联，表示该实体类对象如果在`save`、`update`或者`saveOrUpdate`操作时，会将这个相关联的对象进行相同操作，不需要手动书写代码。

测试用例
``` java
/**
 * 测试级联操作（级联保存更新）
 */
@Test
public void demo4(){
	//        1.获得session对象
	Session session = HibernateUtils.getSession();

	//        2.开启事务
	Transaction transaction = session.beginTransaction();

	//        3.级联保存，不调用从表save()方法

	//        3.1建立主表条目对象，顾客
	Customer customer = new Customer();
	customer.setCust_name("西平平");

	//        3.2建立三个从表对象
	Toucher toucher0 = new Toucher();
	toucher0.setTname("虚构才");

	Toucher toucher1 = new Toucher();
	toucher1.setTname("另计化");

	Toucher toucher2 = new Toucher();
	toucher2.setTname("波来袭");

	//        3.3多对一
	toucher0.setCustomer(customer);
	toucher1.setCustomer(customer);
	toucher2.setCustomer(customer);

	//        3.4一对多
	customer.getToucherSet().add(toucher0);
	customer.getToucherSet().add(toucher1);
	customer.getToucherSet().add(toucher2);

	//        3.5不执行从表条目保存操作
	session.save(customer);

	//        4.提交事务
	transaction.commit();

	//        5.释放资源
	session.close();
}
```

#### 级联删除（不推荐使用）

主表配置
``` xml
<!--集合，一对多关系-->
<!--
	name属性：set集合属性名
	colum属性：外键列名，自定义
	class属性：与我关联的对象完整类名
	
	级联保存：（不需要代码操作，减少代码量）
	cascade
	save-update：级联保存更新
	delete：级联删除
	all：sava-update+delete
-->
<set name="toucherSet" cascade="delete">
	<key column="toucherLinkId"/>
	<one-to-many class="Toucher"/>
</set>
```
测试用例
``` java
/**
 * 测试级联操作（级联删除操作）
 */
@Test
public void demo5() {
	//        1.获得session对象
	Session session = HibernateUtils.getSession();

	//        2.开启事务
	Transaction transaction = session.beginTransaction();

	//        3.级联删除，从表条目不调用delete()方法

	//        删除主表条目对象
	session.delete(session.get(Customer.class,1));

	//        4.提交事务
	transaction.commit();

	//        5.释放资源
	session.close();
}
```
测试结果：主表对象条目对象被删除，与其相关联从表对象条目同时被删除。

























