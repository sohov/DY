---
layout:  post
title:   RabbitMQ高级：死信队列详解
date:   2-03-24 09:23:36 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq
-分布式
-java

---

>应用场景：下单20分钟还未完成支付，订单自动取消（转移到死信队列，并更新订单状态）





## 什么是死信队列？

DLX (全称：Dead-Letter-Exchange 可称为死信交换机)，如果一条消息成为死信 dead message，它不是直接丢弃掉，而是再转发到另外一个交换机，由这个交换机来处理这条死信。即DLX，绑定DLX的队列称之为死信队列。

>DLX与其它正常交换机无区别，当队列中存在死信时，RabbitMQ自动会将此消息重新发布到设置的死信交换机中，进而被路由到死信队列

**➳ 结论：** **死信队列相当于接盘侠，如果某个队列设置了接盘的死信队列，当队列内的消息无法正常消费时，该条消息会被重新路由到指定的死信队列**







## 如何判断一条消息是死信？

- 消息被拒绝 
- 消息过期 
- 队列达到最大长度

>消息被应答机制拒绝时、队列是过期队列，队列中的消息过期时、队列满了无法接收新消息







## 接盘侠设置

>这里将展示两种变更为死信消息的场景，一种是消息过期，一种是达到队列最大长度

### 

### ▎过期队列：消息过期

1. 编写Dead_Queue_DirectConfiguration配置类，声明创建死信交换机、队列、绑定关系

```java
@Configuration
public class Dead_Queue_DirectConfiguration {

    // 1.声明创建direct路由模式的交换机——死信交换机
    @Bean
    public DirectExchange getDead_DirectExchange(){
        return new DirectExchange("dead_Exchange",true,false);
    }

    // 2.声明创建队列——死信队列
    @Bean
    public Queue getDead_Queue1(){
        return new Queue("dead_Queue1",true);
    }

    // 3.绑定交换机与队列的关系，并设置交换机与队列之间的BindingKey
    @Bean
    public Binding getBinding_TTL(){
        // 只有投递消息时指定的RoutingKey与这个BindingKey（dead）匹配上，消息才会被投递到该队列
        return BindingBuilder.bind(getDead_Queue1()).to(getDead_DirectExchange()).with("dead");
    }
}
```

**!! 注意：**死信交换机和死信队列，与普通交换机、队列的声明创建方式一样



2. 编写TTL_Queue_DirectConfiguration配置类，声明创建一个过期队列

>关于TTL过期队列和消息的设置，代码和项目结构与上篇博文《RabbitMQ高级：TTL过期队列/消息设置》一致，详细讲解请移步翻阅。

```java
@Configuration
public class TTL_Queue_DirectConfiguration {
    // 1.声明创建direct路由模式的交换机
    @Bean
    public DirectExchange getDirectExchange(){
        return new DirectExchange("direct_Exchange",true,false);
    }

    // 2.声明创建过期队列队列
    @Bean
    public Queue getTTL_Queue1(){

        // 2.1 设置过期队列——该队列内的所有消息过期时间为5秒
        Map<String,Object> map = new HashMap<>();
        map.put("x-message-ttl",5000);

        // 2.2 设置消息接盘侠：消息过期后，不自动删除，而是将消息重新路由到dead_Exchange交换机
        map.put("x-dead-letter-exchange","dead_Exchange");
        // 2.3 设置消息接盘侠的具体路由key
        map.put("x-dead-letter-routing-key","dead"); // 如果是fanout模式的死信队列，则这里不需要设置投递的RoutingKey


        return new Queue("ttl_Queue1",true,false,false,map);
    }

    // 3.绑定交换机与队列的关系，并设置交换机与队列之间的BindingKey
    @Bean
    public Binding getBinding_Queue_TTL(){
        // 只有投递消息时指定的RoutingKey与这个BindingKey（ttl）匹配上，消息才会被投递到ttl_Queue1队列
        return BindingBuilder.bind(getTTL_Queue1()).to(getDirectExchange()).with("ttl");
    }


}
```

**★ 重点代码详解**

1. 过期队列设置：表示设置该队列是个过期队列，队列内的消息5秒后将过期，消息会被自动删除

```java
map.put("x-message-ttl",5000);
```

2.设置死信交换机：也就是说，如果当前队列的消息无法正常消费，比如超过5秒到期了，消息不会自动删除，而是会被重新投递到dead_Exchange交换机里，死信消息的接盘侠。

```java
map.put("x-dead-letter-exchange","dead_Exchange");
```

3. 接盘侠的RoutingKey：如果创建的死信交换机的类型是direct路由，或者topics主题模式，那么需要指定消息的RoutingKey，将被投递到哪个队列中，因为交换机下可能绑定了多个队列，交换机与队列之间绑定了一个key ，即BindingKey，投递消息时指定一个RoutingKey，只有BindingKey与RoutingKey匹配成功，该条消息才会被交换机投递到相应BindingKey的队列

