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

往模型中添加数据，调用addObject()方法s
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

# 注解式开发

注解式开发是重点。

## 首个程序

### web.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!--设置中央处理器配置位置-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!--classpath表示配置在src文件夹下-->
            <param-value>classpath:dispatcher-servlet.xml</param-value>
        </init-param>

        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```

### 注册组件扫描器
指定处理器所在基本包
``` xml
<!--注册组件扫描器-->
<context:component-scan base-package="cn.pu1satilla.controller"/>
```

### 定义处理器
此时的处理器类无需继承任何父类，实现任何接口。只需在类上与方法上添加相应注解即可。

- `@Controller`：表示当前类为处理器
- `@RequestMapping`：表示当前方法为处理器方法该方法要对value属性所指定的URL进行处理与响应。被注解的方法的方法名可以随意。

``` java
//定义该类为处理器类
@org.springframework.stereotype.Controller
public class FirstController {

    @RequestMapping("/test/firstRequest.do")
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {

        //        1.获取模型与视图
        ModelAndView mv = new ModelAndView();

        //        2.添加对象
        mv.addObject("message", "I am message");

        //        3.设置视图名称
        mv.setViewName("/index.jsp");
        return mv;
    }
}
```

当然，若有多个请求路径均可匹配该处理器方法的执行，则`@RequestMapping`的`value`属性中可以写上一个数组。

``` java
@RequestMapping({"/test/firstRequest.do","/test/firstRequest1.do"})
```

## 请求映射规则的定义

### 命名空间的定义
通过`@RequestMapping`注解可以定义处理器对于请求的映射规则。该注解可以注解在方法上，也可以注解在类上，但意义是不同的。

当url过于冗余时，可以在类上以及方法上同时设置，映射路径为类上路径+方法上路径。

在类上即为uri的命名空间，为方法中相同的uri部分
``` java
//定义该类为处理器类
@RequestMapping({"/test/"})
@org.springframework.stereotype.Controller
public class FirstController {

    @RequestMapping({"firstRequest.do","firstRequest1.do"})
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {

        //        1.获取模型与视图
        ModelAndView mv = new ModelAndView();

        //        2.添加对象
        mv.addObject("message", "I am message");

        //        3.设置视图名称
        mv.setViewName("/index.jsp");
        return mv;
    }
}
```

### uri中通配符的应用

#### 资源名称中使用通配符
```
/*xxx.do
```
表示只要请求的资源名称为结尾。
```
/xxx*.do
```
表示只要请求的资源名称为开头。

#### 资源路径中使用通配符
```
/xxx/*/xxxx.do
```
表示在xxxx.do资源名称前面，只能有两级路径，第一级必须是/xxx，第二级任意
```
 /xxx/**/xxxx.do
```
表示在xxxx.do的资源名称前面，必须以/xxx路径开头，而其它级的路径是否包含，若包含又包含几级，各级又叫什么名称，均随意。这种称为路径级数的可变匹配

#### 对请求方式的定义
``` java
//定义该类为处理器类
@RequestMapping({"/test/"})
@org.springframework.stereotype.Controller
public class FirstController {

    @RequestMapping(value = "/firstRequest.do", method = RequestMethod.POST)
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {

        //        1.获取模型与视图
        ModelAndView mv = new ModelAndView();

        //        2.添加对象
        mv.addObject("message", "I am message");

        //        3.设置视图名称
        mv.setViewName("/index.jsp");
        return mv;
    }
}
```
表示该资源只能通过post方式请求获得。

#### 对请求中携带参数的定义

`@RequestMapping中params`属性中定义了请求中必须携带的参数的要求。以下是几种情况的说明。  
`@RequestMapping(value=”/xxx.do”, params={“name”,”age”})` ：要求请求中必须携带请求参数name与age  
`@RequestMapping(value=”/xxx.do”, params={“!name”,”age”}) `：要求请求中必须携带请求参数age，但必须不能携带参数name  
`@RequestMapping(value=”/xxx.do”, params={“name=zs”,”age=23”})` ：要求请求中必须携带请求参数name，且其值必须为zs；必须携带参数age，其其值必须为23  
`@RequestMapping(value=”/xxx.do”, params=“name!=zs”) `：要求请求中必须携带请求参数name，且其值必须不能为zs

## 处理器方法的参数
处理器方法可以包含以下五类参数，这些参数会在系统调用时由系统自动赋值，即程序员可在方法内直接使用。

- HttpServletRequest
- HttpServletResponse
- HttpSession
- 用于承载数据的Model
- 请求中所携带的请求参数

**此处先记录第5条如何处理请求中所携带的请求参数**

### 逐个参数接收

发送请求参数视图文件`firstReuqestForm.jsp`
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="${pageContext.request.contextPath}/test/firstRequestForm.do" method="post">
    name:<input type="text" name="name">
    age: <input type="text" name="age">
    <input type="submit">
</form>
</body>
</html>
```

请求处理器处理请求方法
``` java
@RequestMapping(value = "firstRequestForm",method = RequestMethod.POST)
public ModelAndView firstRequestForm(String name, int age) throws Exception {

	ModelAndView mv = new ModelAndView();

	mv.addObject("name", name);
	mv.addObject("age", age);
	mv.setViewName("/WEB-INF/acceptRequest.jsp");
	return mv;
}
```

视图文件
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    name=${name} &nbsp;
    age=${age}
