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















































































































































































































































