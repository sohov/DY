

>详细介绍了Spring入门案例的搭建，以及基于XML的核心IoC机制的配置和使用。






**基于最新Spring 5.2.8，介绍Spring核心 IoC机制，以及基于XML的IoC机制的核心配置方式，包括bean的配置、依赖项注入配置，以及各种常用配置信息。没有讲过多源码，提供了大量的案例，对于会使用Spring的人来说可能比较啰嗦，但是比较适合Spring初学者！**

关于Spring的介绍： [Spring与Spring Framework的概述](https://blog.csdn.net/weixin_43767015/article/details/108379777)。





Spring第一例，我们来学习如何通过maven来搭建Spring项目，并且从通过IoC机制从IoC容器中获取对象（ [maven学习资料](https://download.csdn.net/download/weixin_43767015/12788525)）。首先创建一个空的maven项目： ![img](https://img-blog.csdnimg.cn/20200902230326783.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) ![img](https://img-blog.csdnimg.cn/20200902230339444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) 到此空的maven项目创建完毕！ ![img](https://img-blog.csdnimg.cn/20200902230403576.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) 我们在pom.xml中添加Spring核心模块功能的坐标依赖：

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




![img](https://img-blog.csdnimg.cn/20200902230453286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) 只需要这一个依赖即可，实际上它会将其他的核心依赖都一起引入进来的： ![img](https://img-blog.csdnimg.cn/20200902230514672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) 可以看到这就是Spring的Core technologies模块的核心功能依赖了。然后添加一个Bean类。 ![img](https://img-blog.csdnimg.cn/20200902230534513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

```java
/**
 * @author lx
 */
public class HelloSpring {
   
    private String hello = "hello";

    public void say() {
   
        System.out.println("hello");
    }

    public String getHello() {
   
        return hello;
    }

    public void setHello(String hello) {
   
        this.hello = hello;
    }

    public HelloSpring(String hello) {
   
        this.hello = hello;
        System.out.println("初始化");
    }

    public HelloSpring() {
   
        System.out.println("初始化");
    }
}
```


**在resources目录下面新增一个配置文件，命名建议spring-config.xml或者applicationContext.xml，这就是Spring的核心配置文件！** ![img](https://img-blog.csdnimg.cn/20200902230638854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) 然后，就是重头戏了，我们在配置文件里加入一个&lt; bean /&gt;标签。

```java
<!--加入bean标签-->
<bean id="helloSpring" class="com.spring.core.HelloSpring"/>
```



![img](https://img-blog.csdnimg.cn/20200902230719219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center) 最后加入一个测试类： ![img](https://img-blog.csdnimg.cn/20200902230749630.png#pic_center) 编写我们的代码：

```java
/**
 * @author lx
 */
public class SpringCoreFirst {
   
    public static void main(String[] args) {
   
        //通过配置文件创建容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
        //向容器要对象 返回的是object类型,可以强转
        HelloSpring helloSpring = (HelloSpring) ac.getBean("helloSpring");
        //此方法直接返回对应的类型的对象,只有一个该类型对象的话可以省略前面的name或id
        HelloSpring helloSpring1 = ac.getBean("helloSpring", HelloSpring.class);
        System.out.println(helloSpring);
        System.out.println(helloSpring1);
        System.out.println(helloSpring.getHello());
        System.out.println(helloSpring1.getHello());
    }
}
```

**运行之后，有如下输出：**

```java
初始化
com.spring.core.HelloSpring@754ba872
com.spring.core.HelloSpring@754ba872
hello
hello
```

**到此我们的第一个Spring最简单的IoC案例搭建、运行完毕！**


**Spring的核心机制就是Inversion of Control (IoC)，中文译名“控制反转”。控制反转就是将对象创建的方式、属性设置方式反转，以前是开发人员自己通过new控制对象的创建，自己为对象属性赋值。使用Spring之后，将对象以及属性的创建和管理交给了Spring，由Spring来负责控制对象的生命周期、属性控制以及和其他对象间的关系，达到类与类之间的解耦功能，同时还能实现类实例的复用。**


**dependency injection (DI)，中文译名依赖注入。在Spring官方文档中有这样一句话：“IoC is also known as dependency injection (DI)”，即IoC也被称为DI。DI是Martin Fowler 在2004年初的一篇论文中首次提出的，用于具体描述一个对象获得依赖对象的方式，不是自己主动查找和设置（比如new、set），而是被动的通过IoC容器注入（设置）进来。** ![img](https://img-blog.csdnimg.cn/20200902231703645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

**Spring中管理对象的容器称为IoC容器，IoC容器负责实例化、配置和组装bean。** org.Springframework.beans和org.Springframework.context包是Springframework的IoC容器的基础。

**IoC是一个抽象的概念，具体到Spring中就是以代码的形式实现的，Spring提供了许多IoC容器的实现，其核心是BeanFactory接口以及它的实现类。** BeanFactory接口可以理解为IoC容器的抽象，提供了IoC容器的最基本的功能，比如对单个bean的获取、对bean的作用域判断、获取bean类型、获取bean别名等等功能。BeanFactory直译过来就是Bean工厂，实际上IoC容器中bean的获取就是一种典型的工厂模式，里面的Bean常常就是单例的（当然也可以是其它类型的）。简单的说，IoC容器可以理解为一个大的Map，我们通过配置的id或者name或者其他唯一标识就可以从容器中获取到对应的对象（当然实际上没这么简单，后面讲源码的时候会仔细分析，但是肯定是用到了Map作为容器的）！

**BeanFactory仅仅作为IoC容器的超级接口，但是真正可用的容器实现却不是它，而是它的一系列子类。BeanFactory有两个主要的容器实现：DefaultListableBeanFactory（类）和ApplicationContext（接口）。这里没有讲解源码，后面会讲到，现在不必过于深究！**

## 2.1 DefaultListableBeanFactory类

**DefaultListableBeanFactory是IoC容器的一种真正实现，也是原始的默认实现，通常作为自定义BeanFactory的父类。它通过Resource加载Spring的xml配置信息，通过XmlBeanDefinitionReader解析配置xml文件，随后Bean信息会被存储到IoC容器中，启动IOC容器就可以使用getBean方法从IOC容器中获取bean对象。**

**DefaultListableBeanFactory加载Bean对象的方式被称为“消极/懒加载”，消极加载在启动IoC容器的时只是将配置文件中的配置信息加载进容器，而不会创建容器中所配置的bean对象，当使用getBean方法访问对象的时候才创建所需的对象。**

**DefaultListableBeanFactory的使用如下（基于第一个Spring项目）：**

```java
@Test
public void test() {
   
    //懒加载
    Resource res = new ClassPathResource("spring-config.xml");
    DefaultListableBeanFactory defaultListableBeanFactory = new DefaultListableBeanFactory();
    XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(defaultListableBeanFactory);
    xmlBeanDefinitionReader.loadBeanDefinitions(res);
    //将这两行getBean代码注释掉再运行，IoC容器将不会真正的创建对象
    HelloSpring helloSpring = defaultListableBeanFactory.getBean("helloSpring", HelloSpring.class);
    HelloSpring helloSpring2 = defaultListableBeanFactory.getBean("helloSpring", HelloSpring.class);
    System.out.println(helloSpring);
    System.out.println(helloSpring2);
}
```

Spring中还有一个XmlBeanFactory容器，继承了DefaultListableBeanFactory，实际上XmlBeanFactory就是对DefaultListableBeanFactory和XmlBeanDefinitionReader的封装调用而已，从Spring 3.1开始就已被废弃。XmlBeanFactory的使用如下：

```java
@Test
public void test2() {
   
    //懒加载
    Resource res = new ClassPathResource("spring-config.xml");
    BeanFactory ioc = new XmlBeanFactory(res);
    HelloSpring helloSpring = ioc.getBean("helloSpring", HelloSpring.class);
    HelloSpring helloSpring1 = ioc.getBean("helloSpring", HelloSpring.class);
    System.out.println(helloSpring);
    System.out.println(helloSpring1);
}
```

DefaultListableBeanFactory和XmlBeanFactory都是是适用于单体应用的IoC容器，而且用的比较少。

## 2.2 ApplicationContext 接口

org.Springframework.context.ApplicationContext接口是org.Springframework.beans.factory.BeanFactory的子接口，它继承了BeanFactory的全部功能，负责实例化、配置和组装bean，同时添加了与Spring AOP集成、消息资源处理（用于国际化）、事件发布、应用层特定的上下文WebApplicationContext等等新功能。

**ApplicationContext通过读取配置元数据获取关于要实例化、配置和组装的对象的指令。配置元数据以XML、Java注释或Java代码来表示，它定义了组成应用程序的对象以及这些对象之间的丰富依赖关系，实际上我们案例中的xml中的bean配置方式就是一种配置原数据。**

**由于ApplicationContext包含BeanFactory的所有功能，并且还包含更多的功能，现在，Spring推荐我们使用ApplicationContext的实现类来作为IoC容器，ApplicationContext提供了很多IoC容器实现。**

ClassPathXmlApplicationContext主要是从类路径去加载配置文件，也支持从文件路径加载配置文件，这就是Spring案例中的方法。

FileSystemXMLApplicationContext主要是从文件路径去加载配置文件，也支持读取类路径下的配置文件，它们都是适用于单体应用的IoC容器。还有一个XmlWebApplicationContext，它是专门为web开发所准备的，用于通过监听器启动并加载web根目录下的配置文件信息，这一个IoC容器也是我们后面源码学习的重点。

**ApplicationContext采用的是非消极加载，也就就是说在IoC容器启动的时候就将配置的所有默认对象都创建起来并保存在容器中。并且默认的懒加载和饿加载的对象都是单例的，构造方法只被调用一次！**


如同我们的案例一样，当我们配置好需要交给IoC管理的对象，然后启动IoC容器，此时我们就可以直接从IoC容器中获取我们想要的对象了。**其中，交给IoC容器管理的对象被称为bean，或者说bean是一个由SpringIOC容器实例化、组装和管理的对象。**

**Spring官方文档将我们定义的bean以及它们的依赖关系信息称为配置元数据（configuration metadata），配置元数据可以以XML、Java注释或Java代码来表示，它定义了组成应用程序的对象以及这些对象之间的丰富依赖关系。**

1. 基于XML的配置：最原始的Spring配置方式，现正在被取代！ 
2. 基于注解的配置：Spring 2.5引入了对基于注解的配置元数据的支持。比如@Autowired、@PostConstruct、@PreDestroy方法。 
3. 基于Java的配置：从Spring 3开始，Spring JavaConfig项目提供的许多特性成为核心Spring框架的一部分。因此，可以使用Java而不是XML文件来定义应用程序的bean。比如@Configuration、@Bean、@Import和@DependsOn注解。

**实际上，我们定义的配置元数据最开始被加载到Spring中之后会被转变为BeanDefinition，BeanDefinition会记录下所有解析到的bean定义，将解析结果保存起来的好处就是此后就不至于每次用到配置信息的时候都去解析一遍配置元数据。**

BeanDefinition中包含以下数据:

1. 包限定类名：通常是要定义的 bean 的实际实现类。 
2. Bean所属的包的全限定类名：表示bean的实际类型。 
3. Bean行为的配置元素：用于说明bean在容器中的作用范围、生命周期回调函数等。 
4. 对 Bean完成其工作所需的其他豆类的引用。这些引用也称为协作者或依赖项。 
5. 要在新创建的对象中设置的其他配置设置。例如，用于管理连接池的 bean中的连接数的大小限制。 
6. ……其他属性。

**配置元数据信息被记录到BeanDefinition容器中之后，随即启动IoC容器，随后通过IoC容器才会真正的开始依赖注入的过程，即bean的初始化，以及随后的将依赖关系（依赖的bean）注入到bean中，这里实际上涉及到源码原理，后面会专门讲，现在不必过于深究！**

**现在我们以ApplicationContext为容器，以最基础的XML配置的方式来介绍怎么定义配置元数据！随后的而文章中我们会介绍使用注解和Java代码的方式怎么定义配置元数据！**


基于XML的元数据配置是最原始的一种配置方式，到今天看起来有点过时了，但是这对于我们后面学习基于注解和Java代码方式的元数据配置有一定帮助！

基于XML的元数据配置必定离不开XML标签，Spring为bean的配置提供了多个标签和多个属性可以选择。

**< beans />标签通常作为< bean />和其他标签的容器，并且作为文档中的根元素。< beans />标签可以为内部的全部< bean />标签提供默认值，< beans />标签也能够嵌套使用，用来为部分< bean />标签提供默认值。在XML中，使用< bean />标签表示将一个bean交给IoC容器管理。另外还有一个< alias >标签为< bean />指定别名。**

## 4.1 多个XML配置文件

通常，一个企业级的项目的不同模块可能会使用不同的XML配置文件，这有利于分清楚每个模块自己的配置。

**我们可以在使用ApplicationContext的时候，在构造函数中传递多个XML文件地址参数来将这些文件都读取。当然如果不想使用构造函数传递多个xml文件地址，我们可以使用一个或多个< import >标签将其他XML配置文件的信息导入到一个配置文件中。**

例如我们现在增加一个名为spring-config2.xml配置文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/Spring-beans.xsd">

        <bean name="helloSpring2" class="com.spring.core.HelloSpring"/>
</beans>
```

然后我们使用构造器传递两个资源的路径，即可解析到这两个配置文件：

```java
@Test
public void constructor() {
   
    //构造函数传递多个参数
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml","spring-config2.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
}
```

结果如下：

```java
初始化
初始化
[helloSpring, helloSpring2]
```

或者在spring-config.xml文件中使用&lt; import resource=“spring-config2.xml”/&gt;来引入另一个配置文件，随后的构造函数中我们只需要传递spring-config.xml文件路径即可解析到这两个配置文件：

```java
<import resource="spring-config2.xml"/>
```

```java
@Test
public void imports() {
   
    //构造函数传递1个参数
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
}
```

结果如下：

```java
初始化
初始化
[helloSpring, helloSpring2]
```

## 4.2 命名Bean

**在IoC容器中，< bean />标签的name和id属性可以为bean命名，通过名字可以找容器获取这个bean的实例！id是全局唯一的，name可以有多个，可以通过英文逗号，分号或者空格来区分多个name。同一个bean的id和name可以一样，不同的bean的name和id都不能一样。**

```java
<bean id="helloSpring" name="helloSpring helloSpring3" class="com.spring.core.HelloSpring"/>
```

**当没有指定name或者id时，IoC容器会为Bean自动分配一个唯一名字，但是如果希望依靠ref注入bean，那么仍然需要命名。**

**对于XML装配的bean（使用< bean />标签），IoC的默认命名规则是：** 如果没有指定id或者name，那么将使用“类的全路径名#0”、“类的全路径名#1”、“类的全路径名#2”……的方式来为bean命名，有n个没有命名的同类型bean，那么名字后面就是从[0,n-1]类似于索引递增的进行命名，如果中途遇到同名bean，那么跳过这个索引，使用下一个。

另外，< alias/>标签还可以为bean定义别名，别名和bean的名称具有同样的效果！

### 4.2.1 测试

加入如下配置：

```java
<bean name="com.spring.core.HelloSpring#1 helloSpring " class="com.spring.core.HelloSpring"/>
<!--加入两个未命名的Bean-->
<bean class="com.spring.core.HelloSpring"/>
<bean class="com.spring.core.HelloSpring"/>
<!--别名-->
<alias name="helloSpring" alias="helloSpring2"/>
```

测试：

```java
@Test
public void idName() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    HelloSpring helloSpring0 = ac.getBean("com.spring.core.HelloSpring#1", HelloSpring.class);
    HelloSpring helloSpring1 = ac.getBean("helloSpring", HelloSpring.class);
    HelloSpring helloSpring4 = ac.getBean("helloSpring2", HelloSpring.class);
    HelloSpring helloSpring2 = ac.getBean("com.spring.core.HelloSpring#0", HelloSpring.class);
    HelloSpring helloSpring3 = ac.getBean("com.spring.core.HelloSpring#2", HelloSpring.class);
    System.out.println(helloSpring0);
    System.out.println(helloSpring1);
    System.out.println(helloSpring4);
    System.out.println(helloSpring2);
    System.out.println(helloSpring3);

    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
}
```

结果如下，都能够获取成功：

```java
初始化
初始化
初始化
HelloSpring{
   hello='hello'}
HelloSpring{
   hello='hello'}
HelloSpring{
   hello='hello'}
HelloSpring{
   hello='hello'}
HelloSpring{
   hello='hello'}
[com.spring.core.HelloSpring#1, com.spring.core.HelloSpring#0, com.spring.core.HelloSpring#2]
```

## 4.3 实例化Bean

**在XML文件中，< bean />标签不是一个对象，其本质上是用来描述创建一个或多个对象的方式。当需要创建Bean的时候，容器会查看bean的创建方式，并使用由该bean定义封装的配置元数据来创建实际对象。**

使用XML配置的bean，通常至少需要指定class属性（实例工厂方法和parent继承&lt; bean /&gt;除外），这个class属性（在内部对应BeanDefinition实例上的Class属性）表示IoC容器实例化的对象的类型。

**IoC容器在帮我们实例化bean对象的方法包括构造器、静态工厂、实例工厂三种，我们可以指定实例化方式。**

### 4.3.1 构造函数实例化

**大多数情况下，比如我们前面写的的所有案例。IoC容器使用构造器来创建bean实例，底层是基于反射的机制。因此我们必须提供无参构造器或者对应的参数的构造器，否则将抛出异常！**

我们将HelloSpring的无参构造器注释掉，然后运行上面随便一个案例，都会抛出下面的异常（针对无参构造器）。

```java
No default constructor found; nested exception is java.lang.NoSuchMethodException: com.spring.core.HelloSpring.<init>()
```

意思很明显就是找不到无参构造方法！

#### 4.3.1.1 内部类实例化

有时候我们想要在XML中配置一个类的内部类，那么当然可以，我们只需要在class中指明内部类的全路径类名就行了，和外部类路径一样的格式，当然我们也可以使用$将内部类名与外部类名分开（如果嵌套超过两层的内部类，那么就必须使用 $。另外，对于type属性，则同样需要使用 $）。

需要注意的是，对于非静态内部类，它的构造器实际上是需要依赖注入一个外部类对象（在反编译之后就能看见），即它的构造器不是无参构造器，因此我们需要使用constructor-arg注入一个外部类的bean，这样才不会报错！而静态内部类而不需要依赖外部类对象！

我们在HelloSpring中添加两个内部类：

```java
public static class StaticInnerClass {
   
    public StaticInnerClass() {
   
        System.out.println("静态内部类初始化");
    }
}

public class InnerClass {
   
    public InnerClass() {
   
        System.out.println("内部类初始化");
    }
}
```

然后在配置文件中：

```java
<!--静态内部类的初始化，和外部类一样的。 使用.或者$将内部类名与外部类名分开都行-->
<bean class="com.spring.core.HelloSpring.StaticInnerClass"/>
<bean class="com.spring.core.HelloSpring.StaticInnerClass"/>

<bean class="com.spring.core.HelloSpring$StaticInnerClass"/>
<bean class="com.spring.core.HelloSpring$StaticInnerClass"/>


<!--非静态内部类的初始化需要依赖外部类对象-->
<bean name="helloSpring " class="com.spring.core.HelloSpring"/>

<bean class="com.spring.core.HelloSpring.InnerClass">
    <!--属性依赖注入，后面会讲-->
    <constructor-arg ref="helloSpring"/>
</bean>
<bean class="com.spring.core.HelloSpring$InnerClass">
    <!--属性依赖注入，后面会讲-->
    <constructor-arg ref="helloSpring"/>
</bean>
```

测试：

```java
@Test
public void inner() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
}
```

结果如下，成功实例化：

```java
初始化
静态内部类初始化
静态内部类初始化
静态内部类初始化
静态内部类初始化
内部类初始化
内部类初始化
[helloSpring, com.spring.core.HelloSpring.StaticInnerClass#0, com.spring.core.HelloSpring.StaticInnerClass#1, com.spring.core.HelloSpring$StaticInnerClass#0, com.spring.core.HelloSpring$StaticInnerClass#1, com.spring.core.HelloSpring.InnerClass#0, com.spring.core.HelloSpring$InnerClass#0]
```

### 4.3.2 静态工厂方法实例化

**实际上就是工厂模式的应用，这种方式实际上是我们自己创建对象，但是由Spring调用，通过Spring标签获取！但是也只是调用一次创建方法，将创建的对象存入容器，后续同样只是取出（单例情况下）！注意创建对象的静态工厂并不会实例化！**

**使用静态工厂方法实例化bean时，在< bean />标签中的class属性不再是要获取的bean的全路径类名，而是静态工厂的全路径类名，同时使用名为factory-method的属性指定获取bean对象的工厂方法的名称（注意该方法必须是静态方法）。**

首先我们要有一个静态工厂类，提供一个静态方法用于获取bean对象：

```java
/**
 * @author lx
 */
public class HelloSpringStaticFactory {
   

    private static HelloSpring helloSpring = new HelloSpring();

    /**
     * 静态工厂方法
     *
     * @return 返回HelloSpring实例
     */
    public static HelloSpring getHelloSpring() {
   
        System.out.println("静态工厂方法");
        return helloSpring;
    }

    public HelloSpringStaticFactory() {
   
        System.out.println("静态工厂不会初始化");
    }
}
```

随后就是配置文件的编写:

```java
<!--class表示静态工厂的全路径类名-->
<!--factory-method表示静态工厂方法-->
<bean name="helloSpring" class="com.spring.core.HelloSpringStaticFactory" factory-method="getHelloSpring"/>
```

测试：

```java
@Test
public void staticMethod() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("helloSpring", HelloSpring.class));
}
```

结果如下，成功实例化：

```java
静态工厂方法
初始化
[helloSpring]
com.spring.core.HelloSpring@45820e51
```

**很明显，静态工厂方法违背了Spring的初衷，因为我们还是要编写代码new对象，但是适用于那种需要对一个集合进行实例化的情况，因为集合的实例化如果使用配置文件编写的话，那也挺麻烦的。**

### 4.3.3 实例工厂方法实例化

**实例工厂本身要实例化工厂类，随后从工厂实例的非静态方法中调用方法获取所需的bean。**

**使用实例工厂方法实例化bean时，在< bean />标签中的class属性置空，使用factory-bean的属性指定实例工厂的名字，使用factory-method属性指定非静态工厂方法的名称。factory bean虽然代表一个工厂，但是其实例仍然交给Spring管理，另外Spring中还有一个FactoryBean，这只是一个类！**

首先我们要有一个实例工厂类，提供一个实例方法用于获取Bean对象：

```java
/**
 * @author lx
 */
public class HelloSpringInstanceFactory {
   

    private static HelloSpring helloSpring = new HelloSpring();

    /**
     * 静态工厂方法
     *
     * @return 返回HelloSpring实例
     */
    public  HelloSpring getHelloSpring() {
   
        System.out.println("实例工厂方法");
        return helloSpring;
    }

    public HelloSpringInstanceFactory() {
   
        System.out.println("实例工厂会初始化");
    }
}
```

随后就是配置文件的编写：

```java
<!--实例化工厂-->
<bean id="helloSpringInstanceFactory" class="com.spring.core.HelloSpringInstanceFactory"/>
<!--factory-bean表示实例工厂的名字-->
<!--factory-method表示实例工厂方法-->
<bean name="helloSpring" factory-bean="helloSpringInstanceFactory" factory-method="getHelloSpring"/>
```

测试：

```java
@Test
public void instanceMethod() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("helloSpring", HelloSpring.class));
}
```

结果如下，工厂和bean都成功实例化：

```java
初始化
实例工厂会初始化
实例工厂方法
[helloSpringInstanceFactory, helloSpring]
com.spring.core.HelloSpring@45820e51
```

实际上，一个静态工厂或者实例工厂都可以配置多个工厂方法，那样更方便管理bean！另外，采用实例工厂方法实例化时，bean的初始化可以写在配置文件中（后面讲属性注入的时候会讲到），相比静态工厂方法更加灵活！

另外还有parent属性继承&lt; bean /&gt;也可以实例化bean，这个后面会讲到！


一个企业级项目的bean不可能仅仅是像上面我们讲的案例那样，仅仅只实例化一个对象，实际上可能会依赖到很多的属性，下面来看看依赖属性的注入，这是IoC的重点。

**依赖项注入（Dependency injection 、DI）是指对象仅仅通过构造函数参数、工厂方法的参数或从工厂方法构造或返回对象实例后在其上设置属性来定义其依赖（要使用）的其他对象。随后，IoC容器在创建bean时会自动注入这个bean的依赖项。这个过程基本上和bean主动通过类的构造器和setter方法来设置其依赖项的过程是相反的，因此DI也称为控制反转（Inversion of Control、IoC），或者说是IoC的实现。**

**我们前面讲的IoC案例，将bean的创建交给Spring来管理，这是一种IoC。但是bean之间的依赖关系却没有实现，此前我们在一个对象中依赖到另一个对象时，需要手动引入依赖的对象，有了DI之后，由来DI维护bean之间的依赖关系，并且自动注入需要的依赖项，这不也是一种IoC吗？而且使用DI之后，具有依赖关系的对象之间没有了强耦合关系，对象不需要主动查找、获取其依赖项，甚至不知道依赖项的具体位置，它们都在容器中，这一切交给DI就行了。**

**和bean的多种实例化的方式一样，属性依赖注入（DI）的方式也是有两种：构造器依赖注入和setter方法依赖注入。我们可以指定注入方式！**

## 5.1 构造器依赖注入

**构造器依赖注入是由IoC容器调用带有许多参数的构造器来完成的，每个参数表示一个依赖项。这和调用带有特定参数的静态工厂方法来构造bean几乎是一样的。**

首先我们创建一个ConstructorBased类，用来测试构造器依赖注入：

```java
/**
 * @author lx
 * 构造器依赖注入
 */
public class SimpleConstructorBased {
   
    /**
     * 依赖的两个属性
     */
    private String property1;
    private String property2;

    /**
     * 测试构造器依赖注入
     */
    public SimpleConstructorBased(String property1, String property2) {
   
        this.property1 = property1;
        this.property2 = property2;
        System.out.println("构造器依赖注入");
    }

    @Override
    public String toString() {
   
        return "SimpleConstructorBased{" +
                "property1='" + property1 + '\'' +
                ", property2='" + property2 + '\'' +
                '}';
    }
}
```

要想使用构造器依赖注入方式，需要依赖&lt; bean /&gt;标签的子标签&lt; constructor-arg &gt;，一个&lt; constructor-arg &gt;标签表示一个属性。

这里我们新建一个DI.xml配置文件，用于测试依赖注入。在文件中配置我们的bean：

```java
<!--构造函数属性注入-->
<bean id="simpleConstructorBased" class="com.spring.core.SimpleConstructorBased">
    <!--一个constructor-arg表示一个属性-->
    <constructor-arg value="v1"/>
    <constructor-arg value="v2"/>
</bean>
```

测试：

```java
/**
 * @author lx
 */
public class DITest {
   

    /**
     * 构造器依赖注入
     */
    @Test
    public void simpleConstructorBased() {
   
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
        System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
        System.out.println(ac.getBean("simpleConstructorBased", SimpleConstructorBased.class));
    }
}
```

结果如下，说明注入依赖属性成功：

```java
构造器依赖注入
[simpleConstructorBased]
SimpleConstructorBased{
   property1='v1', property2='v2'}
```

### 5.1.1 构造函数参数的解析

我们配置的&lt; constructor-arg &gt;标签表示一个参数，容器会对该标签进行解析，以保证能够匹配到一个合适的构造函数。

**解析方式有很多种，最直接的就是通过< constructor-arg >标签的value属性指定该标签对应的参数的值，容器会自动解析参数个数和参数值的类型并选择对应的构造器，上面的案例就是根据参数值value直接解析！**

#### 5.1.1.1 指定参数名

**对于引用类型的参数如果它有明确的不同的类型，容器有可能能够正常解析，但是对于基本类型和String类型，容器有时候不能正常解析！特别是对于多个构造器并且具有相同参数个数的情况！**

这里有一个SimpleConstructorBased2类：

```java
/**
 * @author lx
 * 构造器依赖注入
 */
public class SimpleConstructorBased2 {
   
    /**
     * 依赖的两个属性
     */
    private int property1;
    private String property2;
    private boolean property3;

    /**
     * 测试构造器依赖注入1
     */
    public SimpleConstructorBased2(int property1, String property2) {
   
        this.property1 = property1;
        this.property2 = property2;
        System.out.println("构造器依赖注入1");
    }

    /**
     * 测试构造器依赖注入2
     */
    public SimpleConstructorBased2(int property1, boolean property3) {
   
        this.property1 = property1;
        this.property3 = property3;
        System.out.println("构造器依赖注入2");
    }

    @Override
    public String toString() {
   
        return "SimpleConstructorBased2{" +
                "property1=" + property1 +
                ", property2='" + property2 + '\'' +
                ", property3=" + property3 +
                '}';
    }
}
```

可以看到，它有两个构造器，参数都是基本类型，且参数个数一样多，现在我们来写配置文件：

```java
<bean id="simpleConstructorBased2"
 class="com.spring.core.SimpleConstructorBased2">
    <constructor-arg value="1"/>
    <constructor-arg value="true"/>
</bean>
```

我们可能会希望调用第二个构造器，运行测试一下：

```java
/**
 * 构造器依赖注入
 */
@Test
public void simpleConstructorBased2() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("simpleConstructorBased2", SimpleConstructorBased2.class));
}
```

会发现实际上调用了第一个构造器：

```java
构造器依赖注入1
[simpleConstructorBased2]
SimpleConstructorBased2{
   property1=1, property2='true', property3=false}
