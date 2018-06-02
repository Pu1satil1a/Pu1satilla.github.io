---
title: SpringMVC(三)
description: 视图解析器、SpringMVC第一个注解式开发程序
categories:
 - Web
 - Java
 - SpringMVC
tags: [Web, Java, SpringMVC]
---


# ModelAndView

模型与视图
- addObject()方法向模型中添加数据
- setViewName()方法向模型添加视图名称，表示即要跳转的页面

## 本质是HashMap
当调用addObject()方法时，即往模型中添加数据，实际上是往一个HashMap中添加数据。

查看源码：

往模型中添加数据，调用addObject()方法
``` java
mav.addObject("message", "这是doFirst()方法");
```

addObject()方法内调用getModleMap()方法获取模型对象，进一步查看其类型。
``` java
public ModelAndView addObject(String attributeName, Object attributeValue) {
	getModelMap().addAttribute(attributeName, attributeValue);
	return this;
}
```

发现类型为ModelMap,查看该类
``` java
public ModelMap getModelMap() {
	if (this.model == null) {
		this.model = new ModelMap();
	}
	return this.model;
}
```

发现继承ModelMap
``` java
public class ModelMap extends LinkedHashMap<String, Object> 
```

HashMap是一个单向链表数组，只能查找下一个元素，不能查找上一个元素。

## 视图解析器ViewResolver

视图解析器ViewResolver接口负责将处理结果生成View视图。常用的实现类有四种

### InternalResourceViewResolver视图解析器
该视图解析器用于完成对当前Web应用内部资源的封装与跳转。而对于内部资源的查找规则是，将ModelAndView中指定的视图名称与为视图解析器配置的前辍与后辍相结合的方式，拼接成一个Web应用内部资源路径。拼接规则是：前辍+ 视图名称+ 后辍。InternalResourceView解析器会把处理器方法返回的模型属性都存放到对应的**request**中，然后将请求转发到目标URL。

如果不指定前缀与后缀，则可以将相对路径直接填入setViewName()方法中即可。

缺点：
- 只可以完成对内部资源封装后的跳转，但无法跳向外部资源，如外部网页。
- 对于内部资源的定义，也只能定义一种格式的资源：存放在同一目录的统一文件类型的资源文件。

#### 配置文件
``` xml
<!--注册处理器映射器-->
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="urlMap">
		<!--指定键值对，键为请求路径，值为请求处理器-->
		<map>
			<entry key="/welcome.do" value-ref="myController"/>
		</map>
	</property>
</bean>

<!--
	注册处理器:
		注入属性方法名称解析器
-->
<bean id="myController" class="cn.Pu1satilla.handlers.MyController">
	<property name="methodNameResolver" ref="parameterMethodNameResolver"/>
</bean>

<!--注入方法名称解析器-->
<bean id="parameterMethodNameResolver"
	  class="org.springframework.web.servlet.mvc.multiaction.ParameterMethodNameResolver">
	<property name="paramName" value="param"/>
</bean>
```

#### 处理器
``` java
public class MyController extends MultiActionController {
	public ModelAndView doFirst(HttpServletRequest request, HttpServletResponse response) throws Exception {
		ModelAndView mav = new ModelAndView();

		mav.addObject("message", "这是doFirst()方法");

		mav.setViewName("/WEB-INF/demo/welcome.jsp");
		return mav;
	}
}	
```

### BeanNameViewResolver
BeanNameViewResolver视图解析器，顾名思义就是将资源封装为“Spring容器中注册的Bean实例”，ModelAndView通过设置视图名称为该Bean的id属性值来完成对该资源的访问。所以在springmvc.xml中，可以定义多个View视图Bean，让处理器中ModelAndView通过对这些Bean的id的引用来完成向View中封装资源的跳转。

