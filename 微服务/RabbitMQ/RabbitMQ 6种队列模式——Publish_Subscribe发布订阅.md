**发布订阅 (publish/subscribe)**
----------------------------

> 将消息发送给不同类型的消费者。做到发布一次，消费多个。

在上一篇博文中我们介绍了工作[队列](https://so.csdn.net/so/search?q=%E9%98%9F%E5%88%97&spm=1001.2101.3001.7020)。如果说工作队列是将一个任务完全分发给一个消费者。那么在发布订阅模式里，所做的完全不同 ，就是：**把一个消息交付给多个消费者**

### ▎举例说明

假设我们有一个订单系统，用户进行下单支付，下单成功后，根据业务处理一般都会消息通知用户相关信息。例如通过邮件+手机+微信等方式进行消息推送支付成功信息。

**利用 MQ 实现业务异步处理，支付成功向消息队列投递，消费者取出消息后进行业务处理：**

![](https://img-blog.csdnimg.cn/37cd927af33941ab8ecdf93675dbc7fc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

这种方式不但总耗时长，并且业务混乱，实际上短信、邮件、微信 是不同的业务逻辑，不应该放在一块处理，而应该根据业务进行拆分，如下图：

![](https://img-blog.csdnimg.cn/78d29fb135ef4efb87cbae7d64af4085.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

  


代码展示
----

### 准备条件

 提醒：由于生产者和消费者的代码大同小异，为了方便，编写一个通用的连接工具类。

```Java
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

### **▸ 发布订阅（**Publish/Subscribe**）**

> 发布一次消息，消费多个。将消息路由给多个队列，多个消费者在不同队列中进行消费。这种模式叫做“发布/订阅”。类似于特别关注，我发布了一篇文章，关注我的粉丝就能看到推文

一个队列对应一个消费者，Publish模式还多了一个exchange(交换机 转发器) ，这时候我们要获取消息，就需要队列绑定到交换机上，交换机把消息发送到队列 , 消费者才能获取队列的消息。

![](https://img-blog.csdnimg.cn/fbe55f14478c4f86ab6d39476edb9c4e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**1\. 生产者：**定义一个生产者，将消息投递到交换机，代码如下

```Java
public class Producer {
 
    public static void main(String[] args) {
        // 1.获取连接
        Connection connection = MQConnectionUtils.getConnection("生产者","test_host");
        Channel channel = null;
        try {
            // 2.通过连接建立通道
            channel = connection.createChannel();
 
            String exchangeName = "my-exchange";
            // 3.通过通道创建交换机 (第一个参数为交换机名称，第二个参数为交换机的类型)
            channel.exchangeDeclare(exchangeName, "fanout");
 
            // 4.发送消息到交换机
            String message = "你好 梅花十三！";
            channel.basicPublish(exchangeName, "", null, message.getBytes());
            System.out.println("消息生产成功！");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            MQConnectionUtils.close(connection,channel);
        }
    }
}
```

**2\. 多个消费者：**我们定义2个队列绑定到该交换机，同时也是2个消费者进行对消息的消费，为了投机取巧，直接用消费者类实现Runnable接口，主函数创建2个线程模拟2个消费者，如下

```Java
public class Consumer implements Runnable{
 
    public static void main(String[] args) {
        // 定义2个线程，线程名称就用队列名称（投机取巧，避免写2个消费者实例，代码一样只是绑定的队列要不同）
        new Thread(new Consumer(),"queue1").start();
        new Thread(new Consumer(),"queue2").start();
    }
 
    public void run() {
        final String name = Thread.currentThread().getName();
        
        // 1.获取连接
        Connection connection = MQConnectionUtils.getConnection("生产者","test_host");
 
        Channel channel = null;
        try {
            // 2.通过连接建立通道
            channel = connection.createChannel();
 
            // 3.通过通道创建队列
            /**
             * @Params1  队列名称
             * @Params2  是否持久化    true：持久化，该队列将在服务器重启后依然继续存在
             * @Params3  是否独占队列  true：独占，仅限于此连接
             * @Params4  自动删除（最后一条消息被消费完毕后，是否把队列自动删除）
             * @Params5  队列的其他属性（构造参数）
             *
             *  面试题：所谓持久化即消息存盘，非持久化会存盘吗？ 回答：会存盘，但会随着服务器宕机而丢失
             */
            channel.queueDeclare(name, true, false, false, null);
 
            // 4.绑定交换机和队列的关系
            /**
             * @Params1  队列名称
             * @Params2  需绑定的交换机名称
             * @Params3  路由key，用于direct或者topic模式，通过某个routingKey绑定交换机
             */
            channel.queueBind(name,"my-exchange","");
 
 
            // 5.消费消息
            /**  @param1：队列名称
             *   @param2：是否自动应答  true：是，消息一旦被消费成功，消息则从队列中删除
             *   @param3：消息送达时的回调
             *   @param4：消费者被取消时的回调
             */
            channel.basicConsume(name,true, new DeliverCallback() {
                public void handle(String consumerTag, Delivery message) throws IOException {
                    System.out.println("从"+name+"队列中接收消息成功！内容：" + new String(message.getBody(), "UTF-8"));
                }
            }, new CancelCallback() {
                public void handle(String consumerTag) throws IOException {
                    System.out.println("接收消息失败。。。。。");
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**!! 注意：**虽然我们上述也说了，如果消息发送到了一个没有绑定队列的交换机时，消息就会丢失！但由于交换机在生产者类创建的（也可以在消费者类创建），因此我们先必须启动Producer类，使其创建交换机（第一次启动的消息会丢失，因为交换机没有绑定队列）

**3\. 启动：**启动顺序为 Producer类——>消费者类（Consumer类的main函数）——>Producer类

**4\. 运行结果**

 a）producer第二次启动，消息生产成功

![](https://img-blog.csdnimg.cn/8d938718db6143e1a6152126b6da80da.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

b）切换至Consumer运行面板，可以看到2个消费者，从2个队列中进行了消息的消费

![](https://img-blog.csdnimg.cn/fb3339f9870b4141b9b2a70f58233b14.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**✦ 结论：**创建一个交换机my-exchange，将类型设置为fanout广播模式，创建2个队列，分别是 queue1、queue2并进行绑定该交换机，交换机在收到生产者的消息后，会将消息路由到其下绑定的2个队列中，2个队列中存储的消息的内容都是一样的，多个消费者到不同的队列中进行消费。