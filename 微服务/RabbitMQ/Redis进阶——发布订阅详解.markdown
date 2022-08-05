
## 什么是发布订阅？

>Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 的subscribe 命令可以让客户端订阅任意数量的频道， 每当有新信息发送到被订阅的频道时， 信息就会被发送给所有订阅指定频道的客户端。



☛ 下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：




![img](https://img-blog.csdnimg.cn/0045ba42dc8948d2ab873615967e7de6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



☛ 当有新消息通过publish命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：




![img](https://img-blog.csdnimg.cn/80a14db3795e451cafda46d214620da4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



### ☆ 为什么要用发布订阅？

>熟悉消息中间件的同学都知道，针对消息订阅发布功能，市面上很多大厂使用的是<code>kafka</code>、<code>RabbitMQ</code>、<code>ActiveMQ</code>,&nbsp;<code>RocketMQ等这几种，</code>redis的订阅发布功能跟这三者相比，相对轻量，针对数据准确和安全性要求没有那么高可以直接使用，适用于小公司。

redis 的List数据类型结构提供了&nbsp;blpop 、brpop 命令结合 rpush、lpush 命令可以实现消息队列机制，基于双端链表实现的发布与订阅功能

**这种方式存在两个局限性：**

- 不能支持一对多的消息分发。 
- 如果生产者生成的速度远远大于消费者消费的速度，易堆积大量未消费的消息



**◇ 双端队列图解如下：**


![img](https://img-blog.csdnimg.cn/665685f70b754f75acd26c95faa660f2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

✦解析：双端队列模式只能有一个或多个消费者轮着去消费，却不能将消息同时发给其他消费者



**◇ 发布/订阅模式图解如下：**


![img](https://img-blog.csdnimg.cn/33e9c0e96d4e4df881a81b5c9178d30d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

✦解析：redis订阅发布模式，生产者生产完消息通过频道分发消息，给订阅了该频道的所有消费&nbsp;







## 发布/订阅如何使用？

>Redis有两种发布/订阅模式：



**操作命令如下**


 &nbsp;

### 基于频道(Channel)的发布/订阅

>"发布/订阅" 包含2种角色：发布者和订阅者。发布者可以向指定的频道(channel)发送消息；订阅者可以订阅一个或者多个频道(channel)，所有订阅此频道的订阅者都会收到此消息。


![img](https://img-blog.csdnimg.cn/74b90c1ea2774fbdaae124a622dffad7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



- **订阅者订阅频道**subscribe channel [channel ...]

```java
--------------------------客户端1（订阅者） ：订阅频道 ---------------------

# 订阅 “meihuashisan” 和 “csdn” 频道（如果不存在则会创建频道）
127.0.0.1:6379> subscribe meihuashisan csdn 
Reading messages... (press Ctrl-C to quit)

1) "subscribe"    -- 返回值类型：表示订阅成功！
2) "meihuashisan" -- 订阅频道的名称
3) (integer) 1    -- 当前客户端已订阅频道的数量

1) "subscribe"
2) "csdn"
3) (integer) 2

#注意：订阅后，该客户端会一直监听消息，如果发送者有消息发给频道，这里会立刻接收到消息
```



- **发布者发布消息**&nbsp;&nbsp;publish channel message

```java
--------------------------客户端2（发布者）：发布消息给频道 -------------------


# 给“meihuashisan”这个频道 发送一条消息：“I am meihuashisan”
127.0.0.1:6379> publish meihuashisan "I am meihuashisan"
(integer) 1  # 接收到信息的订阅者数量，无订阅者返回0
```



客户端2 (发布者)发布消息给频道后，此时我们再来观察 客户端1 (订阅者)的客户端窗口变化：

```java
# --------------------------客户端1（订阅者） ：订阅频道 -----------------

127.0.0.1:6379> subscribe meihuashisan csdn 
Reading messages... (press Ctrl-C to quit)

1) "subscribe"    -- 返回值类型：表示订阅成功！
2) "meihuashisan" -- 订阅频道的名称
3) (integer) 1    -- 当前客户端已订阅频道的数量

1) "subscribe"
2) "csdn"
3) (integer) 2



 ---------------------变化如下：（实时接收到了该频道的发布者的消息）------------

1) "message"           -- 返回值类型：消息
2) "meihuashisan"      -- 来源（从哪个频道发过来的）
3) "I am meihuashisan" -- 消息内容
```

**命令操作图解如下：**

>注意：如果是先发布消息，再订阅频道，不会收到订阅之前就发布到该频道的消息！


![img](https://img-blog.csdnimg.cn/7d1927e79e334cb0af3f5950e2124d99.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

><strong>注意：</strong>进入订阅状态的客户端，不能使用除了<span style="color:#be191c;"><code><span style="background-color:#f3f3f4;">subscribe</span></code></span>、<span style="color:#be191c;"><code><span style="background-color:#f3f3f4;">unsubscribe</span></code></span>、<span style="color:#be191c;"><code><span style="background-color:#f3f3f4;">psubscribe&nbsp;</span></code></span>和&nbsp;<code><span style="color:#be191c;"><span style="background-color:#f3f3f4;">punsubscribe</span></span>&nbsp;</code>这四个属于"发布/订阅"之外的命令，否则会报错！





### ★ 实现原理

>底层通过字典实现。<span style="color:#be191c;"><code><span style="background-color:#f3f3f4;">pubsub_channels&nbsp;</span></code></span>是一个字典类型，保存订阅频道的信息：<strong>字典的key为订阅的频道， 字典的value是一个链表， 链表中保存了所有订阅该频道的客户端</strong>

```java
struct redisServer { 
  /* General */ 
  pid_t pid; 

  //省略百十行 

  // 将频道映射到已订阅客户端的列表(就是保存客户端和订阅的频道信息)
  dict *pubsub_channels; /* Map channels to list of subscribed clients */ 
}
```

**实现图如下：**


![img](https://img-blog.csdnimg.cn/068c78192c924724a62e2d1739eff0d2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**频道订阅**：订阅频道时先检查字段内部是否存在；不存在则为当前频道创建一个字典且创建一个链表存储客户端id；否则直接将客户端id插入到链表中。

**取消频道订阅**：取消时将客户端id从对应的链表中删除；如果删除之后链表已经是空链表了，则将会把这个频道从字典中删除。

**发布**：首先根据 channel 定位到字典的键， 然后将信息发送给字典值链表中的所有客户端









### 基于模式(pattern)的发布/订阅

>如果有某个/某些模式和该频道匹配，所有订阅这个/这些频道的客户端也同样会收到信息。



**图解**

下图展示了一个带有频道和模式的例子， 其中com.ahead.*频道匹配了com.ahead.juc频道和 com.ahead.thread频道， 并且有不同的客户端分别订阅它们三个，如下图：

>当有信息发送到<span style="color:#be191c;">com.ahead.thread </span>频道时， 信息除了发送给 <strong>client 4 </strong>和 <strong>client 5 </strong>之外， 还会发送给订阅 <span style="color:#be191c;">com.ahead.*&nbsp;&nbsp;</span>频道模式的 <strong>client x </strong>和 <strong>client y</strong>


![img](https://img-blog.csdnimg.cn/1a5f8ecb7c034950952042bc8c53b15a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

✦解析：反之也是，如果当有消息发送给&nbsp;com.ahead.juc频道，消息发送给订阅了 juc 频道的客户端之外，还会发送给订阅了com.ahead.*频道的客户端：&nbsp;**client x 、client y**

>通配符中?表示1个占位符，*表示任意个占位符(包括0)，?*表示1个以上占位符。





- **订阅者订阅频道**psubscribe pattern [pattern ...]

```java
--------------------------客户端1（订阅者） ：订阅频道 ---------------------


#  1. ------------订阅 “a?” "com.*" 2种模式频道--------------
127.0.0.1:6379> psubscribe a? com.*
# 进入订阅状态后处于阻塞，可以按Ctrl+C键退出订阅状态
Reading messages... (press Ctrl-C to quit) 



# 2. ---------------订阅成功-------------------

1) "psubscribe"  -- 返回值的类型：显示订阅成功
2) "a?"          -- 订阅的模式
3) (integer) 1   -- 目前已订阅的模式的数量


1) "psubscribe"
2) "com.*"
3) (integer) 2



# 3. ---------------接收消息 （已订阅 “a?” "com.*" 两种模式！）-----------------

# ---- 发布者第1条命令： publish ahead "hello"
结果：没有接收到消息，匹配失败，不满足 “a?” ，“？”表示一个占位符， a后面的head有4个占位符


# ---- 发布者第2条命令：  publish aa "hello" （满足 “a?”）
1) "pmessage" -- 返回值的类型：信息
2) "a?"       -- 信息匹配的模式：a?
3) "aa"       -- 信息本身的目标频道：aa
4) "hello"    -- 信息的内容："hello"


# ---- 发布者第3条命令：publish com.juc "hello2"（满足 “com.*”, *表示任意个占位符）
1) "pmessage" -- 返回值的类型：信息
2) "com.*"    -- 匹配模式：com.*
3) "com.juc"  -- 实际频道：com.juc
4) "hello2"   -- 信息："hello2"

# ---- 发布者第4条命令： publish com. "hello3"（满足 “com.*”, *表示任意个占位符）
1) "pmessage" -- 返回值的类型：信息
2) "com.*"    -- 匹配模式：com.*
3) "com."     -- 实际频道：com.
4) "hello3"   -- 信息："hello3"
```



- **发布者发布消息**&nbsp;&nbsp;publish channel message

```java
--------------------------客户端2（发布者）：发布消息给频道 -------------------

注意：订阅者已订阅 “a?” "com.*" 两种模式！


# 1. ahead 不符合“a?”模式，?表示1个占位符
127.0.0.1:6379> publish ahead "hello"  
(integer) 0    -- 匹配失败，0:无订阅者


# 2. aa 符合“a?”模式，?表示1个占位符
127.0.0.1:6379> publish aa "hello"      
(integer) 1

# 3. 符合“com.*”模式，*表示任意个占位符
127.0.0.1:6379> publish com.juc "hello2" 
(integer) 1

# 4. 符合“com.*”模式，*表示任意个占位符
127.0.0.1:6379> publish com. "hello3" 
(integer) 1
```



**命令操作图解如下：**


![img](https://img-blog.csdnimg.cn/5bfec453cf6c414cb7b43c17bdff3a9d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### ★ 实现原理

>底层是pubsubPattern节点的链表

```java
struct redisServer {
    //...
    list *pubsub_patterns; 
    // ...
}

// 1303行订阅模式列表结构：
typedef struct pubsubPattern {
    client *client;  -- 订阅模式客户端
    robj *pattern;   -- 被订阅的模式
} pubsubPattern;
```



实现图如下：


![img](https://img-blog.csdnimg.cn/d15e1b37850c4048b030b01afaf61509.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**模式订阅**：新增一个pubsub_pattern数据结构添加到链表的最后尾部，同时保存**客户端ID**。

**取消模式订阅**：从当前的链表pubsub_pattern结构中删除需要取消的pubsubPattern结构。







## 使用小结

订阅者(listener)负责订阅频道(channel)；发送者(publisher)负责向频道发送二进制的字符串消息，然后频道收到消息时，推送给订阅者。

**✦ 使用场景**

- 电商中，用户下单成功之后向指定频道发送消息，下游业务订阅支付结果这个频道处理自己相关业务逻辑 
- 粉丝关注功能 
- 文章推送



**✦ 使用注意**

<li id="dcs-faq-0427017__li984415575911">客户端需要及时消费和处理消息。 客户端订阅了channel之后，如果接收消息不及时，可能导致DCS实例消息堆积，当达到消息堆积阈值（默认值为32MB），或者达到某种程度（默认8MB）一段时间（默认为1分钟）后，服务器端会自动断开该客户端连接，避免导致内部内存耗尽。  
<li id="dcs-faq-0427017__li178449595911">客户端需要支持重连。 当连接断开之后，客户端需要使用subscribe或者psubscribe重新进行订阅，否则无法继续接收消息。  
<li id="dcs-faq-0427017__li208447517592">不建议用于消息可靠性要求高的场景中。 Redis的pubsub不是一种可靠的消息系统。当出现客户端连接退出，或者极端情况下服务端发生主备切换时，未消费的消息会被丢弃。 





