## 01- 你们项目中哪里用到了Redis ? 

在我们的项目中很多地方都用到了Redis , Redis在我们的项目中主要有三个作用 : 

1. 使用Redis做热点数据缓存/接口数据缓存
2. 使用Redis存储一些业务数据 , 例如 : 验证码 , 用户信息 , 用户行为数据 , 数据计算结果 , 排行榜数据等
3. 使用Redis实现分布式锁 , 解决并发环境下的资源竞争问题

## 02- Redis的常用数据类型有哪些 ?

Redis 有 5 种基础数据结构，它们分别是：string(字符串)、list(列表)、hash(字典)、set(集 合) 和 zset(有序集合)



## 03- Redis的数据持久化策略有哪些 ? 

Redis 提供了两种方式，实现数据的持久化到硬盘。
1. RDB 持久化(全量)，是指在指定的时间间隔内将内存中的数据集快照写入磁盘。
2. AOF持久化(增量)，以日志的形式记录服务器所处理的每一个写、删除操作

> RDB和AOF一起使用, 在Redis4.0版本支持混合持久化方式 ( 设置 aof-use-rdb-preamble yes )

## 04- Redis的数据过期策略有哪些 ? 

1. **惰性删除** ：只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。

   > 数据到达过期时间，不做处理。等下次访问该数据时，我们需要判断
   >
   > 1. 如果未过期，返回数据
   > 2. 发现已过期，删除，返回nil

2. **定期删除** ： 每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

   > 默认情况下 Redis 定期检查的频率是每秒扫描 10 次，用于定期清除过期键。当然此值还可以通过配置文件进行设置，在 redis.conf 中修改配置“hz”即可，默认的值为**hz 10** 
   >
   > 定期删除的扫描并不是遍历所有的键值对，这样的话比较费时且太消耗系统资源。Redis 服务器采用的是随机抽取形式，每次从过期字典中，取出 20 个键进行过期检测，过期字典中存储的是所有设置了过期时间的键值对。如果这批随机检查的数据中有 25% 的比例过期，那么会再抽取 20 个随机键值进行检测和删除，并且会循环执行这个流程，直到抽取的这批数据中过期键值小于 25%，此次检测才算完成
   >
   > Redis 服务器为了保证过期删除策略不会导致线程卡死，会给过期扫描增加了最大执行时间为 25ms      

定期删除对内存更加友好，惰性删除对 CPU 更加友好。两者各有千秋，所以 Redis 采用的是 **定期删除+惰性删除** 



## 05- Redis的数据淘汰策略有哪些 ? 

Redis 提供 8 种数据淘汰策略：

**淘汰易失数据(具有过期时间的数据)**

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最不经常使用的数据淘汰
3. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
4. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

**淘汰全库数据**

1. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）
2. **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key
3. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰

**不淘汰**

1. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

## 06- 你们使用Redis是单点还是集群 ? 哪种集群 ?

我们Redis使用的是哨兵集群 ,  一主二从 ,  三个哨兵 , 三台Linux机器



## 07- Redis集群有哪些方案, 知道嘛 ? 

我所了解的Redis集群方案 

1. 主从复制集群 :  读写分离, 一主多从 , 解决高并发读的问题
2. 哨兵集群 : 主从集群的结构之上 , 加入了哨兵用于监控集群状态 , 主节点出现故障, 执行主从切换 , 解决高可用问题
3. Cluster分片集群 : 多主多从 , 解决高并发写的问题, 以及海量数据存储问题 , 每个主节点存储一部分集群数据

## 08- 什么是 Redis 主从同步？

Redis 的主从同步(replication)机制，允许 Slave 从 Master 那里，通过网络传输拷贝到完整的数据备份，从而达到主从机制。

主数据库可以进行读写操作，当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的，并接收主数据库同步过来的数据。一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。

![image-20220527093045515](assets/image-20220527093045515.png)