```

**这就是由于Spring不能分辨基本类型和String造成的不能确定到底使用哪一个构造器的情况，这种情况怎么办呢？我们可以使用< constructor-arg >标签的name属性来指定参数名字。**

我们对配置文件进行改造，指定参数名。注意：是构造器的参数名字，而不是依赖的属性的名字：

```java
<bean id="simpleConstructorBased2" 
class="com.spring.core.SimpleConstructorBased2">
    <constructor-arg value="1" name="property1"/>
    <constructor-arg value="true" name="property3"/>
</bean>
```

继续测试之后结果如下，确实按照我们的要求调用了第二个构造器：

```java
构造器依赖注入2
[simpleConstructorBased2]
SimpleConstructorBased2{
   property1=1, property2='null', property3=true}
```

#### 5.1.1.2 指定参数类型

**指定参数名能在一定程度上解决找不到对应构造函数的情况。但是，有可能存在这样一种情况：多个构造器，具有相同的参数名和数量，但是参数类型不一致的情况，这样的情况下，仍然不能确定到底使用哪一个构造器。**

这里有一个SimpleConstructorBasedx类：

```java
/**
 * @author lx
 * 构造器依赖注入
 */
public class SimpleConstructorBasedx {
   
    /**
     * 依赖的四个属性
     */
    private String property1;
    private String property2;
    private int property3;
    private boolean property4;


