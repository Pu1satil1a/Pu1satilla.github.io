---
title: SpringMVC(五)
description: springMVC核心技术
categories:
 - Web
 - Java
 - SpringMVC
tags: [Web, Java, SpringMVC]
---

# 请求转发和重定向

转发：跳转到页面或者跳转到其他处理器。

重定向：重定向相当于客户端再次发送请求，不能访问WEB-INF内部资源。

## 返回ModelAndView转发
可以不显式指出，若需要显式指出则需要在`setViewName()`指定的视图前添加forward，且此时的视图不会再与视图解析器中的前缀与后缀进行拼接。

### 请求转发到页面
当通过请求转发跳转到目标资源时，若需要向下传递数据，处理可以使用request，session外，还可以将数据存放于ModelAndView中的Model中。页面通过El表达式可直接访问该数据。

通过添加`forward:`标记表明了为请求转发，即使配置了视图解析器无法作用于当前路径。
``` java
@RequestMapping(value = "firstRequestFormByPerson.do", method = RequestMethod.POST)
public ModelAndView firstRequestFormByPerson(Person person, HttpServletRequest request) {

	ModelAndView mv = new ModelAndView();

	mv.addObject("person", person);
	mv.setViewName("forward:/WEB-INF/acceptRequest.jsp");

	return mv;
}
```

### 请求转发到处理器
``` java
@RequestMapping(value = "firstRequestForm.do", method = RequestMethod.POST)
public ModelAndView firstRequestForm(@RequestParam(name = "pname") String name,
									 @RequestParam(name = "page") int age)
		throws Exception {

	ModelAndView mv = new ModelAndView();

	mv.addObject("name", name);
	mv.addObject("age", age);
	mv.setViewName("forward:firstRequestFormByPerson.do");
	return mv;
}

@RequestMapping(value = "firstRequestFormByPerson.do", method = RequestMethod.POST)
public ModelAndView firstRequestFormByPerson(Person person, HttpServletRequest request) {

	ModelAndView mv = new ModelAndView();

	mv.addObject("person", person);
	mv.setViewName("forward:/WEB-INF/acceptRequest.jsp");

	return mv;
}
```

## 返回ModelAndView重定向
返回`ModelAndView`时的重定向，需在`setViewName()`指定的视图前添加`redirect:`，且此时的视图不会再与视图解析器中的前辍与后辍进行拼接。即必须写出相对于项目根的路径。故此时的视图解析器不再需要前辍与后辍。

### 重定向到页面
在重定向时，请求参数时无法通过request属性向下一级资源传递的。但可以通过以下方式将请求参数向下传递。

#### ModelAndView携带参数
当`ModelAndView`中的`Model`存入数据后，视图解析器`InternalResourceViewResolver`会将map中的key与value，以请求参数的形式放到请求的URL后。

但需要注意：
1. 由于视图解析器会将Map的value放入到URL后作为请求参数传递出去，所以无论什么类型的value，均会变为String。故此，放入到Model中的value，只能是基本数据类型与String，不能是自定义类型的对象数据。
2. 重定向的页面中是无法从request中读取数据的。但由于map中的key与value，以请求参数的形式放到了请求的URL后，所以，页面可以通过EL表达式中的请求参数param读取。
3. 重定向的页面不能是/WEB-INF中的页面。

#### HttpSession携带参数
类似于Servlet中session用法，将模型存储session中转发给jsp文件，可以从jsp文件的session域中获取内容。

### 重定向到处理器
重定向到其它处理器时，若要携带参数，完全可以采用前面的方式。而对于目标Controller接收这些参数，则各不相同。

#### ModelAndView携带参数
目标Controller在接收这些参数时，只要保证目标Controller的方法形参名称与发送Controller发送的参数名称相同即可接收。当然，目标Controller也可以进行参数的整体接收。只要保证参数名称与目标Controller接收参数类型的属性名相同即可。

``` java
@RequestMapping("/redirect/")
@Controller
public class Redirection {

    @RequestMapping("send.do")
    public ModelAndView sendMessage() {
        ModelAndView mv = new ModelAndView();

        mv.addObject("message", "重定向到处理器");
        mv.setViewName("redirect:accept.do");
        return mv;
    }

    @RequestMapping("accept.do")
    public ModelAndView acceptMessage(String message) {
        ModelAndView mv = new ModelAndView();

        mv.addObject("message", message);
        mv.setViewName("forward:/redirect.jsp");
        return mv;
    }
}
```
重定向之后转发的内容显示在jsp上

