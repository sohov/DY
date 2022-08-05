

>  基于最新Spring 5.x，详细介绍了Spring的@EventListener事件发布机制的概念和使用方法，以及一些问题的解决办法！

  **事件发布机制，可以简单的理解为在系统达到某个状态或者进行某个操作的时候（这被称为事件源），导致某个事件被触发，对应的事件监听器会捕获到这个事件，明确系统达到了某个状态或者进行了某个操作，随后事件监听器进行一系列额外的操作。**   比如发送短信或者邮件的操作，通常在一系列前置方法完成之后触发，那么发送短信或者邮件的方法可以置于一个事件监听器内部，然后当前一系列前置方法完成，发布一个事件，该监听器将会受到这个事件，这表示可以触发发送短信或者邮件的操作。   **通常，事件发布机制用于代码解耦，异步事件还能提升响应速度，这类似于简单的消息队列，因此相比与真正的MQ，它的应用范围还是比较有限的，因为它的任务将会缓存到内存中，如果服务器重启或者挂了，那么没来得及执行的事件任务就没了。**







  Spring的ApplicationContext上下文容器中提供了一系列的事件发布机制，并通过ApplicationEvent类和ApplicationListener接口提供一系列默认的事件和监听器的实现。**Spring 事件发布机制，本质上就是基于标准的观察者设计模式（Observer design pattern）。**   Spring事件发布机制的关键类如下：

1. ApplicationEvent：应用程序事件抽象，用于触发某个监听器完成一系列操作，所有应用程序事件都扩展这个类，它是事件源和事件监听器的内部桥梁。继承了位于JDK核心rt.jar包中的Java的事件基类EventObject。 
2. ApplicationListener：监听器抽象，专门监听对应的事件，进而完成一系列操作。 
3. ApplicationEventPublisher：应用程序事件发布者，用于发布自定义事件。实际上，我们的ApplicationContext上下文容器就是一个ApplicationEventPublisher的实现类。 
4. ApplicationEventMulticaster：应用事件广播器，内部保存的所有监听器，用于进行ApplicationContext事件广播。ApplicationEventPublisher的publishEvent发布事件操作的内部就是委托ApplicationEventMulticaster来做的。通常使用者不必关心。

  **自 Spring 4.2 起，Spring事件发布机制得到显著增强，可以通过ApplicationEventPublisher发布任意自定义事件，并且事件不必再继承ApplicationEvent，它们自动包装为一个事件，并且还可以通过@EventListener注解监听事件。**   **Spring提供了一系列标准事件，也就是一系列容器状态相关的应用程序事件，它们都继承了ApplicationEvent。**



  **只需要很简单的配置即可使用Spring事件发布机制！**

1. **事件**：我们可以自定义事件。 
 <ol> 
  1. 常见的就是通过继承ApplicationEvent来实现自定义事件。 
  1. Spring 4.2之后，我们不必继承ApplicationEvent，ApplicationEventPublisher新增的一个publishEvent重载方法，该方法传递的Object参数即可，Spring将自动包装为一个**PayloadApplicationEvent**类型的事件（该类型也继承了ApplicationEvent）。 
 </ol>  
4. **事件发布者**：ApplicationContext上下文容器本身就实现了ApplicationEventPublisher接口，也就是事件发布者，因此我们可以直接通过容器来发布事件。在组件类中想要获取也很简单： 
 <ol> 
  4. 由于**ApplicationEventPublisher**可以自定注入（这在此前学习IoC源码的时就见过了），因此我们只需要在组件类中添加一个ApplicationEventPublisher属性，Spring就会自动为我们注入事件发布者实例（实际上注入的就是当前容器实例）。引入事件发布者的类必须交给Spring管理。 
  4. 实现**ApplicationEventPublisherAware**这个感知接口，实现了这个接口的组件类在初始化时将会自动回调setApplicationEventPublisher感知接口，在参数中传递applicationEventPublisher实例。自定义的事件发布者必须交给Spring管理。 
 </ol>  
