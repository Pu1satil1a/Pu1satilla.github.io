---
title: Spring(四)
description: Spring的Aop初识
categories:
 - Web
 - Java
 - Spring
tags: [Web, Java, Spring]
---



# Spring的AOP概念

## AOP思想
`AOP（Aspect-OrientedProgramming，面向方面编程）`，可以说是OOP`（Object-Oriented Programing，面向对象编程）`的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当我们需要为分散的对象引入公共行为的时候，OOP则显得无能为力。也就是说，OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如日志功能。日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。对于其他类型的代码，如安全性、异常处理和透明的持续性也是如此。这种散布在各处的无关的代码被称为横切`（cross-cutting）`代码，在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

而AOP技术则恰恰相反，它利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手将这些剖开的切面复原，不留痕迹。

## 代理机制

spring的AOP的底层用到两种代理机制：
- JDK的动态代理：针对实现了接口的类产生代理
- Cglib的动态代理：针对没有实现接口的类产生代理，应用的是底层的字节码增强技术生成当前类的子类对象

Java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

## JDK动态代理
JDK动态代理只能对实现了接口的类生成代理，而不能针对类。

**接口**
``` java
package cn.Pu1satilla.JDKproxy;

public interface People {
    void speak();
    void eat(String something);
}
```

**接口实现类**
``` java
package cn.Pu1satilla.JDKproxy;

public class Actor implements People {

    @Override
    public void speak() {
        System.out.println("绿人");
    }

    @Override
    public void eat(String something) {
        System.out.println("吃双黄蛋" + something);
    }
}
```

**JDK代理类**
``` java
package cn.Pu1satilla.JDKproxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class JdkProxy implements InvocationHandler {

    //    生成需要代理的目标对象
    private Object neededProxy;

    //    获取当前目标代理对象
    public Object getCurrentProxy(Object currentProxy) {
        neededProxy = currentProxy;
        return Proxy.newProxyInstance(
                neededProxy.getClass().getClassLoader(),
                neededProxy.getClass().getInterfaces(),
                this
        );
    }

    /**
     * 强化方法
     *
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.getName().equals("eat")) {
            System.out.println("这里运行强化功能！");
            return method.invoke(neededProxy, Arrays.toString(args));
        } else
            return method.invoke(neededProxy, args);
    }
}
```

## Cglib动态代理
Cglib采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势植入横切逻辑。JDK动态代理与Cglib动态代理军事实现Spring AOP的基础。

CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，因为是继承，所以该类或方法最好不要声明成final 。

**Cglib代理**
``` java
package cn.Pu1satilla.proxy_demo;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibProxy implements MethodInterceptor {

    //    定义存放需要Cglib代理的对象
    private Object neededProxy;

    //    传入需要代理的对象
    public CglibProxy(Object currentProxy) {
        neededProxy = currentProxy;
    }

    public Object getCurrentProxy() {

        //        创建Cglib核心类
        Enhancer enhancer = new Enhancer();

        //        设置父类
        enhancer.setSuperclass(neededProxy.getClass());

        //        设置回调
        enhancer.setCallback(this);

        //        生成代理
        return enhancer.create();
    }

    /**
     * 截取方法，强化方法
     *
     * @param o
     * @param method
     * @param objects
     * @param methodProxy
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("强化方法");
        return method.invoke(neededProxy, objects);
    }
}
```

**测试类**
``` java
package cn.Pu1satilla.proxy_demo;

public class Demo {
    public static void main(String[] args) {

        //        获得JDK代理对象
        People actorProxy = (People) new JdkProxy().getCurrentProxy(new Actor());

        //        调用代理对象方法
        actorProxy.eat("香肠");

        //        获得Cglib代理对象
        CglibProxy cglibProxy = new CglibProxy(new Actor());
        People actorCglibProxy = (People) cglibProxy.getCurrentProxy();

        //        调用代理对象方法
        actorCglibProxy.speak();
    }
}

```
**测试结果**
```
这里运行强化功能！
吃双黄蛋[香肠]
强化方法
绿人
```

## aop名词学习
```
Joinpoint（连接点）：目标对象中，所有可以增强的方法
Pointcut（切入点）：目标对象，增强的方法
Advice（通知/增强）：增强的代码
Target（目标对象）：被代理对象
Weaving（织入）：将通知应用到切入点的过程
Proxy（代理）：将通知植入到目标对象之后，形成代理对象
aspect（切面）：切入点+通知
```

# Aop配置以及应用

## AOP配置（XML）

### 导包
![](/assets/images/Spring/aop_configuration.PNG)

