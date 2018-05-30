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

`DispatcherServlet`有一个`doService()`方法，用于处理请求以及响应，在客户端访问时tomcat分配该路径时会被调用。该方法为当前`request`确定处理器，返回处理器执行链给`DispatcherServlet`。
``` java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
				//	为当前请求获取处理器执行链，getHandler()方法
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Error err) {
			triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```

获取处理器执行链方法`getHandler()`
``` java
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		/*
		 * 循环遍历所有以及被赋值处理器映射器
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

## 流程
1. 中央调度器根据执行链中的处理器找到相应处理器适配器
2. 处理器适配器执行处理器
3. 将处理器执行获取内容传递给处理器适配器
4. 处理器适配器将结果返回给中央调度器

在`DispatcherServlet`中有一个方法用于获取`HandlerAdapter`
``` java

```













































































































































































