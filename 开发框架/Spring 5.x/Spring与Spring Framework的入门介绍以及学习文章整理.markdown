

>Spring的介绍。本系列文章将会基于最新Spring 5.x，讲解Spring的使用方法和核心原理！







Spring 框架来自2003年，到目前为止目前，Spring 框架可以说已经发展成为了一个生态体系或者技术体系，它包含了Spring Framework、Spring Boot、Spring Cloud、Spring data、Spring security、Spring AMQP等等项目（ [Spring官网的project地址](https://spring.io/projects)）。


从配置到安全，从 Web 应用程序到大数据—无论应用程序的基础结构需求是什么，都会有一个Spring项目来帮助你构建它，单独说Spring，它就像一个工具箱，内部有非常多的工具，我们选择使用合适的工具来实现我们的项目需求。 ![img](https://img-blog.csdnimg.cn/20200831164755577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

每一个项目（Project），对应不同的功能，一个项目又可能是实现类似功能的多个模块的集合。比如Spring Data项目就是一系列数据库访问、操作功能模块的集合，常见的模块包括Spring Data JDBC、Spring Data JPA等关系型数据库模块，以及Spring Data Redis等非关系型数据库模块，每个模块用于解决具体的不同的问题。

模块化的思想是Spring中非常重要的思想，每个模块既可以单独使用，又可与其他模块联合使用。我们的项目中用到某些技术的时候，我们就选择某些模块来使用即可，不会将其他模块将引入进来。

**Spring的优点（来自Spring官方）：**

1. Spring是开源的且社区活跃，被世界各地开发人员信任以及使用，也有来自科技界所有大牌的贡献，包括阿里巴巴、亚马逊、谷歌、微软等等，不用担心框架没人维护或者被废弃的情况。 
2. Spring Framework提供了一个简易的开发方式，其基础就是Spring Framework的Inversion of Control (IoC，控制反转)和Dependency Injection (DI，依赖注入)。这种开发方式，将避免那些可能致使底层代码变得繁杂混乱的大量的属性文件和帮助类。 
3. Spring提供了对其他各种优秀框架（Struts、Hibernate、Hessian、Quartz……）的直接支持，不同的框架整合更加流畅。 
4. Spring是高生产力的。Spring Boot 改变程序员的Java 编程方式，约定大于配置的思想以及嵌入式的web服务器资源，从根本上简化了很多繁杂的工作。同时我们可以将 Spring Boot 与 Spring Cloud 丰富的支持库、服务器、模版相结合，快速的构建微服务项目并完美的实现服务治理。 
5. Spring 是高性能的。使用Spring Boot能够快速启动项目，同时最新Spring 5.x支持非阻塞的响应式编程，能够极大地提升响应效率，并且Spring Boot的devtools可以帮助开发者快速迭代项目。而对于初学者，甚至可以使用Spring Initializr（https://start.spring.io/）在几秒钟之内启动一个新的Spring项目。 
6. Spring 是安全的。Spring Security 使您可以更轻松地与行业标准安全方案集成，并提供默认安全的可信解决方案。


Spring Framework直译过来就是“Spring架构”，实际上最开始Spring框架就是指的部分目前Spring Framework中的功能，后来Spring的支持功能越来越多，于是将Spring的一些核心功能抽取出来作为Spring Framework项目，另外一些相似的功能作为单独项目。可以说，Spring Framework是整个Spring生态的基石，其他的项目以及整合的其他框都是依赖Spring Framework，特别是Core technologies核心功能。

Spring Framework“提供了在企业环境中采用Java语言所需的一切”，使用Spring Framework我们就能快速的开发出一个完整的企业级应用程序！它同样是分模块的，应用程序可以选择所需的模块。

**实际上，广义的Spring是指整个Spring家族以及全部项目，包括Spring Boot、Spring Cloud等等，可以看作一个生态体系，而狭义的Spring就是指Spring Framework，其他的项目比如Spring Boot、Sping Cloud等等都是以Spring Framework作为基础演变而来的！下面我们以Spring代指Spring Framework。**

**Spring已经发展到了第五个大版本，最新Spring 5.x有如下几个模块：**

1. **Core**：所有Spring框架组件能够正常运行所依赖的核心技术模块。包括IoC容器（依赖注入、控制反转），事件，资源，i18n，验证，数据绑定，类型转换，SpEL，AOP……。我们使用其他模块的时候，核心技术模块是必须的！它提供了最基本的Spring功能支持。 
2. **Testing**：测试支持模块。包括模拟对象，TestContext框架，Spring MVC测试，WebTestClient（Mock Objects, TestContext Framework, Spring MVC Test, WebTestClient）。 
3. **Data Access**：数据库支持模块。包括事务，DAO支持，JDBC，ORM，编组XML（Transactions, DAO Support, JDBC, O/R Mapping, XML Marshalling）。 
4. **Web Servlet**：基于Servlet规范的web框架支持模块。包括Spring MVC, WebSocket, SockJS, STOMP Messaging。它们是同步阻塞式的通信的。 
5. **Web Reactive**：基于响应式的web框架支持模块。包括Spring WebFlux, WebClient, WebSocket。它们是异步非阻塞式（响应式）通信的。 
6. **Integration**：第三方功能支持模块。包括远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存等等服务支持。 
7. **Languages**：其它基于JVM语言支持模块。包括Kotlin，Groovy等动态语言。

Spring Framework 5.x（以下简称Spring 5.x）版本的代码现在已升级为使用Java8中的新特性，比如接口的static方法、lambda表达式与stream流。因此，如果想要使用Spring 5.x，那么要求开发人员的最低版本为JDK8，最高支持JDK11。官方建议将Java SE 8 undape 60 作为 Java 8的最低小版本！

Spring 5引入了Spring Web Flux，它是一个更优秀的非阻塞响应式Web编程框架，而且能更好处理大量并发连接，不需要依赖Servlet容器，不调用Servlet API，可以在不是 Servlet 容器的服务器上（如 Netty）运行，被希望用来替代Spring MVC。因为Spring MVC是基于Servlet API 构建的同步阻塞式I/O 的Web 框架，这意味了不适合处理大量并发的情况，但是目前Spring 5.x仍然支持Spring MVC。

**Spring 5.x部分遵守JavaEE 7的规范，兼容 JavaEE 8。比如它遵循的规范有：**

1. Java Servlet 3.1  [JSR 340](https://jcp.org/en/jsr/detail?id=340)：Spring支持纯Servlet API编程，但是Spring有自己的Spring MVC。 
2. Java API for WebSocket  [JSR 356](https://www.jcp.org/en/jsr/detail?id=356)：Spring支持纯WebSocket API编程。 
3. Java API for JSON Processing  [JSR 353](https://jcp.org/en/jsr/detail?id=353)：Spring支持JavaJSON格式规范。 
4. Java Message Service API (JMS)  [JSR 914](https://jcp.org/en/jsr/detail?id=914)：支持JMS消息协议API。 
5. Java Persistence 2.1（JPA） [JSR 338](https://jcp.org/en/jsr/detail?id=338)：基于O/R映射的Java持久化标准规范，Spring支持纯JPA规范编程，但是Spring有自己的Spring Data JPA。 
6. Dependency Injection  [JSR 330](https://jcp.org/en/jsr/detail?id=330)：依赖注入注解。但是Spring有自己的一套更强大的注解。 
7. 其他更多规范…………

现在让我们一起来学习Spring框架吧。目前的Spring官网的提供的 [快速开始教程](https://start.Spring.io/)都已被替换成了Spring Boot项目。Spring Boot基于约定大于配置的思想，相比传统Spring项目，提供了开箱即用的编程体验，大大的减少了开发人员的编写配置文件工作，隐藏了很多原理性的东西，初学者直接使用Spring Boot快速构建项目可能会得到更好的体验！如果我们仅仅想要“会用”的话，那么起始从Spring Boot开始是不错的选择（后面也会更新Spring Boot教程）！

为了学的更加深入，现在我们还是从手写配置文件开始搭建Spring项目，从最开始的Spring Framework Core 模块开始，从IoC开始，慢慢的增加模块和功能，最后我们再使用Spring Boot来搭建项目，这样既能深入了解Spring的核心原理，也能更好的体验到Spring Boot给开发人员带来的好处到底是什么！但是我们并不会从jar包版本开始，而是使用maven作为项目管理工具，使用坐标方式引入jar包，因此如果不会maven那么先去 [学习maven](https://download.csdn.net/download/weixin_43767015/12788525)吧！

**本系列教程分为两部分：**

1. 首先是第一部分，基于目前最新（2020.09）5.2.8.RELEASE的Spring稳定版本，从Spring核心基本功能开始，主要是讲怎么用，会依据Spring的官方文档给出一个中文教程，但并不是官方文档的直接翻译，它们的区别是：首先就是本教程使用中文对知识点进行整合然后再展示出来，其次会对官方文档中某些没有提及或者讲的过于简单的知识点进行补充，并且对于某些不常用的知识点进行舍去（因为文档中的知识点多而杂，但是我们对于大部分知识点是用不到的）。另外，最重要的是对于大部分讲解的知识点都会提供一个完整的案例（官方文档中的案例大多是说明性的案例）。 
2. 在讲完一个大的模块的知识点和用法之后，会对其中的重要原理源码进行解析。比如IoC容器模块，我们首先会讲IoC的bean的定义和注入，属性依赖的定义和注入，XML配置、注解配置以及Java代码配置方式等等核心用法，随后会讲解IoC容器启动原理，bean实例化原理、依赖注入原理等等重要流程源码（并非全部流程，因为Spring源码实在太多了，必须学会取舍）！


Spring的相关文章已经更新了很多篇了：

 [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)

 [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

https://spring.io/

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