### 目标对象类
``` java
package cn.Pu1satilla.AOP;

import org.springframework.stereotype.Component;

@Component("cat")

public class Cat implements Animal {

    @Override
    public void speak() {
        System.out.println("喵喵喵");
    }

    @Override
    public void eat(String animal) {
        System.out.println("猫吃" + animal);
    }

    @Override
    public void run() {
        System.out.println("猫追耗子");
    }

    @Override
    public void shit() {
        System.out.println("猫也拉屎");
        int i = 1 / 0;
    }
}
```

### 通知类
``` xml
package cn.Pu1satilla.AOP;

import org.aspectj.lang.ProceedingJoinPoint;

public class AopAdvice {

    //    前置通知
    public void before(){
         System.out.println("前置增强=============");
    }

    //    后置通知
    public void afterReturning(){
        System.out.println("运行后增强=============（出现异常不会调用）");
    }

    //    环绕通知
    public Object around(ProceedingJoinPoint pjd) throws Throwable {

        System.out.println("环绕增强前部分=============");
        Object proceed = pjd.proceed();
        System.out.println("环绕增强后部分=============");
        return proceed;
    }

    //    异常通知
    public void afterException() {
        System.out.println("出现异常=============");
    }

    //    后置通知
    public void after(){
        System.out.println("运行后增强=============（出现异常也会调用）");
    }
}
```

### AOP表达式(配置)

``` xml
execution(* com.sample.service.impl..*.*(..))
```

对应符号解释：

符号 | 含义
---- | ----
execution()|表达式主体
第一天“*”符号|标识返回值的类型任意
com.sample.service.impl 	|AOP所切的服务的包名，即，我们的业务部分
包名后面的”..“ 	|表示当前包及子包
第二个”*“ 	|表示类名，*即所有类。此处可以自定义，下文有举例
.*(..) 	|表示任何方法名，括号表示参数，两个点表示任何参数类型

### 配置

**设置使用Cglib作为spring的aop代理**
``` xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

**配置通知对象（增强方法的对象）**
``` xml
<!--配置通知对象-->
<bean name="AopAdvice" class="cn.Pu1satilla.AOP.AopAdvice"/>
```

**AOP切面配置**
``` xml
<!--配置切面-->
<aop:aspect ref="AopAdvice">
	<!--将名为AopAdvice的前置通知织入到名为strength的切入点-->
	<aop:before method="before" pointcut-ref="strength"/>
	<!--将名为AopAdvice的环绕通知织入到名为strength的切入点-->
	<aop:around method="around" pointcut-ref="strength"/>
	<!--将名为AopAdvice的后置通知（出现异常不会调用）织入到名为strength的切入点-->
	<aop:after-returning method="afterReturning" pointcut-ref="strength"/>
	<!--将名为AopAdvice的异常通知织入到名为strength的切入点-->
	<aop:after-throwing method="afterException" pointcut-ref="strength"/>
	<!--将名为AopAdvice的后置通知织入到名为strength的切入点-->
	<aop:after method="after" pointcut-ref="strength"/>
</aop:aspect>
```

**整合**
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--activities annotation-->
    <context:annotation-config/>

    <!--扫描指定包-->
    <context:component-scan base-package="cn.Pu1satilla.AOP cn.Pu1satilla.domain"/>

    <!--配置通知对象-->
    <bean name="AopAdvice" class="cn.Pu1satilla.AOP.AopAdvice"/>

    <!--指定spring的代理为Cglib-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>

    <!--进行aop配置-->
    <aop:config>

        <!--配置切入点表达式：哪些类的哪些方法需要进行增强-->
        <aop:pointcut id="strength" expression="execution(* cn.Pu1satilla.AOP.Cat.*(..))"/>

        <!--配置切面-->
        <aop:aspect ref="AopAdvice">
            <!--将名为AopAdvice的前置通知织入到名为strength的切入点-->
            <aop:before method="before" pointcut-ref="strength"/>
            <!--将名为AopAdvice的环绕通知织入到名为strength的切入点-->
            <aop:around method="around" pointcut-ref="strength"/>
            <!--将名为AopAdvice的后置通知（出现异常不会调用）织入到名为strength的切入点-->
            <aop:after-returning method="afterReturning" pointcut-ref="strength"/>
            <!--将名为AopAdvice的异常通知织入到名为strength的切入点-->
            <aop:after-throwing method="afterException" pointcut-ref="strength"/>
            <!--将名为AopAdvice的后置通知织入到名为strength的切入点-->
            <aop:after method="after" pointcut-ref="strength"/>
        </aop:aspect>
    </aop:config>
</beans>
```

