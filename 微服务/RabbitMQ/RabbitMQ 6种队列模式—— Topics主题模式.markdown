
>路由模式是指定固定的路由键 routingKey，而主题模式可以模糊匹配路由键 routingKey。

## 

## 主题模式

>topics 模式支持模糊匹配RoutingKey，就像是sql中的 like子句模糊查询，而路由模式等同于sql中的where子句等值查询

topic&nbsp;交换机背后的路由算法类似于&nbsp;direct&nbsp;交换，使用特定路由键发送的消息将被传递到使用匹配绑定键绑定的所有队列。


![img](https://img-blog.csdnimg.cn/img_convert/8a0e92775bb79ef8717d56560f4d6ac2.png)

如上图，主题模式不能具有任意的 routingKey，必须由一个英文句点“.”分隔的字符串（分割符），比如 “fruit.orange.mango”。







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

### ▸ topic主题方式

>通过模糊路由到队列。该方式的Routing key必须具有固定格式：以 . 间隔的一串单词，比如：quick.orange.rabbit，Routing key 最多不能超过255byte。

交换机和队列的Binding key用通配符来表示，有两种语法：

- *可以替代一个单词； 
- #可以替代 0 或多个单词；



>例如<span style="color:#be191c;"> #.com.#</span><br> #可以表示0级或多级。xx.com、com.xx、com、xx.com.xx.xx、xx.xx.com.xx都可以




![img](https://img-blog.csdnimg.cn/b2bd2b8fa60049dc995aa802393c7998.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>上图中，队列1 与交换机的 绑定kye 为 “ <span style="color:#be191c;">* .orange. *&nbsp;</span>”，当消息的Routing key为三个单词，且中间的单词为 orange 时，消息将进入队列1 。



**1. 生产者：**建立一个生产者producer类，创建交换机、队列等

```java
public class Producer {

    public static void main(String[] args) {

        // 1.获取连接
        Connection connection = MQConnectionUtils.getConnection("生产者","test_host");

        Channel channel = null;
        try {
            // 2.通过连接建立通道
            channel = connection.createChannel();


            // 3.通过通道创建一个交换机my-exchange
            /** @params1 ：交换机名称
             *  @params2 ：交换机类型（topic主题模式）
             *  @params3 ：是否持久化 true：是，服务器重启交换机不会消失
             */
            String exchangeName = "my-exchange";
            channel.exchangeDeclare(exchangeName, "topic",true);

            // 4.创建第一个队列：my-queue1
            String queue1 = "my-queue1";
            channel.queueDeclare(queue1, true, false, false, null);
            // 5. 绑定交换机与队列的关系，指定一个绑定的key
            /** @params1 ：要绑定的队列
             *  @params2 ：要绑定到哪个交换机
             *  @params3 ：交换机和队列之间绑定了一个key，这个key就是BindingKey
             */
            String BindingKey = "*.orange.*"; // BindingKey：交换机与队列之间绑定的key
            channel.queueBind(queue1,exchangeName,BindingKey);


            // 创建第二个队列：my-queue2 （同上述4步）
            String queue2 = "my-queue2";
            channel.queueDeclare(queue2, true, false, false, null);
            // 绑定交换机与队列my-queue2 （同上述5步）
            // 主题模式的交换机，同一个队列是可以绑定多个BindingKey的，这里给队列my-queue2绑定2个key
            channel.queueBind(queue2,exchangeName,"*.*.rabbit");
            channel.queueBind(queue2,exchangeName,"#.lazy");


            // 6.发送消息给队列
            /** @params1 ：消息发送到哪个交换机
             *  @params2 ：路由key，这个交换机可能绑定了很多队列，但这条消息我不想发给该交换机下全部的队列
             *             那么我需要指定一个RoutingKey，用来识别消息最终进入哪个队列
             *   ————上述队列与交换机的绑定中有指定BindingKey，只有消息的RoutingKey与BindingKey模糊匹配上，交换机才会把消息发给该队列
             *  @params3 ：消息内容
             */
            String routingKey = "my.lazy"; // 投递消息时指定的RoutingKey
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


![img](https://img-blog.csdnimg.cn/4bcbd4e0969647f8afba78631f107f42.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**3. 查看交换机、队列信息**


![img](https://img-blog.csdnimg.cn/5e15f58efe5a486f83c8523e66cd8d55.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**4. 查看消息投递结果**


![img](https://img-blog.csdnimg.cn/8badb271ddfe4d9d8d727b414661fd68.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**✦ 消息投递解析**

>在代码第6步中，投递消息时指定的&nbsp;routingKey = "<span style="color:#be191c;">my.lazy</span>"

①. 队列my-queue1与交换机绑定的key是*.orange.*，星号* 表示至少，且只能占位一个单词。代表进入该队列的消息，指定的routingKey只能是3个单词组成，单词与单词之间用*****间隔符，且中间的单词是orange，很明显队列my-queue1不满足

②.&nbsp;my-queue1与交换机绑定了2个key，分别是：*.*.rabbit、#.lazy，第一个key=*.*.rabbit ，与第一个队列相似，消息投递指定的routingKey只能是3个单词组成，且最后一个单词为rabbit，很明显也不符合投递。第二个key= #.lazy ，由于#号表示，占位0个或多个，也就是说，不管routingKey是 lazy 还是 xx.xx.lazy 都是符合的，但是lazy单词后面不能再有单词，比如 lazy.xx...







