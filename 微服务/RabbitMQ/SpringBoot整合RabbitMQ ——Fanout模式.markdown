
>模拟用户下订单，将订单信息封装，通过MQ进行消息分发。

假设我们有一个订单系统，用户进行下单支付，下单成功后，根据业务处理一般都会消息通知用户相关信息。例如通过邮件+手机+qq 等方式进行消息推送支付成功信息。


![img](https://img-blog.csdnimg.cn/dc7a61567f674965905ed7b6cda231b2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





## ▎生产者 Producer

>完整的项目结构如下：


![img](https://img-blog.csdnimg.cn/723b79b3e35b4285abacaa4362649556.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





**1. 创建SpringBoot 项目**


![img](https://img-blog.csdnimg.cn/9c5949e487a7457f8e93fe78f7d4e614.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**2. 勾选引入web、RabbitMQ依赖**


![img](https://img-blog.csdnimg.cn/6773141a20a14b58a99395063aa61a8f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**3. 配置文件配置rabbit参数**


![img](https://img-blog.csdnimg.cn/03f3c6bb7bc24eec8c124a692cf483f6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>注意：如果是连接本地localhost，Spring默认有配置本地可以不需要配置，默认是guest用户


![img](https://img-blog.csdnimg.cn/91a281313eba4c44828bfa0fa6aeb520.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**4. 编写**Fanout_RabbitMQConfiguration**类，声明交换机、队列等信息**

```java
@Configuration
public class RabbitMQConfiguration {

    // 1.声明fanout广播模式的交换机
    @Bean
    public FanoutExchange getExchange(){
        /**
         * @params1 ：交换机名称
         * @params2 ：是否持久化
         * @params4 ：是否自动删除
         */
        return new FanoutExchange("fanout_order_exchange",true,false);
    }

    // 2.声明三个队列队列：emailQueue、smsQueue、qqQueue
    @Bean
    public Queue getEmailQueue(){

        /**
         * @params1 ：队列名称
         * @params2 ：队列是否持久化（如果是，则重启服务不会丢失）
         * @params3 ：是否是独占队列（如果是，则仅限于此连接）
         * @params4 ：是否自动删除（最后一条消息消费完毕，队列是否自动删除）
         */
        return new Queue("email_fanout_Queue",true,false,false);
    }
    @Bean
    public Queue getSMSQueue(){
        return new Queue("sms_fanout_Queue",true,false,false);
    }
    @Bean
    public Queue getQqQueue(){
        return new Queue("qq_fanout_Queue",true,false,false);
    }

    // 3.绑定交换机与队列的关系
    @Bean
    public Binding getEmailBinding(){
        return BindingBuilder.bind(getEmailQueue()).to(getExchange());
    }
    @Bean
    public Binding getSMSBinding(){
        return BindingBuilder.bind(getSMSQueue()).to(getExchange());
    }
    @Bean
    public Binding getQQBinding(){
        return BindingBuilder.bind(getQqQueue()).to(getExchange());
    }

}
```



**5. 创建订单服务，模拟下单**

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
    public void createOrder(String userId, String productId, int num){
        // 1.根据商品ID查询库存是否充足

        // 2.生成订单
        String orderId = UUID.randomUUID().toString();
        System.out.println("订单生成成功....");

        // 3.将订单id封装成MQ消息，投递到交换机
        /**@params1 ：交换机名称
         * @params2 ：路由key/队列名称
         * @params3 ：消息内容
         */
        template.convertAndSend("fanout_order_exchange","",orderId);

    }
}
```



**6. 测试类进行测试**

```java
@SpringBootTest
class RabbitOrderSpringbootProducerApplicationTests {

    @Autowired
    private OrderService orderService;

    @Test
    void contextLoads() {
        orderService.createOrder("1001","96",1);
    }
}
```

运行结果


![img](https://img-blog.csdnimg.cn/4487398551834ad4ae5f847b5d897dac.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**7. 登陆图形化界面，查看队列等信息**


![img](https://img-blog.csdnimg.cn/8e279d5a65eb4d21bc041ff9afb1a43f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>从上图可看出，交换机和队列等信息已创建完毕，由于是fanout广播模式的交换机，因此绑定该交换机的所有队列，消息都被投递到各个队列&nbsp;









## ▎消费者 Consumer

>完整的项目结构如下：


![img](https://img-blog.csdnimg.cn/1c5b299c1ec5458683573d31970a38c6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





**!! 注意：新建项目与上述producer项目是一样的流程，这里不再展示**



**1. application.yml 配置文件如下**

>端口号需要不一致，由于生产者和消费者是2个项目，生产者项目配置的是8080，因此消费者需要与之不同的端口号，否则启动报：<span style="color:#be191c;">端口已被占用 </span>异常


![img](https://img-blog.csdnimg.cn/2eac41aa691746afb46079e92e530331.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**2. 新建3个消费者，分别监听生产者项目中，声明的3个队列，格式如下**

>qq 和 sms 的消费者类与下面一致，这里不再展示，唯一不同的是，监听队列需要修改为正确的名字

```java
/**
 @RabbitListener 可以标注在类上面，需配合 @RabbitHandler 注解一起使用
 @RabbitListener 标注在类上面表示当有收到消息的时候，就交给@RabbitHandler的方法处理，
                 具体使用哪个方法处理，根据 MessageConverter转换后的参数类型 
                （message的类型匹配到哪个方法就交给哪个方法处理）
*/
@Component
@RabbitListener(queues = {"email_fanout_Queue"})  // 监听email_fanout_Queue队列
public class EmailConsumer {

    // 接收消息
    @RabbitHandler
    public void receiveMess(String message){
        System.out.println("EmailConsumer 接收到订单消息————>"+message);
    }
}
```



**3. 启动主程序，运行消费者**


![img](https://img-blog.csdnimg.cn/6365f643a6c74ab1ab47be8a62cc10d7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)











