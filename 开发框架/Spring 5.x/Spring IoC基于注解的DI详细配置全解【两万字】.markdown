

>详细介绍了基于注解和Java Config的核心IoC机制的配置和使用。

本次我们介绍基于注解和Java Config的Spring IoC机制的核心配置方式，包括bean的配置、依赖项注入配置，以及各种常用配置信息。本文基于最新Spring 5.x。

关于Spring IoC的XML配置： [IoC入门以及基于XML的IoC配置全解](https://blog.csdn.net/weixin_43767015/article/details/108380043)。







Spring基于注解的配置提供了基于XML配置的替代方案，它依赖注解（基于字节码元数据和反射）获取信息并描述bean，而不是使用XML标签来声明bean。开发人员不需要写XML文件，而是使用注解标注在相关类、方法或字段声明上。很明显，注解将配置信息从XML文件中移动到组件类本身。

**基于注解的配置的引入提出了一个问题，即这种方法是否比XML配置"更好"。** 简短的回答是"视情况而定"。更好的回答是，每种方法都有其优缺点，通常由开发人员决定哪种策略更适合他们。由于它们的定义方式，注释在声明中提供了大量上下文，从而导致配置更短、更简洁。但是，XML擅长在不接触其源代码或重新编译组件的情况下配置组件信息。比如对于外部jar包中的类，此时不能在源码上加入注解，那就只能使用XML配置。一些开发人员喜欢将配置靠近源代码，而另一些开发人员则认为带注解的类不再是纯粹的POJO。此外，注解配置的一个缺点是配置信息变得分散且难以控制。

我们可以同时使用注解和XML配置，注解注入在XML注入之前执行。因此，当两者同时对相同的某些属性使用时，XML配置会覆盖注解注入的配置。

随着Spring版本的不断更新，对于注解的支持也在不断进步，一些最早的注解被抛弃了，另一些更强大的注解被推荐使用，到目前（Spring 5.x），已经提供了大量的注解方便开发者使用。本文将讲解其中用的最常用的一些最基本的IoC容器的注解，对于其他注解，我们会在后续相关的文章中一一介绍。

例如，在Spring 2.0的时候，曾提供了@Required注解，用于标注bean的属性对应的setter方法，它表示受影响的属性必须在配置时必须在XML中配置注入或者自动注入。如果未注入受影响的bean属性，则容器将抛出BeanInitializationException异常。然而从Spring5.1开始，@Required注解正式被弃用，因为Spring建议将必须的依赖通过构造器注入或使用初始化回调方式注入。Spring2.5还增加了对JSR-250注解的支持，如@PostConstruct和@PreDstroy。Spring 3则为javax.inject包中包含的JSR-330（Java依赖注入）注解添加了支持，例如@Inject和@Named。

**学习基于注解的IoC配置之前，我强烈建议学习Java注解的基本原理：深入理解Java中的注解的概念、用法以及案例演示。同时本文会有和基于XML配置的对照，IoC入门以及基于XML的IoC配置全解。**


开启注解支持的快捷方式是在原始的XML配置文件中加入context Schema约束。

## 2.1 添加context Schema

原始的Spring XML配置文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
</beans>
```

加入context Schema约束之后的配置文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context 
       https://www.springframework.org/schema/context/spring-context.xsd">
    
</beans>
```

## 2.2 开启注解支持

随后我们需要开启注解支持配置，简单的说，开启检测、处理某些注解的功能。仅仅对于Spring Ioc核心注解支持，这里有两个配置可选：&lt; context:annotation-config /&gt;和&lt; context:component-scan base-package="" /&gt;，它们有什么区别呢？

### 2.2.1 annotation-config

&lt; context:annotation-config/ &gt;用于对**已注册在容器中bean**的内部进行注解检测、激活，对于bean的注册注解则不检测，因此可能还需要使用XML或者其他方式将bean交给IoC容器之后，该annotation-config才会对容器中的bean的相关内部注解进行激活控制。

这个配置实际上是帮助我们尝试向容器注册AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor这四个BeanPostProcessor的bean，不同的BeanPostProcessor用于检测、处理各自对应的的注解服务，相当于注解处理器。常见&lt; context:annotation-config/ &gt;可激活的注解如下：

1. Spring的@Required、@Autowired、@Lookup、@value； 
2. JSR-330的@Inject（如果可用） 
3. JSR 250的@Resource，@PostConstruct和@PreDestroy（如果有的话）； 
4. JAX-WS 的 @WebServiceRef（如果可用）； 
5. EJB 3 的 @EJB （如果可用）； 
6. JPA的@PersistenceContext和@PersistenceUnit（如果可用）

注意：不会激活Spring的@Transactional注解；可以使用&lt;tx：annotation-driven /&gt;标签来激活。同样，Spring的缓存相关的注解也需要明确启用。

```java
<context:annotation-config/>
<!--annotation-config用于替代下面几个bean的注入-->
<bean id="autowiredAnnotationBeanPostProcessor"  class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor "/>
<bean id="commonAnnotationBeanPostProcessor"  class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor"/>
<bean id="requiredAnnotationBeanPostProcessor"  class="org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor"/>
<!--spring-context依赖默认不支持jpa-->
<bean id="persistenceAnnotationBeanPostProcessor"  class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>
```

### 2.2.2 component-scan

&lt; context:component-scan base-package=""/ &gt;**具有context:annotation-config的全部功能**。另外，必须要指定一个base-package属性，表示将会自动扫描的指定的包路径下面的所有类，并将带有注册注解的bean注册到IoC容器中。默认情况下，将检测Spring @Component、@Repository、@Service、@Controller、@RestController、@ControllerAdvice 和@Configuration等注解。

通常，我们不会在XML中注册bean，而在代码中使用注解注入依赖，因此我们只需要使用&lt; context:component-scan base-package=""/ &gt;就行了。可以配置多个component-scan 标签，当然也可以在base-package属性中配置扫描多个包路径，可以使用“ ”、“,”、“;”等符号分隔，将会扫描该路径下的所有子包的类。

另外，可以使用通配符 * 。一个 * 就表示其中一级路径，两个 ** 就表示其中多级路径或者没有路径。比如base-package="com. * .core. * "，表示扫描com.xx.core包路径的core包下面的所有的子包下面的类。base-package="com. ** .core. ** "，表示扫描具有com和core包路径的core包下面的类和core包的所有的子包下面的类，等价于base-package=“com.**.core”。超过两个 * 就和一个 * 的含义一样！


另外还有其他的属性，后面再介绍！ ![img](https://img-blog.csdnimg.cn/20200908104329204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)


## 3.1 @Component组件

**@Component注解可以标注在类上。IoC容器启动时，会自动在component-scan配置的包范围之中扫描被@Component注解标注的类，将它们注册到IoC容器中，我们称这些类为“组件”，就是相当于XML的< bean >标签，即定义了一个bean，并且交给容器。**

通常，任何需要交给Spring管理的组件（bean）都可以使用@Component注解标注。自Spring2.5起，在web分层项目架构中，为了区分某个Spring管理的组件属于哪一层架构，因此根据@Component注解衍生出来了三个注解：@Repository、@Service和@Controller。它们默认都是单例bean。

@Controller用于标注表示层的组件，@Service用于标注服务层的组件，@Repository用于标注持久层的组件。当组件不好归类的时候，才建议使用@Component。

目前来看这几个注解没有太多实际意义的差别，但是未来的Spring版本将可能对这三个衍生组件进行增强，比如目前@Repository已经被支持作为 [持久性层中自动异常转换的标记](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#orm-exception-translation)，以及@Controller被用于查找Spring MVC的Handler。

@Component、@Repository、@Service和@Controller将会使用类似XML的构造函数的方式实例化bean。实际上基于注解的配置相比于XML的配置更加灵活，也有许多不同的地方，我们后面会讲到！

### 3.1.1 IoC注解配置第一例

新建一个空的maven工程，引入如下依赖：

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
        <scope>compile</scope>
    </dependency>
</dependencies>
```

添加spring-config.xml配置文件，开启注解支持：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.spring.core.*"/>
</beans>
```


新建ComponentOne类，用于测试@Component注解 ![img](https://img-blog.csdnimg.cn/20200908104850692.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

```java
/**
 * @author lx
 */
@Component("componentOne")
public class ComponentOne {
   

    public ComponentOne() {
   
        System.out.println("ComponentOne：" + this);
    }


    @Component("componentTwo")
    public class ComponentTwo {
   
        public ComponentTwo() {
   
            System.out.println("ComponentTwo：" + this);
        }
    }

    @Component("componentThree")
    public static class ComponentThree {
   
        public ComponentThree() {
   
            System.out.println("ComponentThree：" + this);
        }
    }
}
```

Annotation类用于测试：

```java
@Test
public void anno() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("componentOne", ComponentOne.class));
    System.out.println(ac.getBean("componentOne", ComponentOne.class));
    System.out.println(ac.getBean("componentTwo", ComponentOne.ComponentTwo.class));
    System.out.println(ac.getBean("componentTwo", ComponentOne.ComponentTwo.class));
    System.out.println(ac.getBean("componentThree", ComponentOne.ComponentThree.class));
    System.out.println(ac.getBean("componentThree", ComponentOne.ComponentThree.class));
}
```

结果如下，可以看到，成功从容器获取了bean实例，并且默认都是单例的bean实例：

```java
ComponentThree：com.spring.core.ann.ComponentOne$ComponentThree@145eaa29
ComponentOne：com.spring.core.ann.ComponentOne@275710fc
ComponentTwo：com.spring.core.ann.ComponentOne$ComponentTwo@17c386de
com.spring.core.ann.ComponentOne@275710fc
com.spring.core.ann.ComponentOne@275710fc
com.spring.core.ann.ComponentOne$ComponentTwo@17c386de
com.spring.core.ann.ComponentOne$ComponentTwo@17c386de
com.spring.core.ann.ComponentOne$ComponentThree@145eaa29
com.spring.core.ann.ComponentOne$ComponentThree@145eaa29
```

### 3.1.2 bean的命名

在扫描到一个被托管的组件时，它的对应的bean名称由BeanNameGenerator策略和@Component、@Repository、@Service、@Controller注解指定的value值一起生成。

**如果注解设置了value值，那么value值作为bean的名字，外部类bean name 不能重复；普通内部类之间bean name 可以重复，启动时不会异常，但是不建议重复，可能造成bean覆盖，在运行时抛出异常；静态内部类之间bean name 不能重复。因此，指定的bean name都不应该重复。**

如果没有指定value值和BeanNameGenerator，默认bean name生成的策略为：

1. 对于外部类，默认bean名称生成器将返回小写的非限定类名。但是如果类名有多个字符并且前两个字符都是大写时，将直接返回简单类名。这些规则遵循Java命名空间类java.beans.introspector的decapitalize（shortClassName）静态方法定义的规则。 
2. 对于普通内部类，默认bean名称生成器将返回全限定外部类名$内部类名。 
3. 对于静态内部类，默认bean名称生成器将返回小写的非限定外部类名.内部类名。静态内部类之间bean name 不能重复。

案例如下，一个ComponentName类，仅仅使用@Component注解，不设置value值，不设置BeanNameGenerator，用于测试默认命名。

```java
/**
 * @author lx
 */
@Component
public class ComponentName {
   

    public ComponentName() {
   
        System.out.println("ComponentName：" + this);
    }


    @Component
    public class ComponentNameTwo {
   
        public ComponentNameTwo() {
   
            System.out.println("ComponentNameTwo：" + this);
        }
    }

    @Component
    public static class ComponentNameThree {
   
        public ComponentNameThree() {
   
            System.out.println("ComponentNameThree：" + this);
        }
    }
}
@Component
public class COOmponentName {
   
    public COOmponentName() {
   
        System.out.println("COOmponentName：" + this);
    }
}


@Component
public class COOmponentName {
   
    public COOmponentName() {
   
        System.out.println("COOmponentName：" + this);
    }
}
```

测试：

```java
@Test
public void annoName() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println("--------------------");
    //普通外部类，默认bean名称生成器将返回小写的非限定类名。
    System.out.println(ac.getBean("componentName", ComponentName.class));
    //普通外部类，如果类名有多个字符并且前两个字符都是大写时，将直接返回简单类名。
    System.out.println(ac.getBean("COOmponentName", COOmponentName.class));
    //普通内部类，默认bean名称生成器将返回全限定外部类名$内部类名。
    System.out.println(ac.getBean("com.spring.core.ann.ComponentName$ComponentNameTwo", ComponentName.ComponentNameTwo.class));
    //静态内部类，默认bean名称生成器将返回小写的非限定外部类名.内部类名。
    System.out.println(ac.getBean("componentName.ComponentNameThree", ComponentName.ComponentNameThree.class));
}
```

结果如下，成功获取到了bean实例：

```java
COOmponentName：com.spring.core.ann.COOmponentName@400cff1a
ComponentNameThree：com.spring.core.ann.ComponentName$ComponentNameThree@4c6e276e
ComponentName：com.spring.core.ann.ComponentName@534df152
ComponentNameTwo：com.spring.core.ann.ComponentName$ComponentNameTwo@3ee0fea4
--------------------
com.spring.core.ann.ComponentName@534df152
com.spring.core.ann.COOmponentName@400cff1a
com.spring.core.ann.ComponentName$ComponentNameTwo@3ee0fea4
com.spring.core.ann.ComponentName$ComponentNameThree@4c6e276e
```

#### 3.1.2.1 自定义bean name生成器

如果不想依赖注解设置的value和默认的bean命名策略，那么我们可以提供一个自定义的bean name生成器。

首先，需要实现BeanNameGenerator接口，并确保该实现包含一个默认的无参数构造器。然后，在配置组件扫描器时，在name-generator属性上设置该生成器的完全限定的类名。

```java
/**
 * bean名称生成器
 *
 * @author lx
 */
public class MyNameGenerator implements BeanNameGenerator {
   

    /**
     * bean的命名接口
     *
     * @param definition 用于生成名称的bean的定义
     * @param registry   bean定义注册中心
     * @return 自定义的bean name
     */
    @Override
    public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
   
        //简单的直接返回bean的类名，仅供测试
        return definition.getBeanClassName();
    }
}
```

配置文件，加入name-generator属性：

```java
<context:component-scan base-package="com.spring.core.*" name-generator="com.spring.core.ann.MyNameGenerator"/>
```

再一次测试，bean name全部填写全限定类名：

```java
@Test
public void annoGenerateName() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");

    //全部都要求全限定类名，则可以获取该bean的实例

    System.out.println(ac.getBean("com.spring.core.ann.ComponentName", ComponentName.class));
    System.out.println(ac.getBean("com.spring.core.ann.ComponentName", ComponentName.class));

    System.out.println(ac.getBean("com.spring.core.ann.COOmponentName", COOmponentName.class));
    System.out.println(ac.getBean("com.spring.core.ann.COOmponentName", COOmponentName.class));

    System.out.println(ac.getBean("com.spring.core.ann.ComponentName$ComponentNameTwo", ComponentName.ComponentNameTwo.class));
    System.out.println(ac.getBean("com.spring.core.ann.ComponentName$ComponentNameTwo", ComponentName.ComponentNameTwo.class));

    System.out.println(ac.getBean("com.spring.core.ann.ComponentName$ComponentNameThree", ComponentName.ComponentNameThree.class));
    System.out.println(ac.getBean("com.spring.core.ann.ComponentName$ComponentNameThree", ComponentName.ComponentNameThree.class));
}
```

结果如下，成功获取到bean实例：

```java
COOmponentName：com.spring.core.ann.COOmponentName@57d5872c
ComponentNameThree：com.spring.core.ann.ComponentName$ComponentNameThree@3bbc39f8
ComponentName：com.spring.core.ann.ComponentName@4ae3c1cd
ComponentThree：com.spring.core.ann.ComponentOne$ComponentThree@29f69090
ComponentOne：com.spring.core.ann.ComponentOne@568bf312
ComponentNameTwo：com.spring.core.ann.ComponentName$ComponentNameTwo@4034c28c
ComponentTwo：com.spring.core.ann.ComponentOne$ComponentTwo@14ec4505
com.spring.core.ann.ComponentName@4ae3c1cd
com.spring.core.ann.ComponentName@4ae3c1cd
com.spring.core.ann.COOmponentName@57d5872c
com.spring.core.ann.COOmponentName@57d5872c
com.spring.core.ann.ComponentName$ComponentNameTwo@4034c28c
com.spring.core.ann.ComponentName$ComponentNameTwo@4034c28c
com.spring.core.ann.ComponentName$ComponentNameThree@3bbc39f8
com.spring.core.ann.ComponentName$ComponentNameThree@3bbc39f8
```

**如果此时运行最开始的测试案例，则会抛出异常！一般情况下，我们要么通过value指定bean名值，要么使用默认命名规则，不建议自定义bean name生成器！**

## 3.2 @Bean工厂

在Spring的一系列Component组件注解或者@Configuration配置注解标注的类的内部，可以使用@Bean注解标注方法，表示向IoC容器添加bean定义。

@Bean的作用同样类似于&lt; bean &gt;标签，和@Component系列注解的区别是相当于使用工厂模式实例化bean。即我们在方法中编写实例化bean的逻辑，然后返回该bean，这样bean就会被注册到IoC容器中，IoC容器会自动调用该方法创建bean实例。可以类比基于XML的工厂方法实例化bean那部分。

对于使用@Bean注册的bean，它的默认命名策略就是取方法名作为bean的名字。

@Bean一般用于注册外部jar包中的类，因为这些类不能的源码不能够改，不能添加注解，此时只有使用@Bean的方式了。实际上@Bean注解更多的使用在@Configuration配置注解类内部，@Bean里面也有很多配置可以设置，后面会讲到。

案例如下，一个BeanFactoryDemo类，该类本身被@Component注解标注，有一个被@Bean标注的方法：

```java
/**
 * @author lx
 */
@Component
public class BeanFactoryDemo {
   


//    /**
//     * 由于方法名为beanFactoryDemo，和@Component的命名策略冲突导致名字重复，进而抛出异常
//     */
//    @Bean
//    public BeanFactoryDemo beanFactoryDemo() {
   
//        return new BeanFactoryDemo();
//    }

    /**
     * 使用@Bean的方式注册bean
     */
    @Bean
    public BeanFactoryDemo getBeanFactoryDemo() {
   
        return new BeanFactoryDemo();
    }
}
```

测试：

```java
@Test
public void beanFactory() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //这是@Component注册的bean
    System.out.println(ac.getBean("beanFactoryDemo", BeanFactoryDemo.class));
    //这是@Bean注册的bean
    System.out.println(ac.getBean("getBeanFactoryDemo", BeanFactoryDemo.class));
}
```

结果如下，成功实例化：

```java
com.spring.core.ann.BeanFactoryDemo@a4102b8
com.spring.core.ann.BeanFactoryDemo@11dc3715
```

## 3.3 @Scope组件作用域

基于注解的组件的默认作用域都是singleton单例，如果想要指定不同的作用域，可以使用@Scope注解，可以在注解中提供作用域的名称。

@Scope注解仅仅在一系列Component组件注解标注的类和@Bean标注的工厂方法上有效果，scope含有的的作用域和XML配置的作用域参数一致，就是相当于&lt; bean &gt;标签的scope属性，详情参考XML的配置文章。

通常，我们只会配置singleton和prototype两种作用域，甚至基本不会配置，默认就是singleton。如果要配置，这两个值在 ConfigurableBeanFactory 接口中有SCOPE_SINGLETON 和 SCOPE_PROTOTYPE定义好的字面常量值，我们可以直接使用，当然也可以直接写字符串。

案例如下，新建一个ScopeDemo类，用于测试@Scope注解：

```java
/**
 * @author lx
 */
@Component
public class ScopeDemo {
   
    public ScopeDemo() {
   
        System.out.println("ScopeDemo初始化：" + this);
    }

    /**
     * 默认就是singleton
     */
    @Component("scopeO")
    public static class ScopeO {
   
        public ScopeO() {
   
            System.out.println("ScopeO初始化：" + this);
        }
    }

    /**
     * 加上注解不指定作用域也是默认singleton
     */
    @Component("scopeA")
    @Scope
    public static class ScopeA {
   
        public ScopeA() {
   
            System.out.println("ScopeA初始化：" + this);
        }
    }

    /**
     * 可以通过ConfigurableBeanFactory.SCOPE_SINGLETON引用singleton字符串
     */
    @Component("scopeB")
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    public static class ScopeB {
   
        public ScopeB() {
   
            System.out.println("ScopeB初始化：" + this);
        }
    }

    /**
     * 可以通过ConfigurableBeanFactory.SCOPE_PROTOTYPE引用prototype字符串
     */
    @Component("scopeC")
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    @Qualifier
    public static class ScopeC {
   
        public ScopeC() {
   
            System.out.println("ScopeC初始化：" + this);
        }
    }


    /**
     * 默认@Bean就是singleton
     * 加上注解不指定作用域也是默认singleton
     */
    @Bean
    @Scope
    public ScopeDemo getScopeDemo1() {
   
        return new ScopeDemo();
    }

    /**
     * 指定prototype
     */
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public ScopeDemo getScopeDemo2() {
   
        return new ScopeDemo();
    }
}
```

测试：

```java
@Test
public void scope() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println("----@Component和@Bean singleton 的bean，在创建容器时就会初始化----");
    System.out.println(ac.getBean("scopeO", ScopeDemo.ScopeO.class));
    System.out.println(ac.getBean("scopeO", ScopeDemo.ScopeO.class));
    System.out.println(ac.getBean("scopeA", ScopeDemo.ScopeA.class));
    System.out.println(ac.getBean("scopeA", ScopeDemo.ScopeA.class));
    System.out.println(ac.getBean("scopeB", ScopeDemo.ScopeB.class));
    System.out.println(ac.getBean("scopeB", ScopeDemo.ScopeB.class));
    System.out.println(ac.getBean("getScopeDemo1", ScopeDemo.class));
    System.out.println(ac.getBean("getScopeDemo1", ScopeDemo.class));
    System.out.println("----@Component和@Bean prototype的bean，在获取时才会初始化----");
    System.out.println(ac.getBean("scopeC", ScopeDemo.ScopeC.class));
    System.out.println(ac.getBean("scopeC", ScopeDemo.ScopeC.class));
    System.out.println(ac.getBean("getScopeDemo2", ScopeDemo.class));
    System.out.println(ac.getBean("getScopeDemo2", ScopeDemo.class));
}
```

结果如下，符合预期：

```java
ScopeA初始化：com.spring.core.ann.ScopeDemo$ScopeA@6ee12bac
ScopeB初始化：com.spring.core.ann.ScopeDemo$ScopeB@ca263c2
ScopeO初始化：com.spring.core.ann.ScopeDemo$ScopeO@589b3632
com.spring.core.ann.ScopeDemo$ScopeO@589b3632
com.spring.core.ann.ScopeDemo$ScopeO@589b3632
com.spring.core.ann.ScopeDemo$ScopeA@6ee12bac
com.spring.core.ann.ScopeDemo$ScopeA@6ee12bac
com.spring.core.ann.ScopeDemo$ScopeB@ca263c2
com.spring.core.ann.ScopeDemo$ScopeB@ca263c2
ScopeC初始化：com.spring.core.ann.ScopeDemo$ScopeC@21507a04
com.spring.core.ann.ScopeDemo$ScopeC@21507a04
ScopeC初始化：com.spring.core.ann.ScopeDemo$ScopeC@143640d5
com.spring.core.ann.ScopeDemo$ScopeC@143640d5
```

当然，我们可以自定义scope注解解析器，实际上Spring的注解都有对应的注解解析器。自定义解析策略，只需要实现ScopeMetadataResolver接口即可，类似于上面的自定义bean name生成器，都是不建议使用的，实际上注解默认解析器就是AnnotationScopeMetadataResolver，是一个ScopeMetadataResolver接口的实现，有兴趣可以去看看。

### 3.3.1 生命周期管理

**在此前基于XML的IoC配置文章中，我们说过，生命周期更长的bean中注入生命周期更短的bean时，可以使用查找方法注入< lookup-method/>来解决短生命周期的bean实例的获取的问题，使得短生命周期的bean能够被正常管理。**

**这里，@Scope注解也能解决这个问题。主要的就是设置它的proxyMode属性，使得注入的是一个代理对象，生命周期更长的bean获取生命周期更短的bean实例时，实际上是通过代理对象去获取具有正常生命周期的bean实例的。**

**如果我们的短生命周期bean实现了接口时，那么proxyMode可以选择设置为ScopedProxyMode.INTERFACES，这表示将使用JDK动态代理来创建代理对象，代理对象同样实现这个接口；如果我们的短生命周期bean没有实现接口并且不是final类型时，那么proxyMode可以选择设置为ScopedProxyMode. TARGET_CLASS，这表示将使用CGLIB动态代理来创建代理对象，代理对象继承目标类。**

如下案例，一个com.spring.aop.scopedproxy.Prototype类，短生命周期的bean，它仅仅被设置为原型：

```java
/**
 * @author lx
 */
@Component
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Prototype  {
   
    public Prototype() {
   
        System.out.println("create Prototype");
    }
}
```

一个长生命周期的com.spring.aop.scopedproxy.Singleton类，内部注入了一个property的bean：

```java
/**
 * @author lx
 */
@Component
public class Singleton {
   
    @Resource
    private Prototype prototype;

    public Prototype getPrototype() {
   
        return prototype;
    }
}
```

配置文件，spring-scopedProxy.xml，添加如下配置：

```java
<context:component-scan base-package="com.spring.aop.scopedproxy"/>
```

测试： @Test

```java
public void scopedProxy() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-scopedProxy.xml");
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
com.spring.aop.scopedproxy.Singleton@7ff95560
com.spring.aop.scopedproxy.Prototype@add0edd
com.spring.aop.scopedproxy.Prototype@add0edd
-----------------
true
com.spring.aop.scopedproxy.Prototype@add0edd
com.spring.aop.scopedproxy.Prototype@add0edd
```

可以看到，虽然我们注入的bean是原型的，看是看起来并没有被成功控制，因为每次都是同一个原型bean对象，我们在@Scope注解中加上proxyMode = ScopedProxyMode.TARGET_CLASS属性之后，再次测试：

```java
com.spring.aop.scopedproxy.Singleton@7cc0cdad
create Prototype
com.spring.aop.scopedproxy.Prototype@5656be13
create Prototype
com.spring.aop.scopedproxy.Prototype@4218d6a3
-----------------
true
create Prototype
com.spring.aop.scopedproxy.Prototype@76505305
create Prototype
com.spring.aop.scopedproxy.Prototype@14cd1699
```

**可以看到，每次获取原型对象都是获取的新建的对象，成功的控制了对象的生命周期为“原型”，这就是@Scope注解解决长生命周期的bean调用短生命周期的bean的问题，主要是靠proxyMode属性，相比于查找方法注入< lookup-method/>标签来说，是不是变得很简单了？**

## 3.4 @Autowired依赖自动注入

前面我们讲了一些bean注册的注解，有@Repository、@Service、@Controller、@Bean，对于类或者方法使用这些注解表示将bean交给IoC容器管理，然后我们就可以从IoC容器中获取对应bean名称的实例对象了。我们同样可以使用注解实现属性注入。

@Autowired注解实际上就是Spring提供的一个依赖自动注入的注解，在此前基于XML的配置文章中，我们就曾经了解过自动注入，XML的自动注入都是通过构造器或者setter方法，并且还有各种各样的限制，@Autowired注解则使用更加灵活方便，功能也更加强大！同样，@Autowired要求其所在的类被注册到IoC容器中。JSR-330的@Inject注解可以一定程度上代替Spring的@Autowired注解。

@Autowired可以用在构造器、属性、方法上，甚至，某些时候不使用该注解就能实现自动注入。@Autowired注解是使用AutowiredAnnotationBeanPostProcessor处理器来解析的，后面的文章会分析BeanPostProcessor的原理，现在先学习如何使用。

**@Autowired的自动注入规则如下：**

1. 首先在容器中查询要注入参数、属性的对应类型的bean（类型可以向下兼容匹配）； 
2. 如果只查询到一个，就将该bean实例注入进去到对应类型的参数、属性中，没查询到则抛出异常； 
3. 如果查询的结果不止一个，那么继续根据属性、参数名称作为bean name来查找； 
4. 如果查询到使用该参数作为name的bean，就将该bean实例注入进去到对应类型的参数、属性中。那么如果没查询到，那么会抛出异常。

## 3.4.1 标注在构造器上

@Autowired标注在构造器上时，表示对该构造器的所有参数从IoC中查找匹配的对象注入进去，查找的类型就是参数类型，查找的name就是参数名。

如下案例，一个AutowiredConstructorDemo，用于测试标注在构造器上：

```java
/**
 * @author lx
 */
@Component
public class AutowiredConstructorDemo {
   

    private DemoA demoA;

    private DemoB demoB;

    /**
     * 要求在IoC中查找并注入两个依赖属性
     */
    @Autowired
    public AutowiredConstructorDemo(DemoA demoA, DemoA demoB) {
   
        this.demoA = demoA;
        this.demoB = (DemoB) demoB;
    }

    @Component("demoA")
    public static class DemoA {
   
        public DemoA() {
   
            System.out.println("DemoA："+this);
        }
    }

    @Component("demoB")
    public static class DemoB extends DemoA {
   
        public DemoB() {
   
            System.out.println("DemoB："+this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredConstructorDemo{" +
                "demoA=" + demoA +
                ", demoB=" + demoB +
                '}';
    }
}
```

测试：

```java
@Test
public void autowired() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredConstructorDemo", AutowiredConstructorDemo.class));
}
```

结果如下，成功注入：

```java
DemoA：com.spring.core.ann.AutowiredConstructorDemo$DemoA@2357d90a
DemoA：com.spring.core.ann.AutowiredConstructorDemo$DemoB@64c87930
DemoB：com.spring.core.ann.AutowiredConstructorDemo$DemoB@64c87930
AutowiredConstructorDemo{
   demoA=com.spring.core.ann.AutowiredConstructorDemo$DemoA@2357d90a, demoB=com.spring.core.ann.AutowiredConstructorDemo$DemoB@64c87930}
```

在上面的案例中，由于DemoB继承了DemoA，而参数类型为DemoA，因此在按照类型查找bean是会查找到两个bean（多态，类型向下兼容，会将子类bean也查询出来），一个demoA、一个demoB，随后会根据参数名匹配bean name继续注入，最终会注入成功，如果我们的参数名不是demoA和demoB，那么会由于匹配不到唯一的bean而抛出异常！

现在，我们尝试把构造器上的注解去掉，会发生什么呢？实际上，我们测试之后会发现，所依赖的对象照样被注入进来了。实际上，从Spring4.3开始，如果目标bean只定义了一个构造函数，那么就不再需要在此类构造函数上使用@Autowired注解，Spring会自动尝试注入。当然，这要求要么是无参构造器，要么构造器参数类型的bean在Spring容器中都能找到，否则还是会报错！下面有规律总结！

#### 3.4.1.1 非必须依赖

@Autowired注解标注的构造器上的参数默认都必须全部注入，即要保证所有的参数都能在IoC容器中找到对应的bean。

如果我们想要一种能找到就注入，找不到就不注入的情况，那么我们可以设置@Autowired注解的required属性为false（默认为true），为false就表示构造构造器上的参数都是非必须的依赖，这样即使找不到依赖项也不会报错！

我们将@Autowired的required设为false，并且demoB的Component注解注释掉，看看会发生什么。

```java
/**
     * 要求在IoC中查找并注入两个依赖属性
     */
    @Autowired(required = false)
    public AutowiredConstructorDemo(DemoA demoA, DemoB demoB) {
   
        System.out.println("demoA"+"demoB");
        this.demoA = demoA;
        this.demoB = demoB;
    }
    //@Component("demoB")
    public static class DemoB extends DemoA {
   
        public DemoB() {
   
            System.out.println("DemoB："+this);
        }
    }
```

还是刚才的测试，结果抛出了异常，并没有出现所希望的按需注入：

```java
expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {
   }
```

实际上@Autowired的规则没这么简单，它的规则如下：

1. 如果一个bean有多个构造器，那么只能有一个构造器添加@Autowired注解并设置required为true或者添加@Autowired注解（默认required就为true）。这表示容器将会只使用该构造器进行依赖注入。 
2. @Autowired注解的required属性设置为false，那么可以对多个构造器应用该注解，这表示有多个构造器可以作为“候选构造器”。对于这些构造器，Spring将选择能够满足最多、最匹配的依赖项的构造器作为依赖注入的构造器，“最多”就是能满足依赖注入的参数最多的构造器，“最匹配”就是@Autowired的注入规则，匹配类型能满足的优先（其中类型一致的更优先），匹配名称能满足的其次。 
3. 如果希望Spring最终选择其中一个候选构造器作为依赖注入的构造器，必须满足该构造器上的参数全部都能成功注入，其他的参数则忽略。如果所有的候选构造器中没有一个能满足构造器上的参数全部都能成功注入，那么会直接调用默认无参构造器，这是所有依赖都不会注入。上面抛出的原因就是候选构造器不满足要求，但是也没有提供无参构造器，到这无法实例化bean。 
4. 如果一个bean声明多个构造器，但没有一个构造器应用@Autowired注解，则将使用无参构造器，这样的话就不会进行构造器依赖注入。如果类仅声明一个构造器，Spring将默认使用这个构造器进行依赖注入，即使没有@Autowired注解。另外，带注解的构造器不必是public的。

有了上面的规则，我们现在知道怎么处理这个问题，那就是加一个无参构造器即可，或则加一个只有demoA参数的构造器，并且让它作为候选构造器，这样就能满足注入demoA并且也不会抛出异常。

```java
/**
 * @author lx
 */
@Component
public class AutowiredConstructorDemo {
   

    private DemoA demoA;

    private DemoB demoB;

    @Autowired(required = false)
    public AutowiredConstructorDemo(DemoA demoA, DemoB demoB) {
   
        System.out.println("demoA"+"demoB");
        this.demoA = demoA;
        this.demoB = demoB;
    }

    /**
     * 添加只有demoA参数的构造器，并且作为候选构造器
     */
    @Autowired(required = false)
    public AutowiredConstructorDemo(DemoA demoA) {
   
        this.demoA = demoA;
    }



    public AutowiredConstructorDemo() {
   
    }

    @Component("demoA")
    public static class DemoA {
   
        public DemoA() {
   
            System.out.println("DemoA："+this);
        }
    }

    //@Component("demoB")
    public static class DemoB extends DemoA {
   
        public DemoB() {
   
            System.out.println("DemoB："+this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredConstructorDemo{" +
                "demoA=" + demoA +
                ", demoB=" + demoB +
                '}';
    }
}
```

测试结果如下：

```java
DemoA：com.spring.core.ann.AutowiredConstructorDemo$DemoA@7f9fcf7f
AutowiredConstructorDemo{
   demoA=com.spring.core.ann.AutowiredConstructorDemo$DemoA@7f9fcf7f, demoB=null}
```

现在继续测试，继续添加只有一个只有demoB参数的构造器，并且让它作为候选构造器，然后将DemoA的bean移除容器，将DemoB的bean交给容器。

```java
/**
 * @author lx
 */
@Component
public class AutowiredConstructorDemo {
   

    private DemoA demoA;

    private DemoB demoB;

    @Autowired(required = false)
    public AutowiredConstructorDemo(DemoA demoA, DemoB demoB) {
   
        System.out.println("demoA"+"demoB");
        this.demoA = demoA;
        this.demoB = demoB;
    }

    /**
     * 添加只有demoA参数的构造器，并且作为候选构造器
     */
    @Autowired(required = false)
    public AutowiredConstructorDemo(DemoA demoA) {
   
        System.out.println("demoA");
        this.demoA = demoA;
    }

    /**
     * 添加只有demoA参数的构造器，并且作为候选构造器
     */
    @Autowired(required = false)
    public AutowiredConstructorDemo(DemoB demoB) {
   
        System.out.println("demoB");
        this.demoB = demoB;
    }

    public AutowiredConstructorDemo() {
   
        System.out.println("无参");
    }

    //@Component("demoA")
    public static class DemoA {
   
        public DemoA() {
   
            System.out.println("DemoA："+this);
        }
    }

    @Component("demoB")
    public static class DemoB extends DemoA {
   
        public DemoB() {
   
            System.out.println("DemoB："+this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredConstructorDemo{" +
                "demoA=" + demoA +
                ", demoB=" + demoB +
                '}';
    }
}
```

测试结果如下：

```java
DemoA：com.spring.core.ann.AutowiredConstructorDemo$DemoB@2357d90a
DemoB：com.spring.core.ann.AutowiredConstructorDemo$DemoB@2357d90a
demoAdemoB
AutowiredConstructorDemo{
   demoA=com.spring.core.ann.AutowiredConstructorDemo$DemoB@2357d90a, demoB=com.spring.core.ann.AutowiredConstructorDemo$DemoB@2357d90a}
```

我们发现，调用了最多参数的构造器。虽然DemoA的bean被移除容器，但是DemoA的bean还在容器中，在根据类型查找时，由于类型向下兼容，这两个参数都能匹配到demoB的bean，由于该构造器匹配到的依赖项最多，因此选择该构造器进行依赖注入。并且最终两个属性都注入一个相同的bean，那就是demoB。

### 4.1.2 使用Optional

根据上面的@Autowired的规则，我们发现，即使设置了required=false，只要有一个依赖项不能注入，那么该构造器就不会使用。为此，我们必须将可能的构造器参数组合都列出来全部作为候选构造器，才能最终实现“有就注入，没有就不注入”的功能，非常麻烦。

**现在，我们可以使用Java的Optional来实现真正的“有就注入，没有就不注入”的功能。关于Optional类，在Java 8 lambda的文章中有详细描述，它是Java8新增的一个类，表示可能包含或不包含非空值的容器对象，用来防止空指针异常。**

我们来对比一下使用Optional和不使用Optional的区别！

如下案例，一个AutowiredConstructorOptionalDemo类，测试Optional的作用，首先提供两个构造器，两个参数的构造器为候选构造器，一个无参构造器。OptionalDemoA注册到了容器中，OptionalDemoB并没有注册到容器中：

```java
/**
 * @author lx
 */
@Component
public class AutowiredConstructorOptionalDemo {
   

    private OptionalDemoA optionalDemoA;

    private OptionalDemoB optionalDemoB;


    @Autowired(required = false)
    public AutowiredConstructorOptionalDemo(OptionalDemoA optionalDemoA, OptionalDemoB optionalDemoB) {
   
        System.out.println("optionalDemoA" + "optionalDemoB");
        this.optionalDemoA = optionalDemoA;
        this.optionalDemoB = optionalDemoB;
    }

    public AutowiredConstructorOptionalDemo() {
   
        System.out.println("无参");
    }

    @Component("optionalDemoA")
    public static class OptionalDemoA {
   
        public OptionalDemoA() {
   
            System.out.println("OptionalDemoA：" + this);
        }
    }

    //@Component("optionalDemoB")
    public static class OptionalDemoB extends OptionalDemoA {
   
        public OptionalDemoB() {
   
            System.out.println("OptionalDemoB：" + this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredConstructorOptionalDemo{" +
                "optionalDemoA=" + optionalDemoA +
                ", optionalDemoB=" + optionalDemoB +
                '}';
    }
}
```

测试：

```java
OptionalDemoA：com.spring.core.ann.AutowiredConstructorOptionalDemo$OptionalDemoA@7f9fcf7f
无参
AutowiredConstructorOptionalDemo{
   optionalDemoA=null, optionalDemoB=null}
```

可以看到，最终调用了无参构造器的方法，optionalDemoA的依赖虽然在容器中，但是也没有注入进来，仅仅是因为optionalDemoB的依赖不在容器中，并且如果我们不提供无参构造器还会抛出异常。

现在使用Optional来改造，将上面两个构造器都注释掉，使用Optional来包装我们的参数类型，只提供两个参数的构造器，其他参数不变：

```java
/**
 * @author lx
 */
@Component
public class AutowiredConstructorOptionalDemo {
   

    private OptionalDemoA optionalDemoA;

    private OptionalDemoB optionalDemoB;


//    @Autowired(required = false)
//    public AutowiredConstructorOptionalDemo(OptionalDemoA optionalDemoA, OptionalDemoB optionalDemoB) {
   
//        System.out.println("optionalDemoA" + "optionalDemoB");
//        this.optionalDemoA = optionalDemoA;
//        this.optionalDemoB = optionalDemoB;
//    }
//
//    public AutowiredConstructorOptionalDemo() {
   
//        System.out.println("无参");
//    }

    public AutowiredConstructorOptionalDemo(Optional<OptionalDemoA> optionalDemoA,
                                            Optional<OptionalDemoB> optionalDemoB) {
   
        System.out.println("optionalDemoA" + "optionalDemoB");
        optionalDemoA.ifPresent(i -> this.optionalDemoA = i);
        optionalDemoB.ifPresent(i -> this.optionalDemoB = i);
    }

    @Component("optionalDemoA")
    public static class OptionalDemoA {
   
        public OptionalDemoA() {
   
            System.out.println("OptionalDemoA：" + this);
        }
    }

    //@Component("optionalDemoB")
    public static class OptionalDemoB extends OptionalDemoA {
   
        public OptionalDemoB() {
   
            System.out.println("OptionalDemoB：" + this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredConstructorOptionalDemo{" +
                "optionalDemoA=" + optionalDemoA +
                ", optionalDemoB=" + optionalDemoB +
                '}';
    }
}
```

继续测试，结果如下：

```java
OptionalDemoA：com.spring.core.ann.AutowiredConstructorOptionalDemo$OptionalDemoA@7f9fcf7f
optionalDemoAoptionalDemoB
AutowiredConstructorOptionalDemo{
   optionalDemoA=com.spring.core.ann.AutowiredConstructorOptionalDemo$OptionalDemoA@7f9fcf7f, optionalDemoB=null}
```

我们发现，不但没有报错，而且IoC容器中存在的依赖OptionalDemoA也注入进来了，不存在的依赖则默认为null，这样才是真正符合我们的“有就注入，没有就不注入”的功能，而且实现的非常简单！

我们可以对非必需的依赖都使用Optional包装，表示该项依赖是非必需的。实际上，这可以看作装饰设计模式的应用，Spring无论找没找到对应的依赖，都会将返回值分别使用一个Optional对象包装。这样，就是返回null，但是Optional对象不为null，因此不会报错。

Optional还会抵消required = true带来的强制依赖注入非null检查。即，即使没有这个依赖项并且required=true，但是如果使用Optional包装依赖参数的类型，那么永远不会抛出异常。因此，对于必须依赖项，不建议使用Optional包装！对于非必须依赖项，则建议使用Optional包装！

#### 3.4.1.3 使用@Nullable

在Spring 5.0之后，我们还可以使用Spring提供的@Nullable注解或者JSR-305中的@Nullable注解标注到参数字段上，实现和Optional同样的“有就注入，没有就不注入”的功能。并且，@Nullable同样会抵消无论是默认的还是手动指定的required=true带来的强制依赖注入非null检查。

如下案例，有一个AutowiredConstructorNullableDemo类，用于测试@Nullable注解：

```java
/**
 * @author lx
 */
@Component
public class AutowiredConstructorNullableDemo {
   

    private NullableDemoA nullableDemoA;

    private NullableDemoB nullableDemoB;


    public AutowiredConstructorNullableDemo(@Nullable NullableDemoA nullableDemoA,
                                            @Nullable NullableDemoB nullableDemoB) {
   
        System.out.println("调用两参数构造器");
        this.nullableDemoA = nullableDemoA;
        this.nullableDemoB = nullableDemoB;
    }

    @Component("nullableDemoA")
    public static class NullableDemoA {
   
        public NullableDemoA() {
   
            System.out.println("NullableDemoA：" + this);
        }
    }

    //@Component("nullableDemoB")
    public static class NullableDemoB extends NullableDemoA {
   
        public NullableDemoB() {
   
            System.out.println("NullableDemoB：" + this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredConstructorNullableDemo{" +
                "nullableDemoA=" + nullableDemoA +
                ", nullableDemoB=" + nullableDemoB +
                '}';
    }
}
```

测试：

```java
@Test
public void autowiredConstructorNullable() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredConstructorNullableDemo", AutowiredConstructorNullableDemo.class));
}
```

结果如下，成功实现“有就注入，没有就不注入”的功能，并且没有抛出异常：

```java
NullableDemoA：com.spring.core.ann.AutowiredConstructorNullableDemo$NullableDemoA@7f9fcf7f
调用两参数构造器
AutowiredConstructorNullableDemo{
   nullableDemoA=com.spring.core.ann.AutowiredConstructorNullableDemo$NullableDemoA@7f9fcf7f, nullableDemoB=null}
```

### 3.4.2 标注在方法上

@Autowired标注在方法上时，表示对该方法的所有参数从IoC中查找匹配的对象注入进去，查找的类型就是参数类型，查找的name就是参数名。

实际上，我们上面讲的构造器就是一种特殊的方法。因此，标注在方法上时，和标注在构造器上差不多。**对于已经被@Bean注解标注的方法，方法参数的依赖会从IoC容器中会自动查找并注入，这就类似于只有一个构造器是的参数会自动注入一样！**

如下案例，有一个AutowiredSetterDemo类，用于测试标注在方法上：

```java
/**
 * @author lx
 */
@Component
public class AutowiredSetterDemo {
   


    private SetterDemoA setterDemoA;
    private SetterDemoB setterDemoB;


    //标注在方法上

    @Autowired
    public void setSetterDemoA(SetterDemoA setterDemoA) {
   
        this.setterDemoA = setterDemoA;
    }

    @Autowired
    public void setSetterDemoB(SetterDemoB setterDemoB) {
   
        this.setterDemoB = setterDemoB;
    }


    /**
     * 在@Bean注解标注的方法上，参数会自动尝试注入，不需要标注@Autowired，如果没找到依赖就会抛出异常
     * 因此如果是非必须依赖，那么可以使用@Autowired(required = false)或者Optional包装参数类
     */
    @Bean
    public SetterDemoA getSetterDemoA(Optional<SetterDemoB> setterDemoB) {
   
        setterDemoB.ifPresent(demoB -> System.out.println("@Bean自动注入参数依赖：" + demoB));
        return new SetterDemoA();
    }

    @Component
    public static class SetterDemoA {
   
        public SetterDemoA() {
   
            System.out.println("SetterDemoA初始化：" + this);
        }
    }

    @Component
    public static class SetterDemoB {
   
        public SetterDemoB() {
   
            System.out.println("SetterDemoB初始化：" + this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredSetterDemo{" +
                "setterDemoA=" + setterDemoA +
                ", setterDemoB=" + setterDemoB +
                '}';
    }
}
```

测试：

```java
@Test
public void autowiredSetter() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredSetterDemo", AutowiredSetterDemo.class));
}
```

结果如下，成功注入：

```java
SetterDemoA初始化：com.spring.core.ann.AutowiredSetterDemo$SetterDemoA@5b0abc94
SetterDemoB初始化：com.spring.core.ann.AutowiredSetterDemo$SetterDemoB@14a2f921
@Bean自动注入参数依赖：com.spring.core.ann.AutowiredSetterDemo$SetterDemoB@14a2f921
SetterDemoA初始化：com.spring.core.ann.AutowiredSetterDemo$SetterDemoA@4b44655e
AutowiredSetterDemo{
   setterDemoA=com.spring.core.ann.AutowiredSetterDemo$SetterDemoA@5b0abc94, setterDemoB=com.spring.core.ann.AutowiredSetterDemo$SetterDemoB@14a2f921}
```

### 3.4.3 标注在属性上

@Autowired标注在属性上时，表示对该属性从IoC中查找匹配的对象注入进去，查找的类型就是属性类型，查找的name就是属性名。

这种方式实际上是在开发过程中最常见的方式，因为只需要标注在属性上即可，不需要提供构造器，或者方法，非常简单。它的原理是通过反射获取该类对应的字段，然后将依赖设置进去的，因此必须要提供相应的构造器和方法！

如下案例，有一个AutowiredFieldDemo类，用于测试标注在属性上：

```java
/**
 * @author lx
 */
@Component
public class AutowiredFieldDemo {
   

    //直接标注在字段上即可，如果是非必须依赖，那么可以使用@Autowired(required = false)

    @Autowired
    private FieldDemoA fieldDemoA;

    @Autowired(required = false)
    private FieldDemoB fieldDemoB;

    @Component
    public static class FieldDemoA {
   
        public FieldDemoA() {
   
            System.out.println("FieldDemoA初始化：" + this);
        }
    }

    @Component
    public static class FieldDemoB {
   
        public FieldDemoB() {
   
            System.out.println("FieldDemoB初始化：" + this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredFieldDemo{" +
                "fieldDemoA=" + fieldDemoA +
                ", fieldDemoB=" + fieldDemoB +
                '}';
    }
}
```

测试：

```java
@Test
public void autowiredField() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredFieldDemo", AutowiredFieldDemo.class));
}
```

结果如下，成功注入：

```java
FieldDemoA初始化：com.spring.core.ann.AutowiredFieldDemo$FieldDemoA@2357d90a
FieldDemoB初始化：com.spring.core.ann.AutowiredFieldDemo$FieldDemoB@64c87930
AutowiredFieldDemo{
   fieldDemoA=com.spring.core.ann.AutowiredFieldDemo$FieldDemoA@2357d90a, fieldDemoB=com.spring.core.ann.AutowiredFieldDemo$FieldDemoB@64c87930}
```

### 3.4.4 获取全部bean

我们也可以使用以上方式混合标注。另外，还可以通过将注解添加到需要该类型数组的字段或方法，就可以从ApplicationContext中获取到该特定类型的所有bean，并且回向下兼容，这同样适用于类型化集合List、Set、Map……，同样支持上面三种注入方式。

对于有顺序的集合，我们还可以指定注入的bean排列顺序，通过在bean定义上添加@Order注解，并且设置int类型的值来对有序集合的依赖注入顺序进行排序，@Order的值越小，则排序越靠前，@Order对应有一个Ordered接口，内部有两个int类型的常量字段HIGHEST_PRECEDENCE和LOWEST_PRECEDENCE，代表int类型的最小值和最大值，即表示排在有序集合头部或者尾部，我们可以直接使用这两个字段！注意@Order不代表单例bean的启动顺序，仅仅是注入到集合的顺序。

如下案例，有一个AutowiredAllDemo类，用于测试注入全部bean：

```java
/**
 * @author lx
 */
@Component
public class AutowiredAllDemo {
   

    //获取该类型的全部bean，会向下兼容

    @Autowired
    private AutowiredAllA[] autowiredAllAS;

    @Autowired
    private AutowiredAllB[] autowiredAllBS;

    private AutowiredAllC[] autowiredAllCS;

    @Autowired
    private List<AutowiredAllA> autowiredAllAList;

    private Set<AutowiredAllA> autowiredAllASet;

    /**
     * Map的key为String类型，就是bean的名字
     */
    @Autowired
    private Map<String, AutowiredAllA> autowiredAllAMap;

    /**
     * 支持方法注入
     */
    @Autowired
    public void setAutowiredAllASet(Set<AutowiredAllA> autowiredAllASet) {
   
        this.autowiredAllASet = autowiredAllASet;
    }

    /**
     * 支持构造器注入
     */
    public AutowiredAllDemo(AutowiredAllC[] autowiredAllCS) {
   
        this.autowiredAllCS = autowiredAllCS;
    }

    @Component
    public static class AutowiredAllA {
   
    }

    @Component
    public static class AutowiredAllB extends AutowiredAllA {
   
    }

    /**
     * 指定最高优先
     */
    @Order(Ordered.HIGHEST_PRECEDENCE)
    @Component
    public static class AutowiredAllC extends AutowiredAllB {
   
    }


    @Override
    public String toString() {
   
        return "AutowiredAllDemo{" + "\n" +
                "autowiredAllAS=" + Arrays.toString(autowiredAllAS) + "\n" +
                ", autowiredAllBS=" + Arrays.toString(autowiredAllBS) + "\n" +
                ", autowiredAllCS=" + Arrays.toString(autowiredAllCS) + "\n" +
                ", autowiredAllAList=" + autowiredAllAList + "\n" +
                ", autowiredAllASet=" + autowiredAllASet + "\n" +
                ", autowiredAllAMap=" + autowiredAllAMap +
                '}';
    }
}
```

测试：

```java
@Test
public void autowiredAll() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredAllDemo", AutowiredAllDemo.class));
}
```

结果如下，都成功注入：

```java
AutowiredAllDemo{
   
autowiredAllAS=[com.spring.core.ann.AutowiredAllDemo$AutowiredAllC@6107227e, com.spring.core.ann.AutowiredAllDemo$AutowiredAllA@184cf7cf, com.spring.core.ann.AutowiredAllDemo$AutowiredAllB@2fd6b6c7]
, autowiredAllBS=[com.spring.core.ann.AutowiredAllDemo$AutowiredAllC@6107227e, com.spring.core.ann.AutowiredAllDemo$AutowiredAllB@2fd6b6c7]
, autowiredAllCS=[com.spring.core.ann.AutowiredAllDemo$AutowiredAllC@6107227e]
, autowiredAllAList=[com.spring.core.ann.AutowiredAllDemo$AutowiredAllC@6107227e, com.spring.core.ann.AutowiredAllDemo$AutowiredAllA@184cf7cf, com.spring.core.ann.AutowiredAllDemo$AutowiredAllB@2fd6b6c7]
, autowiredAllASet=[com.spring.core.ann.AutowiredAllDemo$AutowiredAllA@184cf7cf, com.spring.core.ann.AutowiredAllDemo$AutowiredAllB@2fd6b6c7, com.spring.core.ann.AutowiredAllDemo$AutowiredAllC@6107227e]
, autowiredAllAMap={
   autowiredAllDemo.AutowiredAllA=com.spring.core.ann.AutowiredAllDemo$AutowiredAllA@184cf7cf, autowiredAllDemo.AutowiredAllB=com.spring.core.ann.AutowiredAllDemo$AutowiredAllB@2fd6b6c7, autowiredAllDemo.AutowiredAllC=com.spring.core.ann.AutowiredAllDemo$AutowiredAllC@6107227e}}
```

### 3.4.5 其他

如果使用三种混合注入，那么它们的先后顺序是：构造器注入-&gt;属性反射注入-&gt;方法注入，因此它们对同一个属性设置的依赖可能会相互覆盖，后面的注入行为将覆盖前面的注入行为。

我们还可以使用@Autowired直接注入已知可解析依赖项：BeanFactory、ApplicationContext、Environment、ResourceLoader、ApplicationEventPublisher和MessageSource。这些接口及其扩展接口（如ConfigurableApplicationContext或ResourcePatternResolver）将自动解析并注入，无需特殊设置。

而由于@Autowired, @Inject, @Value, 和 @Resource 注解是在Spring的BeanPostProcessor中处理的，这意味着我们不能将这些注解用在自定义的BeanPostProcessor，BeanFactoryPostProcessor类中。这些类的bean配置必须使用XML或者@Bean方法来显示指定。

前面我们说过，对于@Bean注解标注的方法，它的方法的参数会被自动注入，但是它的自动注入功能远远没有这么简单。

实际上，@Bean注解标注的方法，一般我们会在内部创建的返回的bean实例，并设置相应的属性，这个属性就是通过参数传递来的。如果我们没有主动设置属性，按理说应该返回的bean应该是一个空属性的bean，然而，如果我们在该bean的类中设置了注解将该bean交给了容器管理，比如类上面标注了@Component（@Controller、@Service、@Repository）、@Configuration……，并且使用了方法或者属性的属性自动依赖注入，那么通过@Bean即使返回该类的空对象，IoC容器同样会对我们自己实例化的bean自动注入依赖，并且我们自己在@Bean的方法中自己设置的其他属性也将会被替代。所以说，为了避免这种问题，通常@Bean用于为那些不能通过直接添加@Component等注解交给IoC容器管理的bean设置属性并交给IoC容器，这样就不会出现问题，一定要注意！（在后面的@Qualifier注解部分会见识到）。

## 3.5 @primary主要候选依赖

@Autowired在按照类型无法选择唯一的依赖注入时（比如存在多个类型兼容或者相同的候选依赖），可以使用Spring的@Primary注解，@Primary注解表示当有多个bean是该类型依赖项的候选对象时，应该优先考虑特定的被@Primary注解修饰的bean。

@Primary类似于XML装配中的primary属性，区别是：XML装配中的primary是绑定到一个bean上面的，而@Primary注解装配则是绑定到类上面的。

如果同一个类型（包括兼容类型）的不同bean中有超过一个bean被标注了@Primary注解。那么如果是采用方法注入或者字段注入或者采用构造器注入并且required=true，在运行时将直接抛出异常：available: more than one ‘primary’ bean found among candidates……。如果采用构造器注入并且required=false，那么将不会选择该构造器，最终可能会选择无参构造器，即不注入该依赖，如果没有其他构造器可选，那么同样抛出异常。

使用了@Primary注解，那么就不会走后面根据name匹配的逻辑，如果@Primary注解无法筛选出唯一的依赖，那么直接抛出异常或者不注入该依赖（上面讲了）。

如下案例，有一个AutowiredPrimaryDemo类，测试@primary：

```java
/**
 * @author lx
 */
@Component
public class AutowiredPrimaryDemo {
   

    @Autowired
    private NullableDemoA nullableDemo;

    @Component("nullableDemoA")
    public static class NullableDemoA {
   
        public NullableDemoA() {
   
            System.out.println("NullableDemoA：" + this);
        }
    }

    /**
     * 选择主要依赖
     */
    @Component("nullableDemoB")
    @Primary
    public static class NullableDemoB extends NullableDemoA {
   
        public NullableDemoB() {
   
            System.out.println("NullableDemoB：" + this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredPrimaryDemo{" +
                "nullableDemo=" + nullableDemo +
                '}';
    }
}
```

测试：

```java
@Test
public void autowiredPrimary() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredPrimaryDemo", AutowiredPrimaryDemo.class));
}
```

结果如下，成功选择主依赖（选择的NullableDemoB）：

```java
NullableDemoA：com.spring.core.ann.AutowiredPrimaryDemo$NullableDemoA@15bb6bea
NullableDemoA：com.spring.core.ann.AutowiredPrimaryDemo$NullableDemoB@525f1e4e
NullableDemoB：com.spring.core.ann.AutowiredPrimaryDemo$NullableDemoB@525f1e4e
AutowiredPrimaryDemo{
   nullableDemo=com.spring.core.ann.AutowiredPrimaryDemo$NullableDemoB@525f1e4e}
```

## 3.6 @Qualifier选择bean name

@Qualifier注解可以设置一个value值，该值就是一个bean的名字。@Qualifier只有被标注在属性或者方法/构造器的参数前面才会生效。基于XML的配置中，我们也可以使用&lt; Qualifier&gt;标签完成类似的功能。

如下案例，有一个AutowiredQualifierDemo类，用于测试@Qualifier注解：

```java
/**
 * @author lx
 */
@Component
public class AutowiredQualifierDemo {
   

    /**
     * Qualifier指定bean name为qualifierDemoB的bean注入进来
     */
    @Autowired
    @Qualifier("qualifierDemoA")
    private QualifierDemoA qualifierDemo;


    public void setQualifierDemo(QualifierDemoA qualifierDemo) {
   
        this.qualifierDemo = qualifierDemo;
    }

    /**
     * 标注在参数前面才会生效
     */
    @Bean
    public AutowiredQualifierDemo getAutowiredQualifierDemo(@Qualifier("qualifierDemoB") QualifierDemoA qualifierDemo) {
   
        System.out.println("qualifierDemo--------" + qualifierDemo);
        AutowiredQualifierDemo autowiredQualifierDemo = new AutowiredQualifierDemo();
        //虽然现在设置的是qualifierDemoB对象，但是交给容器之后会被替换，相当于又被注入了一次qualifierDemoA
        autowiredQualifierDemo.setQualifierDemo(qualifierDemo);
        return autowiredQualifierDemo;
    }

    @Component("qualifierDemoA")
    public static class QualifierDemoA {
   
        public QualifierDemoA() {
   
            System.out.println("QualifierDemoA：" + this);
        }
    }

    @Component("qualifierDemoB")
    public static class QualifierDemoB extends QualifierDemoA {
   
        public QualifierDemoB() {
   
            System.out.println("QualifierDemoB：" + this);
        }
    }

    @Override
    public String toString() {
   
        return "AutowiredQualifierDemo{" +
                "nullableDemo=" + qualifierDemo +
                '}';
    }
}
```

测试：

```java
@Test
public void autowiredQualifier() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredQualifierDemo", AutowiredQualifierDemo.class));
    //通过@Bean返回的getAutowiredQualifierDemo对象和交给IoC容器管理的autowiredQualifierDemo对象的属性是同一个对象
    //这就是在前面说的@Bean对于已经交给IoC容器自动管理的bean的自动注入问题，我们在@Bean中设置的属性被覆盖了。
    System.out.println(ac.getBean("getAutowiredQualifierDemo", AutowiredQualifierDemo.class));
}
```

结果如下，成功选择bean name：

```java
QualifierDemoA：com.spring.core.ann.AutowiredQualifierDemo$QualifierDemoA@5b0abc94
QualifierDemoA：com.spring.core.ann.AutowiredQualifierDemo$QualifierDemoB@14a2f921
QualifierDemoB：com.spring.core.ann.AutowiredQualifierDemo$QualifierDemoB@14a2f921
qualifierDemo--------com.spring.core.ann.AutowiredQualifierDemo$QualifierDemoB@14a2f921
AutowiredQualifierDemo{
   nullableDemo=com.spring.core.ann.AutowiredQualifierDemo$QualifierDemoA@5b0abc94}
AutowiredQualifierDemo{
   nullableDemo=com.spring.core.ann.AutowiredQualifierDemo$QualifierDemoA@5b0abc94}
```

## 3.7 泛型选择注入

Java泛型类型也可以隐式作为类型选择，可以用在属性或者参数上。如果某一些bean都实现了一个通用的接口，那么可以使用接口类型和泛型类型作为属性的类型，这同样适用于数组和集合。

如下案例，有一个AutowiredGenericDemo，用于测试泛型选择注入：

```java
/**
 * 泛型选择
 *
 * @author lx
 */
@Component
public class AutowiredGenericDemo {
   

    @Autowired
    private AutowiredGeneric<AutowiredGenericA> autowiredGenericAAutowiredGeneric;
    @Autowired
    private AutowiredGeneric<AutowiredGenericB> autowiredGenericBAutowiredGeneric;
    @Autowired
    private AutowiredGeneric<AutowiredGenericC> autowiredGenericCAutowiredGeneric;

    @Autowired
    private AutowiredGeneric<AutowiredGenericA>[] autowiredGenerics;

    //……其他集合也可以

    @Component
    public static class AutowiredGenericA implements AutowiredGeneric<AutowiredGenericA> {
   

    }

    @Component
    public static class AutowiredGenericB implements AutowiredGeneric<AutowiredGenericB> {
   
    }

    @Component
    public static class AutowiredGenericC implements AutowiredGeneric<AutowiredGenericC> {
   
    }

    public interface AutowiredGeneric<T> {
   

    }

    @Override
    public String toString() {
   
        return "AutowiredGenericDemo{" + "\n" +
                "autowiredGenericAAutowiredGeneric=" + autowiredGenericAAutowiredGeneric + "\n" +
                ", autowiredGenericBAutowiredGeneric=" + autowiredGenericBAutowiredGeneric + "\n" +
                ", autowiredGenericCAutowiredGeneric=" + autowiredGenericCAutowiredGeneric + "\n" +
                ", autowiredGenerics=" + Arrays.toString(autowiredGenerics) +
                '}';
    }
}
```

测试：

```java
@Test
public void autowiredGeneric() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("autowiredGenericDemo", AutowiredGenericDemo.class));
}
```

结果如下，成功选择注入：

```java
AutowiredGenericDemo{
   
autowiredGenericAAutowiredGeneric=com.spring.core.ann.AutowiredGenericDemo$AutowiredGenericA@a4102b8
, autowiredGenericBAutowiredGeneric=com.spring.core.ann.AutowiredGenericDemo$AutowiredGenericB@3b07a0d6
, autowiredGenericCAutowiredGeneric=com.spring.core.ann.AutowiredGenericDemo$AutowiredGenericC@11a9e7c8
, autowiredGenerics=[com.spring.core.ann.AutowiredGenericDemo$AutowiredGenericA@a4102b8]}
```

## 3.8 @Resource依赖自动注入

类似于@Autowired注解，@Resource注解也是一个依赖注入注解。

相比于@Autowired，@Resource有一下特点：

1. @Resource注解是JSR-250提供的，位于javax.annotation.Resource，Spring也支持自动依赖注入。而@Autowired则是Spring提供的注解。 
2. @Resource注入规则为： 
 <ol> 
  2. 默认通过名称注入，首先获取name属性，作为查找的beanName，如果没有name属性： 
   <ol> 
    2. 如果是字段就默认属性名作为查找的beanName 
    2. 如果是setter方法：截取setter方法名的"set"之后的部分，并进行处理：如果至少开头两个字符是大写，那么就返回原截取的值，否则返回开头为小写的截取的值作为查找的beanName。 
   </ol>  
  2. 如果没有设置name属性或者没有在IoC容器中找到同名的bean实例或者bean定义，那么尝试使用类型匹配依赖，如果没有找到该类型的bean，或者找到多个（类型会向下兼容），那么抛出异常！和@Autowired正好相反！ 
  2. **更详细的规则在源码解析文章中有体现！** 
 </ol>  
8. @Resource也可以指定name和type属性，表示只通过bean name或者bean type匹配依赖项，两个属性都指定值，那么要求name和type都要匹配才行，type可以向上兼容。 
9. @Resource注解只有标注在属性和单个参数的方法上才生效，多个参数则抛出异常：@Resource annotation requires a single-arg method。因此，如果注入目标是一个构造器或多参数方法，那么应该坚持使用@Autowired。 
10. @Resource也可以配合@Qualifier使用，但是@Resource的name优先级高于@Qualifier指定的值。 
11. @Resource标注的方法或者属性，默认表示依赖必须注入，并且没有requird选项，不支持@Nullable，但是支持Optional包装。 
12. @Resource通过CommonAnnotationBeanPostProcessor处理器解析。同样可以直接注入已知的可解析依赖项：BeanFactory、ApplicationContext、ResourceLoader、ApplicationEventPublisher，和MessageSource接口。

如下案例，有一个ResourceDemo类，用于测试@Resource注解：

```java
/**
 * @author lx
 */
@Component
public class ResourceDemo {
   

    //注入已知可解析依赖项
    @Resource
    private ApplicationContext context;

    //默认按照属性名->类型

    @Resource
    private AutowiredGeneric autowiredGenericA;
    @Resource
    private AutowiredGenericB autowiredGeneric;


    /**
     * 指定bean name
     */
    @Resource(name = "autowiredGenericA")
    private AutowiredGeneric autowiredGeneric1;

    /**
     * 指定bean type
     */
    @Resource(type = AutowiredGenericB.class)
    private AutowiredGeneric autowiredGeneric2;


    /**
     * 指定bean name and class  要求都要匹配
     */
    @Resource(name = "autowiredGenericB", type = AutowiredGeneric.class)
    private AutowiredGeneric autowiredGeneric3;


    private AutowiredGeneric autowiredGeneric4;
    private AutowiredGeneric autowiredGeneric5;
    private AutowiredGeneric autowiredGeneric6;
    private AutowiredGeneric autowiredGeneric7;
    private AutowiredGeneric autowiredGeneric8;

    /**
     * 可以使用在方法上 同样是默认先name后type
     */
    @Resource
    public void setter4(AutowiredGeneric autowiredGenericA) {
   
        this.autowiredGeneric4 = autowiredGenericA;
    }


    /**
     * 可以配合Qualifier使用
     */
    @Resource
    public void setter5(@Qualifier("autowiredGenericB") AutowiredGeneric autowiredGeneric) {
   
        this.autowiredGeneric5 = autowiredGeneric;
    }

    /**
     * Resource和Qualifier都存在时，Resource的name 优先级更高
     */
    @Resource(name = "autowiredGenericA")
    public void setter6(@Qualifier("autowiredGenericB") AutowiredGeneric autowiredGeneric) {
   
        this.autowiredGeneric6 = autowiredGeneric;
    }


//    /**
//     * 多参数方法将抛出异常
//     */
//    @Resource
//    public void setterTwo(AutowiredGeneric autowiredGenericA, AutowiredGeneric autowiredGenericB) {
   
//        this.autowiredGeneric7 = autowiredGenericA;
//        this.autowiredGeneric8 = autowiredGenericB;
//    }


    /**
     * 支持Optional包装
     */
    @Resource
    public void setterTwo(Optional<AutowiredAllDemo> autowiredAllDemo) {
   
        System.out.println(autowiredAllDemo);
    }

    @Component("autowiredGenericA")
    public static class AutowiredGenericA implements AutowiredGeneric {
   

    }

    @Component("autowiredGenericB")
    public static class AutowiredGenericB implements AutowiredGeneric {
   
    }

    public interface AutowiredGeneric {
   

    }

    @Override
    public String toString() {
   
        return "ResourceDemo{" + "\n" +
                "context=" + context + "\n" +
                "autowiredGenericA=" + autowiredGenericA + "\n" +
                ", autowiredGeneric=" + autowiredGeneric + "\n" +
                ", autowiredGeneric1=" + autowiredGeneric1 + "\n" +
                ", autowiredGeneric2=" + autowiredGeneric2 + "\n" +
                ", autowiredGeneric3=" + autowiredGeneric3 + "\n" +
                ", autowiredGeneric4=" + autowiredGeneric4 + "\n" +
                ", autowiredGeneric5=" + autowiredGeneric5 + "\n" +
                ", autowiredGeneric6=" + autowiredGeneric6 + "\n" +
                ", autowiredGeneric7=" + autowiredGeneric7 + "\n" +
                ", autowiredGeneric8=" + autowiredGeneric8 +
                '}';
    }
}
```

测试：

```java
@Test
public void resource() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("resourceDemo", ResourceDemo.class));
}
```

结果如下，符合规则：

```java
Optional.empty
ResourceDemo{
   
context=org.springframework.context.support.ClassPathXmlApplicationContext@198e2867, started on Sat Sep 05 15:40:03 CST 2020
autowiredGenericA=com.spring.core.ann.ResourceDemo$AutowiredGenericA@184cf7cf
, autowiredGeneric=com.spring.core.ann.ResourceDemo$AutowiredGenericB@2fd6b6c7
, autowiredGeneric1=com.spring.core.ann.ResourceDemo$AutowiredGenericA@184cf7cf
, autowiredGeneric2=com.spring.core.ann.ResourceDemo$AutowiredGenericB@2fd6b6c7
, autowiredGeneric3=com.spring.core.ann.ResourceDemo$AutowiredGenericB@2fd6b6c7
, autowiredGeneric4=com.spring.core.ann.ResourceDemo$AutowiredGenericA@184cf7cf
, autowiredGeneric5=com.spring.core.ann.ResourceDemo$AutowiredGenericB@2fd6b6c7
, autowiredGeneric6=com.spring.core.ann.ResourceDemo$AutowiredGenericA@184cf7cf
, autowiredGeneric7=null
, autowiredGeneric8=null}
```

## 3.9 @Value外部属性注入

@Value类似于XML配置中的value属性和标签，可以用于字面量值的直接注入，或者提取外部properties/yml文件中的属性，基于SPEL的语法的字符串操作，提取容器中的bean的属性，转换为复杂集合，甚至进行值的计算……。实际上，在XML的配置中也可以使用SPEL的语法，关于SPEL属性注入的语法，后面会专门有文章讲解！

@Value的使用非常灵活，下面仅仅介绍常见用法。后面在Spring Boot中还会遇到另一个更加灵活的@ConfigurationProperties注解。

@Value可以用在属性、方法（普通方法、@Bean方法都行），参数（普通方法、@Bean方法、构造器都行）上。用在方法上时，表示对方法都注入相同的字面量值。

### 3.9.1 注入字面量值

@Value可以直接注入基本类型、包装类型、字符串类型等字面量值，Spring提供的内置转换器支持允许自动处理简单的类型转换，还可以转换为数组、集合等复杂结构，甚至通过SPEL操作字符串方法比如大小写、截取、拼接……等等，更多操作详情看下面的案例。

案例如下，一个ValueDemoStr类，测试字面量的注入：

```java
/**
 * Value直接注入字面量
 *
 * @author lx
 */
@Component
public class ValueDemoStr {
   
    //基本类型

    @Value("1")
    private byte b;
    @Value("1")
    private char c;
    @Value("1")
    private short s;
    @Value("1")
    private int i;
    @Value("1")
    private long l;
    @Value("1")
    private float f;
    @Value("1")
    private double d;
    //boolean类型机器姬包装类可以使用true或者false字符串
    //也可以使用0代表false，1代表true
    @Value("0")
    private boolean boo;

    //字符串

    @Value("1")
    private String string;
    @Value("1")
    private CharSequence charSequence;

    //包装类型

    @Value("1")
    private Byte aByte;
    @Value("1")
    private Character character;
    @Value("1")
    private Short aShort;
    @Value("1")
    private Integer integer;
    @Value("1")
    private Long aLong;
    @Value("1")
    private Float aFloat;
    @Value("1")
    private Double aDouble;
    @Value("1")
    private Boolean aBoolean;

    //某些整形基本类型数组，使用,分隔即可自动拆分
    @Value("1,2,3,0,11,2")
    private short[] shorts;
    @Value("1,2,3,0,11,2")
    private int[] ints;
    @Value("1,2 3")
    private long[] longs;


    //下面的要使用SPEL 表达式  #{'要分割的数据'.进行的操作}
    @Value("#{'1,2,3,0,11,2'.split(',')}")
    private byte[] bytes;
    @Value("#{'1,0,1,1'.split(',')}")
    private char[] chars;
    @Value("#{'1,0,1,1'.split(',')}")
    private float[] floats;
    @Value("#{'1,0,1,1'.split(',')}")
    private double[] doubles;
    @Value("#{'1,0,1,1'.split(',')}")
    private boolean[] booleans;

    //SPEL可以调用String类的方法很多方法

    //大写
    @Value("#{'abc'.toUpperCase()}")
    private String upperCase;
    //截取
    @Value("#{'abc'.substring(1)}")
    private String substring;
    //拼接
    @Value("#{'abc'.concat('def')}")
    private String concat;

    //当然支持更多字符串操作…………

    //SPEL计算

    @Value("#{4+2}")
    private int add;
    @Value("#{4-2}")
    private int subtract;
    @Value("#{4*2}")
    private int multiply;
    @Value("#{4/2}")
    private int divide;
    //SPEL取一个随机数
    @Value("#{ T(java.lang.Math).random() * 100.0}")
    public double property1;

    //当然支持更多复杂计算…………

    //SPEL转换数组和集合

    //单value集合    #{'value1,value2'.split(',')}

    @Value("#{'11,22,xx'.split(',')}")
    private String[] strings;
    @Value("#{'11,22, x x'.split(',')}")
    private CharSequence[] charSequences;
    @Value("#{'property1,property2,property2,property3'.split(',')}")
    private List<String> list;
    //set会自动去重
    @Value("#{'property1,property2,property2,property3'.split(',')}")
    private Set<String> set;

    //map集合 #{mapJsonValue} 类似于JSON格式  会自动去重

    @Value("#{
   {key1: 'value1', key2: 'value2', key2: 'value3'}}")
    private Map<String,String> map;


    //@Value可以用在方法上，表示对方法的所有参数都注入同一个指定值

    private int anInt1;
    private int anInt2;
    private boolean anInt3;

    @Value("11")
    public void setAnInt(int anInt1) {
   
        this.anInt1 = anInt1;
    }

    @Value("1")
    public void setTwo(int anInt1, boolean anInt2) {
   
        this.anInt2 = anInt1;
        this.anInt3 = anInt2;
    }


    //@Value用在方法参数上也行，表示对指定参数注入指定值

    private int anInt4;

    public ValueDemoStr(@Value("22") int anInt4) {
   
        this.anInt4 = anInt4;
    }

    @Override
    public String toString() {
   
        return "ValueDemoStr{" +
                "b=" + b +
                ", c=" + c +
                ", s=" + s +
                ", i=" + i +
                ", l=" + l +
                ", f=" + f +
                ", d=" + d +
                ", boo=" + boo +
                ", string='" + string + '\'' +
                ", charSequence=" + charSequence +
                ", aByte=" + aByte +
                ", character=" + character +
                ", aShort=" + aShort +
                ", integer=" + integer +
                ", aLong=" + aLong +
                ", aFloat=" + aFloat +
                ", aDouble=" + aDouble +
                ", aBoolean=" + aBoolean +
                ", shorts=" + Arrays.toString(shorts) +
                ", ints=" + Arrays.toString(ints) +
                ", longs=" + Arrays.toString(longs) +
                ", bytes=" + Arrays.toString(bytes) +
                ", chars=" + Arrays.toString(chars) +
                ", floats=" + Arrays.toString(floats) +
                ", doubles=" + Arrays.toString(doubles) +
                ", booleans=" + Arrays.toString(booleans) +
                ", upperCase='" + upperCase + '\'' +
                ", substring='" + substring + '\'' +
                ", concat='" + concat + '\'' +
                ", add=" + add +
                ", subtract=" + subtract +
                ", multiply=" + multiply +
                ", divide=" + divide +
                ", property1=" + property1 +
                ", strings=" + Arrays.toString(strings) +
                ", charSequences=" + Arrays.toString(charSequences) +
                ", list=" + list +
                ", set=" + set +
                ", map=" + map +
                ", anInt1=" + anInt1 +
                ", anInt2=" + anInt2 +
                ", anInt3=" + anInt3 +
                ", anInt4=" + anInt4 +
                '}';
    }
}
```

测试：

```java
@Test
public void valueDemoStr() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("valueDemoStr", ValueDemoStr.class));
}
```

### 3.9.2 注入文件的属性

@Value可以获取外部proerties或者yml文件的属性并注入。

1. 注入一般属性的格式就是：$ {key}，如果配置文件没有该key，则$ {key}本身将尝试作为值注入。（对于Spring Boot来说，则默认要求必须要有该key，否则抛出异常）。 
2. 注入单value集合的格式就是：#{’$ {key}’.split(’,’)}，里面的”,”是一个分隔符号，我们也可以设置自己的分隔符号。我们也可以使用String的其他方法，比如转换大小写，替换、截取、拼接等等，详情看上面的案例 
3. 注入key-value的map集合的格式就类似于JSON格式：{key1: ‘value1’, key2: ‘value2’} 
4. 可以看出来，实际上外部配置文件的value的写法和直接注入字面量值的写法都差不多（yml文件可能有区别）。只是通过$ {key}将value值引入进来而已。 
5. 可以在配置文件中，使用$ {otherKey}获取其他key的值作为自己的值。更多用法比如数学运算操作看下面的案例！


我们在resource目录下建立一个value.proerties文件： ![img](https://img-blog.csdnimg.cn/20200908120853631.png#pic_center) 然后在里面写一些键值对：

```java
key=value
a.b.c=xxx
#一定要统一编码解码格式，否则中文乱码
maohao:冒号分隔
kongge 空格分隔
int=4
double=2
boolean=1
#可以获取其他key的值作为value
xxx=${key}
#array 指定你想要的分隔符号，然后使用split拆分即可
booleans=1,0,0,1
#list 指定你想要的分隔符号，然后使用split拆分即可
list=1;xx;0;1;11
#set 指定你想要的分隔符号，然后使用split拆分即可 会自动去重
set=1,xx,0,1,11
#properties和map都是Map类型 会自动去重
#类似于JSON格式，可以获取其他key的值作为value作为map的key或者value
#properties
properties={key1: 'value1', ${xxx}: '${xxx}'}
#map
redirectUrl={key: '${key}',key: '${a.b.c}',${xxx}:'${booleans}'}
```

添加一个如下的类，用于测试读取配置文件的属性：

```java
/**
 * Value注入外部文件的值
 * 需要@PropertySource("classpath:value.properties")读取指定外部文件
 *
 * @author lx
 */
@Component
@PropertySource(value="classpath:value.properties",encoding="UTF-8")
public class ValueProperties {
   

    //注入字面量，通过 ${键} 格式来获取值并注入到属性中

    @Value("${key}")
    private String value;
    @Value("${int}")
    private char c;
    @Value("${double}")
    private double d;
    @Value("${boolean}")
    private boolean b;
    @Value("${xxx}")
    private String xxx;

    @Value("${maohao}")
    private String maohao;
    @Value("${kongge}")
    private String kongge;

    //取多个属性
    @Value("${kongge}${maohao}")
    private String concat;


    //SPEL注入数组和单value集合的样式

    @Value("#{'${booleans}'.split(',')}")
    private boolean[] booleans;
    @Value("#{'${list}'.split(';')}")
    private List<String> stringList;
    @Value("#{'${set}'.split(',')}")
    private Set<String> stringSet;

    //SPEL注入map的样式

    @Value("#{${properties}}")
    private Properties properties;
    @Value("#{${redirectUrl}}")
    private Map<String,String> map;


    //SPEL计算，当然还有其他更多复杂操作

    @Value("#{${int}+${double}}")
    private int add;
    @Value("#{${int}-${double}}")
    private int subtract;
    @Value("#{${int}*${double}}")
    private int multiply;
    @Value("#{${int}/${double}}")
    private int divide;


    @Override
    public String toString() {
   
        System.out.println("------properties-------");
        properties.forEach((o, o2) -> {
   
            System.out.println(o);
            System.out.println(o2);
        });
        System.out.println("------map-------");
        map.forEach((o, o2) -> {
   
            System.out.println(o);
            System.out.println(o2);
        });
        System.out.println("------str-------");
        return "ValueProperties{" +
                "value='" + value + '\'' +
                ", c=" + c +
                ", d=" + d +
                ", b=" + b +
                ", xxx='" + xxx + '\'' +
                ", maohao='" + maohao + '\'' +
                ", kongge='" + kongge + '\'' +
                ", concat='" + concat + '\'' +
                ", add='" + add + '\'' +
                ", subtract='" + subtract + '\'' +
                ", multiply='" + multiply + '\'' +
                ", divide='" + divide + '\'' +
                ", booleans=" + Arrays.toString(booleans) +
                ", stringList=" + stringList +
                ", stringSet=" + stringSet +
                ", properties=" + properties +
                ", map=" + map +
                '}';
    }
}
```

测试：

```java
@Test
public void valueProperties() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("valueProperties", ValueProperties.class));
}
```

#### 3.9.2.1 多配置文件与空属性检查

在上面的案例中，我们使用@PropertySource注解加载单个配置文件，如果有多个配置文件，那么可以使用@PropertySources加载多个配置文件，如果多个配置文件中有相同的key，那么后加载的配置文件将覆盖之前加载的配置文件的value值。先后加载的顺序就是声明加载的顺序，先声明的先加载。也可以就使用@PropertySource，填写多个文件参数，使用,分隔。另外，加载进来的配置全局通用，其他的bean也可以直接使用不需要再加载了。因此，实际上配置文件常常在@Configuration配置类中加载，而不是普通的bean中加载。

Spring默认情况下，如果配置文件没有key，则将$ {key}本身将尝试作为值注入，如果注入成功（比如属性是String类型则肯定能注入成功）则不会抛出异常。

如果我们需要启动检查没有的key，要对不存在的值保持严格控制，让它抛出异常，我们可以向IoC容器注册一个PropertySourcesPlaceholderConfigurer对象！

通常我们使用@Bean方式将PropertySourcesPlaceholderConfigurer注册到IoC容器中，并且要求对应的方法是static方法。在Spring Boot中，默认情况下，会有一个PropertySourcesPlaceholderConfigurer被初始化，自动从application.properties and application.yml中加载配置。

另外，可以重写PropertySourcesPlaceholderConfigurer的setPlaceholderPrefix、setPlaceholderSuffix、setValueSeparator方法来配置我们自己的占位符和解析。默认占位符实际上就是“${”、“}”等符号，Spring通过解析前置、后置占位符来获取key的信息，或者对里面的值进行拆分。

**在此前基于XML的配置中，我们使用property-placeholder来引入外部的配置文件，然后就可以在XML文件或者类中使用同样的语法获取配置文件的信息。比如，引入前面的配置文件：< context:property-placeholder location=“classpath:value.properties,value3.properties” />。由于基于XML的配置就是使用的PropertySourcesPlaceholderConfigurer。因此基于XML的配置和Spring Boot项目一样，如果没有对应的key都会直接启动报错！**

```java
/**
 * Value注入外部文件的值
 *
 * @author lx
 */
@Component
//使用@PropertySources
@PropertySources(value = {
   
        //value3.properties在后面，因此value3将会覆盖value的同名key的值
        @PropertySource(value = "classpath:value.properties", encoding = "UTF-8"),
        @PropertySource(value = "classpath:value3.properties", encoding = "UTF-8")
})
//也可以就使用@PropertySource，同样按照定义顺序加载配置文件
//@PropertySource(value = {"classpath:value3.properties", "classpath:value.properties"})
public class ValuePropertiesCheck {
   

    @Value("${key}")
    private String value;

    //Spring默认情况下，如果配置文件没有对应的key，则将${key}本身将尝试作为值注入

    @Value("${zzz}")
    private String zzz;


    @Override
    public String toString() {
   
        return "ValuePropertiesCheck{" +
                "value='" + value + '\'' +
                ", zzz='" + zzz + '\'' +
                '}';
    }

    /**
     * 添加一个PropertySourcesPlaceholderConfigurer，将会在启动时检查配置文件没有对应的key的情况，并且抛出异常
     * 一定要static修饰的方法，在一开始就执行！
     */
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
   
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

测试：

```java
@Test
public void valuePropertiesCheck() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("valuePropertiesCheck", ValuePropertiesCheck.class));
}
```

启动后将直接抛出异常而停止运行！因为没有key为zzz的属性。

#### 3.9.2.2 指定默认值

上面讲了，通过@PropertySource 引入的配置文件时，如果@Value引用的key不存在，那么默认将整个$ {key}作为值注入，或者通过XML/Spring Boot使用PropertySourcesPlaceholderConfigurer引入配置文件时，如果@Value引用的key不存在，那么直接启动抛出异常。

实际上，我们也可以为key设置默认值，当某个属性的key在配置文件中不存在时，就使用我们指定的默认值。这种情况在不同web服务模块使用同一套后端service业务模块时非常有用。

语法非常简单：${key:defaultValue}，如果只有“:”而不写默认值，那么表示注入空字符串，因此注意类型转换异常。defaultValue可以使用SPEL表达式，实际上这里的defaultvalue只要是字面量部分的值都可以。

如下案例，有一个ValuePropertiesDefault，用于测试默认值：

```java
/**
 * Value注入外部文件的值设置默认值
 *
 * @author lx
 */
@Component
@PropertySource(value = {
   "classpath:value3.properties", "classpath:value.properties"})
public class ValuePropertiesDefault {
   

    @Value("${key:}")
    private String key;

    /**
     * 外部文件是没有zzz这个key的，因此设置默认值，这里默认注入空字符串""
     */
    @Value("${qaz:}")
    private String qaz;

    /**
     * 外部文件是没有zzz这个key的，因此设置默认值，这里默认注入1
     * 如果不写默认值，那么Spring尝试将""转换为int类型，进而会报错
     */
    @Value("${wsx:1}")
    private int wsx;

    /**
     * 可以默认注入
     */
    @Value("${edc:1,2,3,4}")
    private int[] edc;

    //可以使用SPEL表达式


    /**
     * 外部文件有key这里的属性，因此不会使用默认值
     */
    @Value("${key:#{'abc'.concat('def')}}")
    private String concat;


    @Value("${rfv:#{'1,0,1,1'.split(',')}}")
    private double[] rfv;


    @Value("${tgb:#{
   {key1: 'value1', key2: 'value2', key2: 'value3'}}}")
    private Map<String,String> map;

    //………………更多功能


    @Override
    public String toString() {
   
        return "ValuePropertiesDefault{" +
                "key='" + key + '\'' +
                ", qaz='" + qaz + '\'' +
                ", wsx=" + wsx +
                ", edc=" + Arrays.toString(edc) +
                ", rfv=" + Arrays.toString(rfv) +
                ", concat='" + concat + '\'' +
                ", map=" + map +
                '}';
    }

    /**
     * 添加一个PropertySourcesPlaceholderConfigurer，将会在启动时检查配置文件没有对应的key的情况，并且抛出异常
     * 一定要static修饰的方法，在一开始就执行！
     */
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
   
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

测试：

```java
@Test
public void valuePropertiesDefault() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("valuePropertiesDefault", ValuePropertiesDefault.class));
}
```

结果如下：

```java
ValuePropertiesDefault{
   key='value', qaz='', wsx=1, edc=[1, 2, 3, 4], rfv=[1.0, 0.0, 1.0, 1.0], concat='value', map={
   key1=value1, key2=value3}}
```

我们发现，即使没有对应的key，也没有抛出异常，而是使用默认值。

### 3.9.3 注入bean的属性

@Value可以获取其他bean的属性值然后注入到自己的属性中，这实际上也是SPEL的作用，但是这要求尝试获取属性的bean的属性提供相应的getter方法或者属性是public修饰的。对于集合类型，也可以注入集合的索引元素或者map的key的value。

如下案例，有一个ValueOtherBean类，用于测试获取其他bean的属性值：

```java
/**
 * @author lx
 */
@Component
public class ValueOtherBean {
   

    //SPEL获取其他bean的属性值

    /**
     * 字面量
     */
    @Value("#{valueOtherBeanA.property1}")
    private String valueOtherBeanAproperty1;
    @Value("#{valueOtherBeanA.property2}")
    private double valueOtherBeanAproperty2;

    /**
     * 数组/集合
     */
    @Value("#{valueOtherBeanA.property3}")
    private String[] valueOtherBeanAproperty3;

    @Value("#{valueOtherBeanA.property4}")
    private String[] valueOtherBeanAproperty4;

    @Value("#{valueOtherBeanA.property5}")
    private Map<String,Integer> valueOtherBeanAproperty5;

    /**
     * 数组/集合某个索引/key的数据
     */
    @Value("#{valueOtherBeanA.property3[0]}")
    private String valueOtherBeanAproperty3One;
    @Value("#{valueOtherBeanA.property4[1]}")
    private String[] valueOtherBeanAproperty4One;

    @Value("#{valueOtherBeanA.property5[key2]}")
    private Integer valueOtherBeanAproperty5Key3;



    //其他bean
    //想要使用其他bean的属性，要求其他bean提供属性的getter方法或者属性必须是public修饰的

    @Component("valueOtherBeanA")
    public static class ValueOtherBeanA {
   
        /**
         * 一个字符串
         */
        @Value("property1")
        private String property1;
        /**
         * SPEL取一个随机数
         */
        @Value("#{ T(java.lang.Math).random() * 100.0}")
        private double property2;

        @Value("#{'property1,property2,property3'.split(',')}")
        private String[] property3;

        @Value("#{'property1,property2,property3'.split(',')}")
        public List<String> property4;

        @Value("#{
   {key1: '1', key2: '2'}}")
        public Map<String,Integer> property5;

        public String getProperty1() {
   
            return property1;
        }

        public double getProperty2() {
   
            return property2;
        }

        public String[] getProperty3() {
   
            return property3;
        }
    }

    @Override
    public String toString() {
   
        return "ValueOtherBean{" +
                "valueOtherBeanAproperty1='" + valueOtherBeanAproperty1 + '\'' +
                ", valueOtherBeanAproperty2=" + valueOtherBeanAproperty2 +
                ", valueOtherBeanAproperty3=" + Arrays.toString(valueOtherBeanAproperty3) +
                ", valueOtherBeanAproperty4=" + Arrays.toString(valueOtherBeanAproperty4) +
                ", valueOtherBeanAproperty5=" + valueOtherBeanAproperty5 +
                ", valueOtherBeanAproperty3One='" + valueOtherBeanAproperty3One + '\'' +
                ", valueOtherBeanAproperty4One=" + Arrays.toString(valueOtherBeanAproperty4One) +
                ", valueOtherBeanAproperty5Key3=" + valueOtherBeanAproperty5Key3 +
                '}';
    }
}
```

测试：

```java
@Test
public void valueOtherBean() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("valueOtherBean", ValueOtherBean.class));
}
```

结果如下，成功获取并注入：

```java
ValueOtherBean{
   valueOtherBeanAproperty1='property1', valueOtherBeanAproperty2=1.5384076971526883, valueOtherBeanAproperty3=[property1, property2, property3], valueOtherBeanAproperty4=[property1, property2, property3], valueOtherBeanAproperty5={
   key1=1, key2=2}, valueOtherBeanAproperty3One='property1', valueOtherBeanAproperty4One=[property2], valueOtherBeanAproperty5Key3=2}
```

### 3.9.4 注入其他资源

@Value除了注入字面量、外部属性文件、SPEL的表达式计算、SPEL的复杂类型属性注入、SPEL其他bean的属性注入……等等，还可以注入一些特别的属性或者资源。

@Value还可以使用SPEL直接注入操作系统相关资源属性、以及Resource资源。

如下案例，测试直接注入操作系统相关资源属性、以及Resource资源：

```java
/**
 * Value注入资源
 *
 * @author lx
 */
@Component
public class ValueResource {
   

    //SPEL直接注入系统属性

    @Value("#{systemProperties['java.home']}")
    private String javaHome;

    @Value("#{systemProperties['java.version']}")
    private String javaVersion;

    //SPEL注入Resource资源

    /**
     * 注入文件资源
     */
    @Value("classpath:value.properties")
    private Resource classPathResource;

    @Value("G:\\Idea\\springcoreannotation\\src\\main\\resources\\value3.properties")
    private Resource classPathContextResource;

    /**
     * 注入URL资源
     */
    @Value("https://spring.io/")
    private Resource urlResource;


    @Override
    public String toString() {
   
        return "ValueResource{" +
                "javaHome='" + javaHome + '\'' +
                ", javaVersion='" + javaVersion + '\'' +
                ", classPathResource=" + classPathResource +
                ", classPathContextResource=" + classPathContextResource +
                ", urlResource=" + urlResource +
                '}';
    }
}
```

测试：

```java
@Test
public void valueResource() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("valueResource", ValueResource.class));
}
```

结果如下：

```java
ValueResource{
   javaHome='C:\Java\jdk1.8.0_144\jre', javaVersion='1.8.0_144', classPathResource=class path resource [value.properties], classPathContextResource=class path resource [G:/Idea/springcoreannotation/src/main/resources/value3.properties], urlResource=URL [https://spring.io/]}
```

## 3.10 @PostConstruct和@PreDestroy生命周期回调

我们在基于XML的配置部分详细已经介绍过bean的生命周期回调功能，详情看那一部分。@PostConstruct和@PreDestroy是来自JSR-250的bean生命周期注解，在Spring2.5开始同样引入了对这些注解的支持，同样使用CommonAnnotationBeanPostProcessor处理器来解析这两个注解，component-scan默认将CommonAnnotationBeanPostProcessor已经注册了。

@PostConstruct表示初始化bean回调，@PreDestroy表示销毁bean回调。和Resource一样，这三个注解是 JDK 6 到 8 的标准 Java 库的一部分，位于rt.jar包下的java.annotation包中。使用时无需引入任何依赖。但是JDK9及其之后的版本，javax.annotation脱离了Java核心库，想要使用时只能引入maven依赖：javax.annotation-api（artifactId）。

```java
<!-- https://mvnrepository.com/artifact/javax.annotation/javax.annotation-
api -->
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

如下案例，一个BeanCallback类，用于测试生命周期回调：

```java
/**
 * @author lx
 */
@Component
public class BeanCallback {
   

    /**
     * 单例bean
     * 可以进行bean创建回调和bean销毁回调
     */
    @Component("beanCallbackA")
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    public static class BeanCallbackA{
   

        @PostConstruct
        public void postConstruct() {
   
            System.out.println("BeanCallbackA bean创建回调");
        }

        @PreDestroy
        public void preDestroy() {
   
            System.out.println("BeanCallbackA bean销毁回调");
        }

        public BeanCallbackA() {
   
            System.out.println("BeanCallbackA 构造器");
        }

    }


    /**
     * 原型bean
     * 只能进行bean创建回调，不会进行bean销毁回调
     */
    @Component("beanCallbackB")
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public static class BeanCallbackB{
   

        @PostConstruct
        public void postConstruct() {
   
            System.out.println("BeanCallbackB bean创建回调");
        }

        @PreDestroy
        public void preDestroy() {
   
            System.out.println("BeanCallbackB bean销毁回调");
        }

        public BeanCallbackB() {
   
            System.out.println("BeanCallbackB 构造器");
        }

    }
}
```

测试：

```java
@Test
public void beanCallback() {
   
    System.out.println("---创建容器---");
    //新建容器，将会实例化单例的bean，触发bean创建回调
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println("---获取bean实例---");
    //获取bean实例，将会实例化原型的bean，触发bean创建回调
    System.out.println(ac.getBean("beanCallbackA", BeanCallback.BeanCallbackA.class));
    System.out.println(ac.getBean("beanCallbackB", BeanCallback.BeanCallbackB.class));
    //关闭容器，将会销毁bean，触发单例bean的销毁回调
    System.out.println("---容器关闭---");
    ac.close();
}
```

结果如下：

```java
---创建容器---
BeanCallbackA 构造器
BeanCallbackA bean创建回调
---获取bean实例---
com.spring.core.ann.BeanCallback$BeanCallbackA@2aa5fe93
BeanCallbackB 构造器
BeanCallbackB bean创建回调
com.spring.core.ann.BeanCallback$BeanCallbackB@5c1a8622
---容器关闭---
BeanCallbackA bean销毁回调
```

## 3.11 @DependsOn强制顺序

这个注解就类似于XML配置中的depends-on属性，用于控制某一个或者多个bean在该bean实例化之前先顺序实例化，在该bean销毁之后倒序销毁。对于注解标注的bean如果是prototype则DependsOn无效，而对于被依赖的bean即使是prototype的也会顺序初始化，但不会执行销毁回调。更多详情可以看基于XML配置的depends-on部分。

如下案例，有一个BeanDependsOn类，在beanDependsOnA实例化之前，beanDependsOnC和beanDependsOnB顺序实例化，在beanDependsOnA销毁之后，beanDependsOnC和beanDependsOnB倒序销毁。

```java
/**
 * @author lx
 */
@Component
public class BeanDependsOn {
   


    /**
     * 可以指定多一个依赖的bean多个bean将按照顺序先后初始化
     * 销毁时按照倒序销毁
     */
    @DependsOn({
   "beanDependsOnC", "beanDependsOnB"})
    @Component("beanDependsOnA")
    //@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public static class BeanDependsOnA {
   

        @PostConstruct
        public void postConstruct() {
   
            System.out.println("BeanDependsOnA bean创建回调");
        }

        @PreDestroy
        public void preDestroy() {
   
            System.out.println("BeanDependsOnA bean销毁回调");
        }

        public BeanDependsOnA() {
   
            System.out.println("BeanDependsOnA 构造器");
        }

    }

    @Component("beanDependsOnB")
    public static class BeanDependsOnB {
   

        @PostConstruct
        public void postConstruct() {
   
            System.out.println("BeanDependsOnB bean创建回调");
        }

        @PreDestroy
        public void preDestroy() {
   
            System.out.println("BeanDependsOnB bean销毁回调");
        }

        public BeanDependsOnB() {
   
            System.out.println("BeanDependsOnB 构造器");
        }
    }

    @Component("beanDependsOnC")
    //@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public static class BeanDependsOnC {
   

        @PostConstruct
        public void postConstruct() {
   
            System.out.println("BeanDependsOnC bean创建回调");
        }

        @PreDestroy
        public void preDestroy() {
   
            System.out.println("BeanDependsOnC bean销毁回调");
        }

        public BeanDependsOnC() {
   
            System.out.println("BeanDependsOnC 构造器");
        }

    }
}
```

测试：

```java
@Test
public void beanDependsOn() {
   
    System.out.println("---创建容器---");
    //新建容器，将会实例化单例的bean，触发bean创建回调
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    //System.out.println(ac.getBean("beanDependsOnA", BeanDependsOn.BeanDependsOnA.class));
    //关闭容器，将会销毁bean，触发单例bean的销毁回调
    System.out.println("---容器关闭---");
    ac.close();
}
```

结果如下，bean的创建和销毁都保证了顺序（对于singleton的bean）：

```java
---创建容器---
BeanDependsOnC 构造器
BeanDependsOnC bean创建回调
BeanDependsOnB 构造器
BeanDependsOnB bean创建回调
BeanDependsOnA 构造器
BeanDependsOnA bean创建回调
---容器关闭---
BeanDependsOnA bean销毁回调
BeanDependsOnB bean销毁回调
BeanDependsOnC bean销毁回调
```

## 3.12 @Lazy延迟

@Scope注解可以与@Component（@Repository、@Service、@Controller）、@Bean、@Configuration、@Autowired、@Inject等注解使用，@Lazy和XML配置的lazy-init属性效果基本一致，默认为true，详情参考XML的配置文章。

1. 与@Configuration、@Component（@Repository、@Service、@Controller）、@Bean注解连用并且scope=singleton时，这些类或者@Bean方法默认延迟初始化。也可以手动设置@Lazy的value值：true表示延迟初始化，false表示不延迟初始化，但是@Lazy注解只对singleton的bean有效！ 
2. 与@Component（@Repository、@Service、@Controller）、@Configuration注解连用时，类中的所有@Bean方法的bean默认延迟初始化，如果某些@Bean方法上存在@Lazy注解并且value=false，那么表示将会立即初始化该bean，如果该方法是静态方法，那么外部lazy的类不受影响，如果是非静态的，那么外部lazy的类也会跟着初始化！ 
3. 与@Autowired、@Inject注解连用时，它将会创建一个惰性代理对象注入依赖。这个特点可以用来解决循环依赖（但实际上@Autowired本来就可以解决循环依赖，因此没啥大用）。


## 4.1 JSR-330概述

从Spring3.0之开始，Spring提供了对于JSR-330依赖注入标准注解的支持。这些注解和Spring注解一样被扫描和解析。JSR-330 是 Java 的依赖注入标准，定义了一些规范：

1. 一些类型被另一个类型依赖，我们就把这些类型叫做这个类型的“依赖（dependency）”； 
2. 运行时查找依赖的过程，称为“解析（resolving）依赖”； 
3. 如果找不到依赖的实例，称该依赖是”不能满足的 unsatisfied”，这将导致应用运行失败； 
4. 在”依赖注入 dependency injection”机制中，提供所需依赖的工具称为”依赖注入器 dependency injector”。

Java EE中的javax.inject包就对应该标准提供的相关注解，但是并没有定义依赖注入的配置和实现方式，由具有依赖注入功能的框架自己实现依赖注入器，实际上就是注解处理器，Spring框架自然提供了相应的实现。

JSR-330提供了5个注解（Inject、Qualifier、Named、Scope、Singleton）和1个扩展接口（Provider），Spring的spring-context依赖中没有集成JSR-330标准注解，想要使用它，我们必须引入另一个maven依赖：

```java
<!-- https://mvnrepository.com/artifact/javax.inject/javax.inject -->
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

## 4.2. 简单使用

@Named可以用在类上，用来代替Spring的@Component组件。

@Named可以用在参数上，指定注入bean的名称，用来代替Spring的@Qualifier。内部提供的@Qualifier只能用来构建注解。

@Inject可以用在方法、构造器、参数上，用来代替Spring的@Autowired。不能指定限定名，因此常与@Named连用，没有required属性，因此也可以与Optional或@Nullable一起使用。

@Singleton注解，类似于Spring的@Scope(“singleton”)，没有提供其他作用域。内部的@Scope注解只能用来构建注解。

如下案例，一个JSR330Demo类，用于测试JSR-330：

```java
/**
 * @author lx
 */
@Named
public class JSR330Demo {
   
    public JSR330Demo() {
   
        System.out.println("JSR330Demo: "+this);
    }


    private JSR330DemoA getJSR330DemoA;
    @Inject
    public void setJsr330DemoA(@Named("getJSR330DemoA") JSR330DemoA jsr330DemoA) {
   
        this.getJSR330DemoA = jsr330DemoA;
    }

    @Inject
    private JSR330DemoA jsr330DemoA2;

    private JSR330DemoA jsr330DemoA3;


    @Inject
    public JSR330Demo( JSR330DemoA jsr330DemoA3) {
   
        this.jsr330DemoA3 = jsr330DemoA3;
    }


    @Named("JSR330DemoA")
    public static class JSR330DemoA {
   
        public JSR330DemoA() {
   
            System.out.println("JSR330DemoA: "+this);
        }
    }

    @Bean
    public JSR330DemoA getJSR330DemoA(){
   
        JSR330DemoA jsr330DemoA = new JSR330DemoA();
        System.out.println("getJSR330DemoA: "+jsr330DemoA);
        return jsr330DemoA;
    }


    @Override
    public String toString() {
   
        return "JSR330Demo{" +
                "getJSR330DemoA=" + getJSR330DemoA +
                ", jsr330DemoA2=" + jsr330DemoA2 +
                ", jsr330DemoA3=" + jsr330DemoA3 +
                '}';
    }
}
```

测试：

```java
@Test
public void jSR330Demo() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("JSR330Demo", JSR330Demo.class));
}
```

结果如下：

```java
JSR330DemoA: com.spring.core.ann.JSR330Demo$JSR330DemoA@12d3a4e9
JSR330DemoA: com.spring.core.ann.JSR330Demo$JSR330DemoA@2a32de6c
getJSR330DemoA: com.spring.core.ann.JSR330Demo$JSR330DemoA@2a32de6c
JSR330Demo{
   getJSR330DemoA=com.spring.core.ann.JSR330Demo$JSR330DemoA@2a32de6c, jsr330DemoA2=com.spring.core.ann.JSR330Demo$JSR330DemoA@12d3a4e9, jsr330DemoA3=com.spring.core.ann.JSR330Demo$JSR330DemoA@12d3a4e9}
```


## 5.1 Java配置概述

在上面我们开启注解支持的方式是在XML中配置context:component-scan标签，以及扫描包，此后我们就可以使用各种注解来配置此前需要在XML文件中编写的bean的定义。其实，对于这些最基本的配置，比如上面的注解支持和扫描包等等，都可以使用注解来完成。

spring-config.xml文件通常被称为核心配置文件，Java代码来取代XML文件之后，对应的Java类通常被称为Java配置中心（Java-configuration）。使用@Configuration注解来支持Java配置中心的工作，具有@Configuration注解的类也被称为Java配置类。@Configuration就是类似于XML文件中的&lt; beans &gt;标签。

此前我们的@Bean都是和@Component搭配使用，实际上，@Bean更常和@Configuration搭配使用，通常对于外部jar包中的bean我们会在java配置类中使用@Bean的方式交给IoC容器管理。

最简单的@Configuration可能如下所示：

```java
@Configuration
public class AppConfig {
   

    @Bean
    public MyService myService() {
   
        return new MyServiceImpl();
    }
}
```

等价于下面的XML配置：

```java
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

### 5.1.1 @Configuration和@Component的区别

实际上@Configuration和@Component标注的类都可以作为配置类（内部都可以使用@Bean为容器提供对象），但是它们还是有很大的区别。

@Bean在@Component（@Controller、@Service、@Repository）注解标注的类中声明时，它们将在“精简（lite）”模式下处理；而在@Configuration注解标注的类中声明时，它们将在“完整（full）”模式下处理。

@Configuration标注的配置类，将会通过cglib生成代理对象，因此这要求配置类不能是final的类。代理类会对内部被@Bean标注的方法进动态代理， 当一个被@Bean标注的方法调用另一个被@Bean标注的方法时，实际上是调用的代理方法，该方法会直接返回IoC容器中的bean对象，因此在不同@Bean方法中调用另一个被@Bean标注的方法时，将会被重定向道向容器“要”对象，因此将会返回同一个对象。而@Component标注的类不会走cglib来代理@Bean 方法的调用，对于类可以是final的，不同@Bean方法中调用另一个被@Bean标注的方法时，就是相当于直接调用一次方法，因此会返回一个新的对象。为了避免这个问题，通常@Bean和@Configuration一起使用！

但是如果你希望避免任何cglib代理，那么请考虑在非@Configuration类中声明@Bean方法。然后，@Bean方法之间的跨方法调用不会被拦截。

如下案例，有一个ComponentBean和ConfigurationBean类，用于测试二者的区别：

```java
/**
 * 被@Component修饰的类可以是final的
 *
 * @author lx
 */
@Component
public class ComponentBean {
   

    public static class ComponentBeanA {
   
        public ComponentBeanA() {
   
        }
    }

    public static class ComponentBeanB {
   

        private ComponentBeanA componentBeanA;

        public ComponentBeanB(ComponentBeanA componentBeanA) {
   
            this.componentBeanA = componentBeanA;
        }
    }

    @Bean("componentBeanA")
    public ComponentBeanA getComponentBeanA() {
   
        ComponentBeanA componentBeanA = new ComponentBeanA();
        System.out.println("Component-----"+componentBeanA);
        return componentBeanA;
    }

    /**
     * 在该bean方法中尝试调用getComponentBeanA()
     * 将会直接调用该方法获取一个新的对象，因此会打印三次，
     * 它和容器中的componentBeanA不是同一个对象
     */
    @Bean
    public ComponentBeanB getComponentBeanB() {
   
        ComponentBeanA componentBeanA = getComponentBeanA();
        System.out.println("Component-----"+componentBeanA);
        return new ComponentBeanB(componentBeanA);
    }
}

//------------------------------------

/**
 * 被@Configuration修饰的类不可以是final的
 *
 * @author lx
 */
@Configuration
public class ConfigurationBean {
   

    public static class ConfigurationBeanA {
   
        public ConfigurationBeanA() {
   
        }
    }

    public static class ConfigurationBeanB {
   
        private ConfigurationBeanA configurationBeanA;

        public ConfigurationBeanB(ConfigurationBeanA configurationBeanA) {
   
            this.configurationBeanA = configurationBeanA;
        }
    }

    @Bean("configurationBeanA")
    public ConfigurationBeanA getConfigurationBeanA() {
   
        ConfigurationBeanA configurationBeanA = new ConfigurationBeanA();
        System.out.println("Configuration-----" + configurationBeanA);
        return configurationBeanA;
    }

    /**
     * 在该bean方法中尝试调用getConfigurationBeanA()
     * 将会走代理方法获取容器中的对象，因此会打印两次，
     * 它们是同一个容器中的对象
     */
    @Bean
    public ConfigurationBeanB getConfigurationBeanB() {
   
        ConfigurationBeanA configurationBeanA = getConfigurationBeanA();
        System.out.println("Configuration-----" + configurationBeanA);
        return new ConfigurationBeanB(configurationBeanA);
    }
}
```

测试：

```java
@Test
public void componentAndConfigurationBean() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println("--------------------");
    System.out.println(ac.getBean("componentBean", ComponentBean.class));
    System.out.println(ac.getBean("componentBeanA", ComponentBean.ComponentBeanA.class));
    //输出configurationBean，在控制台可以明显看出来是一个CGLIB代理类对象
    System.out.println(ac.getBean("configurationBean", ConfigurationBean.class));
    System.out.println(ac.getBean("configurationBeanA", ConfigurationBean.ConfigurationBeanA.class));
}
```

结果如下：

```java
Component-----com.spring.core.ann.ComponentBean$ComponentBeanA@c267ef4
Component-----com.spring.core.ann.ComponentBean$ComponentBeanA@30ee2816
Component-----com.spring.core.ann.ComponentBean$ComponentBeanA@30ee2816
Configuration-----com.spring.core.ann.ConfigurationBean$ConfigurationBeanA@7ed7259e