    /**
     * 测试构造器依赖注入1
     */
    public SimpleConstructorBasedx(String property1, boolean property2) {
   
        this.property1 = property1;
        this.property4 = property2;
        System.out.println("构造器依赖注入1");
    }

    /**
     * 测试构造器依赖注入2
     */
    public SimpleConstructorBasedx(int property1, boolean property2) {
   
        this.property3 = property1;
        this.property4 = property2;
        System.out.println("构造器依赖注入2");
    }


    /**
     * 测试构造器依赖注入3
     */
    public SimpleConstructorBasedx(String property1, String property2) {
   
        this.property1 = property1;
        this.property2 = property2;
        System.out.println("构造器依赖注入3");
    }



    /**
     * 测试构造器依赖注入4
     */
    public SimpleConstructorBasedx(String property1, int property3, String property2) {
   
        this.property1 = property1;
        this.property2 = property2;
        this.property3 = property3;
        System.out.println("构造器依赖注入4");
    }

    /**
     * 测试构造器依赖注入5
     */
    public SimpleConstructorBasedx(String property1, String property2, int property3) {
   
        this.property1 = property1;
        this.property2 = property2;
        this.property3 = property3;
        System.out.println("构造器依赖注入5");
    }

    @Override
    public String toString() {
   
        return "SimpleConstructorBasedx{" +
                "property1='" + property1 + '\'' +
                ", property2='" + property2 + '\'' +
                ", property3=" + property3 +
                ", property4=" + property4 +
                '}';
    }
}
```

对于前三个构造器，构造器的形参列表参数名字完全一致，现在我们来写配置文件：

```java
<bean id="simpleConstructorBasedx"
 class="com.spring.core.SimpleConstructorBasedx">
    <constructor-arg name="property1" value="1"/>
    <constructor-arg name="property2" value="true"/>
