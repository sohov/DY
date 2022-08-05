

>  基于最新Spring 5.x，详细介绍了Spring的事务管理机制的概念和使用。包括编程式事务和声明式事务。

  要想比较轻松的学习Spring事务，先决条件是了解事务的基础知识，比如什么是事务、事务的ACID特性、事务的异常、事务的隔离级别等一些数据库的基本知识，因为事务的概念，就是来自于数据库。对于这些基础知识，本文默认大家了解。

  **本文主要讲解一下部分的知识，并没有涉及到源码，都是基于本地事务，即单个数据库，适合初学者**：

1. **Spring事务管理抽象架构的核心类和API，以及如何配置Spring事务；** 
2. **Spring事务的相关特性，比如传播行为、隔离级别、回滚规则、超时时间等；** 
3. **声明式事务管理的使用；** 
4. **编程式事务管理的使用；**







  **在数据库中，事务是工作的逻辑单元，一个事务是由一个或多个完成一组的相关行为的SQL语句组成，通过事务机制确保这一组SQL语句所作的操作要么都成功执行，完成整个工作单元操作，要么一个也不执行(都失败)。** 重要的是，一个事务当中的所有操作要么都成功，要么都失败，这样的特性，保证了用户每一个操作的可靠性，即使中途出现异常，也不会破坏原来的数据。比如常见的转帐案例，因为转账操作分为多个步骤，这就要用事务来处理，用以保证数据的一致性。   **事务是数据库提供的特性**，因此我们可以直接通过操作数据库来操作事务，但是作为Java开发工程师，更多的是使用现成的Java事务API来管理事务，这些API都非常的底层，对于大多数程序员来说，仅仅需要会用就行了！Java的API都是通过连接（Connection）来操作数据库的，这里的Connection，本质上就是一个项目到数据库的Socket连接，建立连接之后就能向数据库发送各种操作指令！事务的管理也是基于连接，并且一个事务对应单个连接！   **我们本次学习的Spring事务管理API，或者说以前学过的JDBC原始事务管理API，其最终还是通过调用数据库本身的机制来实现事务的，但是，Spring提供了一种更加简单并且强大的的方式对事务管理进行了支持！我们主要学习的就是Spring事务管理的特性以及如何使用，随后会学习一部分源码！**   全面的事务支持是使用Spring框架的如此火热的原因之一，同时Spring的事务管理属于Spring Framework部分的核心知识， Spring事务同样是基于AOP机制实现的，可以看作是AOP的最佳实践之一。Spring框架的事务管理有以下好处：

1. **与Spring数据访问抽象完美集成**，Spring数据访问抽象定义了访问数据库的标准，通过此可以引入其他的数据访问框架！ 
2. **为不同的数据访问框架的事务管理提供了一致的抽象编程模型**。比如Java Transaction API (JTA)、JDBC、Hibernate、Java Persistence API (JPA)、Mybatis等，它们都有自己的控制事务的方式，但是Spring数据访问抽象能够与这些外部框架无缝集成，而Spring事务管理又与数据访问抽象完美集成，那么当在Spring项目中引入这些框架之后，可以统一使用Spring提供的API或者声明方式来管理事务。 
3. **提供了声明式事务管理和编程式事务管理的支持**。大多数用户更喜欢声明式事务管理，在大多数情况下，Spring也建议采用声明式事务管理。 
4. **支持全局事务**（Global Transactions，也就是分布式事务，单个操作涉及多个不同的数据源/数据库）和本地事务（Local Transactions，单个操作设计一个数据源/数据库）的所有功能。相比于传统全局事务API（如JTA），Spring提供了更加简单的编程式事务管理API。相比于传统本地事务编程API，Spring提供了声明式事务编程，降低了代码侵入性！


  **Spring事务管理抽象的核心接口就是PlatformTransactionManager事务管理器、TransactionDefinition事务定义、TransactionStatus事务状态。我们在开发中非常有可能不直接接触这些API，特别是Spring Boot项目中。**

## 2.1 PlatformTransactionManager事务管理器

  **Spring事务抽象的关键是事务策略的概念。事务策略由TransactionManager接口定义，TransactionManager接口仅仅是一个标志性接口，没有任何方法可以实现，如果实现了这个接口，那么就标志着该类属于一个事务管理器。** 它的org.springframework.transaction.**PlatformTransactionManager**子接口是命令式事务管理的核心接口，而org.springframework.transaction.**ReactiveTransactionManager**子接口则是响应事务管理的核心接口，这是Spring 5.2新增的功能，用于配合Spring Data MongoDB 2.2和Spring Data R2DBC 1.0依赖。   我们更常用的是**PlatformTransactionManager**，即命令式事务管理器，它的API如下：

```java
/**
 * 命令式事务管理器
 */
public interface PlatformTransactionManager extends TransactionManager {
   

    /**
     * 返回当前活动事务（当前线程栈的事务）或根据指定的传播行为创建一个新事务
     *
     * @param definition 事务定义实例（可为null），描述了一个事务的传播行为、隔离级别、超时时间等属性。
     * @return 表示新事务或当前事务的事务状态对象
     */
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    /**
     * 提交给定事务，实际上和该事物的状态有关，如果事务状态被标记为回滚，则执行回滚而不是提交。
     *
     * @param status 事务状态对象，由getTransaction方法返回
     */
    void commit(TransactionStatus status) throws TransactionException;

    /**
     * 执行给定事务的回滚。
     *
     * @param status 事务状态对象，由getTransaction方法返回
     */
    void rollback(TransactionStatus status) throws TransactionException;
}
```

  **主要包括事务的提交的方法commit，回滚的方法rollback，以及根据传递的事务定义TransactionDefinition获取一个事务状态对象TransactionStatus的方法getTransaction，实际上获取的事务状态对象就可以看作事务对象本身。**   **这里的PlatformTransactionManager是一个服务提供方接口，采用的是Service Provider Interface服务注册发现机制，即SPI，这是一种JDK自带的服务发现机制，在这里用于实现事务管理器服务的扩展发现。**   PlatformTransactionManager虽然是一个服务提供方接口，但是**spring-tx自己仍然提供了一些默认实现**，常见实现是org.springframework.transaction.jta.**JtaTransactionManager**，用于管理全局事务（分布式事务），org.springframework.jdbc.datasource.**DataSourceTransactionManager**，用于管理本地事务，使用Spring JDBC或者myBatis进行数据库访问和操作时使用。其他的外部jar包实现类还有org.springframework.orm.jpa.**JpaTransactionManager**，依赖JPA规范的API进行数据库访问和操作时可以使用，org.springframework.orm.hibernate5.**HibernateTransactionManager**，依赖Hibernate进行数据库访问和操作时可以使用。