Configuration-----com.spring.core.ann.ConfigurationBean$ConfigurationBeanA@7ed7259e
--------------------
com.spring.core.ann.ComponentBean@5fdcaa40
com.spring.core.ann.ComponentBean$ComponentBeanA@c267ef4
com.spring.core.ann.ConfigurationBean$$EnhancerBySpringCGLIB$$4b2d6e0a@6dc17b83
com.spring.core.ann.ConfigurationBean$ConfigurationBeanA@7ed7259e
```

## 5.2 基于配置类实例化容器

下面了我们来看如何基于Java配置类来启动和实例化IoC容器！

如果我们使用Jasva配置类来代替XML文件，那么我们的IoC容器就不能是ClassPathXmlApplicationContext，而是使用在Spring3.0的时候推出的AnnotationConfigApplicationContext。它是ApplicationContext的一个实现，他可以接收@Configuration， @Component ，和使用JSR-330元注解的类作为输入。

当@Configuration标注的类作为输入提供时，@Configuration类本身将被注册为一个 bean ，并且类内声明的 @Bean 方法也被注册成为一个 bean。当@Component和JSR-330类作为输入时，它们同样被注册为一个bean。在这些bean的所在类中，并可以使用诸如@Autowired或@Inject之类的注解用于依赖注入。

使用起来很简单，类似于XML，不过只需要传递@Configuration、@Component（@Controller、@Service、@Repository）、JSR-330注解注解标注的类的class即可：

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(JSR330Demo.class,ConfigurationBean.class,ComponentBean.class);
```

