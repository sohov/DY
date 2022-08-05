

>  基于最新Spring 5.x，详细介绍了Spring的@Scheduled调度任务的概念和使用方法！

  调度任务，简单的说就是定时任务，这是web项目中非常有用，通常用于设置在某些固定的时刻执行特定的操作，比如设置调度任务在凌晨的时候自动同步数据！Spring也提供了自己的调度任务机制，下面我们简单的学习一下！





## 1.1 TaskScheduler调度器

  Spring2.0时提供了的异步任务抽象TaskExecutor，在此前我们就学习过了。而在Spring 3.0的时候又提供了调度任务抽象TaskScheduler，称为“调度器”。   TaskScheduler接口提供了很多方法，可以通过多种规则来设置、执行调度任务！

```java
/**
 * 调度任务抽象接口
 */
public interface TaskScheduler {
   

    /**
     * 计划给定的Runnable任务，每当Trigger触发器指示下一个执行时间时调用它。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     *
     * @param task    可执行的任务
     * @param trigger Trigger触发器接口的实现，例如cronTrigger触发器，可以使用cron 表达式来指定任务调度规则
     * @return 一个ScheduledFuture对象，表示调度任务结果，如果给定的触发器从来不会被触发（下次执行时间nextExecutionTime方法返回null）
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    @Nullable
    ScheduledFuture<?> schedule(Runnable task, Trigger trigger);

    /**
     * 计划给定的Runnable任务，在指定的执行时间调用它，该任务只会执行一次
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     * 这是Spring 5.0新增的一个默认方法，基于Java8，其内部就是默认调用的schedule(Runnable, Date)抽象方法
     *
     * @param task      可执行的任务
     * @param startTime 任务的启动执行时间（如果是过去时间，则任务将立即、尽快执行），Instant类型
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    default ScheduledFuture<?> schedule(Runnable task, Instant startTime) {
   
        return schedule(task, Date.from(startTime));
    }

    /**
     * 计划给定的Runnable任务，在指定的执行时间调用它，该任务只会执行一次。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     *
     * @param task      可执行的任务
     * @param startTime 任务的启动执行时间（如果是过去时间，则任务将立即、尽快执行），Date类型
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    ScheduledFuture<?> schedule(Runnable task, Date startTime);

    /**
     * 计划给定的Runnable任务，在指定的执行时间调用它，随后在给定的周期period之后重复调用它，即FixedRate模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     * 这是Spring 5.0新增的一个默认方法，基于Java8，其内部就是默认调用的scheduleAtFixedRate(Runnable, Date, long)抽象方法。
     *
     * @param task      可执行的任务
     * @param startTime 任务的首次执行时间（如果是过去时间，则任务将立即、尽快执行），Instant类型
     * @param period    后续两个任务开始执行之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    default ScheduledFuture<?> scheduleAtFixedRate(Runnable task, Instant startTime, Duration period) {
   
        return scheduleAtFixedRate(task, Date.from(startTime), period.toMillis());
    }

    /**
     * 计划给定的Runnable任务，在指定的执行时间调用它，随后在给定的周期period之后重复调用它，即FixedRate模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     *
     * @param task      可执行的任务
     * @param startTime 任务的首次执行时间（如果是过去时间，则任务将立即、尽快执行），Date类型
     * @param period    后续两个任务开始执行之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    ScheduledFuture<?> scheduleAtFixedRate(Runnable task, Date startTime, long period);

    /**
     * 计划给定的Runnable任务，没指定首次启动时间，因此任务将立即、尽快执行，随后在给定的周期period之后重复调用它，即FixedRate模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     * 这是Spring 5.0新增的一个默认方法，基于Java8，其内部就是默认调用的scheduleAtFixedRate(Runnable, long)抽象方法。
     *
     * @param task   可执行的任务
     * @param period 后续两个任务开始执行之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    default ScheduledFuture<?> scheduleAtFixedRate(Runnable task, Duration period) {
   
        return scheduleAtFixedRate(task, period.toMillis());
    }

    /**
     * 计划给定的Runnable任务，没指定首次启动时间，因此任务将立即、尽快执行，随后在给定的周期period之后重复调用它，即FixedRate模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     *
     * @param task   可执行的任务
     * @param period 后续两个任务开始执行之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    ScheduledFuture<?> scheduleAtFixedRate(Runnable task, long period);

    /**
     * 计划给定的Runnable任务，在指定的执行时间调用它，随后在上一次任务完成之后，间隔delay并再次调用它，即FixedDelay模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     * 这是Spring 5.0新增的一个默认方法，基于Java8，其内部就是默认调用的scheduleWithFixedDelay(Runnable, Date, long)抽象方法。
     *
     * @param task      可执行的任务
     * @param startTime 任务的首次执行时间（如果是过去时间，则任务将立即、尽快执行），Instant类型
     * @param delay     后续从上一次任务完成到下一个任务开始之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    default ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, Instant startTime, Duration delay) {
   
        return scheduleWithFixedDelay(task, Date.from(startTime), delay.toMillis());
    }

    /**
     * 计划给定的Runnable任务，在指定的执行时间调用它，随后在上一次任务完成之后，间隔delay并再次调用它，即FixedDelay模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     *
     * @param task      可执行的任务
     * @param startTime 任务的首次执行时间（如果是过去时间，则任务将立即、尽快执行），Instant类型
     * @param delay     后续从上一次任务完成到下一个任务开始之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, Date startTime, long delay);

    /**
     * 计划给定的Runnable任务，没指定首次启动时间，因此任务将立即、尽快执行，随后在上一次任务完成之后，间隔delay并再次调用它，即FixedDelay模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     * 这是Spring 5.0新增的一个默认方法，基于Java8，其内部就是默认调用的scheduleWithFixedDelay(Runnable, long)抽象方法。
     *
     * @param task  可执行的任务
     * @param delay 后续从上一次任务完成到下一个任务开始之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    default ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, Duration delay) {
   
        return scheduleWithFixedDelay(task, delay.toMillis());
    }

    /**
     * 计划给定的Runnable任务，没指定首次启动时间，因此任务将立即、尽快执行，随后在上一次任务完成之后，间隔delay并再次调用它，即FixedDelay模式。
     * 一旦调度程序关闭或返回的ScheduledFuture被取消，则执行将结束。
     *
     * @param task  可执行的任务
     * @param delay 后续从上一次任务完成到下一个任务开始之间的时间间隔
     * @return 一个ScheduledFuture对象，表示调度任务结果
     * @throws TaskRejectedException 如果给定任务因内部原因（例如执行了拒绝策略或者正在关闭执行器线程池）而未被接受
     */
    ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, long delay);

}
```

  这些方法中：

