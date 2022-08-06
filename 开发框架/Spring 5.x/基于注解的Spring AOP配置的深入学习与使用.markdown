

>  基于最新Spring 5.x，详细介绍了基于Java注解的核心Spring AOP机制的配置和使用。

  **在此前Spring 5.x 学习(5)—两万字的基于XML的Spring AOP配置全解的文章之后，本次我们介绍基于注解的核心Spring AOP机制的配置和使用。包括Spring AOP的注解支持的开启，@Aspect、 @Pointcut、@DeclareParents等注解的详细配置与使用。没有讲过多源码，提供了大量的案例，对于会使用Spring AOP的人来说可能比较啰嗦，但是比较适合Spring初学者！**   **某些细节，比如切入点表达式语法，这两种配置方式都是一样的，我们在前一篇文章就已经介绍过了，本文不再赘述！**







  如同IoC支持基于XML和基于注解两种配置方式一样，Spring AOP同样支持基于XML和基于注解两种配置方式。当然，这同样带来了一个新的问题，我们应该使用基于XML的配置还是基于注解的配置？   基于XML的样式可能对目前的大部分 Spring 用户来说是最熟悉的样式，同时比较符合某些程序员心中对于“真正的POJO”的定义，基于XML的Spring AOP配置对代码没有入侵性。使用XML配置的另一个好处就是，能够将所有的切面都集中配置在一个XML文件中，对于一个大型项目来说可能比较方便管理配置。   基于XML的AOP配置，实际上将配置和代码实现分隔开了，可能对于初学者不容易理解代码和配置的关系，比如advice通知，它对应通知类中的一个方法，而基于注解的AOP，将代码和配置都封装到一个类中，比如一个@Aspectj标注的类，它就表示一个切面类，它的里面的切入点以及通知等都和该类的一个方法绑定，**将切面真正的模块化，比较直观，方便理解。**   使用注解通常都会节省开发人员的配置时间和难度，节省时间。另外对于Spring AOP来说，使用注解配置还有它独特的好处，比如给一个Pointcut切入点命名，然后可以在其它切入点中复用该切入点，这是XML配置中没有的功能！   实际上，这些注解实际（下面会讲）是Aspectj 框架在第五版本结合Java5引入的新特性。Spring AOP非常机智的和Aspectj框架采用相同的注解样式（即同样的注解），并且使用AspectJ框架提供的解析和匹配切入点表达式的库，但是，AOP运行时仍然是纯Spring AOP动态代理机制，并且不依赖于AspectJ的编译器或weaver。因此，**使用注解还有一个好处就是可以在Spring AOP和Aspectj框架之间无缝切换，方便后续功能升级（如果需要超过Spring AOP的功能，虽然大概率不会出现）！**   **总的来说，Spring团队推荐基于注解的Spring AOP配置！**


  基于注解所需的依赖和基于XML所需的依赖一致，其中spring-context包含了Spring IoC、Spring AOP等核心依赖，而aspectjweaver则是AspectJ框架的依赖，Spring使用该依赖来解析AspectJ的切入点表达式语法，以及AOP的注解支持。

```java
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
        <scope>compile</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
    <!--用于解析AspectJ的切入点表达式语法，以及AOP的注解支持-->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>${aspectjweaver}</version>
    </dependency>
</dependencies>
```


  要想使用AOP的相关注解，我们需要开启注解支持。可以使用XML或Java风格的配置启用注解自动配置的支持。

## 3.1 基于XML的配置

  引入AOP 相关schema文件，加入aop命名空间&lt; aop:aspectj-autoproxy /&gt;标签即可。然后Spring将会查找被@Aspect注解标注的bean，这表明它是一个切面bean，然后就会进行AOP的自动配置，比如里面的通知和切入点表达式解析。   它的proxy-target-class属性用于指定使用哪一种代理，默认为false，表示先使用JDK代理，不符合要求再使用Cglib代理，改成true就表示使用CGlib代理。

```java
<?xml version="1.0" encoding="UTF-8"?>
<!--加入AOP 相关schema-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启注解自动配置-->
    <aop:aspectj-autoproxy/>
</beans>
```

