

>  基于最新Spring 5.x，详细介绍了Spring的@Async异步任务的概念和使用方法，以及一些问题的解决办法！

  Spring异步任务机制非常的有用，特别是在那些记录日志、发端短信、发送邮件等等非核心的业务上面，或者用在一些系统内部任务上，可以优化代码结构，加快程序响应速度，提升用户体验。







  **Spring的异步任务机制，可以让调用者立即返回而无需等待任务（方法）完成，可以避免方法阻塞，提升响应效率，通常用于日志记录、发送邮件、短信等非核心业务中。我们只需要非常简单的配置即可使用Spring异步任务机制。**

## 1.1 开启异步任务支持

  我们可以使用注解或者XML配置的方式开启异步任务支持。

1. 对于注解的方式，一般我们将@EnableAsync注解添加到@Configuration配置类上面，表示开启异步任务支持。 
2. 对于XML的方式，我们使用&lt; task:annotation-driven/&gt;标签来开启异步任务支持。

## 1.2 任务执行器

  **Spring提供了TaskExecutor任务执行器抽象接口，这等同于JDK 5.0中的java.util.concurrent.Executor执行器，简单的说任务执行器就是用于执行各种任务（方法）。关于Executor，可以看看这篇文章：JUC—六万字的Executor线程池框架源码深度解析。**   不同的TaskExecutor有不同的执行策略，最常见的就是线程池执行器，当然还有其他类型的执行器，比如单线程执行器、同步执行器等等，因此不能笼统的将线程池和任务执行器划等号。Spring已经提供了各种版本的TaskExecutor实现，并且很多都是可配置的，通常来说我们无需自定义TaskExecutor实现。当我们想要使用的时候，只需要将这些bean定义注册到容器中即可。


  Spring异步任务最关键的就是多线程的使用，那么这就和上面的任务执行器关联起来了。我们**可以通过@Bean注解配置执行器或者基于XML自定义执行器bean。** 通常自定义的执行器都是采用ThreadPoolTaskExecutor类型，但是这里的执行器只要是基于Executor接口都可以执行异步任务，也就是说JDK中的线程池也行。   **基于XML的配置还可以使用< task:executor/>标签来快速定义一个ThreadPoolTaskExecutor类型的执行器，对应的线程前缀就是“执行器id-”。**

## 1.3 @Async异步任务

  **无论是通过注解还是XML的方式开启异步任务支持，对于异步任务（方法）本身，都使用@Async注解描述，没有XML的描述方式，这个注解也是Spring 3.0添加的注解。**

1. **@Async的语义仍然是通过代理来实现的**。可以是JDK的动态代理或者CGLIB代理，通过@EnableAsync注解的**proxyTargetClass**属性或者&lt; task:annotation-driven/&gt;标签的**proxy-target-class**属性来控制，默认为false，表示优先使用JDK代理，否则再尝试CGLIB代理，如果改为true，表示强制CGLIB代理。关于Spring AOP，我们此前的文章就讲过了使用和源码！ 
2. **被@Async标注的方法，称为异步方法，被@Async标注的类，它内部的所有方法都是异步方法，方法上的注解优先级最高，如果方法上存在@Async注解则直接使用该注解，如果没有，再查找类上的@Async注解。** 注意如果采用JDK代理则只能代理接口的方法；如果采用CGLIB代理则不能代理final、static、private的方法，并且类不能是final的；由于代理的限制，如果同一个类的方法相互调用，如果@Async方法不是在调用链首位，那么被调用的@Async方法不会生效，实际上，基于AOP的其他配置都不会生效，比如事物，因为里面的方法实际上是通过目标对象本身调用的，并且如果配置了@Async方法，那么这些种情况不能通过普遍的方式解决，比较有效的解决办法是可以在注入的属性**加上@Lazy注解**，或者将方法写到不同的类里面。如果@Async方法不能满足上面的要求，则还是通过调用线程去执行该方法，可能不会抛出异常，因此难以察觉； 
3. @Async注解标注的异步方法通常没有返回值，但是**可以有返回值**，这需要使用一个**Future**类型的对象来接收，它的泛型类型可以是实际返回值类型，通过Future.get()来获取真实返回值或者抛出异常。实际上还可以使用**ListenableFuture**、**CompletableFuture**来接收，这两个类作为异步获取结果类，更加适合与异步方法交互！ 
4. @Async如果与生命周期回调方法结合使用，比如@PostConstruct方法，那么在执行生命周期回调时，不会异步执行，因为它时通过目标对象直接执行的。 
5. **@Async的value可以指定一个我们自定义的执行器**的名字，这将导致对于该方法或者该类的@Async方法检是用执行执行器中的线程去执行，这样有利于区分各种异步任务，如果没有指定，那么将**查找默认执行器**： 
 <ol> 
  5. 首先是选择通过Java配置的AsyncConfigurer的getAsyncExecutor方法返回的执行器（该执行器不受到Spring管理，默认返回null）或者是通过&lt; task:annotation-driven/&gt;标签的executor属性指向的执行器（默认没有设置）。 
  5. 如果上面的方法都没获取到执行器，那么继续判断，在容器中查找如果有一个TaskExecutor类型的执行器，那么该执行器作为默认执行器；如果有多个或者没有任何一个，那么将查找beanName或者别名为“**taskExecutor**”类型为Executor的执行器作为默认执行器，如果还是找不到，那么将创建一个**SimpleAsyncTaskExecutor**类型的执行器作为默认执行器（该执行器不受到Spring管理）。 
 </ol>  