</bean>
```

我们可能想要的是调用第2个构造器，即为property3和property4属性注入值，测试一下：

```java
@Test
public void simpleConstructorBasedx() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("simpleConstructorBasedx", SimpleConstructorBasedx.class));
}
```

运行之后我们发现实际上调用的第3个构造函数，即把它们都解析为了String类型：

```java
构造器依赖注入3
[simpleConstructorBasedx]
SimpleConstructorBasedx{
   property1='1', property2='true', property3=0, property4=false}
```

**这种情况怎么办呢？我们可以使用< constructor-arg >标签的type属性来指定参数类型，type的值为该属性类型的类的全路径名，基本类型就等于该类型的名字。**

我们对配置文件进行改造，指定类型：

```java
<bean id="simpleConstructorBasedx"
 class="com.spring.core.SimpleConstructorBasedx">
    <constructor-arg name="property1" value="1" type="int"/>
    <constructor-arg name="property2" value="true"  type="boolean"/>
</bean>
```

继续测试之后结果如下，确实按照我们的要求调用了第2个构造器：

```java
构造器依赖注入2
[simpleConstructorBasedx]
SimpleConstructorBasedx{
   property1='null', property2='null', property3=1, property4=true}
```

#### 5.1.1.3 指定参数顺序

**虽然可以指定参数名和参数类型，但是在某些情况下仍然有问题，比如当存在两个构造器形参列表类型一致，但是参数顺序不一致，这样的情况下，Spring仍然不能确定到底使用哪一个构造器。**

比如对于SimpleConstructorBasedx的第4、第5个构造器，它们的形参列表名字和类型一致，但是参数顺序不一致，我们对配置文件进行改造：

```java
<bean id="simpleConstructorBasedx" 
class="com.spring.core.SimpleConstructorBasedx">
    <!--一个constructor-arg表示一个属性-->
    <constructor-arg name="property1" value="xx" type="java.lang.String"/>
    <constructor-arg name="property3" value="1" type="int"/>
    <constructor-arg name="property2" value="yy" type="java.lang.String"/>