### 2.1.1 扩展-SPI简介



  **API是接口和实现类都由服务提供方实现，调用方仅仅只有调用方法的权限，一般的，如果接口和实现类位于同一个jar包体系中，就属于API。而SPI，则是由服务调用方提供接口，由服务实现方（提供方）提供接口的实现。以数据库驱动Driver来说，我们的项目作为使用者，仅仅提供了Driver驱动接口（本地JDK提供的），而具体的驱动实现则是不同的数据库厂商来实现的，我们引入的数据驱动jar包的中并不包含Driver接口，仅包含实现，这就是典型的SPI机制。**   使用SPI机制的服务提供方，需要在classpath下的META-INF/services目录中新建一个以服务接口全路径为名的文件，内容则是服务实现类的全路径名，一行一个实现。服务实现的查找、加载则是通过JDK自带的java.util.ServiceLoader类的load静态方法实现的，该方法会查找全部本地以及引入的jar包中的META-INF/services目录下的具体服务实现类，并且封装到一个ServiceLoader对象中，该对象是一个Iterable的实现，可以进行for-each循环迭代！此后，我们就可以对ServiceLoader进行迭代来选择适合自己需求的服务实现，在迭代的时候会实例化全部服务实现类！   DriverManager作为数据库驱动的发现与管理者，在加载DriverManager类时，就会在静态块中采用SPI机制通过ServiceLoader.load方法来发现和注册数据库驱动，这里也是双亲委派模型的破坏的一个案例。   以下是mysql 8的驱动jar包中的文件，可以看到符合SPI的规则，那么DriverManager类在加载时将会在静态块张中自动加载mysql驱动，因此如果我们使用DriverManager来操作数据库，那么我们根本不用手动通过Class.forName注册mysql驱动。 ![img](https://img-blog.csdnimg.cn/20201219215023487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   同样的，ojdbc8的jar包中也使用了SPI机制： ![img](https://img-blog.csdnimg.cn/20201219215043903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   请注意，某些数据驱动如果不符合SPI规范，比如某些低版本的驱动可能没有对应的文件，那么就仍然需要手动调用Class.forName注册驱动。   在基础服务或者一些基础框架中，SPI机制使用得非常广泛，比如JDBC的Driver、Dubbo、Druid、Kafka等等。SPI可以实现服务的动态发现、替换功能，不需要改动基础框架的源码即可实现功能扩展，符合开闭原则，提供的实现类对基础代码也没有侵入性！但是注意的是，SPI和JDBC规范是两码事，SPI是一个通用的服务发现与注册机制，后一个则是SUN公司制定的一套通过Java语言访问、操作数据库的API规范接口，各个数据库厂商提供的具体实现的jar包则被称为数据库驱动！

## 2.2 TransactionStatus事务状态

  **TransactionStatus是一个事务的状态表示接口，为事务代码提供了一些控制事务执行和查询事务状态的简单方法。PlatformTransactionManager的getTransaction方法就会返回一个TransactionStatus对象，实际上TransactionStatus对象就代表一个事务对象。**   TransactionStatus的常见方法如下：


## 2.3 TransactionDefinition事务定义

  **TransactionDefinition是一个定义了符Spring 事务属性的接口，内部具有获取各种Spring事务属性的方法，并且定义了以下几种属性常量：**

1. **Propagation：事务的传播行为。** 通常，事务范围内的所有代码都在该事务中运行。但是，如果事务上下文已存在，则可以指定传播行为。例如，代码可以在现有事务（常见情况下）中继续运行，也可以挂起现有事务并创建新的事务。实际上，Spring 提供了 EJB CMT 熟悉的所有事务传播选项。 
2. **Isolation：事物的隔离级别。** 用来表示此事务与其他事务的工作隔离的程度。例如，此事务能否看到来自其他事务的未提交的写入？ 
3. **Timeout：超时时间。** 此事务在超时之前运行多长时间，超时将自动回滚。 
4. **Read-only status：只读状态。** 当代码只有读取没有修改数据时，可以使用只读事务。在某些情况下，只读事务是一种有用的优化，例如当使用 Hibernate的时候。

```java
/**
 * 事务定义接口
 */
public interface TransactionDefinition {
   

    /*传播行为常量*/

    int PROPAGATION_REQUIRED = 0;

    int PROPAGATION_SUPPORTS = 1;

    int PROPAGATION_MANDATORY = 2;

    int PROPAGATION_REQUIRES_NEW = 3;

    int PROPAGATION_NOT_SUPPORTED = 4;

    int PROPAGATION_NEVER = 5;

    int PROPAGATION_NESTED = 6;

    /*隔离级别常量*/

    int ISOLATION_DEFAULT = -1;

    int ISOLATION_READ_UNCOMMITTED = 1;

    int ISOLATION_READ_COMMITTED = 2;

    int ISOLATION_REPEATABLE_READ = 4;

    int ISOLATION_SERIALIZABLE = 8;

    /*默认超时时间常量*/

    int TIMEOUT_DEFAULT = -1;


    /**
     * 返回传播行为。必须返回此接口规定的传播行为常量中的一个
     * 默认值为 PROPAGATION_REQUIRED
     *
     * @return 传播行为
     */
    default int getPropagationBehavior() {
   
        return PROPAGATION_REQUIRED;
    }

    /**
     * 返回隔离级别。必须返回此接口规定的隔离级别常量中的一个，这些常量与java.sql.Connection中的常量匹配
     * 专门设计用于PROPAGATION_REQUIRED 或 PROPAGATION_REQUIRES_NEW，因为它仅适用于新启动的事务。
     * <p>
     * 默认值为 ISOLATION_DEFAULT 。请注意，不支持自定义隔离级别的事务管理器在返回给定ISOLATION_DEFAULT以外的任何其他级别时将引发异常。
     *
     * @return 隔离级别
     */
    default int getIsolationLevel() {
   
        return ISOLATION_DEFAULT;
    }

    /**
     * 返回事务超时时间。必须返回秒级别的时间，或TIMEOUT_DEFAULT
     * 专门设计用于PROPAGATION_REQUIRED 或 PROPAGATION_REQUIRES_NEW，因为它仅适用于新启动的事务。
     * <p>
     * 默认值为 ISOLATION_DEFAULT 。
     *
     * @return 事务超时时间
     */
    default int getTimeout() {
   
        return TIMEOUT_DEFAULT;
    }

    /**
     * 返回是否作为只读事务进行优化。
     * <p>
     * 如果仅作为只读事务，那么返回true，默认返回false
     */
    default boolean isReadOnly() {
   
        return false;
    }

    /**
     * 返回此事务的名称。可以是null。将用作要在事务监视器中显示的事务名称（如果适用）（例如WebLogic中）。
     * <p>
     * 在 Spring 声明性事务中，名称默认是 所属类的完全限定的类名+ "." + 方法名
     */
    @Nullable
    default String getName() {
   
        return null;
    }


    // 静态生成器方法

    /**
     * 返回具有默认值的不可修改的事务定义。
     * <p>
     * 如果是出于自定义目的，应该使用可修改的DefaultTransactionDefinition代替。
     *
     * @since 5.2
     */
    static TransactionDefinition withDefaults() {
   
        return StaticTransactionDefinition.INSTANCE;
    }

}
```

### 2.3.1 Propagation事务的传播行为

  **事务的传播行为是Spring的特性，它指的是多个事务方法之间相互调用时，事务如何在这些方法间传播。比如多个方法的事务是合并到一起提交/回滚，还是这些方法各自作为独立的事务分别提交/回滚（也就是嵌套事务），这就是事务传播行为来确定的。由于Mysql数据库本身不支持嵌套事务，因此实际上Spring的“嵌套事务”是通过mysql的Savepoint保存点来实现的！**   **如果使用声明式事务管理，那么Spring采用的是AOP来支持事务管理，在方法前会根据配置的事务属性来决定是否需要通过AOP拦截器来开启一个事务。因此，传播行为主要是针对声明式事务管理的扩展。**   **Spring提供了七种事物的传播行为！下面的“当前事务”是指当前线程调用栈中是否已经开启过事务！**

#### 2.3.1.1 PROPAGATION_REQUIRED

  **值为0，Spring默认的传播行为。如果当前存在事务，则当前方法加入到当前事务中去，使用当前外层事务的属性。如果当前不存在事务，就创建一个新事务运行。**   有一个事务方法m1，它的传播行为是PROPAGATION_REQUIRED，如果它被调用时不存在事务，那么它将开启一个新事物，如果在m1中调用了事务方法m2，并且m2的传播行是PROPAGATION_REQUIRED，那么m2将加入到m1的事务中去。   **注意，这里和后面的“加入”的含义，表示使用同一个物理事务，内部的事务将会静默的忽略掉自己的设置的隔离级别、超时时间、只读标注这几个属性，这些属性统一使用最外部事务方法的设置值。其他属性比如rollbackFor回滚异常类型，则针对单个方法可以单独设置，最终该事物是否回滚是通过判断所有外层和内层的事务方法是否都回滚或者不回滚来设置的，如果有任何一个方法指定回滚，则所有方法和操作都会回滚！**   **另外，如果仅仅是在内层加入的方法中通过setRollbackOnly方法设置为仅回滚（不是在最外层事务方法中设置的），并且事务回滚时如果没有其他异常抛出，则会抛出一个异常：“UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only”，来提醒开发者所有的外部和内部操作都已被回滚！这一般对于内层PROPAGATION_SUPPORTS或者内层PROPAGATION_REQUIRED或者内层PROPAGATION_MANDATORY生效，对于内层PROPAGATION_NOT_SUPPORTED则无效。**

  如果单独调用m1：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //do something
}
```

  那么相当于开启一个事务：

```java
public void tran() {
   
    //获得一个事务管理器
    DataSourceTransactionManager txManager = getTransactionManager();
    //m1开启一个新事务
    TransactionStatus status = txManager.getTransaction(TransactionDefinition);
    try {
   
        //执行m1方法业务逻辑
        m1();
    } catch (Exception ex) {
   
        //回滚
        txManager.rollback(status);
        throw ex;
    }
    //提交
    txManager.commit(status);
}
```

  如果m1中调用m2：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //do something
    m2();
}

@Transactional(propagation = Propagation.REQUIRED)
public void m2() {
   
    //do something
}
```

  那么m2不会再开启事务，而是加入到m1开启的事务中去：

```java
public void tran() {
   
    //获得一个事务管理器
    DataSourceTransactionManager txManager = getTransactionManager();
    //m1开启一个新事务
    TransactionStatus status = txManager.getTransaction(TransactionDefinition);
    try {
   
        //执行m1方法业务逻辑，m1内部还调用m2方法，m2不再开启新事务
        m1();
    } catch (Exception ex) {
   
        //回滚
        txManager.rollback(status);
        throw ex;
    }
    //提交
    txManager.commit(status);
}
```

#### 2.3.1.2 PROPAGATION_SUPPORTS

  **值为1。如果当前存在事务，当前方法加入到当前事务中去，如果当前不存在事务，则当前方法直接以非事务的方式执行。**   有一个事务方法m2，它的传播行为是PROPAGATION_SUPPORTS，如果它被调用时不存在事务，那么它将以非事务的方式运行，如果M2被调用时已经在调用方法中开启了事务，那么M2将加入到当前的外层事务中去。

  如果单独调用m2：

```java
@Transactional(propagation = Propagation.SUPPORTS)
public void m2() {
   
    //do something
}
```

  那么相当于不开启任何事务，当作普通方法运行：

```java
public void tran() {
   
    //单独调用m2，相当于不开启事务
    m2();
}
```

  如果在开启了事务的方法m1中调用m2：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //do something
    m2();
}

@Transactional(propagation = Propagation.SUPPORTS)
public void m2() {
   
    //do something
}
```

  那么m2加入到m1的事务中去，这和上面是一样的！

#### 2.3.1.3 PROPAGATION_MANDATORY

  值为2。如果当前存在事务，则当前方法加入到该事务中去，如果当前不存在事务，则当前方法直接抛出异常。   有一个事务方法M2，它的传播行为是PROPAGATION_MANDATORY，如果它被调用时不存在事务，那么它将直接抛出异常：new IllegalTransactionStateException(“No existing transaction found for transaction marked with propagation ‘mandatory’”)，如果M2被调用时已经在调用方法中开启了事务，那么M2将加入到当前的外层事务中去。

  如果单独调用m2：

```java
@Transactional(propagation = Propagation.MANDATORY)
public void m2() {
   
    //do something
}
```


  那么抛出异常： ![img](https://img-blog.csdnimg.cn/20201219215936796.png#pic_center)

  如果在开启了事务的方法m1中调用m2：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //do something
    m2();
}

@Transactional(propagation = Propagation.MANDATORY)
public void m2() {
   
    //do something
}
```

  那么m2加入到m1的事务中去，这和上面是一样的！ 那么m2加入到m1的事务中去，这和上面是一样的！

