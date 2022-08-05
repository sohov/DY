---
layout:  post
title:   RabbitMQ 6种队列模式——Simple简单队列
date:   2-03-22 09:38:53 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq
-分布式

---

**目录**

 [六类工作队列模式](#%E5%85%AD%E7%B1%BB%E5%B7%A5%E4%BD%9C%E9%98%9F%E5%88%97%E6%A8%A1%E5%BC%8F)

 [简单队列模式概念](#%E7%AE%80%E5%8D%95%E9%98%9F%E5%88%97%E6%A8%A1%E5%BC%8F%E6%A6%82%E5%BF%B5)

 [maven项目构建](#maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA)

 [生产者生产消息](#%E7%94%9F%E4%BA%A7%E8%80%85%E7%94%9F%E4%BA%A7%E6%B6%88%E6%81%AF)

 [查看当前rabbitmq信息](#%E6%9F%A5%E7%9C%8B%E5%BD%93%E5%89%8Drabbitmq%E4%BF%A1%E6%81%AF)

 [► 运行producer类](#%E2%96%BA%20%E8%BF%90%E8%A1%8Cproducer%E7%B1%BB)

 [消费者消费消息](#%E6%B6%88%E8%B4%B9%E8%80%85%E6%B6%88%E8%B4%B9%E6%B6%88%E6%81%AF)

 [► 运行consumer类](#%E2%96%BA%20%E8%BF%90%E8%A1%8Cconsumer%E7%B1%BB)




## 六类工作队列模式

>&nbsp;官网介绍：<a href="https://www.rabbitmq.com/getstarted.html" title="RabbitMQ Tutorials — RabbitMQ">RabbitMQ Tutorials — RabbitMQ</a>

**▸ 简单队列模式**：一个消息生产者，一个消息消费者，一个队列。也称为点对点模式

**▸ 工作模式**：一个消息生产者，一个交换器，一个消息队列，多个消费者。同样也称为点对点模式

**▸ 发布/订阅模式**：无选择接收消息，一个消息生产者，一个交换器，多个消息队列，多个消费者

**▸ 路由模式**：在发布/订阅模式的基础上，有选择的接收消息，通过 routing 路由进行匹配接收消息

**▸ 主题模式**：在发布/订阅模式的基础上，根据主题匹配进行筛选是否接收消息，比第四类更灵活。

**▸ RPC模式**：类模式是拥有请求/回复的。也就是有响应的，上面5种都没有。





## 简单队列模式概念

>一个生产者 P 发送消息到队列 Q，一个消费者 C 接收 （<span style="color:#be191c;">1</span>:<span style="color:#be191c;">1</span>:<span style="color:#be191c;">1</span> 模式）


![img](https://img-blog.csdnimg.cn/img_convert/dc4ac5e2759dd98d185baa0c0cef1a95.png)





## maven项目构建

>简单队列模式，使用最基本的maven项目构建演示

1. 创建maven项目，next 一直下一步进行构建。


![img](https://img-blog.csdnimg.cn/9de55b16e0f74aea81cf0d7dc8eb9591.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



2. 引入rabbitmq 的maven依赖

```java
<dependencies>
     <dependency>
          <groupId>com.rabbitmq</groupId>
          <artifactId>amqp-client</artifactId>
          <version>5.12.0</version>
     </dependency>
</dependencies>
```





## 生产者生产消息

>新建producer生产者类，用于生产消息到消息队列

```java
// 所有的中间件技术都是基于TCP/IP协议基础之上构建新型的协议规范，只不过rabbitmq遵循的是amqp
public class Producer {
    
    public static void main(String[] args) {

        // 1.建立连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 基于TCP/IP协议自然而然需要端口、ip等，比如mysql，redis等中间件
        factory.setHost("127.0.0.1");
        factory.setPort(5672);
        factory.setUsername("wpf2");
        factory.setPassword("123");
        factory.setVirtualHost("test_host");

        Connection connection = null;
        Channel channel = null;
        try {
            // 2.通过连接工厂建立连接
            connection = factory.newConnection("我的第一个RabbitMQ Demo");

            // 3.通过连接建立通道
            channel = connection.createChannel();

            // 4.通过通道创建交换机，声明队列，绑定关系，路由key，发送消息，和接收消息
            String queueName = "queue001";
            /**
             * @Params1  队列名称
             * @Params2  是否持久化    true：持久化，该队列将在服务器重启后依然继续存在
             * @Params3  是否独占队列  true：独占，仅限于此连接
             * @Params4  自动删除（最后一条消息被消费完毕后，是否把队列自动删除）
             * @Params5  队列的其他属性（构造参数）
             *
             *  面试题：所谓持久化即消息存盘，非持久化会存盘吗？ 回答：会存盘，但会随着服务器宕机而丢失
             */
            channel.queueDeclare(queueName, true, false ,false, null);

            // 5.准备消息内容
            String message = "你好 梅花十三！";

            // 6.发送消息给队列
            /**  @param1：交换机    面试题：可以存在没有交换机的队列吗？ 不可能，虽未指定但会存在默认的交换机
             *   @param2：队列，路由key
             *   @param3：消息的状态控制
             *   @param4：消息主题
             */
            channel.basicPublish("", queueName, null, message.getBytes());

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            // 7.关闭通道
            if(channel!=null && channel.isOpen()){
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭连接
            if(connection!=null){
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



☛**关于以下连接工厂参数说明**

```java
// 1.IP地址，如果是本地用以下，如果是服务器，用服务器的IP
factory.setHost("127.0.0.1");

// 2.默认端口号，可通过配置文件进行修改，自行百度，rabbitmq启动失败，可能是端口号被占用
factory.setPort(5672);

// 3.用户名和密码是自主创建的，不懂的翻阅博文：RabbitMQ的角色分类和管理
factory.setUsername("wpf2");
factory.setPassword("123");

// 4.vhost之前博文有详解过，请翻阅：RabbitMQ的角色分类和管理
factory.setVirtualHost("test_host");
```

**!! 注意：在运行producer类之前，我们先来看一下目前已存在的连接、交换机、队列信息！！**





### 查看当前rabbitmq信息

>图形化界面不知道如何进入的，请翻阅：《RabbitMQ的安装与卸载》里面有手把手详细教学

**1. 连接信息**


![img](https://img-blog.csdnimg.cn/11c50ffe20c649bd993ae5f9b7bdd2da.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**2. 通道信息**


![img](https://img-blog.csdnimg.cn/ca11152a1c904a8191dee0183bc0fdf4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**3. 交换机信息**


![img](https://img-blog.csdnimg.cn/b43c9afffafa47e9bbe642e1a1dc03e4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**4. 队列信息**


![img](https://img-blog.csdnimg.cn/17655c32ab264a33aeb22213797285e6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### ► 运行producer类

>试试运行刚刚创建的生产者类，看看会发生什么

1. 点击main函数运行


![img](https://img-blog.csdnimg.cn/6fcba1bf089c41049b0ba1233e03adc4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



&nbsp;2. 运行完毕，打开图形化界面，可以看到消息已经到队列中去了，


![img](https://img-blog.csdnimg.cn/0d4aa5c6e76048e094873d18cb567159.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

><strong><span style="color:#be191c;">!! </span>说明：</strong>如果想要查看连接和通道信息，可以进行debug模式断点一步步运行结合图形化界面查看，由于producer类运行完毕，就关闭了连接和通道，所以图形化界面里是没有的





## 消费者消费消息

>新建consumer消费者类，用于从消息队列中消费消息

```java
// 代码同生产者基本一致，差异在于第4-6步
public class Consumer {

    public static void main(String[] args) {
        // 1.建立连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setPort(5672);
        factory.setUsername("wpf2");
        factory.setPassword("123");
        factory.setVirtualHost("test_host");

        Connection connection = null;
        Channel channel = null;
        try {
            // 2.通过连接工厂建立连接
            connection = factory.newConnection("我的第一个RabbitMQ Demo");

            // 3.通过连接建立通道
            channel = connection.createChannel();

            // 4.消费消息
            /**  @param1：消费的队列名称
             *   @param2：是否自动应答  true：是，消息一旦被消费成功，消息则从队列中删除
             *   @param3：消息送达时的回调
             *   @param4：消费者被取消时的回调
             */
            channel.basicConsume("queue001",true, new DeliverCallback() {
                public void handle(String consumerTag, Delivery message) throws IOException {
                    System.out.println("接收成功！消息内容：" + new String(message.getBody(), "UTF-8"));
                }
            }, new CancelCallback() {
                public void handle(String consumerTag) throws IOException {
                    System.out.println("接收消息失败。。。。。");
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            // 7.关闭通道
            if(channel!=null && channel.isOpen()){
                try {
                    channel.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            // 8.关闭连接
            if(connection!=null){
                try {
                    connection.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



### ► 运行consumer类


![img](https://img-blog.csdnimg.cn/aeed70e55b444a7bb6486a1a42d1fd62.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



我们来查看下图形界面，消息的状态


![img](https://img-blog.csdnimg.cn/bd46f9818f664dd5b1d0aac4f5a81f72.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)







### 关于声明队列的参数说明

```java
queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                                 Map<String, Object> arguments)
```

queue：queue的名称

exclusive：排他队列，如果一个队列被声明为排他队列，该队列仅对首次申明它的连接可见，并在连接断开时自动删除。这里需要注意三点：

1. 排他队列是基于连接可见的，同一连接的不同信道是可以同时访问同一连接创建的排他队列 
2. “首次”，如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同； 
3. 即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除的，这种队列适用于一个客户端发送读取消息的应用场景。

autoDelete：自动删除，如果该队列没有任何订阅的消费者的话，该队列会被自动删除。这种队列适用于临时队列。





