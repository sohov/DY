

>  基于Spring5.x，详细介绍了Spring Cache提供的基于注解的声明式缓存的概念以及使用！比如@Cacheable、@CacheEvict、@CachePut、@Caching、@CacheConfig注解。








  在一个大型互联网项目中，缓存是必不可少的应用，我们会在很多的场景下来使用缓存。缓存有很多种实现方案，常见的有Redis、Ehcache、Memcached等。如果我们使用过这些缓存，那么我们知道它们都提供了不同的jar包，具有不同的使用方式并提供了不同的API，这样就无形中加大了程序员的学习和使用成本。   自Spring Framework 3.1 版本以来，Spring 框架支持以透明方式向现有基于 Spring框架的应用程序添加各种缓存，与此前学习的Spring transaction（事务）的抽象支持类似，缓存抽象允许使用Spring提供的一致地API整合各种缓存解决方案，这样的话对代码的影响最小，减少了程序员的学习和使用成本。   自 Spring Framework 4.1 开始，在 JSR-107 注解和更多自定义选项的支持下，缓存抽象得到了显著扩展，提供了更多的开发方式。

## 1.2 缓存（Cache）和缓冲区（Buffer）

  专业术语的“缓存（Cache）”和“缓冲区（Buffer）”往往可互换使用。但是请注意，它们表示不同的东西。   传统上，缓冲区用作在快速实体和慢速实体之间传递数据的中间临时存储。由于一方必须等待另一方（这会影响性能），缓冲区通过允许整个数据块同时移动而不是以小块方式移动来缓解这种情况。数据仅允许从缓冲区中写入和读取一次。此外，至少有一方是知道缓冲区的存在的。 另一方面，根据定义，缓存是隐藏的，任何一方都不知道缓存的发生。它还提高了性能，但这是通过让相同的数据以一种快速的方式读取多次来实现的。   更多的缓存（Cache）和缓冲区（Buffer）可以看看这篇文章：https://en.wikipedia.org/wiki/Cache_(computing)#The_difference_between_buffer_and_cache

## 1.3 了解Spring缓存抽象（Cache Abstraction）

  Spring的缓存抽象（Cache Abstraction）将缓存操作应用于Java方法。也就是说，每次调用目标方法时，缓存抽象都会应用缓存行为，检查该方法是否已经被调用过（即该操作绑定的缓存是否存在），如果已调用，则直接返回缓存的结果，而无需调用实际方法。如果尚未调用该方法（缓存不存在），则调用该方法，并且将结果缓存起来并返回给用户，以便下次调用该方法时直接返回缓存的结果。以上的逻辑实际上也是所有缓存组件的通用逻辑。   这样，对于输入给定的参数并返回一致的可重用的结果的比较耗时、耗资源的方法（无论是 CPU密集型还是IO 密集型），引用缓存抽象之后，实际上在一定时间内，原本的方法只会被调用一次，提升了效率，降低了资源的消耗。并且缓存抽象的逻辑以透明方式应用到给定的方法，对调用程序没有任何干扰。除了手动改变改变缓存和结果之外，应用缓存抽象的方法仅适用于可以保证为给定输入（或参数）返回相同输出（结果）的方法，无论调用多少次。   缓存抽象还提供了其他与缓存类似的操作，例如更新缓存内容或删除一个或所有缓存的能力。如果在应用程序中需要处理可能更改的数据和结果，则这些操作非常有用。   与Spring Framework 中的其他服务一样，Cache缓存服务不是一个具体的缓存实现方案，并非某一种缓存实现的技术，而是一个辅助缓存使用的通用抽象框架。也就是说，Spring缓存抽象不提供实际缓存数据的存储能力，我们仍然需要在项目中引入实际缓存组件（比如Redis、Ehcache、Memcached的jar包以及准备好对应的缓存服务器）来存储缓存数据，但是缓存抽象可以使你不必编写不同的缓存实现的不同处理逻辑（代码），而是可以使用缓存抽象提供的统一的API和注解，即可操作不同的缓存实现。

## 1.4 Spring缓存抽象API

  基于Spring缓存抽象提供的统一API和注解，让开发者更容易将自己选择的不同的缓存实现高效便捷的嵌入到自己的项目中，就可以轻松实现我们希望达到的缓存效果，不必编写额外的API。   Spring缓存抽象中通过API操作缓存的功能是由 org.springframework.cache.Cache和org.springframework.cache.CacheManager这两个超级接口及其实现类共同实现的。Cache中定义常见缓存操作的接口，它代表缓存实例，而CacheManager作为缓存管理器，用于管理多个Cache实例，以及基于SPI发现、接入多个第三方缓存！   Spring Cache的使用方法和原理都类似于Spring对事务管理的支持。Spring Cache是作用在方法上的，其核心思想和原理仍然是基于Spring AOP的动态代理技术，我们在此前就学了它Spring AOP以及源码，算作Spring AOP的最佳实践之一！   使用Spring Cache需要我们做两方面的事：

1. **缓存配置**：配置缓存存储，主要配置Cache和CacheManager，比如集成Redis就需要配置RedisCacheManager。 
2. **缓存声明**：声明需要缓存的方法和策略，和Spring对事务管理的支持一样，Spring Cache也有基于注解和基于XML配置两种方式。


  **缓存抽象提供了多个存储集成选项，也就是具体的缓存实现。要使用它们，我们需要声明适当的 CacheManager。**   CacheManager是Spring定义的一个用来或者和管理Cache实例的接口。Spring自身已经为我们提供了多种CacheManager的实现，如果我们需要使用其它类型的缓存时，我们可以自己来实现Spring的CacheManager接口或AbstractCacheManager抽象类。   若注解了@EnableCaching，则spring可以基于SPI机制自动发现并配置cacheManager，只要有一种可用于缓存实例提供的即可，常用的有Ehcache、Redis等实现。

## 2.1 基于 JDK 的ConcurrentMap缓存

  **基于 JDK 的缓存实现位于org.springframework.cache.concurrent包下。它允许您使用 ConcurrentHashMap 作为支持缓存的存储仓库。**   下面的示例演示如何配置两个ConcurrentHashMap缓存：

