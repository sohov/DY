>     基于最新Spring 5.x，详细介绍了[AOP](https://so.csdn.net/so/search?q=AOP&spm=1001.2101.3001.7020)的概念以及基于XML的核心Spring AOP机制的配置和使用。

  **本次我们介绍AOP的概念以及基于XML的核心Spring AOP机制的配置和使用。包括Spring AOP的开启，< aop:config >、< aop:aspect >、< aop:pointcut >、< aop:declare-parents >、< aop:advisor >等标签的详细配置以及切入点表达式的详细语法。没有讲过多源码，提供了大量的案例，对于会使用Spring AOP的人来说可能比较啰嗦，但是比较适合Spring初学者！**  

### 文章目录

*   [1 AOP的概述](#1_AOP_5)
*   [2 AOP的概念](#2_AOP_15)
*   *   [2.1 核心概念](#21__16)
    *   *   [2.1.1 Joinpoint](#211_Joinpoint_18)
        *   [2.1.2 Pointcut](#212_Pointcut_63)
        *   [2.1.3 Advice](#213_Advice_66)
        *   [2.1.4 Aspect](#214_Aspect_76)
        *   [2.1.5 其他概念](#215__78)
    *   [2.2 Spring AOP与AspectJ](#22_Spring_AOPAspectJ_84)
    *   *   [2.2.1 来源和目的](#221__86)
        *   [2.2.2 织入方式与性能](#222__89)
        *   [2.2.3 切入点支持](#223__101)
        *   [2.2.4 使用范围](#224__105)
        *   [2.2.5 其他](#225__108)
        *   [2.2.6 应用](#226__112)
*   [3 基于XML的AOP配置](#3_XMLAOP_117)
*   *   [3.1 Spring AOP第一例](#31_Spring_AOP_119)
    *   [3.2 相关依赖](#32__245)
    *   [3.3 引入AOP schema](#33_AOP_schema_259)
    *   [3.4 IoC管理bean](#34_IoCbean_285)
    *   [3.5 aop:config 配置](#35_aopconfig__293)
    *   [3.6 aop:aspect 切面](#36_aopaspect__304)
    *   *   [3.6.1 配置通知](#361__386)
        *   [3.6.2 环绕通知](#362__530)
        *   [3.6.3 JoinPoint](#363_JoinPoint_634)
        *   [3.6.4 传递参数](#364__826)
        *   *   [3.6.4.1 传递参数值](#3641__829)
            *   [3.6.4.2 传递方法注解](#3642__970)
    *   [3.7 aop:pointcut 切入点表达式](#37_aoppointcut__1064)
    *   *   [3.7.1 切入点指示符PCD](#371_PCD_1147)
        *   [3.7.2 execution PCD](#372_execution_PCD_1176)
        *   *   [3.7.2.1 常见语法](#3721__1192)
        *   [3.7.3 within PCD](#373_within_PCD_1493)
        *   [3.7.4 this和target PCD](#374_thistarget_PCD_1517)
        *   *   [3.7.4.1 this和target测试](#3741_thistarget_1531)
            *   *   [3.7.4.1.1 指向接口](#37411__1556)
                *   [3.7.4.1.2 指向类](#37412__1625)
        *   [3.7.5 args PCD](#375_args_PCD_1668)
        *   [3.7.6 @args PCD](#376_args_PCD_1683)
        *   [3.7.7 @target和@within PCD](#377_targetwithin_PCD_1692)
        *   *   [3.7.7.1 @target和@within测试](#3771_targetwithin_1704)
        *   [3.7.8 @annotation PCD](#378_annotation_PCD_1822)
        *   [3.7.9 bean PCD](#379_bean_PCD_1830)
        *   [3.7.10 组合 PCD](#3710__PCD_1838)
    *   [3.8 aop:declare-parents 引介](#38_aopdeclareparents__1845)
    *   [3.9 aop:advisor 通知器](#39_aopadvisor__1960)
    *   [3.10 scoped-proxy 作用域代理](#310_scopedproxy__2054)
*   [4 总结](#4__2160)

1 AOP的概述
========

  **“面向切面编程（Aspect Oriented Programming，简称AOP）通过提供另一种程序结构的思考方式来补充面向对象的编程（Object Oriented Programming，OOP）”——[Spring官方文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)。面向切面编AOP程是OOP后，又一种重要的编程思维方式。**  
  系统中的业务，通常分为核心业务和非核心业务。比如一个客服系统，它的核心业务就是客服管理、电话服务和客户记录等，非核心业务包括登陆鉴权、日志记录、异常处理等。不同逻辑的核心业务通常都需要依赖相同逻辑的非核心业务，非核心业务的经常出现在非核心业务的前后，用来保证系统的安全性和稳定性。  
  OOP的编程思维中，基本模块单元是类（class），OOP将不同的业务对象的抽象成为一个个的类，不同的业务操作抽象成不同的方法，这样的好处是能获得更加清晰高效的逻辑单元划分！一个完整的业务逻辑就是调用不同的对象、方法来组合完成，这类似于流水线，核心业务和非核心业务都在里面，每一个步骤按照顺序执行。这样看来，业务逻辑之间的耦合关系非常严重，核心业务的代码之间通常需要手动嵌入大量非核心业务的代码，比如日志记录、事务管理。对于这种跨对象和跨业务的重复的、公共的非核心逻辑，OOP没有特别好的处理方式。  
  如果说OOP的编程中，业务代码逻辑是固定运行流程的从上到下的流水线关系，核心业务逻辑和非核心业务逻辑之间相互杂揉，那么AOP技术就能将不同业务流程中的相同的非核心业务逻辑从源代码中彻底抽离出来，形成一个独立的服务（比如日志记录、权限校验、异常处理、事物机制）。而当程序在编译/运行的时候，又能在不修改源代码的情况下，动态的选择在程序执行流程中的某些地方，比如方法运行前后，抛出异常时等，将这些非核心服务逻辑插入到核心代码逻辑中。  
  AOP的基本模块单元是切面（aspect），所谓切面，其实就是对不同业务流水线中的相同业务逻辑进行进一步的抽取形成的一个“横截面”。AOP计数让业务中的核心模块和非核心模块的耦合性进一步降低，实现了代码的复用，减少了代码量，提升开发效率，并有利于代码未来的可扩展性和可维护性。  
  **简单的说，OOP对业务中每一个功能进行抽取、封装成类、方法，让代码更加模块化，在一定程度上实现了代码的复用。此时，一个完整的业务通过一定顺序的调用对象的方法模块的来实现。如果脱离对象层面，基于业务逻辑的站在更高层面来看这种编程方式，带来的缺点是对于业务中的重复的代码模块，需要在源代码中，在业务的不同阶段重复调用。而AOP则可以对业务中重复调用的模块进行抽取，让业务中的核心逻辑与非核心逻辑进一步解耦，源代码中不需要手动调用这个重复代码的模块，在更高的层实现了代码的复用，有利于后续代码的维护升级。AOP的层次更高，AOP和OOP不是竞争关系，AOP是对OOP的补充！**  
  上面的文字可能有一些空洞，下面以图形的样式来看看单纯的OOP的业务和使用AOP之后的业务：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916232557609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)  
  现在让我们来学习AOP技术，本文主要讲使用方式，而不是源码。

2 AOP的概念
========

2.1 核心概念
--------

  **AOP中有一些核心概念以及术语，这些概念在大部分AOP框架之间是通用的，并且它们并非他别易懂。注意（甩锅），这些概念并不是Spring AOP定义的（或许就是AspectJ框最开始定义的），Spring AOP仅仅是接受（屈服）了这些概念，或许Spring是怕如果也搞一套自己的概念可能会让用户更加困惑？**

### 2.1.1 Joinpoint

  **连接点。程序执行时的一些特定位置/点位，这些点位就是可能被AOP框架拦截并织入代码的地方。** 常见的有下面这些点位：

|Method Call|方法被调用时，即在一个方法中调用另一个方法的时。
|Method Execution|方法执行时。即某个方法内部开始执行时。
|Constructor Call|某个构造器被调用时。
|Constructor Execution|构造器内部开始执行时。
|Field Set|通过方法或者直接设置某个变量的值时。
|Field Get|通过方法或者直接访问某个变脸的值时。
|Exception Handlers|异常抛出时。
|Staitc Initialization|类的静态属性/代码块被初始化/执行时。
|initialization|对象通过构造器初始化时

  还有更多的连接点，[Appendix A. AspectJ Quick Reference](https://www.eclipse.org/aspectj/doc/released/progguide/quick.html)：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091623285683.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)  
  **目前，在spring AOP中，连接点只支持method execution，即方法执行连接点，并且不能应用于在同一个类中相互调用的方法。**

### 2.1.2 Pointcut

  **切入点。用来匹配要进行切入的Joinpoint集合的表达式，通过切点表达式（pointcut expression，类似正则表达式）可以确定符合条件的连接点作为切入点。**  
  Spring AOP使用与AspectJ框架同样的切点表达式语法。后面会介绍语法样式。

### 2.1.3 Advice

  **通知。切面的具体行为/功能，在pointcut匹配到的Joinpoint位置，会插入指定类型的Advice。** Spring AOP中的通知的类型有：

1.  Before advice：前置通知。在切入点方法之前运行但不能阻止执行到切入点方法的通知（除非它抛出异常）。
2.  After Returning advice：后置通知。在切入点方法正常完成后要运行的通知（例如，如果方法返回并且不引发异常）。
3.  After Throwing advice：异常通知。如果切入点方法通过引发异常而退出，则要执行的通知。
4.  After Finally advice：最终通知。无论切入点方法退出的方式如何（正常或异常返回），都要执行的通知。
5.  Around advice：环绕通知。Around通知可以在切入点方法调用前后执行自定义行为。它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。

  后面会详细讲解通知的配置与使用！

### 2.1.4 Aspect

  **切面。切入点（Pointcut）和该位置的通知（Advice）的结合。或者说就是开头所说的跨多个业务的被抽离出来的公共业务模块，就像一个“横截面”一样，对应Java代码中被@AspectJ标注的切面类或者使用XML配置的切面。** 后面会详细讲解切面的配置与使用！

### 2.1.5 其他概念

1.  Introduction：引介。一种特殊的通知（advice），在不修改类代码的前提下，Introduction可以在运行期为类动态地添加一些额外的方法或属性，实际用的比较少。
2.  Target Object：目标对象。代理的目标对象，被代理对象，也被称为“advised object”。Spring AOP 是通过使用运行时代动态理实现的。
3.  AOP Proxy：代理。一个类被AOP织入增强后，就会产生一个代理对象。在Spring框架中，AOP代理是JDK动态代理或CGLIB代理。
4.  Weaving：织入。是指把切面应用到目标对象来创建新的代理对象的过程。织入的时期可以是编译时织入（AspectJ），也可以使用运行时织入（Spring AOP）。

2.2 Spring AOP与AspectJ
----------------------

  对于Java程序员来说，最常使用的支持AOP的框架有两个，一个是Spring AOP，另一个就是AspectJ框架，Spring AOP与AspectJ共用上面的基本概念，那么它们之间有什么区别和联系呢？

### 2.2.1 来源和目的

  AspectJ是由Eclipse开源的一个AOP框架，基于Java平台，致力于提供了最完整的AOP实现方式，官方地址为：http://www.eclipse.org/aspectj/。  
  Spring AOP是Spring提供的一个AOP框架。目的并不是提供最完整的AOP实现，相反，其目的是在AOP实现和SpringIOC之间提供紧密的集成，以帮助解决企业应用程序中的大多数常见的需求和问题（方法织入）。官方地址为：https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop。

### 2.2.2 织入方式与性能

  **AspectJ属于静态织入**，它使用了专门的称为 AspectJ 编译器 (ajc) 的编译器，在Java源码被编译的时候，就将切面织入到目标对象所属类的字节码文件中，并不会生成新的代理类字节码。因此，AspectJ 在运行时不做任何事情，没有任何额外的开销，因为切面在类编译的时候就织入了。  
  **Spring AOP属于动态织入，其原理就是动态代理。** 在运行时，会临时动态的生成目标对象的代理类，其性能可能不如AspectJ。Spring AOP使用了两种动态代理机制：

1.  JDK动态代理——这要求目标对象**必须实现了至少一个合理的接口（不是说只要是实现了接口就会使用JDK代理，具体规则在AOP源码部分有讲解）。** 借助java内部的反射机制，动态产生一个实现和目标对象所属相同接口的代理类。然后通过接口中的方法名，在动态生成的代理类中调用业务实现类的同名方法并进行代理，因此**只能代理接口的方法，这是Spring AOP 的默认代理方式（注意在 SpringBoot2.x 版本中，默认采用CGLIB代理）。**
2.  CGLIB动态代理 ——这要求**目标对象的类不能是final修饰的**。如果目标对象没有实现任何接口，则会选择使用 CGLIB 代理。借助asm机制，会把被目标对象的class文件加载进来，修改其字节码生成一个继承了目标对象的子类，通过重写业务方法进行代理，因此**需要代理的方法也不能是private/final/static的。**
3.  **默认情况下，无论是JDK还是CGLIB的代理，同一个目标对象中的方法相互调用的时候不会触发后续方法的代理机制**，因为默认情况下最终是通过this即目标对象去调用方法的。当然可以手动解决，方法还很多，随便举几个例子：
    1.  可以**将该类“本身”的实例注入到该类中**，实际上注入的是一个代理对象，并且这也算一种自身循环依赖，但是Spring可以帮我们解决，所以不用怕报错，然后**通过这个代理对象手动去调用另一个方法即可。**
    2.  **将这两个方法分开到不同的目标类里面**——有点233……。
    3.  **对于XML，设置< aop:config/>标签的expose-proxy属性为true，对于注解，设置@EnableAspectJAutoProxy的exposeProxy属性为true，其目的就是将代理对象暴露出来，然后代码中使用AopContext.currentProxy()即可获取代理对象，然后强制转型去调用方法即可（注意类型兼容性）。**
    4.  **以上方式对于事务方法也有效，但是如果该类存在[@Async异步任务](https://blog.csdn.net/weixin_43767015/article/details/110135495)方法，那么@Async方法应该使用第一种方式并且在引入的自身代理对象上加上@Lazy注解，让其再进行代理封装。**

### 2.2.3 切入点支持

  Spring AOP致力于提供企业中最常见的问题的解决方案。**仅仅支持Method Execution被作为切入点，即将方法调用作为切入点，这也是开发中最常见的切入点。** 仅仅支持方法调用的织入（或者从动态代理的层面说，仅仅能拦截方法并织入相应的逻辑）。  
  使用JDK的动态代理，那么被代理类必须实现接口，因此只能代理接口的方法；使用CGLIB动态代理，那么被代理类必须能够被继承，不能是final的，并且被代理的方法不能是private/final/static方法，因为它们不能被继承或者覆盖（重写、代理）。**默认情况下，同一个目标对象中的方法相互调用的时候不会触发后续方法的代理机制。**  
  AspectJ致力于提供了最完整的AOP实现方式，支持所有的Joinpoint连接点被选为切入点，支持构造器、属性、方法、静态块……的织入，更加灵活。由于是编译时静态织入，直接修改字节码文件，因此上面的对于Spring AOP的种种限制，都可以被AspectJ突破。

### 2.2.4 使用范围

  Spring AOP只能和Spring IOC联合使用，AOP作用的对象只能是被IoC容器管理的bean。  
  AspectJ则可以单独使用，作用于任何pojo对象上。

### 2.2.5 其他

  Spring支持无缝集成AspectJ框架，因此也能使用AspectJ的全部功能。  
  Spring2.0以后新增了对AspectJ切点表达式的支持。AspectJ框架在1.5版本时，通过JDK5的注解技术引入了一批注解，比如@AspectJ、@Pointcut、相关Advice注解，支持使用注解的声明式方式来定义切面，Spring同样支持使用和AspectJ相同的注解，并且。但是，这相当于一个模版，底层仍然是走的Spring AOP的动态代理逻辑，并且不依赖于AspectJ的编译器或者织入器。  
  Spring AOP相比于AspectJ，它的学习难度更低，更容易上手。特别是如果我们开发时使用了Spring框架，那么建议就使用Spring AOP。

### 2.2.6 应用

  Spring中AOP的主要应用就是：

1.  声明式服务的支持，比如声明式的事务控制机制。
2.  让用户自定义切面，用AOP补充OOP的使用。比如通过AOP实现多数据源自动切换。

3 基于[XML](https://so.csdn.net/so/search?q=XML&spm=1001.2101.3001.7020)的AOP配置
============================================================================

  下面我们来学习使用XML进行配置和使用Spring AOP！

3.1 Spring AOP第一例
-----------------

  如前几篇文章的案例一样，建立一个空maven项目：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916233823933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)  
  在pom.xml中引入相关依赖：

```xml
<properties>
    <spring-framework.version>5.2.8.RELEASE</spring-framework.version>
    <aspectjweaver>1.9.6</aspectjweaver>
</properties>
<dependencies>
    <!--spring 核心组件所需依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring-framework.version}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
    <!--用于解析AspectJ的切入点表达式语法-->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>${aspectjweaver}</version>
    </dependency>
</dependencies>
```


  我们在源码路径下加入两个类：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916233857754.png#pic_center)

```Java
/**
 * 第一个Spring AOP案例
 * 该类用于定义通知的逻辑
 *
 * @author lx
 */
public class FirstAopAspect {

    /**
     * 通知的行为/逻辑
     */
    public void helloAop() {
        System.out.println("hello Aop");
    }
}
//------------------
/**
 * 第一个Spring AOP案例
 * 被代理的目标对象
 *
 * @author lx
 */
public class FirstAopTarget {

    /**
     * 配置被代理的方法
     */
    public void target() {
        System.out.println("Method is proxyed");
    }

    /**
     * 没被代理的方法
     */
    public void target2() {
        System.out.println("Method is not proxyed");
    }
}
```


  resources加入如下spring-config.xml配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--需要添加Spring AOP的Schema支持-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--被代理类和通知类都交给IoC管理-->
    <bean class="com.spring.aop.FirstAopTarget" name="firstAopTarget"/>
    <bean class="com.spring.aop.FirstAopAspect" name="firstAopAspect"/>
    <!--aop的相关配置都写到aop:config标签中-->
    <aop:config>
        <!--配置 aspect 切面-->
        <aop:aspect id="myAspect" ref="firstAopAspect">
            <!--配置 advice 通知 以及应用的切入点(表达式)-->
            <aop:before method="helloAop" pointcut="execution(public void com.spring.aop.FirstAopTarget.target())"/>
            <aop:after method="helloAop" pointcut="execution(public void com.spring.aop.FirstAopTarget.target())"/>
        </aop:aspect>
    </aop:config>
</beans>
```


  测试：

```java
/**
 *
 * @author lx
 */
public class AopTest {
    public static void main(String[] args) {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
        //2.获取对象
        FirstAopTarget firstAopTarget = (FirstAopTarget)ac.getBean("firstAopTarget");
        //3.尝试调用被代理类的相关方法
        firstAopTarget.target();
        System.out.println("-------------");
        firstAopTarget.target2();
    }
}
```


  结果如下：

```java
hello Aop
Method is proxyed
hello Aop
-------------
Method is not proxyed
```


  可以看到，被代理的类的相关方法被成功织入相关通知逻辑，AOP第一例到此结束！下面我们来讲一讲详细配置！

3.2 相关依赖
--------

  Spring AOP需要引入两个依赖。本次我们同样直接使用spring-context依赖，因为它帮助我们引入了Spring其他核心的依赖，这其中包括Spring AOP的依赖，我们进入spring-context内部就能发现，它已经帮助我们依赖了其他组件：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916234115830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)  
  然后，我们还需要引入aspectjweaver依赖，我们前面说过Spring支持AspectJ的切入点表达式的语法，这个依赖就是用来解析切入点表达式的。

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<!--用于解析AspectJ的切入点表达式语法-->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>${aspectjweaver}</version>
</dependency>
```


3.3 引入AOP schema
----------------

  Spring的原始配置文件仅支持IoC的配置，如果想要使用aop的XML配置，我们需要手动引入AOP Schema（命名空间），然后就能使用aop相关标签。aop标签用于配置Spring中的所有AOP，包括Spring自己的基于代理的AOP框架和Spring与AspectJ AOP框架的集成。Schema属于XML的知识，它可以帮助省略相关配置。  
  普通schema：

```xml
<beans xmlns=http://www.springframework.org/schema/beans
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
       
</beans>
```


  加入aop schema之后：

```xml
<!--需要添加Spring AOP的Schema支持-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">

</beans>
```


3.4 IoC管理bean
-------------

  想要使用Spring AOP，无论是需要被AOP增强的类还是定义通知的切面类，都需要交给IoC容器管理。

```xml
<!--被代理类和通知类都交给IoC管理-->
<bean class="com.spring.aop.FirstAopTarget" name="firstAopTarget"/>
<bean class="com.spring.aop.FirstAopAspect" name="firstAopAspect"/>
```


3.5 aop:config 配置
-----------------

  aop的相关配置都写在< aop:config >标签中的，用于实现Spring自动代理机制。

```xml
<!--aop的相关配置都写到aop:config标签中-->
<aop:config>
    <!--相关配置-->
</aop:config>
```


  在< aop:config >标签中可以包含pointcut->advisor->aspect标签。请注意，这些标签必须按该顺序声明。  
  **< aop:config >标签可以配置proxy-target-class属性为"true"，这表示强制使用CGlib动态代理，多个< aop:config >的属性是共享的。**

3.6 aop:aspect 切面
-----------------

  **< aop:aspect >标签用于配置切面，其内部可以定义具体的应用到哪些切入点的通知。**< aop:aspect >标签本身可以配置三个属性：

1.  id：一个切面的唯一标识符。
2.  ref：用于引用的外部专门定义的通知bean，bean的内部定义了一系列advice通知的逻辑（方法），在< aop:aspect >内部定义通知的时候可以通过method属性引用这些方法。通知bean实际上就是一个普通bean，只不过将内部的方法的逻辑作为通知的逻辑。
3.  order：切面的排序。当有多个通知在同一个切入点执行时，指定通知执行的先后顺序，未指定order属性时默认值为Integer.MAX_VALUE，即2147483647。order越小的切面，其内部定义的前置通知越先执行，后置通知越后执行。相同的order的切面则按照切面从上到下定义顺序先后执行前置通知，反向执行后置通知。

  如下案例，加入一个被代理类和一个通知类，用于测试order：

```java
/**
 * 测试order
 * @author lx
 */
public class AopAspectOrder {

    /**
     * 通知的行为/逻辑
     */
    public void advance1() {
        System.out.println("advance1");
    }

    /**
     * 通知的行为/逻辑
     */
    public void advance2() {
        System.out.println("advance2");
    }
}
//--------------------
/**
 * @author lx
 */
public class AopTargetOrder {

    public void target() {
        System.out.println("test order");
    }
}
```


  配置文件如下，定义两个切面，并且都包含对于同一个切入点的同类型通知：

```xml
<bean class="com.spring.aop.aop.AopAspectOrder" name="aopAspectOrder"/>
<bean class="com.spring.aop.aop.AopTargetOrder" name="aopTargetOrder"/>
<aop:config>
    <!--两个切面，配置同一个切入点-->
    <!--切面order 2147483647-->
    <aop:aspect id="myAspect" ref="aopAspectOrder">
        <aop:before method="advance1" pointcut="execution(public void com.spring.aop.aop.AopTargetOrder.target())"/>
        <aop:after method="advance1" pointcut="execution(public void com.spring.aop.aop.AopTargetOrder.target())"/>
    </aop:aspect>
    <!--切面order 2147483646-->
    <aop:aspect id="myAspect" ref="aopAspectOrder" order="2147483646">
        <aop:before method="advance2" pointcut="execution(public void com.spring.aop.aop.AopTargetOrder.target())"/>
        <aop:after method="advance2" pointcut="execution(public void com.spring.aop.aop.AopTargetOrder.target())"/>
    </aop:aspect>
</aop:config>
```


  测试：

```Java
@Test
public void testOrer(){
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    AopTargetOrder aopTargetOrder = (AopTargetOrder)ac.getBean("aopTargetOrder");
    //3.尝试调用被代理类的相关方法
    aopTargetOrder.target();
}
```


  结果如下：

```Java
advance2
advance1
test order
advance1
advance2
```


### 3.6.1 配置通知

  **在< aop:aspect >切面标签内部可以使用对应的标签配置对应类型的5种通知。它们都有如下属性：**

1.  **method**：通知的逻辑，这个逻辑是写在方法中的，就是对应上面< aop:aspect >标签对应的bean中的方法名。
2.  **pointcut**：一个切入点表达式，用于指定该通知可以应用到的切入点集合。通过这个表达式可以匹配到某些方法作为该通知的切入点。
3.  **ponitcut-ref**：用于指定切入点的表达式的引用。可以通过< aop:ponitcut >单独定义切入点。
4.  **arg-names**：按顺序使用“,”分隔的需要匹配的方法参数名字符串。用于配合切入点表达式，更加细致的匹配方法，更重要的是可以进行参数值的传递（后面会将）。

  **我们前面说过有5种类型的通知，自然有5个标签相互对应：**

1.  < aop:before >标签用于配置前置通知before advice，在切入点方法执行之前执行。
2.  < aop:after-returning >标签用于配置后置通知after-returning advice。在切入点方法正常执行之后可能会执行。后置通知的方法能够接收切入点方法的返回值作为参数，只需要配置returning属性，returning属性值就是后置通知方法的参数名，参数类型需要与返回值类型匹配（基本类型可以自动拆装箱，但是不能自动强制转型），或者向上兼容（可使用父类、父接口接收），否则后置通知不会被执行。
3.  < aop:after-throwing >标签用于配置异常通知after-throwing advice。在前置通知、切入点方法和后置通知中抛出异常之后可能会执行。异常通知的方法能够接收前置通知、切入点方法和后置通知中抛出的异常作为参数，只需要配置throwing属性，throwing属性值就是异常通知方法的参数名。参数类型需要与抛出的异常类型匹配或者向上兼容（可使用父类、父接口接收），否则异常通知不会被执行。
4.  < aop:after >标签用于配置最终通知after advice。无论切入点方法是否正常执行，它都会在after-returning advice或者after-throwing advice后面执行。
5.  < aop:around >标签用于配置环绕通知around advice。

  **如果都配置了前置通知、后置通知、异常通知、最终通知，那么可能的执行流程如下：**

1.  如果正常执行，那么执行流程是：前置通知->切入点方法->后置通知->最终通知。
    1.  如果后置通知方法设置了参数并且正常捕获了切入点方法的返回值，或者没有设置参数，那么后置通知可以正常执行，否则后置通知不会被执行。
2.  如果前置通知中抛出异常，那么执行流程是：前置通知（抛出异常）->异常通知->最终通知。
    1.  如果异常通知方法设置了参数并且正常捕获了该异常，或者没有设置参数，那么异常通知可以执行，否则异常通知不会被执行。
3.  如果切入点方法中抛出异常，那么执行流程是：前置通知->切入点方法（抛出异常）->异常通知->最终通知。
    1.  如果异常通知方法设置了参数并且正常捕获了该异常，或者没有设置参数，那么异常通知可以执行，否则异常通知不会被执行。
4.  如果后置通知中抛出异常，那么执行流程是：前置通知->切入点方法->后置通知（抛出异常）->异常通知->最终通知。
    1.  如果异常通知方法设置了参数并且正常捕获了该异常，或者没有设置参数，那么异常通知可以执行，否则异常通知不会被执行。
5.  如果异常通知中抛出异常，那么前面的执行流程不会改变，异常通知的参数也能捕获前面几种步骤中抛出的异常，区别就是异常通知（它是一个方法）也会抛出自己的异常而已，后置通知也会依旧执行。
6.  如果最终通知中抛出异常，那么前面的执行流程不会改变，区别就是最终通知也会抛出自己的异常而已。
7.  异常最终会在最后一个流程执行完毕之后输出到控制台抛出（无论异常通知有没有捕获）。根据上面的情况，我们发现，不同的执行步骤中都可以抛出自己的异常，比如：前置通知->切入点方法（抛出自己的异常）->异常通知（抛出自己的异常）->最终通知（抛出自己的异常）。那么最终会抛出什么异常呢？实际上，最终，后面步骤中抛出的异常信息会覆盖前面步骤中抛出的异常信息，即，最终控制台输出的只是最后面的步骤中抛出的异常的信息，因此可能会对异常的定位有一些干扰。所以，我们尽量不要在通知方法中抛出自己的异常！

  如下案例，用于测试4种通知的逻辑：

```Java
/**
 * 测试4种通知
 *
 * @author lx
 */
public class AopTarget4Advice {

    public int target() {
        System.out.println("---test 4 advice target---");
        //引发一个异常，可能被异常通知捕获，并尝试传递给通知方法的参数
        //int j=1/0;

        //返回值，可以被后置通知捕获，并传递给后置通知的方法参数
        return 3;
    }
}
//------------------
/**
 * 测试4种通知
 *
 * @author lx
 */
public class AopAspect4Advice {

    /**
     * 前置通知
     */
    public void beforeAdvice() {
        System.out.println("before advice");
        //这将引发一个异常，可以被异常通知捕获，并尝试传递给通知方法的参数，这还会导致切入点方法不被执行
        //int j=1/0;
    }

    /**
     * 后置通知
     */
    public void afterReturningAdvance(int i) {
        System.out.println("i: " + i);
        System.out.println("after-returning advice");
        //这将引发一个异常，可以被异常通知捕获，并尝试传递给通知方法的参数
        //最终抛出异常是时，它将覆盖前面所有流程中抛出的异常。
        //Object o=null;
        //o.toString();
    }

    /**
     * 异常通知
     */
    public void afterThrowingAdvance(Exception e) {
        System.out.println("e: " + e.getMessage());
        System.out.println("after-throwing advice");
        //这将引发一个异常，它将覆盖前面所有流程中抛出的异常。
        //int j=1/0;
    }

    /**
     * 最终通知
     */
    public void afterAdvance() {
        System.out.println("after advice");
        //这将引发一个异常，它将覆盖前面所有流程中抛出的异常，最终输出的异常就行就是该异常。
        //int j=1/0;
    }

}
```


  配置文件：

```xml
<bean class="com.spring.aop.AopAspect4Advice" 
name="aopAspect4Advice"/>
<bean class="com.spring.aop.AopTarget4Advice" name="aopTarget4Advice"/>
<aop:config>
    <aop:aspect id="4Advice" ref="aopAspect4Advice">
        <aop:before method="beforeAdvice"
                    pointcut="execution(public int com.spring.aop.AopTarget4Advice.target())"/>
        <!--可以尝试接收切入点方法的返回值-->
        <aop:after-returning method="afterReturningAdvance"
                             pointcut="execution(public int com.spring.aop.AopTarget4Advice.target())"
                             returning="i"/>
        <!--可以尝试接收前置通知、切入点方法、后置通知的抛出的异常-->
        <aop:after-throwing method="afterThrowingAdvance"
                            pointcut="execution(public int com.spring.aop.AopTarget4Advice.target())"
                            throwing="e"/>
        <aop:after method="afterAdvance"
                   pointcut="execution(public int com.spring.aop.AopTarget4Advice.target())"/>
    </aop:aspect>
</aop:config>
```


  测试：

```java 
@Test
public void test4Adivce(){
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    AopTarget4Advice aopTarget4Advice = (AopTarget4Advice)ac.getBean("aopTarget4Advice");
    //3.尝试调用被代理类的相关方法
    aopTarget4Advice.target();
}
```


  一次正常的执行流程如下：

```java 
before advice
---test 4 advice target---
i: 3
after-returning advice
after advice
```


### 3.6.2 环绕通知

  **前面学习了四种通知，最后一种环绕通知around advice比较特殊，它能在切入点方法之前、之后、抛出异常时都执行，并且可以控制切入点方法到底什么时候执行、怎么执行的一种通知。**  
  环绕通知使用< aop:around >标签配置，并且通常情况下，环绕通知都是独立使用的，即其他通知基本上可以相互配合，但是如果配置了环绕通知，那么基本上不会配置其他四种通知。  
  环绕通知的方法的第一个参数必须是ProceedingJoinPoint类型，在通知逻辑（方法体）中，对ProceedingJoinPoint调用proceed()方法将会导致执行切入点方法的逻辑，proceed()的返回的值就是切入点方法的返回值，我们可以对该返回值进行加工，而环绕通知方法的返回值就是外部调用切入点方法获取的最终返回值，如果没有返回值，那么外部调用切入点方法获取的最终返回值为null。环绕通知的返回值类型应该和切入点方法的返回值类型一致或者兼容。  
  proceed方法还能传递一个数组，该数组就是切入点方法所需的参数。可以通过对ProceedingJoinPoint调用getArgs获取外部调用切入点方法时传递进来的参数数组，也可以在环绕通知的逻辑中自己设置参数。  
  我们可以在可以在环绕通知的方法主体中调用proceed方法一次、多次或根本不调用，所有这些都是合法的。这就是上面所说的可以控制切入点方法到底什么时候执行、怎么执行的功能。因此一个around advice在一定程度上具有前面四个通知的全部功能！

  如下案例，用于测试环绕通知：

```java 
/**
 * 测试环绕通知
 *
 * @author lx
 */
public class AopTargetAround {

    public int target(int x,int y) {
        System.out.println("---test around advice target---");
        //引发一个异常
        //int j=1/0;
        //返回值
        return x+y;
    }
}
//-------------
/**
 * 环绕通知
 *
 * @author lx
 */
public class AopAspectAround {

    /**
     * 一定要有ProceedingJoinPoint类型的参数
     */
    public int around(ProceedingJoinPoint pjp) {
        int finalReturn = 0;
        Object[] args = pjp.getArgs();
        System.out.println("外部传递的参数: " + Arrays.toString(args));
        System.out.println("==前置通知==");
        try {
            //proceed调用切入点方法，args表示参数，proceed就是切入点方法的返回值
            Object proceed = pjp.proceed(args);
            //也可以直接掉用proceed方法，它会自动传递参数外部的参数
            //Object proceed = pjp.proceed(args);
            System.out.println("切入点方法的返回值: " + proceed);
            System.out.println("==后置通知==");
            finalReturn = (int) proceed;
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("==异常通知==");
            finalReturn = 444;
        } finally {
            System.out.println("==最终通知==");
        }
        //外部调用切入点方法获取的最终返回值
        return finalReturn;
    }
}
```


  配置文件：

```xml
<!--环绕通知测试-->
<bean class="com.spring.aop.AopAspectAround" name="aopAspectAround"/>
<bean class="com.spring.aop.AopTargetAround" name="aopTargetAround"/>
<aop:config>
    <aop:aspect id="around" ref="aopAspectAround">
        <!--配置环绕通知 和其他通知的配置都差不多-->
        <aop:around method="around" pointcut="execution(public int com.spring.aop.AopTargetAround.target(int,int))"/>
    </aop:aspect>
</aop:config>
```


  测试：

```Java
@Test
public void aroundAdivce() {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    AopTargetAround aopTargetAround = (AopTargetAround) ac.getBean("aopTargetAround");
    //3.尝试调用被代理类的相关方法
    int x = 2;
    int y = 1;
    System.out.println("-----外部调用切入点方法传递的参数: " + x + " " + y);
    int target = aopTargetAround.target(x, y);
    //最终返回值
    System.out.println("-----外部调用切入点方法获取的最终返回值: " + target);
}
```


  结果如下：

```java
-----外部传递的参数: 2 1
外部传递的参数: [2, 1]
==前置通知==
---test around advice target---
切入点方法的返回值: 3
==后置通知==
==最终通知==
-----外部调用切入点方法获取的最终返回值: 3
```


### 3.6.3 JoinPoint

  **任何advice通知方法的第一个参数都可以被声明为org.aspectj.lang.JoinPoint类型，JoinPoint是连接点方法的抽象，提供了访问当前被通知方法的目标对象、代理对象、方法参数等数据的方法。环绕通知的参数类型应该使用ProceedingJoinPoint，它也是JoinPoint的一个实现。所有传入的JoinPoint的实际类型都是MethodInvocationProceedingJoinPoint。**

  **JoinPoint的相关方法以及解释如下：**

|String toString()|返回使用“execution(”和“)”包装的连接点方法的签名。实际上就是对getSignature()返回的Signature的toString()方法的返回值包装。返回值和参数类型使用简单类名。
|String toShortString()|返回使用“execution(”和“)”包装的连接点方法的简要签名。实际上就是对getSignature()返回的Signature的toShortString ()方法的返回值包装。省略返回值、参数类型、类路径。
|String toLongString();|返回使用“execution(”和“)”包装的连接点方法的完整签名。实际上就是对getSignature()返回的Signature的toLongString ()方法的返回值包装。返回值和参数类型使用全路径名。
|Object getThis()|返回当前AOP代理对象
|Object getTarget()|返回当前AOP目标对象
|Object[] getArgs()|返回当前被通知方法传递的实际参数值数组
|Signature getSignature()|返回当前连接点方法的签名
|SourceLocation getSourceLocation()|返回连接点方法所在类文件中的位置，相关方法不支持
|String getKind()|返回当前连接点的类型。Spring AOP为method-execution。
|StaticPart getStaticPart()|返回连接点静态部分，实际上就是返回当前JoinPoint对象

  **ProceedingJoinPoint有两个额外的方法：**
|Object proceed() throws Throwable|执行目标方法，默认使用外部传递的参数。
|Object proceed(Object[] args) throws Throwable|执行目标方法，使用该方法传递的数组的值作为参数。

  因此，环绕通知应该使用ProceedingJoinPoint，用来执行目标方法。一般用的最多的方法就是getArgs()和proceed()。

  如下案例，我们来测试JoinPoint的方法：

```Java
/**
 * @author lx
 */
public class JoinPointAspectTarget  {

    public List<Integer> target(String str, Date date) {
        System.out.println("_____target_____");
        return Stream.of(1,2,3).collect(Collectors.toList());
    }
}
//----------------------
/**
 * @author lx
 */
public class JoinPointAspectAdvice {

    public void advice(JoinPoint joinPoint) {
        System.out.println("_______advice______");
        System.out.println(joinPoint.getClass());
        System.out.println(joinPoint.toString());
        System.out.println(joinPoint.toShortString());
        System.out.println(joinPoint.toLongString());
        System.out.println(joinPoint.getThis());
        System.out.println(joinPoint.getTarget());
        System.out.println(Arrays.toString(joinPoint.getArgs()));
        System.out.println(joinPoint.getSignature());
        System.out.println(joinPoint.getSourceLocation());
        System.out.println(joinPoint.getKind());
        System.out.println(joinPoint.getStaticPart());
        System.out.println("_____advice_____");
    }

    public Object adviceAdvice(ProceedingJoinPoint joinPoint) {
        System.out.println("_______adviceAdvice______");
        System.out.println(joinPoint.getClass());
        System.out.println(joinPoint.toString());
        System.out.println(joinPoint.toShortString());
        System.out.println(joinPoint.toLongString());
        System.out.println(joinPoint.getThis().getClass());
        System.out.println(joinPoint.getTarget().getClass());
        System.out.println(Arrays.toString(joinPoint.getArgs()));
        System.out.println(joinPoint.getSignature());
        //SourceLocation相关方法不支持
        System.out.println(joinPoint.getSourceLocation());
        System.out.println(joinPoint.getKind());
        System.out.println(joinPoint.getStaticPart());
        //执行方法
        Object proceed = null;
        try {
             proceed = joinPoint.proceed();
            System.out.println(proceed);
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("_____adviceAdvice_____");
        return proceed;
    }
}
```


  配置文件：

```xml
<bean class="com.spring.aop.joinPoint.JoinPointAspectAdvice"
 name="joinPointAspectAdvice"/>
<bean class="com.spring.aop.joinPoint.JoinPointAspectTarget" name="joinPointAspectTarget"/>
<!--通知参数-->
<aop:config>
    <aop:pointcut id="jp1" expression="execution(* com.spring.aop.joinPoint.JoinPointAspectTarget.target(..))"/>
    <aop:aspect ref="joinPointAspectAdvice">
        <aop:before pointcut-ref="jp1" method="advice"/>
        <aop:after-returning pointcut-ref="jp1" method="advice"/>
        <aop:after pointcut-ref="jp1" method="advice"/>
        <!--<aop:around pointcut-ref="jp1" method="adviceAdvice"/>-->
    </aop:aspect>
</aop:config>
```


  测试：

```java
@Test
public void jp() {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    JoinPointAspectTarget joinPointAspectTarget = (JoinPointAspectTarget) ac.getBean("joinPointAspectTarget");
    //3.尝试调用被代理类的相关方法
    List<Integer> xx = joinPointAspectTarget.target("xx", new Date());
    System.out.println(xx);
}
```


  结果如下：

```java 
_______advice______
class org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint
execution(List com.spring.aop.joinPoint.JoinPointAspectTarget.target(String,Date))
execution(JoinPointAspectTarget.target(..))
execution(public java.util.List com.spring.aop.joinPoint.JoinPointAspectTarget.target(java.lang.String,java.util.Date))
com.spring.aop.joinPoint.JoinPointAspectTarget@69fb6037
com.spring.aop.joinPoint.JoinPointAspectTarget@69fb6037
[xx, Wed Sep 16 12:50:10 CST 2020]
List com.spring.aop.joinPoint.JoinPointAspectTarget.target(String,Date)
org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint$SourceLocationImpl@5552768b
method-execution
execution(List com.spring.aop.joinPoint.JoinPointAspectTarget.target(String,Date))
_____advice_____
//............
```


  将其他通知注释，使用环绕通知继续测试，结果如下：

```java
_______adviceAdvice______
class org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint
execution(List com.spring.aop.joinPoint.JoinPointAspectTarget.target(String,Date))
execution(JoinPointAspectTarget.target(..))
execution(public java.util.List com.spring.aop.joinPoint.JoinPointAspectTarget.target(java.lang.String,java.util.Date))
class com.spring.aop.joinPoint.JoinPointAspectTarget$$EnhancerBySpringCGLIB$$c509993a
class com.spring.aop.joinPoint.JoinPointAspectTarget
[xx, Wed Sep 16 12:51:32 CST 2020]
List com.spring.aop.joinPoint.JoinPointAspectTarget.target(String,Date)
org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint$SourceLocationImpl@36b4fe2a
method-execution
execution(List com.spring.aop.joinPoint.JoinPointAspectTarget.target(String,Date))
_____target_____
[1, 2, 3]
_____adviceAdvice_____
[1, 2, 3]
```


### 3.6.4 传递参数

  前面我们讲了，可以通过returning传递目标方法的返回值或者throwing传递抛出的异常给相关的后置通知或者异常通知方法。在上面的介绍中，我们也可以使用JoinPoint的getArgs获取数组参数。实际上，我们也可以将参数值传递给通知方法的参数！  
  **下面的传递参数部分的内容比较复杂，开发中用的不多！**

#### 3.6.4.1 传递参数值

  如下案例，有两个类：

```java
/**
 * @author lx
 */
public class ParamAspectTarget {
    public ParamAspectTarget target(int i, Date date, String string) {
        //构造一个异常
        //int y=1/0;
        ParamAspectTarget paramAspectTarget = new ParamAspectTarget();
        System.out.println("target: "+paramAspectTarget);
        return paramAspectTarget;
    }
}
//-----------------
/**
 * @author lx
 */
public class ParamAspectAdvice {

    public void before(JoinPoint joinPoint, int i2, Date date, String string ) {
        System.out.println("-----before-----");
        System.out.println(joinPoint);
        System.out.println(i2);
        System.out.println(date);
        System.out.println(string);
        System.out.println("-----before-----");
    }
```


​    
```java
    public void afterReturning(JoinPoint joinPoint, Date date, String string, ParamAspectTarget returned) {
        System.out.println("-----afterReturning-----");
        System.out.println(joinPoint);
        System.out.println(date);
        System.out.println(string);
        System.out.println(returned);
        System.out.println("-----afterReturning-----");
    }

    public void afterThrowing(JoinPoint joinPoint, int i, Exception e ,Date date ) {
        System.out.println("-----afterThrowing-----");
        System.out.println(joinPoint);
        System.out.println(i);
        System.out.println(date);
        System.out.println(e);
        System.out.println("-----afterThrowing-----");
    }

    public void after(JoinPoint joinPoint, int i, Date date, String string) {
        System.out.println("-----after-----");
        System.out.println(joinPoint);
        System.out.println(i);
        System.out.println(date);
        System.out.println(string);
        System.out.println("-----after-----");
    }

    public Object around(ProceedingJoinPoint joinPoint, int i, Date date, String string) {
        System.out.println("-----around-----");
        System.out.println(joinPoint);
        System.out.println(i);
        System.out.println(date);
        System.out.println(string);
        System.out.println("-----around-----");
        Object proceed = null;
        try {
            proceed = joinPoint.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }finally {
            return proceed;
        }
    }
}
```


  配置文件：

```xml
<!--传递参数-->
<bean class="com.spring.aop.param.ParamAspectAdvice" name="paramAspectAdvice"/>
<bean class="com.spring.aop.param.ParamAspectTarget" name="paramAspectTarget"/>
<!--通知参数-->
<aop:config>
    <!--args参数名与通知方法的参数名一致-->
    <aop:pointcut id="par1"
                  expression="execution(* com.spring.aop.param.ParamAspectTarget.target(..)) and args(i2,date,string) "/>
    <!--args参数名与通知方法的参数名不一致，并且执行传递某些位置的参数-->
    <aop:pointcut id="par2"
                  expression="execution(* com.spring.aop.param.ParamAspectTarget.target(..)) and args(*,date1,str) "/>

    <aop:pointcut id="par3"
                  expression="execution(* com.spring.aop.param.ParamAspectTarget.target(..)) and args(i,date,..) "/>

    <aop:pointcut id="par4"
                  expression="execution(* com.spring.aop.param.ParamAspectTarget.target(..)) and args(i,date,string) "/>

    <aop:aspect ref="paramAspectAdvice">
        <!--args参数名与通知方法的参数名一致时，arg-names属性可以省略-->
        <aop:before method="before" pointcut-ref="par1" arg-names="joinPoint,i2,date,string"/>
        <!--arg-names按照顺序为每一个参数命名，想要参入参数，那么需要对应该args中的名字-->
        <!--将会按照顺序和类型进行赋值，如果类型不匹配，那么该通知方法不会被调用-->
        <aop:after-returning method="afterReturning" pointcut-ref="par2"
                             arg-names="joinPoint,date1,str,return" returning="return"/>

        <!--args参数名与通知方法的参数名一致时，arg-names属性可以省略-->
        <aop:after-throwing method="afterThrowing" pointcut-ref="par3" throwing="e"/>
        <!--args参数名与通知方法的参数名一致时，arg-names属性可以省略-->
        <aop:after method="after" pointcut-ref="par4" />
        <!--args参数名与通知方法的参数名一致时，arg-names属性可以省略-->
        <!--<aop:around method="around" pointcut-ref="par4"/>-->
    </aop:aspect>
</aop:config>
```


  **一般来说，我们的通知方法的参数与目标方法的参数名称、顺序、类型一致，然后只需要在切入点表达式中配置args()，并且按目标方法的顺序声明参数名称，即可实现参数值通过方法参数的传递，并且可以省略arg-names属性。**  
  **当然，args()指定的参数名称也可以与通知方法的参数名不一致，此时我们需要在通知中配置arg-names属性，该属性的值使用“,”分隔，一个值代表一个通知方法的参数名，它的名称需要与args()中定义的名称一致，这样也可以进行参数的传递。**  
  **如果类型不匹配，那么不会进行参数传递，也不会执行通知方法。即args()也可以用来进行切入点的筛选。**  
  测试结果如下，成功通过参数注入：

```Java
-----before-----
execution(ParamAspectTarget com.spring.aop.param.ParamAspectTarget.target(int,Date,String))
333
Wed Sep 16 15:42:14 CST 2020
xx
-----before-----
target: com.spring.aop.param.ParamAspectTarget@c333c60
-----afterReturning-----
execution(ParamAspectTarget com.spring.aop.param.ParamAspectTarget.target(int,Date,String))
Wed Sep 16 15:42:14 CST 2020
xx
com.spring.aop.param.ParamAspectTarget@c333c60
-----afterReturning-----
-----after-----
execution(ParamAspectTarget com.spring.aop.param.ParamAspectTarget.target(int,Date,String))
333
Wed Sep 16 15:42:14 CST 2020
xx
-----after-----
```


#### 3.6.4.2 传递方法注解

  **实际上，我们使用@annotation()的PCD也可以将方法上的注解传递给通知方法，然后就可以在通知方法中拦截注解的内容了。**  
  目标方法如下，上面有多个注解：

```Java
@Scope(value = "111")
@Description("描述")
@Lazy
public void target() {
    System.out.println("annotation target");
}
```


  通知方法如下，用于具有某些注解的方法被调用时可以触发。  
  **注意一个@annotation()只能传递一个注解，如果一个通知方法要传递多个注解，那么需要使用多个@annotation()。**

```java
public void anno1(Scope scopee){
    System.out.println(scopee);
    System.out.println(scopee.value());
    System.out.println("---scope---");
}

public void anno2(Description scopee){
    System.out.println(scopee);
    System.out.println(scopee.value());
    System.out.println("---description---");
}
public void anno3(Lazy lazy){
    System.out.println(lazy);
    System.out.println(lazy.value());
    System.out.println("---lazy---");
}
public void anno4(Test scopee){
    System.out.println(scopee);
    System.out.println("---test---");
}
```


  配置文件如下，我们在上一个案例的< aop:aspect ref=“paramAspectAdvice”>下添加配置：

```xml
<!--类似于方法参数的传递，配合使用@annotation()，内部可以传递一个自定义的参数名字-->
<!--如果这个名字和某些通知方法的参数名一致，那么对应通知的的arg-names参数可以省略-->

<!--这里参数名定义为：scopee-->
<aop:pointcut id="ann1"
              expression="execution(* com.spring.aop.param.ParamAspectTarget.target()) and @annotation(scopee)"/>
<!--尝试传递两个注解作为参数，需要多个@annotation-->
<aop:pointcut id="ann2"
              expression="execution(* com.spring.aop.param.ParamAspectTarget.target())
              and @annotation(scopee) and @annotation(description) "/>
<!--该通知对应anno1方法，其参数名就是scopee，因此arg-names参数省略-->
<aop:before method="anno1" pointcut-ref="ann1"/>
<!--该通知对应anno2方法，其参数名就是scopee，因此arg-names参数省略-->
<aop:before method="anno2" pointcut-ref="ann1"/>
<!--该通知对应anno3方法，其参数名是lazy，因此arg-names参数不能省略，必须和@annotation中设定的名字一致-->
<aop:before method="anno3" pointcut-ref="ann1" arg-names="scopee"/>
<!--该通知对应anno4方法，其参数名就是scopee，因此arg-names参数省略-->
<aop:before method="anno4" pointcut-ref="ann1" arg-names="scopee"/>
<!--传递两个注解参数-->
<aop:before method="anno5" pointcut-ref="ann2" />
```


  测试：

```java 
@Test
public void ann() {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    ParamAspectTarget paramAspectTarget = (ParamAspectTarget) ac.getBean("paramAspectTarget");
    //3.尝试调用被代理类的相关方法
    paramAspectTarget.target();
}
```


  测试结果如下，ann1、ann2、ann3、ann5的通知方法都被触发了：

```java 
@org.springframework.context.annotation.Scope(proxyMode=DEFAULT, value=111, scopeName=)
111
---anno1 scope---
@org.springframework.context.annotation.Description(value=描述)
描述
---anno2 description---
@org.springframework.context.annotation.Lazy(value=true)
true
---anno3 lazy---
111
描述
---anno5 two Anno---
annotation target
```


  **实际上，@within, @target, @annotation, 和 @args、this()、target()被称为不同类型的切入点表达式，即不同的PCD（下面马上介绍），它们数据都可以按照上面的方式将不同的参数传递给通知方法并进行切入点筛选，下面将进行切入点表达式的详细介绍，我们会认识更多的切入点的筛选方法！**

3.7 aop:pointcut 切入点表达式
-----------------------

  每一个通知中，都可以配置自己的切入点表达式，很多时候切入点表达式都是一样的，我们可以使用< aop:pointcut >标签定义一个独立的切入点表达式，使得多个切面和advisor通过pointcut-ref属性共享一个切入点表达式。  
  **expression属性：用于定义切入点表达式，id属性：用于给切入点表达式提供一个唯一标识，通过该标识引用对应的切面表达式。**  
  < aop:pointcut >可以定义在< aop:aspect >内部，这表示该表达式只能在当前切面内部的通知中引用，也可以定义在< aop:config >内部，此时表示所有切面的所有通知中都能引用，注意< aop:pointcut >标签的定义顺序在< aop:aspect >标签之前，另外，如果有多个< aop:config >中有相同id的切入点表达式，则最终只会应用最后定义的切入点表达式，它们的切面最终会合在一起。

  如下案例，用于测试aop:pointcut标签：

```java
/**
 * pointcut 测试
 * @author lx
 */
public class AopTargetPointcut {

    public void target1() {
        System.out.println("test pointcut1");
    }

    public void target2() {
        System.out.println("test pointcut2");
    }
}
//----------------
/**
 * pointcut 测试
 * @author lx
 */
public class AopAspectPointcut {

    public void advice1() {
        System.out.println("pointcut advice1");
    }
```


​    
```java
    public void advice2() {
        System.out.println("pointcut advice2");
    }
}
```


  配置文件，不同的通知都可以引用同一个切入点表达式：

```xml
<!--aop:pointcut测试-->
<bean class="com.spring.aop.AopAspectPointcut" name="aopAspectPointcut"/>
<bean class="com.spring.aop.AopTargetPointcut" name="aopTargetPointcut"/>
<aop:config>
    <!--配置一个所有切面的所有通知都能引用的表达式  aop:pointcut标签要定义在最前面 -->
    <aop:pointcut id="p1" expression="execution(public void com.spring.aop.AopTargetPointcut.target1())"/>

    <!--一个切面-->
    <aop:aspect id="as1" ref="aopAspectPointcut">
        <!--pointcut-ref 引用切面表达式-->
        <aop:before method="advice1" pointcut-ref="p1"/>
    </aop:aspect>

    <!--一个切面-->
    <aop:aspect id="as2" ref="aopAspectPointcut">
        <!--配置只能是当前切面内部的所有通知都能引用的表达式-->
        <aop:pointcut id="p2" expression="execution(public void com.spring.aop.AopTargetPointcut.target2())"/>
        <!--pointcut-ref 引用切面表达式-->
        <aop:before method="advice1" pointcut-ref="p2"/>
        <aop:after method="advice2" pointcut-ref="p1"/>
    </aop:aspect>
</aop:config>
```


  测试：

```java
@Test
public void testPointcut () {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    AopTargetPointcut aopTargetPointcut = (AopTargetPointcut) ac.getBean("aopTargetPointcut");
    //3.尝试调用被代理类的相关方法
    aopTargetPointcut.target1();
    System.out.println("----------");
    aopTargetPointcut.target2();
}
```


  **Spring AOP 仅支持 Spring bean 的方法执行连接点，因此我们可以将切入点表达式视为用来匹配 Spring bean 上的某些方法。**  
  Spring AOP支持的切入点表达式的语法完全使用AspectJ 5的切入点表达式语法，下面我们主要学习常用的切入点表达式的语法。完整的AspectJ切入点表达式语言的语法非常丰富，可以看[AspectJ 编程指南](https://www.eclipse.org/aspectj/doc/released/progguide/index.html)或者[AspectJ 5 Developer’s Notebook](https://www.eclipse.org/aspectj/doc/released/adk15notebook/index.html)。

### 3.7.1 切入点指示符PCD

  **首先我们需要认识一下AspectJ的切入点指示符（pointcut designators，简称PCD），PCD用来指明切入点表达式的大概目的，简单的说就是通过匹配什么来进行连接点筛选。由于在Spring AOP中目前只有执行方法这一个连接点，目前Spring 5.x的AOP支持使用如下PCD：**

1.  execution()：通过匹配某些类型的某些方法签名来匹配连接点方法。
2.  within()：通过匹配某些类型来匹配连接点方法。
3.  this()：通过匹配AOP的代理对象类型来匹配连接点方法。
4.  target()：通过匹配AOP的目标对象类型来匹配连接点方法。
5.  args()：通过匹配方法参数的数量、类型、顺序来匹配连接点方法。
6.  @target()：通过匹配类型上的某些注解类型来匹配连接点方法。
7.  @args()：通过匹配方法参数的所属类型上的某些注解来匹配连接点方法。
8.  @within()：通过匹配类型及子类型上的某些注解类型来匹配连接点方法。
9.  @annotation()：通过匹配方法上的某些注解类型来匹配连接点方法。
10.  bean()：通过匹配某些bean name来匹配连接点方法。Spring AOP特有的。

  其他的AspectJ的PCD，比如call、get、set、preinitialization、staticinitialization、initialization、handler、adviceexecution、withincode、cflow、cflowbelow、if、@this和@withincode等等目前的Spring AOP均不支持。在使用Spring AOP的情况下，在切入点表达式中使用这些PCD会导致引发IllegalArgumentException，但是未来可能会扩展。

  **PCD还支持通配符：**

> 1.  \* 任意数量的字符；
> 2.  .. 任意项的重复；主要用于execution的declaring-type-pattern中，表示匹配当前包及其子包，以及execution的param-pattern中，表示匹配任意数量参数。
> 3.  \+ 该类型及其子类型；

  **PCD还支持运算符（基于XML的配置中，建议使用and、or、not 来表示）：**

> 1.  &&(and) 两个条件都要匹配。
> 2.  ||(or) 两个条件匹配一个。
> 3.  !(not) 不能匹配该条件。

  下面来详细学习上面的PCD！

### 3.7.2 execution PCD

  **因为Spring AOP仅支持方法执行的切入点，execution的切入点表达式使用方法的签名来筛选切入点方法，因此execution是Spring AOP中使用的最多的一种PCD，这是我们着重要学习的！**  
  execution的切入点表达式语法格式如下：

> execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)

1.  带有问号的表示可选项；
2.  modifiers-pattern：方法的访问修饰符（可选），如public、protected；
3.  ret-type-pattern：方法的返回值类型全路径名，java.lang包下的类可以写简单类名。
4.  declaring-type-pattern：方法所属类的全路径名（可选）；
5.  name-pattern：方法名；
6.  param-pattern：方法参数列表类型，要求使用全路径类名，java.lang包下的类可以写简单类名。多个参数类型使用“,”分隔，按照指定顺序匹配。
7.  throws-pattern：匹配异常类型列表（可选），之前需要添加throws关键字，后面同样是异常的全路径类名，一般使用“,”分隔，java.lang包下的异常类可以写简单类名。没有就表示有没有抛出异常都无所谓。
8.  表达式中匹配的都是声明的类型，而不是实际类型，并且默认不向下兼容，可是手动使用“+”。

#### 3.7.2.1 常见语法

1.  **一个完整的execution表达式案例如下**

     execution(public String com.spring.aop.execution.AopTarget.target(String,Object) throws NullPointerException)
    

  它表示方法修饰符为public，返回值为String类型，所属的类路径和方法名称为com.spring.aop.execution.AopTarget.target，参数类型为String和Object类型，并且方法上抛出NullPointerException。

2.  **访问修饰符可以省略，表示匹配任何访问修饰符**

    execution(String com.spring.aop.execution.AopTarget.target(String,Object))
    
3.  **可以使用void表示无返回值，使用*表示任意类型的返回值（包括void）**

    execution(* com.spring.aop.execution.AopTarget.target(String,Object))
    
4.  **可以使用*，表示匹配任意包名，但是有几级子包路径，就需要写几个***

    execution(* com.*.*.aop.*.AopTarget.target(String,Object))
    

  它表示的包路径是：com包下的任意名字的一级子包下面的任意名字的一级子包下面的aop包下面的任意名字的一级子包下面的AopTarget类的target方法。  
  因此，如果存在com.xx.yy.aop.zz.AopTarget、com.aa.bb.aop.cc.AopTarget……等路径的同名target方法，那么都能被匹配（假设其他条件也匹配）。

5.  **可以使用 ..，来表示匹配当前包及下面的任意多级路径子包，及其任意名字的子包**

    execution(* com..AopTarget.target(String,Object))
    

  它表示的包路径是：com包下任意多级包路径、任意名字的子包下面的AopTarget类的target方法。  
  因此，如果存在com.AopTarget、com.aa.bb.cc.AopTarget……等路径的同名target方法，那么都能被匹配（假设其他条件也匹配）。  
  相比于*，..的含义明显更广泛。

6.  **可以使用*，表示匹配任意类名，或者类名的前缀/后缀/中缀**

    execution(* *..*.target(String,Object))
    

  匹配任意包名、包路径下的任意类中的target方法，参数类型为String和object，返回值任意。

    execution(* *..AopTarget*.target(String,Object))


  匹配任意包名、包路径下的类名以“AopTarget”为前缀的任意类中的target方法，参数类型为String和object，返回值任意。

    execution(* *..*Execution*.target(String,Object))


  匹配任意包名、包路径下的类名包含“Execution”字符的任意类中的target方法，参数类型为String和object，返回值任意。  
  实际上，前面讲使用*代表某一级包名的时候，*也可以表示包名的前缀/后缀/中缀……。

7.  **类/包路径可以省略**

    execution(* target(String,Object))
    

  它表示的包/类路径是：任意包下的任意类。

8.  **可以使用*，表示匹配任意方法名，或者方法名的前缀/后缀/中缀**

    execution(* *..*.*(String,Object))
    

  匹配任意包名、包路径下的任意类中的任意名字的方法，参数类型为String和object，返回值任意。

    execution(* *..*.target*(String,Object)


  匹配任意包名、包路径下的任意类中的方法名以“target”为前缀的任意方法，参数类型为String和object，返回值任意。

    execution(* *..*.tar*et(String,Object))


  匹配任意包名、包路径下的任意类中的方法名以“tar”为前缀，以“et”为后缀的任意方法，参数类型为String和object，返回值任意。

9.  **参数列表可以使用*表示某个位置的参数可以是任何类型，但是必须有参数**

    execution(* *..*.*(*))
    

  匹配任意包名、包路径下的任意类中的任意名字的方法，只能且必须有一个参数，参数类型任意，返回值任意。

    execution(* *..*.*(*,*))


  匹配任意包名、包路径下的任意类中的任意名字的方法，只能且必须有两个参数，参数类型任意，返回值任意。

    execution(* *..*.*(String,*))


  匹配任意包名、包路径下的任意类中的任意名字的方法，只能且必须有两个参数，一个参数类型为String，第二个参数类型任意，返回值任意。  
  另外，由于指定的参数类型为类的全路径类名，因此也能使用*、..，规则同上。

10.  **参数列表可以使用..表示任意数量、类型的参数**

    execution(* *..*.*(..))
    

  匹配任意包名、包路径下的任意类中的任意名字的方法，参数类型、数量、顺序任意，返回值任意。

    execution(* *..*.*(String,..))


  匹配任意包名、包路径下的任意类中的任意名字的方法，第一个参数必须存在且类型为String，后续的参数类型、数量、顺序任意，返回值任意。

11.  **可以在throws之后声明匹配多个异常，异常的类路径当然也可以使用*、..**

    execution(* *..*.*(..)throws NullPointerException,*.omg..SystemException)
    

  需要匹配抛出两个异常的方法，一个是NullPointerException，另一个是SystemException。  
  当然，可以使用前面的 and、or、not运算符，and就和“,”的含义一样。还可以使用“+”，表示某个异常以及其子类异常。

    execution(* *..*.*(..)throws Exception+)


  匹配的方法至少抛出一个异常，该异常可以是Exception类型及其子类型。

    execution(* *..*.*(..)throws not NullPointerException)


  匹配的方法不能抛出NullPointerException异常。

    execution(* *..*.*(..)throws  NullPointerException or NumberFormatException)


  匹配的方法至少抛出一个异常，该异常可以是NullPointerException类型或者NumberFormatException类型。

12.  **实际上涉及到类型的都可以在类的后面使用+，表示匹配某个类型及其子类型**

    execution(String+ *..AopTarget*+.target(String+,Object))
    

匹配任意包名、包路径下的类名以“AopTarget”为前缀的任意类及其子类中的target方法，参数类型为String及其子类型和object类型，返回值为String及其子类型。

13.  **实际上涉及到类型的都可以使用and、or、not运算符**

    execution(* (!com.spring..AopTarget+).target(..))
    

  匹配非“com.spring包及其子包下的AopTarget类及其子类的target方法”。

    execution(* (com..AopTarget+ and java.lang.Comparable+).*(..))


  匹配属于com包以及子包下的AopTarget类及其子类型，并且，还属于java.lang.Comparable接口及其子类型的类的任何方法。相当于匹配一个——AopTarget类及其子类，并且实现了Comparable接口的类。

    execution(* (com..AopTarget+ or java.lang.Comparable+).*(..))


  匹配属于com包以及子包下的AopTarget类及其子类型，或者，属于java.lang.Comparable接口及其子类型的类的任何方法。相当于匹配一个——AopTarget类及其子类，或者Comparable接口的实现类。

    execution(* *(String || Object,Object))


  匹配任何包的任何类的任何返回值的方法，且第一个参数可以是String或者Object类型，第二个类型为Object类型。

    execution(* *(String+ and Object+,Object))


  匹配任何包的任何类的任何返回值的方法，且第一个参数必须是String类及其子类型，并且还属于Object类及其子类型，第二个类型为Object类型。

    execution(* *(!Object,Object))


  匹配任何包的任何类的任何返回值的方法，且第一个参数必须非Object类型，第二个类型为Object类型。

    execution((String or Integer) *(..))


  匹配任何包的任何类的任何方法，要求返回值类型为String或者Integer类型。

14.  **可以通过方法上的注解来筛选**

    execution(@org.junit.Test * *(..))
    

  任何标注了@Test注解的方法。

    execution(@org..Value @org..Scope * *(..))


  任何标注了@Value和@Scope注解的方法。注解的类路径也可以使用*和..。

    execution(@(org..Value || org..Scope) * *(..))


  任何标注了@Value或者@Scope注解的方法。

    execution(@(org..Value || !org..Scope) * *(..))


  任何具有注解，并且（标注了@Value或者没有标注@Scope注解的方法。即如果表达式有通过方法上的注解来筛选的行为，那么任何没有注解的方法都将不会被选中。

15.  **可以通过方法的返回值所属类上的注解来筛选**

    execution((@org..Repository *) *(..))
    

  返回值的所属类上标注了@Repository注解的方法。

    execution((@org..Repository @org..Scope *) *(..))


  返回值的所属类上标注了@Repository注解和@Scope注解的方法。

    execution((@(org..Repository or org..Scope) *) *(..))


  返回值的所属类上标注了@Repository注解或@Scope注解的方法。

    execution((@(!org..Repository or org..Scope) *) *(..))


  返回值的所属类上，必须有注解，并且（没有标注@Repository注解或标注了@Scope注解）的方法。即如果表达式有通过方法的返回值所属类上的注解来筛选的行为，那么任何方法的返回值所属类没有注解的方法都将不会被选中。

16.  **可以通过方法本身所属类上的注解来筛选**

    execution(* (@org..Repository *..AopTarget*).*(..))
    

  类名以“AopTarget”为前缀并且标注了@Repository注解的方法。

    execution(* (@org..Repository @org..Scope *).*(..))


  标注了@Repository注解和@Scope注解的类的方法。

    execution(* (@(org..Repository || org..Scope) *).*(..))


  标注了@Repository注解或@Scope注解的类的方法。

    execution(* (@(!org..Repository || !org..Scope) *).*(..))


  标注有注解，并且（没有标注@Repository注解或没有标注@Scope注解）的类的方法。即如果表达式有通过方法本身所属类上的注解来筛选的行为，那么任何方法本身所属类上的没有注解的方法都将不会被选中。

17.  **可以通过方法参数上的注解来筛选**

    execution(* *(@org..Value (java.io.Serializable+ and Comparable+),..))
    

  至少带有一个参数的方法，第一个参数的类型必须是Serializable及其子类型和Comparable及其子类型，并且参数上标注了@org包下的任意@Value注解。

    execution(* *(@org..Value (*),@org..Value (*)))


  两个参数的方法，且两个参数都标注了@Value注解。

    execution(* *(@org..Value @org..Qualifier (String),@org..Value (*)))


  两个参数的方法，且第一个参数标注了@Value注解和@Qualifier注解，且类型为String，第二个参数标注了@Value注解。

    execution(* *(@(org..Value or org..Qualifier) (*),..))


  至少带有一个参数的方法，第一个参数的标注了@Value注解或者@Qualifier注解。

    execution(* *(!@org..Value @org..Qualifier (*),..))


  至少带有一个参数的方法，第一个参数的必须没有标注@Value注解，并且必须标注@Qualifier注解。

    execution(* *(@(!org..Value or org..Qualifier) (*),..))


  至少带有一个参数的方法，并且第一个参数必须标注有注解，并且（没有标注@Value注解或者标注@Qualifier注解）。即如果表达式有通过方法参数上的注解来筛选的行为，那么任何对应位置的方法参数上的没有注解的方法都将不会被选中。

18.  **可以通过方法参数的所属类上的注解来筛选**

    execution(* *(@org..Repository *,..))
    

  至少一个参数的方法，且第一个参数的所属类上标注了@Repository注解。

    execution(* *(!@org..Repository @org..Scope *,..))


  至少一个参数的方法，且第一个参数的所属类上标注了@Scope注解，并且没有标注@Repository注解。

    execution(* *(@(!org..Repository or org..Scope) *,..))


  至少一个参数的方法，且第一个参数的所属类上必须标注了注解，并且，没有标注@Repository注解或者标注了@Scope注解。即如果表达式有通过方法参数的所属类上的注解来筛选的行为，那么任何对应位置的方法参数的所属类上的没有注解的方法都将不会被选中。

    execution(* *(@org..Value (@org..Repository *),@org..Value (@org..Component *)))


  两个参数的方法，且两个参数上都必须标注@Value注解，且第一个参数所属类上必须标注@Repository注解，第二个参数所属类上面必须标注@Component注解。

19.  **可以通过方法参数的泛型类型来进一步筛选**

  请注意，如果使用XML配置，那么建议使用转义字符，“<”表示“<”，“>”表示“>”。

    execution(* *( java.util.List &lt; Number+ &gt; ,..))


  至少一个参数的方法，第一个参数为List类型，并且泛型参数的类型要求Number及其子类类型。那么List参数类型的方法将不会被当作切入点。

    execution(* *( java.util.List &lt; @org..Component * &gt; ,..))


  至少一个参数的方法，第一个参数为List类型，并且要求泛型参数所属的类型上标注有@Component注解。

    execution(* *( java.util.List &lt; @(!org..Repository or org..Component) * &gt; ,..))


  至少一个参数的方法，第一个参数为List类型，并且要求泛型参数所属的类型上必须标注有注解，并且（没有标注@Repository注解或者标注了@Component注解）。  
泛型匹配可能有些兼容问题，不建议使用！

### 3.7.3 within PCD

  within的切入点表达式使用类型来筛选切入点方法，很明显within PCD的粒度控制不及execution PCD，一般用的比较少。  
  within的切入点表达式语法格式为：within(declaring-type-pattern)。

    within(*)


  任何包的任何类下面的全部方法。

    within(com.spring.*.AopTargetExecution2)


  com.spring包的下一级子包下的以“AopTarget”为前缀类名的类的全部方法。

    within(com.spring..AopTarget*)


  com.spring包的任意子包级别下的以“AopTarget”为前缀类名的类的全部方法。

    within(com.spring..AopTargetExecution || com.spring..AopTargetExecution2)


  com.spring包下的任意包级别下的AopTargetExecution和AopTargetExecution2类的全部方法。

    within(*..AopTarget* and Comparable+)


  任何包下的以“AopTarget”为前缀类名，并且属于Comparable及其子类类型的类的全部方法。

### 3.7.4 this和target PCD

  this()表示匹配某类及其代理子类的方法，target()表示匹配某类的及其子类方法。内部的表达式不支持 * 和 .. ，只能使用全路径名（java.lang包下的类同样可以写简单类名）。上一部分也说过，它们还能传递参数,分别传递代理对象和目标对象。  
  **根据Spring AOP使用的动态代理方式的区别，this和tasget PCD 在某些情况下具有一致的含义，某些情况下则不同：**

1.  如果方法所属目标类实现了接口，那么Spring默认走JDK的代理，将目标类和代理类都会实现相同的接口，在代理对象中调用目标对象的方法来进行增强。我们从容器获取的bean实际上就是一个代理类对象。
2.  如果方法所属目标类没有实现接口，那么Spring走CGlib的代理，代理类会继承目标类，然后在代理类对象中调用父目标类的方法来进行增强。我们从容器获取的bean实际上就是一个代理类对象。
3.  可以手动指定使用CGlib代理：< aop:aspectj-autoproxy proxy-target-class=“true”/>或者< aop:config proxy-target-class=“true”>

  **结合this()和target()的含义，可以有如下结果：**

1.  如果表达式指向一个接口，无论是JDK代理还是CGlib代理，该接口下的目标类生成的对应的代理类也相当于实现了该接口，this()和target()具有相同的结果；
2.  如果表达式指向一个类，并且该类没有实现接口，那么该类及其子类将使用CGlib代理，代理类继承了目标类，那么this()和target()还是具有相同的结果；
3.  如果表达式指向一个类，并且该类实现了接口，那么该类及其子类将默认使用JDK代理，代理类和目标类虽然实现了共同的接口，但是它们之间没有任何父子联系，那么this()和target()将具有不同的结果；如果指定使用Cglib代理，那么this()和target()具有相同的结果。

#### 3.7.4.1 this和target测试

  如下案例：

```java
/**
 * 目标类的接口
 */
public interface AspectInterface {
     void target();
}
//----------------
/**
 * 通知类
 */
public class AspectAdvice {
    public void thisAdvice() {
        System.out.println("___thisAdvice___");
    }

    public void targetAdvice() {
        System.out.println("___targetAdvice___");
    }
}
```


​    

##### 3.7.4.1.1 指向接口

  我们首先测试指向接口的表达式，并且使用JDK代理（默认）：

```xml
<bean class="com.spring.aop.thisTarget.AspectInterfaceImpl" 
name="aspectInterface"/>
<bean class="com.spring.aop.thisTarget.AspectAdvice" name="aspectAdvice"/>
<aop:config>
    <!--定义切入点-->
    <!--this 指向一个顶级接口-->
    <aop:pointcut id="th1"  expression="this(com.spring.aop.thisTarget.AspectInterface)"/>
    <!--target 指向一个顶级接口-->
    <aop:pointcut id="ta1"  expression="target(com.spring.aop.thisTarget.AspectInterface)"/>

    <!--定义切面-->
    <aop:aspect id="tt" ref="aspectAdvice">
        <aop:before method="thisAdvice" pointcut-ref="th1"/>
        <aop:before method="targetAdvice" pointcut-ref="ta1"/>
    </aop:aspect>
</aop:config>
```


  测试：

```java
@Test
public void testInterface() {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取bean对象，实际上是一个代理对象
    AspectInterface aspectInterface = ac.getBean("aspectInterface", AspectInterface.class);
    System.out.println("class : " + aspectInterface.getClass());
    System.out.println(aspectInterface instanceof AspectInterface);
    //使用JDK的代理，不要强转为AspectInterfaceImpl类型，因为代理类和目标类没有父子关系，它们仅仅是都实现了AspectInterface而已
    //使用CGlib的代理，可以强转为AspectInterfaceImpl类型，因为代理类继承了目标类，间接的，它们也都同属于一个接口体系
    System.out.println(aspectInterface instanceof AspectInterfaceImpl);
    //3.尝试调用被代理类的相关方法
    System.out.println("------------");
    aspectInterface.target();
}
```


  结果如下：

```xml
class : class com.sun.proxy.$Proxy9
true
false
------------
___thisAdvice___
___targetAdvice___
target
```


  可以看到，this和target通知都生效了。  
  随后，我们加入如下配置，强制使用CGlib动态代理：

    <aop:config proxy-target-class="true">


  其他不变，继续测试，结果如下：

    class : class com.spring.aop.thisTarget.AspectInterfaceImpl$$EnhancerBySpringCGLIB$$73051835
    true
    true
    ------------
    ___thisAdvice___
    ___targetAdvice___
    target


  可以看到，this和target通知都生效了。这说明，指向接口时，两个代理的效果都是一样的！

##### 3.7.4.1.2 指向类

  我们接着测试指向类的表达式，并且实现了接口：

```xml
<aop:config>
    <!--定义切入点-->
    <!--this 指向一个类-->
    <aop:pointcut id="th2"  expression="this(com.spring.aop.thisTarget.AspectInterfaceImpl)"/>
    <!--target 指向一个类-->
    <aop:pointcut id="ta2"  expression="target(com.spring.aop.thisTarget.AspectInterfaceImpl)"/>

    <!--定义切面-->
    <aop:aspect id="tt2" ref="aspectAdvice">
        <aop:before method="thisAdvice" pointcut-ref="th2"/>
        <aop:before method="targetAdvice" pointcut-ref="ta2"/>
    </aop:aspect>
</aop:config>
```


  还是使用上面的测试代码，在默认JDK的代理情况下，结果如下：

```java 
class : class com.sun.proxy.$Proxy7
true
false
------------
___targetAdvice___
target
```

  **可以看到，this()并没匹配到，因为它匹配AspectInterfaceImpl及其子类的代理类。而返回对象代理类型和AspectInterfaceImpl没有父子关系，因此this()不会被匹配，而在代理对象中会调用目标类对象的方法，因此target()会被匹配。**  
  如果强制使用CGlib代理，那么结果如下：

```java 
class : class com.spring.aop.thisTarget.AspectInterfaceImpl$$EnhancerBySpringCGLIB$$73051835
true
true
------------
___thisAdvice___
___targetAdvice___
target
```


  可以看到，this()和target()并匹配到了，因为Cglib代理类属于AspectInterfaceImpl的子类，所以this()会被匹配，而target()自然会被匹配。  
  如果我们的AspectInterface不实现接口，那么默认就会使用Cglib匹配，同样，this()和target()都会匹配到！

### 3.7.5 args PCD

  args()通过匹配方法参数的数量、类型、顺序来匹配连接点方法，用的也比较少。语法格式为：args(param-pattern)。  
  参数类型为全路径名，不支持 \* 和 .. ，java.lang包下的类可以写简单类名。args属于运行时动态匹配，可以匹配的某个参数的运行时传递的类型及其子类型，开销比较大。execution表达式中的param-pattern则是严格的类型匹配，不支持向下兼容。

    args(*)


  匹配只有一个参数的任何方法。

    args(java.io.Serializable,..)


  匹配至少有一个参数的任何方法，且第一个参数类型为运行时传递的Serializable类型及其子类型。

### 3.7.6 @args PCD

  **通过匹配方法参数的所属类型上的某些注解来匹配连接点方法，语法为@args(annotation-type)。上一部分也说过，它还能传递参数，传递方法参数的所属类型上的注解。**  
  注解类型使用全路径名，不支持使用“*”和“…”，Java.lang包下的注解可以使用简单类名。

    @args(org.springframework.stereotype.Repository,..)


  第一个参数所属类型标注有@Repository注解的方法。

### 3.7.7 @target和@within PCD

  **@target通过类型上的某些注解类型来匹配连接点方法，语法为@target(annotation-type)。@within()通过匹配类型上的某些注解类型来匹配连接点方法，还会匹配具有该注解的类的子类，语法为@ within (annotation-type)。上一部分也说过，它们还能传递参数，传递方法所属类型上的注解。**  
  注解类型使用全路径名，不支持使用 \* 和 ..，Java.lang包下的注解可以使用简单类名。

    @target(org.springframework.stereotype.Repository)


  匹配标注了@Repository注解的类的全部方法。

    @within(org.springframework.stereotype.Repository)


  匹配标注了@Repository注解的类的全部方法，以及通过这些类的子类（要求同样交给IoC管理）bean调用的父类方法。  
  **二者的相同点都很好理解，但是不同点很多文章都没有解释清楚，其实也很好理解，下面一起来测试一下就行了！**

#### 3.7.7.1 @target和@within测试

  如下案例，Parent类上标注了@Deprecated注解（该注解本身不具备继承性），Child类继承了Parent类，但没有标注@Deprecated注解，重写了target2方法，没有重写target1方法：

```java 
/**
 * @author lx
 */
@Deprecated
public class Parent {

    public void target1() {
        System.out.println("target1");
    }

    public void target2() {
        System.out.println("target2");
    }
}
//-------------------
/**
 * @author lx
 */
public class Child extends Parent {

    /**
     * 重写了一个target2方法
     */
    @Override
    public void target2() {
        super.target2();
    }

    /**
     * 自己的target3方法
     */
    public void target3() {
        System.out.println("target3");
    }
}
//------------------
/**
 * 通知类
 */
public class AspectWTAdvice {
    public void advice() {
        System.out.println("___withinTarget___");
    }
}
```


  配置文件如下，首先使用@within测试：

```xml
<bean class="com.spring.aop.withinTarget.AspectWTAdvice"
 name="aspectWTAdvice"/>
<bean class="com.spring.aop.withinTarget.Parent" name="parent"/>
<bean class="com.spring.aop.withinTarget.Child" name="child"/>
<aop:config>
    <!--一个within通知-->
    <aop:pointcut id="within" expression="@within(Deprecated)"/>
    <!--一个target通知-->
    <aop:pointcut id="target" expression="@target(Deprecated)"/>

    <aop:aspect id="wt" ref="aspectWTAdvice">
        <!--使用within通知-->
        <aop:before method="advice" pointcut-ref="within"/>
    </aop:aspect>
</aop:config>
```


  测试，我们使用子类bean child来调用方法：

```java
@Test
public void testWT() {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    Child child = (Child) ac.getBean("child");
    Parent parent = (Parent) ac.getBean("parent");
    //3.尝试调用被代理类的相关方法
    System.out.println("---------");
    //子类bean调用父类的方法
    child.target1();
    //子类bean调用重写的方法
    child.target2();
    //子类bean调用自己的方法
    child.target3();

    System.out.println("=======");
    //具有该注解的类调用自己的方法，会被匹配
    parent.target1();
}
```


  结果如下：

```xml
---------
___withinTarget___
target1
target2
target3
=======
___withinTarget___
target1
```


  **使用@within时，虽然Child子类没有@Deprecated注解注解，但是其父类具有该注解，因此子类bean调用target1方法可以被匹配，但是这要求子类bean被容器管理并且是直接调用具有该注解的父类的方法，子类重写的方法和子类自己的方法都不能被匹配到，即target2和target3方法都不能被匹配到！**  
  接下来切换到target：< aop:before method=“advice” pointcut-ref=“target”/>，同样的测试，结果如下：

```java
---------
target1
target2
target3
=======
___withinTarget___
target1
```


  **很明显，使用@target时，子类bean调用父类的方法时不能被匹配到，@target的匹配范围小于@within。**  
  到此验证完毕！

### 3.7.8 @annotation PCD

  **通过匹配方法上的某些注解类型来匹配连接点方法，语法为@annotation (annotation-type)。上一部分也说过，它还能传递参数，用于传递方法上的注解。**  
  注解类型使用全路径名，不支持使用“*”和“…”，Java.lang包下的注解可以使用简单类名。

    @annotation(org.springframework.beans.factory.annotation.Value)


  匹配方法上标注有@Value注解的方法。

### 3.7.9 bean PCD

  通过匹配某些bean name来匹配连接点方法，语法为：bean(Bean id/name)，允许*。  
  bean PCD仅在SpringAOP中受支持，在单独使用AspectJ中不受支持。它是AspectJ定义的标准PCD的特定于Spring的扩展，因此不能用于使用@Aspect声明的切面类。

    bean(aopTarget*)


  匹配以aopTarget为前缀名的bean的全部方法。

### 3.7.10 组合 PCD

  我们可以将使用上面不同类型的PCD使用 ||(or) &&(and) !(not)组合起来。

    execution(public String *(..)) and args(Comparable,..)


  方法访问修饰符为public，返回值类型为String，第一个参数传递的类型为Comparable及其子类类型，如果execution中有指定，那么以execution为主。

3.8 aop:declare-parents 引介
--------------------------

  **在不修改源代码的前提下，Introduction（引介）可以在运行期为类动态地添加一些额外的方法或属性。**  
  **我们需要使用到< aop:declare-parents >标签，该标签是< aop:aspect >标签的子标签，用于定义bean的动态增强的行为，内部有四个属性：**

1.  types-matching：需要增强的的类扫描路径，该路径下的被Spring管理的bean都将被增强。
2.  implement-interface: 新的增强方法所属的接口，实际上Spring创建的目标类的代理类会同时实现这个接口（无论是JDK还是CGlib），因此可以多一些功能。
3.  default-impl：一个实现了上面接口的类，作为新的增强方法的默认实现类。
4.  delegate-ref：引入的一个增强方法的实现类bean的id。

  如下案例，原始类：

```java 
/**
 * @author lx
 */
public class BasicFunction {
    public void get(){
        System.out.println("get");
    }

    public void update(){
        System.out.println("update");
    }
}
```


  增强接口和默认实现类：

```java 
public interface AddFunction {
    void delete();

    void insert();

    String str = "str";
}
```


  增强接口和默认实现类：

```java 
public interface AddFunction {
    void delete();

    void insert();

    String str = "str";
}
//----------------
public class AddFunctionImpl implements AddFunction {
    @Override
    public void delete(){
        System.out.println("delete");
    }

    @Override
    public void insert(){
        System.out.println("insert");
    }
}
```


  配置文件：

```xml
<!--Introductions-->
<bean class="com.spring.aop.introduction.BasicFunction" name="basicFunction"/>
<aop:config>
    <aop:aspect>
        <!--types-matching: 需要增强的的类扫描路径，该路径下的被Spring管理的bean都将被增强-->
        <!--implement-interface: 新实现的增强接口-->
        <!--default-impl: 增强接口的方法的默认实现-->
        <aop:declare-parents
                types-matching="com.spring.aop.introduction.*"
                implement-interface="com.spring.aop.introduction.AddFunction"
                default-impl="com.spring.aop.introduction.AddFunctionImpl"/>
    </aop:aspect>
</aop:config>
```


  测试：

```java
@Test
public void introduction() {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    Object o = ac.getBean("basicFunction");
    System.out.println(o.getClass());
    System.out.println(o instanceof AddFunctionImpl);
    System.out.println(o instanceof AddFunction);
    System.out.println(o instanceof BasicFunction);
    System.out.println("------------");
    BasicFunction basicFunction=(BasicFunction) o;
    //3.尝试调用被代理类的相关方法
    basicFunction.get();
    basicFunction.update();
    //转换类型，调用新增的方法
    AddFunction addFunction=(AddFunction) basicFunction;
    addFunction.delete();
    addFunction.insert();
    System.out.println(AddFunction.str);
}
```


  结果如下：

```java
class com.spring.aop.introduction.BasicFunction$$EnhancerBySpringCGLIB$$af279a47
false
true
true
------------
get
update
delete
insert
str
```


3.9 aop:advisor 通知器
-------------------

  < aop:advisor >标签用于定义一个通知器。它和< aop:aspect >类似，想象与一个小切面，二者的区别是：aspect可以配置多个不同类型的通知，引用的通知方法可以是一个普通Spring bean中的方法，而advisor只需要引入一个通知类bean即可，但是该bean要求实现Advice相关接口，Advice是“通知”的抽象，一般是实现MethodBeforeAdvice, AfterReturningAdvice, ThrowsAdvice这几个接口。  
  **基本不是涉及到自定义Advice的开发！通常，advisor通知器被用来管理事物，即advice-ref属性配置对一个< tx:advice >的引用就行了，后面学习事物的时候就知道了。**

  如下案例，仅用于演示，实际开发中基本没有这样的用法：

```java
public class AdvisorTarget {

    public void target() {
        //抛出一个异常
        //int i=1/0;
        System.out.println("---业务---");
    }
}
//------------
/**
 * < aop:advisor >主要用于声明式事物管理的配置
 *
 * @author lx
 */
public class TransactionManagement implements MethodBeforeAdvice, AfterReturningAdvice, ThrowsAdvice {
    /**
     * 前置通知
     */
    @Override
    public void before(Method arg0, Object[] arg1, Object arg2) {
        System.out.println("前置通知！");
    }

    /**
     * 后置通知
     */
    @Override
    public void afterReturning(Object arg0, Method arg1, Object[] arg2,
                               Object arg3) {
        System.out.println("后置通知!");
    }

    //异常通知
```


​    
```java
    public void afterThrowing(RemoteException ex) {
        System.out.println("异常通知!");
        // Do something with remote exception
    }

    public void afterThrowing(Exception ex) {
        System.out.println("异常通知!");
    }

    public void afterThrowing(Method method, Object[] args, Object target, Exception ex) {
        System.out.println("异常通知!");
    }

//    public void afterThrowing(Method method, Object[] args, Object target, ServletException ex) {
//        System.out.println("异常通知!");
//    }
}
```


  配置文件：

```xml
<!--advisor-->
<!--bean-->
<bean id="advisorTarget" class="com.spring.aop.advisor.AdvisorTarget"/>
<bean id="transactionManagement" class="com.spring.aop.advisor.TransactionManagement"/>

<aop:config>
    <!--切入点-->
    <aop:pointcut expression="execution(* *.target(..))" id="tx"/>
    <!--通知器配置  advice-ref指定一个实现了Advice接口的bean  pointcut-ref指向一个切入点-->
    <aop:advisor advice-ref="transactionManagement" pointcut-ref="tx"/>
</aop:config>
```


  测试：

```java
@Test
public void advisor () {
    //1.获取容器
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //2.获取对象
    AdvisorTarget advisorTarget = (AdvisorTarget) ac.getBean("advisorTarget");
    //3.尝试调用被代理类的相关方法
    advisorTarget.target();
}
```


  结果如下：

```java
前置通知！
---业务---
后置通知!
```


3.10 scoped-proxy 作用域代理
-----------------------

  **对于因为长生命周期的bean调用短生命周期的bean而导致短生命周期的bean不能被正常管理的问题，此前在IoC的学习的文章中，我们见过了两种解决方法，第一种是基于XML的< lookup-method>查找方法注入标签，第二种是基于注解的@Scope(proxyMode=xx)注解。**  
  **现在我们来学习第三种解决方法，那就是基于XML的< aop:scoped-proxy/>AOP作用域代理标签，这个标签更加的独立，和此前的< aop:config/>标签并无多大联系。**  
  **这个标签同样具有proxy-target-class属性，这个属性用于确定要使用什么代理方式，默认置为true，表示创建基于CGLIB的代理，这要求目标类不能是final的，可以设置为false，表示创建基于JDK的动态代理，这要求目标类实现了接口。**  
  **它的使用也很简单，只需要将其设置到短生命周期的bean的XML配置中即可！**  
  如下案例，两个测试类，一个是短生命周期的bean一个是长生命周期的bean，短生命周期的bean被注入到长生命周期的bean中：

```java
/**
 * @author lx
 */
public class Prototype  {
    public Prototype() {
        System.out.println("create Prototype");
    }
}

//--------------------------------

/**
 * @author lx
 */
public class Singleton {
    private Prototype prototype;

    public Prototype getPrototype() {
        return prototype;
    }

    public void setPrototype(Prototype prototype) {
        this.prototype = prototype;
    }
}
```


  配置文件，spring-aop-scopedProxy.xml：

```xml
<!--设置为prototype作用域-->
<bean class="com.spring.aop.scopedproxy.Prototype" id="prototype" scope="prototype">
</bean>
<!--设置为singleton作用域-->
<bean class="com.spring.aop.scopedproxy.Singleton" id="singleton">
    <!--短生命周期的bean被注入到长生命周期的bean中-->
    <property name="prototype" ref="prototype"/>
</bean>
```


  测试：

```java
@est
public void scopedProxy() {
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-aop-scopedProxy.xml");
    Singleton bean = ac.getBean(Singleton.class);
    System.out.println(bean);
    System.out.println(bean.getPrototype());
    System.out.println(bean.getPrototype());

    System.out.println("-----------------");

    Singleton bean2 = ac.getBean(Singleton.class);
    System.out.println(bean2==bean);
    System.out.println(bean2.getPrototype());
    System.out.println(bean2.getPrototype());
}
```


  结果如下:

```java
create Prototype
com.spring.aop.scopedproxy.Singleton@4671e53b
com.spring.aop.scopedproxy.Prototype@2db7a79b
com.spring.aop.scopedproxy.Prototype@2db7a79b
-----------------
true
com.spring.aop.scopedproxy.Prototype@2db7a79b
com.spring.aop.scopedproxy.Prototype@2db7a79b
```


  可以看到，虽然我们注入的bean是原型的，看是看起来并没有被成功控制，因为每次都是同一个原型bean对象，我们在原型bean定义中加上< aop:scoped-proxy/>之后：

```xml
<!-设置为prototype作用域-->
<bean class="com.spring.aop.scopedproxy.Prototype" id="prototype" scope="prototype">
    <!--加入作用域代理标签-->
    <aop:scoped-proxy/>
</bean>
<!--设置为singleton作用域-->
<bean class="com.spring.aop.scopedproxy.Singleton" id="singleton">
    <!--短生命周期的bean被注入到长生命周期的bean中-->
    <property name="prototype" ref="prototype"/>
</bean>
```


  再次测试：

```java
com.spring.aop.scopedproxy.Singleton@197d671
create Prototype
com.spring.aop.scopedproxy.Prototype@23986957
create Prototype
com.spring.aop.scopedproxy.Prototype@23f7d05d
-----------------
true
create Prototype
com.spring.aop.scopedproxy.Prototype@1e730495
create Prototype
com.spring.aop.scopedproxy.Prototype@7d3a22a9
```


  **可以看到，每次获取原型对象都是获取的新建的对象，成功的控制了对象的生命周期为“原型”，这就是使用< aop:scoped-proxy/>标签解决长生命周期的bean调用短生命周期的bean的问题的方式。**

4 总结
====

  刚接触的时候Spring AOP很多人还是比较迷茫的，特别是一些比较生僻的概念，但是如果你跟着做几个案例，那么应该就能很快明白这些概念的具体含义了！  
  **本文讲解了AOP的概念以及基于XML的Spring AOP配置，常用标签就是：< aop:config >：用来写aop的相关配置以及指定代理方式；< aop:aspect >：用于配置切面；< aop:pointcut >：用于配置切入点表达式；< aop:declare-parents >用于配置Introduction；< aop:advisor >用于配置一个通知器，这些标签都能和最开始的概念对应上，还是比较简单的。**  
  实际上工作中，Spring AOP按照语法规定来配置、使用就行了，需要配置的切入点表达式也基本上都是非常简单的，本文用了大量篇幅讲解切入点表达式的语法，工作中90%情况下的所用的语法不及本文讲解的语法内容的十分之一，另外还有其他的比如参数传递、引介等等知识，这些知识对于大部分项目来说都是用不上的，所以虽然本文较长，但是完全可以按自己情况选择性学习就行了，时间不是很充足的情况下，没必要一个个的知识点的认真学完。本文也没有讲过多的Spring AOP的原理、源码，主要是讲如何使用，原理部分可能会在后面的文章中进行分析。

**相关文章：**  
  Spring官网：[https://docs.spring.io/](https://docs.spring.io/)  
  XML：[Spring 5.x 学习(2)—两万字的IoC入门以及基于XML的IoC配置全解](https://blog.csdn.net/weixin_43767015/article/details/108380043)  
  注解：[Spring 5.x 学习(3)—两万字的基于注解的IoC配置全解](https://blog.csdn.net/weixin_43767015/article/details/108467731)

> 如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！