---
title: Idea 部署web项目
description: 如何使用工具Idea部署web项目 
categories:
 - Web
 - IDEA
tags: [Web, Idea]
---

# WEB-INF&META-INF
1. 打开项目结构
2. 找到facets选项
3. 点击+，并且选择web.xml，设置WEB-INF路径
4. 点击Add Application Server specific descriptor... 选择tomcat context descriptor和默认配置文件路径
5. 编写xml文件
6. 编写代码从config.xml中获取配置的内容

![](/assets/images/IDEA/INF.png)
完成Idea配置文件的配置