```java
<!--spring自己基于java.util.concurrent.ConcurrentHashMap实现的缓存管理器（该功
能是从Spring3.1开始提供）-->
<bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
    <property name="caches">
        <set>
            <!--注册两个缓存配置-->
            <!-- p:name表示设置当前缓存的name，也就是对应着注解或者XML配置的cacheNames属性-->
            <!-- 此处类concurrentMapCacheFactoryBean作为一个FactoryBean，作用是产生ConcurrentMapCache缓存类实例，可用于测试或简单的缓存方案-->
            <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="default"/>
            <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="books"/>
        </set>
    </property>
</bean>
```

  **SimpleCacheManager是专门针对给定缓存集合处理的一个简单缓存管理器，可用于测试或简单缓存声明。**   **由于ConcurrentMapCache缓存是由应用程序创建的，因此它绑定到其应用程序的生命周期，使其适合基本用例、测试或简单应用程序。缓存扩展良好且速度非常快，但它不提供任何管理、持久性功能或，因此实际开发项目基本不会用到！**   **由于它默认采用ConcurrentHashMap作为缓存仓库，因此不需要引入其他依赖！只需要spring-context依赖：**

```java
<!--spring 核心组件所需依赖-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context-support -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

## 2.2 基于 Ehcache 的缓存

  EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，支持内存和磁盘两种存储。Ehcache默认是使用的是本地的内存做缓存，因此如果存在集群、分布式部署的项目则不适用！   Ehcache 3.x 完全符合 JSR-107 标准，无需专门的支持。Ehcache 2.x 实现位于org.springframework.cache.ehcache包中。**同样，要使用它，您需要声明相应的EhCacheCacheManager缓存管理器。**

```java
<!-- Ehcache 库设置，并且指定ehcache.xml配置文件并加载配置 -->
<!-- ehcache.xml仲可以配置多个缓存 -->
<bean id="ehcache"
      class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean" p:config-location="ehcache.xml"/>
<!-- cacheManager, 指定为EhCacheCacheManager的实现 -->
<bean id="cacheManager"
      class="org.springframework.cache.ehcache.EhCacheCacheManager" p:cache-manager-ref="ehcache"/>
```

  **EhCacheCacheManager位于spring-context-support扩展依赖中，而Spring提供的EhCacheCacheManager实际上还是依赖内部的EhCache的CacheManager完成缓存管理，因此还需要引入EhCache的依赖：**

```java
<!--spring 核心组件所需依赖-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context-support -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
<!--ehcache原始依赖库-->
<!-- https://mvnrepository.com/artifact/net.sf.ehcache/ehcache -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.6</version>
</dependency>
```

  **Ehcache的CacheManager是通过Spring提供的EhCacheManagerFactoryBean来生成的，其可以通过指定ehcache的配置文件位置来生成一个Ehcache的CacheManager。若未指定则将按照Ehcache的默认规则取classpath根路径下的ehcache.xml文件，若该文件也不存在，则获取Ehcache对应jar包中的ehcache-failsafe.xml文件作为配置文件。ehcache.xml中可以配置多个缓存实例对象，在使用时可以选择使用某个缓存。**   因为ehcache使用的不多，后面有机会我们再讲具体如何使用吧！

## 2.3 基于Caffeine的缓存

  Caffeine是 Guava 缓存的 Java 8 重写，相对于guava cache还是有不少的改进，使得性能高于guava cache。地址为：https://github.com/ben-manes/caffeine/wiki。   Caffeine的Spring实现位于 org.springframework.cache.caffeine包中。下面示例如何配置Caffeine缓存管理器：

```java
<bean id="cacheManager"
      class="org.springframework.cache.caffeine.CaffeineCacheManager"/>
```

  **是的就是这么简单！CaffeineCacheManager位于spring-context-support扩展依赖中，而Spring提供的CaffeineCacheManager实际上还是依赖内部的Caffeine的Cache来完成缓存管理，因此还需要引入Caffeine的依赖：**

```java
<!--spring 核心组件所需依赖-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context-support -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
<!--caffeine原始依赖库-->
<!-- https://mvnrepository.com/artifact/com.github.ben-manes.caffeine/caffeine -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.8.8</version>
</dependency>
```

  因为Caffeine使用的不多，后面有机会我们再讲具体如何使用吧！

## 2.4 基于JSR-107规范的缓存

  Spring 的缓存抽象也可以使用符合 JSR-107 的缓存。Jcache的实现位于 org.springframework.cache.jcache包中。   同样，要使用它，您需要声明相应的缓存管理器。下面的示例演示如何这样做：

```java
<bean id="cacheManager"
      class="org.springframework.cache.jcache.JCacheCacheManager"
      p:cache-manager-ref="jCacheManager"/>

<!-- 配置遵守JSR-107 缓存规范的的CacheManager-->
<bean id="jCacheManager" .../>
```

  **JSR-107 是一个缓存规范，因此我们需要自己配置符合JSR-107规范（实现了JSR-107的javax.cache.CacheManager接口）的CacheManager。**   **JCacheCacheManager位于spring-context-support扩展依赖中，而Spring提供的JCacheCacheManager实际上还是依赖内部符合JSR-107规范的CacheManager来完成缓存管理，因此还需要引入JSR-107规范的依赖：**

```java
<!--spring 核心组件所需依赖-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context-support -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
<!--JSR-107 原始依赖库-->
<!-- https://mvnrepository.com/artifact/javax.cache/cache-api -->
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
    <version>1.1.1</version>
