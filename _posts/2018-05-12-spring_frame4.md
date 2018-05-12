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
Weaving（植入）：将通知应用到切入点的过程
Proxy（代理）：将通知植入到目标对象之后，形成代理对象
aspect（切面）：切入点+通知
```

# Aop配置以及应用

## AOP配置

### AOP表达式

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











