> 主从数据同步主要分二个阶段  : 
>
> **第一阶段 : 全量复制阶段**
>
> - slave节点请求增量同步
> - master节点判断replid，发现不一致，拒绝增量同步
> - master将完整内存数据生成RDB，发送RDB到slave
> - slave清空本地数据，加载master的RDB
>
> **第二阶段 : 增量复制阶段**
>
> - master将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给slave
> - slave执行接收到的命令，保持与master之间的同步



## 09- Redis分片集群中数据是怎么存储和读取的 ? 

Redis集群采用的算法是哈希槽分区算法。Redis集群中有16384个哈希槽（槽的范围是 0 -16383，哈希槽），将不同的哈希槽分布在不同的Redis节点上面进行管理，也就是说每个Redis节点只负责一部分的哈希槽。在对数据进行操作的时候，集群会对使用CRC16算法对key进行计算并对16384取模（slot = CRC16(key)%16383），得到的结果就是 Key-Value 所放入的槽，通过这个值，去找到对应的槽所对应的Redis节点，然后直接到这个对应的节点上进行存取操作



## 10- 你们用过Redis的事务吗 ? 事务的命令有哪些 ? 

Redis 作为 NoSQL 数据库也同样提供了事务机制。在 Redis 中，MULTI / EXEC / DISCARD / WATCH 这四个命令事务的相关操作命令

我们在开发过程中基本上没有用到过Redis的事务

## 11- Redis 和 Memcached 的区别有哪些？

1.  Redis 提供复杂的数据结构，丰富的数据操作 , Memcached 仅提供简单的字符串。
2. Redis原生支持集群模式 , Memcached不支持原生集群
3. 
    Memcached 不支持持久化存储，重启时，数据被清空,  Redis 支持持久化存储，重启时，可以恢复已持久化的数据

## 12- Redis的内存用完了会发生什么？

如果达到设置的上限，Redis 的写命令会返回错误信息（ 但是读命令还可以正常返回。） 
也可以配置内存淘汰机制， 当 Redis 达到内存上限时会冲刷掉旧的内容。



## 13- Redis和Mysql如何保证数据⼀致?

1. 先更新Mysql，再更新Redis，如果更新Redis失败，可能仍然不⼀致

2. 先删除Redis缓存数据，再更新Mysql，再次查询的时候在将数据添加到缓存中

  > 这种⽅案能解决1 ⽅案的问题，但是在⾼并发下性能较低，⽽且仍然会出现数据不⼀致的问题，
  > ⽐如线程1删除了 Redis缓存数据，正在更新Mysql，
  > 此时另外⼀个查询再查询，那么就会把Mysql中⽼数据⼜查到 Redis中

3. 使用MQ异步同步, 保证数据的最终一致性



我们项目中会根据业务情况 , 使用不同的方案来解决Redis和Mysql的一致性问题 : 

1. 对于一些一致性要求不高的场景  , 不做处理

   > 例如 : 用户行为数据 , 我们没有做一致性保证 , 因为就算不一致产生的影响也很小

2. 对于时效性数据  , 设置过期时间

   > 例如  : 接口缓存数据 ,  我们会设置缓存的过期时间为 60S , 那么可能会出现60S之内的数据不一致, 60S后缓存过期, 重新从数据库加载就一致了

3. 对于一致性要求比较高但是时效性要求不那么高的场景 , 使用MQ不断发送消息完成数据同步直到成功为止

   > 例如 : 首页广告数据 , 首页推荐数据
   >
   > 数据库数据发生修改----> 发送消息到MQ  -----> 接收消息更新缓存
   >
   > 消息不丢失/重复消费 : 消息状态表/消息消费表

4. 对于一致性和时效性要求都比较高的场景 , 使用分布式事务 , Seata的TCC模式

   > 很少用

   

## 14- 什么是缓存穿透 ? 怎么解决 ?


缓存穿透是指查询一条数据库和缓存都没有的一条数据，就会一直查询数据库，对数据库的访问压力就会增大，缓存穿透的解决方案

有以下2种解决方案 ： 

- 缓存空对象：代码维护较简单，但是效果不好。 
- 布隆过滤器：代码维护复杂，效果很好





## 15- 什么是缓存击穿 ? 怎么解决 ?

缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大

解决方案 : 