8. 对于具有Future返回值的异步方法，可以很方便的管理执行时的异常，因为产生的异常会被封装到Future中，同样是通过get方法抛出。对于无返回值的异步方法，默认异常处理程序是SimpleAsyncUncaughtExceptionHandler，它的逻辑是**直接在调用对应方法的线程中通过error级别日志打印这个异常信息（不是抛出异常）**。我们可以自定义一个异常处理器来处理这种异常，对于Java配置，可以通过**重写AsyncConfigurer的getAsyncUncaughtExceptionHandler方法返回一个AsyncUncaughtExceptionHandler处理器；对于XML配置可以设置< task:annotation-driven/>标签的exception-handler属性指向一个异常处理器bean定义。** 
9. **Spring不能为@Async注解标注的类解决setter方法和反射字段注解的循环依赖注入（包括自己注入自己）**，将会抛出：“……This means that said other beans do not use the final version of the bean……”异常，根本原因这个AOP代理对象不是使用通用的AbstractAutoProxyCreator的方法创建的，而是使用AsyncAnnotationBeanPostProcessor后处理器来创建的，Spring目前没有解决这个问题。解决办法是**在引入的依赖项上加一个@Lazy注解**，原理就是再给它加一层AOP代理……。而其他的，Spring可以解决比如由于事物或者通知方法创建的AOP代理的循环依赖。


## 2.1 基于XML的配置

  maven依赖：

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version> 5.2.8.RELEASE</version>
</dependency>
```

  一个测试类，com.spring.integration.tasks.xmltaskexecutor.AsyncMethod：

```java
public class AsyncMethod {
   

    @Async
    public void log() {
   
        System.out.println("-----log："+Thread.currentThread().getName());
    }

    @Async
    public void log2() {
   
        System.out.println("-----log2："+Thread.currentThread().getName());
        log3();
    }

    @Async
    public void log3() {
   
        System.out.println("-----log3："+Thread.currentThread().getName());
    }
}
```

  spring-config.xml配置文件，注意引入task的命名空间（idea可自动引入）：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd">


    <!--异步方法类-->
    <bean class="com.spring.integration.tasks.xmltaskexecutor.AsyncMethod" name="asyncTest"/>
    <!--开启异步任务支持-->
    <task:annotation-driven />

    <!--这里配置了一个Spring的ThreadPoolTaskExecutor标准执行器-->
    <bean class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor" name="threadPoolTaskExecutor">
        <property name="corePoolSize" value="5"/>
        <property name="maxPoolSize" value="5"/>
        <property name="keepAliveSeconds" value="5"/>
        <property name="queueCapacity" value="5"/>
        <property name="threadNamePrefix" value="threadPoolTaskExecutor-"/>
    </bean>

    <!--通过<task:executor/>标签快速配置一个Spring的ThreadPoolTaskExecutor标准执行器-->
    <task:executor id="executor" pool-size="100-10000" queue-capacity="10"/>


    <!--这里仅仅是配置了一个JDK的ThreadPoolExecutor-->
    <bean class="java.util.concurrent.ThreadPoolExecutor" name="taskExecutor">
        <constructor-arg name="corePoolSize" value="5"/>
        <constructor-arg name="maximumPoolSize" value="6"/>
        <constructor-arg name="keepAliveTime" value="5"/>
        <constructor-arg name="unit" value="SECONDS"/>
        <constructor-arg name="workQueue">
            <bean class="java.util.concurrent.LinkedBlockingQueue"/>
        </constructor-arg>
    </bean>
</beans>
```

  可以看到，我们配置了三个不同的执行器，我们来测试一下：