当然也可以初始化无参容器，然后使用register(Class&lt;?&gt;…)方法对其进行配置：

```java
AnnotationConfigApplicationContext ac2 = new AnnotationConfigApplicationContext();
ac2.register(JSR330Demo.class,ConfigurationBean.class,ComponentBean.class);
ac2.refresh();
```

使用AnnotationConfigApplicationContext启动时，此前的JSR330Demo将报错，需要注释掉有参构造器。开发中不建议使用JSR-330注解，Spring提供的注解更加强大。

上面的做法还是太麻烦了，如果有很多这样的类，那怎么办，难道还要一个一个的写吗？自然不需要，Spring也提供了扫描包的注解@ComponentScan。

```java
@Configuration
@ComponentScan("com.spring.core.*")
public class JavaConfig {
   }
```

这就等于XML中的component-scan的功能：

```java
<context:component-scan base-package="com.spring.core.*"/>
```

此时，我们只需要传递JavaConfig.class就行了：

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(JavaConfig.class);
```

同样，AnnotationConfigApplicationContext 还提供了scan(String…)方法和component-scan配置、@ComponentScan注解一样的功能：

```java
AnnotationConfigApplicationContext ac2 = new AnnotationConfigApplicationContext();
ac2.scan("com.spring.core.*");
ac2.refresh();
```

AnnotationConfigWebApplicationContext是专门为web开发提供的ApplicationContext。当你配置ContextLoaderListener，或者DispatcherServlet时，可以使用它，也可以是不使用。


Spring Boot项目中，我们通常也是使用注解配置包扫描！@ComponentScan将会扫描指定包下面的任何基于@Component元注解或者JSR-330注解的类，这些类将被注入到Spring容器中。另外，@Configuration也是基于@Component元注解的： ![img](https://img-blog.csdnimg.cn/2020090814040892.png#pic_center) 因此，如果@Configuration类也处于扫描包的范围之类，那么@Configuration类也会被作为一个bean注入到Spring容器中，因此也支持使用@Value、@Autowired等注解，而它内部的所有@Bean方法也都被处理并注册到为容器中（记住@Configuration不同@Bean方法之间的调用返回的是同一个容器中的对象）。

## 5.3 @Bean注解的详细使用

此前我们就简单介绍过@Bean注解，@Bean是方法级的注解，和XML中的&lt; bean &gt;标签有同样的作用。现在来看看它的一些属性和其他用法。

### 5.3.1 生命周期回调

@Bean注解定义的任何类中都支持常规的生命周期回调，并且可以使用jsr-250中的@PostConstruct和@PreDestroy注解。

@Bean注解定义的任何类中还支持Spring的回调。如果bean实现了InitialingBean、DisposableBean或Lifecycle接口，那么容器将会在正确的时间调用它们各自的回调方法。

上面的操作此前就讲过了，现在我们要讲的是@Bean的initMethod和destroyMethod特有属性，实际上也是回调方法的定义，这类似于基于XML配置的init-method和destroy-method属性。

如下案例，用于测试@Bean的initMethod和destroyMethod属性：

```java
public class JavaConfigOne {
   