#### 2.3.1.4 PROPAGATION_REQUIRES_NEW

  **值为3。当前方法开启一个新事物独立运行，从不参与外部的现有事务。则当内部事务开始执行时，外部事务（如果存在）将被挂起，内务事务结束时，外部事务将继续执行。**   **始终对每一个方法开启一个新的物理事务，内部事务和外部事务是相互独立的，内部事务可以独立提交或回滚，外部事务不受内部事务的回滚状态的影响，并且内部事务的锁在完成后立即释放。这种独立的内部事务还可以声明其自己的隔离级别、超时和只读设置，而不是继承外部事务的特征。**

  如果单独调用m2：

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void m2() {
   
    //do something
}
```

  那么相当于开启一个事务：

```java
public void tran() {
   
    //获得一个事务管理器
    DataSourceTransactionManager txManager = getTransactionManager();
    //m2开启一个新事务
    TransactionStatus status = txManager.getTransaction(TransactionDefinition);
    try {
   
        //执行m2方法业务逻辑
        m2();
    } catch (Exception ex) {
   
        //回滚
        txManager.rollback(status);
        throw ex;
    }
    //提交
    txManager.commit(status);
}
```

  如果在开启了事务的方法m1中调用m2：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //m1 do something1
    m2();
    //m1 do something2
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void m2() {
   
    //do something
}
```

  那么将会创建两个物理级别的事务，它们之间没有联系，并且外部事物会先被挂起，内部事务执行完毕之后再恢复：

```java
public void tran() {
   
    //获得一个事务管理器
    DataSourceTransactionManager txManager = getTransactionManager();
    //m1开启一个新事务ts1
    TransactionStatus ts1 = txManager.getTransaction(TransactionDefinition);
    try {
   

        //m1 do something1

        //挂起m1的ts1事务
        //这个方法实际上由Spring来调用，位于AbstractPlatformTransactionManager中
        suspend(ts1);

        //m2开启一个新事务ts2
        TransactionStatus ts2 = txManager.getTransaction(TransactionDefinition);
        try {
   
            //执行m2方法业务逻辑
            m2();
        } catch (Throwable ex) {
   
            //回滚ts2事务
            txManager.rollback(ts2);
        }
        //提交ts2事务
        txManager.commit(ts2);

        //恢复m1的ts1事务
        //这个方法实际上由Spring来调用，位于AbstractPlatformTransactionManager中
        resume(ts1);

        //m1 do something2

    } catch (Throwable throwable) {
   
        //回滚ts1
        txManager.rollback(ts1);
    }
    //提交ts1
    txManager.commit(ts1);
}
```

#### 2.3.1.5 PROPAGATION_NOT_SUPPORTED

  值为4。当前方法一定以非事务的方式运行，如果当前存在事务，则把当前事务挂起，直到当前方法执行完毕，才恢复外层事务。

  如果单独调用m2：

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void m2() {
   
    //do something
}
```

  那么相当于不开启任何事务，当作普通方法运行：

```java
public void tran() {
   
    //单独调用m2，相当于不开启事务
    m2();
}
```

  如果在开启了事务的方法m1中调用m2：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //m1 do something1
    m2();
    //m1 do something2
}

@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void m2() {
   
    //do something
}
```

  那么会将当前事务挂起，并且对m2以非事务的方式运行，随后恢复当前事务：

```java
public void tran() {
   
    //获得一个事务管理器
    DataSourceTransactionManager txManager = getTransactionManager();
    //m1开启一个新事务ts1
    TransactionStatus ts1 = txManager.getTransaction(TransactionDefinition);
    try {
   

        //m1 do something1

        //挂起m1的ts1事务
        //这个方法实际上由Spring来调用，位于AbstractPlatformTransactionManager中
        suspend(ts1);

        //执行m2方法业务逻辑，以非事务的方式执行
        m2();

        //恢复m1的ts1事务
        //这个方法实际上由Spring来调用，位于AbstractPlatformTransactionManager中
        resume(ts1);

        //m1 do something2

    } catch (Throwable throwable) {
   
        //回滚ts1
        txManager.rollback(ts1);
    }
    //提交ts1
    txManager.commit(ts1);
}
```

#### 2.3.1.6 PROPAGATION_NEVER

  **值为5。当前方法一定以非事务的方式运行，并且如果当前存在事务，则直接抛出异常：new IllegalTransactionStateException(“Existing transaction found for transaction marked with propagation ‘never’”)。这和PROPAGATION_MANDATORY正好相反。**

  如果单独调用m2：

```java
@Transactional(propagation = Propagation.NEVER)
public void m2() {
   
    //do something
}
```

  那么相当于不开启任何事务，当作普通方法运行：

```java
public void tran() {
   
    //单独调用m2，相当于不开启事务
    m2();
}
```

  如果在开启了事务的方法m1中调用m2：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //m1 do something1
    m2();
    //m1 do something2
}

@Transactional(propagation = Propagation.NEVER)
public void m2() {
   
    //do something
}
```


  那么直接抛出异常： ![img](https://img-blog.csdnimg.cn/20201219220232614.png#pic_center)

#### 2.3.1.7 PROPAGATION_NESTED

  **值为6。如果当前存在事务，则创建一个新“事务”作为当前事务的嵌套事务来运行；如果当前没有事务，则等价于PROPAGATION_REQUIRED，即会新建一个事务运行。**   **内层PROPAGATION_NESTED方法的传播行为通常被称为会开启“嵌套事务”，然而，这个“嵌套事务”实际上是通过SavePoint保存点来实现的，这个保存点属于当前已存在的外部事物，所以说仍然只有一个物理事物，这就是真正的“嵌套事务”的实现，仅仅是保存点而已。基于保存点的特性，此时“内层”事务依赖“外层”事物（实际上就是同一个事务）。内层事务操作失败时只是自身回到到保存点的位置，不会引起外层事务的回滚，而外层事务因失败而回滚时，内层事务所做的所有动作也会回滚。在提交时，在外层事务提交之后内层事务才能提交，仅需要提交外层事务即可。由于实际上只有一个物理事务，那么内层事务会继承外层外层事务的隔离级别和超时设置等属性。**   **想要使用PROPAGATION_NESTED，还需要把AbstractPlatformTransactionManager的nestedTransactionAllowed属性设为true（默认为false）。但是DataSourceTransactionManager构造器会自动将其设置为true，所以我们一般不用设置。**   **由于PROPAGATION_REQUIRED实际上仅会开启单个物理事务，因此隔离级别、超时时间、只读标注这几个属性，这些属性仍然统一使用最外部事务方法的设置值。**

  如果单独调用m2：

```java
@Transactional(propagation = Propagation.NESTED)
public void m2() {
   
    //do something
}
```

  那么相当于开启一个事务：

```java
public void tran() {
   
    //获得一个事务管理器
    DataSourceTransactionManager txManager = getTransactionManager();
    //m2开启一个新事务
    TransactionStatus status = txManager.getTransaction(TransactionDefinition);
    try {
   
        //执行m2方法业务逻辑
        m2();
    } catch (Exception ex) {
   
        //回滚
        txManager.rollback(status);
        throw ex;
    }
    //提交
    txManager.commit(status);
}
```

  如果在开启了事务的方法m1中调用m2：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void m1() {
   
    //m1 do something1
    m2();
    //m1 do something2
}

@Transactional(propagation = Propagation.NESTED)
public void m2() {
   
    //do something
}
```

  那么仅仅会创建一个物理级别的事务，内部嵌套子事务以SavePoint保存点的方法记录，这是真正的嵌套事务，“内部”事务受到“外部”事物的影响：

```java
public void tran() {
   
    //获得一个事务管理器
    DataSourceTransactionManager txManager = getTransactionManager();
    //m1开启一个新事务ts1
    TransactionStatus ts1 = txManager.getTransaction(TransactionDefinition);
    try {
   

        //m1 do something1

        //m2开启一个新事务ts2
        //实际上就是在当前事务中创建一个保存点而已
        Object savepoint = ts1.createSavepoint();

        try {
   
            //执行m2方法业务逻辑
            m2();
        } catch (Throwable ex) {
   
            //回滚ts2事务，这里仅仅是对保存点的操作
            ts1.rollbackToSavepoint(savepoint);
        }

        //m1 do something2

    } catch (Throwable throwable) {
   
        //回滚ts1事务
        txManager.rollback(ts1);
        throw throwable;
    }
    //提交ts1事务
    txManager.commit(ts1);
}
```

### 2.3.2 Isolation事务的隔离级别

  **数据库允许多个事务的并行，事物的隔离级别就用来表示此事务与其他并行事务的工作隔离的程度。** 例如，此事务能否看到来自其他事务的未提交的写入？不同的数据库有不同的默认事务隔离级别。由于并行事务，不同的隔离级别可能会带来不同的数据访问现象/安全问题。换句话说，**事务的隔离级别是根据是否会出现下面的某些现象来区分的！**   **事务的隔离级别是数据库事务自身的特性，Spring和JDBC中的隔离级别常量仅仅是为了与数据库的隔离级别对应！也就是说，这个隔离级别最终还是依靠数据库来实现的！**

  在SQL92标准中，针对大部分数据库定义了4个标准的事务隔离级别：