7. **事件监听器**：事件的处理是在事件监听器中实现的。 
 <ol> 
  7. 我们可以实现**ApplicationListener**接口来**定义自己的事件监听器**，同时ApplicationListener支持泛型，也就是说我们可以通过泛型监听某一个类型的事件！自定义的事件监听器必须交给Spring管理。 
  7. Spring 4.2之后，我们不必自定义监听器，而是使用@EventListener注解，注解标注的**方法的参数类型就是pushEvent方法传递的参数的类型**，因此它可以表示某一个事件的类型，或者直接表示传递的数据的类型（最新Spring 4.2的pushEvent方法可以传递任意类型的参数数据）。**也可以不需要参数**而在注解的classes/value属性中可以指定要监听的一批事件类型。在注解的condition属性中还可以指定匹配条件，满足条件的事件才会监听。该注解是通过EventListenerMethodProcessor后处理器解析的，注解标注的方法将被解析为一个ApplicationListenerMethodAdapter监听器对象。 
  7. **支持对监听器排序**，排序规则就是order值排序，常见的做法是在@EventListener方法上添加@Order注解（如果没有该注解，那么默认设置order值为0），或者在实现类上加入@Order注解/实现Ordered接口来实现，值越小，优先级越高！注意，由于这个排序仅仅是调用的排序，如果采用异步任务，那么最终完成的先后顺序无法保证！实际上，这是采用AnnotationAwareOrderComparator比较器来排序的，所以它还支持@Priority注解、PriorityOrdered接口排序，比较优先级为PriorityOrdered&gt;Ordered&gt;@Ordered&gt;@Priority，如果没有order值，那么将返回Integer.MAX_VALUE，即最低优先级。 
 </ol>  