- 热点数据提前预热
- 设置热点数据永远不过期。
- 加锁 , 限流

## 16- 什么是缓存雪崩 ? 怎么解决 ?

缓存雪崩/缓存失效 指的是大量的缓存在同一时间失效，大量请求落到数据库 导致数据库瞬间压力飙升。

造成这种现象的 原因是，key的过期时间都设置成一样了。

解决方案是，key的过期时间引入随机因素

## 17- 数据库有1000万数据 ,Redis只能缓存20w数据, 如何保证Redis中的数据都是热点数据 ?

配置Redis的内容淘汰策略为LFU算法 , 这样会把使用频率较低的数据淘汰掉 , 留下的数据都是热点数据







## 18- Redis分布式锁如何实现 ? 

Redis分布式锁主要依靠一个`SETNX`指令实现的 ,  这条命令的含义就是“SET if Not Exists”，即不存在的时候才会设置值。

只有在key不存在的情况下，将键key的值设置为value。如果key已经存在，则SETNX命令不做任何操作。

这个命令的返回值如下。

- 命令在设置成功时返回1。
- 命令在设置失败时返回0。

假设此时有线程A和线程B同时访问临界区代码，假设线程A首先执行了SETNX命令，并返回结果1，继续向下执行。而此时线程B再次执行SETNX命令时，返回的结果为0，则线程B不能继续向下执行。只有当线程A执行DELETE命令将设置的锁状态删除时，线程B才会成功执行SETNX命令设置加锁状态后继续向下执行

```
Boolean isLocked = stringRedisTemplate.opsForValue().setIfAbsent(PRODUCT_ID, "binghe");
```

当然我们在使用分布式锁的时候也不能这么简单, 会考虑到一些实际场景下的问题 , 例如 : 

1. 死锁问题

   > 在使用分布式锁的时候, 如果因为一些原因导致系统宕机, 锁资源没有被释放, 就会产生死锁
   >
   > 解决的方案 :  上锁的时候设置锁的超时时间 
   >
   > ```
   > Boolean isLocked = stringRedisTemplate.opsForValue().setIfAbsent(PRODUCT_ID, "binghe", 30, TimeUnit.SECONDS);
   > ```

2. 锁超时问题

   > 如果业务执行需要的时间,  超过的锁的超时时间 , 这个时候业务还没有执行完成, 锁就已经自动被删除了
   >
   > 其他请求就能获取锁, 操作这个资源 , 这个时候就会出现并发问题 , 解决的方案 : 
   >
   > 1. 引入Redis的watch dog机制, 自动为锁续期
   > 2. 开启子线程 , 每隔20S运行一次, 重新设置锁的超时时间

3. 归一问题

   > 如果一个线程获取了分布式锁,  但是这个线程业务没有执行完成之前 , 锁被其他的线程删掉了 , 又会出现线程并发问题 , 这个时候就需要考虑归一化问题
   >
   > 就是一个线程执行了加锁操作后，后续必须由这个线程执行解锁操作，加锁和解锁操作由同一个线程来完成。
   >
   > 为了解决只有加锁的线程才能进行相应的解锁操作的问题，那么，我们就需要将加锁和解锁操作绑定到同一个线程中，可以使用ThreadLocal来解决这个问题 , 加锁的时候生成唯一标识保存到ThreadLocal  , 并且设置到锁的值中 , 释放锁的时候, 判断线程中的唯一标识和锁的唯一标识是否相同, 只有相同才会释放
   >
   > ```java
   > public class RedisLockImpl implements RedisLock{
   >     @Autowired
   >     private StringRedisTemplate stringRedisTemplate;
   >     
   >     private ThreadLocal<String> threadLocal = new ThreadLocal<String>();
   >     
   >     @Override
   >     public boolean tryLock(String key, long timeout, TimeUnit unit){
   >         String uuid = UUID.randomUUID().toString();
   >         threadLocal.set(uuid);
   >         return stringRedisTemplate.opsForValue().setIfAbsent(key, uuid, timeout, unit);
   >     }
   >     @Override
   >     public void releaseLock(String key){
   >         //当前线程中绑定的uuid与Redis中的uuid相同时，再执行删除锁的操作
   >         if(threadLocal.get().equals(stringRedisTemplate.opsForValue().get(key))){
   >           stringRedisTemplate.delete(key);   
   >         }
   >     }
   > }
   > ```