1. **读未提交(Read Uncommitted）** 
 <ol> 
  1. **读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。** 
  1. 一个事务在执行过程中，既可以访问其他事务未提交的新插入的数据， 又可以访问未提交的修改数据。如果一个事务已经开始写数据，则另外一个事务则不允许同时进行写操作，但允许其他事务读此行数据。此隔离级别可防止丢失更新，但是存在脏读、不可重复读、幻读的问题。 
  1. 写事务阻止其他写事务，避免了丢失更新，但是没有阻止其他读事务；读事务则不会阻止其他任何事务。 
 </ol>  
5. **读已提交(Read Committed）** 
 <ol> 
  5. **Sql Server , Oracle的默认隔离级别。读已提交，顾名思义，就是一个事务要等另一个事务提交后才能读取数据。** 
  5. 解决了脏读问题，还有不可重复读和幻读的问题。 
  5. 写事务会阻止其他读写事务。读事务不会阻止其他任何事务。 
 </ol>  
9. **可重复读(Repeatable Read）** 
 <ol> 
  9. **MySQL的默认隔离级别。可重复读，就是在开始读取数据（事务开启）时，不再允许修改操作。** 
  9. 写事务会阻止其他读写事务，读事务会阻止其他写事务，但不会阻止其他读事务。可重复读阻止的写事务仅包括update和delete，不包括insert。因此可能出现幻读，但这不是绝对的。 
  9. **实际上，mysql在默认的可重复读隔离级别下，mvcc的普通的查询是快照读，提供了一致性视图，不会看到别的事务插入的数据的，也就不存在所谓的“幻读”了，而如果使用当前读，那么则会加上行锁+Gap间隙锁，其他事务的插入操作则根本无法进行，因此实际上mysql的InnoDB引擎已经使用mvcc解决了幻读的问题（见《高性能MySQL》）。那么这里的幻读是什么呢？这里更多是指的oracle的操作，所以你用mysql测试的话，幻读很有可能测不出来。** 
 </ol>  
13. **序列化／串行化(Serializable）** 
 <ol> 
  13. **Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读问题，严重影响程序性能。** 
 </ol> 

  **SQL92标准的隔离级别与是否会发生的异常现象对应关系为：**


  事务中遇到的这些异常与事务的隔离级别设置有关，事务的隔离级别设置越高，异常就出现的越少，但并发效果就越低，事务的隔离级别设置越小，异常出现的越多，并发效果越高。   **MySQL默认隔离级别： Repeatable Read，并且支持全部四种级别。**   **Oracle默认的隔离级别是read committed，并且支持上述四种隔离级别中的两种:read committed 和serializable。**

#### 2.3.2.1 常见数据访问异常

  **对于不同的事务隔离级别，可能会下面的某些现象（Phenomena）/数据异常，这些现象表示的是符合条件的的数据集在并发事务中可能更改的方式。**   一共有三种现象：脏读（Dirty Read）、不可重复读（Non-Repeatable Read）、幻读（Phantom Read，不一定会发生）：

1. 脏读（Dirty Read） 
 <ol> 
  1. 当一个事务修改数据时，另一事务读取了该数据，但是第一个事务由于某种原因取消对数据修改，使数据返回了原状态，这时第二个事务读取的数据与数据库中数据不一致，这就叫脏读。脏读导致一个事务读取了被其他事务修改但还未提交的数据。 
  1. 如：事务T1修改了一条数据，但是还未提交，事务T2此时读取到了这条修改后了的数据，如果此时T1将事务回滚，这个时候T2读取到的数据就是脏数据。 
 </ol>  
4. 不可重复读（Non-Repeatable Read） 
 <ol> 
  4. 是指一个事务读取数据库中的数据后，另一个事务则修改或删除数据，当第一个事务再次执行同一查询时，就会发现数据已经发生了改变，这就是不可重复读。不可重复读所导致的结果就是一个事务前后两次读取的数据不相同，可以读取到其他事务更新/删除后的数据。 
  4. 即不可重复读发生在一个事务执行两次或两次以上的相同查询时，查询结果不一致。这通常是由于另一个并发事务在两次查询之间修改/删除并提交了符合查询条件的数据。 
  4. 如：事务T1按照条件读取一批记录，紧接着事务T2修改并提交了T1刚刚读取的那一批记录，然后T1再一次查询，发现与第一次读取的记录不同。 
 </ol>  
8. 幻读（Phantom Read，不一定会发生） 
 <ol> 
  8. 是指一个事务读取数据库中的数据后，另一个事务则插入了数据数据，当第一个事务再次执行同一查询时，就会发现数据已经发生了改变，多出来了数据，这就是幻读。幻读所导致的结果就是一个事务前后两次读取的数据不相同，可以读取到其他事务新插入的数据。 
  8. 即幻读发生在一个事务执行两次或两次以上的相同查询时，查询结果不一致。这通常是由于另一个并发事务在两次查询之间插入并提交了符合查询条件的数据。 
  8. 如：事务T1按照条件读取一批记录，紧接着事务T2插入并提交了符合T1查询条件的记录，然后T1再一次查询，发现与第一次读取的记录不同。 
 </ol> 

#### 2.3.2.2 Spring事务隔离级别

  **Spring提供的事务隔离级别常量就是为了和上面四种级别对应而已：**


  **事务的隔离级别是在一个事务启动的时候设置的，因此，只有对于那些具有可能启动一个新事务的传播行为（PROPAGATION_REQUIRES_NEW、PROPAGATION_REQUIRED、ROPAGATION_NESTED）的方法来说，声明的事务隔离级别参数才有意义。**

### 2.3.3 Timeout事务的超时时间

  **表示允许一个事务执行的最长时间，时间单位为秒，默认值为-1，表示没有超时时间。如果超过该时间限制但事务还没有完成，则自动回滚事务。**   **事务的超时时间是在一个事务启动的时候开始的计算的，因此，只有对于那些具有可能启动一个新事务的传播行为（PROPAGATION_REQUIRES_NEW、PROPAGATION_REQUIRED、ROPAGATION_NESTED）的方法来说，声明的事务超时参数才有意义。**

### 2.3.4 Read-only事务的只读状态

  **当代码只有读取没有修改数据时，可以设置readOnly属性为true（默认为false），表示一个这是一个只读事务。在某些情况下，只读事务对于某些数据库来说可能是一种有用的优化机会。**   **事务的只读状态是在一个事务启动的时候设置的，因此，只有对于那些具有可能启动一个新事务的传播行为（PROPAGATION_REQUIRES_NEW、PROPAGATION_REQUIRED、ROPAGATION_NESTED）的方法来说，声明的事务只读状态参数才有意义。**

### 2.3.5 rollBack事务的回滚规则

  **默认情况下，Spring事务只在出现未受检异常（unchecked exception，即运行时异常，RuntimeException的子类），以及出现Error级别的异常时回滚，而在出现受检查异常（checked exception，除了RuntimeException和Error之外的异常）时不回滚，因为受检异常可能是作为一个业务异常而代替返回值的结果，因此，如果遇到了受检查异常，仍然会提交事务。**   **同样，我们可以声明在出现特定受检查异常时像运行时异常一样回滚，也可以声明一个事务在出现特定的异常时不回滚，即使特定的异常是运行时异常。回滚规则可以通过@Transactional注解的rollbackFor属性和noRollbackFor属性，以及< tx:method/>标签的rollback-for和no-rollback-for属性来设置！**   **如果回滚和不回滚的规则设置了相同的异常，那么在抛出该异常时将会回滚！**   **如果抛出的异常没有匹配我们设定的规则匹配，那么仍然会采用默认规则，即当前异常属于RuntimeException或者Error级别的异常时，事务才会回滚。**   **匹配的时候实际上是采用最佳匹配规则，详细规则在Spring事务的源码的RuleBasedTransactionAttribute部分有讲到！**

## 2.4 TransactionManager的配置

  **TransactionManager事务管理器的getTransaction方法根据传递的事务定义TransactionDefinition对象获取一个事务状态对象TransactionStatus，因此，无论是声明式事务管理还是编程式事务管理，TransactionManager的配置都非常的重要！**   我们可以根据不同的开发环境配置不同的事务管理器实现，最常见的就是PlatformTransactionManager接口的DataSourceTransactionManager实现类，用于管理本地事务，使用Spring JDBC或者myBatis进行数据库访问和操作时使用。下面我们看看DataSourceTransactionManager的配置！   **非常简单，我们需要告诉DataSourceTransactionManager事务管理器，它管理的是哪一个数据源/数据库的事务，因此一个DataSourceTransactionManager需要与一个DataSource连用。**

  下面是采用DataSourceTransactionManager管理事务的通用XML配置：

```java
<!--druid数据源连接池-->
<bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url"
              value="jdbc:mysql://xxx"/>
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>

<!--DataSourceTransactionManager事务管理器 用来管理数据库事务-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--配置一个druidDataSource数据源-->
    <constructor-arg name="dataSource" ref="druidDataSource"/>
</bean>

<!--开启注解事务支持，如果没采用事务注解，那么可以不配置-->
<!--<tx:annotation-driven/>-->
```

  **可以看到，还是很简单的！该配置适用于Spring JDBC和Mybatis，如果是Hibernate，那么需要配置HibernateTransactionManager并且注入LocalSessionFactoryBean，如果是JPA，那么需要配置JpaTransactionManager并且注入EntityManagerFactory。**

  下面是Java Config方式的配置，更加的简单：

```java
/**
 * 配置Druid数据源
 */
@Bean
public DruidDataSource druidDataSource() {
   
    DruidDataSource druidDataSource = new DruidDataSource();
    //为了方便，直接硬编码了，我们可以通过@Value引入外部配置，
    //如果使用Spring boot就更简单了，直接使用@ConfigurationProperties引入外部配置
    //简单的配置数据库连接信息，其他连接池信息采用默认配置
    druidDataSource.setUrl("jdbc:mysql://xxx");
    druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    druidDataSource.setUsername("root");
    druidDataSource.setPassword("123456");
    return druidDataSource;
}

/**
 1. 配置DataSourceTransactionManager
 2. 用于管理某一个数据库的事务
 */
@Bean
public DataSourceTransactionManager transactionManager() {
   
    //传入一个数据源
    return new DataSourceTransactionManager(druidDataSource());
}
```


  **编程式事务管理是我们最熟悉的事务管理方式，所谓“编程式”，简单的说，就是在业务代码中手动调用比如beginTransaction()、commit()、rollback()等方法进行事务管理。可想而知，事务代码和业务代码耦合，不利于后续开发和架构升级，但好处是事务控制的粒度非常细，可以精确到代码行级别！**   **Spring同样提供了自己的编程式事务控制的API，并且有两种方式来实现：**

1. 使用上层的**TransactionTemplate**或者transactionoperator。 
2. 直接使用的**TransactionManager**的实现。

  Spring团队通常建议在命令编程中使用TransactionTemplate进行程序化事务管理，对于响应式代码则推荐使用transactionoperator，这两个类是一个模板类，提供了统一的API。第二种方法则更加底层和麻烦，它需要使用PlatformTransactionManager、TransactionDefinition 和 TransactionStatus这三个对象来各自的实现类来进行事务管理。

## 3.1 TransactionTemplate事务管理模版

  **TransactionTemplate事务模板与其他 Spring 模板类比如 JdbcTemplate一样，它使用回调方法机制，简化、封装了原始事务方法的调用，不需要显式地进行开始事务、提交事务等方法的手动调用，我们的业务代码只关注想要做的事情。**

### 3.1.1 TransactionTemplate的配置

  **要想使用事务模版类，我们需要传递一个事务管理器类，因为TransactionTemplate底层仍然是依靠TransactionManager来实现事务管理的！**

#### 3.1.1.1 配置依赖

  **maven依赖如下，我们引入了spring-jdbc即可，jdbc的依赖中包含了JdbcTemplate，在本文中，我们直接使用Spring提供的可以JdbcTemplate来访问数据库，不再引入其他数据库框架。spring-jdbc还包含了spring-tx的依赖，spring-tx的依赖就是用于支持Spring事务管理，当然也可以单独引入。**   关于JdbcTemplate，它其实非常的简单，一篇文章就能够学会了，我们在此前就讲过了： [Spring 5.x 学习(9)—Spring JDBC(JdbcTemplate)深入学习及使用案例](https://blog.csdn.net/weixin_43767015/article/details/111038624)。

```java
<properties>
    <spring-framework.version>5.2.8.RELEASE</spring-framework.version>
    <mysql-connector-java>8.0.16</mysql-connector-java>
    <druid>1.2.3</druid>
    <lombok>1.18.12</lombok>
    <junit>4.12</junit>
</properties>
<dependencies>
    <!--spring 核心组件所需依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring-framework.version}</version>
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

#### 3.1.1.2 配置类

  配置JDBC模版与事务模版，TransactionTemplate是线程安全的，因为实例不保持任何会话状态。但是，TransactionTemplate实例会保持配置状态。因此，虽然许多类可以共享TransactionTemplate的单个实例，但**如果某个类需要使用具有不同设置的TransactionTemplate（例如，不同的隔离级别、不同的数据库、不同的transactionManager），则需要创建两个不同的TransactionTemplate实例。**   我们也可以在TransactionTemplate上指定事务属性，例如传播模式，隔离级别，超时等。默认情况下，TransactionTemplate实例具有默认的事务属性，来自于它的父类DefaultTransactionDefinition。   XML的配置也很简单，就是配置bean，这里我们采用Java Config配置，舍弃XML文件：

```java
@ComponentScan
@Configuration
public class TxStart {
   

    /**
     * 配置Druid数据源
     */
    @Bean
    public DruidDataSource druidDataSource() {
   
        DruidDataSource druidDataSource = new DruidDataSource();
        //为了方便，直接硬编码了，我们可以通过@Value引入外部配置，
        //如果使用Spring boot就更简单了，直接使用@ConfigurationProperties引入外部配置
        //简单的配置数据库连接信息，其他连接池信息采用默认配置
        druidDataSource.setUrl("jdbc:mysql://xxx");
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

    /**
     * 配置TransactionTemplate
     * 用于方便编程式的事务操作
     */
    @Bean
    public TransactionTemplate transactionTemplate() {
   
        //传入一个TransactionManager
        return new TransactionTemplate(transactionManager());
    }
}
```

#### 3.1.1.3 数据库表

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

  插入一些数据

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

#### 3.1.1.4 测试类

  我们直接使用spring-test组件进行测试，可以直接引入依赖项，比较方便！   测试类如下：

```java
/**
 * @author lx
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = TxStart.class)
public class TxTest {
   

    /**
     * jdbc模版，用于操作数据库
     */
    @Resource
    private JdbcTemplate jdbcTemplate;

    /**
     * 事务模版，用于操作事务
     */
    @Resource
    private TransactionTemplate transactionTemplate;
}
```

  我们首先看看配置成功没有：

```java
@Test
public void check() {
   
    System.out.println(jdbcTemplate);
    System.out.println(transactionTemplate);
}
```

  结果如下：

```java
org.springframework.jdbc.core.JdbcTemplate@e4487af
PROPAGATION_REQUIRED,ISOLATION_DEFAULT
```

  可以看到确实配置成功了，下面开始使用！

### 3.1.2 TransactionTemplate的使用

  **使用TransactionTemplate，最关键的就是它的execute方法。该方法传递一个TransactionCallback对象作为参数，这个TransactionCallback是一个函数式接口，可以使用lambda表达式，也可以使用匿名对象，其中就包含需要在事务上下文中运行的业务代码。**   一个使用案例如下，我们使用lambda表达式：

```java
@Test
public void transactionTemplateTest() {
   
    //transactionTemplate的execute方法便捷的处理事务
    Integer execute = transactionTemplate.execute(status -> {
   
        //在doInTransaction方法中调用业务代码，并且支持返回值
        return jdbcOperate();
    });
    System.out.println(execute);
}

/**
 * 业务操作
 */
private int jdbcOperate() {
   
    //插入的sql
    String sql = "insert into tx_study (name,age) values (?,?)";
    //调用jdbcTemplate的update方法，插入数据
    return jdbcTemplate.update(sql, "insert", 20);
}
```

  使用匿名对象的代码如下：

```java
@Test
public void transactionTemplateTest() {
   
    //transactionTemplate的execute方法便捷的处理事务
    Integer execute = transactionTemplate.execute(new TransactionCallback<Integer>() {
   
        @Override
        public Integer doInTransaction(TransactionStatus status) {
   
            //在doInTransaction方法中调用业务代码，并且支持返回值
            return jdbcOperate();
        }
    });
    System.out.println(execute);
}

/**
 * 业务操作
 */
private int jdbcOperate() {
   
    //插入的sql
    String sql = "insert into tx_study (name,age) values (?,?)";
    //调用jdbcTemplate的update方法，插入数据
    return jdbcTemplate.update(sql, "insert", 20);
}
```

  **可以看到，我们利用transactionTemplate的execute方法处理事务，业务代码就写在doInTransaction方法中，并且支持返回值。不需要显式调用任何其他的事务管理的 API。它会自动帮我开启事物、提交/回滚事务。默认情况下，如果抛出非受检异常（RuntimeException）或者Error异常则会回滚事务，抛出其他受检异常异常或者方法执行成功则会提交事务！**   下面的代码模拟抛出一个非受检异常：

```java
@Test
public void tpExTest() {
   
    //transactionTemplate的execute方法便捷的处理事务
    transactionTemplate.execute(status -> {
   
        //在doInTransaction方法中调用业务代码，并且支持返回值
        return jdbcOperateEx();
    });
}

/**
 * 业务操作，模拟抛出非受检异常
 */
private int jdbcOperateEx() {
   
    //插入的sql
    String sql = "insert into tx_study (name,age) values (?,?)";
    //调用jdbcTemplate的update方法，插入数据
    int insert = jdbcTemplate.update(sql, "insert2", 21);

    //手动制造一个非受检异常
    int i = 1 / 0;
    return insert;
}
```

  **执行之后，抛出了ArithmeticException，它是一个RuntimeException。我们查看数据库，发现数据并没有被插入进去，这说明事务自动回滚了。**   **对于受检异常，实际上使用该方法时不允许直接抛出受检异常，而是必须处理！此时我们可以使用doInTransaction方法传递的参数TransactionStatus，这个就是当前事务对象，我们可以调用它的setRollbackOnly方法手动设该事务的唯一结果就回滚，用来代替抛出异常：**

```java
@Test
public void tpCheckExTest() {
   
    //transactionTemplate的execute方法便捷的处理事务
    Integer execute = transactionTemplate.execute(status -> {
   
        //受检异常必须处理，否则编译不通过
        try {
   
            return jdbcOperateCheckEx();
        } catch (IOException e) {
   
            //调用setRollbackOnly方法回滚事务
            status.setRollbackOnly();
            return null;
        }
    });
    System.out.println(execute);
}

/**
 * 业务操作，模拟抛出非受检异常
 */
private int jdbcOperateCheckEx() throws IOException {
   
    //插入的sql
    String sql = "insert into tx_study (name,age) values (?,?)";
    //调用jdbcTemplate的update方法，插入数据
    int insert = jdbcTemplate.update(sql, "insert2", 21);
    //手动抛出一个受检异常
    throw new IOException();
}
```

  **执行之后查看数据库，发现同样没有插入数据，说明事务已经回滚。**   **如果不需要返回值，那么可以使用executeWithoutResult方法：**

```java
@Test
public void tpExecuteWithoutResult() {
   
    //transactionTemplate的executeWithoutResult方法便捷的处理不需要返回值的事务操作
    transactionTemplate.executeWithoutResult(status -> jdbcOperateSelect());
}

/**
 * 业务操作，没有返回值
 */
private void jdbcOperateSelect() {
   
    //查询的sql
    String sql = "select * from tx_study where id = ?";
    TxStudy txStudy = jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), 1);
    System.out.println(txStudy);
}
```

## 3.2 TransactionManager的使用

  **除了使用封装好的TransactionTemplate之外，我们还可以直接使用TransactionManager来管理事务，毕竟它的名字就叫“事务管理器”，肯定提供了各种管理的方法，当然这会更加麻烦！这需要PlatformTransactionManager、TransactionDefinition 和 TransactionStatus这三个对象搭配使用。通过PlatformTransactionManager的getTransaction方法根据TransactionDefinition创建一个事务TransactionStatus事务，并且事务的开启、提交、回滚都需要手动控制，Spring不会帮我们控制！**

  常见案例如下：

```java
@Test
public void tmTest() {
   
    //创建一个事务定义对象，可以设置事务的属性
    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
    //只能以编程方式执行时才能显式设置事务名称
    def.setName("SomeTxName");
    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    //通过getTransaction方法获取一个事务，此时已经开启了事务
    TransactionStatus transaction = transactionManager.getTransaction(def);
    
    //调用业务方法
    try {
   
        jdbcOperateTm();
    } catch (Exception e) {
   
        e.printStackTrace();
        //需要手动回滚事务
        transactionManager.rollback(transaction);
    }
    
    //需要手动提交事务
    transactionManager.commit(transaction);
}

/**
 * 业务操作
 */
private void jdbcOperateTm() {
   
    //插入的sql
    String sql = "insert into tx_study (name,age) values (?,?)";
    //调用jdbcTemplate的update方法，插入数据
    int insert = jdbcTemplate.update(sql, "transactionManager1", 22);
}
```



  **大多数 Spring用户选择声明式事务管理，因为此选项对应用程序代码的影响是最小的，它最符合非侵入式轻量级容器的理念。**   **Spring声明式事务基于Spring AOP，可以说是Spring AOP的最佳实践之一，对于使用了使用AOP声明式配置的bean，将会生成一个AopProxy代理对象，当调用事务方法的时候，实际上是通过代理对象去调用的，因此该方法会被拦截，随后在代理对象的拦截器链中通过TransactionInterceptor拦截器配合TransactionManager来实现对该方法的事务控制。下面是在事务代理上调用方法的概念视图（这里的涉及到Spring AOP的代理源码机制，前面的文章讲过）：** ![img](https://img-blog.csdnimg.cn/20201219222839107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   并且Spring声明式事务的配置被单独提取出来形成了自己的一套配置，因此在刚学习的时候，不必取学习Spring AOP的通用配置，但是如果想要深入学习的话，那么Spring AOP的源码是肯定要看的！   **由于Spring声明式事务基于Spring AOP，因此不需要在业务代码中编写事务控制代码，不影响业务逻辑，这就是非入侵式的好处。当然也有缺点：**

1. **AOP的机制导致了声明式事务只能进行方法级别的事务管理，而编程式事务则可以精确到代码行级别，但即使如此，声明式事务简单、优雅的配置仍是事务控制的最优选择！** 
2. **AOP的机制导致了同一个AOP类中的事务方法互相调用时，被调用方法的事务配置不会生效，因为Spring AOP的代理机制最终还是通过原始目标对象本身去调用目标方法的，这样被调用的方法就会因为是原始对象调用的而不被拦截，当然也有解决办法，和此前同一个类的AOP方法互相调用的解决办法是一样的，那就是获取代理对象，通过代理对象去调用内层方法！** 
3. **无论是基于JDK动态代理还是CGLIB代理，由于本身的缺陷，它们代理的方法的增强都具有限制。对于JDK的代理，目标类必须实现符合规则的接口（不是说只要是实现了接口就会使用JDK代理，具体规则在AOP源码部分有讲解），并且只能代理实现的接口的方法，而对于CGLIB的代理，目标类不能是final的，并且需要代理的方法也不能是private/final/static的。这些AOP代理的限制也是事务增强方法的限制。事务的代理类型也是通过标签 的proxy-target-class属性或者注解的proxyTargetClass属性统一控制的。**

  **声明式事务既可以基于XML，也可以基于注解，或者混合使用二者，它们的底层原理都差不多，只是使用方式不一样！我们将介绍这两种方式！**

## 4.1 基于XML的声明式事务

### 4.1.1 配置依赖

  **基于XML的声明式事务在以前是最流行的声明式事务配置方式！我们将使用Spring 2.x 引入了的 tx事务命名空间，结合使用 aop命名空间，方便快捷的配置事务。由于使用了切入点表达式，因此还需要引入aspectjweaver的依赖，此时项目的总依赖为：**

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

### 4.1.2 配置业务类

  首先，我们模拟一个web项目中的Service，通常我们在service层中配置事务。

```java
/**
 * @author lx
 */
public interface TxStudyService {
   

    TxStudy getTxStudy(Long id);

    List<TxStudy> getTxStudy(String name);

    void insertTxStudy(TxStudy txStudy);

    void updateTxStudy(TxStudy txStudy);
}
```

  这是实现类：

```java
@Service
public class TxStudyServiceImpl implements TxStudyService {
   

    /**
     * jdbc模版，用于操作数据库
     */
    @Resource
    private JdbcTemplate jdbcTemplate;

    /*这里仅仅是为了测试方便，省去了DAO/Mapper层*/

    @Override
    public TxStudy getTxStudy(Long id) {
   
        String sql = "select * from tx_study where id = ?";
        return jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), id);
    }

    @Override
    public List<TxStudy> getTxStudy(String name) {
   
        String sql = "select * from tx_study where name like ?";
        return jdbcTemplate.query(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), "%" + name + "%");
    }

    @Override
    public void insertTxStudy(TxStudy txStudy) {
   
        String sql = "insert into tx_study (name,age) values (?,?)";
        jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge());
    }

    @Override
    public void updateTxStudy(TxStudy txStudy) {
   
        String sql = "UPDATE tx_study SET name=?, age=? where id = ?";
        jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge(), txStudy.getId());
    }
}
```

可以看到，两个查找，一个插入，一个更新，我们假设查找的行为需要使用只读事务，而其他行为则需要读写型普通事务，并且事务执行超时时间为10s。下面我们使用XML统一对这些方法配置事务并且配置事务属性！

### 4.1.3 数据库表

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

### 4.1.4 事务配置文件

  建立一个tx.xml文件，一种常见的事务配置如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.alibaba.com/schema/stat http://www.alibaba.com/schema/stat.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启Spring IoC组件注解包扫描，方便操作-->
    <context:component-scan base-package="com.spring.tx.xml"/>

    <!--druid数据源连接池-->
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url"
                  value="jdbc:mysql://47.94.229.245:3306/test?useSSL=false&amp;allowPublicKeyRetrieval=true&amp;serverTimezone=UTC"/>
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>

    <!--jdbcTemplate 使用Spring JDBC模板类来操作数据库-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--配置一个druidDataSource数据源-->
        <constructor-arg name="dataSource" ref="druidDataSource"/>
    </bean>

    <!--DataSourceTransactionManager事务管理器 用来管理数据库事务-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--配置一个druidDataSource数据源-->
        <constructor-arg name="dataSource" ref="druidDataSource"/>
    </bean>


    <!--配置一个事务的AOP通知，通过transaction-manager属性关联一个事务管理器
    默认查找id/name为transactionManager的事务管理器，因此如果事务管理器beanName就是transactionManager，那么该属性可以省略 -->
    <!--该标签可以配置多个，可以使用不同的事务管理器-->
    <!--此标签可以定义定义许多方法的事务语义（其中事务语义包括传播设置、隔离级别、回滚规则等）-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--一个标签集，内部可以配置多个<tx:method/>标签 -->
        <tx:attributes>
            <!--配置某些方法的事务属性，这些方法是从aop:pointcut切入点表达式匹配到的方法中进一步筛选出来的-->
            <!-- 该标签可配置的属性
            name： 表示匹配的方法名，可以使用通配符*
            read-only：是否是只读事务。默认false，不是。
            isolation：指定事务的隔离级别。默认值为DEFAULT，即对应ISOLATION_DEFAULT
            propagation：指定事务的传播行为。默认值为REQUIRED，即对应PROPAGATION_REQUIRED
            timeout：指定超时时间。默认值为：-1。永不超时。
            rollback-for：指定将触发回滚的一个或多个异常，使用","分隔。如不指定则使用默认策略。
            no-rollback-for：指定不会触发回滚的一个或多个异常，使用","分隔。如不指定则使用默认策略。
            -->
            <!--以"get"开始的所有方法都配置只读属性-->
            <tx:method name="get*" read-only="true" timeout="30"/>
            <!-- 其他剩余方法使用默认事务属性配置-->
            <tx:method name="*" timeout="30"/>
        </tx:attributes>
    </tx:advice>

    <!-- AOP配置，确保上述事务通知正常运行-->
    <aop:config>
        <!--配置aop切入点表达式   该表达式用于确定到底有哪些方法需要进行事务管理-->
        <!--这里，我们配置TxStudyServiceImpl的所有方法均需要被事务管理-->
        <aop:pointcut id="studyServicePt" expression="execution(* com.spring.tx.xml.TxStudyServiceImpl.*(..))"/>
        <!--配置aop通知器 使用advice-ref引用上面定义的事务通知（通过指向它的id） 通过pointcut-ref引入上面定义的aop切入点表达式（通过指向它的id）
        这样，通过pointcut-ref引入的切入点表达式匹配的方法就可以应用于通过advice-ref引入的事务通知机制以及各种属性
        -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="studyServicePt"/>
    </aop:config>

</beans>
```

  除了druidDataSource、jdbcTemplate、transactionManager之外，基于XML的声明式事务就是依靠tx命名空间标签和aop命名空间标签完成的，下面详细解释。