```java
public class XmlTaskExecutorTest {
   
    
    public static void main(String[] args) {
   
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
        AsyncMethod asyncMethod = ac.getBean(AsyncMethod.class);
        System.out.println(asyncMethod.getClass());
        System.out.println("--------" + Thread.currentThread().getName() + "--------");
        asyncMethod.log();
        asyncMethod.log2();
    }
}
```

  结果如下：

```java
class com.spring.integration.tasks.xmltaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$5da972aa
--------main--------
-----log2：pool-1-thread-2
-----log：pool-1-thread-1
-----log3：pool-1-thread-2
```

  可以看到，成功的进行了异步调用，并且很明显，通过线程名称“pool-x-thread-y”可知采用了JDK的ThreadPoolExecutor，即“taskExecutor”作为默认执行器。如果我们注释其中一个TaskExecutor，比如将threadPoolTaskExecutor这个执行器的配置注释掉，再次测试结果如下：

```java
class com.spring.integration.tasks.xmltaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$62c5372c
--------main--------
-----log：executor-1
-----log2：executor-2
-----log3：executor-2
```

  由于只有一个TaskExecutor，那么将使用这个TaskExecutor作为默认执行器，也就是“executor”。如果我们在&lt; task:annotation-driven/&gt;标签中添加executor属性，值为“taskExecutor”，那么表示将“taskExecutor”这个执行器作为默认执行器，再次测试，结果如下：

```java
class com.spring.integration.tasks.xmltaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$5ecd4858
--------main--------
-----log：pool-1-thread-1
-----log2：pool-1-thread-2
-----log3：pool-1-thread-2
```

  我们最后将注释放开，并且删除executor属性，恢复到最开始的状态，然后将JDK的ThreadPoolExecutor的名字改为其他值，比如“taskExecutor1”，再次测试，结果如下：

```java
class com.spring.integration.tasks.xmltaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$5da972aa
--------main--------
-----log：SimpleAsyncTaskExecutor-1
-----log2：SimpleAsyncTaskExecutor-2
-----log3：SimpleAsyncTaskExecutor-2
```

  可以看到，Spring采用了最后的策略，即内部创建一个SimpleAsyncTaskExecutor作为默认执行器，并且还有一段警告日志输出：

```java
More than one TaskExecutor bean found within the context, and none is named 'taskExecutor'. 
Mark one of them as primary or name it 'taskExecutor' (possibly as an alias) in order to use it for async processing: [threadPoolTaskExecutor, executor]
```

  它的意思就是没有手动设置默认执行器，并且存在多个TaskExecutor类型的执行器，并且没有名为“taskExecutor”的执行器，那么这时就会使用最后的策略！

## 2.2 基于注解的配置

  基于注解的配置更加常用！   一个测试类，com.spring.integration.tasks.anntaskexecutor.AsyncMethod：

```java
@Component
public class AsyncMethod {
   

    @Async
    public void log() {
   
        System.out.println("-----log："+Thread.currentThread().getName());
    }

    @Async
    public void log2() {
   
        System.out.println("-----log2："+Thread.currentThread().getName());
        log3();
    }

    @Async
    public void log3() {
   
        System.out.println("-----log3："+Thread.currentThread().getName());
    }
}
```

  参数组件类，com.spring.integration.tasks.anntaskexecutor.ConfigurationStart：

