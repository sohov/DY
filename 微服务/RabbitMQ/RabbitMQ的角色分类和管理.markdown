---
layout:  post
title:   RabbitMQ的角色分类和管理
date:   2-03-24 09:25:43 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq

---

## RabbitMQ角色分类

总共分为5种角色，每个角色对应不同的权限信息

### 

### 1. none

>啥也干不了，也无法登陆到图形化界面

不能访问 management plugin



### 2. management 普通管理员

><span style="color:#494949;">相当于个人中心，只查看自己的相关节点信息</span>

- 列出自己可以通过AMQP登陆的虚拟机 
- 查看自己的虚拟机节点virtual hosts的queues，exchange 和bindings信息 
- 查看和关闭自己的channels 和 connections 
- 查看有关自己的虚拟机节点virtual hosts的统计信息，包括其他用户在该节点virtual hosts的活动信息



### 3. policymaker 策略制定者

><span style="color:#494949;">在个人中心基础上，可以管理 (创建、删除) 自己的虚拟机节点和参数信息</span>

- 包含management所有权限 
- 查看和创建和删除自己的virtual hosts所属的policies 和 parameters信息

### 

### 4. monitoring

>管理员一样，除了看自己的还是看别人的，但是只能看，不能操作别人的

- 包含management所有权限 
- 罗列出所有virtual hosts，包括不能登陆的virtual hosts 
- 查看其他用户的connections和channels使用情况 
- 查看所有的virtual hosts的全局统计信息

### 

### 5. administrator 超级管理员

>顶级管理员，可登陆控制台、查看所有信息、可对 rabbitmq进行管理 （全部）

- 最高权限 
- 可以创建和删除virtual hosts 
- 可以查看，创建和删除users 
- 查看创建permisssions 
- 关闭所有用户的connections



## 

## RabbitMQ的用户管理

```java
# 添加用户
sudo rabbitmqctl add_user [用户名]  [密码]  

# 授权用户为超级管理员
sudo rabbitmqctl set_user_tags [用户名] administrator  

# 修改密码
sudo rabbitmqctl change_password  [用户名] [新密码]   

# 删除用户
sudo rabbitmqctl delete_user [用户名] 

# 查看所有用户
sudo rabbitmqctl list_users
```

**!! 注意：**使用rabbitmqctl 的命令时会提示 rabbitmqctl: command not found，这是因为没有配置rabbitmq的环境变量！





### ▎举例说明：添加用户

```java
# 添加一个用户，用户名： wpf  密码：123456
sudo rabbitmqctl add_user wpf 123456

# 给wpf用户授权为超级管理员角色
sudo rabbitmqctl set_user_tags wpf administrator
```

**1. 添加用户 和 设置角色标签**


![img](https://img-blog.csdnimg.cn/25df1b3d9ae446b596f1ed3a146de223.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**2. 查看用户：**使用“guest”来宾用户登陆图形化界面，查看admin菜单栏可以查看所有用户


![img](https://img-blog.csdnimg.cn/0a15fa9c970241629efdb24248e0e080.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**☁ 思考：**眼尖的伙伴可能已经看到了，在 “ Can access virtual hosts” 这栏中，我们刚刚添加的用户wpf 没有权限，那么这个属性是啥意思呢？


![img](https://img-blog.csdnimg.cn/dcc9f89a42b348ba9af76087c403faea.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16) &nbsp;

### 

## 什么是 VirtualHost ？

>virtual host 虚拟主机。它是 RabbitMQ 分配权限的最小细粒度。

### 基本概念

安装一个 RabbitMQ 服务器，每一个 RabbitMQ 服务器都能创建出许多虚拟的消息服务器，这些虚拟的消息服务器就是我们所说的虚拟主机（virtual host），一般简称为 vhost。


![img](https://img-blog.csdnimg.cn/53dd5abf0afe4281b4103b66cb3c5425.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

本质上，每一个 vhost 都是一个独立的小型 RabbitMQ 服务器，vhost 中会有自己的消息队列、消息交换机以及相应的绑定关系（exchanges，queues，bingdings）等等，并且拥有自己独立的权限。不同的 vhost 中的队列和交换机不能互相绑定，这样既能保证运行安全又能避免命名冲突。

><strong>大白话：</strong>如果exchangeA 和queueA 只能让用户A访问，exchangeB 和queueB 只能让用户B访问，要达到这种需求，只能为exchangeA 和queueA创建一个vhostA，为exchangeB 和queueB 创建vhostB。



### VirtualHost 的命令操作

```java
# 添加vhost，名为：myvhost
sudo rabbitmqctl add_vhost myvhost

# 删除, 删除一个vhost，与这个 vhost 相关的消息队列、交换机以及绑定关系等，统统都会被删除。
sudo rabbitmqctl delete_vhost myvhost

# 查看所有
sudo rabbitmqctl list_vhosts
```



### VirtualHost的操作权限隔离机制

>virtual host只是起到一个命名空间的作用，'<strong><span style="color:#be191c;">/</span></strong>' 是系统默认的vritual_host。

如下图，这里创建了2个用户，其中guest是默认的用户，都是超级管理员，且管理不同的vhost


![img](https://img-blog.csdnimg.cn/270a6a9d380e404598e6b32530b57e6b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



1. 首选查看host 列表，重点关注用户wpf2的vhost管理权限


![img](https://img-blog.csdnimg.cn/b39ffa2d32d14ad2a1bb349723bedcb0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

&nbsp;2. 删除不属于它vhost下的交换机时，会提示没有操作权限


![img](https://img-blog.csdnimg.cn/0ec39f44d81d44ec944609879d56e9dc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 



## VirtualHost 用户授权与删除

>在用户管理中，我们刚刚添加了一个wpf用户，是没有任何vhost的管理权限的


![img](https://img-blog.csdnimg.cn/d691301e5e5747ddbf229f48e59c3c40.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



### ▎添加授权：为某个用户添加某个 vhost 的管理权限

><strong><span style="color:#be191c;">!! </span>注意：</strong>RabbitMQ会缓存每个connection或channel的权限验证结果、因此权限发生变化后需要重连才能生效。

1. 使用如下命令进行授权

```java
# 格式 （配置，读，写）
set_permissions [-p <vhost>] <user> <conf> <write> <read>

# 示例：给“wpf”用户授权“test_host”下的全部资源的配置和读写权限
sudo rabbitmqctl set_permissions -p test_host wpf ".*" ".*" ".*"
```




![img](https://img-blog.csdnimg.cn/7a9987cfcf5e440388eb1ae3dd082087.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

**☛ 补充说明：最后面三个 ".*" 含义分别如下：**

- 用户在所有资源上都拥有可配置权限（创建/删除消息队列、创建/删除交换机等）。 
- 用户在所有资源上都拥有写权限（发消息）。 
- 用户在所有资源上都拥有读权限（消息消费，清空队列等）。



2. 查看授权结果


![img](https://img-blog.csdnimg.cn/6e9fcb5892f5417a91b840a61ae4109f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### ▎清除权限：清除某个用户的某个 vhost管理权限

1. 清除用户wpf 管理的 test_host权限


![img](https://img-blog.csdnimg.cn/6e02f02495ac41518741a46de5ea6977.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

2. 查看授权结果&nbsp;


![img](https://img-blog.csdnimg.cn/9f97fe23692c4e9ca7c3b4a64a6da479.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)







