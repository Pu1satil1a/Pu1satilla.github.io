---
title: SpringMVC(四)
description: 注解式开发
categories:
 - Web
 - Java
 - SpringMVC
tags: [Web, Java, SpringMVC]
---


注解式开发是重点。

# 首个程序

## web.xml
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

## 注册组件扫描器
指定处理器所在基本包
``` xml
<!--注册组件扫描器-->
<context:component-scan base-package="cn.pu1satilla.controller"/>
```

## 定义处理器
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

# 请求映射规则的定义

## 命名空间的定义
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

## uri中通配符的应用

### 资源名称中使用通配符
```
/*xxx.do
```
表示只要请求的资源名称为结尾。
```
/xxx*.do
```
表示只要请求的资源名称为开头。

### 资源路径中使用通配符
```
/xxx/*/xxxx.do
```
表示在xxxx.do资源名称前面，只能有两级路径，第一级必须是/xxx，第二级任意
```
 /xxx/**/xxxx.do
```
表示在xxxx.do的资源名称前面，必须以/xxx路径开头，而其它级的路径是否包含，若包含又包含几级，各级又叫什么名称，均随意。这种称为路径级数的可变匹配

### 对请求方式的定义
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

### 对请求中携带参数的定义

`@RequestMapping中params`属性中定义了请求中必须携带的参数的要求。以下是几种情况的说明。  
`@RequestMapping(value=”/xxx.do”, params={“name”,”age”})` ：要求请求中必须携带请求参数name与age  
`@RequestMapping(value=”/xxx.do”, params={“!name”,”age”}) `：要求请求中必须携带请求参数age，但必须不能携带参数name  
`@RequestMapping(value=”/xxx.do”, params={“name=zs”,”age=23”})` ：要求请求中必须携带请求参数name，且其值必须为zs；必须携带参数age，其其值必须为23  
`@RequestMapping(value=”/xxx.do”, params=“name!=zs”) `：要求请求中必须携带请求参数name，且其值必须不能为zs

# 处理器方法的参数
处理器方法可以包含以下五类参数，这些参数会在系统调用时由系统自动赋值，即程序员可在方法内直接使用。

- HttpServletRequest
- HttpServletResponse
- HttpSession
- 用于承载数据的Model
- 请求中所携带的请求参数

**此处先记录第5条如何处理请求中所携带的请求参数**

## 逐个参数接收

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

## 请求参数中文乱码
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

当使用过滤器filter还是出现乱码，则需要修改tomcat的server.xml文件
``` xml
   <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443"  URIEncoding="utf-8"/>
```

## 校正请求参数名@RequestParam
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

## 整体参数接受

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

## 域属性参数接收

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

## 路径变量@PathVariable

# 处理器方法的返回值

## 返回ModelAndView
若处理器方法处理完后，需要跳转到其它资源，且又要在跳转的资源间传递数据，此时处理器方法返回ModelAndView比较好。当然，若要返回`ModelAndView`，则处理器方法中需要定义`ModelAndView`对象。在使用时，若该处理器方法只是进行跳转而不传递数据，或只是传递数据而并不向任何资源跳转（如对页面的Ajax异步响应），此时若返回ModelAndView，则将总是有一部分多余：要么Model多余，要么View多余。即此时返回ModelAndView将不合适。

## 返回String
处理器方法返回的字符串可以指定逻辑视图名，通过视图解析器解析可以将其转换为物理视图地址。

### 返回内部资源逻辑视图名
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