```java
@Configuration
@EnableAsync
@ComponentScan
public class ConfigurationStart {
   

    private final LongAdder longAdder = new LongAdder();

    /**
     * 这里仅仅是配置了一个JDK的ThreadPoolExecutor
     */
    @Bean
    public Executor taskExecutor() {
   

        return new ThreadPoolExecutor(3, 5, 3, TimeUnit.SECONDS, new LinkedBlockingQueue<>(), r -> {
   
            longAdder.increment();
            //线程命名
            return new Thread(r, "JDK线程-" + longAdder.longValue());
        });
    }

    /**
     * 这里仅仅是配置了一个Spring的ThreadPoolTaskExecutor
     */
    @Bean
    public ThreadPoolTaskExecutor threadPoolTaskExecutor1() {
   
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //配置核心线程数
        executor.setCorePoolSize(5);
        //配置最大线程数
        executor.setMaxPoolSize(10);
        //配置队列大小
        executor.setQueueCapacity(800);
        //配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix("threadPoolTaskExecutor1-");
        // rejection-policy：拒绝策略，由调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        return executor;
    }


    /**
     * 这里仅仅是配置了一个Spring的ThreadPoolTaskExecutor
     */
    @Bean
    public ThreadPoolTaskExecutor threadPoolTaskExecutor2() {
   
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //配置核心线程数
        executor.setCorePoolSize(5);
        //配置最大线程数
        executor.setMaxPoolSize(10);
        //配置队列大小
        executor.setQueueCapacity(800);
        //配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix("threadPoolTaskExecutor2-");
        // rejection-policy：拒绝策略，由调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        return executor;
    }
}
```

  上面这些配置和基于XML的配置差不多，只不过改成了更方便的注解而已，我们测试一下：

```java
public class AnnTaskExecutorTest {
   

    public static void main(String[] args) {
   
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(ConfigurationStart.class);
        AsyncMethod asyncMethod = ac.getBean(AsyncMethod.class);
        System.out.println(asyncMethod.getClass());
        System.out.println("--------" + Thread.currentThread().getName() + "--------");
        asyncMethod.log();
        asyncMethod.log2();
    }

}
```

  结果如下，可以预料到将会采用JDK的执行器：

```java
class com.spring.integration.tasks.anntaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$7abfce8
--------main--------
-----log2：JDK线程-2
-----log：JDK线程-1
-----log3：JDK线程-2
```

  如果我们添加一个AsyncConfigurer的实现com.spring.integration.tasks.anntaskexecutor.MyAsyncConfigurer，并且重写getAsyncExecutor方法，返回一个自定义的Executor：

```java
@Component
public class MyAsyncConfigurer implements AsyncConfigurer {
   
    @Override
    public Executor getAsyncExecutor() {
   
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //配置核心线程数
        executor.setCorePoolSize(5);
        //配置最大线程数
        executor.setMaxPoolSize(10);
        //配置队列大小
        executor.setQueueCapacity(800);
        //配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix("myAsyncConfigurer-");
        // rejection-policy：拒绝策略，由调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        /*
         * 注意，这里配置的ThreadPoolTaskExecutor不会被Spring管理，因此需要手动initialize初始化
         */
        executor.initialize();
        return executor;
    }
}
```

  再次测试，很明显getAsyncExecutor方法返回的执行器优先级最高：

```java
class com.spring.integration.tasks.anntaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$5afb4f00
--------main--------
-----log2：myAsyncConfigurer-2
-----log3：myAsyncConfigurer-2
-----log：myAsyncConfigurer-1
```

  当然，Spring推荐我们为每一个@Async指定一个执行器：

```java
@Component
public class AsyncMethod {
   

    @Async("taskExecutor")
    public void log() {
   
        System.out.println("-----log："+Thread.currentThread().getName());
    }

    @Async("threadPoolTaskExecutor1")
    public void log2() {
   
        System.out.println("-----log2："+Thread.currentThread().getName());
        log3();
    }

    @Async("threadPoolTaskExecutor2")
    public void log3() {
   
        System.out.println("-----log3："+Thread.currentThread().getName());
    }
}
```

  再次测试，很明显，将会使用我们指定的执行器：

