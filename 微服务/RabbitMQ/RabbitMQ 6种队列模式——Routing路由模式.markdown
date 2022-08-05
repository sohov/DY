---
layout:  post
title:   RabbitMQ 6种队列模式——Routing路由模式
date:   2-03-22 09:39:11 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq
-分布式

---

## 路由模式

>一个交换机下面可能会绑定多个队列，但有时候我们不想把这条消息，发送给该交换机下绑定的所有队列中，那么路由模式就能够解决我们的需求。

路由模式跟发布订阅模式类似，在订阅模式的基础上加上了类型，订阅模式下的消息是分发到所有绑定到该交换机的队列，路由模式下的消息只分发到绑定在交换机下指定Routing key的队列。


![img](https://img-blog.csdnimg.cn/f0a11c4d74c24cf0a87984130908f6b1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

如上图所示，交换机的类型= direct 路由模式，队列1 与 该交换机之间绑定的key=email，而队列2与该交换机绑定的key=phone，当生产者投递消息时，可以指定一个Routing key，用来识别这条消息最终存储到哪个队列中，只有消息的 Routing key 与Binding key 相同时，交换机才会把消息发给该队列。





## 代码展示



### 准备条件

&nbsp;提醒：由于生产者和消费者的代码大同小异，为了方便，编写一个通用的连接工具类。

```java
public class MQConnectionUtils {

    // 获取连接
    public static Connection getConnection(String connectionName,String vHost){
        Connection connection = null;

        // 1.建立连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setPort(5672);
        factory.setUsername("wpf2");
        factory.setPassword("123");
        factory.setVirtualHost(vHost);

        try {
            // 2.通过连接工厂建立连接
            connection = factory.newConnection(connectionName);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        return connection;
    }

    // 释放资源
    public static void close(Connection connection, Channel channel){
        // 1.关闭通道
        if(channel!=null && channel.isOpen()){
            try {
                channel.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        // 2.关闭连接
        if(connection!=null){
            try {
                connection.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```



### 

### ▸ direct路由方式

>Direct类型交换机的路由算法是：要想一个消息能到达这个队列，需要BindingKey和RoutingKey正好能匹配得上！

如下图所示，虽然交换机绑定了两个队列，但投递消息时指定RoutingKey 为 email ，只有队列1与交换机的BindingKey匹配上了，队列2与交换机的BindingKey=phone，因此不符合


![img](https://img-blog.csdnimg.cn/17801a42624e45c6929a62c50e9b2cf1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



><strong><span style="color:#be191c;">!! </span>需要注意的是</strong>



同时，交换机也支持多重绑定。不同的队列可以用相同的Binding key与同一交换机绑定。如下图，当消息的Routing key为black时，消息将进入 Q1 和 Q2。


![img](https://img-blog.csdnimg.cn/6630191b46ff476294ecbf51353b489b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





**1. 生产者：**定义一个生产者类，建立连接、创建交换机、队列，并绑定关系

```java
public class Producer {

    public static void main(String[] args) {

        // 1.获取连接
        Connection connection = MQConnectionUtils.getConnection("生产者","test_host");

        Channel channel = null;
        try {
            // 2.通过连接建立通道
            channel = connection.createChannel();


            // 3.通过通道创建一个交换机my-exchange，第一个参数为交换机名称，第二个参数为交换机类型
            String exchangeName = "my-exchange";
            channel.exchangeDeclare(exchangeName, "direct");

            // 4.通过通道创建2个队列：my-queue1、my-queue2
            String queue1 = "my-queue1";
            channel.queueDeclare(queue1, true, false, false, null);
            String queue2 = "my-queue2";
            channel.queueDeclare(queue2, true, false, false, null);

            // 5. 指定BindingKey，绑定交换机my-exchange 和 队列my-queue1、my-queue2的关系
            String BindingKey = "email";
            /** @params1 ：要绑定的队列
             *  @params2 ：要绑定到哪个交换机
             *  @params3 ：交换机和队列之间绑定了一个key，这个key就是BindingKey
             */
            channel.queueBind(queue1,exchangeName,BindingKey);
            String BindingKey2 = "phone";
            channel.queueBind(queue2,exchangeName,BindingKey2);


            // 6.发送消息给队列
            /** @params1 ：消息发送到哪个交换机
             *  @params2 ：路由key，这个交换机可能绑定了很多队列，但这条消息我不想发给该交换机下全部的队列
             *             那么我需要指定一个RoutingKey，用来识别消息最终进入哪个队列
             *   ————上述队列与交换机的绑定中有指定BindingKey，只有消息的RoutingKey与BindingKey相同时，交换机才会把消息发给该队列
             *  @params3 ：消息内容
             */
            String routingKey = "phone";
            channel.basicPublish(exchangeName, routingKey, null, "你好 梅花十三！".getBytes());
            System.out.println("消息生产完毕！");

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            MQConnectionUtils.close(connection,channel);
        }
    }
}
```

**2. 运行生产者类**


![img](https://img-blog.csdnimg.cn/e46fbd400d2040ebb2bb55fbc419a515.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**3. 图形化查看：**我们可以看到交换机已被我们创建成功


![img](https://img-blog.csdnimg.cn/2f460335fb2247a2b11908a0e86713ac.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**4. 查看队列信息**，从下图可知，只有my-queue2队列中被投递了一条消息，因为我们指定的routing key = phone，只有my-queue2与交换机绑定的binding key =phone 匹配成功！


![img](https://img-blog.csdnimg.cn/c4fa024af03e477c858a64e28bb0523c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**5. 查看交换机与队列绑定关系**


![img](https://img-blog.csdnimg.cn/ee1d2b93278140068efe38890a8d45df.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**✦ 结论：**两个队列设置的binding key不一样，接收到的消息就不一样。路由模式下，决定消息向队列推送的主要取决于路由，而不是交换机，即便这2个队列都绑定在同一个交换机下！