4. 可重入问题

   > 当一个线程成功设置了锁标志位后，其他的线程再设置锁标志位时，就会返回失败。
   >
   > 还有一种场景就是在一个业务中, 有个操作都需要获取到锁, 这个时候第二个操作就无法获取锁了 , 操作会失败
   >
   > 例如 : 下单业务中, 扣减商品库存会给商品加锁, 增加商品销量也需要给商品加锁 , 这个时候需要获取二次锁
   >
   > 第二次获取商品锁就会失败  , 这就需要我们的分布式锁能够实现可重入
   >
   > 实现可重入锁最简单的方式就是使用计数器 , 加锁成功之后计数器  +  1 , 取消锁之后计数器 -1  , 计数器减为0  , 真正从Redis删除锁
   >
   > ```java
   > public class RedisLockImpl implements RedisLock{
   >     @Autowired
   >     private StringRedisTemplate stringRedisTemplate;
   >     
   >     private ThreadLocal<String> threadLocal = new ThreadLocal<String>();
   >     
   >     private ThreadLocal<Integer> threadLocalInteger = new ThreadLocal<Integer>();
   >     
   >     @Override
   >     public boolean tryLock(String key, long timeout, TimeUnit unit){
   >         Boolean isLocked = false;
   >         if(threadLocal.get() == null){
   >             String uuid = UUID.randomUUID().toString();
   >          threadLocal.set(uuid);
   >             isLocked = stringRedisTemplate.opsForValue().setIfAbsent(key, uuid, timeout, unit);
   >         }else{
   >             isLocked = true;   
   >         }
   >         //加锁成功后将计数器加1
   >         if(isLocked){
   >             Integer count = threadLocalInteger.get() == null ? 0 : threadLocalInteger.get();
   >             threadLocalInteger.set(count++);
   >         }
   >         return isLocked;
   >     }
   >     
   >     @Override
   >     public void releaseLock(String key){
   >         //当前线程中绑定的uuid与Redis中的uuid相同时，再执行删除锁的操作
   >         if(threadLocal.get().equals(stringRedisTemplate.opsForValue().get(key))){
   >             Integer count = threadLocalInteger.get();
   >             //计数器减为0时释放锁
   >             if(count == null || --count <= 0){
   >               stringRedisTemplate.delete(key);      
   >             }
   >         }
   >     }
   > }
   > ```

5. 阻塞与非阻塞问题

   > 在使用分布式锁的时候 , 如果当前需要操作的资源已经加了锁, 这个时候会获取锁失败, 直接向用户返回失败信息 , 用户的体验非常不好 , 所以我们在实现分布式锁的时候, 我们可以将后续的请求进行阻塞，直到当前请求释放锁后，再唤醒阻塞的请求获得分布式锁来执行方法。
   >
   > 具体的实现就是参考自旋锁的思想, 获取锁失败自选获取锁, 直到成功为止 , 当然为了防止多条线程自旋带来的系统资料消耗, 可以设置一个自旋的超时时间 , 超过时间之后, 自动终止线程 , 返回失败信息
   >
   > ![image-20220527110835621](assets/image-20220527110835621.png)

## 19- 你的项目中哪里用到了分布式锁 

在我最近做的一个项目中 , 我们在任务调度的时候使用了分布式锁

早期我们在进行定时任务的时候我们采用的是SpringTask实现的 , 在集群部署的情况下, 多个节点的定时任务会同时执行 , 造成重复调度 , 影响运算结果, 浪费系统资源

这里为了防止这种情况的发送, 我们使用Redis实现分布式锁对任务进行调度管理 , 防止重复任务执行



后期因为我们系统中的任务越来越多 , 执行规则也比较多 , 而且单节点执行效率有一定的限制 , 所以定时任务就切换成了XXL-JOB , 系统中就没有再使用分布式锁了