```java
class com.spring.integration.tasks.anntaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$5afb4f00
--------main--------
-----log2：threadPoolTaskExecutor1-1
-----log3：threadPoolTaskExecutor1-1
-----log：JDK线程-1
```

  **经过多次测试，我们会发现log3()方法虽然标注了@Async注解，但是异步执行没有生效，因为该方法被log2()调用，而这两个方法都在同一个组件类中，这是由于代理的局限性导致的，这里我们首先尝试通过自己注入自己这种通用的方法来解决：**

```java
@Component
public class AsyncMethod {
   

    @Resource
    private AsyncMethod asyncMethod;

    @Async("taskExecutor")
    public void log() {
   
        System.out.println("-----log：" + Thread.currentThread().getName());
    }

    @Async("threadPoolTaskExecutor1")
    public void log2() {
   
        System.out.println("-----log2：" + Thread.currentThread().getName());
        //期望通过引入的代理对象调用log3()方法，这样就能保证同一个类中的AOP增强生效
        asyncMethod.log3();
    }

    @Async("threadPoolTaskExecutor2")
    public void log3() {
   
        System.out.println("-----log3：" + Thread.currentThread().getName());
    }

}
```

  启动，发现直接报错，这就是前面说的异常：

```java
Error creating bean with name 'asyncMethod': Bean with name 'asyncMethod' has been injected into other beans [asyncMethod] in its raw version as part of a circular reference, but has eventually been wrapped. 
This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.
```

  解决办法前面也给了，那就是加一个@Lazy注解，再添加一层代理，这里将注入代理对象的代理对象：