    public JavaConfigOne() {
   
        System.out.println("JavaConfigOne constructor");
    }

    public void init() {
   
        System.out.println("JavaConfigOne init");
    }

    public void destroy() {
   
        System.out.println("JavaConfigOne destroy");
    }
}
/**
 * @author lx
 */
@Configuration()
@ComponentScan("com.spring.core.*")
public class JavaConfig {
   

    /**
     * 回调
     */
    @Bean(initMethod = "init",destroyMethod = "destroy")
    public JavaConfigOne javaConfigOne(){
   
        return new JavaConfigOne();
    }
}
```

测试：

```java
@Test
public void javaConfigBeanCallback() {
   
    AnnotationConfigApplicationContext ac2 = new AnnotationConfigApplicationContext(JavaConfig.class);
    ac2.close();
}
```

结果如下：

```java
JavaConfigOne constructor
JavaConfigOne init
JavaConfigOne destroy
```

### 5.3.2 bean的名字和描述

默认情况下，使用@Bean标注的方法的名称作为结果bean的名字。但是可以使用name属性覆盖，另外也可以传递一个字符串数组，提供多个名字， bean名字重复不会报错，但是会发生意想不到的问题，比如通过某个名字取到的确实另外的bean实例，一定要主动避免不同@Bean的名字重复。

@description注解用于向@Bean（包括@Component）添加文档描述，在jmx中更可能会用到。

```java
@Bean
@Description("bean name 默认方法名")
public JavaConfigOne getJavaConfigOne() {
   
    return new JavaConfigOne();
}