#### 配置文件
``` xml
    <bean id="/welcome" class="cn.Pu1satilla.handlers.MyController"/>

    <!--指定id视图处理器-->
    <bean class="org.springframework.web.servlet.view.BeanNameViewResolver"/>

    <!--定义一个内部资源解析器-->
    <bean class="org.springframework.web.servlet.view.JstlView" id="jstlView">
        <property name="url" value="/WEB-INF/demo/welcome.jsp"/>
    </bean>
```

#### 处理器类
``` java
public class MyController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {

        ModelAndView mav = new ModelAndView();

        mav.addObject("message", "测试！！！！！！！！！！");

        mav.setViewName("jstlView");
        return mav;
    }
}
```

### XmlViewResolver
当需要定义的View视图对象很多时，就会使该xml文件变得很大，特别臃肿，不便于管理。配置视图解析器为XmlViewServlet时，通过设置属性location指定xml文件位置。
``` xml
<!--配置视图解析器：内部资源解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>


<!--配置视图解析器：外部xml视图解析器-->
<bean class="org.springframework.web.servlet.view.XmlViewResolver">
	<property name="location" value="classpath:Views.xml"/>
</bean>
```

View视图的配置文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
        定义一个内部资源解析器
            id为处理器输入视图值
        JstlView表示指定内部资源路径
    -->
    <bean class="org.springframework.web.servlet.view.JstlView" id="jstlView">
        <property name="url" value="/WEB-INF/demo/welcome.jsp"/>
    </bean>

    <!--定义一个外部资源view对象-->
    <bean class="org.springframework.web.servlet.view.RedirectView" id="baidu">
        <property name="url" value="http://baidu.com"/>
    </bean>
</beans>
```

### ResourceBundleViewResolver
当需要将视图View对象配置于properties文件中时，使用ResourceBundleViewResolver视图解析器。

格式要求：  
```
	资源名称.(class) = 封装资源的View全限定性类名
	资源名称.url = 资源路径
```

#### 定义properties文件
``` properties
#定义id为jstlView的视图解析器类
jstlView.(class)=org.springframework.web.servlet.view.JstlView
#定义id为jstlView的视图解析器路径
jstlView.url=/WEB-INF/demo/welcome.jsp

baidu.(class)=org.springframework.web.servlet.view.RedirectView
baidu.url=http://baidu.com
``` 

#### 配置文件
``` xml
<!--注册处理器-->
<bean id="/welcome.do" class="cn.Pu1satilla.handlers.MyController"/>

<!--properties文件内参数作为视图解析器-->
<bean class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
	<!--value值为src根路径下文件名前缀-->
	<property name="basename" value="spring-views"/>
</bean>
```

#### 处理器
``` java
public class MyController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {

        ModelAndView mav = new ModelAndView();

        mav.addObject("message", "测试！！！！！！！！！！");
        
        //        输入视图解析器id
        mav.setViewName("baidu");
        return mav;
    }
}
```

### 视图解析器优先级
有时经常需要应用一些视图解析器策略来解析视图名称，即当同时存在多个视图解析器均可解析ModelAndView中的同一视图名称时，哪个视图解析器会起作用呢？

视图解析器有一个order属性，专门用于设置多个视图解析器的优先级。**数字越小，优先级越高。数字相同，先注册的优先级高。**一般不为`InternalResourceViewResolver`解析器指定优先级，即让其优先级是最低的。

``` xml
<!--注册处理器-->
<bean id="/welcome.do" class="cn.Pu1satilla.handlers.MyController"/>

<!--配置内置路径视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/demo/"/>
	<property name="suffix" value=".jsp"/>
</bean>

<!--配置xml文件视图解析器-->
<bean class="org.springframework.web.servlet.view.XmlViewResolver">
	<property name="location" value="classpath:Views.xml"/>
	<property name="order" value="3"/>
</bean>

<!--配置properties文件视图解析器-->
<bean class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
	<property name="basename" value="spring-views"/>
	<property name="order" value="1"/>
</bean>
```




















































