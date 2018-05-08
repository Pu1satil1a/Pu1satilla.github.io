---
title: Hibernate(五)
description:  Hibernate表与表之间的关系，包含一对多，多对一以及多对多
categories:
 - Web
 - Java
 - Hibernate
tags: [Web, Java, Hibernate, MySQL, 多表]
---


# 一对多|多对一关系

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

将映射添加到主配置文件
``` xml
<mapping resource="cn/Pu1satilla/domain/Toucher.hbx.xml"/>
<mapping class="cn.Pu1satilla.domain.Toucher"/>
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

级联操作是指当主控方执行保存、更新或者删除操作时，其关联对象（被控方）也执行相同的操作。在映射文件中通过对cascade属性的设置来控制是否对关联对象采用级联操作，级联操作对各种关联关系都是有效的。

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

`cascade="save-update"`在相关联的属性设置级联，表示该实体类对象如果在`save`、`update`或者`saveOrUpdate`操作时，会将这个相关联的对象进行相同操作，不需要手动书写代码。主表进行配置`cascade`属性，可以控制相关联从表对象，从表配置`cascade`属性，可以控制相关联主表属性。

测试用例（主表控制从表）
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

### 关系维护

双向关联产生的多余SQL语句

#### 测试用例
``` java
@Test
public void demo6(){
	//        1.获得session对象
	Session session = HibernateUtils.getSession();

	//        2.开启事务
	Transaction transaction = session.beginTransaction();

	//        3.建立一个客户，创建两个联系人
	Customer customer = new Customer();
	customer.setCust_name("张三");

	Toucher toucher0 = new Toucher();
	Toucher toucher1 = new Toucher();
	toucher0.setTname("李四");
	toucher1.setTname("王五");

	//        建立关系
	toucher0.setCustomer(customer);
	toucher1.setCustomer(customer);
	customer.getToucherSet().add(toucher0);
	customer.getToucherSet().add(toucher1);

	//        级联保存联系人
	session.save(customer);	//因为在配置文件配置了主键自增（generator：native）以及级联保存（cascade=“save-update”）
										//Hibernate将customer、toucher0、toucher1持久化

	//        4.提交事务
	transaction.commit();

	//        5.释放资源
	session.close();
}
```
#### 控制台结果
```
Hibernate: 
    insert 
    into
        customer.customer
        (cust_name, cust_level, cust_source, cust_linkman, cust_phone, cust_mobile) 
    values
        (?, ?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        customer.toucher
        (tname, toffice_phone, tpersoner_phone, tsex, toucherLinkId) 
    values
        (?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        customer.toucher
        (tname, toffice_phone, tpersoner_phone, tsex, toucherLinkId) //在联系人插入时维护过一次外键（由从表对象联系人维护）
    values
        (?, ?, ?, ?, ?)
Hibernate: 
    update
        customer.toucher 
    set
        toucherLinkId=? //在客户更新时再次维护外键（由主表对象客户进行维护）
    where
        tid=?
Hibernate: 
    update
        customer.toucher 
    set
        toucherLinkId=? 
    where
        tid=? 
```
发现在insert语句就修改了外键的操作，在后续update操作又进行了一次，其中有一次就多余了。原因是在联系人插入时已经维护了一次外键而在客户更新又维护了一次外键。

通过放弃一方维护权避免两次维护外键内容（一的一方放弃外键的维护权）。

#### 配置
``` xml
<!--
	name属性：set集合属性名
	colum属性：外键列名，自定义
	class属性：与我关联的对象完整类名

	级联保存：（不需要代码操作，减少代码量）
	cascade
	save-update：级联保存更新
	delete：级联删除
	all：sava-update+delete

	关系维护：（一的一方放弃关系维护）
	inverse属性：放弃关系维护属性 true表示放弃，默认为false
-->
<set name="toucherSet" inverse="true">
	<key column="toucherLinkId"/>
	<one-to-many class="Toucher"/>
</set>
```
#### 再次测试
再次运行测试用例，查看控制台：
``` 
Hibernate: 
    insert 
    into
        customer.customer
        (cust_name, cust_level, cust_source, cust_linkman, cust_phone, cust_mobile) 
    values
        (?, ?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        customer.toucher
        (tname, toffice_phone, tpersoner_phone, tsex, toucherLinkId) 
    values
        (?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        customer.toucher
        (tname, toffice_phone, tpersoner_phone, tsex, toucherLinkId) 
    values
        (?, ?, ?, ?, ?)
```
发现由主表放弃了外键的维护权，没有进行外键的`update`操作。

















