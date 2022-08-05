---
layout:  post
title:   RabbitMQ 内部结构原理介绍
date:   2-03-22 09:40:22 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq
-中间件
-分布式

---

## RabbitMQ简介

>RabbitMQ是一个用Erlang语言开发的、实现了AMQP协议的消息中间件。

AMQP :（Advanced Message Queue，高级消息队列协议）它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制



## 为什么选择RabbitMQ

1. 除了Qpid，RabbitMQ是唯一一个实现了AMQP标准的消息服务器； 
2. 可靠性，RabbitMQ的持久化支持，保证了消息的稳定性； 
3. 高并发，RabbitMQ使用Erlang语言开发，Erlang是一种面向并发的编程语言，天生自带高并发和高可用特性； 
4. 集群部署简单，正是因为Erlang使得RabbitMQ集群部署变的超级简单； 
5. 社区活跃度高；



## RabbitMQ的核心组件

>生产者发送消息给交换机，交换机根据路由规则，将不同的消息路由到不同的队列，消费者订阅/监听队列，当有消息过来时，就立即消费。

**下图是RabbitMQ的结构模型：**


![img](https://img-blog.csdnimg.cn/681fcf603fcb4885b737203ece36f934.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

&nbsp;▎**Broker**

Broker简单理解就是RabbitMQ服务器，图中灰色的整个部分。后面说Broker说的就是RabbitMQ服务器。



### ▎VHost 虚拟主机

>相当于数据库(vhost)一样，本地连接可以创建多个数据库，数据库里有多个表（交换机、队列等等），起到隔离作用。

每个RabbitMQ服务器可以开设多个虚拟主机vhost（图中橘色的部分），每一个vhost本质上是一个mini版的RabbitMQ服务器，拥有自己的 "交换机exchange、绑定Binding、队列Queue"，更重要的是每一个vhost拥有独立的权限机制，这样就能安全地使用一个RabbitMQ服务器来服务多个应用程序，其中每个vhost服务一个应用程序。

每一个RabbitMQ服务器都有一个默认的虚拟主机 "/"，客户端连接RabbitMQ服务时须指定vHost，如果不指定默认连接的就是"/"。



### ▎ConnectionFactory 连接管理器

Connection工厂，负责创建和管理Connection的。



### ▎Connection 连接

无论是生产者还是消费者，都需要和 Broker 建立连接，这个连接就是Connection（看图），是一条 TCP 连接 ，一个生产者或一个消费者与 Broker 之间只有一个Connection，即只有一条TCP连接。



### ▎Exchange 交换机

>用于接受、分配消息

交换机的作用就是根据路由规则，将消息转发到对应的队列上。



### ▎Channel 信道

>消息推送使用的通道

信道是建立在真实的TCP连接内的虚拟连接。AMQP的命令都是通过信道发送出去的，每条信道都会被指派一个唯一ID，不论是发布消息、订阅队列还是接收消息都是通过信道完成的。一个TCP连接下包含多个信道，实现共用TCP、减少TCP创建和销毁的开销。



### ▎Queue 队列

用于存储生产者的消息



### ▎Routing key 路由键

>用于把生产者的数据分配到交换机上

Routing key是消息头的属性，生产者将消息发送到交换机时，会在消息头上携带一个 key，这个 key就是routing key，来指定这个消息的路由规则。等同于数据库中的where条件一样



### ▎Binding

绑定，可理解成一个动词，作用就是把exchange交换机和queue队列按照路由规则绑定起来。



### ▎Binding key 绑定键

>用于把交换机的消息绑定到队列上

在绑定Exchange交换机与Queue队列时，一般会指定一个binding key，生产者将消息发送给Exchange时，消息头上会携带一个routing key，当binding key与routing key相匹配时，消息将会被路由到对应的Queue中。





## 消息是怎么从生产者到消费者的？

首先，生产者发送消息到交换机，消息头上携带一个routing key，通过routing key，交换机就知道该把消息发到哪个队列，随后交换机把消息发送到相应的队列中，再由队列将消息发送给消费者，消费者监听(订阅)某些队列，当有消息过来时，就立即处理消息。


![img](https://img-blog.csdnimg.cn/a9b90aae6e424080b5a76be8b5f2f337.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





## 交换机如何路由消息到队列？

>队列是用来缓存存储消息的，生产者不会直接的向队列发送任何消息。实际上，生产者都不知道将消息交付给哪个队列。

生产者只将消息往exchange交换机中发送。交换机接收到消息之后，再把消息路由给具体队列，最终消费者从队列中获取消息进行消费。

也就是说，交换机一边从生产者中接收消息，一边将消息推到队列。需要注意的是：如果消息发送到了一个没有绑定队列的交换机时，消息就会丢失！ &nbsp;

**☁ 那么交换机是怎么将消息路由到具体队列的呢？**

上面我们说了，生产者发送消息给交换机，消息头上会携带一个routing key，通过routing key，交换机就知道该把消息分发到哪个队列，这些规则都通过exchange类型来定义。RabbitMQ 的交换机有四种类型：fanout、direct、topic、headers。



### ▸ fanout 广播方式

&nbsp; &nbsp;就跟广播一样，会将消息投递给所有绑定在此交换器的队列。


![img](https://img-blog.csdnimg.cn/fbe55f14478c4f86ab6d39476edb9c4e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

### ▸ direct路由方式

在 direct 模式里，交换机和队列之间绑定了一个 key（这个key就是Binding key），只有消息的 Routing key 与Binding key 相同时，交换机才会把消息发给该队列。


![img](https://img-blog.csdnimg.cn/d54a25ca27ae4c8b8019e7e4da197644.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>如上图，消息的Routing key 为 qq 时，消息将进入队列1，Routing key 为 email 或&nbsp; 时，消息将进入队列2。若消息的 key 是其他字符串，被交换机直接遗弃。



同时，交换机也支持多重绑定。不同的队列可以用相同的Binding key与同一交换机绑定。如下图，当消息的Routing key为black时，消息将进入 Q1 和 Q2。


![img](https://img-blog.csdnimg.cn/6630191b46ff476294ecbf51353b489b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### ▸ topic主题方式

通过模糊路由到队列。该方式的Routing key必须具有固定格式：以 . 间隔的一串单词，比如：quick.orange.rabbit，Routing key 最多不能超过255byte。

交换机和队列的Binding key用通配符来表示，有两种语法：

-  * 可以替代一个单词；  
-  # 可以替代 0 或多个单词； 


![img](https://img-blog.csdnimg.cn/b2bd2b8fa60049dc995aa802393c7998.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>上图中，Q1与交换机的 绑定kye 为 "<span style="color:#be191c;"> * .orange. *</span>"，当消息的Routing key为三个单词，且中间的单词为 orange 时，消息将进入 Q1。





### ▸ headers参数方式

不常用，headers交换机是通过Headers头部来将消息映射到队列的，Headers头部携带一个Hash结构，Hash结构中要求携带一个键"x-match"，这个键的Value可以是any或者all，这代表消息携带的Hash是需要全部匹配(all)，还是仅匹配一个键(any)就可以了。相比直连交换机，首部交换机的优势是匹配的规则不被限定为字符串String类型。

>例如，headers类型的交换机，发送消息时，设置x=1参数，满足被投递消息的队列只有 1 和 3

### 





### ▎扩展：默认交换机

default Exchange，默认交换机的名字是空字符串。

>发送消息时不指定交换机的名称，则会发到"默认交换机"上。默认的Exchange不进行Binding操作，任何发送到该Exchange的消息都会被转发到 "Queue名字和Routing key相同的队列"中，如果vhost中不存在和Routing key同名的队列，则该消息会被抛弃。





## 消息发送原理

>必须连接到RabbitMQ才能发布和消费消息，那怎么连接和发送消息的呢？

你的应用程序和Rabbit Server之间会创建一个TCP连接，一旦TCP打开，并通过了认证（即连接前发送给Rabbit服务器连接信息和用户名和密码），认证通过应用程序和Rabbit就创建一条AMQP信道（Channel）

信道是创建在“真实”TCP上的虚拟连接，AMQP命令都是通过信道发送出去的，每个信道都会有一个唯一的ID，不论是发布消息，订阅队列或者介绍消息都是通过信道完成的。


![img](https://img-blog.csdnimg.cn/ce018754a82e4fe591367a3f76467ee2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)







## ✷ 常见问题解答



### 1、RabbitMQ为什么基于channel通道去处理而不是TCP连接？

频繁创建和销毁TCP连接会造成很大开销，操作系统每秒能创建的TCP也是有限的，很快就会遇到系统瓶颈。将TCP连接作为一个长连接，长连接中存在很多信道channel，高并发场景下，一个连接可以创建多个通道，性能很高，又能确保每个连接的私密性

>创建连接需要经过3次握手，关闭连接4次挥手



### 2、为什么使用 RabbitMQ，什么场景使用？

>RabbitMQ核心点：解藕、削峰、异步。异步分发，处理数据能力高效和稳健，解藕

**单体架构的项目所有的业务都堆积到一个项目里面，随着项目的不断发展与前进，单体架构往往不能满足业务需求**

比如，销售生成订单，订单支付成功后，可能需发送短信提醒订单成功，代理商返佣，累加积分等一系列操作，如果该操作是串行化，则订单支付成功后，在发送短信步骤失败了，因为事务的关系从而导致整个订单回滚，产生退款，易丢失客户，严重影响客户体验感。

如果采用异步方式，那么处理方式极可能就是使用线程池异步处理方式，需要自己维护线程池的持久性，高可靠，高可用等等都需自己实现，涉及到持久性，就要考虑把消息进行存盘、写盘，写盘过程中还要考虑磁盘内存资源的转换比，不能全部把消息放到内存中，容易爆掉，爆掉又需要存入磁盘，耦合在代码中，瓜分JVM内存占用，造轮子，费力不讨好！故此引入消息队列



**✸ 线程池存在问题：**

1. 耦合度高 
2. 需自己写线程池维护成本过高 
3. 出现消息可能丢失，需自己做消息补偿 
4. 如何保证消息的可靠性需要自己写 
5. 如果服务器承载不了，需要自己去写高可用


![img](https://img-blog.csdnimg.cn/67f79f7ca1b74aa79e571b9d8203f77f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**➳ RabbitMQ异步消息队列的方式**

1. 完全解藕，用MQ建立桥接 
2. 有独立的线程池和运行模型 
3. 出现消息可能会丢失，MQ有持久化功能 
4. 如何保证消息的可靠性，死信队列和消息转移等 
5. 如果服务器承载不了，需自己去写高可用，HA镜像模型高可用


![img](https://img-blog.csdnimg.cn/d1a5c7abaf3747b68f70b7e07142e08a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

>按照以上约定，用户的响应时间相当于是订单信息写入数据库的时间，也就是50毫秒，注册邮件和发送短信等，写入消息队列后，直接返回，因此写入消息队列的速度很快，基本可忽略，架构改变后，系统的吞吐量提高到每秒20QPS，比串行化提高了3倍，并行提高了两倍

 &nbsp;