11. **异步事件**： 
 <ol> 
  11. 默认情况下，事件监听器的执行操作是通过发布事件的线程来执行的，也就是说事件的发布和监听处理是在同一个线程中**同步阻塞**的，这样就不符合大部分使用者的初衷。为此，我们可以在监听方法上使用@Async异步任务注解，这样，当一个线程发布事件之后即可立即返回，Spring会使用其他线程去执行监听方法，这样才算是真正的将事件发布和监听执行解耦。关于异步任务，我们此前已经讲解过了，使用也非常简单： [Spring 5.x 学习(7)—@Async异步任务机制应用详解](https://blog.csdn.net/weixin_43767015/article/details/110135495)！ 
 </ol> 


  maven依赖：

```java
<properties>
        <spring-framework.version>5.2.8.RELEASE</spring-framework.version>
    </properties>
    <dependencies>
        <!--spring 核心组件所需依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>
    </dependencies>
```

  一个自定义事件类：

```java
/**
 * 自定义事件
 *
 * @author lx
 */
public class MyApplicationEvent extends ApplicationEvent {
   
    private String name;


    /**
     * @param source 事件发生或与之关联的对象（从不为null）
     */
    public MyApplicationEvent(Object source, String name) {
   
        super(source);
        this.name = name;
    }

    public String getName() {
   
        return name;
    }

}
```

  MyApplicationListener监听器类，监听MyApplicationEvent事件：

```java
/**
 * 自定义事件监听器，监听MyApplicationEvent事件
 *
 * @author lx
 */
@Component
public class MyApplicationListener implements ApplicationListener<MyApplicationEvent> {
   

    /**
     * 使用@Async开启异步事件
     */
    @Async
    @Override
    public void onApplicationEvent(MyApplicationEvent event) {
   
        System.out.println("-------------MyApplicationListener事件处理线程: " + Thread.currentThread().getName() + "-" + Thread.currentThread().hashCode());
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
        System.out.println(event.getSource());
        //创建事件的时间毫秒值
        System.out.println(event.getTimestamp());
        System.out.println(event.getName());
    }

}
```

  MyNewApplicationListener监听器类，监听PayloadApplicationEvent事件：

```java
/**
 * 自定义事件监听器，监听PayloadApplicationEvent<String>事件
 * PayloadApplicationEvent的泛型就是传递的参数的类型
 */
@Component
//注解排序
//@Order(1)
//@Priority(1)
public class MyNewApplicationListener implements ApplicationListener<PayloadApplicationEvent<String>>, PriorityOrdered {
   
    /**
     * 使用@Async开启异步事件
     */
    @Async
    @Override
    public void onApplicationEvent(PayloadApplicationEvent<String> event) {
   
        System.out.println("-------------MyNewApplicationListener事件处理线程: " + Thread.currentThread().getName() + "-" + Thread.currentThread().hashCode());
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
        //传递的参数
        System.out.println(event.getPayload());
        System.out.println(event.getResolvableType());
        System.out.println(event.getSource());
        System.out.println(event.getTimestamp());
    }

    /**
     * 实现了PriorityOrdered接口的优先级最高，其次才会比较order值
     */
    @Override
    public int getOrder() {
   
        return 10;
    }
}
```

  AnnoApplicationListener监听器类，基于@EventListener注解：

```java
/**
 * Spring 4.2新的@EventListener注解
 * 这里的AnnoApplicationListener没有实现ApplicationListener接口
 */
@Component
public class AnnoApplicationListener {
   

    /**
     * 使用@Async开启异步事件
     * <p>
     * 这里表示监听PayloadApplicationEvent<String>类型的事件
     */
    @Async
    @EventListener
    //排序
    @Order(5)
    public void listen(PayloadApplicationEvent<String> event) {
   
        System.out.println("-------------listen事件处理线程: " + Thread.currentThread().getName() + "-" + Thread.currentThread().hashCode());
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
        //传递的参数
        System.out.println(event.getPayload());
        System.out.println(event.getResolvableType());
        System.out.println(event.getSource());
        System.out.println(event.getTimestamp());
    }

    /**
     * 使用@Async开启异步事件
     * <p>
     * 这里表示监听PayloadApplicationEvent<String>类型的事件
     */
    @Async
    @EventListener
    public void listen1(PayloadApplicationEvent<String> event) {
   
        System.out.println("-------------listen1事件处理线程: " + Thread.currentThread().getName() + "-" + Thread.currentThread().hashCode());
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
        //传递的参数
        System.out.println(event.getPayload());
        System.out.println(event.getResolvableType());
        System.out.println(event.getSource());
        System.out.println(event.getTimestamp());
    }

    /**
     * 使用@Async开启异步事件
     * <p>
     * 该方法具有String类型的参数，表示监听参数数据类型为String类型的事件
     */
    @Async
    @EventListener
    public void listen2(String event) {
   
        System.out.println("-------------listen2事件处理线程: " + Thread.currentThread().getName() + "-" + Thread.currentThread().hashCode());
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
        //传递的参数
        System.out.println(event);
    }

    /**
     * 使用@Async开启异步事件
     * <p>
     * 该方法没有参数，但是classes指定为String和Integer，表示监听参数数据类型为String和Integer类型的事件
     */
    @Async
    @EventListener(classes = {
   String.class, Integer.class})
    public void listen3() {
   
        System.out.println("-------------listen3事件处理线程: " + Thread.currentThread().getName() + "-" + Thread.currentThread().hashCode());
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
    }
}
```

  一个服务，内部获取了ApplicationEventPublisher，可以发布事件：

```java
/**
 * 一个服务，内部获取了ApplicationEventPublisher，可以发布事件
 *
 * @author lx
 */
@Component
public class EventService implements ApplicationEventPublisherAware {
   
    /**
     * 1 可以直接注入ApplicationEventPublisher
     */
    @Resource
    private ApplicationEventPublisher applicationEventPublisher1;

    /**
     * 2 自己接收ApplicationEventPublisher
     */
    private ApplicationEventPublisher applicationEventPublisher2;

    /**
     * 实现ApplicationEventPublisherAware接口，将会自动回调setApplicationEventPublisher方法
     *
     * @param applicationEventPublisher Spring传递的applicationEventPublisher参数
     */
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
   
        applicationEventPublisher2 = applicationEventPublisher;
    }

    /**
     * 通用发布ApplicationEvent事件
     */
    public void pushEvent(ApplicationEvent applicationEvent) {
   
        applicationEventPublisher1.publishEvent(applicationEvent);
        applicationEventPublisher2.publishEvent(applicationEvent);
    }

    /**
     * Spring 4.2的新功能，发布任意事件，且不需要是ApplicationEvent类型
     * 将会自动封装为一个PayloadApplicationEvent事件类型
     */
    public void pushEvent(Object applicationEvent) {
   
        applicationEventPublisher1.publishEvent(applicationEvent);
        applicationEventPublisher2.publishEvent(applicationEvent);
    }


    /**
     * 测试，通过这两个方式获取的applicationEventPublisher是否就是同一个并且就是当前上下文容器
     */
    public void applicationEventPublisherTest() {
   
        System.out.println(applicationEventPublisher1 instanceof AnnotationConfigApplicationContext);
        System.out.println(applicationEventPublisher2 instanceof AnnotationConfigApplicationContext);
        System.out.println(applicationEventPublisher1 == applicationEventPublisher2);
    }
}
```

  注解支持启动组件类：

```java
@ComponentScan
@Configuration
//注解开启异步任务支持，没有这个注解无法开启异步任务
//@EnableAsync
public class StartConfig {
   
}
```

  测试类：

```java
public class EventTest {
   

    private static EventService eventService;

    static {
   
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(StartConfig.class);
        eventService = ac.getBean(EventService.class);
    }


    /**
     * 测试获取的applicationEventPublisher是否就是同一个发布者，并且就是当前上下文容器
     */
    public static void applicationEventPublisherTest() {
   
        eventService.applicationEventPublisherTest();
    }

    /**
     * 发布ApplicationEvent事件
     */
    public static void pushEvent() {
   
        //新建一个事件
        MyApplicationEvent myApplicationEvent = new MyApplicationEvent("ApplicationEvent事件", "MyApplicationEvent");
        //发布事件
        System.out.println("---------发布事件-----------");
        eventService.pushEvent(myApplicationEvent);
    }

    /**
     * Spring 4.2的新功能，发布任意事件，且不需要是ApplicationEvent类型
     * 将会自动封装为一个PayloadApplicationEvent事件类型
     */
    public static void pushEventNew() {
   
        //发布事件
        System.out.println("---------Spring 4.2发布事件-----------");
        eventService.pushEvent("PayloadApplicationEvent事件");
    }

    /**
     * 如果事件类型不符合，那么不会被监听到
     */
    public static void pushEventNotlisten() {
   
        //发布事件
        System.out.println("---------发布不会被监听到的事件-----------");
        eventService.pushEvent(111111);
    }

    public static void main(String[] args) throws InterruptedException {
   
        /*
         * 1 测试获取的applicationEventPublisher是否就是同一个发布者，并且就是当前上下文容器
         */
        applicationEventPublisherTest();

        long start = System.currentTimeMillis();
        System.out.println(start);

        /*
         * 2 发布ApplicationEvent事件
         */
        pushEvent();
        /*
         * 3 Spring 4.2的新功能，发布任意事件，不需要是ApplicationEvent类型
         * 将会自动封装为一个PayloadApplicationEvent事件类型
         */
        pushEventNew();

        /*
         * 4 如果事件类型不符合，那么不会被监听到
         */
        pushEventNotlisten();


        /*
         * 如果我们取消异步任务的支持，我们会发现，这些事件都是通过主线程同步执行的，到最后才会输出"事件发布返回"
         * 而如果开启异步任务，那么事件的处理就不需要发布事件的线程执行了，提升了速度，主线程将很快返回
         */
        System.out.println("---------事件发布返回，用时: " + (System.currentTimeMillis() - start));
    }
}
```

**相关文章：**    [https://spring.io/](https://spring.io/)    [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)    [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