``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${message}
</body>
</html>
```
重定向不能发送实体对象，但是可以接收实体对象，只需要将实体属性名一一对应进行传递，通过一个实体进行接收。

#### HttpSession携带参数
通过传递session对象重定向给另一个处理器
``` java
@RequestMapping("sendBySession.do")
public ModelAndView sendMessageBySession(HttpSession httpSession) {

	ModelAndView mv = new ModelAndView();
	httpSession.setAttribute("message", "重定向到处理器携带session");

	mv.setViewName("redirect:acceptBySession.do");

	return mv;
}

@RequestMapping("acceptBySession.do")
public ModelAndView acceptMessageBySession(HttpSession httpSession) {

	ModelAndView mv = new ModelAndView();
	String session = (String) httpSession.getAttribute("message");
	System.out.println(session);

	mv.addObject("sessin", session);
	mv.setViewName("forward:/redirect.jsp");

	return mv;
}
```

视图显示
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

    ${sessin}
</body>
</html>
```

## 返回String的请求转发
当处理器方法返回String时，该String即为要跳转的视图。**在其前面加上前辍forward:，则可显式的指定跳转方式为请求转发。同样，视图解析器不会对其进行前辍与后辍的拼接。**请求转发的目标资源无论是一个页面，还是一个Controller，用法相同。

注意，此时不能再使用ModelAndView传递数据了。因为在处理器方法中定义的ModelAndView对象就是个局部变量，方法运行结束，变量销毁。而当前的处理器方法返回的为String，而非ModelAndView，所以这个创建的ModelAndView就不起作用了。不过，需要注意的是，对于处理器方法返回字符串的情况，当处理器接收到请求中的参数后，发现用于接收这些参数的处理器方法形参为包装类对象，则除了会将参数封装为对象传递给形参外，还会存放到request域属性中。

``` java
@Controller
public class Forward {

    @RequestMapping(value = "/forwardByString.do" ,method = RequestMethod.POST)
    public String forWardByString(Person person) {
		//    person被存储在了request域中
        return "forward:/forward_1.jsp";
    }
}
```
修改视图文件
``` html
${requestScope.person}
```

## 返回String时的重定向
在处理器方法返回的视图字符串的前面添加前辍`redirect:`，则可实现重定向跳转。当重定向到目标资源时，若需要向下传递参数值，除了可以直接通过请求URL携带参数，通过`HttpSession`携带参数外，还可使用其它方式。

### 重定向到页面

#### 形参Model
可以在Controller形参中添加Model参数，将要传递的数据放入Model中进行参数传递。**该方式同样也是将参数拼接到了重定向请求的URL后，所以放入其中的数据只能是基本类型数据，不能是自定义类型。**


``` java
@Controller
public class Forward {

    @RequestMapping(value = "/forwardByString.do", method = RequestMethod.POST)
    public String forWardByString(Person person, Model model) {
        model.addAttribute("pname", person.getPname());
        model.addAttribute("page", person.getPage());

        return "redirect:/forward_1.jsp";
    }
}
```

修改视图文件
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

${param.get("pname")}
${param.get("page")}
</body>
</html>
```

#### 形参RedirectAttributes
`RedirectAttributes`是`Spring3.1`版本后新增的功能，专门是用于携带重定向参数的。它是一个继承自Model的接口，其底层仍然使用`ModelMap`实现。通过`addAttribute()`方法会将参数名及参数值放入该Map中，然后视图解析器会将其拼接到请求的URL中。所以，这种携带参数的方式，不能携带自定义对象。

``` java
@Controller
public class Forward {

