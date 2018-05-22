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





























































































































