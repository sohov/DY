
>RabbitMQ可以对消息和队列设置TTL(消息的过期时间)，消息在队列的生存时间一旦超过设置的TTL值，就称为dead message， 消费者将无法再收到该消息。

### TTL 过期时间

对消息设置预期的时间，超过此时间后，消息被自动删除，消费者再无法接收获取



### 设置方式

1.  通过队列属性设置：队列中所有消息都有相同的过期时间  
2.  对消息进行单独设置：每条消息TTL可以不同 

><strong><span style="color:#be191c;">!! </span>注意：</strong>如同时使用2种方式，过期时间以最小的数值为准。



## 准备：项目搭建

>本文示例中是SpringBoot 整合RabbitMQ项目

1. 新建springBoot 项目


![img](https://img-blog.csdnimg.cn/d2c2c8b3fb984d6da3692ecc5145a143.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

&nbsp;2. 引入依赖


![img](https://img-blog.csdnimg.cn/afa81095bac743e79e4b3b3b8e21ea74.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

3. 配置文件配置rabbit参数


![img](https://img-blog.csdnimg.cn/03f3c6bb7bc24eec8c124a692cf483f6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

><strong>&nbsp;注意：</strong>如果是连接本地localhost，Spring默认有配置本地可以不需要配置，默认是guest用户


![img](https://img-blog.csdnimg.cn/91a281313eba4c44828bfa0fa6aeb520.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)







## 过期队列

>搭建路由模式的交换机，设置队列为过期队列

完整的项目结构如下


![img](https://img-blog.csdnimg.cn/776684610cc34beab39dbb729be232b8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



1. 编写TTL_Queue_DirectConfiguration类，声明创建交换机、队列，以及绑定关系

```java
@Configuration
public class TTL_Queue_DirectConfiguration {
    // 1.声明创建direct路由模式的交换机
    @Bean
    public DirectExchange getDirectExchange(){
        /** @params1 : 交换机名称
         *  @params2 : 是否持久化（是，重启后不会消失）
         *  @params3 : 是否自动删除（交换机无消息可投递，则自动删除交换机）
         */
        return new DirectExchange("direct_Exchange",true,false);
    }
    // 2.声明创建过期队列队列
    @Bean
    public Queue getTTL_Queue1(){

        /** 2.1 设置该队列内的所有消息过期时间为5秒
         *      key：rabbitmq图形化界面中的规定名称
         *      value：默认是int类型值，否则报错，单位毫秒
         */
        Map<String,Object> map = new HashMap<>();
        map.put("x-message-ttl",5000);

        /** @params1 : 队列名称
         *  @params2 : 是否持久化（true：重启后不会消失）
         *  @params3 : 是否独占队列（true：仅限于此连接使用）
         *  @params4 : 是否自动删除（队列内最后一条消息被消费后，队列将自动删除）
         */
        return new Queue("ttl_Queue1",true,false,false,map);
    }

    // 3.绑定交换机与队列的关系，并设置交换机与队列之间的BindingKey
    @Bean
    public Binding getBinding_TTL(){
        // 投递消息时指定的RoutingKey与此BindingKey（ttl）匹配上，消息才会被投递到ttl_Queue1队列
        return BindingBuilder.bind(getTTL_Queue1()).to(getDirectExchange()).with("ttl");
    }
}
```

**☛ 温馨提示：**上述过期队列map参数中的key， 是图形化界面规定的固定key设置，不是随意写的


![img](https://img-blog.csdnimg.cn/418ad0f9a08348a2892e3ab7d432f665.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



2. 编写OrderService类，模拟用户下单，通过MQ进行消息异步分发

```java
@Service
public class OrderService {

    // 注入Spring提供的rabbit模板服务
    @Autowired
    private RabbitTemplate template;


    // 模拟用户下单
    public void createOrder3(String userId, String productId, int num){
        // 1.根据商品ID查询库存是否充足
        ....

        // 2.生成订单
        String orderId = UUID.randomUUID().toString();
        System.out.println("订单生成成功....");

        // 3.将订单id封装成MQ消息，投递到交换机
        /**@params1 ：交换机名称
         * @params2 ：投递消息指定的RoutingKey路由键或者队列名称
         * @params3 ：消息内容
         */
        template.convertAndSend("direct_Exchange","ttl",orderId);
    }
}
```

3. SpringBoot 测试类进行测试，调用orderService 中的创建订单方法

```java
@SpringBootTest
class RabbitOrderSpringbootProducerApplicationTests {

    // 1.注入订单服务
    @Autowired
    private OrderService orderService;


    @Test
    void testDirect_TTL() {
        // 2.调用订单服务中的创建订单方法
        /** @params1 : 用户ID
         *  @params2 : 产品ID
         *  @params3 : 购买数量
         */
        orderService.createOrder3("1001","96",1);
    }
}
```

4. 运行测试类


![img](https://img-blog.csdnimg.cn/d8e3c920b7df423ea3b6533021124471.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



5. 图形化登陆查看队列结果信息


![img](https://img-blog.csdnimg.cn/da800baf321c4c0895decce1d7e5eb5f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

><strong><span style="color:#be191c;">&nbsp;!! </span>注意：</strong>这条被投递进队列的消息，5秒后将自动被删除


![img](https://img-blog.csdnimg.cn/bd57dff7cd784b3fa2898a5a84167151.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





**▶ 扩展：除了通过代码设置过期队列，还可以通过图形化界面创建过期队列**


![img](https://img-blog.csdnimg.cn/a52c77a4e44d4334a3e451e9ed4b2789.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)







## 单条过期消息

>同样是搭建路由模式的交换机，我们针对单条消息进行设置过期时间

1. 整体架构同过期队列中的一样，在原configuration 配置文件基础上，新增一个新的队列


![img](https://img-blog.csdnimg.cn/c48435569ead4a2e8e68b3321de182e9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



2. 针对单条消息设置过期时间


![img](https://img-blog.csdnimg.cn/f3e39583d15f4f23a67741bffaeeeb7c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**扩展：**如果针对单条消息进行设置，在springboot中需要借助MessagePostProcessor消息加工器对消息进行加工！&nbsp;

```java
// 消息加工类，可以针对消息进行特殊处理
MessagePostProcessor processor = new MessagePostProcessor() {
    @Override
    public Message postProcessMessage(Message message) throws AmqpException {
        // 设置消息的编码格式
        message.getMessageProperties().setContentEncoding("UTF-8");
        
        // 设置消息5秒后过期
        message.getMessageProperties().setExpiration("5000");

        // 设置消息优先级 （优先级分为消息优先级和队列优先级）
        // ——队列优先级高的会先被处理，消息优先级高的会先被消费
        message.getMessageProperties().setPriority(5);

        // 设置消息持久化
        message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        return message;
    }
};
```



3. 测试类运行


![img](https://img-blog.csdnimg.cn/7dbfa62f45ce4de691cd783ce1360217.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



4. 查看图形化结果


![img](https://img-blog.csdnimg.cn/8e3cd1b7e7494e2086439537e53661de.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>针对单条消息设置过期时间，超出时间后该条消息将会被自动删除，该队列内的其他消息不受影响&nbsp;







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