</body>
</html>
```

### 请求参数中文乱码
对于前面所接收的请求参数，若含有中文，则会出现中文乱码问题。Spring对于请求参数中的中文乱码问题，给出了专门的字符集过滤器：`spring-web-4.2.4.RELEASE.jar的org.springframework.web.filter`包下的`CharacterEncodingFilter`类。
![](/assets/images/springMVC/charset.png)

配置文件注册过滤器
``` xml
<!--配置乱码解决过滤器-->
<filter>
	<filter-name>CharacterEncoding</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

	<!--设置属性-->
	<init-param>
		<param-name>encoding</param-name>
		<param-value>utf-8</param-value>
	</init-param>

	<!--设置强制使用指定的字符集：true则代码中指定的字符集不起作用，
			false则代码中若指定了字符集，就使用代码指定字符集，
		没指定就使用xml配置字符集-->
	<init-param>
		<param-name>forceEncoding</param-name>
		<param-value>true</param-value>
	</init-param>
</filter>

<!--过滤路径-->
<filter-mapping>
	<filter-name>CharacterEncoding</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

### 校正请求参数名@RequestParam
当请求参数跟接收参数不一致时，就无法接收到对应参数，要求不能更改参数名，接收到参数，这时需要使用`@RequestParam`。

修改jsp文件，将参数设置为pname，page。
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="${pageContext.request.contextPath}/test/firstRequestForm.do" method="post">
    name:<input type="text" name="pname">
    age:   <input type="text" name="page">
    <input type="submit">
</form>
</body>
</html>
```

给控制器请求参数添加注解参数
``` java
    @RequestMapping(value = "firstRequestForm", method = RequestMethod.POST)
    public ModelAndView firstRequestForm(@RequestParam(name = "pname") String name, 
                                         @RequestParam(name = "page") int age) 
            throws Exception {

        ModelAndView mv = new ModelAndView();

        mv.addObject("name", name);
        mv.addObject("age", age);
        mv.setViewName("/WEB-INF/acceptRequest.jsp");
        return mv;
    }
```

### 整体参数接受

将处理器方法的参数定义为一个对象，只要保证请求参数名与这个对象的属性同名即可。

修改接收参数
``` java
@RequestMapping(value = "firstRequestFormByPerson", method = RequestMethod.POST)
public ModelAndView firstRequestFormByPerson(Person person) {

	ModelAndView mv = new ModelAndView();

	mv.addObject("person", person);
	mv.setViewName("/WEB-INF/acceptRequest.jsp");

	return mv;
}
```

jsp文件
``` jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    person=${person}
</body>
</html>
```

### 域属性参数接收

定义school实体类，修改person实体类
``` java
public class Person {
    private String pname;
    private int page;
    private School pschool;

    public School getPschool() {
        return pschool;
    }

    public void setPschool(School pschool) {
        this.pschool = pschool;
    }

    public String getPname() {
        return pname;
    }

    public void setPname(String pname) {
        this.pname = pname;
    }

    public int getPage() {
        return page;
    }

    public void setPage(int page) {
        this.page = page;
    }

    @Override
    public String toString() {
        return "Person{" +
                "pname='" + pname + '\'' +
                ", page=" + page +
                ", pschool=" + pschool +
                '}';
    }
}

public class School {
    private String sname;
    private String location;

    public String getSname() {
        return sname;
    }

    public void setSname(String sname) {
        this.sname = sname;
    }

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    @Override
    public String toString() {
        return "School{" +
                "sname='" + sname + '\'' +
                ", location='" + location + '\'' +
                '}';
    }
}
```

修改请求页面
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>

<body>

<form action="${pageContext.request.contextPath}/test/firstRequestFormByPerson.do" method="post">
    name:<input type="text" name="pname"> &nbsp;
    age: <input type="text" name="page">  &nbsp;
    学校名:  <input type="text" name="pschool.sname"> &nbsp;
    学校地址: <input type="text" name="pschool.location"> &nbsp;
    <input type="submit">
</form>

</body>

</html>
```

### 路径变量@PathVariable

## 处理器方法的返回值

### 返回ModelAndView
若处理器方法处理完后，需要跳转到其它资源，且又要在跳转的资源间传递数据，此时处理器方法返回ModelAndView比较好。当然，若要返回ModelAndView，则处理器方法中需要定义ModelAndView对象。在使用时，若该处理器方法只是进行跳转而不传递数据，或只是传递数据而并不向任何资源跳转（如对页面的Ajax异步响应），此时若返回ModelAndView，则将总是有一部分多余：要么Model多余，要么View多余。即此时返回ModelAndView将不合适。

### 返回String
处理器方法返回的字符串可以指定逻辑视图名，通过视图解析器解析可以将其转换为物理视图地址。

#### 返回内部资源逻辑视图名
可以在中央调度器中设置路径的前缀以及后缀。
``` xml
    <!--注册视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```

调用处理器方法
``` java
@RequestMapping(value = "returnString.do")
public String returnString(HttpServletRequest request) {

	request.setAttribute("message", "returnString");

	return "index";
}
```

在web文件根目录下修改index.jsp文件
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>$Title$</title>
</head>
<body>
${message}
</body>
</html>
```

当然也可以写内部路径全名，删除视图解析器配置，修改返回路径值即可，不进行详细解释。

#### 返回View对象名
若要跳转的资源是外部资源，则视图解析器可以使用BeanNameViewResolver。详细配置见![配置式视图解析器](http://localhost:4000/web/java/springmvc/2018/06/02/springMVC_framework3/#beannameviewresolver)

配置完成后处理器返回该视图解析器id即可。

### 返回void

对于处理器方法返回void的应用场景，主要有两种

#### ServletAPI
将处理器类似于Servlet使用，完成模型构建，通过reqeust进行转发给指定jsp文件，或者通过response进行重定向某资源。

#### Ajax响应
若处理器对请求处理后，无需跳转到其它任何资源，此时可以让处理器方法返回void。

##### 导包


### 返回Object




