## 3.2 基于Java的配置

  和IoC的注解支持可以使用注解Java配置开启一样，AOP的注解支持同样可以使用注解Java配置开启，从而彻底舍弃XML配置文件。

```java
/**
 * @author lx
 */
@Configuration
@ComponentScan("com.spring.aop.annotation")
//Java配置开启AOP注解支持
@EnableAspectJAutoProxy
public class MainConfiguration {
   
}
```

  @Configuration和@ComponentScan我们此前讲过，这是IoC的Java注解配置，重要的就是@EnableAspectJAutoProxy注解，该注解用于开启Spring AOP的注解自动配置支持！   它的proxyTargetClass属性用于指定使用哪一种代理，默认为false，表示先使用JDK代理，不符合要求再使用Cglib代理，改成true就表示使用CGlib代理。




  新建一个maven项目，加入上面的依赖，并且通过注解开启AOP注解支持： ![img](https://img-blog.csdnimg.cn/20200918164111158.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   新建一个类，添加@Component、@Aspect注解，将该类作为切面类。 ![img](https://img-blog.csdnimg.cn/20200918164128756.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

```java
/**
 * 一个切面类
 *
 * @author lx
 */
@Component
@Aspect
public class FirstAopAnnotationAspect {
    }
```

  然后定义一些Pointcut切入点和通知advice（都绑定到方法上），并将它们结合，形成一个个的切面，此时一个的切面类就定义好了：

```java
/**
 * 一个切面类
 *
 * @author lx
 */
@Component
@Aspect
public class FirstAopAnnotationAspect {
   
    /**
     * 一个切入点，绑定到方法上
     */
    @Pointcut("execution(* *(..))")
    public void pointCut() {
   
    }


    /**
     * 一个前置通知，绑定到方法上
     */
    @Before("pointCut()")
    public void before() {
   
        System.out.println("----before advice----");
    }

    /**
     * 一个后置通知，绑定到方法上
     */
    @AfterReturning("pointCut()")
    public void afterReturning() {
   
        System.out.println("----afterReturning advice----");
    }
}
```


  加入目标类： ![img](https://img-blog.csdnimg.cn/20200918164315150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

```java
/**
 * 一个目标类
 *
 * @author lx
 */
@Component
public class FirstAopAnnotationTarget {
   

    /**
     * 该方法将被增强
     */
    public void target() {
   
        System.out.println("target");
    }
}
```

  测试：

```java
@Test
public void firstAnnotationTest(){
   
    //1 获取容器
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfiguration.class);
    //2 获取bean
    FirstAopAnnotationTarget ft = ac.getBean("firstAopAnnotationTarget", FirstAopAnnotationTarget.class);
    //3 调用目标方法
    ft.target();
}
```

  结果如下：

```java
----before advice----
target
----afterReturning advice----
```

  目标方法如预期被成功增强，到此Spring AOP注解配置第一例结束！


  @Aspect注解对应着XML配置中的&lt; aop:aspect &gt;标签。   在开启Spring AOP注解自动配置支持以后，Spring 容器将会在扫描包路径下（@ComponentScan配置的路径）扫描任何具有@Aspect注解的bean，该类被当作切面类并且用于自动配置Spring AOP。但是，请注意@Aspect注解不会被Spring组件扫描工具自动检测，因为我们还需要为切面类添加一个@Component（根据语义，可以选择@Repository、@Service和@Controller）注解，将其注册到Spring 容器中。   切面类可以有自己的方法和字段，就和普通类一样，它们还可以包含pointcut切入点、advice通知和introduction声明，这些都是通过方法来绑定的。   在Spring AOP中，切面类本身不能成为其他切面通知的目标，类上面标注了@Aspect注解之后，该类的bean将从AOP自动配置bean中排除，因此，切面类里面的方法都不能被代理。

  如下案例，有两个切面类，它们的切入点表达式都是匹配任何类的任何方法：

```java
/**
 * 一个切面类
 *
 * @author lx
 */
@Component
@Aspect
public class AspectMethod1 {
   
    /**
     * 一个切入点，绑定到方法上，该切入点匹配“任何方法”
     */
    @Pointcut("execution(* *(..))")
    public void pointCut() {
   
    }

    /**
     * 一个前置通知，绑定到方法上
     */
    @Before("pointCut()")
    public void before() {
   
        System.out.println("----before advice----");
    }

    /**
     * 切面类的方法，无法被增强
     */
    public void aspectMethod1() {
   
        System.out.println("aspectMethod1");
    }
}
//-----------------
/**
 * 一个切面类
 *
 * @author lx
 */
@Component
@Aspect
public class AspectMethod2 {
   
    /**
     * 一个切入点，绑定到方法上，该切入点匹配“任何方法”
     */
    @Pointcut("execution(* *(..))")
    public void pointCut() {
   
    }

    /**
     * 一个前置通知，绑定到方法上
     */
    @Before("pointCut()")
    public void before() {
   
        System.out.println("----before advice----");
    }

    /**
     * 切面类的方法，无法被增强
     */
    public void aspectMethod2() {
   
        System.out.println("aspectMethod2");
    }
}
```

  测试：

```java
@Test
public void aspectMethod() {
   
    //1 获取容器
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfiguration.class);
    //2 获取切面类bean
    AspectMethod1 am1 = ac.getBean("aspectMethod1", AspectMethod1.class);
    AspectMethod2 am2 = ac.getBean("aspectMethod2", AspectMethod2.class);
    //2 调用切面类方法，并不会被代理
    am1.aspectMethod1();
    am2.aspectMethod2();
}
```

  结果如下，它们的方法并没有被代理：

```java
aspectMethod1
aspectMethod2
```


  @Pointcut对应着XML配置中的&lt; aop:pointcut &gt;标签，都用来定义一个切入点，用来控制在什么地方、什么情况下执行通知，切入点表达式的语法都是一样的，我们在上一篇基于XML的配置中已经详细介绍了切入点表达式的语法，在此不再赘述。   @Pointcut注解标注在一个切面类的方法上，该方法名就是该切入点的名字，然后可以在通知中通过名字引用该切入点（要带括号），也可以将多个切入点通过名字引用使用&amp;&amp;、||、!组合起来成为一个新的切入点，这是基于XML的配置所不具备的功能。   另外，在处理企业应用程序时，为了更加模块化，通常可以将所有的切入点定义在一个切面类中，该类专门用于存放切入点，其他切面类用于存放通知，切入点在所有切面类中是共享的！

```java
/**
 1. 切面类，专门用于存放切入点定义
 */
@Aspect
@Component
public class AspectPointcut {
   

    /**
     * 任何方法的切入点
     */
    @Pointcut("execution(* *(..))")
    public void pointcut1() {
   
    }

    /**
     * AspectPointcutTarget类的target1方法的切入点
     */
    @Pointcut("execution(* *..AspectPointcutTarget.target1(..))")
    public void pointcut2() {
   
    }

    /**
     * AspectPointcutTarget类的任何方法的切入点
     */
    @Pointcut("within(*..AspectPointcutTarget)")
    public void pointcut3() {
   
    }

    /**
     * 第一个参数是String类型的任何方法的切入点
     */
    @Pointcut("args(String,..)")
    public void pointcut4() {
   
    }

    /**
     * 通过切入点名称组合切入点
     * <p>
     * AspectPointcutTarget类的第一个参数是String类型的任何方法的切入点
     */
    @Pointcut("pointcut3() && pointcut4()")
    public void pointcut5() {
   
    }

    //…………
}
```


  advice通知同样可以使用注解声明，并且绑定到一个方法上，一共有五种通知注解。下面的注解分别对应着XML配置中的&lt; aop:before &gt;、&lt; aop:after-returning &gt;、&lt; aop:after-throwing &gt;、&lt; aop:after &gt;、&lt; aop:around &gt;这几个标签，它们的性质都是类似的，我们在基于XML的配置中已经详细讲解过了，在此不再赘述！

1. @Before用于配置前置通知before advice，在切入点方法执行之前执行。 
2. @AfterReturning标签用于配置后置通知after-returning advice。在切入点方法正常执行之后可能会执行。 
3. @AfterThrowing标签用于配置异常通知after-throwing advice。在前置通知、切入点方法和后置通知中抛出异常之后可能会执行。 
4. @After标签用于配置前置通知after advice。无论切入点方法是否正常执行，它都会在其后面执行。 
5. @Around标签用于配置环绕通知around advice。

  如下案例，用于测试注解配置通知的使用：

```java
/**
 * 切面类
 */
@Aspect
@Component
public class AspectAdvice {
   
    /**
     * 切入点，用于筛选通知将在哪些方法上执行
     */
    @Pointcut("execution(* AspectAdviceTarget.target())")
    public void pt() {
   
    }


    //五种通知

    /**
     * 前置通知
     * 可以通过名称引用定义好的切入点
     */
    @Before("pt()")
    public void before() {
   
        System.out.println("---before---");
    }

    /**
     * 后置通知
     * 也可以定义自己的切入点
     */
    @AfterReturning("execution(* AspectAdviceTarget.target())")
    public void afterReturning() {
   
        System.out.println("---afterReturning---");
    }

    /**
     * 异常通知
     */
    @AfterThrowing("pt()")
    public void afterThrowing() {
   
        System.out.println("---afterThrowing---");
    }

    /**
     * 最终通知
     */
    @After("pt()")
    public void after() {
   
        System.out.println("---after---");
    }

//    /**
//     * 环绕通知
//     *
//     * @param pjp 连接点
//     */
//    @Around("pt()")
//    public Object around(ProceedingJoinPoint pjp) {
   
//        System.out.println("---around---");
//        try {
   
//            System.out.println("前置通知");
//            //调用目标方法
//            Object proceed = pjp.proceed();
//            System.out.println("后置通知");
//            return proceed;
//        } catch (Throwable throwable) {
   
//            System.out.println("异常通知");
//            throwable.printStackTrace();
//            return null;
//        } finally {
   
//            System.out.println("最终通知");
//        }
//    }
}
//--------------
/**
 * 目标类
 *
 * @author lx
 */
@Component
public class AspectAdviceTarget {
   

    public int target() {
   
        System.out.println("target");
        //抛出一个异常
        //int i = 1 / 0;
        return 33;
    }
}
```

  测试：

```java
@Test
public void aspectAdvice() {
   
    //1 获取容器
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfiguration.class);
    //2 获取目标类bean
    AspectAdviceTarget aspectAdviceTarget = ac.getBean("aspectAdviceTarget", AspectAdviceTarget.class);
    //2 调用目标方法
    aspectAdviceTarget.target();
}
```

  关闭环绕通知，正常情况下：

```java
---before---
target
---afterReturning---
---after---
```

## 7.1 参数绑定

  在基于XML的配置中，我们说过可以将各种参数、返回值、异常信息传递给通知方法，在基于注解的配置中同样可以实现。

### 7.1.1 返回值和异常绑定

  @AfterReturning注解中的returning属性，它的值是后置通知方法中的参数名，用来将切入点方法的返回值绑定到该参数上。   @AfterThrowing注解中的throwing属性，它的值是异常通知方法中的参数名，用来将前置通知、切入点方法、后置通知执行过程中抛出的异常绑定到该参数上。另外，这里的最后抛出的异常同样遵循后者优先的覆盖原则，详见基于XML配置的文章！   参数绑定时，类型应该一致或者兼容！

  如下案例，测试返回值和异常参数绑定：

```java
/**
 * 切面类
 * @author lx
 */
@Component
@Aspect
public class AspectArgument {
   

    @Pointcut("within(AspectArgumentTarget)")
    public void pt() {
   
    }

    /**
     * 后置通知，获取切入点方法的返回值作为参数
     *
     * @param date 切入点方法的返回值
     */
    @AfterReturning(value = "pt()", returning = "date")
    public void afterReturning(Date date) {
   
        System.out.println("----afterReturning----");
        System.out.println("Get the return value : " + date);
        System.out.println("----afterReturning----");
    }

    /**
     * 异常通知，获取抛出的异常作为参数
     *
     * @param e 前置通知、切入点方法、后置通知执行过程中抛出的异常
     */
    @AfterThrowing(value = "pt()", throwing = "e")
    public void afterThrowing(Exception e) {
   
        System.out.println("----afterThrowing----");
        System.out.println("Get the exception : " + Arrays.toString(e.getStackTrace()));
        System.out.println("----afterThrowing----");
    }
}
//------------------
@Component
public class AspectArgumentTarget {
   

    public Date target() {
   
        Date date = new Date();
        System.out.println("target return: " + date);
        //抛出一个异常
        //int i=1/0;
        return date;
    }
}
```

  测试：

```java
@Test
public void aspectArgument() {
   
    //1 获取容器
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfiguration.class);
    //2 获取目标类bean
    AspectArgumentTarget aspectArgumentTarget = ac.getBean("aspectArgumentTarget", AspectArgumentTarget.class);
    //2 调用目标方法
    Date target = aspectArgumentTarget.target();
    System.out.println(" returned value: "+target);
}
```

  正常情况下的结果：

```java
target return: Fri Sep 18 00:00:41 CST 2020
----afterReturning----
Get the return value : Fri Sep 18 00:00:41 CST 2020
----afterReturning----
 returned value: Fri Sep 18 00:00:41 CST 2020
```

  抛出异常时的结果：

```java
target return: Fri Sep 18 00:01:33 CST 2020
----afterThrowing----
Get the exception : [com.spring.aop.annotation.aspectargument.AspectArgumentTarget.target(AspectArgumentTarget.java:14)//……………………
----afterThrowing----
//最终抛出的异常
java.lang.ArithmeticException: / by zero
at com.spring.aop.annotation.aspectargument.AspectArgumentTarget.target(AspectArgumentTarget.java:14)
//……………………
```

### 7.1.2 其他参数绑定

  其他参数绑定，需要使用切入点表达式的语法，包括切入点方法参数args、代理对象this、目标对象target，和相关注解（@within, @target, @annotation, 和 @args）都可以绑定到通知方法的参数中。这些内容我们在基于XML的配置部分已经详细讲解了，在此不再赘述！   在通知方法的第一个参数同样可以默认设置一个JoinPoint，用来获取当前连接点的信息。   绑定参数的时候，会通过参数名字注入参数，如果类型不兼容，那么将不会执行该通知。预定义的切入点，如果要绑定参数，那么切入点绑定的方法同样必须有这些参数，并且参数名字同样要对应。

  如下案例，进行一些参数绑定测试：

```java
/**
 * 一些参数绑定测试
 */
@Component
@Aspect
public class AspectAttribute {
   
    /**
     * 使用args()传递参数值给通知方法的参数，args用于指定通知方法的参数名称对应切入点方法的第几个参数，这将会进行参数值的注入，如果类型不匹配，那么该通知将不会执行（不会抛出异常）
     * argNames属性也可以指定参数名，这个属性在使用注解时基本都可以省略
     */
    @Before(value = "execution(* com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(..)) && args(i,s,aspectAttributeTarget,..)", argNames = "joinPoint,i,s,aspectAttributeTarget")
    public void before(JoinPoint joinPoint, int i, String s, AspectAttributeTarget aspectAttributeTarget) {
   
        System.out.println("----------before attribute----------");
        System.out.println(joinPoint);
        System.out.println(i);
        System.out.println(s);
        System.out.println(aspectAttributeTarget);
        System.out.println("----------before attribute----------");

    }

    /**
     * 使用this()传递代理对象
     * 使用target()传递目标对象
     */
    @Before(value = "execution(* com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(..)) && this(aspectAttributeTargetAop) && target(aspectAttributeTarget)")
    public void before2(JoinPoint joinPoint, AspectAttributeTarget aspectAttributeTargetAop, AspectAttributeTarget aspectAttributeTarget) {
   
        System.out.println("----------before this&target----------");
        System.out.println(joinPoint);
        System.out.println("当前代理对象: " + aspectAttributeTargetAop.getClass());
        System.out.println("当前目标对象: " + aspectAttributeTarget.getClass());
        System.out.println("----------before this&target----------");

    }

    /**
     * 预定义的切入点。可以在Pointcut中定义一个传递参数的模版，这要求Pointcut绑定的方法同样具有参数
     * 在通知中引用这个Pointcut时，需要在指定参数位置传递所需的参数名（这个名字是通知方法的参数名）
     */
    @Pointcut(value = "execution(* com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(..)) && args(i,s,aspectAttributeTarget,..)")
    public void attribute(int i, String s, AspectAttributeTarget aspectAttributeTarget) {
   
    }

    /**
     * 引入预定义的切入点，绑定参数和返回值
     * 在通知中引用这个Pointcut时，需要在指定参数位置传递所需的参数名（这个名字是通知方法的参数名）
     * 使用args()传递参数值给通知方法的参数，如果args对应的名称一致，那么可以省略argNames属性（argNames属性用于确定参数名称）
     */
    @AfterReturning(value = "attribute(i,s,aspectAttributeTarget)", returning = "date")
    public void afterReturning(JoinPoint joinPoint, int i, String s, AspectAttributeTarget aspectAttributeTarget, Date date) {
   
        System.out.println("----------afterReturning attribute&returned value----------");
        System.out.println(joinPoint);
        System.out.println(i);
        System.out.println(s);
        System.out.println(aspectAttributeTarget);
        System.out.println(date);
        System.out.println("----------afterReturning attribute&returned value----------");
    }


    /**
     * 引入预定义的切入点，绑定参数和异常
     * 在通知中引用这个Pointcut时，需要在指定参数位置传递所需的参数名（这个名字是通知方法的参数名）
     * args()还可以传递我们所需要的参数而不是全部，参数位置也不一定需要和切入点方法的参数位置一致，只要参数名称对应
     */
    @AfterThrowing(value = "attribute(i1,*,aspectAttributeTarget1)", throwing = "e")
    public void afterThrowing(JoinPoint joinPoint, int i1, Exception e, AspectAttributeTarget aspectAttributeTarget1) {
   
        System.out.println("----------afterThrowing attribute&exception----------");
        System.out.println(joinPoint);
        System.out.println(i1);
        System.out.println(aspectAttributeTarget1);
        System.out.println("exception: " + e.getMessage());
        System.out.println("----------afterThrowing attribute&exception----------");
    }

    /**
     * 可以使用@annotation()绑定切入点方法的注解作为参数
     */
    @After(value = "execution(* com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(..)) && @annotation(description)")
    public void after(JoinPoint joinPoint, Description description) {
   
        System.out.println("----------after annotation----------");
        System.out.println(joinPoint);
        System.out.println("annotation: " + description);
        System.out.println("----------after annotation----------");
    }
    
    //……………………
}

//-------------

@Component
public class AspectAttributeTarget {
   

    @Description("描述")
    public Date target(int i, String s, AspectAttributeTarget aspectAttributeTarget) {
   
        Date date = new Date();
        System.out.println("target return: " + date);
        //抛出一个异常
        //int j=1/0;
        return date;
    }
}
```

  测试：

```java
@Test
public void aspectAttribute() {
   
    //1 获取容器
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfiguration.class);
    //2 获取目标类bean
    AspectAttributeTarget aspectAttributeTarget = ac.getBean("aspectAttributeTarget", AspectAttributeTarget.class);
    //2 调用目标方法
    AspectAttributeTarget aspectAttributeTarget1 = new AspectAttributeTarget();
    System.out.println("AspectAttributeTarget: " + aspectAttributeTarget1);
    Date target = aspectAttributeTarget.target(33, "参数", aspectAttributeTarget1);
    System.out.println(" returned value: " + target);
}
```

  正常结果如下，实现了参数绑定注入：

```java
AspectAttributeTarget: com.spring.aop.annotation.aspectattribute.AspectAttributeTarget@1151e434
----------before attribute----------
execution(Date com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(int,String,AspectAttributeTarget))
33
参数
com.spring.aop.annotation.aspectattribute.AspectAttributeTarget@1151e434
----------before attribute----------
----------before this&target----------
execution(Date com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(int,String,AspectAttributeTarget))
当前代理对象: class com.spring.aop.annotation.aspectattribute.AspectAttributeTarget$$EnhancerBySpringCGLIB$$d15713be
当前目标对象: class com.spring.aop.annotation.aspectattribute.AspectAttributeTarget
----------before this&target----------
target return: Fri Sep 18 10:18:44 CST 2020
----------afterReturning attribute&returned value----------
execution(Date com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(int,String,AspectAttributeTarget))
33
参数
com.spring.aop.annotation.aspectattribute.AspectAttributeTarget@1151e434
Fri Sep 18 10:18:44 CST 2020
----------afterReturning attribute&returned value----------
----------after annotation----------
execution(Date com.spring.aop.annotation.aspectattribute.AspectAttributeTarget.target(int,String,AspectAttributeTarget))
annotation: @com.sun.org.glassfish.gmbal.Description(key=, value=描述)
----------after annotation----------
 returned value: Fri Sep 18 10:18:44 CST 2020
```

## 7.2 通知顺序

  当同一个连接点方法中绑定了多个同一个类型的通知时，有时需要指定通知的执行顺序，在基于XML的配置中我们讲了&lt; aop:aspec t&gt;切面标签中可以使用order属性指定执行通知顺序，基于注解的配置也能执行通知执行顺序。   将通知类实现Ordered接口或者添加@Order注解，即可实现切面通知的排序。未指定order值时，默认值为Integer.MAX_VALUE。order越小的切面，其内部定义的前置通知越先执行，后置通知越后执行。相同的order的切面则无法确定它们内部的通知执行顺序，同一个切面内的相同类型的通知也无法确定执行顺序。

  如下案例，测试order对通知顺序的影响：

```java
/**
 * order测试
 * order越小的切面，其内部定义的前置通知越先执行，后置通知越后执行。
 * 相同的order的切面则无法确定它们内部的通知执行顺序，同一个切面内的相同类型的通知也无法确定执行顺序。
 */
@Component
public class AspectOrder {
   
    /**
     * 默认order为Integer.MAX_VALUE
     */
    @Component
    @Aspect
    public static class AspectOrder1 {
   

        @Pointcut("execution(* *..AspectOrderTarget.*())")
        public void pt() {
   
        }

        @Before("pt()")
        public void before() {
   
            System.out.println("-----Before advice 1-----");
        }

        @AfterReturning("pt()")
        public void afterReturning() {
   
            System.out.println("-----afterReturning advice 1-----");
        }

        @After("pt()")
        public void after() {
   
            System.out.println("-----after advice 1-----");
        }
    }

    /**
     * 使用注解
     */
    @Component
    @Aspect
    @Order(Integer.MAX_VALUE - 2)
    public static class AspectOrder2 {
   

        @Before("com.spring.aop.annotation.aspectorder.AspectOrder.AspectOrder1.pt()")
        public void before() {
   
            System.out.println("-----Before advice 2-----");
        }

        @AfterReturning("com.spring.aop.annotation.aspectorder.AspectOrder.AspectOrder1.pt()")
        public void afterReturning() {
   
            System.out.println("-----afterReturning advice 2-----");
        }

        @After("com.spring.aop.annotation.aspectorder.AspectOrder.AspectOrder1.pt()")
        public void after() {
   
            System.out.println("-----after advice 2-----");
        }
    }

    /**
     * 实现Ordered接口
     */
    @Component
    @Aspect
    public static class AspectOrder3 implements Ordered {
   

        @Before("com.spring.aop.annotation.aspectorder.AspectOrder.AspectOrder1.pt()")
        public void before() {
   
            System.out.println("-----Before advice 3-----");
        }

        @AfterReturning("com.spring.aop.annotation.aspectorder.AspectOrder.AspectOrder1.pt()")
        public void afterReturning() {
   
            System.out.println("-----afterReturning advice 3-----");
        }

        @After("com.spring.aop.annotation.aspectorder.AspectOrder.AspectOrder1.pt()")
        public void after() {
   
            System.out.println("-----after advice 3-----");
        }

        /**
         * @return 获取order
         */
        public int getOrder() {
   
            return Integer.MAX_VALUE - 1;
        }
    }
}

//----------------

/**
 * @author lx
 */
@Component
public class AspectOrderTarget {
   

    public void target() {
   
        System.out.println("target");
    }
}
```

  测试：

```java
@Test
public void aspectOrder() {
   
    //1 获取容器
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfiguration.class);
    //2 获取目标类bean
    AspectOrderTarget aspectOrderTarget = ac.getBean("aspectOrderTarget", AspectOrderTarget.class);
    //2 调用目标方法
    aspectOrderTarget.target();
}
```

  结果如下：

```java
-----Before advice 2-----
-----Before advice 3-----
-----Before advice 1-----
target
-----afterReturning advice 1-----
-----after advice 1-----
-----afterReturning advice 3-----
-----after advice 3-----
-----afterReturning advice 2-----
-----after advice 2-----
```


  @DeclareParents对应基于XML配置中的&lt; aop:declare-parents &gt;标签，即Introduction（引介）。在不修改源代码的前提下，Introduction可以在运行期为类动态地添加一些额外的方法或属性。   @DeclareParents绑定到一个切面类的属性上，该属性的类型就是新增的功能接口，实际上Spring创建的目标类的代理类会同时实现这个接口（无论是JDK还是CGlib），因此可以多一些功能。@DeclareParents还有两个属性：

1. value：需要增强的的类扫描路径，该路径下的被Spring管理的bean都将被增强。 
2. defaultImpl：一个实现了新增的功能接口的类，作为新的增强方法的默认实现类。

  如下案例，用来测试@DeclareParents：   基本功能类：

```java
/**
 * @author lx
 */
@Component
public class BasicFunction {
   
    public void get(){
   
        System.out.println("get");
    }

    public void update(){
   
        System.out.println("update");
    }
}
```

  新增功能和默认实现：

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

  设置Introduction：

```java
@Component
@Aspect
public class AspectIntroduction {
   

    /**
     * value: 需要增强的的类扫描路径，该路径下的被Spring管理的bean都将被增强
     * DeclareParents绑定的属性所属类型AddFunction: AddFunction
     * defaultImpl: 增强接口的方法的默认实现
     */
    @DeclareParents(value = "*..BasicFunction", defaultImpl = AddFunctionImpl.class)
    public static AddFunction addFunction;
}
```

  测试：

```java
@Test
public void aspectIntroduction() {
   
    //1 获取容器
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MainConfiguration.class);
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
    System.out.println(addFunction.str);
}
```

  结果如下，实现了功能的增强：

```java
class com.spring.aop.annotation.introductions.BasicFunction$$EnhancerBySpringCGLIB$$b389fa67
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


  **本次我们学习了基于注解的Spring AOP配置，在此前基于XML的配置的基础上，使用我们可以非常快速的学会使用注解的Spring AOP配置。某些细节内容这两者都是一样的，这些东西都放在在上一篇基于XML的AOP配置中讲过了。**   我们可以在一个项目中同时使用基于XML的配置和基于注解的配置，但是通常没必要这么做。   实际上，在Spring程序中很多地方都隐式的使用了AOP代理机制，比如对于@Configuration标注的类，将默认创建一个CGlib代理类，这样就能代理它内部的@Bean方法，当@Bean方法互相调用时，将会返回同一个对象，这是其他组件注解比如@Component所不具备的功能。再比如Spring的自动事物管理，传播行为，@Transactional注解等也是使用的AOP代理机制来实现的，后面我们会介绍Spring的自动事务管理机制。

**相关文章：**   Spring官网： [https://docs.spring.io/](https://docs.spring.io/)   AOP XML： [Spring 5.x 学习(5)—两万字的基于XML的Spring AOP配置全解](https://blog.csdn.net/weixin_43767015/article/details/108632778)。

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