1. 只接收task和startTime参数的schedule方法指定的调度任务只会执行一次，而所有其他方法都能够安排任务重复运行。 
2. 基于固定速率（fixed-rate） 和固定延迟（fixed-delay）模式的方法用于指定简单、定期执行的任务。 
3. 接收Trigger触发器参数的schedule方法最灵活。

  Spring提供了一些TaskScheduler接口的默认实现，常见的就是**ThreadPoolTaskScheduler**，它们的内部核心机制都依靠了JUC中的ScheduledExecutorService接口的默认实现ScheduledThreadPoolExecutor，关于JUC的ScheduledExecutorService的源码，我们此前就已经学习过了（ [JUC—六万字的Executor线程池框架源码深度解析](https://blog.csdn.net/weixin_43767015/article/details/108025620)），这个接口专门用于提供调度任务服务！   另外还有一个**ConcurrentTaskScheduler**，它默认的执行器只有单个线程，但是它可以将JDK中的ScheduledExecutorService转换为Spring的TaskScheduler。

## 1.2 Trigger触发器

  **Trigger**又名触发器，注意区分数据库中的触发器，这里的Trigger是Spring 3.0时提供的一个接口，它的基本思想是可以通过任意指定条件（甚至上一次的执行情况）来确定任务下一次执行时间。它是一种更加灵活的任务触发规则抽象！   **Trigger**接口本身非常简单，提供了一个nextExecutionTime方法，用于根据上一次任务的执行上下文获取下一次任务的执行时间：

```java
/**
 * 触发器抽象接口
 */
public interface Trigger {
   

    /**
     * 根据给定的触发器上下文确定下一个执行时间
     *
     * @param triggerContext 上下文对象，封装了给定任务的上次执行时间和上次完成时间
     * @return 触发器定义的下一次执行时间，如果触发器不再触发， 则返回null
     */
    @Nullable
    Date nextExecutionTime(TriggerContext triggerContext);

}
```

  任务上一次的执行详情保存在TriggerContext上下文中，TriggerContext接口本身也很简单：

```java
/**
 * 触发器上下文抽象接口
 * 封装了给定任务的上次执行时间和上次完成时间
 */
public interface TriggerContext {
   

    /**
     * 返回上一次任务计划执行时间，如果未安排则返回null
     */
    @Nullable
    Date lastScheduledExecutionTime();

    /**
     * 返回上一次任务实际执行时间，如果未安排则返回null
     */
    @Nullable
    Date lastActualExecutionTime();

    /**
     * 返回上一次任务的完成时间，如果未安排则返回null
     */
    @Nullable
    Date lastCompletionTime();

}
```

## 1.3 Trigger的实现

  Spring提供了Trigger接口的两个实现。最有趣的是**CronTrigger**，它允许基于 cron 表达式非常灵活的定义调度任务。cron 表达式非常非常的灵活和强大，例如下面定义的任务在工作日（周一到周五）的上午9点到下午五点的每小时第15分钟运行：

```java
public static void main(String[] args) {
   
    ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
    threadPoolTaskScheduler.initialize();
    threadPoolTaskScheduler.schedule(() -> System.out.println("cron"), new CronTrigger("0 15 9-17 * * MON-FRI"));
}
```

  关于cron表达式，这里不过多讲解，因为网上有很多学习文章教，下面是一些可以在线生成cron表达式的网站：https://qqe2.com/cron、https://www.bejson.com/othertools/cron/。   另一个实现就是接收固定周期的**PeriodicTrigger**，它可以的构造器可以指定一个周期时间以及一个全局的事件单位，可以通过setInitialDelay方法指定第一次执行的延迟时间，通过setFixedRate方法来指示构造器中的周期应该是固定速率（fixed-rate）还是固定延迟（fixed-delay）。   由于TaskScheduler接口已经定义了固定速率（fixed-rate）或固定延迟（fixed-delay）的调度任务方法，因此尽可能直接使用这些方法。PeriodicTrigger通常是在依赖Trigger的组件中使用。   下面定义的任务在1秒之后开始执行，并且在上一次任务执行完毕和下一次执行开始中间间隔3秒，即固定延迟（fixed-delay），任务执行时间为1秒：

```java
public static void main(String[] args) {
   
    //设置上一次任务执行完毕和下一次执行开始中间间隔1秒
    PeriodicTrigger periodicTrigger = new PeriodicTrigger(3, TimeUnit.SECONDS);
    //设置任务第一次执行的延迟时间
    periodicTrigger.setInitialDelay(1);
    //设置为固定延迟（fixed-delay）的调度任务，这也是默认类型
    periodicTrigger.setFixedRate(false);

    ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
    threadPoolTaskScheduler.initialize();
    AtomicLong atomicLong = new AtomicLong(System.currentTimeMillis());
    System.out.println(0);
    threadPoolTaskScheduler.schedule(() -> {
   
        long newTime = System.currentTimeMillis();
        long time = newTime - atomicLong.get();
        atomicLong.set(newTime);
        System.out.println(time);
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
        long newTime2 = System.currentTimeMillis();
        long time2 = newTime2 - atomicLong.get();
        atomicLong.set(newTime2);
        System.out.println("任务执行时间: " + time2);
    }, periodicTrigger);
}
```


  相比于 [Spring异步任务](https://blog.csdn.net/weixin_43767015/article/details/110135495)，我们可以使用注解或者XML配置的方式支持调度任务。

## 2.1. 基于XML的配置

  maven依赖：

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version> 5.2.8.RELEASE</version>
</dependency>
```

  一个测试类，com.spring.integration.tasks.schedule.xml.XmlScheduleMethod：

```java
/**
 * 三个调度任务
 *
 * @author lx
 */
public class XmlScheduleMethod {
   
    public final static LongAdder scheduledCount=new LongAdder();
    public void Scheduled1() {
   
        scheduledCount.increment();
        System.out.println("-----Scheduled1：" + Thread.currentThread().getName());
        //执行3秒
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(3));
    }

    public void Scheduled2() {
   
        scheduledCount.increment();
        System.out.println("-----Scheduled2：" + Thread.currentThread().getName());
        //执行1秒
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
    }

    public void Scheduled3() {
   
        scheduledCount.increment();
        System.out.println("-----Scheduled3：" + Thread.currentThread().getName());
        //执行2秒
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
    }
}
```

  schedule-config.xml配置文件，注意引入task的命名空间（idea可自动引入）：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd   http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd">

    <!--调度方法类-->
    <bean class="com.spring.integration.tasks.schedule.xml.XmlScheduleMethod" name="xmlScheduleMethod"/>

    <!--定义调度器-->
    <!--id属性将作为该调度器线程池中的线程的前缀-->
    <!--pool-size属性用于指定调度器线程池的线程数，默认为一个线程-->
    <task:scheduler id="scheduler1" pool-size="1"/>
    <task:scheduler id="scheduler2" pool-size="1"/>

    <!--定义调度器与调度任务的关系-->
    <!--scheduler属性可以引用一个调度器，如果没有指定调度器，那么采用默认调度器，只有一个线程-->
    <task:scheduled-tasks scheduler="scheduler1">
        <!--该标签用于定义一个调度任务，是核心标签-->
        <!--ref用于指定一个bean的引用-->
        <!--method用于指定该bean中的一个方法-->
        <!--cron用于指定一个cron表达式-->
        <!--fixed-delay用于指定固定延迟（以毫秒为单位）-->
        <!--fixed-rate用于指定固定速率（以毫秒为单位）-->
        <!--initial-delay用于指定初始延迟（以毫秒为单位）-->
        <!--trigger用于指定一个触发器，直接通过触发器进行调度-->
        <!--可以配置多个task:scheduled标签，他们的执行顺序和定义的先后顺序无关，和配置的属性有关-->
        <task:scheduled ref="xmlScheduleMethod" method="Scheduled1" fixed-rate="2000"/>
        <task:scheduled ref="xmlScheduleMethod" method="Scheduled2" fixed-delay="1000" initial-delay="1000"/>
    </task:scheduled-tasks>

    <task:scheduled-tasks scheduler="scheduler2">
        <task:scheduled ref="xmlScheduleMethod" method="Scheduled3" cron="*/1 * * * * MON-FRI"/>
    </task:scheduled-tasks>
</beans>
```

  可以看到，配置还是非常简单的！

1. **< task:scheduler/>** 标签用于定义一个调度器，实际类型为ThreadPoolTaskScheduler，它的id属性将作为该调度器线程池中的线程的前缀，pool-size属性则用于指定调度器线程池的线程数，默认为一个线程，对于那种一个调度器多个调度任务的情况，多线程可能会提升执行效率！ 
2. **< task:scheduled-tasks/>** 标签用于定义调度任务。它的scheduler属性指向一个定义好的调度器的id，如果没有该属性，那么默认采用线程数为1的ThreadPoolTaskScheduler调度器，线程名为“pool-x-thread-y”。 
3. **< task:scheduled/>** 是核心标签，用于配置调度任务，它提供了很多属性：


  测试类如下：

```java
/**
 * @author lx
 */
public class XmlScheduleMethodTest {
   
    public static void main(String[] args) {
   
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("schedule-config.xml");
        //阻塞15秒让其运行调度任务
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(20));
        ac.close();
        System.out.println("20秒内执行调度任务次数: " + XmlScheduleMethod.scheduledCount.sum());
    }
}
```

  **测试结果在20秒内大概执行16次调度任务，我们将调度器的线程数都改为2：**

```java
<task:scheduler id="scheduler1" pool-size="2"/>
<task:scheduler id="scheduler2" pool-size="2"/>
```

  **再次测试，结果在20秒内大概执行24次调度任务，效率很明显的提升了。那么如果我们将线程数量改为3甚至更大呢？实际上我们会发现，其效率并没有进一步提升，因为这两个调度器的任务量本来就不是很大，并且调度任务有间隔时间的限制，就算有再多的线程，如果没有任务去执行也是没用的，因此每一个调度器两个线程足以应付，开启更多的线程反而会浪费资源！**

## 2.2 基于注解的调度任务

### 2.2.1 开启注解支持

1. 对于Java Config配置的方式，一般我们将@EnableScheduling注解添加到配置类上面，表示开启调度任务注解的支持。 
2. 对于XML的方式，我们使用&lt; task:annotation-driven/&gt;标签来开启调度任务注解的支持，该标签还能同时开启异步任务的支持。该标签的scheduler属性用于指定注解标注的调度任务的调度执行器，如果不指定，那么将默认使用单个线程的调度器！

### 2.2.2 @Scheduled和@Schedules注解

  **Spring中的调度任务可以使用@Scheduled和@Schedules注解绑定到一个bean方法，这两个注解也是Spring 3.0添加的注解。**

1. 请注意，要调度的方法必须返回 void ，并且不能有任何参数。如果该方法需要与应用程序上下文中的其他对象进行交互，则通常通过依赖项注入提供这些对象。 
2. 自Spring 4.3开始，任何scope作用域的bean都支持@Scheduled调度方法！比如，如果是prototype的bean，那么获取的每一个实例都将独立执行调度任务！ 
3. @Scheduls注解需要指定一个@Scheduled注解数组，这表示将多个@Scheduled调度任务策略绑定到一个方法上！ 
4. @Scheduled注解用于将一个调度策略绑定到Spring管理的bean的方法上，并且可以配置各种触发器元数据属性。一个方法可以配置多个@Scheduled，这就类似于@Scheduls注解。

  **@Scheduled注解的属性如下：**


### 2.2.3 注解案例

  这里我们讲解Java Config的配置方式，彻底舍弃XML文件，这也是目前最流行的方式（类似于Spring Boot）。   一个测试类com.spring.integration.tasks.schedule.ann.AnnScheduleMethodConfig

```java
/**
 * 三个调度任务
 *
 * @author lx
 */
@Configuration
@EnableScheduling
@ComponentScan
public class AnnScheduleMethodConfig {
   

    public final static LongAdder SCHEDULED_COUNT = new LongAdder();

    @Scheduled(fixedDelay = 2000)
    public void Scheduled1() {
   
        SCHEDULED_COUNT.increment();
        System.out.println("-----Scheduled1：" + Thread.currentThread().getName());
        //执行3秒
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(3));
    }

    @Scheduled(fixedRate = 1000)
    public void Scheduled2() {
   
        SCHEDULED_COUNT.increment();
        System.out.println("-----Scheduled2：" + Thread.currentThread().getName());
        //执行1秒
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
    }

    @Schedules({
   @Scheduled(fixedRate = 1000), @Scheduled(fixedDelay = 2000)})
    //@Scheduled(cron = "*/5 * * * * MON-FRI")
    public void Scheduled3() {
   
        SCHEDULED_COUNT.increment();
        System.out.println("-----Scheduled3：" + Thread.currentThread().getName());
        //执行2秒
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
    }
}
```

  测试类：

```java
public class AnnScheduleStart {
   
    public static void main(String[] args) {
   
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AnnScheduleMethodConfig.class);
        //阻塞15秒让其运行调度任务
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(20));
        ac.close();
        System.out.println("20秒内执行调度任务次数: " + AnnScheduleMethodConfig.SCHEDULED_COUNT.sum());
    }
}
```

  **可以看到，配置还是很简单的，但是我们发现，这几个调度任务都始终使用同一个调度线程。实际上，在使用注解的默认情况下，所有配置的调度任务都将会使用单个线程的默认调度器去执行。**   **当然，我们可以配置指定的调度器，但是在此之前，我们应该明白Spring对于调度器的查找规则，在Spring 5.2.8.RELEASE版本中，scheduler的配置规则在ScheduledAnnotationBeanPostProcessor后处理器的finishRegistration方法中：**

1. 如果beanFactory中存在**SchedulingConfigurer**类型的bean，那么全部实例化，并且使用AnnotationAwareOrderComparator比较器进行排序，也就是order排序。随后依次调用这些SchedulingConfigurer的**configureTasks**方法，这个方法可以用于手动配置调度任务以及调度器。 
2. 经过上面的过程，如果存在调度任务并且调度器没有手动配置。那么在beanFactory中**查找唯一的一个TaskScheduler类型的调度器**，随后实例化并设置为默认调度器。如果查找失败，那么可能是存在多个TaskScheduler类型的调度器，或者不存在TaskScheduler类型的调度器： 
 <ol> 
  2. 如果是存在多个TaskScheduler类型的调度器，那么继续**查找名为“taskScheduler”的TaskScheduler类型的调度器**，随后实例化并设置为默认调度器。 
  2. 如果是不存在TaskScheduler类型的调度器，那么继续**查找唯一的一个ScheduledExecutorService类型的调度器**，随后实例化并设置为默认调度器。如果查找失败，那么可能是存在多个ScheduledExecutorService类型的调度器，或者不存在ScheduledExecutorService类型的调度器： 
   <ol> 
    2. 如果是存在多个ScheduledExecutorService类型的调度器，那么继续**查找名为“taskScheduler”的ScheduledExecutorService类型的调度器**，随后实例化并设置为默认调度器。 
   </ol>  
 </ol>  
6. 如果以上操作均失败，那么表示**不存在唯一的TaskScheduler以及ScheduledExecutorService类型的调度器，或者存在多个TaskScheduler/ScheduledExecutorService类型的调度器但是没有一个名为“taskScheduler”**。那么将会在最后的**registrar.afterPropertiesSet()方法中，创建一个只有一个工作线程的ConcurrentTaskScheduler调度器**。


![img](https://img-blog.csdnimg.cn/20201215161520158.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   明白了Spring 注解调度任务的调度器的查找规则，那么我们自然就可以很容易的实现自定义调度器：

1. 向Spring中**添加名为“taskScheduler”的TaskScheduler/ScheduledExecutorService类型的调度器**。 
2. 或者是**添加唯一的一个TaskScheduler/ScheduledExecutorService类型的调度器**，此时它的名字可以随意取。 
3. 或者是**添加SchedulingConfigurer的实现**，在重写的configureTasks方法中，通过参数taskRegistrar的setScheduler方法手动配置一个调度器！

  我们采用最简单的方法，在AnnScheduleMethodConfig中添加一个@Bean方法，配置一个自己的调度器：

```java
/**
 * 创建自己的调度器
 */
@Bean
public TaskScheduler taskExecutor() {
   
    ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
    threadPoolTaskScheduler.setPoolSize(5);
    threadPoolTaskScheduler.setThreadNamePrefix("myTaskExecutor");
    return threadPoolTaskScheduler;
}
```

  **再次测试，此时我们可以发现Spring已经采用了我们自定义的调度器，配置成功！**   **除此之外，Spring支持异步调度任务，即@Async和@Scheduled结合使用，关于Spring异步任务：Spring 5.x 学习(7)—@Async异步任务机制应用详解**。


  JDK为我们提供了两个原生了任务调度工具，即**Timer+TimerTask**、**ScheduledExecutorService**，Timer中的多个任务只能使用一个线程去执行，因此任务之间的执行情况会相互影响，后来出现的 [ScheduledExecutorService](https://blog.csdn.net/weixin_43767015/article/details/108025620)支持多线程并发的去执行多个调度任务，弥补了这个缺陷。   但是JDK的调度任务工具都比较原始，只支持固定速率（fixed-rate）或固定延迟（fixed-delay）的调度任务，不灵活，Spring schedule则解决了这个问题，**支持cron表达式，可以配置任意基于时钟的调度任务。**   然而，无论是JDK调度任务还是Spring schedule，一个非常大的缺陷就是，**这些调度任务都是基于单个JVM实例，和项目偶合在一起**，如果web项目集群、分布式部署，那么就会有多个JVM实例，因此会造成同一个调度任务被多次触发，并且它们的功能并不全面，比如在项目运行时添加、修改、取消调度任务，另外调度任务较多时由于分散在各个项目中，也不利于管理！**因此上面的调度任务包括本次介绍的Spring Schedule基本上不会在生产环境使用**。   解决办法就是使用专门用于调度任务的中间件，从将调度任务从项目中独立出来，常见的Java调度任务框架有**Quartz、Elastic-Job、XXL-JOB**，它们都支持集群、分布式部署，现在用得更多的是Elastic-Job和XXL-JOB，并且它们的上手也很简单，后续我们再介绍。**如果想要深入学习这些调度任框架原理，那么先看Spring Schedule的源码是一个不错的入门方法，因为它比较简单！**

**相关文章：**    [https://spring.io/](https://spring.io/)    [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)    [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