    @RequestMapping(value = "/forwardByString.do", method = RequestMethod.POST)
    public String forWardByString(Person person, RedirectAttributes attributes) {
        attributes.addAttribute("pname", person.getPname());
        attributes.addAttribute("page", person.getPage());

        return "redirect:/forward_1.jsp";
    }
}
```

### 重定向到Controller

重定向到Controller时，携带参数的方式，除了可以使用请求URL后携带方式，`HttpSession`携带方式，Model形参携带方式，及`RedirectAttributes`形参的`addAttibute()`携带方式外，还可以使用`RedirectAttributes`形参的`addFlushAttibute()`携带方式。

#### Model形参携带参数
可以在Controller形参中添加Model参数，将要传递的数据放入Model中进行参数传递。**该方式同样也是将参数拼接到了重定向请求的URL后，所以放入其中的数据只能是基本类型数据，不能是自定义类型**
``` java
@Controller
public class Forward {

    @RequestMapping(value = "/forwardByString.do", method = RequestMethod.POST)
    public String forWardByString(Person person, Model model) {

        model.addAttribute("pname", person.getPname());
        model.addAttribute("page", person.getPage());

        return "redirect:/acceptRedirectByString.do";
    }

    @RequestMapping(value = "/acceptRedirectByString.do", method = RequestMethod.GET)
    public String accRedirectByString(String pname, int page, Model model) {
        model.addAttribute("pname", pname);
        model.addAttribute("page", page);

        return "redirect:/forward_1.jsp";
    }
}
```

#### RedirectAttributes形参（了解）
`RedirectAttributes`形参的`addFlushAttibute()`携带方式不会将放入其中的属性值通过请求URL传递，所以其中可以存放任意对象。

## 返回void请求转发与重定向
与Servlet调用一致。

# 异常处理
常用的SpringMVC异常处理方式主要有三种：
- 使用系统定义好的异常处理器SimpleMappingExceptionResolver
- 使用自定义异常处理器
- 使用异常处理注解

## SimpleMappingExceptionResolver

该方式只需要在SpringMVC配置文件中注册该异常处理器Bean即可。该Bean比较特殊，没有id属性，无需显式调用或被注入给其它`<bean/>`，当异常发生时会自动执行该类。

### 自定义异常类
定义三个异常类：NameException、AgeException、PeopleException。其中PeopleException是另外两个异常的父类。
``` java
public class PeopleException extends Exception{
    public PeopleException() {
    }

    public PeopleException(String message) {
        super(message);
    }
}

public class NameException extends PeopleException{
    public NameException() {
    }

    public NameException(String message) {
        super(message);
    }
}

public class AgeException extends PeopleException{
    public AgeException() {
    }

    public AgeException(String message) {
        super(message);
    }
}
```

### 修改Controller
``` java
@Controller
public class ExceptionController {
    @RequestMapping(value = "/login.do", method = RequestMethod.POST)

    public ModelAndView SimpleExceptionResolver(String name, int age) throws PeopleException {

        ModelAndView mv = new ModelAndView();

        if (!name.equals("翠花")) {
            throw new NameException("登录名" + name + "错误");
        }
        if (age > 50 || age < 10) {
            throw new AgeException("年龄不符");
        }

        Person person = new Person();
        person.setPname(name);
        person.setPage(age);
        mv.addObject("person", person);

        mv.setViewName("forward:/exceptionPage/success.jsp");
        return mv;
    }
}
```

### 注册异常处理器

``` xml
<!--注册异常类-->
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">

	<!--
		属性exceptionMappings：用于指定具体的不同类型的异常所对应的异常响应页面。
			key为异常类的全限定性类名，value则为响应页面路径

		属性defaultErrorView：对于没有在属性exceptionMapping中指定异常对应页面的
							 通过该属性指定默认跳转页面

		属性exceptionAttribute：捕获到的异常对象
							   一般异常响应页面中使用
	-->
	<property name="exceptionMappings">
		<props>
			<prop key="cn.pu1satilla.Exception.NameException">/exceptionPage/nameError.jsp</prop>
			<prop key="cn.pu1satilla.Exception.AgeException">/exceptionPage/ageError.jsp</prop>
		</props>
	</property>

	<property name="defaultErrorView" value="/exceptionPage/error.jsp"/>
	<property name="exceptionAttribute" value="ex"/>
</bean>
```
- exceptionMapping: 用于指定具体的不同类型的异常所对应的异常响应页面。Key为异常类的全限定性类名，value则为响应页面路径
- defaultErrorView: 指定默认的异常响应页面。若发生的异常不是`exceptionMappings`中指定的异常，则使用默认异常响应页面。
- exceptionAttribute：捕获到的异常对象。一般异常响应页面中使用。

### 定义异常响应页面
分别建立配置文件中路径文件
``` html
<body>
    ${ex.message}
