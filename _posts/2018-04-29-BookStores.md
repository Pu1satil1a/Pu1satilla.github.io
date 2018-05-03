---
title: 书城项目(一)
description: 通过完成书城项目，巩固Ajax|jsp+Servlet+mysql知识，第一节完成书城注册功能
categories:
 - Web
 - Java
 - Project
 - Mysql
 - Jsp
 - Ajax
tags: [Web, Java,Project]
---


# 项目开发流程(公司之间是有差异的)
## 面试：
1. 项目金额
不知道，公司保密
2. 项目大小
询问哪方面：
- 多少人开发（根据自己写的项目定）
- 项目周期（商城1年多，进销存系统3-5个月）

## 项目步骤
1. 确定项目需求 ----- 外包（拿下一个项目）
2. 编写《需求说明书》----- 不涉及技术，只涉及业务需求                 需要跟用户沟通
3. 编写《概要涉及说明书》----- 涉及技术的宏观内容，数据库涉及，页面原型   需要跟用户沟通
4. 编写《详细涉及说明书》----- 相当于伪代码
5. 编码阶段coding ----- 根据《详细设计说明书》 ----- 单元测试（项目一年时间，coding4个月）
6. 联测 ----- 项目组内部的行为
7. 测试组进行全面的专业测试 ----- 《测试报告》
8. 上线（测试阶段）
9. 维护和二次开发 
 
# 注册

## 导包
![]()

## 步骤
1.  使用Ajax | Jsp + Serlvet + MySQL
2. Jsp使用js的validate插件完成用户注册以及用户名是否存在校验

## 建表
``` sql

```
  
 
# 出现的问题

## JQuery

1. 获取当前元素对象 
$("xxx").事件(
	function(){
		$(this)
	}
) 
2. 小心将data写成date
3. 周期函数setInterview

## Ajax
1. 如果设置异步请求，注意在异步请求没有完成，全局的代码已经完成好，可能会造成Ajax代码无法作用，需要将异步Ajax设置成同步　

## Jsp

1. 获取域中对象${xxxxx} 
2. 不能实时加载域对象，若是全局获取域对象属性，局部进行后台刷新，并且获取域对象属性，全局域对象属性不变。不能跟Ajax一样实时获取
3. 尽量用ajax 
 
## 后台JSON
``` java
jsonObject = JSON.parseObject("{'boolean':'true'}");
response.getWriter().write(String.valueOf(jsonObject));
```
 ## 表单校验
 
1. 重复密码----->"equalTo":"#password",#password为第一次输入密码的input框的id，不对应校验会无法通过
``` html
// 校验表单
$("#register_form").validate(
	{
		rules: {
			"username": {
				"required": true,
				"rangelength": [5, 8],
				"isExist": true
			},
			"password": {
				"required": true,
				"rangelength": [6, 12]
			},
			"confirm_password": {
				"required": true,
				"equalTo": "#password"
			},
			"email": {
				"required": true,
				"email": true
			}
		},
		messages: {
			"username": {
				"required": "请输入用户名",
				"rangelength": "用户名长度为5-8",
				"isExist": "用户名已存在"
			},
			"password": {
				"required": "请输入密码",
				"rangelength": "密码长度为6-12"
			},
			"confirm_password": {
				"required": "请再次输入密码",
				"equalTo": "密码必须一致"
			},
			"email": {
				"required": "请输入邮箱地址",
				"email": "请输入正确邮箱格式"
			}
		}
	}
);
``` 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 