1. 要应用的事务语义封装在 &lt; tx:txAdvice/&gt;事务通知标签中，它实际上就是aop的一个特性化通知标签。该标签的transaction-manager属性关联一个transactionManager事务管理器，默认查找id/name为“transactionManager”的事务管理器，因此如果配置的事务管理器bean的id/name就是transactionManager，那么该属性可以省略。如果要连接的 TransactionManager的bean 具有任何其他名称，则必须显式指定该属性。可以配置多个事务通知标签，使用不同的事务管理器。 
2. &lt; tx:attributes/&gt;标签是一个标签集，内部可以配置多个&lt; tx:method/&gt;标签，因此我们主要看&lt; tx:method/&gt;标签。 
3. &lt; aop:config/&gt;则是Spring AOP的配置标签，包括它的子标签，我们 [在Spring AOP的部分](https://blog.csdn.net/weixin_43767015/category_10402193.html)已经讲过了。事务的代理类型也是通过aop标签的proxy-target-class属性控制的。 
4. &lt; aop:pointcut/&gt;，aop切入点表达式，该表达式用于确定到底有哪些方法需要进行事务管理。使用Aspect表达式，极大的增强了声明式事务的灵活性。 
5. &lt; aop:advisor/&gt;通知器，使用advice-ref引用定义的&lt;tx：txAdvice/&gt;（通过指向它的id） ，通过pointcut-ref引入上面定义的&lt; aop:pointcut/&gt;（通过指向它的id）。这样，通过pointcut-ref引入的切入点表达式匹配的方法就可以应用于通过advice-ref引入的事务通知机制以及各种属性。可以配置多个通知器。 
6. &lt; tx:method/&gt;标签就是基于XML的声明式事务的核心配置标签，该标签用于配置某个或者某些方法的事务属性，这些方法是从aop:pointcut切入点表达式（下面）匹配到的方法中进一步筛选出来的。它具有如下可配置属性：


  **默认回滚策略：执行时如果抛出RuntimeException（运行时异常，非受检异常）以和Error及其它们的子类时，将会回滚。如果抛出其他类型的异常，比如受检异常，那么不会回滚而是提交事务。**   可以同时配置rollback-for和no-rollback-for属性。   甚至在声明式的事务管理配置中，也可以以编程方式指示所需的回滚。虽然简单，但此过程非常具有侵入性。如下操作即可实现：

```java
private void test() throws IOException {
   
    try {
   
        // 业务代码
    } catch (Exception ex) {
   
        // 捕获异常之后，以编程方式手动触发回滚
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

  TransactionAspectSupport.currentTransactionStatus()用于获取当前事务，而setRollbackOnly方法我们在前面就介绍过了，用于设该事务的唯一结果就回滚，用来代替抛出异常。   因此，如果可能，强烈建议通过配置rollback-for和no-rollback-for属性实现声明式的事务回滚。

### 4.1.5 测试

  测试类如下：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:tx.xml")
public class XMLTxText {
   

    @Resource
    private TxStudyService txStudyService;

    @Test
    public void check() {
   
        System.out.println(txStudyService);
        System.out.println(txStudyService.getClass());
    }
}
```

  如何判断某个方法的事务配置是否生效了呢？我们可以使用TransactionAspectSupport.currentTransactionStatus()方法在该方法中获取当前事务，如果开启了事务，那么会返回事务，如果没有事务，那么会抛出异常！   我们改写getTxStudy(Long id)和insertTxStudy(TxStudy txStudy)方法：

```java
@Override
public TxStudy getTxStudy(Long id) {
   
    DefaultTransactionStatus transactionStatus = (DefaultTransactionStatus) TransactionAspectSupport.currentTransactionStatus();
    System.out.println("getTxStudy method Transaction isReadOnly: " + transactionStatus.isReadOnly());
    String sql = "select * from tx_study where id = ?";
    return jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), id);
}

@Override
public void insertTxStudy(TxStudy txStudy) {
   
    DefaultTransactionStatus transactionStatus = (DefaultTransactionStatus) TransactionAspectSupport.currentTransactionStatus();
    System.out.println("insertTxStudy method Transaction isReadOnly: " + transactionStatus.isReadOnly());
    String sql = "insert into tx_study (name,age) values (?,?)";
    jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge());
}
```

  测试：

```java
@Test
public void testTx() {
   
    TxStudy txStudy = txStudyService.getTxStudy((long) 1);
    System.out.println(txStudy);
    txStudyService.insertTxStudy(new TxStudy("insertTxStudy", 23));
}
```

  结果如下：

```java
getTxStudy method Transaction isReadOnly: true
TxStudy{
   createTime=2019-04-21 23:55:15.0, id=1, name='Google', age=12}
insertTxStudy method Transaction isReadOnly: false
```

  可以发现，find方法的事务是只读事务，而其他方法的事务不是，说明设置成功。   下面测试异常是否会回滚！我们改写insertTxStudy(TxStudy txStudy)方法，制造一个**非受检异常**：

```java
@Override
public void insertTxStudy(TxStudy txStudy) {
   
    String sql = "insert into tx_study (name,age) values (?,?)";
    jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge());
    //制造一个非受检异常
    int i = 1 / 0;
}
```

  测试：

```java
@Test
public void testEx() {
   
    txStudyService.insertTxStudy(new TxStudy("insertTxStudyEx", 24));
}
```

  我们发现抛出了异常，查看数据库之后并没有新增的数据，说明事务控制成功，如果我们抛出受检异常呢？我们继续改写insertTxStudy(TxStudy txStudy)方法，制造一个**受检异常**：

```java
@Override
public void insertTxStudy(TxStudy txStudy) throws FileNotFoundException {
   
    String sql = "insert into tx_study (name,age) values (?,?)";
    jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge());
    //制造一个受检异常
    throw new FileNotFoundException();
}
```

  继续刚才的测试，我们发现**同样抛出了异常，查看数据库，发现还是插入了数据，这就是Spring默认的回滚机制，对于除了RuntimeException和Error之外的异常都会提交事务而不是回滚！**   我们配置rollback-for="FileNotFoundException"属性，此时再次测试，发现数据库不会插入数据，这就是事务已经回滚了！如果此时再次抛出一个非受检异常，我们会发现数据库还是没有增加数据，这说明rollback-for是在默认规则上添加新的异常类型，而不是将默认的异常回滚类型覆盖掉！但是如果手动设置了no-rollback-for="RuntimeException"属性，那么即使抛出了非受检异常，那么还是不会回滚！

## 4.2 基于注解的声明式事务

  除了基于XML的事务配置声明方法之外，还可以使用基于注解的方法，并且更加简单。

### 4.2.1 开启注解支持

  在XML中，使用&lt; tx:annotation-driven /&gt;标签开启事务注解的支持。此时我们不必再配置&lt; tx:advice/&gt;和&lt; aop:config/&gt;，可以直接在Spring管理的bean中添加事务注解。&lt;tx：annotation-driven /&gt;的常用属性如下：

1. **transaction-manager**属性，用于指定驱动事务的事务管理器的 beanName。此属性不是必需的，默认值为transactionManager，只有在所需的事务管理器的 beanName不是"transactionManager"时才需要显式指定。 
2. **proxy-target-class**属性，用于指定控制为使用@Transactional注解注解的类创建的事务代理类型。如果proxy-target-class属性设置为true，则创建基于类的代理。如果proxy-target-class为false或者省略了该属性，则会默认创建基于标准JDK接口的代理。 
3. **order**属性，使用@Transactional注解的bean，当多个通知在特定连接点执行时，控制事务通知器的执行顺序。

### 4.2.2 @Transactional注解说明

  默认支持的注解是 Spring 的 @Transactional、JTA 1.2 的 @Transactional（如果可用）和 EJB3 的@TransactionAttribute（如果可用）。最常用的就是Spring提供的@Transactional。并且事务的语义（如传播设置、隔离级别、回滚规则等）都可以在注解元数据（属性）中定义。   @Transactional注解可以标注在类、接口、方法上，标注在类上时，类中的所有方法都尝试应用同一个@Transactional注解，同时方法上的注解的配置优先级高级类上的注解配置。Spring团队建议使用时仅标注在具体类以及具体类的方法，而不是标注在接口或者接口方法上。因为这只有在使用基于接口的代理（JDK动态代理）时它才会生效，如果使用基于类的代理（proxy-target-class =“true”）或基于weaving的aspect（mode=“aspectj”），则代理无效。   如果@Transactional注解同时标注在类上和方法上，方法上的注解优先级最高，如果方法上存在@Transactional注解则直接使用该注解，如果没有，再查找类上的@Transactional注解。   由于基于Spring AOP，因此同一个代理对象的事务方法互相调用，则被调用的方法的事务注解配置不会生效。另外，必须完全初始化代理对象以提供预期的行为，因此对于创建对象时的回调方法进行代理也是无效的，比如@PostConstruct方法的代理无效。   另外，@Transactional注解具有和XML的声明式配置同样的限制，并且，注解的限制更多，在使用时，应仅将@Transactional注解应用于具有public可见性的方法，如果使用@Transactional注解标注protected，private或package级别的方法，虽然则不会引发异常，但是事务配置不会生效（XML中则可以生效）。如果需要非public方法注解生效，那么应该使用使用AspectJ。

  Spring的@Transactional的配置属性如下：


  目前，不能对事务的名称进行显式控制。对于声明性事务，事务名称始终是完全限定的类名称 + 事务通知类的方法名称。   另外，如果具有多个事务管理器，那么可以使用@Transactional注解的value或transactionManager属性指定要使用的某个事务管理器的标识，通常是事务管理器的beanName或者&lt; qualifier/&gt;标识符。在这种情况下，单个方法将在单独配置的事务管理器下运行，如果未配置，那么将默认使用&lt; tx:annotation-driven&gt;中的transaction-manager指定的事务管理器。 @Transactional注解还能标注在注解上，具有@Transactional元注解的注解同样看作一个事务注解。

### 4.2.3 注解配置案例

  我们将基于XML的声明式事务的配置，改成基于注解的配置。   首先的配置文件，我们将原来的service复制到com.spring.tx.ann包下面，新建一个txAnn.xml，基于注解的配置文件如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd   http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启Spring IoC组件注解包扫描，方便操作-->
    <context:component-scan base-package="com.spring.tx.ann"/>

    <!--druid数据源连接池-->
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url"
                  value="jdbc:mysql://47.94.229.245:3306/test?useSSL=false&amp;allowPublicKeyRetrieval=true&amp;serverTimezone=UTC"/>
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>

    <!--jdbcTemplate 使用Spring JDBC模板类来操作数据库-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--配置一个druidDataSource数据源-->
        <constructor-arg name="dataSource" ref="druidDataSource"/>
    </bean>

    <!--DataSourceTransactionManager事务管理器 用来管理数据库事务-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--配置一个druidDataSource数据源-->
        <constructor-arg name="dataSource" ref="druidDataSource"/>
        <qualifier value="11"/>
    </bean>

    <!--启用基于注解的事务行为配置，通过transaction-manager属性关联一个事务管理器 -->
    <!--默认值为transactionManager，因此如果事务管理器beanName就是transactionManager，那么该属性可以省略-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

  随后对TxStudyServiceImpl加入@Transactional注解，使用注解配置：

```java
@Service
@Transactional(timeout = 30)
public class TxStudyServiceImpl implements TxStudyService {
   

    /**
     * jdbc模版，用于操作数据库
     */
    @Resource
    private JdbcTemplate jdbcTemplate;

    /*这里仅仅是为了测试方便，省去了DAO层*/

    @Override
    @Transactional(readOnly = true, timeout = 30)
    public final TxStudy getTxStudy(Long id) {
   
        DefaultTransactionStatus transactionStatus = (DefaultTransactionStatus) TransactionAspectSupport.currentTransactionStatus();
        System.out.println("getTxStudy method Transaction isReadOnly: " + transactionStatus.isReadOnly());
        String sql = "select * from tx_study where id = ?";
        return jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), id);
    }

    @Override
    @Transactional(readOnly = true, timeout = 30)
    public List<TxStudy> getTxStudy(String name) {
   
        String sql = "select * from tx_study where name like ?";
        return jdbcTemplate.query(sql, BeanPropertyRowMapper.newInstance(TxStudy.class), "%" + name + "%");
    }

    @Override
    public void insertTxStudy(TxStudy txStudy) {
   
        String sql = "insert into tx_study (name,age) values (?,?)";
        jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge());
    }

    @Override
    public void updateTxStudy(TxStudy txStudy) {
   
        String sql = "UPDATE tx_study SET name=?, age=? where id = ?";
        jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge(), txStudy.getId());
    }
}
```

  到此，我们的工作已经完成了，可以看出来，使用注解的配置还是非常的简单的。我们来测试一下！   测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:txAnn.xml")
public class AnnTest {
   

    @Resource
    private TxStudyService txStudyService;

    @Test
    public void check() {
   
        System.out.println(txStudyService);
        System.out.println(txStudyService.getClass());
    }
}
```

  我们尝试调用getTxStudy：

```java
@Test
public void getTxStudy() {
   
    System.out.println(txStudyService.getTxStudy((long) 1));
}
```

  结果如下：

```java
getTxStudy method Transaction isReadOnly: true
TxStudy{
   createTime=2019-04-21 23:55:15.0, id=1, name='Google', age=12}
```

  说明事务配置成功，接下来我们在insertTxStudy中抛出一个非受检异常：

```java
@Override
public void insertTxStudy(TxStudy txStudy) throws IOException {
   
    String sql = "insert into tx_study (name,age) values (?,?)";
    jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge());
    //抛出一个RuntimeException
    throw new RuntimeException();
}
```

  测试：

```java
@Test
public void insertTxStudy() {
   
    txStudyService.insertTxStudy(new TxStudy("TransactionalEx", 25));
}
```

  结果抛出了异常，我们去数据库查看，发现并没有插入数据，这说明事务控制成功！接下来我们改为抛出一个FileNotFoundException，即受检异常：

```java
@Override
public void insertTxStudy(TxStudy txStudy) throws FileNotFoundException {
   
    String sql = "insert into tx_study (name,age) values (?,?)";
    jdbcTemplate.update(sql, txStudy.getName(), txStudy.getAge());
    //抛出一个RuntimeException
    //throw new RuntimeException();
    //抛出一个FileNotFoundException
    throw new FileNotFoundException();
}
```

  结果同样抛出了异常，但这次数据库插入了数据，这就是Spring的默认回滚规则，在遇到受检异常时，默认会提交事务而不是回滚！

## 4.3 纯注解的声明式事务

  **在上面，我们已经使用了@Transactional注解来控制事务，但是，对于一些基础性的配置仍然使用到了XML文件，这看起来比较可笑，下面我们来舍弃XML，改为纯注解和Java Config配置。**   新建一个com.spring.tx.pure包，配置类如下：

```java
@ComponentScan
@Configuration
@EnableTransactionManagement
public class PureAnnStart {
   
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