</bean>
```

运行之后，可能我们想要调用第4个构造器，但是实际上调用的第5个构造器：

```java
构造器依赖注入5
[simpleConstructorBasedx]
SimpleConstructorBasedx{
   property1='xx', property2='yy', property3=1, property4=false}
```

**这种情况怎么办呢？我们可以使用< constructor-arg >标签的index属性来指定参数出现的索引位置，index从0开始，0就表示第一个参数，1就表示第二个参数……以此类推。注意：你可以不把顺序写完整，Spring会自动寻找能通过你写的顺序识别的构造器，但是你不能把顺序写错了，那样找不到构造器就会报错！**

我们对配置文件进行改造，指定参数顺序，这里我们发现只需要指定int类型的参数顺序1或者2，Spring就能分辨出两个构造器：

```java
<bean id="simpleConstructorBasedx" 
class="com.spring.core.SimpleConstructorBasedx">
    <!--一个constructor-arg表示一个属性-->
    <constructor-arg name="property1" value="xx" type="java.lang.String"/>
    <constructor-arg name="property3" value="1" type="int" index="1"/>
    <constructor-arg name="property2" value="yy" type="java.lang.String"/>
</bean>
```

继续测试之后结果如下，确实按照我们的要求调用了第4个构造器：

```java
构造器依赖注入4
[simpleConstructorBasedx]
SimpleConstructorBasedx{
   property1='xx', property2='yy', property3=1, property4=false}
