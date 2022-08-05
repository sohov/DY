---
layout:  post
title:   RabbitMQ 高级：分布式事务详解案例——可靠生产（二）
date:   2-03-24 09:24:21 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-java
-spring
-开发语言

---

>在上一篇博文中，我们讲述了分布式事务存在的问题，本文将用一个简单的demo代码演示分布式系统存在的事务问题，以及通过MQ的方式进行解决，达成数据最终一致性方案




![img](https://img-blog.csdnimg.cn/701593fe67b14d9792bd9f161a5a2248.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**以上图所示流程为例，我们建立订单服务和配送中心两套系统，具体流程如下**

1. 用户在订单服务系统完成下单等操作 
2. 订单信息存入订单数据库 
3. 获取订单ID，通过http远程调用配送中心接口，并携带参数订单ID 
4. 配送中心根据订单ID，进行骑手接单，数据存库等操作

**!! 注意：**订单服务创建订单时包裹了事务，配送中心接收订单ID，进行派单流程时也包裹了事务，因此，总共存在2个事务







## 项目架构

**系统：**订单服务（负责用户订单的接收）、运单中心（负责订单的配送）

**使用技术：**SpringBoot + SpringMVC + JdbcTemplate

**项目内容：**两套系统，订单系统下单，配送系统骑手接单

**功能描述：**订单系统创建订单，封装订单信息远程调用rpc接口（配送系统），配送服务通过订单详情进行对订单配送等追踪物流信息

>问题描述：创建订单有事务，调用远程接口（运单系统）如果超时，则抛出异常，事务回滚，由于是2个系统，独立的jvm，独立的数据库连接，连接过程中事务的管理只能控制自身的系统，而无法去回滚其他系统， 最终导致两个订单系统和配送系统的数据不一致。

**数据库**

```java
-- 订单系统数据库
create schema test collate utf8mb4_0900_ai_ci;

-- 订单表
create table orders
(
    id       int          not null primary key, -- 订单ID
    context  varchar(200) not null,             -- 商品内容
    num      int          null,                 -- 数量
    username varchar(200) null                  -- 用户名称
);


------------------------------------------------------------------------

-- 配送系统数据库
create schema test2 collate utf8mb4_0900_ai_ci;

-- 配送数据表
create table kd_Order
(
    id      int auto_increment primary key,-- 主键ID
    orderId int           null,            -- 订单ID
    name    varchar(200)  null,            -- 骑手名称
    status  int default 0 null             -- 配送状态 0.配送中 1.配送完成  2.取消
);
```





### dispatcher 配送中心系统

>项目结构如下


![img](https://img-blog.csdnimg.cn/9014d432dd284402b65e1da2df7e2d37.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





1. 相关依赖

```java
<!-- spring -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- lombok 依赖-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- jdbctemplate 依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

2. application.yml 配置文件参数如下

```java
# 配送系统的端口号要与订单系统不一样
server:
  port: 9000

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test2?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

3. 编写DispatchService实现类，完成骑手接单数据入库等操作

```java
@Service
@Transactional(rollbackFor = Exception.class)
public class DispatchService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void dispatch(String orderId) throws Exception {
        String addSql = "insert into kd_Order(orderId, name,status)  values (?,?,?)";
        int count = jdbcTemplate.update(addSql, orderId, "骑手张飞","0");
        if(count!=1){
            throw new Exception("物流信息创建失败，原因【数据库插入失败】");
        }
    }
}
```

4. 编写web层DispatchController类

```java
@RestController
@RequestMapping("/dispatch")
public class DispatchController {
    @Autowired
    private DispatchService dispatchService;

    // 添加订单后，添加骑手接单信息
    @GetMapping("/order")
    public String lock(String orderId) throws Exception {

        // 服务与服务之间的调用，可能会存在网络抖动，或者延迟
        if(orderId.equals("100001")){
            Thread.sleep(3000L);  // 模拟业务处理耗时。接口调用者会认为超时
        }
        dispatchService.dispatch(orderId);
        return "success";
    }
}
```





### order 订单系统

>项目结构如下


![img](https://img-blog.csdnimg.cn/76ef4b948fb247f5ae1809870bf809bb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



&nbsp;1. 相关依赖 （ 与配送中心系统一致 ）

2. 实体类Order

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

3. OrderDao接口，完成数据信息入库

```java
@Service
public class OrderDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    // 创建订单
    public void saveOrder(Order order) throws Exception {
        String addSql = "insert into orders (id,username,context,num) values (?,?,?,?)";
        int count = jdbcTemplate.update(addSql, order.getOrderId(),order.getUserName(), order.getContext(), order.getNum());
        if(count!=1){
            throw new Exception("订单创建失败，原因【数据库插入失败】");
        }
    }
    /**
     * 保存消息到本地数据库
     */
    public void saveLocalMessage(Order order) throws Exception {
        String addSql = "insert into message_order (context,orderId,status) values (?,?,?)";
        int count =jdbcTemplate.update(addSql, order.getContext(), order.getOrderId(),"0");
        if(count!=1){
            throw new Exception("冗余订单信息保存失败，原因【数据库插入失败】");
        }
    }
}
```

3. 实现层 OrderService

```java
@Service
public class OrderService {

    @Autowired
    private OrderDao orderDao;


    // 订单创建，整个方法添加事物
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(Order order) throws Exception {

        // 1.订单信息--插入订单系统，订单数据库事物
        orderDao.saveOrder(order);

        // 2.通过http接口发送订单信息到运单系统
        String result = dispatchHttpApi(order.getOrderId());
        if(!"success".equals(result)){
            throw new Exception("订单系统创建失败，原因是运单接口调用失败！");
        }
    }


    // 模拟http请求接口发送，运单系统，将订单号传过去
    public String dispatchHttpApi(int orderId){

        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();

        // 连接超时时间，连接上服务器(握手成功)的时间，超出抛出connect timeout
        factory.setConnectTimeout(3000); // 3秒

        // 处理超时：数据读取超时时间(毫秒)，服务器返回数据(response)的时间，超过抛出read timeout
        factory.setReadTimeout(2000); // 2秒

        // 发送http请求
        String url = "http://localhost:9000/dispatch/order?orderId="+orderId;

        RestTemplate restTemplate = new RestTemplate(factory);
        String result = restTemplate.getForObject(url,String.class);
        return result;
    }


}
```

4. springBoot测试类，模拟订单下单

```java
@SpringBootTest
class RabbitConfirmApplicationTests {
    @Autowired
    private OrderService orderService;

    @Test
    void createOrder() throws Exception {
        Order order = new Order();
        order.setOrderId(100001);
        order.setUserName("张三");
        order.setContext("方便面"); // 订单内容
        order.setNum(1);
        orderService.createOrder(order);
    }
}
```



### 运行流程

1. 启动配送系统服务


![img](https://img-blog.csdnimg.cn/109d59af26af4b3490fcf39d5c7385c4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



2. 运行订单系统中的测试下单方法


![img](https://img-blog.csdnimg.cn/43fae73845cb49f8bbe8739ae773da76.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

&nbsp;3. 查看运行结果


![img](https://img-blog.csdnimg.cn/c0218bc0f23f464abfdafaa4788f2d29.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**✷ 调用配送系统时，发生超时异常**

由于订单服务设置了2秒处理超时机制，在配送中心系统中，休眠了3秒，从而引发订单系统调用配送中心的远程接口，造成读超时，导致订单系统的事务进行了回滚。


![img](https://img-blog.csdnimg.cn/b8ffa55964d348b0a0cf5fbec184b496.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



由于分布式事务问题，配送中心的事务不受订单服务的事务控制，因此配送系统休眠3秒后，继续处理业务插入数据，而订单服务由于远程接口处理超时，导致事务回滚，**数据的完整性遭到破坏！**



![img](https://img-blog.csdnimg.cn/50910725db6640ed80419f16d759bd0b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)![img](https://img-blog.csdnimg.cn/59e76786738147ea9566f24bb7bb117a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)









## 基于MQ方案解决分布式事务问题

>主要注意的是：使用了RabbitMQ并不能完全解决分布式系统事务所引发的数据不一致性问题，必须通过一系列的策略和手段，来结合消息中间件，一起达成数据最终一致性解决方案



### ▎基于MQ的分布式事务整体设计思路

使用消息中间件来接收消息的载体，将订单信息封装成消息投递到交换机，交换机再转发到队列，由队列推送消息到配送中心，最后配送中心系统进行读取消息消费，整体流程如下：


![img](https://img-blog.csdnimg.cn/0fe291e181844da081f90c21a529de22.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**✦ 说明：**MQ类似于一个中间商一样，双方系统通过中间商来进行交流沟通，到达异步处理效果。





### ▎Order系统代码改造

>在上述order系统的基础上，我们需要增加rabbit消息中间件的支持，以及对相关代码改造。

**① 代码变更：**流程如下

1. 配置MQ的连接信息 
2. 声明创建与绑定相关MQ组件：交换机、队列、交换机&队列 
3. 增加消息service类，用于消息投递 
4. service层执行订单创建、封装订单信息、向消息队列投递订单消息

**② 项目结构如下：**


![img](https://img-blog.csdnimg.cn/418729d5174d4bd5bb9eb2c176853b14.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



1. 在原有订单服务系统的基础上增加RabbitMQ等相关依赖

```java
<!-- rabbit 依赖-->
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
     <groupId>org.springframework.amqp</groupId>
     <artifactId>spring-rabbit-test</artifactId>
     <scope>test</scope>
</dependency>

<!-- json对象格式化依赖 -->
<dependency>
      <groupId>net.sf.json-lib</groupId>
      <artifactId>json-lib</artifactId>
      <version>2.2.3</version>
      <classifier>jdk15</classifier><!-- jdk版本 -->
</dependency>
```

2. 在application.yml配置文件中，配置MQ的连接信息

```java
server:
  port: 8080

spring:
  rabbitmq:
    host: 127.0.0.1
    username: wpf2
    password: 123
    port: 5672
    virtual-host: test_host


  datasource:
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

3. 编写Order_RabbitConfiguration配置类，声明创建交换机、队列以及绑定关系

```java
// 也可以不用编写以下配置类，直接在图形化界面创建好队列和交换机也可以
@Configuration
public class Order_RabbitConfiguration {

    @Bean
    public FanoutExchange getExchange(){
        return new FanoutExchange("order_confirm_fanoutExchange",true,false);
    }

    @Bean
    public Queue getQueue(){
        return new Queue("order_confirm_fanoutQueue",true);
    }

    @Bean
    public Binding getBinding(){
        return BindingBuilder.bind(getQueue()).to(getExchange());
    }
}
```

4.编写MQService类，用于消息投递

```java
@Service
public class MQService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    // 投递订单信息到mq队列
    public void pushMessage(Order order){
        // 将订单对象转成json
        String objStr = JSONObject.fromObject(order).toString();
        // 投递消息到交换机中
        rabbitTemplate.convertAndSend("order_confirm_fanoutExchange","",objStr,new CorrelationData(order.getOrderId()+""));
    }
}
```

5.&nbsp;改造OrderService类代码，创建订单后，再把订单信息投递到消息队列

```java
@Service
public class OrderService {

    @Autowired
    private OrderDao orderDao;
    @Autowired
    private MQService mqService;

    public void createMQOrder(Order order) throws Exception {
        // 1.下单
        orderDao.saveOrder(order);

        // 2.通过mq分发消息
        mqService.pushMessage(order);
    }
}
```

6. 测试类进行测试，检测消息是否能够投递到队列中


![img](https://img-blog.csdnimg.cn/b80bcb42bcc24f6ea9b0ff542adcb1e9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

7. 查看图形化结果


![img](https://img-blog.csdnimg.cn/9ff654f7a48641c7a063b2d012da5e30.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

点击进去队列，通过图形化界面获取消息，可以看到具体被投递消息的内容


![img](https://img-blog.csdnimg.cn/01edf1b16f6f4ea482497729a06721e0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



8. 查看数据库插入结果&nbsp;


![img](https://img-blog.csdnimg.cn/c5a340288572413ab8be21b625fcf250.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**➳ 结论：**上述过程中，我们完成了消息中间件的引入，创建订单之后，把订单信息封装再投递到消息队列中，作为订单系统需要做的使命已经完成！





### ☁ 思考：消息百分百投递到队列中？

假如我们将消息投递给交换机，而交换机路由不到队列该怎么处理呢，比如此时刚好MQ服务器宕机了，或网络延迟导致，导致路由失败。

在 springboot 中 如果交换机找不到队列默认是直接丢弃！因此我们需要保证：订单服务作为一个生产消息者，消息必须投递成功到队列，如果投递失败了，需要将消息进行重新投递等等策略！ &nbsp;





## publisher 生产者确认机制

>使用MQ作为消息存储，达到消息异步分发，就需要考虑两个问题：<strong><span style="color:#be191c;">可靠生产</span>、<span style="color:#be191c;">可靠消费</span></strong>。

**什么是publisher生产者确认机制？**

就是你去驿站寄快递（投递消息），快递员揽收快递后，给你一个快递单号（回执单），接下来你的快递就会被物流系统运送，而你的任务是保证这个快递已经成功寄出去了，即快递员揽收成功！

在订单服务系统里，我们在投递消息到交换机中时，可能由于网络波动，或者MQ服务宕机导致投递不成功，就可能引发配送中心无法读取到订单信息，造成数据丢失。那么就必须保证可靠生产！

**1、基于MQ的分布式事务消息的可靠生产问题**


![img](https://img-blog.csdnimg.cn/e634194c00a8458f885274dad31d9689.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**☛ 说明：**如果想要消息的可靠生产，就必须知道消息的投递状态，因此，我们需要改造订单服务的代码，在原来的基础上增加一张消息冗余表，订单创建的同时，冗余一份订单数据到消息表中：

1. 记录订单每条数据和是否投递成功的状态 
2. 利用RabbitMQ提供的publisher生产者确认机制，更新消息冗余表中的投递状态！

 &nbsp;

### ▎代码变更：可靠生产

1. 在订单db中，增加了一张订单消息冗余表

```java
-- 存放在订单服务的数据库中
use test;

-- 冗余订单表
create table message_order 
(
    id      int auto_increment primary key,-- 主键
    context varchar(200)  null,  -- 商品内容
    status  int default 0 null,  -- 状态 0.未投递  1.投递成功  2.消息异常
    orderId varchar(200)  not null
);
```

2. 改造OrderDao类，保存订单的同时，冗余一份订单消息数据

```java
@Service
public class OrderDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    // 原创建订单
    public void saveOrder(Order order) throws Exception {
        String addSql = "insert into orders (id,username,context,num) values (?,?,?,?)";
        int count = jdbcTemplate.update(addSql, order.getOrderId(),order.getUserName(), order.getContext(), order.getNum());
        if(count!=1){
            throw new Exception("订单创建失败，原因【数据库插入失败】");
        }
    }


    // 可靠生产：创建订单
    public void saveMQOrder(Order order) throws Exception {
        // 1.保存订单到数据库
        String addSql = "insert into orders (id,username,context,num) values (?,?,?,?)";
        int count = jdbcTemplate.update(addSql, order.getOrderId(),order.getUserName(), order.getContext(), order.getNum());
        if(count!=1){
            throw new Exception("订单创建失败，原因【数据库插入失败】");
        }
        /**
          2.冗余一份订单信息到本地数据库
            原因：下单之后，rabbit可能会出现宕机，就引发消息没有投递成功，为了消息可靠生产，对消息做一次冗余
         */
        saveLocalMessage(order);
    }

    /**
     * 保存消息到本地数据库
     */
    public void saveLocalMessage(Order order) throws Exception {
        String addSql = "insert into message_order (context,orderId,status) values (?,?,?)";
        int count =jdbcTemplate.update(addSql, order.getContext(), order.getOrderId(),"0");
        if(count!=1){
            throw new Exception("冗余订单信息保存失败，原因【数据库插入失败】");
        }
    }
}
```

**!! 注意：**dao类中，我没有去除原来的saveOrder方法，而是新增了，因此在 OrderService中，调用dao的保存订单方法，记得修改为新的方法为&nbsp;saveMQOrder，而不是&nbsp;saveOrder



3. 在MQService类中增加publisher确认机制

```java
@Service
public class MQService {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private JdbcTemplate jdbcTemplate;
 
    
     // publisher生产者确认
    /**
     * 注解好多人以为是Spring提供的，其实是java自己的注解
     * @PostConstruct ：用来修饰一个非静态的void方法，在服务器加载servlet时运行，
                        PostConstruct在构造函数之后执行，init()方法前执行
     * 作用：在当前对象加载完依赖注入的bean后，运行该注解修饰的方法，而且只运行一次。
     * （该注解可以用来当作一些初始化的处理）
     */
    @PostConstruct
    public void regCallback(){
        // 注意：下面内容也可放到消息投递的方法里，为了代码层次感，因此封装为单独的方法
        // 生产者确认机制：消息发送成功后，给予生产者的消息回执，来确保生产者的可靠性
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                // 如果ack为true 代表消息已投递成功
                String orderId = correlationData.getId();
                if(!ack){
                    // 消息投递失败了，这里可能要进行其他方式存储
                    System.out.println("MQ队列应答失败，orderId是："+orderId);
                    return;
                }

                // 应答成功业务逻辑
                try {
                    // 投递成功：冗余消息表该订单状态改为1，表示该条消息已成功投递到队列中去
                    int count = jdbcTemplate.update("update message_order set status=1 where orderId=?", orderId);
                    if(count==1){
                        System.out.println("本地消息状态修改成功，消息成功投递到消息队列......");
                    }
                }catch (Exception e){
                    System.out.println("本地消息状态修改失败，原因："+e.getMessage());
                }
            }
        });
    }


    // 投递订单信息到mq队列
    public void pushMessage(Order order){
        // 将订单对象转成json
        String objStr = JSONObject.fromObject(order).toString();
        // 投递消息到交换机中
        rabbitTemplate.convertAndSend("order_confirm_fanoutExchange","",objStr,new CorrelationData(order.getOrderId()+""));
    }
}
```

4. service层增加了生产者确认的代码还不够，还需要配置application.yml文件，否则代码会失效

```java
server:
  port: 8080

spring:
  rabbitmq:
    host: 127.0.0.1
    username: wpf2
    password: 123
    port: 5672
    virtual-host: test_host
    publisher-confirm-type: correlated  # 配置生产者确认机制

# NONE：禁用发布确认模式，是默认值
# CORRELATED：发布消息成功到交换器后会触发回调方法
# SIMPLE：经测试有两种效果
    # 其一效果和CORRELATED值一样会触发回调方法
    # 其二在发布消息成功后使用rabbitTemplate调用waitForConfirms或waitForConfirmsOrDie方法等待broker节点返回发送结果
    # 根据返回结果来判定下一步的逻辑，要注意的点是waitForConfirmsOrDie方法如果返回false则会关闭channel，则接下来无法发送消息到broker;


  datasource:
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

5. 测试类运行，看消息是否投递成功


![img](https://img-blog.csdnimg.cn/4a0b9a8aa8ae4793b0bc27b59217fe60.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

6. 查看数据库插入结果&nbsp;



![img](https://img-blog.csdnimg.cn/d19b410dd22e4bd7909887d5a42a397d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)![img](https://img-blog.csdnimg.cn/ec77f8c0274c4f47a4c36539bf2471ea.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

><strong><span style="color:#be191c;">☛ </span>细节说明：</strong>仔细看代码，创建订单时，同时插入了一条冗余订单消息记录，插入时的status=0，数据库中是1，是因为在publisher生产者确认机制代码中，由于收到了消息投递成功的回执，所以修改了此条消息的status=1，表示消息成功投递。



7. 图形化界面，查看生产者可靠的改造代码，消息是否投递成功


![img](https://img-blog.csdnimg.cn/d9913b9349da42469670d56b2d56d4ff.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**➳ 结论：**从结果得知，通过RabbitMQ提供的publisher生产者确认，以及消息冗余机制，基本保证了消息的可靠生产问题，但这种可靠生产的方式的前提是必须保证MQ服务器正常运行。





### 问题一：生产者确认失败问题

答：在上述第5步运行测试类中，由于是一个测试实例，我们运行完之后，服务就立马被关闭了，可能会导致在MQService中的确认机制收不到回执信息，导致MQ队列应答失败，因此，建议在测试类中，调用完orderService.createMQOrder(order); 创建订单之后，增加2秒休眠时间，让确认机制能有充分时间回执



### 问题二：获取回执信息失败问题

如果此时MQ服务器出现了异常故障，那么消息是无法获取到回执信息，怎么解决呢？





### 2、基于MQ的分布式事务消息的可靠生产问题——定时重发

>如果MQ服务器宕机，无法响应，我们可以使用定时器，定时扫描消息冗余表中status=0，也就是未发送成功的消息，进行二次发送，并记录消息的重试次数，如果重试次数&gt;2，说明消息存在异常，那么就可以发送邮件，通知人为处理消息


![img](https://img-blog.csdnimg.cn/55e10d35afb34cd199667a3b3ff18228.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

定时重发代码就不具体展示了，这里只负责提一下MQ服务器宕机，无法响应回执的兜底解决方案

```java
@EnableScheduling
public class TaskMQService {

    // 每隔5秒重发一次消息
    @Scheduled(cron = "xxx")
    public void sendMessage(){
        // 1.把冗余消息表中status为0的状态消息重新查询出来，投递到MQ中
    }
}
```



## 可靠生产流程图

>在订单服务系统中，整体保证可靠生产消息的流程图如下


![img](https://img-blog.csdnimg.cn/e3a30fcfc83e4bc1827e3dab0572a74b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)







## 思考：发布的消息什么时候会被MQ服务器确认?

>发布者确认分为同步和异步两种

**对于不可路由的消息**

broker 将在 exchange 验证消息不会路由到任何队列（发回一个空的队列列表）后发出确认；如果消息被设置为"必需消息"发布，即 BasicPublish() 方法的 "mandatory" 入参为true，则 BasicReturn 事件将在 BasicAcks&nbsp;事件之前触发，否定确认 BasicNacks 事件也是如此.



**对于可路由消息**

当所有队列都接受消息时才触发&nbsp;BasicAcks 事件

1. 对于路由到持久化队列的持久性消息：持久化到磁盘后才会触发 BasicAcks 事件 
2. 对于消息的镜像队列：所有镜像都已接受该消息后才会触发 BasicAcks 事件.



为了保证持久化效率，RabbitMQ不是来一条存一条，而是定时批量地持久化消息到磁盘。RabbitMQ 消息存储一段时间（几百毫秒）之后或者当队列空闲时，才会批量写到磁盘。

>这意味着在恒定负载下：<br> BasicAck 的延迟可以达到几百毫秒。如果队列支持镜像队列，则延迟时间更大

**➳ 建议：**为了提高吞吐量，应用程序采用异步确认方式，或者发布批量消息后等待确认。







## 发布者确认的注意事项

在大多数情况下，RabbitMQ将以与发布时相同的顺序向发布者确认消息（这适用于在单个频道上发布的消息）。但是，发布者确认是异步发出的，可以确认单个消息或一组消息。发出确认的确切时刻取决于消息的传递模式（持久性与瞬态）以及队列的属性。

也就是说，RabbitMQ可能不以消息发布的顺序向发布者发送确认消息。生产者端尽量不要依赖消息确认的顺序处理业务逻辑！！





## 其他拓展：事务机制

>除了生产者确认机制之后，还可以使用事务机制来保证消息的可靠生产

在标准的AMQP 0-9-1，保证消息不会丢失的唯一方法是使用事务：在通道上开启事务，发布消息，提交事务。但是事务是非常重量级的，它使得RabbitMQ的吞吐量降低250倍。

```java
//生产者部分代码:
try{               
    channel.TxSelect();//开启事务机制               
    channel.BasicPublish("", QueueName, null, "hello world");              
    channel.TxCommit();//提交
    Console.WriteLine($"send {msg}");
}catch (Exception e){                
    channel.TxRollback();//回滚
    Console.WriteLine(e);
}
```

**!! 注意：**一旦信道处于生产者确认模式，则不能开启事务，即事务和生产者确认机制只能二选一 &nbsp;





## 写在最后

本文讲述了分布式系统分布式事务引发的数据不一致问题，并且用实际代码例子进行了演示，也讲述了解决此问题的相关方案，比如，引入消息中间件。

消息中间件也有可能造成消息丢失，因此我们需要采用一系列手段和策略，来保证消息的可靠性。

>end：本文例子中只展示了订单服务，作为消息生产者可靠性保证案例，没有讲述配送中心如何进行消息消费的案例和代码，翻阅下一篇博文吧，我干不动了！

 &nbsp;

