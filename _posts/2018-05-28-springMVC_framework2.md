---
title: SpringMVC(二)
description: SpringMVC配置式开发
categories:
 - Web
 - Java
 - SpringMVC
tags: [Web, Java, SpringMVC]
---

配置式开发指的是：处理器是程序员手工定义的、实现了特定接口的类，然后再在SpringMVC配置文件中对该类进行显式的、明确的注册。

# 处理器映射器HandlerMapping
`HandlerMapper`接口负责根据reqeust请求找到对应Handler处理器及Interceptor拦截器，并将它们封装在`HandlerExecutionChain`对象中，返回给中央调度器。

## 源码分析

### 获取所有处理器映射器
1. 当中央处理器DispatcherServlet被创建时，tomcat调用init()方法（其父类HttpServletBean类方法init()）
2. init()方法内调用其父类FrameworkServlet方法initServletBean()方法，initWebApplicationContext()初始化创建springmvc容器，该容器根据配置文件生成很多bean对象，其中就包含处理器适配器，处理器映射器，处理器等等
3. DispatcherServlet从springmvc中取出所有处理器映射器并且赋值给DispatcherServlet成员变量

`DispatcherServlet`类方法`initHandlerMappings`用来给成员变量赋值。
``` java
//		该方法是给成员变量handlerMappings赋值的
private void initHandlerMappings(ApplicationContext context) {
	this.handlerMappings = null;

	if (this.detectAllHandlerMappings) {//默认为true，查询所有处理器映射器，给成员变量handlerMappings赋值，寻找到所有处理器映射器。
		// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
		Map<String, HandlerMapping> matchingBeans =
				BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
		if (!matchingBeans.isEmpty()) {
			this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
			// We keep HandlerMappings in sorted order.
			AnnotationAwareOrderComparator.sort(this.handlerMappings);
		}
	}
	else {
		try {
			HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
			this.handlerMappings = Collections.singletonList(hm);
		}
		catch (NoSuchBeanDefinitionException ex) {
			// Ignore, we'll add a default HandlerMapping later.
		}
	}

	// Ensure we have at least one HandlerMapping, by registering
	// a default HandlerMapping if no other mappings are found.
	if (this.handlerMappings == null) {
		this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
		if (logger.isDebugEnabled()) {
			logger.debug("No HandlerMappings found in servlet '" + getServletName() + "': using default");
		}
	}
}
```

``` java
protected void initStrategies(ApplicationContext context) {
	initMultipartResolver(context);
	initLocaleResolver(context);
	initThemeResolver(context);
	initHandlerMappings(context);
	initHandlerAdapters(context);
	initHandlerExceptionResolvers(context);
	initRequestToViewNameTranslator(context);
	initViewResolvers(context);
	initFlashMapManager(context);
}
```
该方法被`initStrategies()`方法调用，`initStrategies()`被`onRefresh()`方法调用，其父类以及爷爷类有`init()`方法间接调用该方法，而`init()`方法为该Servlet创建时调用方法，因此在该Servlet创建时就确定好所有的处理器映射器、处理器适配器等等。

### 处理器映射器获取处理器执行链    

1. 当`tomcat`接受到请求，且匹配到`DispatcherServlet`来处理，调用`doService()`方法，`reqeust`作为参数传递。**doDispatch()方法进行调度，大部分功能都在该方法内执行。**
2. `doService()`方法里面调用`doDispatch()`方法，以`request`作为参数传递。
3. `doDispatch()`方法获取处理器执行链，调用`DispatcherServlet`类方法`getHandler()`方法获取处理器执行链。

`DispatcherServlet`有一个`doService()`方法，用于处理请求以及响应，在客户端访问时tomcat分配该路径时会被调用。该方法为当前`request`确定处理器，返回处理器执行链给`DispatcherServlet`。
``` java
try {
	doDispatch(request, response);
}
```

`doDispatch()`方法调用类方法`getHandler()`为该请求获取处理器执行链。
``` java
// Determine handler for the current request.
mappedHandler = getHandler(processedRequest);
```

获取当前请求处理器执行链方法`getHandler()`
``` java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
	/*
	 * 循环遍历所有spring的处理器映射器
	 *	if 处理器映射器方法调用getHandler()方法获取到处理器执行链，则跳出循环，
	 * 返回处理器执行链给调用方法doDispatch()
	 */
	for (HandlerMapping hm : this.handlerMappings) {
		if (logger.isTraceEnabled()) {
			logger.trace(
					"Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
		}
		HandlerExecutionChain handler = hm.getHandler(request);
		if (handler != null) {
			return handler;
		}
	}
	return null;
}
```
不同处理器映射器接口`HandlerMapping`实现类自定义获取当前请求处理器映射器方法。常用实现类有`BeanNameUrlHandlerMapping`与`SimpleURLHandlerMapping`，其中`BeanNameUrlHandlerMapping`是spring默认的，在`DispatchServlet.properties`文件中进行指定，需要使用后者则需要在`dispatcher`配置文件中进行指定。

## BeanNameUrlHandlerMapping（默认）
会根据请求的URL与spring容器中定义的处理器bean的name属性值进行匹配，从而在spring容器中找到处理器bean实例。 

``` xml
    <!--注册处理器映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!--注册处理器-->
    <bean id="/x.form" class="cn.Pu1satilla.handlers.MyController"/>
```
分析源码：
``` java
public class BeanNameUrlHandlerMapping extends AbstractDetectingUrlHandlerMapping {

	/**
	 * Checks name and aliases of the given bean for URLs, starting with "/".
	 */
	@Override
	protected String[] determineUrlsForHandler(String beanName) {
		List<String> urls = new ArrayList<String>();
		if (beanName.startsWith("/")) {
			urls.add(beanName);
		}
		String[] aliases = getApplicationContext().getAliases(beanName);
		for (String alias : aliases) {
			if (alias.startsWith("/")) {
				urls.add(alias);
			}
		}
		return StringUtils.toStringArray(urls);
	}

}
```
类方法`determineUrlsForHandler`返回处理之后的urls数组，必须是`startWith("/")`，而urls数组中的url则是中央调度器用于判定该url所对应的类是否作为处理器交给处理器适配器进行适配的依据。