```

## 5.2 setter依赖注入

**setter依赖注入是由IoC容器调用参数的setter方法完成，setter方法是在调用构造器以实例化bean之后完成的。**

**ApplicationContext对于它所管理的bean的支持同时基于构造器和基于setter方法的依赖注入。依赖注入的属性在开始都是value字符串被保存起来，随后会通过PropertyEditor（属性编辑器，Spring内部扩展了java的原生PropertyEditor）转换为对应的实际类型，这个转换过程我们一般不需要编写代码，由IoC容器自动转换，当然我们也可以定义自己的转换器。**

setter方法实际上就是常说的get、set方法中的set方法，因此在使用setter注入的时候需要提供setXXX方法，这个XXX一般就是表示属性名，方法名应该按照Java方法名的规定来定义，属性的第一个字母大写，通常我们可以使用idea来自动生成set方法。

**在XML文件中使用< bean />的子标签< property >来表示一个setter方法，name属性表示属性名 value 属性表示属性值。注意：实际上，name属性只要是使用setXXX方法除了前面的“set”后面的字符串部分都行，不一定是属性名。怎么说呢，setter注入就是调用setter方法，传递参数，至于setter的代码逻辑，则要看你怎么编写了……**

下面是一个测试类，将使用构造器和setter两种方式混合注入：

```java
/**
 * @author lx
 */