### 返回View对象名
若要跳转的资源是外部资源，则视图解析器可以使用BeanNameViewResolver。详细配置见[配置式视图解析器](http://Pu1satilla.github.io/web/java/springmvc/2018/06/02/springMVC_framework3/#beannameviewresolver)

配置完成后处理器返回该视图解析器id即可。

## 返回void

对于处理器方法返回void的应用场景，主要有两种

### ServletAPI
将处理器类似于Servlet使用，完成模型构建，通过reqeust进行转发给指定jsp文件，或者通过response进行重定向某资源。

### Ajax响应
若处理器对请求处理后，无需跳转到其它任何资源，此时可以让处理器方法返回void。

#### 导包
导入fastJson解析包
![](/assets/images/springMVC/JSON_jar.png)

#### 引入JQuery库
![](/assets/images/springMVC/js.png)

#### 前台页面
``` jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
</head>
<body>

<input type="button" value="Ajax请求" id="button">

</body>

<%--引入js库--%>
<script type="text/javascript" src="js/jquery-1.8.3.js"></script>

<%--点击发送Ajax请求--%>
<script>
    $(function () {
        $("#button").click(function () {
            $.ajax({
                type: "POST",
                url: "${pageContext.request.contextPath}/test/ajax.do",
                data: {
                    pname: "ajax测试",
                    page:0
                },
                success: function (data) {
                    alert(data.name);
                },
                dataType: "json"
            })
        })
    })
</script>

</html>
```

#### 后台代码
``` java
@RequestMapping(value = "ajax.do")
public void AjaxDemo(HttpServletResponse response, Person person) throws IOException {
	//        1.将数据存入Map中
	Map<String, Object> map = new HashMap<>();
	map.put("name", person.getPname());
	map.put("age", person.getPage());

	//        2.将map转为json
	String string = JSON.toJSONString(map);

	//        3.将数据返回客户端
	PrintWriter writer = response.getWriter();
	writer.write(string);
	writer.close();
}
```

## 返回Object

一般返回值都是作为模型与视图对象，但是返回的Object类型对象则不是作为模型与视图对象出现，仅仅是在页面显示的数据出现的。

### 导入Jar包
由于返回Object数据，一般都是讲数据转化为JSON对象后床底给浏览器页面。而这个有Object转换为JSON，是由Jackson工具完成的。所以需要导入Jackson的相关jar包
![](/assets/images/springMVC/jackson.png)

### 注册注解驱动
将Object数据转化为JSON数据，需要有http消息转换器HttpMessageConverter完成。当spring容器进行初始化过程中，通过配置就创建了注解驱动，默认创建了7个`HttpMessageConverter`对象。
``` xml
<!--注册HttpMessageConverter对象-->
<mvc:annotation-driven/>
```

### 返回数值型对象
修改控制器方法，添加注解`@ResponseBody`
``` java
@RequestMapping(value = "returnNumber.do")
@ResponseBody
public Object returnNumber() {
	return 123.456;
}
```

修改请求文件
``` html
<input type="button" value="Ajax请求" id="button">
<input type="button" value="返回Object对象类型为数字" id="returnNumber">
```

``` html
// 点击按钮发送请求给/test/returnNumber
$(function () {
	$("#returnNumber").click(function () {
		$.ajax({
			type: "POST",
			url: "${pageContext.request.contextPath}/test/returnNumber.do",
			success: function (data) {
				alert(data);
			},
			dataType: "json"
		})
	})
})
```

### 返回字符串对象
修改控制器方法，当返回值内包含中文需要注释添加属性prodeces，表示指定字符集，设置输出结果类型。
``` java
    @RequestMapping(value = "returnObjectString.do",
                produces = "test/html;charset=utf-8")
    @ResponseBody
    public Object returnString() {
        return "返回String测试";
    }
```

### 返回自定义类型对象
``` java
@RequestMapping(value = "returnObject.do")
@ResponseBody
public Object returnObject() {
	Person person = new Person();
	person.setPname("小明");
	person.setPage(22);
	return person;
}
```

``` html
<input type="button" value="Ajax请求" id="button">
<input type="button" value="返回Object对象类型为数字" id="returnNumber">
<input type="button" value="返回Object对象类型为字符串" id="returnString">
<input type="button" value="返回Object对象类型为自定义类型" id="returnObject">

// 点击按钮发送请求给/test/returnObject
$(function () {
	$("#returnObject").click(function () {
		$.ajax({
			type: "POST",
			url: "${pageContext.request.contextPath}/test/returnObject.do",
			success: function (data) {
				alert(data.pname);
			},
			dataType: "json"
		})
	})
});
```
### 返回Map集合
``` java
@RequestMapping(value = "returnMap.do")
@ResponseBody
public Object returnMap() {
	Person person1 = new Person();
	person1.setPname("小明");
	person1.setPage(22);

	Person person2 = new Person();
	person2.setPname("张三");
	person2.setPage(23);

	HashMap<String, Person> map = new HashMap<>();
	map.put("person1",person1);
	map.put("person2",person2);
	return map;
```

``` html
<input type="button" value="返回Object对象类型为Map集合" id="returnMap">

// 点击按钮发送请求给/test/returnMap
$(function () {
	$("#returnMap").click(function () {
		$.ajax({
			type: "POST",
			url: "${pageContext.request.contextPath}/test/returnMap.do",
			success: function (data) {
				alert(data.person2.pname);
			},
			dataType: "json"
		})
	})
});
```

### 返回List集合
``` java
@RequestMapping(value = "returnList.do")
@ResponseBody
public Object returnList() {
	Person person1 = new Person();
	person1.setPname("小明");
	person1.setPage(22);

	Person person2 = new Person();
	person2.setPname("张三");
	person2.setPage(23);

	ArrayList<Person> list = new ArrayList<>();
	list.add(person1);
	list.add(person2);
	return list;
}
```

``` html
<input type="button" value="返回Object对象类型为List集合" id="returnList">

// 点击按钮发送请求给/test/returnList
$(function () {
	$("#returnList").click(function () {
		$.ajax({
			type: "POST",
			url: "${pageContext.request.contextPath}/test/returnList.do",
			success: function (data) {
				alert(data[1].pname + data[0].pname);
			},
			dataType: "json"
		})
	})
});
```






