</body>
```

## 自定义异常处理器
使用`SpringMVC`定义好的`SimpleMappingExceptionResolver`异常处理器，可以实现发生指定异常后的跳转。但若要实现在捕获到指定异常时，执行一些操作的目的，它是完成不了的。此时，就需要自定义异常处理器。自定义异常处理器，需要实现`HandlerExceptionResolver`接口，并且该类需要在`SpringMVC`配置文件中进行注册。

### 定义异常处理器
当一个类实现了`HandlerExceptionResolver`接口后，只要有异常发生，无论什么异常，都会自动执行接口方法`resolveException()`。

``` java
public class PersonExceptionResolver implements HandlerExceptionResolver {


    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

        ModelAndView mv = new ModelAndView();

        //        1.将异常数据加入模型
        mv.addObject("ex", ex);

        //        2.设置默认错误响应页面
        mv.setViewName("/exceptionPage/error.jsp");

        //        3.如果异常对象是NameException，转发到nameException页面
        if (ex instanceof NameException) {
            mv.setViewName("/exceptionPage/nameException.jsp");
        }

        //        4.如果异常对象是ageException，转发到ageException页面
        if (ex instanceof AgeException) {
            mv.setViewName("/exceptionPage/ageException.jsp");
        }
        return mv;
    }
}
```

### 注册异常处理器
``` xml
<!--注册异常处理器-->
<bean class="cn.pu1satilla.controller.PersonExceptionResolver"/>
```

## 异常处理注解

使用注解`@ExceptionHandler`可以将一个方法指定为异常处理方法。
``` java
@Controller
public class ExceptionController {
    @RequestMapping(value = "/login.do", method = RequestMethod.POST)

    public ModelAndView SimpleExceptionResolver(String name, int age) throws PeopleException {

        ModelAndView mv = new ModelAndView();

        if (!name.equals("翠花")) {
            throw new NameException("登录名" + name + "错误");
        }
        if (age > 50 || age < 10) {
            throw new AgeException("年龄不符");
        }

        Person person = new Person();
        person.setPname(name);
        person.setPage(age);
        mv.addObject("person", person);

        mv.setViewName("forward:/exceptionPage/success.jsp");
        return mv;
    }

    @ExceptionHandler(PeopleException.class)
    public ModelAndView handlePeopleException(Exception ex) {
        ModelAndView modelAndView = new ModelAndView();

        modelAndView.addObject("ex", ex);
        modelAndView.setViewName("/exceptionPage/error.jsp");

        return modelAndView;
    }
}
```

不过，一般不这样使用。而是将异常处理方法专门定义在一个Controller中，让其它Controller继承该Controller即可。但是，这种用法的弊端也很明显：Java是“单继承多实现”的，这个Controller的继承将这唯一的一个继承机会使用了，使得若再有其它类需要继承，将无法直接实现。

# 类型转换器
在前面的程序中，表单提交的无论是int还是double类型的请求参数，用于处理该请求的处理器方法的形参，均可直接接收到相应类型的相应数据，而非接收到String再手工转换。那是因为在SpringMVC框架中，有默认的类型转换器。这些默认的类型转换器，可以将String类型的数据，自动转换为相应类型的数据。

但默认类型转换器并不是可以将用户提交的String，转换为所有用户需要的类型。此时，就需要自定义类型转换器了。例如，在SpringMVC的默认类型转换器中，没有日期类型的转换器，因为日期的格式太多。若要使表单中提交的日期字符串，被处理器方法形参直接接收为`java.util.Date`，则需要自定义类型转换器了。

## 搭建测试环境

注册页面
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="" method="post">
    用户名：<input type="text" name="name"> <br>
    生日：<input type="text" name="birthday"><br>
    <input type="submit" value="注册">
</form>
</body>
</html>

```

控制器
``` java
@Controller
public class RegisterController {
    @RequestMapping("/register.do")
    public ModelAndView registerConverter(String name, Date date) {

        ModelAndView modelAndView = new ModelAndView();

        modelAndView.addObject("name", name);
        modelAndView.addObject("date", date);
        modelAndView.setViewName("/converter/success.jsp");

        return modelAndView;
    }
}
```