public class SimpleSetterBased {
   
    /**
     * 依赖的5个属性
     */
    private String property1;
    private String property2;
    private int property3;
    private boolean property4;
    private int property5;


    /**
     * 构造器依赖注入
     */
    public SimpleSetterBased(String property1, String property2) {
   
        this.property1 = property1;
        this.property2 = property2;
        System.out.println("构造器依赖注入");
    }

    //setter方法依赖注入，idea生成stter方法

    public void setProperty3(int property3) {
   
        System.out.println("setter注入property3");
        this.property3 = property3;
    }
    public void setPr11operty5(int property5) {
   
        System.out.println("setter注入property5");
        this.property5 = property5;
    }

    public void setProperty4(boolean property4) {
   
        System.out.println("setter注入property4");
        this.property4 = property4;
    }


    @Override
    public String toString() {
   
        return "SimpleSetterBased{" +
                "property1='" + property1 + '\'' +
                ", property2='" + property2 + '\'' +
                ", property3=" + property3 +
                ", property4=" + property4 +
                ", property5=" + property5 +
                '}';
    }
}
```

配置文件：

```java
<!--setter and constructor-->
<bean id="simpleSetterBased" class="com.spring.core.SimpleSetterBased">
    <!--构造器参数 name表示参数名 value 表示参数值-->
    <constructor-arg name="property1" value="xxx"/>
    <constructor-arg name="property2" value="yyy"/>
    <!--setter方法 name表示属性名 value 表示属性值-->
    <property name="property3" value="123"/>
    <property name="property4" value="true"/>
    <!--name还可以表示方法名除了set后面的部分，不一定是属性名-->
    <property name="Pr11operty5" value="321"/>
</bean>
```

测试：

```java
@Test
public void setter() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("simpleSetterBased", SimpleSetterBased.class));
}
```

结果如下，我们成功注入了属性：

```java
构造器依赖注入
setter注入property3
setter注入property5
setter注入property4
[simpleSetterBased]
SimpleSetterBased{
   property1='xxx', property2='yyy', property3=123, property4=true, property5=321}
