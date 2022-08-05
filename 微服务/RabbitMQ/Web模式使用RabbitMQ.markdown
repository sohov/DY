---
layout:  post
title:   Web模式使用RabbitMQ
date:   2-03-22 09:40:40 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-RabbitMQ
-rabbitmq
-java
-分布式

---

启动rabbitmq服务后，登陆网址：http://localhost:15672


![img](https://img-blog.csdnimg.cn/fa051eb18f2d4423bccc7914ba03edf7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### ▎添加队列




![img](https://img-blog.csdnimg.cn/0e99977c26c84739a3a6d489ac8958ac.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



### ▎模拟生产者生产消息




![img](https://img-blog.csdnimg.cn/9467e63cfdf04245bfa8d1cdb95d2bb7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**1、进入交换机tab，点击默认交换机（AMQP default）进入**


![img](https://img-blog.csdnimg.cn/b8c55691a8a945f8bb872891d54d9c8a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





**2、消息发送成功弹框显示，点击关闭**




![img](https://img-blog.csdnimg.cn/b267464b7d2f49e39202bd26e6254535.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



### ▎查看/消费消息




![img](https://img-blog.csdnimg.cn/525a61404f03407585cd9d788355e5d8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**1、获取消息**




![img](https://img-blog.csdnimg.cn/ed786da6f2ec4a01ad8f0f9df59ee3fa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

****

**2、详细内容截图**




![img](https://img-blog.csdnimg.cn/5787802c145f4eeaa03ae80d4377c47d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**3、消息应答模式选择注意⚠️**




![img](https://img-blog.csdnimg.cn/f80c11647f27411fbaa17872b28c16a3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### ▎添加交换机




![img](https://img-blog.csdnimg.cn/a24ace16162a42daaf854bd79b3731e7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





### ▎交换机绑定队列

**⭐️ 方式一：在交换机处选择队列绑定**


![img](https://img-blog.csdnimg.cn/90d9fb8724c746728360a5a4dbfc5fba.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)





**输入需绑定的队列信息**




![img](https://img-blog.csdnimg.cn/51e6b39bc3f446069426cfc1c7542a96.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**查看交换机已绑定的队列，可进行解绑操作**




![img](https://img-blog.csdnimg.cn/2b3d88e66b334540ba2700e8eccdbc71.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)

****

**⭐️ 方式二：在队列处选择交换机绑定**




![img](https://img-blog.csdnimg.cn/55d28511cf3e4eebb33c54f650a819c3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



**进行绑定或者解绑已绑定的交换机**


![img](https://img-blog.csdnimg.cn/9606a06596bc44148b7150ab4a3aacbb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAwrfmooXoirHljYHkuIk=,size_20,color_FFFFFF,t_70,g_se,x_16)