  **上面的Java Config就是去除XML文件之后的配置，其中，对于事务来说关键的配置就是@EnableTransactionManagement注解，该注解用于替代< tx:annotation-driven/>标签来开启事务注解驱动！其他的@ComponentScan、@Configuration则属于IoC的注解，我们在此前已经讲过了！**   接着将我们的service复制一份到com.spring.tx.pure包下，创建一个测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = PureAnnStart.class)
public class PureTest {
   
    @Resource
    private TxStudyService txStudyService;

    @Test
    public void check() {
   
        System.out.println(txStudyService);
        System.out.println(txStudyService.getClass());
    }

    @Test
    public void getTxStudy() {
   
        System.out.println(txStudyService.getTxStudy((long) 1));
    }
}
```

  注意上面的@ContextConfiguration(classes = PureAnnStart.class)，它指向我们的配置类，接下来执行getTxStudy测试一下：

```java
getTxStudy method Transaction isReadOnly: true
TxStudy{
   createTime=2019-04-21 15:55:15.0, id=1, name='Google', age=12}
```

  结果说明改造成功！

## 4.4 事务方法相互调用生效

  **前面学习声明式事务配置的时候就说过，在同一个被Spring管理的类中，如果发生了方法的互相调用，那么被调用的方法的事务设置不会生效，实际上，所有基于AOP的配置（比如普通AOP方法的配置、@Async异步任务方法的配置）都不会生效，这是底层AOP技术的局限性导致的，因为要想AOP配置生效，只有通过代理对象去调用对应的方法才行，而如果存在方法互相调用，那么内层方法是由原始目标对象调用的，那么就不存在任何增强逻辑了！**   解决的办法有许多，比如将两个互相调用的方法分散到不同的类里面，如果不想分散，那么仍然可以比较方便的解决！   我们先复现上面的问题！   一个配置类，开启了事务支持：

```java
@ComponentScan
@Configuration
@EnableTransactionManagement
public class EachCallStart {
   

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

