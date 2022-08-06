

>  基于最新Spring 5.x，详细介绍了Spring的@TransactionalEventListener事务绑定事件机制的应用！

  前面就讲过，我们使用@EventListener注解来注册常规事件监听器： [Spring 5.x 学习(8)—@EventListener事件发布机制应用详解](https://blog.csdn.net/weixin_43767015/article/details/110265205)。从Spring 4.2开始，提供了专门针对事务的的监听器，事务监听器可以绑定到事务的某一个阶段，比如说，可以在事务提交成功后才开始处理事件。如果事务的执行结果对后续业务很重要时，就可以使用事务监听器来完成！   有两种方式来实现事务监听，一种是@TransactionalEventListener注解，另一种就是TransactionSynchronizationManager.registerSynchronization手动控制。我们讲解基于注解的配置！另外，关于Spring事务管理，我们此前也讲过了： [Spring 5.x 学习(11)—两万字的Spring事务管理的深入介绍和使用案例](https://blog.csdn.net/weixin_43767015/article/details/111410856)。







  事务监听器专门用于监听事务中发布的事件，@TransactionalEventListener注解包装了@EventListener注解，是普通监听器的加强，但是监听器方法是通过回调触发的，即在事务进行gcommit或者rollback的时候会回调监听器方法进行处理。而其他的，事务事件的发布方式和普通事件的发布方式是一样的，只不过事务事件必须在事务中发布。如果发布了“事务事件”，并且事件类型和某些普通监听器监听的事件类型一致，那么普通监听器也会被触发！   @TransactionalEventListener和@EventListener一样，都是同步处理，即处理事件和发布事件的线程是同一个，因此仍然可能会阻塞线程，但是可以使用@Async进行异步任务处理！   @TransactionalEventListener可以通过phase属性指定触发阶段，有四种：

1. BEFORE_COMMIT：在事务提交之前触发事件。 
2. AFTER_COMMIT：在成功完成提交后触发事件。这是默认的触发阶段。 
3. AFTER_ROLLBACK：如果事务已回滚，则触发事件。 
4. AFTER_COMPLETION：事务完成之后（无论是提交还是回滚）进行触发。如果同时注册AFTER_COMPLETION和AFTER_ROLLBACK/AFTER_COMMIT事件，那么触发的先后顺序是不固定的，但是可以使用@Order注解指定先后顺序。

  @TransactionalEventListener标注的监听器方法，在默认情况下仅仅会被事务中发布的事件触发，如果需要在没有事务也能当作普通时间监听器触发，那么需要将fallbackExecution属性设置为true。


## 2.1 maven依赖

```java
<properties>
    <spring-framework.version>5.2.8.RELEASE</spring-framework.version>
    <mysql-connector-java>8.0.16</mysql-connector-java>
    <druid>1.2.3</druid>
    <lombok>1.18.12</lombok>
    <junit>4.12</junit>
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
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <!--spring-jdbc  内部包括了spring-tx的依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring-framework.version}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <!--mysql数据库驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql-connector-java}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
    <!--druid数据源-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid}</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
    <!--Spring 测试-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring-framework.version}</version>
    </dependency>
    <!--单元测试-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit}</version>
    </dependency>
</dependencies>
```

## 2.2 数据库表

  本人数据库是MySql 8版本。数据库表：

```java
CREATE TABLE `tx_study` (
	`id` INT ( 11 ) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR ( 200 ) DEFAULT NULL COMMENT '姓名',
	`age` INT ( 11 ) DEFAULT NULL COMMENT '年龄',
	`create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
PRIMARY KEY ( `id` ) 
) ENGINE = INNODB AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8;
```

  插入一些数据：

```java
INSERT INTO `tx_study`
VALUES
	( NULL, 'Google', 12, '2019-04-21 15:55:15' ),
	( NULL, '淘宝', 11, CURRENT_TIMESTAMP() ),
	( NULL, '百度', 1, '2018-04-21 15:55:15' ),
	( NULL, '微博', 5, CURRENT_TIMESTAMP() ),
	( NULL, 'Facebook', 5, '2020-04-21 15:55:15' );