转发页面
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${name} <br>
${birthday}
</body>
</html>
```

## 自定义类型转换器
``` java
public class DateConverter implements Converter<String, Date> {

    @Override
    public Date convert(String source) {

        Date date = new Date();

        //        判断传入参数值不为空才进行转换
        if (source != null && !source.equals("")) {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
            try {
                date = dateFormat.parse(source);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
        return date;
    }
}
```

## 配置类型转换器
类型转换器定义完毕后，需要在SpringMVC的配置文件中对类型转换进行配置。首先要注册类型转换器，然后再注册一个转换服务Bean。将类型转换器注入给该转换服务Bean。最后由处理器适配器来使用该转换服务Bean。

``` xml
<!--注册异常处理器-->
<!--<bean class="cn.pu1satilla.controller.PersonExceptionResolver"/>-->

<!--注册类型转换器-->
<bean class="cn.pu1satilla.converter.DateConverter" id="dateConverter"/>

<!--注册类型转换服务-->
<bean class="org.springframework.context.support.ConversionServiceFactoryBean" id="conversionService">
	<!--注入属性：单个类型转换器写法-->
	<property name="converters" ref="dateConverter"/>

	<!--当需要注入多个类型转换器：
		<property name="converters">
			<set>
				<ref bean="dateConverter"/>
			</set>
		</property>
	-->
</bean>

<!--注册mvc注解驱动-->
<mvc:annotation-driven conversion-service="conversionService"/>
```

## 配置多种日期类型转换器
使用正则表达式对传入source资源进行判断为哪种日期格式，不同的日期格式对应不同`SimpleDataFormat`对象。
``` java
public class DateConverter implements Converter<String, Date> {

    @Override
    public Date convert(String source) {

        Date date = new Date();

        //        判断传入参数值不为空才进行转换
        if (source != null && !source.equals("")) {
            SimpleDateFormat dateFormat = getSimpleDateFormat(source);
            try {
                date = dateFormat.parse(source);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }

        return date;
    }

    /**
     * 传入日期格式，对日期格式进行判断，符合则建该日期格式的SimpleDateFormat对象
     *
     * @param source 传入日期格式
     * @return SimpleDateFormat对象
     */
    private SimpleDateFormat getSimpleDateFormat(String source) {

        SimpleDateFormat dateFormat = null;
        if (Pattern.matches("^\\d{4}-\\d{2}-\\d{2}$", source)) {
            dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        } else if (Pattern.matches("^\\d{4}/\\d{2}/\\d{2}$", source)) {
            dateFormat = new SimpleDateFormat("yyyy/MM/dd");
        } else if (Pattern.matches("^\\d{4}\\d{2}\\d{2}$", source)) {
            dateFormat = new SimpleDateFormat("yyyy/MM/dd");
        }
        return dateFormat;
    }
}
```

## 数据回显
当数据转换出现异常，表示用户输入格式有误，这时需要数据回显给用户进行重新输入，可以通过异常进行跳转完成。

### 转换器
当输入格式匹配不上，转换器抛出`TypeMismatchException`
``` java
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {

        Date date = new Date();

        //        判断传入参数值不为空才进行转换
        if (source != null && !source.equals("")) {
            SimpleDateFormat dateFormat = getSimpleDateFormat(source);
            try {
                date = dateFormat.parse(source);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }

        return date;
    }
	
    /**
     * 传入日期格式，对日期格式进行判断，符合则建该日期格式的SimpleDateFormat对象
     *
     * @param source 传入日期格式
     * @return SimpleDateFormat对象
     */
    private SimpleDateFormat getSimpleDateFormat(String source) {

        SimpleDateFormat dateFormat = null;

        if (Pattern.matches("^\\d{4}-\\d{2}-\\d{2}$", source)) {
            dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        } else if (Pattern.matches("^\\d{4}/\\d{2}/\\d{2}$", source)) {
            dateFormat = new SimpleDateFormat("yyyy/MM/dd");
        } else if (Pattern.matches("^\\d{4}\\d{2}\\d{2}$", source)) {
            dateFormat = new SimpleDateFormat("yyyy/MM/dd");
        } else {
            throw new TypeMismatchException("输入日期格式有误", Date.class);
        }

        return dateFormat;
    }
}
```

### 处理器
在处理器内通过注解指定异常处理
``` java
@Controller
public class RegisterController {
    @RequestMapping("/register.do")
    public ModelAndView registerConverter(String name, Date birthday) {

        ModelAndView modelAndView = new ModelAndView();

        modelAndView.addObject("name", name);
        modelAndView.addObject("birthday", birthday);

        modelAndView.setViewName("/converter/success.jsp");
        return modelAndView;
    }

    @ExceptionHandler(TypeMismatchException.class)
    public ModelAndView handleTypeMismatchException(Exception ex) {
        ModelAndView modelAndView = new ModelAndView();

        modelAndView.addObject("ex", ex);
        modelAndView.setViewName("/converter/regsiter.jsp");
        return modelAndView;
    }
}
```

### 测试
打开`localhost:8080/register.do`链接，发现异常信息阅读困难
![](/assets/images/SpringMVC/converter.png)

需要对代码进行改进

### 改进
在异常解析器修改传递参数为方便用户阅读中文：
``` java
@Controller
public class RegisterController {
    @RequestMapping("/register.do")
    public ModelAndView registerConverter(String name, Date birthday) {

        ModelAndView modelAndView = new ModelAndView();

        modelAndView.addObject("name", name);
        modelAndView.addObject("birthday", birthday);

        modelAndView.setViewName("/converter/success.jsp");
        return modelAndView;
    }

    @ExceptionHandler(TypeMismatchException.class)
    public ModelAndView handleTypeMismatchException(Exception ex) {
        ModelAndView modelAndView = new ModelAndView();

        //        对异常信息进行判断
        if (ex.getMessage().contains("ConversionFailedException")) {
            modelAndView.addObject("ex", "日期格式转换失败，请重新输入！！！");
        }
        modelAndView.setViewName("/converter/regsiter.jsp");
        return modelAndView;
    }
}
```

# 数据验证

为了防止用户传入数据不正确导致多方面问题，需要对数据进行验证。输入验证分为客户端验证和服务器端验证，客户端主要通过`JavaScript`进行，而服务器端则主要通过Java代码进行验证。

## 搭建测试环境

### 导入jar包
![](/assets/images/SpringMVC/validator.png)
[下载地址](/source/springmvc_validator_source)

### 在实体上添加验证注解
使用的验证器注解均为`javax.validation.constraints`包中的类。在注解的message属性中，可以使用`{属性名}`的方式来引用指定的注解的属性值。
``` java
public class Person {
    @NotEmpty(message = "姓名不能为空")
    @Size(min = 3, max = 6, message = "姓名长度必须在{min}-{max}之间")
    private String pname;

    @Min(value = 0, message = "年龄不能小于{value}")
    @Max(value = 100, message = "年龄不能大于{value}")
    @NotEmpty(message = "年龄不能为空")
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
```
`Hibernate Validator`中常用的验证注解介绍：

序号|验证注解|说明
---- | ---- | ---- | ----
1|@AssertFalse|验证注解的元素值是false
2|@AssertTrue|验证注解的元素值是true
3|@DecimalMax（value=x）|验证注解的元素值小于等于指定的十进制value值
4|@DecimalMin（value=x）|验证注解的元素值大于等于指定的十进制value值
5|@Digits(integer=整数位数, fraction=小数位数)|验证注解的元素值的整数位数和小数位数上限
6|@Future|验证注解的元素值（日期类型）比当前时间晚
7|@Max（value=x）|验证注解的元素值小于等于指定的value值
8|@Min（value=x）|验证注解的元素值大于等于指定的value值
9|@NotNull|验证注解的元素值丌是null
10|@Null|验证注解的元素值是null
11|@Past|验证注解的元素值（日期类型）比当前时间早
12|@Pattern(regex=正则表达式)|验证注解的元素值不指定的正则表达式匹配
13|@Size(min=最小值, max=最大值)|验证注解的元素值的在min和max（包含）指定区间之内，如字符长度、集合大小
14|@Valid|验证关联的对象，如账户对象里有一个订单对象，指定验证订单对象
15|@NotEmpty|验证注解的元素值丌为null且丌为空（字符串长度丌为0、集合大小丌为0）
16|@Range(min=最小值, max=最大值)|验证注解的元素值在最小值和最大值之间
17|@NotBlank|验证注解的元素值丌为空（丌为null、去除首位空格后长度为0），丌同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的空格
18|@Length(min=下限, max=上限)|验证注解的元素值长度在min和max区间内
19|@Email|验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式

### 控制器
当验证出现问题需要将错误信息携带到注册页面进行显示，要求重新输入，没有问题则顺利通过。
``` java
@Controller
public class RegisterValidatorController {

    @RequestMapping("/validator/register.do")
    public ModelAndView registerValidator(@Validated Person person, BindingResult bindingResult) {

        ModelAndView modelAndView = new ModelAndView();

        //        如果验证错误结果个数>0，传入验证结果，并且返回注册页面，重新注册
        List<ObjectError> allErrors = bindingResult.getAllErrors();

        if (allErrors.size() > 0) {
            FieldError pname = bindingResult.getFieldError("pname");
            FieldError page = bindingResult.getFieldError("page");

            if (pname != null) {
                //                传入名称错误信息到模型
                modelAndView.addObject("pnameError", pname.getDefaultMessage());
            }
            if (page != null) {
                //                传入年龄错误信息到模型
                modelAndView.addObject("pageError", page.getDefaultMessage());
            }
            modelAndView.setViewName("/validator/register.jsp");
        } else {
            //        否则跳到成功页面
            modelAndView.addObject("person", person);
            modelAndView.setViewName("/validator/success.jsp");
        }
        return modelAndView;
    }
}
```

### 前台文件
新建一个jsp页面，含有一个提交的表单，具有name跟age两项。
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="${pageContext.request.contextPath}/validator/register.do">

    姓 名：<input type="text" name="pname">${pnameError}<br>
    年 龄：<input type="text" name="page">${pageError}<br>
    学校 名： <input type="text" name="school.sname"><br>
    学校地址：<input type="text" name="school.slocation"><br>
    <input type="submit" value="注册">

</form>
</body>
</html>
```

# 文件上传

## 导入jar包
[jar包下载地址](/source/upLoad_resource)

## 定义上传页面
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%--必须指定enctype类型，不然spring无法识别--%>
<form action="/upLoad/upLoad.do" method="post" enctype="multipart/form-data">
    照片：<input type="file" name="photo"/> <br>
    <input type="submit" value="上传">
</form>
</body>
</html>
```
**注意：enctype类型必须指定，否则spring无法识别请求是否需要上传文件**

## 定义处理器
``` java
@Controller
@RequestMapping("/upLoad")
public class UpLoadController {

    /**
     * 演示spring上传文件，以图片格式文件作为上传例子
     *
     * @param photo 上传文件参数
     * @param request
     * @return
     */
    @RequestMapping(value = "/upLoad.do", method = RequestMethod.POST)
    public ModelAndView upLoad(MultipartFile photo, HttpServletRequest request) throws IOException {

        //        获取文件原始名称
        String photoName = photo.getOriginalFilename();
        //        获取存储路径
        String realPath = request.getServletContext().getRealPath("/images");
        //        判断文件是否为图片（.jpg|.png）格式
        if (photoName.endsWith(".jpg") || photoName.endsWith(".png")) {
            File file = new File(realPath, photoName);
            photo.transferTo(file);
        } else {
            return new ModelAndView("/upLoad/fail.jsp");
        }
        return new ModelAndView("/upLoad/success.jsp");
    }
}
```

## 配置文件类型处理器
``` java
<!--注册组件扫描器-->
<context:component-scan base-package="cn.pu1satilla.*"/>

<!--注册springmvc注解驱动-->
<mvc:annotation-driven/>

<!--
	注册multipart解析器，这个id名字固定，是由DispatcherServlet直接调用的
		属性：defaultEncoding ========= 默认编码格式
		maxUploadSizePerFile ========= 每个上传文件大小
-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="defaultEncoding" value="utf-8"/>
	<property name="maxUploadSizePerFile" value="1048576"/>
</bean>
```

## 出现的问题
- 在文件夹为空时，idea不能发布空文件到到项目包里，可以通过在空文件夹内新建一个无用文件解决。

# 拦截器
























































































































































