## SimpleUrlHandlerMapping
使用`BeanNameUrlHandlerMapping`映射器缺点：
- 处理器Bean的id为一个url请求路径，而不是Bean的名称，有些不伦不类。
- 处理器Bean的定义与请求url绑定在了一起。若出现多个url请求同一个处理器的情况，就需要在Spring容器中配置多个该处理器类的`<bean/>`。这将导致容器会创建多个该处理器类实例

`SimpleUrlHandlerMapping`处理器映射器，不仅可以将url与处理器的定义分离，还可以对url进行统一映射管理。`SimpleUrlHandlerMapping`处理器映射器，会根据请求的url与Spring容器中定义的处理器映射器子标签的key属性进行匹配。匹配上后，再将该key的value值与处理器bean的id值进行匹配，从而在spring容器中找到处理器bean。

### mappings属性
通过对mapping属性进行配置，指定key为访问路径，值为对应处理器id
``` xml
    <!--注册处理器映射器-->
    <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="mappings">
            <props>
                <prop key="/x.form">controllerDemo1</prop>
            </props>
        </property>
    </bean>

    <!--注册处理器-->
    <bean id="controllerDemo1" class="cn.Pu1satilla.handlers.MyController"/>
```

### urlMap属性
``` xml
    <!--注册处理器映射器-->
    <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="urlMap">
            <map>
                <entry key="/x.form" value-ref="controllerDemo1"/>
            </map>
        </property>
    </bean>

    <!--注册处理器-->
    <bean id="controllerDemo1" class="cn.Pu1satilla.handlers.MyController"/>
```

# 处理器适配器
类似于电源适配器，不同电器所需电压不一致（3v,5v,8v...），而供电所供家庭用电为220v，这样就需要对应电源适配器将220v的电源转换成所需大小电源。

中央调度器根据处理器执行链中的处理器，找到能够执行该处理器的处理器适配器。处理器适配器调用执行处理器。

## 适配器模式
使得原本由于接口不兼容而不能一起工作的那些类可以在一起工作。所以处理器适配器所起的作用是，将多种处理器（实现了不同接口的处理器），通过处理器适配器的适配，使它们可以进行统一标准的工作，对请求进行统一方式的处理。

定义两个实体类，同时具有speak()方法
``` java
public class Dog {
    public void speak(){
        System.out.println("汪汪汪");
    }
}

public class Cat {
    public void speak(){
        System.out.println("喵喵喵");
    }
}
```

定义一个适配器类，具有speak()方法，以实体类作为参数传入，定义一个测试类
``` java
public class Adapter {
    public void speak(Object object) {
        if (object instanceof Cat) {
            ((Cat) object).speak();
        } else if (object instanceof Dog) {
            ((Dog) object).speak();
        }
    }
}

class test{
    public static void main(String[] args) {
        new Adapter().speak(new Cat());
    }
}
```
例子中通过适配器传入对象，调用适配器方法实现对象方法，对于调用实现了不同接口的类统一工作

常用处理器适配器有SimpleControllerHandlerAdapter和HttpRequestHandlerAdapter

## 流程及源码
1. 中央调度器根据执行链中的处理器找到相应处理器适配器
2. 处理器适配器执行处理器，将返回结果集传递给中央调度器

**以下没注明类所属则都为DispatcherServlet**
### 获取处理器适配器

从处理器映射器中获取到处理器执行链，从而在处理器执行链中获取处理器

``` java
// Determine handler adapter for the current request.
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

`getHandlerAdapter()`方法获取处理器是配置，将处理器作为参数传入。
``` java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
	for (HandlerAdapter ha : this.handlerAdapters) {
		if (logger.isTraceEnabled()) {
			logger.trace("Testing handler adapter [" + ha + "]");
		}
		//		如果处理器适配器支持处理器
		if (ha.supports(handler)) {
			return ha;
		}
	}
	throw new ServletException("No adapter for handler [" + handler +
			"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```
对所有spring处理器适配器进行遍历，遍历时将处理器handler作为参数传入处理器适配器supports()方法中，返回布尔值，
真则表示该次遍历的处理器适配器支持reqeust对应的处理器，返回该适配器，跳出循环。

### 处理器适配器执行处理器
``` java
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

## SimpleControllerHandlerAdapter
判断处理器是否实现了Controller接口，调用该适配器supports()方法。
``` java
public boolean supports(Object handler) {
	return (handler instanceof Controller);
}
```

判断结果为真，则表示处理器实现了Controller接口，进行调用该处理器处理请求`handleRequest()`方法。
``` java
public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {

	return ((Controller) handler).handleRequest(request, response);
}
```
该方法用于处理用户提交的请求。通过调用Service层代码，实现对用户请求的计算响应，并最终将计算所得数据及要响应的页面，封装为一个对象ModelAndView，返回给中央调度器。

## HttpRequestHandlerAdapter
适配实现了`HttpReqeustHandler`接口的处理器。
``` java
@Override
public boolean supports(Object handler) {
	return (handler instanceof HttpRequestHandler);
}

@Override
public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {

	((HttpRequestHandler) handler).handleRequest(request, response);
	return null;
}
```

# 处理器
处理器除了实现Controller接口外，还可以继承自一些其它的类来完成一些特殊的功能。































































































































