```

  实体：

```java
public class TxStudy {
   

    private Date createTime;
    private Integer id;
    private String name;
    private Integer age;


    public Date getCreateTime() {
   
        return createTime;
    }

    public void setCreateTime(Date createTime) {
   
        this.createTime = createTime;
    }

    public Integer getId() {
   
        return id;
    }

    public void setId(Integer id) {
   
        this.id = id;
    }

    public String getName() {
   
        return name;
    }

    public void setName(String name) {
   
        this.name = name;
    }

    public Integer getAge() {
   
        return age;
    }

    public void setAge(Integer age) {
   
        this.age = age;
    }

    @Override
    public String toString() {
   
        return "TxStudy{" +
                "createTime=" + createTime +
                ", id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public TxStudy() {
   }

    public TxStudy(String name, Integer age) {
   
        this.name = name;
        this.age = age;
    }
}
```

## 2.3 业务类

  通常，如果我们需要发布事务事件，那么需要在事务方法的一开始就需要发布，不用担心会被提前出发，它是使用的回调机制，在事务的不同阶段会自动回调对应的事务处理器。如果是在代码中间后者后面才发布事件，那么可能由于前一部分的代码抛出了异常而导致事件发布的代码不被执行！   事件发布的方式和普通事件一样，可以使用applicationEventPublisher. publishEvent方法直接发布，参数可以是任意类型，不必是一个事件，Spring会自动为我们封装成一个事件！

```java
@Component
public class EventService {
   
    /**
     * 直接注入ApplicationEventPublisher，用于发布事件
     */
    @Resource
    private ApplicationEventPublisher applicationEventPublisher;
    /**
     * jdbc模版，用于操作数据库
     */
    @Resource
    private JdbcTemplate jdbcTemplate;

    //为了简单，直接在Service中进行数据库访问

    @Transactional
    public void select() {
   
        applicationEventPublisher.publishEvent("查询数据");
        String sql = "select * from tx_study where id = ?";
        System.out.println(jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), 1));
    }
}
```

## 2.4 监听器类

  **我们设置了4个事务事件监听器，分别监听不同阶段的数据类型为String的事件，同时设置两个普通事件监听器，分别监听数据类型为String和Integer的事件。**

```java
@Component
public class MyTransactionalEventListener {
   

    /**
     * 事件回滚后监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void afterRollback(String str) {
   
        //获取事件传递的数据
        System.out.println("AFTER_ROLLBACK: " + str);
    }

    /**
     * 事件提交前监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
    public void beforeCommit(String str) {
   
        //获取事件传递的数据
        System.out.println("BEFORE_COMMIT: " + str);
    }

    /**
     * 事件提交后监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void afterCommit(String str) {
   
        //获取事件传递的数据
        System.out.println("AFTER_COMMIT: " + str);
    }

    /**
     * 事件完成后监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION)
    public void afterCompletion(String str) {
   
        //获取事件传递的数据
        System.out.println("AFTER_COMPLETION: " + str);
    }

    //普通事件监听器

    /**
     * 普通事件监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @EventListener
    public void listen(String str) {
   
        //获取事件传递的数据
        System.out.println("普通事件: " + str);
    }

    /**
     * 普通事件监听器
     *
     * @param integer 发布的事件，要求是Integer及其兼容类型
     */
    @EventListener
    public void listen(Integer integer) {
   
        //获取事件传递的数据
        System.out.println("普通事件: " + integer);
    }
}
```

## 2.5 配置类

  我们使用Java Config形式配置，舍弃XML文件！

```java
@ComponentScan
@Configuration
@EnableTransactionManagement
public class EventStart {
   