### 测试类
``` java
package cn.Pu1satilla.AOP;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;

//由注解创建容器
@RunWith(SpringJUnit4ClassRunner.class)

//指定创建容器时使用配置文件
@ContextConfiguration("classpath:applicationContext.xml")

public class Demo {

    @Resource(name = "cat")
    private Cat cat;

    @Test
    public void demo1(){
        cat.shit();
    }
}
```

## 注解配置

### 导包
![](/assets/images/Spring/aop_configuration.PNG)

### 目标对象类
``` java
package cn.Pu1satilla.AOP;

import org.springframework.stereotype.Component;

@Component("cat")

public class Cat implements Animal {

    @Override
    public void speak() {
        System.out.println("喵喵喵");
    }

    @Override
    public void eat(String animal) {
        System.out.println("猫吃" + animal);
    }

    @Override
    public void run() {
        System.out.println("猫追耗子");
    }

    @Override
    public void shit() {
        System.out.println("猫也拉屎");
        int i = 1 / 0;
    }
}
```

### 通知类
``` xml
package cn.Pu1satilla.AOP;

import org.aspectj.lang.ProceedingJoinPoint;

public class AopAdvice {

    //    前置通知
    public void before(){
         System.out.println("前置增强=============");
    }

    //    后置通知
    public void afterReturning(){
        System.out.println("运行后增强=============（出现异常不会调用）");
    }

    //    环绕通知
    public Object around(ProceedingJoinPoint pjd) throws Throwable {

        System.out.println("环绕增强前部分=============");
        Object proceed = pjd.proceed();
        System.out.println("环绕增强后部分=============");
        return proceed;
    }

    //    异常通知
    public void afterException() {
        System.out.println("出现异常=============");
    }

    //    后置通知
    public void after(){
        System.out.println("运行后增强=============（出现异常也会调用）");
    }
}
```

### 注解相关

**表明该类是通知类,注解配置于类上方**
``` java
@Aspect
public class AopAdvice {}
```

**配置切入点表达式：哪些类的哪些方法需要在目标对象哪些位置进行增强**

**前置通知**
``` java
//    前置通知
@Before("execution(* cn.Pu1satilla.AOP.Cat.*(..))")
public void before(){
	 System.out.println("前置增强=============");
}
```

**后置通知（出现异常不会调用）**
``` java
//    后置通知
@AfterReturning("execution(* cn.Pu1satilla.AOP.Cat.*(..))")
public void afterReturning(){
	System.out.println("运行后增强=============（出现异常不会调用）");
}
```

**环绕通知**
``` java
//    环绕通知
@Around("execution(* cn.Pu1satilla.AOP.Cat.speak())")
public Object around(ProceedingJoinPoint pjd) throws Throwable {

	System.out.println("环绕增强前部分=============");
	Object proceed = pjd.proceed();
	System.out.println("环绕增强后部分=============");
	return proceed;
}
```

**异常通知**
``` java
//    异常通知
@AfterThrowing("execution(* cn.Pu1satilla.AOP.Cat.shit())")
public void afterException() {
	System.out.println("出现异常=============");
}
```

**后置通知（出现异常也会调用）**
``` java
//    后置通知
@After("execution(* cn.Pu1satilla.AOP.Cat.*())")
public void after(){
	System.out.println("运行后增强=============（出现异常也会调用）");
}
```

**整合**
``` java
package cn.Pu1satilla.AOP;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

//@Aspect //表明该类是一个通知类
public class AopAdvice {

    //    前置通知
    @Before("execution(* cn.Pu1satilla.AOP.Cat.*(..))")
    public void before(){
         System.out.println("前置增强=============");
    }

    //    后置通知
    @AfterReturning("execution(* cn.Pu1satilla.AOP.Cat.*(..))")
    public void afterReturning(){
        System.out.println("运行后增强=============（出现异常不会调用）");
    }

    //    环绕通知
    @Around("execution(* cn.Pu1satilla.AOP.Cat.speak())")
    public Object around(ProceedingJoinPoint pjd) throws Throwable {

        System.out.println("环绕增强前部分=============");
        Object proceed = pjd.proceed();
        System.out.println("环绕增强后部分=============");
        return proceed;
    }

    //    异常通知
    @AfterThrowing("execution(* cn.Pu1satilla.AOP.Cat.shit())")
    public void afterException() {
        System.out.println("出现异常=============");
    }

    //    后置通知
    @After("execution(* cn.Pu1satilla.AOP.Cat.*())")
    public void after(){
        System.out.println("运行后增强=============（出现异常也会调用）");
    }
}
```