  需要模拟方法互相调用的类：

```java
@Component
public class EachCall {
   
    /**
     * jdbc模版，用于操作数据库
     */
    @Resource
    private JdbcTemplate jdbcTemplate;

    public void m1() {
   
        m2();
    }

    @Transactional
    public void m2() {
   
        String sql = "insert into tx_study (name,age) values (?,?)";
        jdbcTemplate.update(sql, "EachCall", 30);
        //抛出一个RuntimeException
        throw new RuntimeException();
    }
}
```

  **该类中，我们只对m2方法加了事务注解，并且m1方法中调用了m2方法，并且m2方法抛出了RuntimeException非受检异常，Spring事务默认情况下对于非受检异常将会回滚！**   测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = EachCallStart.class)
public class EachCallTest {
   


    @Resource
    private EachCall eachCall;

    @Test
    public void check() {
   
        System.out.println(eachCall);
        //确实是一个代理类对象
        System.out.println(eachCall.getClass());
    }
}
```

  我们尝试调用eachCall的m1()方法，我们知道m1内部会调用m2方法：

```java
@Test
public void test1() {
   
    eachCall.m1();
}
```

  **调用之后，抛出了异常，查看数据库，发现已经插入了数据，对于非受检异常也没有回滚，这说明被被调用的方法的事务配置失效了。下面我们来解决这个问题！**   **最简单的一种方法是，当前类自己注入自己，实际上注入的是一个代理对象，然后我们就可以使用这个注入的代理对象去调用内层方法了！**   我们对EachCall进行如下改造，引入EachCall代理对象，并且通过引入代理对象调用事务方法：

```java
@Component
public class EachCall {
   