```java
map.put("x-dead-letter-routing-key","dead");
```

**!! 注意：如果是fanout模式的死信交换机，就无需设置**x-dead-letter-routing-key**参数！！**



上述map中的三个参数key是固定的，需要去图形化界面中复制


![img](https://img-blog.csdnimg.cn/4decf5764f1d493ab839b7d975fc9535.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





3. 编写OrderService类，模拟用户下单，通过MQ进行消息的分发

```java
@Service
public class OrderService {
    @Autowired
    private RabbitTemplate template;


    /**
     * 模拟用户创建订单
     * @param userId  客户ID
     * @param productId 产品ID
     * @param num 数量
     */
 
    public void createOrder_ttl_queue(String userId, String productId, int num){
        // 1.根据商品ID查询库存是否充足

        // 2.生成订单
        String orderId = UUID.randomUUID().toString();
        System.out.println("订单生成成功....");

        // 3.将订单id封装成MQ消息，投递到交换机
        /**@params1 ：交换机名称
         * @params2 ：RoutingKey路由键/队列名称
         * @params3 ：消息内容
         */
        template.convertAndSend("direct_Exchange","ttl",orderId);
    }

    }

}
```



4. 运行测试类，调用orderService 创建订单方法


![img](https://img-blog.csdnimg.cn/33aa41f263f7421982224e5529452186.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





5. 图形化查看队列结果


![img](https://img-blog.csdnimg.cn/ec4c95de1dd54a47a0c8b65266d7afcd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



6. 查看五秒后的结果


![img](https://img-blog.csdnimg.cn/28dfd69b7eef4ecbb7a68514af71cf46.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)




我们也可以点进去队列，查看队列详情![img](https://img-blog.csdnimg.cn/79a416f3b71449058b81e485a91f5fd2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**➳ 结果分析：**由上述结果可知，针对过期队列，队列里的消息到期后，会自动删除。如果该队列设置了死信队列，那么过期的消息会被重新投递到死信队列。

>注意：如果是对单条消息设置过期时间，而非设置过期队列，消息过期会自动删除，不会进入到死信队列！







### ▎达到队列最大长度

>代码与上述例子一致，唯一不同点是，在声明队列时，需要设置队列的长度，如下2.1，如果在原有代码上新增或修改队列属性，需要删除之前的队列和交换机，否则启动异常！

```java
/**
     *  消息死信原因
     *     1.消息被拒绝（消息消费时，应答拒绝）
     *     2.消息过期 （仅过期队列内的消息才会投递到死信队列，针对单条消息过期不会进入）
     *     3.队列达到最大长度
     */
    @Bean
    public Queue getTTL_Queue1(){

        // 2.1 设置当前队列最大长度5
        Map<String,Object> map = new HashMap<>();
        map.put("x-max-length",5);

        // 2.2 设置消息接盘侠：消息过期后，不自动删除，而是将消息重新路由到dead_Exchange交换机
        map.put("x-dead-letter-exchange","dead_Exchange");
        // 2.3 设置消息接盘侠的具体路由key：死信交换机是路由模式的交换机
        map.put("x-dead-letter-routing-key","dead"); // 如果是fanout模式的死信队列，则这里不需要设置投递的RoutingKey


        return new Queue("ttl_Queue1",true,false,false,map);
    }
```

2. 测试发送10条消息

```java
// 往队列中投递10条消息，使其超出队列最大长度5
@Test
void createOrder_ttl_queue_maxLength() {
    for (int i = 1; i <= 10; i++) {
        orderService.createOrder_direct("1001", "1", 12);
    }
}
```

3. 运行测试


![img](https://img-blog.csdnimg.cn/67f051403855419c910e47ee71492664.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



4. 查看图形化结果


![img](https://img-blog.csdnimg.cn/29b0fb72bd2c469796e57def2da48f6a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>由上图可知，超出过期队列设置的最大长度5，则剩下的消息都会自动被投递到死信队列。&nbsp;





### ☛ 创建队列注意事项

假设队列已经存在的情况，去重新修改队列的属性，或者新增属性，启动会报异常

```java
@Bean
public Queue ttlQueue() {
   
    // 问题描述：队列已经存在，设置了5秒过期，此时我修改为6秒，重启项目后会报错
    Map<String,Object> map = new HashMap<>();
    map.put("x-message-ttl",5000); 
  
    // ttl_queue已经存在了
    return new Queue("ttl_queue", true,false,false,args);
}
```

**解决方案**

1. 删除该队列 
2. 创建新的队列

>一般选择第二种方式，因为实际线上高速运转的开发场景，还有消费者监听队列，删除队列是非常危险的行为，可能存在消息还未消费完或消费中，可以创建新的队列，将线上新产生的消息修改路由到新队列中









