---
layout:  post
title:   RabbitMQ 高级：分布式事务详解案例——可靠消费（三）
date:   2-03-24 09:24:29 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq
-java
-分布式

---

>阿巴阿巴阿巴....&nbsp; 上篇博文讲述了分布式系统分布式事务的问题，引入RabbitMQ消息中间件的解决方案，详细演示了基于订单系统的可靠生产，本篇文章将用实际代码和例子，讲解消费者可靠消费问题&nbsp;，整体的架构代码翻阅上篇博文，这里就不一一完全贴出来了

## 前言

由于生产者和消费者不直接通信，生产者只负责把消息发送到队列，消费者负责从队列获取消息，消息被"消费"后，是需要从队列中删除的，那怎么确认消息被"成功消费"了呢？






>消费者确认分两种：<strong>自动确认</strong>、<strong>手动确认</strong>

在自动确认模式中，消息在发送到消费者后即被认为"成功消费"，这种模式可以降低吞吐量（只要消费者可以跟上），以降低交付和消费者处理的安全性，这种模式通常被称为“即发即忘”，与手动确认模型不同，如果消费者的TCP连接或通道在真正的"成功消费"之前关闭，则服务器发送的消息将丢失。因此，自动消息确认**应被视为不安全，**并不适用于所有工作负载.

### 

### 消费者过载

手动确认模式通常与有界信道预取(BasicQos方法)一起使用，该预取限制了信道上未完成（“进行中”）的消息数量。自动确认没有这种限制。因此消费者可能会被消息的发送速度所淹没，可能会导致消息积压并耗尽堆，或使操作系统终止其进程。





## 案例说明

>上篇博文讲述了，订单服务的可靠生产问题，本篇文章，将继续改造配送中心的代码

**分布式系统分布式事务解决方案——配送中心篇**