</dependency>
```

## 2.5 其他第三方缓存【redis】

  **我们更常用的第三方缓存，比如Redis、Memcached、MongoDB等等，Spring Cache同样支持和它们集成。，要想在Spring项目中使用它们，您同样需要提供CacheManager缓存管理器和Cache缓存具体实现。**   **这些第三方缓存，因为不幸的是，没有可用的标准，Spring并未提供对应的缓存管理器的实现。对于这样的第三方缓存，通常是第三方厂商自己实现了Spring的缓存抽象，提供了自己的CacheManager缓存管理器和Cache缓存具体实现。**   **比如Redis，在专门的Spring-Data-Redis依赖中除了大名鼎鼎的RedisTemplate之外，还提供了RedisCacheManager，该类就实现了Spring的CacheManager接口，因此Reids可以和Spring缓存抽象集成，可以使用注解操作缓存，而不需要使用RedisTemplate编写Java代码！**   **Redis哨兵和Spring Cache集成的简单配置如下，主要就是在最后多配置了一个cacheManager的配置，其他的配置基本没有变化！**

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

    <context:property-placeholder location="redis.properties"/>

    <!--配置Redis池子-->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}"/>
        <property name="maxTotal" value="${redis.maxActive}"/>
        <property name="maxWaitMillis" value="${redis.maxWait}"/>
        <property name="testOnBorrow" value="${redis.testOnBorrow}"/>
    </bean>

    <!--配置哨兵-->
    <bean id="sentinelConfiguration"
          class="org.springframework.data.redis.connection.RedisSentinelConfiguration">
        <property name="master">
            <bean class="org.springframework.data.redis.connection.RedisNode">
                <property name="name" value="${redis.sentinel.master}"/>
            </bean>
        </property>
        <!--配置三个哨兵-->
        <property name="sentinels">
            <set>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.sentinel1.host}"/>
                    <constructor-arg name="port" value="${redis.sentinel1.port}"/>
                </bean>

                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.sentinel2.host}"/>
                    <constructor-arg name="port" value="${redis.sentinel2.port}"/>
                </bean>

                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="${redis.sentinel3.host}"/>
                    <constructor-arg name="port" value="${redis.sentinel3.port}"/>
                </bean>
            </set>
        </property>
    </bean>

    <!--生产Redis连接的工程-->
    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"/>

    <!--Java中操作redis的对象-->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory"/>
        <!--基于注解支持 存入的对象自动转换为json-->
        <property name="valueSerializer">
            <bean class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer"/>
        </property>
    </bean>

    <!--注册一个缓存管理者 可以使用缓存抽象提供的注解操作缓存-->
    <bean name="cacheManager" class="org.springframework.data.redis.cache.RedisCacheManager">
        <!--要在类或方法的注解的value(cacheNames)中使用 表示一个cache的名称-->
        <constructor-arg name="cacheNames" value="defaultCache"/>
        <!--是否缓存null-->
        <constructor-arg name="cacheNullValues" value="false"/>
        <!--注入redisTemplate-->
        <constructor-arg name="redisOperations" ref="redisTemplate"/>


        <!--可选配置 是否使用前缀-->
        <property name="usePrefix" value="true"/>
        <!--可选配置 设置指定key的过期时间（以秒为单位）-->
        <property name="expires">
            <util:map>
                <!-- 指定key的时间为1500秒 -->
                <entry key="xxx" value="1500"/>
                <entry key="yyy" value="1500"/>
            </util:map>
        </property>
    </bean>
</beans>
```


  **在配置好CacheManager和Cache之后，我们就可以使用Spring Cache提供的统一的声明式缓存服务了，和Spring 声明式事务抽象一样，我们可以基于注解和基于XML配置。**   **目前用的更多的是基于注解的声明式缓存，这也是我们主要学习的部分。**

## 3.1 基于Spring注解的声明式缓存

  **对于声明式缓存，Spring 的缓存抽象提供了一组 Java注解！**

1. @Cacheable：将会缓存调用真实方法（或类中的所有方法）所返回的结果，当后续再次调用该方法并且存在对应key的缓存时，将直接返回缓存而不会执行真实的方法！一般用在查询方法上。 
2. @CacheEvict：根据指定的条件将缓存移除。一般用在更新或者删除的方法上。 
3. @CachePut：在不干扰方法执行的情况下更新缓存。也就是说，该注解将会缓存调用方法（或类中的所有方法）返回的结果，但是和@Cacheable不同的是，它每次都会触发真实方法的调用。一般用在新增方法上。 
4. @Caching：组合要应用于方法上的多个缓存操作注解。 
5. @CacheConfig：在类级别共享一些与缓存相关的常见设置。

  以上这些注解都可以被当做元注解，这样就能使实现自定义缓存注解，Spring能够正常解析。

### 3.1.1 启用缓存注解

  **需要注意的是，即使采用了声明式的缓存注解，Spring也不会自动触发它对应的操作和行为，这就像 Spring 中的许多其他操作一样，对于声明式注解的功能必须首先声明性的启用，比如在声明式事务中通过@EnableTransactionManagement注解或者<tx:annotation-driven/>来开启事务注解的支持！**

  需要一个的开关的好处是，如果你怀疑缓存是某些问题的罪魁祸首，则只需删除这一个开启注解支持的配置开关，这样所有的缓存注解都将失效，而不需要将代码中的所有缓存注解都删除！   **若要启用缓存注解，最简单的方法就是将@EnableCaching注解添加到@Configuration配置类上：**

```java
@Configuration
@EnableCaching
public class CacheConfig {
   
}
```

  或者，对于 XML 配置，可以使用<cache:annotation-driven/>标签来开启注解支持：

```java
<beans xmlns=http://www.springframework.org/schema/beans
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:cache="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="
    http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/cache https://www.springframework.org/schema/cache/spring-cache.xsd">

    <cache:annotation-driven/>
</beans>
```

### 3.1.2 测试配置

下面的学习和测试，我们使用ConcurrentMapCache缓存来进行测试，因为它的配置非常简单，没必要引入其他第三方依赖，并且使用Spring test来进行测试，Spring test的依赖为：

```java
<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -
->
<!--Spring 测试-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
<!--单元测试-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

  **我们都使用Java Config的形式配置缓存管理器，注入两个名为cache1和cache2的基于内存的ConcurrentMapCache的缓存实例。**

```java
@Configuration
//开启缓存注解支持
@EnableCaching
@ComponentScan
public class CacheConfig {
   


    /**
     * 配置ConcurrentMapCacheFactoryBean
     * 用于创建名为cache1的ConcurrentMapCache，允许缓存null
     */
    @Bean
    public ConcurrentMapCacheFactoryBean concurrentMapCacheFactoryBean1() {
   
        ConcurrentMapCacheFactoryBean concurrentMapCacheFactoryBean1 = new ConcurrentMapCacheFactoryBean();
        concurrentMapCacheFactoryBean1.setName("cache1");
        concurrentMapCacheFactoryBean1.setAllowNullValues(true);
        return concurrentMapCacheFactoryBean1;
    }

    /**
     * 配置ConcurrentMapCacheFactoryBean
     * 用于创建名为cache2的ConcurrentMapCache，允许缓存null
     */
    @Bean
    public ConcurrentMapCacheFactoryBean concurrentMapCacheFactoryBean2() {
   
        ConcurrentMapCacheFactoryBean concurrentMapCacheFactoryBean2 = new ConcurrentMapCacheFactoryBean();
        concurrentMapCacheFactoryBean2.setName("cache2");
        concurrentMapCacheFactoryBean2.setAllowNullValues(true);
        return concurrentMapCacheFactoryBean2;
    }

    /*
     * 基于Java Config的 CacheManager的配置
     *
     * 这里配置两个名为cache1和cache2的基于内存的ConcurrentMapCache的缓存实例，仅用于测试
     */