/**
 * bean name
 */
@Bean(name = {
   "javaConfigOne2", "javaConfigOne3"})
@Description("指定bean name")
public JavaConfigOne getJavaConfigOne2() {
   
    return new JavaConfigOne();
}
```

### 5.3.3 bean自动注入

@Bean的autowire属性用于确定是否自动注入，即 bean 是否由 Spring 容器使用 setter 自动注入其依赖项，或者指定byName还是byType。很明显，这里的autowire属性和XML配置中的autowire属性有一些区别，可选配置更少！

@Bean的autowire属性默认为NO，即不自动注入。可选BY_NAME，根据name注入，根据set方法的“set”后面的字符串去匹配bean name，没有匹配到就不注入，或者BY_TYPE，通过setter方法的参数类型自动注入，如果有多个同类型的bean，则抛出异常！

**无论是BY_NAME还是BY_TYPE，匹配到多个方法就会将这些方法全部执行。自 Spring 5.1 开始，已经使用@Bean工厂方法参数制动注入和@Autowired注入取代基于名称/类型的 autowire属性自动注入！**

在JavaConfig中加入如下配置：

```java
@Bean(autowire = Autowire.BY_NAME)
@Description("ByName自动注入")
    public JavaConfigAutowire1 javaConfigAutowireByName() {
   
        return new JavaConfigAutowire1();
    }