![img](https://img-blog.csdnimg.cn/0fe291e181844da081f90c21a529de22.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16) &nbsp;

### dispatcher 配送中心系统相关配置增加

①. 配送中心系统pom.xml依赖

>上篇博文中已经详细展示了配送中心的完整代码，本文例子将不会再逐一贴出，在原有依赖上，增加了mq、lombok、json格式化等依赖

```java
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.2.3</version>
            <classifier>jdk15</classifier><!-- jdk版本 -->
        </dependency>

    </dependencies>
```

②. application.yml 文件增加MQ的连接配置

```java
spring:
  rabbitmq:
    host: 127.0.0.1
    username: wpf2
    password: 123
    port: 5672
    virtual-host: test_host
```

③ . 建立Order实体类，与订单服务系统中一致，主要用于接收消息的json转换

```java
@Data
public class Order {

    // 订单ID
    private int orderId;

    // 用户名
    private String userName;

    // 商品内容
    private String context;

    // 购买数量
    private int num;
}
```



**整体项目结构调整如下：**


![img](https://img-blog.csdnimg.cn/a4118154529a4d1db8ebf34c7cd57caa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)







## 消费者：普通消费

>基于配送中心的消息普通消费，消费者收到消息后即被认为"成功消费"

1. 编写OrderMQConsumer消费者类，监听订单队列，进行消息的消费

```java
/**
 *
 *  @RabbitListener 可以标注在类上面，需配合 @RabbitHandler 注解一起使用
 *  @RabbitListener 标注在类上面表示当有收到消息的时候，就交给 @RabbitHandler 的方法处理
 *  具体使用哪个方法处理，根据 MessageConverter转换后的参数类型 （message的类型匹配到哪个方法就交给哪个方法处理）
 */
// 监听order_confirm_fanoutQueue队列，该队列在订单服务系统声明创建的
@Component
@RabbitListener(queues = {"order_confirm_fanoutQueue"})
public class OrderMQConsumer {

    @Autowired
    private DispatchService dispatchService;

    private int count=1;

    // 接收消息
    @RabbitHandler
    public void receiveMess(String message, Channel channel, CorrelationData correlationData
    , @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws Exception {

        System.out.println("接收到订单消息："+message+"，count："+count++);

        // 2.获取订单信息：mq消息存的是json格式，需要转换回来
        Order order = (Order) JSONObject.toBean(JSONObject.fromObject(message), Order.class);
        String orderId = order.getOrderId()+"";

        // 3.保存运单
        dispatchService.dispatch(orderId);
    }
}
```

2. 由于在上篇博文中，我们已经往消息队列中投递了2条消息，因此这里直接消费即可，不需要再启动订单系统的创建订单方法了


![img](https://img-blog.csdnimg.cn/fbe3afa8586b47e5a725662fdcd08861.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



3. 查看数据库配送表结果


![img](https://img-blog.csdnimg.cn/7d16b3d2e15240d39f5d9cd9fbf5aec4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

4. 查看图形化界面，队列消息的消费情况


![img](https://img-blog.csdnimg.cn/7c4f6346e82e42709e90c08a4d7e02ef.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**➳ 结论：**在消费者类中读取队列中的消息，根据订单ID，调用dispatchService.dispatch(orderId)方法，往配送表插入了2条数据。







### ☁ 思考：MQ服务器宕机或其他因素，导致数据未入库怎么办？

>生产环境往往存在很多风险，比如服务器宕机，系统重启等等


![img](https://img-blog.csdnimg.cn/4caaacd5b0144fc0996f1e20a0666a5d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

由运行结果可知，如果接收消息时发生异常，会触发服务器的重试机制，陷入死循环！如果是集群模式下，会造成MQ服务器奔溃，引发磁盘和内存消耗殆尽，知道服务器宕机为止！


![img](https://img-blog.csdnimg.cn/196c5719061948beb254a0cac8884f1a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)









## 消费者：可靠消费

>解决上述代码异常，导致消息死循环重试的几种方案

1. 控制重试的次数 + 死信队列 
2. try/catch + 手动ack 
3. try/catch + 手动ack + 死信队列处理 + 人工干预





### ▎ 方式一：配置消息重试次数

1.&nbsp;在application.yml文件中配置MQ重试次数，如下


![img](https://img-blog.csdnimg.cn/7465a915d1824629b0658f95e1800682.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

```java
spring:
  rabbitmq:
    host: 127.0.0.1
    username: wpf2
    password: 123
    port: 5672
    virtual-host: test_host
    listener:
      simple:
        retry:
          enabled: true # 开启重试，默认是false关闭状态
          max-attempts: 3 # 最大重试次数，默认是3次
          initial-interval: 2000ms # 每次重试间隔时间
```



2. 启动订单服务，让其创建一条订单，并投递到消息队列


![img](https://img-blog.csdnimg.cn/35419606d0c540bf8b586f8014f1cc4d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

3. 查看消息投递结果


![img](https://img-blog.csdnimg.cn/9d17809c251748508b5671677d606195.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

4. 启动配送中心服务，进行消息消费


![img](https://img-blog.csdnimg.cn/fafa6bb9b62f4de6a676fb20cf5788f1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

5. 查看运行结果


![img](https://img-blog.csdnimg.cn/33c72c250f2545ce9ee3e998452145fa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)


![img](https://img-blog.csdnimg.cn/d94979493f5e4e3cb69d454231623120.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



6. 图形化队列消息结果：达到最大重试次数后，队列中的消息被抛弃，无法再次捞回


![img](https://img-blog.csdnimg.cn/4a93535953494db9a72f99bf6e833c24.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**➳ 结论：**由于我们配置了重试次数，因此消费消息时，即使发生了异常，也不会陷入死循环，不断的重试，最终导致系统奔溃，cpu飙升等情况

>虽然能够解决死循环问题，但是这种情况会造成消息丢失，最终配送中心无法对这个订单进行入库等操作，从而造成 订单系统和 配送系统 数据不一致问题！







### ▎ 方式二：try/catch + 手动确认消息



1. 配送中心系统的application.yml 配置文件中配置开启手动ack


![img](https://img-blog.csdnimg.cn/405f4070a7ce4500b1a060a5d440152d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

```java
# 参数说明：none 不确认   auto 自动确认   manual 手动确认
acknowledge-mode: manual


注意：之前的配置的重试策略的参数可以去除掉了，重试策略本质也是针对消息的确认
     即使没把重试的参数配置删除，也不会生效的，如果开启了手动ack
```



2. 接收消息代码如下

```java
// 接收消息
    @RabbitHandler
    public void receiveMess(String message, Channel channel, CorrelationData correlationData
    , @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws Exception {

        try {
            // 1.获取消息队列的消息
            System.out.println("接收到订单消息："+message+"，count："+count++);

            // 2.获取订单信息：mq消息存的是json格式，需要转换回来
            Order order = (Order) JSONObject.toBean(JSONObject.fromObject(message), Order.class);
            String orderId = order.getOrderId()+"";

            // 3.保存运单
            dispatchService.dispatch(orderId);


            System.out.println(1/0); // 出现异常

            // 4.收到ack告诉mq消息已经正常消费
            channel.basicAck(tag,false);


        }catch (Exception e){
            // 如果出现异常的情况下，根据实际的情况去进行重发
            /** @param1 : 传递标签，消息的tag
             *  @param2 : 确认一条消息还是多条， false：只确认e.DeliverTag这条消息  true：确认小于等于e.DeliverTag的所有消息
             *  @param3 : 消息失败了是否进行重发   
             *            false：消息直接丢弃，不重发，如果绑定了死信队列，则消息打入死信队列
             *            true：重发，设置为true，就不要加到try/catch代码中，否则会进入重发死循环
             */
            channel.basicNack(tag,false,false);
        }
    }
```

**了解三个代码关键处：**


![img](https://img-blog.csdnimg.cn/439f7953d8e7402b8e9da4da7f4502f6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**☛ 流程解析：**

1. 发生异常，流程进入到catch块 
2. channel.basicAck(tag,false);，第一个参数是消息的标签，第二个参数是确认一条消息还是多条，我们设置的是false，表示只确认当前处理的这条消息，确认消费成功了 
3. catch块是针对消息处理的策略，准备如何处理这条消息？直接抛异常丢弃消息，还是触发消息的重新发送，具体需要根据业务进行处理

**!! 注意：用了try/catch ，yml 配置了重试次数没有意义，try/catch会屏蔽掉重试策略！！**

><span style="color:#494949;">上述图第3步，try/catch中</span>如果将<span style="color:#be191c;"><span style="background-color:#f3f3f4;">&nbsp;channel.basicNack(tag,false,false); </span></span><span style="color:#494949;">的第三个参数requeue设置为false，表示消息会直接丢弃，如果队列绑定了接盘侠死信队列，则消息会被转发到死信队列。</span>如果requeue=true，则会进入重试死循环！


![img](https://img-blog.csdnimg.cn/354ad3428c054cc5b50711ca802a396e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**➳ 选择建议：**用了try/catch ，就不要使用try/catch，二选其一即可















### ▎ 方式三：死信队列配置

>如果消息没有正常消费，我们应该设置队列的接盘侠，专门处理这些异常消息，再由一个消费者去监听死信队列，针对消息做特殊处理，整体流程如下：


![img](https://img-blog.csdnimg.cn/ed33321c676a4a4e8f25878cfec6f3b1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**由于我们的队列绑定和声明是在订单服务中完成的，因此需要修改订单系统的代码，配送中心的代码保持方式二中的消费配置，不需要修改。**

1. 声明创建死信组件

```java
@Configuration
public class Order_RabbitConfiguration {

    // 1.配置死信交换机、队列
    @Bean
    public FanoutExchange getDeadExchange(){
        return new FanoutExchange("dead_order_fanoutExchange",true,false);
    }

    @Bean
    public Queue getDeadQueue(){
        return new Queue("dead_order_fanoutQueue",true);
    }

    @Bean
    public Binding getDeadBinding(){
        return BindingBuilder.bind(getDeadQueue()).to(getDeadExchange());
    }

    // 2.配置普通队列
    @Bean
    public FanoutExchange getExchange(){
        return new FanoutExchange("order_confirm_fanoutExchange",true,false);
    }

    @Bean
    public Queue getQueue(){
        // 设置消息接盘侠：队列已满、消息拒收、消息异常 等情况，该条消息就会被重新路由到死信队列
        Map<String,Object> args = new HashMap<>();
        args.put("x-dead-letter-exchange","dead_order_fanoutExchange");
        return new Queue("order_confirm_fanoutQueue",true,false,false,args);
    }

    @Bean
    public Binding getBinding(){
        return BindingBuilder.bind(getQueue()).to(getExchange());
    }
}
```



2. 去图形化界面中，删除原来的队列，由于我们增加了队列的接盘侠，因此重新设置属性的话，需要删除原来的队列，重新创建，否则启动会报异常


![img](https://img-blog.csdnimg.cn/7d28a191250343c5a4f455da5187e91a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**!! 注意：**生产环境不建议这么做，最好是重新创建新的队列进行绑定，生产者路由到新队列中



3. 运行创建订单接口


![img](https://img-blog.csdnimg.cn/292e723891bf4872abd15b6b5349a68b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



4. 图形化界面查看， 消息投递情况和队列创建信息


![img](https://img-blog.csdnimg.cn/c82e38e23cd945a7879e8896cc87d13c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



5. 消费的代码不变，同样是try/catch + 手动ack模式，并且制造一个异常


![img](https://img-blog.csdnimg.cn/6d2c8ef47e12421d960c6f2931bf983e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

&nbsp;6. 启动配送中心系统


![img](https://img-blog.csdnimg.cn/75d9c8c4ce214476af64f52a18a8a144.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

&nbsp;7. 查看图形化结果


![img](https://img-blog.csdnimg.cn/0fd6d9cf1a634d5cbff191bc0274ce36.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### 监听死信队列

>由于上述消费异常，订单队列绑定了接盘侠死信队列，未被正常消费成功的消息会被重新路由到死信队列，因此我们也需要监听死信队列，进行消息消费！

在配送中心系统，新建死信队列的消费者


![img](https://img-blog.csdnimg.cn/5d07c41b8a1643cc88789cf411c78ebe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

```java
@Component
@RabbitListener(queues = {"dead_order_fanoutQueue"}) // 监听死信队列
public class DeadOrderMQConsumer {
    @Autowired
    private DispatchService dispatchService;

    // 接收消息：代码与订单消费者一致
    @RabbitHandler
    public void receiveMess(String message, Channel channel, CorrelationData correlationData
    , @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws Exception {
        try {
            // 1.获取消息队列的消息
            System.out.println("消息进入到了死信队列，开始处理异常消息："+message);
            // 2.获取订单信息：mq消息存的是json格式，需要转换回来
            Order order = (Order) JSONObject.toBean(JSONObject.fromObject(message), Order.class);
            String orderId = order.getOrderId()+"";
            // 3.保存运单
            dispatchService.dispatch(orderId);
            // 4.收到ack告诉mq消息已经正常消费
            channel.basicAck(tag,false);

        }catch (Exception e){
            // 由于消息进入了死信队列，说明消息有异常，需要采取新的措施来处理这条消息
            // 比如人为进行处理，同时也需要从队列中移除这条消息，防止死信队列堆积
            System.out.println("人工干预");
            System.out.println("发送邮件、短信通知技术人员等");
            System.out.println("将消息存入其他DB库，技术人员好根据消息排查");

            // 同样也要Nack这条消息，保障死信队列不会产生消息堆积
            channel.basicNack(tag,false,false);
        }
    }

}
```

2. 重新启动配送系统


![img](https://img-blog.csdnimg.cn/f40628922fc54d49915f82b379aa7db8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

3. 查看图形化界面


![img](https://img-blog.csdnimg.cn/5cc3f55894374575a94154a86862dfa8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**➳ 结论：**由上图可知，死信队列的消息被正常消费成功了，从队列中移除。死信队列的消费代码与订单消费者一致，只是在catch块的处理消息策略，需要额外增加其他处理机制













## 其他问题

yml 配置手动ACK后，消费时没有进行消息确认，会导致重复消费

>我们知道配置重试策略，当达到最大重试次数，消息会从队列中自动删除，如果同时也配置了手动ack，但实际代码没有进行ack的设置，则达到最大重试次数后，消息不会被删除，而是从ready就绪状态，变更为未应答状态

1. 配置了2种


![img](https://img-blog.csdnimg.cn/010f6ab83e03487690950984e40d4568.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

2. 消费代码如下，虽然配置了手动ack参数，但是代码中并没有手动确认

```java
@RabbitHandler
    public void receiveMess(String message, Channel channel, CorrelationData correlationData
            , @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws Exception {
        System.out.println("接收到订单消息："+message+"，count："+count++);

        // 2.获取订单信息：mq消息存的是json格式，需要转换回来
        Order order = (Order) JSONObject.toBean(JSONObject.fromObject(message), Order.class);
        String orderId = order.getOrderId()+"";


        // 3.保存运单
        dispatchService.dispatch(orderId);
    }
```

3. 控制台打印成功消费的信息，但是队列中消息并不会移除，而是从ready就绪状态，变更为未应答状态，重启项目，又会再次重复消费，直到有手动ack的消费者，将这条消息消费掉


![img](https://img-blog.csdnimg.cn/a92fdaa869204ac18325acf2279c0bbb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)


![img](https://img-blog.csdnimg.cn/137d84d3e9fc418a86aa85b7ca8b2f55.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**!! 注意：**由于执行了dispatchService.dispatch(orderId);，导致数据库创建了多条 99087的数据，因此需要注意，如果yml配置了手动确认ack，但代码消费时并没有确认消息就会造成重复消费！













 &nbsp;

## 写在最后：生产环境可靠消费注意事项

>可靠生产中，需要确保消息正确投递到队列中去，由于外界因素，网络波动导致处理延迟等因素，而可能会造成消息的投递失败，或者是多次投递。


![img](https://img-blog.csdnimg.cn/584906b641bb451cba8fb2758c027d79.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

例如订单服务投递消息成功了，但由于MQ服务器宕机，订单服务未及时收到消息投递的回执结果，触发消息的重试机制，消息被二次投递，实际消息队列中存在多条同一个订单消息记录。

**➳ 结论：消费者在消费消息时，要保证数据的幂等性，不能重复消费同一个订单。**







## 基于MQ的分布式事务解决方案总结

><strong>建议：</strong>最好是使用单体架构去处理，避免分布式事务，而非必要同步的非核心业务做成异步，提高响应速度！

优点：

1.  通用性强  
2.  拓展方便  
3.  耦合度低，方案也比较成熟 

缺点：

1.  基于消息中间件，只适合异步场景  
2.  消息会延迟处理，需要业务上能够容忍 