    /**
     * @param caches Spring自动注入全部Cache实现
     */
    @Bean
    public CacheManager cacheManager(Collection<ConcurrentMapCache> caches) {
   
        SimpleCacheManager simpleCacheManager = new SimpleCacheManager();
        simpleCacheManager.setCaches(caches);
        return simpleCacheManager;
    }
    
}
```

### 3.1.3 @Cacheable注解

  **使用@Cacheable注解来标定可缓存的方法（标注在类上就表示类中的所有方法），方法的返回值将被存储在缓存中，以便在随后的调用（使用相同的参数）时直接返回缓存中的值，而无需实际调用该方法。**   **在最简单的使用中，只需要指定与注解方法关联的一个或者多个缓存实例的名称即可，即指定@Cacheable注解的value或者cacheNames属性，这两个属性具有一致的效果，都是传递一个String[]。**

#### 3.1.3.1 简单测试

  我们在cacheable1方法中指定一个@Cacheable注解，需要访问的缓存实例名为cache1：

```java
@Component
public class CacheableTest {
   

    @Cacheable("cache1")
    public double cacheable1(String name) {
   
        System.out.println("----执行方法----");
        //随机返回一个double
        return ThreadLocalRandom.current().nextDouble();
    }
}
```

  测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = CacheConfig.class)
public class CacheTest {
   

    @Resource
    private CacheableTest cacheableTest;



    @Test
    public void cacheable1() {
   
        /*连续调用三次*/
        double xx1 = cacheableTest.cacheable1("xx");
        double xx2 = cacheableTest.cacheable1("xx");
        double xx3 = cacheableTest.cacheable1("xx");
        System.out.println(xx1);
        System.out.println(xx2);
        System.out.println(xx3);
    }
}
```


  这个cacheable1测试方法将会连续调用三次cacheableTest.cacheable1方法，按照正常情况，将会返回不同的结果，如果加上缓存，那么将会返回同一个结果，我们运行测试，结果如下： ![img](https://img-blog.csdnimg.cn/20210308110217608.png#pic_center)   可以发现，缓存生效了，只执行了一次方法，后两次方法都是从缓存实例中获取的被缓存的数据，并没有执行真正的方法，测试成功，下我们进一步深入学习！

#### 3.1.3.2 属性详解

  @Cacheable注解有很多可配置的属性！


##### 3.1.3.2.1 sync属性

  缓存过期之后，如果多个线程同时请求对某个数据的访问，会同时去数据库拿去数据，导致数据库瞬间负荷增高，也就是缓存穿透。并发量比较高的情况下，容易导致数据库挂掉。   Spring 4.3 的@Cacheable注解新增了一个名为sync的boolean类型属性，该属性默认为false，当置为true时，如果有多个线程尝试获取同一key的值，则同步该基础方法的调用，只有一个线程的请求会去到数据库，其他线程都会等待直到缓存可用，可以防止缓存过期时造成的瞬间缓存穿透。注意，这里的同步仅仅是在当前JVM中的同步，如果是集群部署，那么仍然可能造成多个请求到达数据库！   开启sync同步会导致几个限制：

1. unless属性不被支持。 
2. 只能指定一个缓存实例。 
3. 无法合并其他与缓存相关的操作。 
4. 不一定所有的缓存实现都支持这个配置，需要实际验证才行！我们使用的ConcurrentMapCache由于不支持过期时间，因此这个参数无效，Redis则支持该参数。

#### 3.1.3.3 key的生成

##### 3.1.3.3.1 默认key生成

  **由于缓存本质上是键值（key-cvalue）形式的存储，因此每次调用缓存方法都需要转换为适合缓存访问的key。如果没有手动指定key，那么Spring缓存抽象使用基于以下算法的简单key生成器：**

1. 如果方法没有参数，则返回SimpleKey.EMPTY对象作为key，这个对象是单例的！ 
2. 如果只给出一个参数，则返回该参数对象作为key。 
3. 如果给定多个参数，则返回包含所有参数的 SimpleKey对象作为key

  **默认策略下，不同的方法但是如果传递相同的参数，那么可能会造成缓存key的冲突，如果此时方法的返回值不一样，那么可能会因此从缓存获取并转换类型而造成ClassCastException。**   三个缓存方法：

```java
@Cacheable("cache1")
public double defaultkey() {
   
    //随机返回一个double
    return ThreadLocalRandom.current().nextDouble();
}

@Cacheable("cache1")
public double defaultkey(String param1) {
   
    //随机返回一个double
    return ThreadLocalRandom.current().nextDouble();
}

@Cacheable("cache1")
public double defaultkey(String param1, String param2) {
   
    //随机返回一个double
    return ThreadLocalRandom.current().nextDouble();
}
```

  测试：

```java
/**
 * 获取名为cache1的ConcurrentMapCache，注意属性名一定是concurrentMapCacheFactoryBean1
 * 虽然该属性名指向创建它的那个FactoryBean，但是实际上会返回getObject方法的结果，也就是对应的ConcurrentMapCache
 */
@Resource
private ConcurrentMapCache concurrentMapCacheFactoryBean1;

@Test
public void defaultKey() {
   
    System.out.println(cacheableTest.defaultkey());
    System.out.println(cacheableTest.defaultkey());
    System.out.println(cacheableTest.defaultkey("xx"));
    System.out.println(cacheableTest.defaultkey("yy"));
    System.out.println(cacheableTest.defaultkey("11", "22"));
    System.out.println(cacheableTest.defaultkey("22", "11"));
    ConcurrentMap<Object, Object> nativeCache = concurrentMapCacheFactoryBean1.getNativeCache();
    System.out.println(nativeCache);
}
```


  在测试方法中的最后一行进行debug即可看到存储的key： ![img](https://img-blog.csdnimg.cn/20210308110634287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   **只要参数作为业务的key并实现了有效的 hashCode()和 equals()方法（因为SimpleKey对象通过对比参数的hashCode()和 equals()以及它们的顺序来生成自己的HashCode，进而判断是否是对同一个key的请求），此方法就适用于大多数用例。如果不是这样，那么需要更改策略，或者指定key。**

##### 3.1.3.3.2 自定义key生成规则

  对于一个多参数的方法来说，可能某些参数没有必要作为缓存的key，或者某些参数需需要进行某些操作之后才能成为key。   对于此类情况，@Cacheable注释允许指定如何通过key属性生成真正的key。我们可以使用 SpEL 表达式选取感兴趣的参数（或参数的嵌套属性）、执行其他操作，甚至调用任意方法，而无需编写任何代码或实现任何接口。SPEL表达式中我们可以直接使用"#参数名"或者"#p参数索引"或者"#a参数索引"来选取对应的参数，对于嵌套属性使用“.属性名”获取，实际上还可以获取其他信息，我们后面会讲！ 虽然默认策略可能适用于某些方法，但它很少适用于所有方法，而SPEL表达式则非常的强大，因此推荐使用，关于SPEL表达式，我们在此前就详细的介绍了它的语法！   以下示例使用部分基于 SpEL 声明的key：

```java
/**
 * 选取参数
 * 使用第一个参数的值作为key
 */
@Cacheable(value = "cache1", key = "#p0")
public double spelKey1(int param1, int param2) {
   
    return ThreadLocalRandom.current().nextDouble();
}


/**
 * 执行计算
 * 使用第一个参数和第二个参数的和作为key
 */
@Cacheable(value = "cache1", key = "#p0+#p1")
public double spelKey2(int param1, int param2) {
   
    return ThreadLocalRandom.current().nextDouble();
}


/**
 * 调用方法
 * 使用第一个参数的长度作为key
 */
@Cacheable(value = "cache1", key = "#p0.length()")
public double spelKey3(String param1) {
   
    return ThreadLocalRandom.current().nextDouble();
}

/**
 * 调用T类型的静态方法
 * 使用第一个参数的长度作为key
 */
@Cacheable(value = "cache1", key = "T(Math).random()*#param1")
public double spelKey4(String param1) {
   
    return ThreadLocalRandom.current().nextDouble();
}


/**
 * 集合导航
 * 使用参数集合的第一元素作为key
 */
@Cacheable(value = "cache1", key = "#stringList[0]")
public double spelKey4(List<String> stringList) {
   
    return ThreadLocalRandom.current().nextDouble();
}

//还有非常多的操作可以使用SPEL来实现
```

  上面这些代码向我们展示了基于SPEL来选取想要的key是多么的容易。   **如果负责生成key的算法过于复杂或者需要共享，可以定义自定义key生成器。我们需要实现org.springframework.cache.interceptor.KeyGenerator接口。**   有了自定义KeyGenerator之后，需要在keyGenerator属性上指定要使用的KeyGenerator的bean 实现的名称，如下例所示：

```java
/**
 * 指定key生成器
 */
@Cacheable(value = "cache1", keyGenerator = "myKeyGenerator")
public double myKeyGenerator() {
   
    return ThreadLocalRandom.current().nextDouble();
}
```

  **注意，@Cacheable的key和keyGenerator参数是互斥的，同时指定两者的操作会导致异常。**

#### 3.1.3.4 缓存解析器

  **缓存抽象默认使用SimpleCacheResolver缓存解析器。若要提供不同的默认缓存解析器，需要实现 org.springframe.cache.interceptor.CacheResolver 接口。**   **默认缓存解析器非常适合使用单个 CacheManager 且没有复杂缓存解析要求的应用程序。对于使用多个缓存管理器的应用程序，可以设置缓存管理器以用于每个操作，如下例所示：**

```java
@Cacheable(cacheNames="cache1", cacheManager="anotherCacheManager")
public double anotherCacheManager() {
   
    return ThreadLocalRandom.current().nextDouble();
}
```

  当然还可以完全为类似于替换key生成的方式替换CacheResolver。因为CacheResolver内部就关联了一个CacheManager，它们都是差不多的：

```java
@Cacheable(cacheNames="cache1", cacheResolver="anotherCacheResolver")
public double anotherCacheResolver() {
   
    return ThreadLocalRandom.current().nextDouble();
}
```

#### 3.1.3.5 条件缓存

#### 3.1.3.5.1 condition条件

  **有时，方法可能不适合一直使用缓存（例如，它可能取决于给定的参数）。缓存注解通过condition参数支持此类用例，condition参数采用计算结果为true或false的 SPEL 表达式。如果为 true，则缓存该方法的结果。否则，它的行为就像方法未缓存一样（也就是说，无论缓存中包含哪些值或使用什么参数，每次都会调用该方法）。**   例如，只有当参数名称的长度小于 32 时，才缓存以下方法的结果：

```java
@Cacheable(cacheNames = "cache1", condition = "#param.length()<32")
public double condition(String param) {
   
    return ThreadLocalRandom.current().nextDouble();
}
```

##### 3.1.3.5.2 unless条件

  除了condition参数之外，还可以使用unless参数。unless参数同样需要使用SPEL表达式，不过与condition不同的是，unless是在调用方法后计算表达式（如果不会调用方法本身，则不会执行unless计算），因此unless不会影响方法的执行，并且可以引用方法的返回结果来进行计算，并且只有计算结果不为true是才会缓存。   如下案例，如果返回值不为null，那么才进行缓存！这个resulr表示引用的结果！

```java
@Cacheable(cacheNames = "cache1", unless = "#result==null")
public double unless(String param) {
   
    return ThreadLocalRandom.current().nextDouble();
}
```

##### 3.1.3.5.3 优先级和执行顺序

  **condition的优先级高于unless，condition 不指定相当于 true，unless 不指定相当于 false：**

1. 当 condition = false，一定不会缓存； 
2. 当 condition = true，且 unless = true，不缓存； 
3. 当 condition = true，且 unless = false，缓存；

  **执行顺序为：如果condition计算为false，那么无论有没有对应的key的缓存，都会执行原本的方法，只是不会进行缓存操作而已！如果condition为true，那么会接着查找对应key的缓存，如果有就直接返回缓存的数据，如果没有才会执行方法，而只有执行了方法，而才会触发unless的计算，如果unless计算为true，那么不会进行缓存操作，如果为false，那么将会进行缓存操作！**

#### 3.1.3.6 可用的缓存 SpEL 计算上下文

  **此前我们讲过，每个 SpEL 表达式可以根据专用上下文（EvaluationContext）进行计算。对于缓存抽象的SPEL表达式来，说除了传统的内置参数（比如systemEnvironment、systemProperties、environment）之外，框架还提供专用的缓存相关元数据，如参数名称等属性，我们可以在SPEL中直接对它们进行引用，很方便的用于key和条件计算：**


  **缓存抽象中的SPEL同样可以引用其他被Spring管理的bean，只需要“@beanName”即可引用！**   **缓存抽象的SPEL同样支持Safe Navigation运算符，因为我们引用的某些属性可能为null，此时如果在对其进行方法或者属性的调用，那么会抛出NullPointerException。使用Safe Navigation操作符（也就是在导航的.之前加上一个?即可，非常简单）之后，如果引用的对象为null，那么将返回null而不是抛出异常。**   **总之，缓存抽象的SPEL几乎支持全部的SPEL语法和特性，这些语法和特性我们在此前就专门讲过了，再次不再赘述！**

#### 3.1.3.7 null缓存支持


  **一般情况下，null不会缓存，这样如果攻击者制造了数据库不存在的假数据的话，就能使请求每一次都落到数据库中，给数据库带来很大压力，一个常见的做法是，将“null”也缓存起来，这样后续有同样的请求时，将直接返回缓存中的null，而不会去数据库查找。**   在此前，我们需要自己设置一个对象，使它和null对等，当数据库返回null时，就将该对象作为value存入缓存，当从缓存取出来时，如果值等于该对象，那么就表示缓存的null，随后直接返回null。   **在Spring Cache中，某些CacheManager支持通过配置属性的方式来设置是否支持null值的存储，比如ConcurrentMapCacheFactoryBean的allowNullValues 属性，比如RedisCacheManager的cacheNullValues构造器属性（高版本可能有变动）！**   我们将最开始配置的cache1的allowNullValues改为false： ![img](https://img-blog.csdnimg.cn/20210308113545688.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   缓存方法如下，每次都返回null：

```java
@Cacheable(cacheNames = "cache1", key = "T(String).valueOf('nullable')")
public Object nullable() {
   
    System.out.println("nullable");
    return null;
}
```

  测试方法：

```java
@Test
public void nullable() {
   
    ConcurrentMap<Object, Object> nativeCache = concurrentMapCacheFactoryBean1.getNativeCache();
    cacheableTest.nullable();
    cacheableTest.nullable();
    cacheableTest.nullable();
    cacheableTest.nullable();
}
```




  执行之后，将直接抛出异常： ![img](https://img-blog.csdnimg.cn/20210308113626441.png#pic_center)   我们将allowNullValues设置为true（或者取消设置，因为ConcurrentMapCacheFactoryBean的该属性默认为true），再次debug执行： ![img](https://img-blog.csdnimg.cn/2021030811364417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   可以看到，Spring帮我们缓存了一个NullValue对象来代表null值。这个NullValue是Spring提供的，因此无序我们手动配置一个表示null的对象！ ![img](https://img-blog.csdnimg.cn/20210308113703300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

#### 3.1.3.8 标注在类上

  **@Cacheable注解可以标注在类上，此时就表示类中的所有方法应用@Cacheable注解。但是注意，如果某些方法上已经标注了@Cacheable注解，那么这两个注解的属性不会合并，而是方法上的@Cacheable注解的使用优先级大于类上的@Cacheable注解！**   **实际上的原理是，首先在方法上查找全部缓存注解（Cacheable、CacheEvict、CachePut、Caching），如果存在至少任何一个以上任何类型的注解，那么就不会查找类上的缓存注解，也就是说此时类上的缓存注解对于该方法就无效了，如果没有以上任何一个缓存注解，那么才会继续查找类上的全部缓存注解并应用！（源码位于AbstractFallbackCacheOperationSource. computeCacheOperations方法。）**

### 3.1.4 @CachePut注解

  **对于使用@Cacheable标注的方法，Spring在每次执行前都会检查Cache中是否存在相同key的缓存元素，如果存在就不再执行该方法，而是直接从缓存中获取结果进行返回，否则才会执行并将返回结果存入指定的缓存中。@CachePut也可以声明一个方法支持缓存功能（或者类上的所有方法），@CachePut也可以标注在类上和方法上。使用@CachePut时我们可以指定的属性跟@Cacheable是一样的。**   **与@Cacheable不同的是，使用@CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都一定会执行真实的方法，并将结果以键值对的形式存入指定的缓存中。**   当需要更新缓存而不干扰真正方法的执行时，可以使用@CachePut注解。   通常强烈建议不要对同一方法同时使用@CachePut和@Cacheable注解，因为它们有不同的行为。后者使用缓存导致跳过真实的方法调用，而前者将强制调用真实的方法以运行缓存更新。这会导致意外行为，并且，除了特定的角情况（如具有相互排除它们的条件注解）之外，应避免这种声明。另请注意，此类条件不应依赖于结果对象（即#result变量），因为这些条件是经过前面验证以确认排除的。

#### 3.1.5.1 属性详解

  **@CacheEvict注解有很多可配置的属性！其中value(cacheNames)、key和condition的语义与@Cacheable对应的属性类似。即value表示清除操作是发生在哪些Cache上的（对应Cache的名称）；key表示需要清除的是哪个key，如未指定则会使用默认策略生成的key；condition表示清除操作发生的条件。**


#### 3.1.5.2 allEntries

  使用 allEntries 属性从指定名字的缓存实例中删除所有缓存。默认值为false，表示不删除。   当需要清除整个缓存区域时，allEntries会派上用场。所有缓存将在一个操作中删除，而不是删除每一次内存（这需要很长时间，因为它效率低下）。请注意，如果设置了allEntries=true，那么框架将忽略此方案中指定的任何key，因为它不适用（整个缓存被删除，而不是仅一个key对应的缓存）。   缓存类：

```java
@Component
public class CacheEvictTest {
   

    /**
     * 缓存的方法，存放到cache1中
     */
    @Cacheable(value = "cache1")
    public double cacheable(String name) {
   
        return ThreadLocalRandom.current().nextDouble();
    }

    /**
     * 清除的方法，清除cache1中的全部缓存
     */
    @CacheEvict(value = "cache1", allEntries = true)
    public void cacheEvict() {
   
    }
}
```

  测试方法：

```java
@Resource
private CacheEvictTest cacheEvictTest;

@Test
public void cacheEvictTest() {
   
    ConcurrentMap<Object, Object> nativeCache = concurrentMapCacheFactoryBean1.getNativeCache();
    cacheEvictTest.cacheable("cacheable1");
    cacheEvictTest.cacheable("cacheable2");
    cacheEvictTest.cacheEvict();
}
```



  我们debug测试，会发现cache1中的缓存在执行cacheable之后被添加，而在执行cacheEvict之后被全部删除： ![img](https://img-blog.csdnimg.cn/20210308114253546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) ![img](https://img-blog.csdnimg.cn/20210308114308482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

#### 3.1.5.3 beforeInvocation

  **指示是否在方法执行前就删除缓存。如果指定为 true，则在方法还没有执行的时候就删除缓存。默认值为 false，将在方法执行之后删除缓存，如果方法没有执行（因为可能直接从缓存中取数据）或者执行抛出异常，则不会删除缓存。**

### 3.1.6 @Caching注解

  有时，需要指定同一类型的多个注解，例如，在一个方法调用之后，我们可能会添加、或者删除多个不同key，甚至这些key为不同的缓存实例中，这时就需要在一个方法上指定多个对应不同缓存的@CacheEvict 或者@CachePut注解。   @Caching注解允许在同一方法上使用多个相同的@Cacheable、@CachePut、@CacheEvict注解。   下面的示例使用两个@CacheEvict注解：

```java
/**
 * 清除两个key的方法
 */
@Caching(evict = {
   @CacheEvict(cacheNames = "cache1", key = "'cache1'"), @CacheEvict(cacheNames = "cache1", key = "'cache2'")})
public void twoCacheEvict() {
   
}
```

  测试方法：

```java
@Test
public void twoCacheEvict() {
   
    ConcurrentMap<Object, Object> nativeCache1 = concurrentMapCacheFactoryBean1.getNativeCache();
    cacheEvictTest.cacheable("cache1");
    cacheEvictTest.cacheable("cache2");
    cacheEvictTest.cacheable("cache3");
    cacheEvictTest.twoCacheEvict();
}
```



  debug查看结果，在执行完全部cacheable之后，缓存实例中有三个缓存： ![img](https://img-blog.csdnimg.cn/2021030811442684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   当执行完twoCacheEvict之后，名为cache1和cache2的缓存已被删除： ![img](https://img-blog.csdnimg.cn/2021030811444498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

### 3.1.7 @CacheConfig注解

  **到目前为止，我们看到缓存操作提供了许多自定义配置，我们可以为每个操作设置这些配置。但是，某些适用于类的所有方法的通用配置可能非常繁琐，比如为一个类的所有方法能使用相同的缓存实例，在此前我们必须为每一个注解都配置value或者cacheNames属性。此时就是@CacheConfig发挥作用的地方。**   **@CacheConfig是一个类级别的注解，可以在类级别上配置该类的所有方法的缓存操作的通用属性，比如cacheNames、keyGenerator、cacheManager、cacheResolver。如果单独在类上使用这个配置注解但并没有其他实际缓存注解，那么不会有任何缓存操作。**   同样，如果方法自己声明同名属性（无论是来自方法还是类），那么就不会应用@CacheConfig的配置。

## 3.2 JCache (JSR-107)注解

  **自Spring 4.1版本以来，Spring 的缓存抽象完全支持 Jcache的标准注解：@CacheResul、@CachePut、@CacheRemove、@CacheRemoveAll、@CacheDefaults、@CacheKey、@CacheValue。**   即使没有将底层的缓存存储迁移到 JSR-107标准的缓存实现上，我们也可以使用这些注解。因为Spring仅仅是借用了这些注解的定义，内部的解析实现还是使用Spring 的缓存抽象，并提供符合规范的默认CacheResolver和KeyGenerator实现。换句话说，如果你已经在使用 Spring 的缓存抽象，则无需更改底层的缓存存储配置即可直接使用JSR-107的标准缓存注解。这一点就类似于Spring AOP，Spring AOP也可以使用来自于Aspect的切面注解，但是底层注解的解析仍然是Spring自己的逻辑。   下表描述了 Spring 的注解和 JSR-107 对应项之间的主要区别：


  **JCache 具有 javax.cache.annotation.cacheResolver 的概念，该概念与 Spring 的缓存解析器接口相同，但 JCache 仅支持单个缓存。默认情况下，简单实现会根据注解上声明的名称检索要使用的缓存。应该注意的是，如果在注解上未指定缓存名称，则会自动生成默认值。**   **通常情况下，建议还是使用Spring提供的注解而不是JSR-107的注解，因此这部分的知识了解即可！**

## 3.3 基于XML的声明式缓存

  我们也可以使用 XML 配置声明式缓存。我们可以指定目标方法（target method）和缓存指令（caching directives），类似于声明性事务管理的advice的配置。   缓存的XML标签，统一使用“cache”命名空间其他的配置和声明式事务的XML配置差不多，都需要使用到aop的相关标签以及AspectJ切入点表达式，因此基于XML的声明式缓存处理前面所说的的依赖之外还需要引入aspectj的依赖：

```java
<!--用于解析AspectJ的切入点表达式语法-->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
</dependency>
```

  下面是一个简单的配置案例！

```java
/**
 * @author lx
 * 要应用缓存的类
 */
public class CacheXml {
   
    /**
     * 将会插入缓存
     */
    public double cacheable(String key) {
   
        System.out.println(key);
        return ThreadLocalRandom.current().nextDouble();
    }

    /**
     * 将会删除缓存
     */
    public void cacheEvict() {
   
    }
}
```

  spring-cache.xml的配置：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:cache="http://www.springframework.org/schema/cache"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--spring自己基于java.util.concurrent.ConcurrentHashMap实现的缓存管理器（该功能是从Spring3.1开始提供）-->
    <bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
        <property name="caches">
            <set>
                <!--注册两个缓存配置-->
                <!-- p:name表示设置当前缓存的name，也就是对应着注解或者XML配置的cacheNames属性-->
                <!-- 此处类concurrentMapCacheFactoryBean作为一个FactoryBean，作用是产生ConcurrentMapCache缓存类实例，可用于测试或简单的缓存方案-->
                <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="cache1"/>
                <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="cache2"/>
            </set>
        </property>
    </bean>

    <!--配置Bean-->
    <bean id="cacheXml" class="com.spring.cache.xml.CacheXml"/>


    <!-- 缓存通知的定义 -->
    <!-- cache-manager指向一个CacheManager的beanName，默认值就是cacheManager -->
    <cache:advice id="cacheAdvice" cache-manager="cacheManager">
        <!--定义cache规则-->
        <!--cache表示使用的Cache实例的name,可以指定多个，使用,分隔-->
        <cache:caching cache="cache1">
            <!--通过不同的标签定义不同的缓存操作   method 表示该操作要应用的方法名，可以使用*通配-->
            <!--其他的属性和注解的属性都差不多，比如key表示key的生成策略，all-entries对应着注解的allEntries-->
            <cache:cacheable method="cache*" key="#key"/>
            <cache:cache-evict method="cacheEvict" all-entries="true"/>
        </cache:caching>
    </cache:advice>

    <!--Spring AOP的配置  -->
    <aop:config>
        <!--配置通知器 -->
        <!--advice-ref指向一个通知的id  这里将缓存通知应用于AOP-->
        <!--pointcut是切入点表达式，用于筛选可能要被应用于通知的方法，这里的切入点表达式的含义就是 将缓存行为应用于所有CacheXml类的方法-->
        <aop:advisor advice-ref="cacheAdvice" pointcut="execution(* *..*.CacheXml.*(..))"/>
    </aop:config>

</beans>
```

  测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-cache.xml")
public class CacheXmlTest {
   
    @Resource
    private CacheXml cacheXml;

    @Resource
    private SimpleCacheManager simpleCacheManager;

    @Test
    public void test(){
   
        ConcurrentMapCache cache1 = (ConcurrentMapCache) simpleCacheManager.getCache("cache1");
        ConcurrentMap<Object, Object> nativeCache = cache1.getNativeCache();
        //缓存测试
        cacheXml.cacheable("11");
        cacheXml.cacheable("11");
        cacheXml.cacheable("11");
        cacheXml.cacheable("22");
        //缓存清理
        cacheXml.cacheEvict();
    }
}
```



  我们debug测试，即可发现基于XML的缓存配置已经生效了： ![img](https://img-blog.csdnimg.cn/20210308134343714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) ![img](https://img-blog.csdnimg.cn/20210308134355847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   基于XML的声明式缓存支持所有基于注解的配置，因此在XML和注解这两者之间切换应该相当容易。此外，两者都可以在同一个的应用程序中同时使用。   基于 XML 的配置方式触及目标代码。然而，从配置上就能看出来，它本质上更冗长，配置也更加复杂。特别是在处理具有用于缓存行为的重载方法的类时，确定正确的用于缓存方法确实需要额外的进行筛选，因为方法参数不是一个好的筛选器。在这些情况下，我们可以使用 AspectJ 切入点来挑选目标方法并应用适当的缓存功能。   同样，由于XML的配置支持AspectJ 切入点表达式，因此相比于基于注解的配置可以更轻松的对某一个范围（特定的包、特定的类、特定的方法）应用缓存，特别是如果你的方法有一个明确的命名规范，那么通过XML的AspectJ可以很方便的进行缓存管理！   目前而言，使用了Spring Cahce的项目基本上都是基于注解来使用的，XML的配置方式非常的少见，因此这话部分知识了解即可！


## 4.1 Spring缓存抽象的优点

1. 提供了基于注解和XML的声明式缓存配置，可以方便的使得既有代码支持缓存，同时可以解耦业务中的缓存，以前实现缓存的方式，是在业务代码中嵌入了有大量缓存操作的代码，耦合度太高，看着很不优雅。 
2. 基于Spring缓存抽象提供的统一配置和注解，让开发者更容易将自己选择的不同的第三方缓存服务高效便捷的集成到自己的项目中，就可以轻松实现我们希望达到的缓存效果，不必编写额外的API。此前不同的缓存实现也有不同的操作API，也是件麻烦的事。 
3. 实际上，Spring Cache还提供了该抽象的一些开箱即用的默认实现，可以直接拿来使用，即不用安装和部署额外第三方组件就可以使用默认的缓存实现，默认的缓存实现是基于ConcurrentMap实现的ConcurrentMapCache，同时提供了EHCache的实现EhCacheCache以及GemFire、Caffeine、JSR-107 compliant caches的实现，这些缓存实现都是默认基于本地内存的，只需要引入缓存本身的依赖，不必再安装和部署额外第三方组件（比如Redis就需要安装Redis服务器）就可以使用缓存。 
4. 支持Spring Expression Language（SPEL）来引用对象的任意属性或者方法并定义缓存的key和condition，因此具备相当大的灵活性，可以支持非常复杂的语义。 
5. 支持自己定义 key 和自己定义CacheManager，具有相当的灵活性和扩展性。如果SpEL达不到你的预期，可以实现自己的KeyGenerator。

## 4.2 Spring缓存抽象的缺点

1. 默认不支持TTL（Time To Live），也就是缓存过期时间；也不支持TTI（Time To Idle）空闲期，即一个数据多久没被访问将从缓存中移除的时间；还不支持移除策略（Eviction policy），即即如果缓存满了，从缓存中移除数据的策略。 
 <ol> 
  1. 诸如上面属性或者其他具体缓存实现的特有属性应该直接通过Cache的具体实现来设置，并且只有Cache实现自己去完成，Spring Cache仅仅是一个抽象，仅提供了比较基本的缓存行为，或许有一些方案可以设置过期时间但是只能设置一个统一的过期时间，而不能精确到每一个key，这明显不够灵活。 
  1. 上面的这个缺点就导致了Spring Cache的实际应用并不是很广泛，就拿Redis来说，或许大部分人都还是在使用的RedisTemplate来操作Reids缓存，因此它可以设置过期时间、以及选择缓存的数据结构、以及获取缓存的方式，这明显比Spring Cache的几个注解要灵活得多！ 
 </ol>  
4. 无法根据查询结果中的内容生成缓存key，比如getUser(uid)方法，想通过查询出来的user.email生成缓存key就无法实现了，因为#result不能应用在key上面！ 
5. 由于声明式的Spring Cache是基于Spring AOP实现的，因此默认的AOP实现的缺点在Spring Cache上同样不可避免： 
 <ol> 
  5. 默认的AOP的机制导致了声明式缓存只能进行方法级别的缓存管理，也就是说只能在方法执行之前从缓存尝试取数据，方法执行完毕之后才能将结果添加到缓存中！ 
  5. AOP的机制导致了同一个AOP类中的事务方法互相调用时，被调用方法的事务配置不会生效，因为Spring AOP的代理机制最终还是通过原始目标对象本身去调用目标方法的，这样被调用的方法就会因为是原始对象调用的而不被拦截，当然也有解决办法，和此前同一个类的AOP方法互相调用的解决办法是一样的，那就是获取代理对象，通过代理对象去调用内层方法！ 
  5. 无论是基于JDK动态代理还是CGLIB代理，由于本身的缺陷，它们代理的方法的增强都具有限制。对于JDK的代理，目标类必须实现符合规则的接口（不是说只要是实现了接口就会使用JDK代理，具体规则在AOP源码部分有讲解），并且只能代理实现的接口的方法，而对于CGLIB的代理，目标类不能是final的，并且需要代理的方法也不能是private/final/static的。这些AOP代理的限制也是缓存增强方法的限制。而缓存的代理类型也是通过标签的proxy-target-class属性或者注解的proxyTargetClass属性统一控制的。 
 </ol> 

## 4.3 总结

  Spring Cache是一个缓存抽象，将大多数缓存的基本功能抽取出来，然后定义一个统一的使用方式（配置和注解），只要是基于Spring的项目，都可以将第三方缓存与Spring Cache快速的结合起来。这样的好处就是我们只需要掌握Spring Cache的使用就能够相当于掌握了大多数第三方缓存的基本操作，降低了学习成本。   当前，Spring Cache仅仅抽取了缓存的基本操作行为，对于具体的某些缓存的高级行为，Spring Cache是无能为力的，比如Redis，Spring Cache仅能比较方便的存取普通的key和value，而对于特殊的缓存结构，比如hash、set、zset等，以及它们的特殊操作，比如lpush、rpop等等，此时Spring Cache的能力就显得捉襟见肘了。   为此，如果是项目没有使用的特别复杂的缓存操作和缓存结构，那么使用Spring Cache是非常方便（还要考虑过期时间的设置），如果项目的缓存操作比较复杂，那么建议还是建议使用专门的缓存操作API吧，比如RedisTemplate。

**相关文章：**    [https://spring.io/](https://spring.io/)    [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)    [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