    /**
     * 配置Druid数据源
     */
    @Bean
    public DruidDataSource druidDataSource() {
   
        DruidDataSource druidDataSource = new DruidDataSource();
        //为了方便，直接硬编码了，我们可以通过@Value引入外部配置，
        //如果使用Spring boot就更简单了，直接使用@ConfigurationProperties引入外部配置
        //简单的配置数据库连接信息，其他连接池信息采用默认配置
        druidDataSource.setUrl("jdbc:mysql://47.94.229.245:3306/test?useSSL=false&allowPublicKeyRetrieval=true");
        druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("123456");
        return druidDataSource;
    }

    /**
     * 配置JdbcTemplate
     * 直接使用spring-jdbc来操作某一个数据库，不使用其他外部数据库框架
     */
    @Bean
    public JdbcTemplate jdbcTemplate() {
   
        //传入一个数据源
        return new JdbcTemplate(druidDataSource());
    }


    /**
     * 配置DataSourceTransactionManager
     * 用于管理某一个数据库的事务
     */
    @Bean
    public DataSourceTransactionManager transactionManager() {
   
        //传入一个数据源
        return new DataSourceTransactionManager(druidDataSource());
    }
}
```

## 2.6 测试

  测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = EventStart.class)
public class EventTest {
   

    @Resource
    private EventService eventService;

}
```

  首先尝试直接调用方法：

```java
@Test
public void test()  {
   
    eventService.select();
}
```

  结果可能如下（AFTER_COMPLETION和AFTER_COMMIT的输出位置可能相反）：

```java
普通事件: 查询数据
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
BEFORE_COMMIT: 查询数据
AFTER_COMPLETION: 查询数据
AFTER_COMMIT: 查询数据
```

  我们这次测试是在事务中发布的事件，并且事务成功提交。从结果中我们看到，普通的事件监听器被立即触发了，而事务事件监听器则会在事件的指定阶段触发！   我们改写select方法，将方法上的@Transactional注解注释掉，即这次在非事务中发布事件。结果将会如下：

```java
普通事件: 查询数据
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
```

  可以看到，由于事务事件监听器默认情况下只能监听事务中的事件，因此事务事件监听器都没有被触发！   如果我们设置@TransactionalEventListener注解的fallbackExecution属性为true：

```java
/**
 * 事件回滚后监听器
 *
 * @param str 发布的事件，要求是String及其兼容类型
 */
@TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK,fallbackExecution = true)
public void afterRollback(String str) {
   
    //获取事件传递的数据
    System.out.println("AFTER_ROLLBACK: " + str);
}

/**
 * 事件提交前监听器
 *
 * @param str 发布的事件，要求是String及其兼容类型
 */
@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT,fallbackExecution = true)
public void beforeCommit(String str) {
   
    //获取事件传递的数据
    System.out.println("BEFORE_COMMIT: " + str);
}

/**
 * 事件提交后监听器
 *
 * @param str 发布的事件，要求是String及其兼容类型
 */
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT,fallbackExecution = true)
public void afterCommit(String str) {
   
    //获取事件传递的数据
    System.out.println("AFTER_COMMIT: " + str);
}

/**
 * 事件完成后监听器
 *
 * @param str 发布的事件，要求是String及其兼容类型
 */
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION,fallbackExecution = true)
public void afterCompletion(String str) {
   
    //获取事件传递的数据
    System.out.println("AFTER_COMPLETION: " + str);
}
```

  测试结果如下：

```java
普通事件: 查询数据
BEFORE_COMMIT: 查询数据
AFTER_COMMIT: 查询数据
AFTER_COMPLETION: 查询数据
AFTER_ROLLBACK: 查询数据
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
```

  可以看到，这些事务事件监听器就可以监听非事务中发布的事件。但需要注意的是，如果一个事件在事务中发布，一个事务事件监听器的fallbackExecution属性为true，并且不存在该监听器对应的事务阶段（比如阶段为回滚后，但是事务实际上提交成功了），那么该监听器仍然不会被触发！   我们继续改写select方法，让它抛出一个**非受检异常**：