    /**
     * 当前类自己注入自己，实际上注入的是一个代理对象，然后我们就可以使用这个注入的代理对象去调用内层方法了
     * 这实际上是一种循环依赖，但是Spring可以帮我们解决这种字段反射注入的循环依赖
     */
    @Resource
    private EachCall eachCall;
    /**
     * jdbc模版，用于操作数据库
     */
    @Resource
    private JdbcTemplate jdbcTemplate;

    public void m1() {
   
        System.out.println(eachCall.getClass());
        //使用注入的eachCall调用m2方法，即可解决问题
        eachCall.m2();
    }

    @Transactional
    public void m2() {
   
        String sql = "insert into tx_study (name,age) values (?,?)";
        jdbcTemplate.update(sql, "EachCall", 30);
        //抛出一个RuntimeException
        throw new RuntimeException();
    }
}
```

  **再次执行测试方法后，查看数据库并没有发现插入了新数据，这说明事务配置生效了，问题得到解决！**   **另一个解决办法就是，对于XML配置，设置< aop:config/>或者< aop:aspectj-autoproxy/>标签的expose-proxy属性为true，对于注解配置，则设置@EnableAspectJAutoProxy的exposeProxy属性为true，其目的就是将代理对象暴露出来，然后代码中使用AopContext.currentProxy()即可获取代理对象，然后强制转型去调用方法即可（注意类型兼容性）。**   **我们这里采用注解，在EachCallStart配置类上加上@EnableAspectJAutoProxy(exposeProxy = true)注解，这个注解实际上是用于支持Aspectj注解，同时还可以用于配置Spring AOP的全局属性，我们在前面的Spring AOP部分也讲过了。这里我们配置exposeProxy属性为true，表示暴露代理对象！**   随后在m1方法中编写代码：

```java
public void m1() {
   
    //System.out.println(eachCall.getClass());
    //使用注入的eachCall调用m2方法即可，解决问题
    //eachCall.m2();

    //获取当前暴露的AOP代理对象
    EachCall o = (EachCall) AopContext.currentProxy();
    //通过代理对象调用方法
    o.m2();
}
```

  **执行测试，发现数据库同样没有插入数据，这说明这种办法也能解决方法互相调用时内层事务不生效的问题！**


  **本次我们学习了Spring 提供的事务管理机制，包括编程式事务和声明式事务，其中编程式事务学习了TransactionTemplate和TransactionManager的方法，而声明式事务则学习的给予XML和基于注解的配置。Spring 事务底层依靠的是Spring AOP机制，是Spring AOP的最佳实践，而Spring AOP底层又是通过的代理模式实现的！**   **Spring提供的事务机制简化了我们对于数据库事务的管理操作，编程式事务提供了更加简便的事务控制API，而声明式事务虽然没有编程式事务灵活（只能提供方法级别的事务管理），但是它不会侵入业务代码，没有耦合性，配置和使用也更加简单，目前声明式事务管理应用得更加广泛。另一个很重要的点是，Spring事务为不同数据库访问框架的事务控制提供了一致的事务编程模型抽象，比如，无论是JdbcTemplate、Mybatis还Hibernate，在与Spring整合之后都可以使用@Transactional来注解控制事务，降低了开发人员的学习和掌握成本！**

**相关文章：**    [https://spring.io/](https://spring.io/)    [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)    [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