```

## 5.3 工厂方法的依赖注入

**使用静态工厂方法或者实例工厂方式实例bean时，同样可以注入依赖，方法上的参数可视为bean的依赖项，用于构造器注入，同样使用< constructor-arg >标签，而setter注入则不受影响。**

下面是两个工厂：

```java
/**
 * @author lx
 */
public class SimpleSetterBasedInstanceFactory {
   
    private static SimpleSetterBased simpleSetterBased;

    /**
     * 实例工厂依赖注入
     */
    public SimpleSetterBased getSimpleSetterBased(String property1, String property2) {
   
        System.out.println("实例工厂方法");
        simpleSetterBased = new SimpleSetterBased(property1, property2);
        return simpleSetterBased;
    }
}
/**
 * @author lx
 */
public class SimpleSetterBasedStaticFactory {
   
    private static SimpleSetterBased simpleSetterBased;

    /**
     * 静态工厂依赖注入
     */
    public static SimpleSetterBased getSimpleSetterBased(String property1, String property2) {
   
        System.out.println("静态工厂方法");
        simpleSetterBased = new SimpleSetterBased(property1, property2);
        return simpleSetterBased;
    }
}
```

配置文件：

```java
<!--静态工厂依赖注入-->
<bean id="staticSimpleSetterBased" class="com.spring.core.SimpleSetterBasedStaticFactory"
      factory-method="getSimpleSetterBased">
    <!--构造器参数 name表示参数名 value 表示参数值-->
    <constructor-arg name="property1" value="xxx"/>
    <constructor-arg name="property2" value="yyy"/>
    <!--setter方法 name表示属性名 value 表示属性值-->
    <property name="property3" value="123"/>
    <property name="property4" value="true"/>
</bean>

<!--实例工厂依赖注入-->
<bean class="com.spring.core.SimpleSetterBasedInstanceFactory" name="simpleSetterBasedInstanceFactory"/>
<bean id="instanceSimpleSetterBased" factory-bean="simpleSetterBasedInstanceFactory"
      factory-method="getSimpleSetterBased">
    <!--构造器参数 name表示参数名 value 表示参数值-->
    <constructor-arg name="property1" value="xxx"/>
    <constructor-arg name="property2" value="yyy"/>
    <!--setter方法 name表示属性名 value 表示属性值-->
    <property name="property3" value="123"/>
</bean>
```

测试：

```java
@Test
public void factory() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("instanceSimpleSetterBased", SimpleSetterBased.class));
    System.out.println(ac.getBean("staticSimpleSetterBased", SimpleSetterBased.class));
}
```

结果如下，我们成功注入了属性：

```java
静态工厂方法
构造器依赖注入
实例工厂方法
构造器依赖注入
[staticSimpleSetterBased, simpleSetterBasedInstanceFactory, instanceSimpleSetterBased]
SimpleSetterBased{property1='xxx', property2='yyy', property3=123, property4=false}
SimpleSetterBased{property1='xxx', property2='yyy', property3=123, property4=true}
```

## 5.4 依赖注入解析流程

容器执行 bean 依赖项解析简单流程如下：

1. ApplicationContext容器被实例化之后，它包含了所有bean的配置元数据。这些配置元数据可以通过XML、Java代码或注解来指定。 
2. 对于每个bean，其依赖项以属性的set方法、构造函数参数或静态工厂方法的参数的形式表示。当实际创建 bean 时，这些依赖项将提供给 bean。 
3. 每个需要注入的依赖项要设置的实际注入的value值，或对容器中另一个 bean 的ref引用（下面会讲），最开始统一为一个字符串格式 
4. 最后，属性值会从字符串的描述转换为实际属性类型。通过value设置的值， Spring可以自动将以字符串格式提供的值转换为所有内置的类型，如int、long、String、boolean等。当然我们也可以自定义转换方式，比如字符串转换为Date时间类型！

### 5.4.1 循环依赖

**bean的依赖项及其依赖项的依赖项等等会在bean的创建之前被创建，因此，如果我们使用构造器注入，那么可能出现循环依赖（Circular dependencies）的情况。**

例如：A 类需要依赖类 B 的实例，通过构造函数注入，B 类需要依赖类 A 的实例，也是通过构造函数注入。如果为要相互注入的类 A 和 B 配置 bean，这种类似于“蛋生鸡鸡生蛋”场景。IoC容器将在运行时检测到此循环引用，并抛出BeanCurrentlyCreationException。

**一种解决方式就是使用setter注入，当两个互相依赖的bean都创建完毕之后，才会调用set方法进行依赖注入！**

### 5.4.2 构造器和setter注入的选择

**通常我们对于强制依赖项使用构造器，对于可选依赖项使用setter方法，当然也可以在setter方法上加上@Required注解使属性成为必需的依赖项。**

Spring团队现在推荐使用构造器注入，构造器注入能够保证注入的组件不可变，并且确保需要的依赖不为null。此外，构造器注入的依赖总是能够在返回客户端（组件）代码的时候保证完全初始化的状态，还能检测循环依赖。另外，大量的构造函数参数是一种糟糕的代码，这意味着类可能做了太多的事情承担了太多职责，应该重构以做适当的在责任分离。

Setter注入应该主要用于可在类内分配合理默认值的可选依赖项。否则，在代码使用依赖项的任何地方都必须执行非空检查。 Setter注入的一个好处是Setter方法使该类的对象能够在以后重新配置或重新注入。

有时，在处理没有源代码的第三方类时，只有一种选择。例如，如果第三方类不公开任何setter方法，那么构造器注入是唯一的可用方式。

**相关文章**

1.  [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html) 
2.  [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html) 
3. https://spring.io/

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