```java
@Transactional
public void select() {
   
    applicationEventPublisher.publishEvent("查询数据");
    String sql = "select * from tx_study where id = ?";
    System.out.println(jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), 1));
    throw new RuntimeException();
}
```

  再次测试，结果如下（AFTER_COMPLETION和AFTER_ROLLBACK输出位置可能相反）：

```java
普通事件: 查询数据
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
AFTER_ROLLBACK: 查询数据
AFTER_COMPLETION: 查询数据
```

  这就是事务回滚时触发的事件！   如果我们抛出一个受检异常：

```java
@Transactional
public void select() throws FileNotFoundException {
   
    applicationEventPublisher.publishEvent("查询数据");
    String sql = "select * from tx_study where id = ?";
    System.out.println(jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), 1));
    throw new FileNotFoundException();
}
```

  测试结果如下（AFTER_COMPLETION和AFTER_COMMIT的输出位置可能相反）：

```java
普通事件: 查询数据
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
BEFORE_COMMIT: 查询数据
AFTER_COMPLETION: 查询数据
AFTER_COMMIT: 查询数据
```

  **这个结果也印证了Spring事务的默认回滚规则，只有在抛出RuntimeException和Error级别的异常时才会回滚，如果是其他异常比如受检异常，则不会回滚而是提交！**

## 5.3 @Async异步事务事件

  **在上面的测试中，执行业务方法和执行事件方法的线程是同一个，不行的话可以测试一下！**   我们改造一下监听器类，加上线程信息，并且假设它们每一个方法执行需要一秒钟时间：

```java
@Component
public class MyTransactionalEventListener {
   

    /**
     * 事件回滚后监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK,fallbackExecution = true)
    public void afterRollback(String str) {
   
        //假设需要一秒钟处理时间
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
        System.out.println("-----AFTER_ROLLBACK: " + Thread.currentThread().getName());
        //获取事件传递的数据
        System.out.println("AFTER_ROLLBACK: " + str);
    }

    /**
     * 事件提交前监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT,fallbackExecution = true)
    public void beforeCommit(String str) {
   
        //假设需要一秒钟处理时间
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
        System.out.println("-----BEFORE_COMMIT: " + Thread.currentThread().getName());
        //获取事件传递的数据
        System.out.println("BEFORE_COMMIT: " + str);
    }

    /**
     * 事件提交后监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT,fallbackExecution = true)
    public void afterCommit(String str) {
   
        //假设需要一秒钟处理时间
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
        System.out.println("-----AFTER_COMMIT: " + Thread.currentThread().getName());
        //获取事件传递的数据
        System.out.println("AFTER_COMMIT: " + str);
    }

    /**
     * 事件完成后监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION,fallbackExecution = true)
    public void afterCompletion(String str) {
   
        //假设需要一秒钟处理时间
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
        System.out.println("-----AFTER_COMPLETION: " + Thread.currentThread().getName());
        //获取事件传递的数据
        System.out.println("AFTER_COMPLETION: " + str);
    }

    //普通事件监听器

    /**
     * 普通事件监听器
     *
     * @param str 发布的事件，要求是String及其兼容类型
     */
    @EventListener
    public void listen(String str) {
   
        //假设需要一秒钟处理时间
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
        System.out.println("-----普通事件 String: " + Thread.currentThread().getName());
        //获取事件传递的数据
        System.out.println("普通事件: " + str);
    }

    /**
     * 普通事件监听器
     *
     * @param integer 发布的事件，要求是Integer及其兼容类型
     */
    @EventListener
    public void listen(Integer integer) {
   
        //假设需要一秒钟处理时间
        LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
        System.out.println("-----AFTER_ROLLBACK: " + Thread.currentThread().getName());
        //获取事件传递的数据
        System.out.println("普通事件 Integer: " + integer);
    }
}
```

  再改造一下发布事件的service方法，同样加上线程信息：