//    @Bean(autowire = Autowire.BY_TYPE)
//    @Description("ByType自动注入，多个相同类型的bean将抛出异常")
//    public JavaConfigAutowire1 javaConfigAutowireByType() {
   
//        return new JavaConfigAutowire1();
//    }


    @Bean
    public JavaConfigAutowire2 javaConfigAutowire2() {
   
        return new JavaConfigAutowire2();
    }
    @Bean
    public JavaConfigAutowire2 javaConfigAutowire221() {
   
        return new JavaConfigAutowire2();
    }



    public static class JavaConfigAutowire1 {
   
        private JavaConfigAutowire2 javaConfigAutowire2;

        //无论是BY_NAME还是BY_TYPE，匹配到多个方法就会将这些方法全部执行
        //该属性自Spring 5.1开始已经废弃

        public void setJavaConfigAutowire2(JavaConfigAutowire2 xxx) {
   
            System.out.println("setJavaConfigAutowire2");
            this.javaConfigAutowire2 = xxx;
        }

        public void setJavaConfigAutowire221(Object xxx) {
   
            System.out.println("setJavaConfigAutowire221");
            this.javaConfigAutowire2 = (JavaConfigAutowire2) xxx;
        }

        @Override
        public String toString() {
   
            return "JavaConfigAutowire1{" +
                    "javaConfigAutowire2=" + javaConfigAutowire2 +
                    '}';
        }
    }

    public static class JavaConfigAutowire2 {
   

    }