```java
@Component
public class AsyncMethod {
   

    //加入@Lazy注解，再添加一层代理，这里将注入代理对象的代理对象
    @Lazy
    @Resource
    private AsyncMethod asyncMethod;

    @Async("taskExecutor")
    public void log() {
   
        System.out.println("-----log：" + Thread.currentThread().getName());
    }

    @Async("threadPoolTaskExecutor1")
    public void log2() {
   
        System.out.println("-----log2：" + Thread.currentThread().getName());
        //期望通过引入的代理对象调用log3()方法，这样就能保证同一个类中的AOP增强生效
        asyncMethod.log3();
    }

    @Async("threadPoolTaskExecutor2")
    public void log3() {
   
        System.out.println("-----log3：" + Thread.currentThread().getName());
    }

}
```

  再次测试，发现每个方法都是用不同的线程，通过@Lazy注解可以比较优雅的解决了@Async方法互相调用不生效的问题，这种方法也可以解决@Async方法的普通AOP增强的同样问题。关于@Lazy注解，我们此前讲过了源码，可以去看： [Spring 5.x 源码(6)—IoC容器初始化(6)—四万字的refresh源码深度解析(5)](https://blog.csdn.net/weixin_43767015/article/details/109359065)这篇文章。

```java
class com.spring.integration.tasks.anntaskexecutor.AsyncMethod$$EnhancerBySpringCGLIB$$e350bc0f
--------main--------
-----log：JDK线程-1
-----log2：threadPoolTaskExecutor1-1
-----log3：threadPoolTaskExecutor2-1
```

  **如果一个类只有事物或者普通AOP增强，那么不必加@Lazy注解，直接自己注入自己就能解决自己的方法互相调用时增强不生效的问题！**

## 2.3 返回值和异常处理

  **异步方法可以有返回值，需要将异步方法的返回值封装到Future接口中，这里的Future是多线程应用程序中一种常见的设计模式，它的核心思想是：提交任务之后立即返回一个Future类型的结果，提交的任务会被一个新的线程异步执行，不会阻塞调用线程，从而调用线程可以继续执行后面的逻辑，而后续我们也可以通过一系列方法获取Future中任务执行完毕之后真实的返回值，一些Future实现还能添加回调方法。**   **Future的实现有很多，常见的有如下：**


  **异步方法支持三种返回值类型（排除void），这个源码在AsyncExecutionAspectSupport的doSubmit方法中：**

1. CompletableFuture：首先判断如果返回值类型是CompletableFuture及其子类，那么最终会默认返回一个Spring为我们创建的CompletableFuture对象； 
2. ListenableFuture：其次判断如果返回值类型是ListenableFuture及其子类，那么最终会默认返回一个Spring为我们创建的ListenableFutureTask对象； 
3. Future：随后判断如果异步方法返回值类型是Future及其子类，那么最终会默认返回一个Spring为我们创建的FutureTask对象； 
4. 最后，如果以上判断都不满足，即如果异步方法指定了返回其它类型，那么最终将**返回一个null。正常返回时，返回的结果对象和我们在方法中返回的对象也不是同一个。**

  **具有返回值的异步方法执行过程中产生的异常会被封装到Future中，因此很方法方便处理，它的get()方法就能抛出在执行过程中捕获的异常，而对于高级的Future，比如CompletableFuture和ListenableFuture则可以注册异常处理函数。**   对于无返回值的异步方法，异常不能被封装，也不会被直接抛出，而是需要通过异常处理器处理，**默认异常处理器是SimpleAsyncUncaughtExceptionHandler，它的逻辑是直接在调用对应方法的线程中通过error级别日志打印这个异常信息（不是抛出异常）**。可以自定义一个异常处理器来处理抛出的异常，做出各种业务逻辑：**对于Java配置，可以通过重写AsyncConfigurer的getAsyncUncaughtExceptionHandler方法返回一个AsyncUncaughtExceptionHandler异常处理器；对于XML配置可以设置< task:annotation-driven/>标签的exception-handler属性指向一个异常处理器bean定义。**   如下案例，有一个测试类com.spring.integration.tasks.returnandexc.AsyncMethod：

```java
@Component
public class AsyncMethod {
   
    /**
     * 返回Future
     */
    @Async
    public Future<Integer> future(int i) throws InterruptedException {
   
        System.out.println("-----执行future方法的线程：" + Thread.currentThread().getName());
        Thread.sleep(2000);
        ListenableFuture<Integer> integerListenableFuture = AsyncResult.forValue(i);
        System.out.println("方法中的Future :" + integerListenableFuture);
        return integerListenableFuture;
    }
    
    /**
     * 返回CompletableFuture
     */
    @Async
    public CompletableFuture<Integer> completableFuture(int i) throws InterruptedException {
   
        System.out.println("-----执行completableFuture方法的线程：" + Thread.currentThread().getName());
        Thread.sleep(2000);
        CompletableFuture<Integer> integerCompletableFuture = CompletableFuture.completedFuture(i);
        System.out.println("方法中的CompletableFuture :" + integerCompletableFuture);
        //int j=1/0;
        return integerCompletableFuture;
    }
    
    /**
     * 返回ListenableFuture
     */
    @Async
    public ListenableFuture<Integer> listenableFuture(int i) throws InterruptedException {
   
        System.out.println("-----执行listenableFuture方法的线程：" + Thread.currentThread().getName());
        Thread.sleep(2000);
        ListenableFuture<Integer> integerListenableFuture = AsyncResult.forValue(i);
        System.out.println("方法中的ListenableFuture :" + integerListenableFuture);
        return integerListenableFuture;
    }

    /**
     * 无返回
     */
    @Async
    public void noReturn(int i) throws InterruptedException {
   
        System.out.println("-----noReturn：" + Thread.currentThread().getName());
        Thread.sleep(3000);
        //制造一个异常
        int j = 1 / 0;
    }

}
```

  启动组件类com.spring.integration.tasks.returnandexc.ConfigurationStart：

```java
@Configuration
@EnableAsync
@ComponentScan
@EnableAspectJAutoProxy
public class ConfigurationStart {
   

    private final LongAdder longAdder = new LongAdder();

    @Bean
    public Executor taskExecutor() {
   

        return new ThreadPoolExecutor(3, 5, 3, TimeUnit.SECONDS, new LinkedBlockingQueue<>(), r -> {
   
            longAdder.increment();
            //线程命名
            return new Thread(r, "JDK线程-" + longAdder.longValue());
        });
    }
}
```

  自定义异常处理：

```java
@Component
public class MyAsyncConfigurer implements AsyncConfigurer {
   

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
   
        return new AsyncUncaughtExceptionHandler() {
   
            /**
             * 自定义异常处理逻辑
             * @param ex 抛出的异常
             * @param method 抛出异常的方法
             * @param params 抛出异常的方法参数
             */
            @Override
            public void handleUncaughtException(Throwable ex, Method method, Object... params) {
   
                System.out.println("-----自定义异常处理-------");
                System.out.println(method.getName());
                System.out.println(ex.getMessage());
                System.out.println(Arrays.toString(params));
                System.out.println("-----自定义异常处理-------");
            }
        };
    }
}
```

  测试：

```java
public class TaskExecutorTest {
   

    public static void main(String[] args) throws ExecutionException, InterruptedException {
   
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(ConfigurationStart.class);

        AsyncMethod asyncMethod = ac.getBean(AsyncMethod.class);
        System.out.println(asyncMethod.getClass());
        System.out.println("--------" + Thread.currentThread().getName() + "--------");
        /*
         * 1 future
         */
        Future<Integer> future = asyncMethod.future(0);
        System.out.println("FutureTask中获取结果并进行操作的线程: " + Thread.currentThread().getName());
        //get()方法只能同步获取执行结果并进行其他操作，因此可能还是会阻塞调用线程，或者需要手动新开线程等待结果，功能比较简陋
        System.out.println(future.get() + 10);
        System.out.println("返回的Future :" + future);

        /*
         * 2 completableFuture
         */
        CompletableFuture<Integer> integerCompletableFuture = asyncMethod.completableFuture(1);
        //注册一个正常回调函数，当执行完毕时自动回调该函数，参数就是执行结果，这样就不必同步等待了
        integerCompletableFuture.thenApplyAsync(integer -> {
   
            System.out.println("CompletableFuture中获取结果并进行操作的线程:: " + Thread.currentThread().getName());
            System.out.println(integer + 10);
            return null;
        });
        //注册一个异常回调函数，当执行抛出异常时时自动回调该函数
        integerCompletableFuture.exceptionally(throwable -> {
   
            throwable.printStackTrace();
            return null;
        });

        System.out.println("返回的CompletableFuture :" + integerCompletableFuture);

        /*
         * 3 listenableFuture
         */
        ListenableFuture<Integer> integerListenableFuture = asyncMethod.listenableFuture(2);
        //注册一个回调函数，具有异常方法和正常方法，当正常执行完毕时自动回调onSuccess，参数就是执行结果，这样就不必同步等待了
        integerListenableFuture.addCallback(new ListenableFutureCallback<Integer>() {
   
            //执行异常自动回调，onFailure中抛出的异常被忽略
            @Override
            public void onFailure(Throwable o) {
   
                o.printStackTrace();
            }

            //执行成功自动回调，onSuccess中抛出的异常被忽略
            @Override
            public void onSuccess(Integer o) {
   
                System.out.println("ListenableFuture中获取结果并进行操作的线程: " + Thread.currentThread().getName());
                System.out.println(o + 10);
            }
        });
        System.out.println("返回的ListenableFuture :" + integerListenableFuture);


        /*
         * 3 noReturn
         */
        asyncMethod.noReturn(1);
    }
```


  对于Spring Boot应用，如果容器中不存在任何自定义的Executor类型的执行器，那么将在TaskExecutionAutoConfiguration这个自动配置类中默认创建一个ThreadPoolTaskExecutor类型的执行器，名为applicationTaskExecutor ，别名为taskExecutor，线程名为“task-xx”，它将作为默认的执行器，如果有至少一个执行器，那么将不会创建！   **Spring异步任务机制非常的有用，特别是在那些记录日志、发端短信、发送邮件等等非核心的业务上面，可以提升响应速度，提升用户体验。另外异步任务还常常与其他功能和并使用，比如异步定时任务等！**   **本次学习了Spring异步任务的概念和详细使用方法，以及一些问题的解决办法，在后续的文章中，我们将学习Spring异步任务的原理和源码！**

**相关文章：**    [https://spring.io/](https://spring.io/)    [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)    [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