```java
@Transactional
public void select() {
   
    System.out.println("-----service select: " + Thread.currentThread().getName());
    applicationEventPublisher.publishEvent("查询数据");
    String sql = "select * from tx_study where id = ?";
    System.out.println(jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), 1));
}
```

  执行测试：

```java
@Test
public void test() {
   
    long l = System.currentTimeMillis();
    eventService.select();
    System.out.println("业务方法返回耗时： " + (System.currentTimeMillis() - l));
}
```

结果如下：

```java
-----service select: main
-----普通事件 String: main
普通事件: 查询数据
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
-----BEFORE_COMMIT: main
BEFORE_COMMIT: 查询数据
-----AFTER_COMMIT: main
AFTER_COMMIT: 查询数据
-----AFTER_COMPLETION: main
AFTER_COMPLETION: 查询数据
业务方法返回耗时： 4476
```

  **很明显，无论是执行service业务方法，还是执行事件方法，都是同一个main线程搞定的，业务方法返回耗费了超过4秒。实际上Spring提供的事件通知机制默认情况下都是同步事件，即谁发出的事件，谁就去处理事件！**   这种事件发布机制可能违背了一些使用者的初衷，他们希望在发布事件之后，由其他的线程取处理发件的方法，这样业务线程就能避免担任过多的职责而造成比如响应缓慢等问题，提升用户的体验，这就是类似于一个简单的MQ消息队列了！为此我们可以使用Spring提供的@Async异步任务机制，用来开启异步事务事件处理。关于更详细异步任务，我们以前也讲过了，下面主要讲解如何使用！   **首先我们需要在EventStart配置类上添加@EnableAsync注解，这表示开启@Async注解的支持，随后我们在MyTransactionalEventListener监听器类上加上@Async注解，表示尝试为所有方法都开启异步任务，也可以加在指定的方法上，表示为方法开启异步任务！关于Spring异步任务，我们在此前就讲过了：Spring 5.x 学习(7)—@Async异步任务机制应用详解。**   **随后在EventStart中配置一个用于执行异步任务的线程池执行器：**

```java
/**
 * 配置了一个Spring的ThreadPoolTaskExecutor线程池，用于执行异步任务
 */
@Bean
public ThreadPoolTaskExecutor taskExecutor() {
   
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    //配置核心线程数
    executor.setCorePoolSize(5);
    //配置最大线程数
    executor.setMaxPoolSize(10);
    //配置队列大小
    executor.setQueueCapacity(800);
    //配置线程池中的线程的名称前缀
    executor.setThreadNamePrefix("TransactionalExecutor-");
    // rejection-policy：拒绝策略，由调用者所在的线程来执行
    executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
    return executor;
}
```

  再次执行测试：

```java
-----service select: main
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
业务方法返回耗时： 464


-----AFTER_COMMIT: TransactionalExecutor-3
-----BEFORE_COMMIT: TransactionalExecutor-2
BEFORE_COMMIT: 查询数据
AFTER_COMMIT: 查询数据
-----AFTER_COMPLETION: TransactionalExecutor-4
-----普通事件 String: TransactionalExecutor-1
普通事件: 查询数据
AFTER_COMPLETION: 查询数据
```

  **可以发现，业务方法在调用成功之后立即返回，这些事件方法的执行使用不同的线程，成功的实现了异步事件任务！这就是异步任务的好处！同时，异步任务的缺点之一就是，由于采用了多个线程执行不同的任务，它只能控制任务开始执行的顺序，此后这些任务具体谁先执行完谁后执行完都是不可控的，因此，对于执行先后顺序有严格要求的多个任务不适合多线程异步任务，或者，异步任务执行器可以只开启一条线程来解决！**

**相关文章：**    [https://spring.io/](https://spring.io/)    [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)    [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