```

结果如下：

```java
setJavaConfigAutowire2
setJavaConfigAutowire221
JavaConfigAutowire1{
   javaConfigAutowire2=com.spring.core.ann.JavaConfig$JavaConfigAutowire2@6580cfdd}
com.spring.core.ann.JavaConfig$JavaConfigAutowire2@7e0b85f9
com.spring.core.ann.JavaConfig$JavaConfigAutowire2@6580cfdd
```

## 5.4 @Import和@ImportResource

与在XML的配置中使用&lt; import &gt;元素来帮助模块化配置一样，@Import注解允许从另一个配置类加载@Bean定义以及其他配置信息，即引入其他配置类。这样在实例化容器时，不需要显示的引入其他的配置类了，实际上@ComponentScan也有这个功能，因为其他配置类也是一个bean，如果其他配置类不在包扫描的范围里面，那么就可以使用@Import注解了。被引入的其他配置类上，可以不用写@Configuration注解。当然，写上也没问题。

@ImportResource则用于导入Spring的配置文件，让配置文件里面的内容生效，当某些配置写在外部XML中而容器使用注解的容器时，就可以使用@ImportResource将外部配置导入进来；Spring Boot里面没有Spring的XML配置文件，我们自己编写的配置文件，也不能自动识别；想让Spring的配置文件生效，就需要@ImportResource将其他配置文件加载进来。


本次我们介绍了Spring 5.x的IoC核心容器的基于注解和Java配置的基本用法，回想上文中的基于XML的配置，这样看起来基于注解的配置却是简单得多。

回到最开始的问题，即注解配置是否比XML配置"更好"。从目前的Spring发展趋势来看，似乎注解配置更占上峰。

根据我本人的项目经验，对于传统的SSM项目，一般我们自己写的业务类，比如Controller、Service、Mapper等模块的类都是通过注解方式配置的，这样可以让我们省去很多&lt; bean&gt;的配置，而对于Spring及其相关组件的核心配置，比如component-scan扫描包路径（用来支持相关注解）、数据库连接、mvc相关组件（annotation-driven、视图解析器、资源处理）、文件上传处理等等都是采用XML配置的，对于外部组件的集成，比如redis、solr等等也都是有专门的XML配置文件。

后来到了Spring Boot项目，那就更简单了。此前的很多组件的配置文件都不用写了，比如mvc的配置、redis的配置……，因为Spring Boot帮我们做了。我们只需要将一些硬编码的配置信息放在指定的properties/yml文件中以指定的key保存，Spring Boot即可进行自动配置。就算是我们需要自己配置，也是大多采用本次讲的Java Config读取properties中的配置然后使用@Bean注解的形式，不再是XML配置文件的形式了。

本文只是开始，仅仅是单体应用，只是最基础的IoC部分。只有经历过SSM中的繁琐的组件配置，才能明白Spring Boot搭建一个企业级项目到底有多轻松，这些东西我们会在后面的文章中一一见识到！

**相关文章**

1.  [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html) 
2.  [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html) 
3. https://spring.io/

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

