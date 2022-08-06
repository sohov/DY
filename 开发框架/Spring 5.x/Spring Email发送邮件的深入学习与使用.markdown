

>  详细介绍了使用Spring 快速发送邮件的方法。

  **此前我们发送邮件，需要使用JavaMail提供的API来编程：Java EE网络基础(10)—使用JavaMail收发Email电子邮件。Spring Framework 提供了一个实用的程序库，用于发送电子邮件，让程序员免受底层邮件系统细节的影响，并负责代表客户端进行低级资源处理。**







  org.springframework.mail 包是 Spring Framework电子邮件支持的总包。发送电子邮件的中心接口是 MailSender 接口，org.springframework.mail.javamail.JavaMailSender接口继承并扩展了MailSender接口，作为邮件发送器，主要提供了邮件发送接口、透明创建Java Mail的MimeMessage、及邮件发送的配置(如:host/port/username/password…)，我通常使用它的实现类JavaMailSenderImpl。   SimpleMailMessage类是一个简单的值对象，它实现了javaMail的MailMessage，支持封装简单的邮件的属性，比如from和to等属性。此包还包含一系列的受检异常，这些异常是在较低级别的邮件系统异常上提供的更高级别的抽象，而根异常是 MailException。   通常使用org.springframework.mail.javamail.MimeMessageHelper方便的创建 JavaMail 的MimeMessage，可用于支持一些复杂的邮件内容，比如发送一些附件、嵌入图片资源等。   **Spring的JavaMail的API数据Spring Framework的扩展，它位于spring-context-support依赖中，而Spring的API又依赖于sun公司提供的底层JavaMail的API，因此我们至少需要两个依赖：**

```java
<properties>
        <spring-framework.version>5.2.8.RELEASE</spring-framework.version>
        <javax.mail.version>1.6.2</javax.mail.version>
        <junit>4.12</junit>
    </properties>
    <dependencies>
        <!--spring 核心组件所需依赖-->
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context-support -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>
        <!--sun JavaMail 依赖-->
        <!-- https://mvnrepository.com/artifact/com.sun.mail/javax.mail -->
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
            <version>${javax.mail.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
        <!--Spring 测试-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit}</version>
        </dependency>
    </dependencies>
```


  **基础的MailSender 和 SimpleMailMessage的API足以支持发送大部分简单的文本的电子邮件！**

## 2.1 配置

  Spring的对于通用组件类都支持配置式管理，而不需要我们手动创建，因此对于JavaMailSenderImpl和SimpleMailMessage都可以采用配置的方式，把对象的创建交给Spring管理。JavaMailSenderImpl用于发送邮件，是线程安全的，而SimpleMailMessage主要用作一个邮件模版，在使用时，需要使用自己拷贝的对象，否则如果多个线程一起使用，则可能导致线程不安全！   基于XML的配置如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


    <context:property-placeholder location="mail.properties"/>

    <!--配置JavaMailSenderImpl-->
    <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <!--设置邮件服务器主机-->
        <property name="host" value="${mail.sender.host}"/>
        <!--端口号-->
        <property name="port" value="${mail.sender.port}"/>
        <!--用户名-->
        <property name="username" value="${mail.sender.username}"/>
        <!--密码或者授权码-->
        <property name="password" value="${mail.sender.password}"/>
        <!--设置其他属性-->
        <property name="javaMailProperties">
            <props>
                <!--是否需要认证-->
                <prop key="mail.smtp.auth">${mail.sender.auth}</prop>
            </props>
        </property>
    </bean>

    <!--配置一个模版消息，也可以在代码中创建消息-->
    <bean id="templateMessage" class="org.springframework.mail.SimpleMailMessage">
        <!--设置发件人-->
        <property name="from" value="${mail.message.from}"/>
        <!--设置收信人，可以通过传递参数数组设置多个，这里只设置一个-->
        <property name="to" value="${mail.message.to}"/>
        <!--设置抄送人，可以通过传递参数数组设置多个，这里只设置一个-->
        <property name="cc" value="${mail.message.cc}"/>
        <!--设置暗送人，可以通过传递参数数组设置多个，这里只设置一个-->
        <property name="bcc" value="${mail.message.bcc}"/>
        <!--设置邮件主题-->
        <property name="subject" value="${mail.message.subject}"/>
    </bean>

</beans>
```

  基于@Bean注解的JavaConfig配置更简单，这里不再赘述。   mail.properties配置文件：

```java
# 邮件服务器主机，这里选择163邮箱
mail.sender.host=smtp.163.com
# smtp端口号默认就是25
mail.sender.port=25
# 是否需要认证
mail.sender.auth=true
# 用户名
mail.sender.username=xx@163.com
# 密码或者授权码，某些邮箱可能需要开启smtp功能
mail.sender.password=xx


# 发件人
mail.message.from=xx@163.com
# 收件人
mail.message.to=xx@qq.com
# 抄送人
mail.message.cc=xx@yeah.net
# 暗送人
mail.message.bcc=xx@ikang.com
# 主题，这个可以在代码中配置
mail.message.subject=Spring Email
```

## 2.2 测试

  这里使用Spring test来执行测试：

```java
/**
 * @author lx
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring-mail.xml")
public class EmailTest {
   

    /*
     * 在开发中，这些这些可以直接注入到组件类中
     */

    @Resource
    private MailSender mailSender;
    @Resource
    private SimpleMailMessage templateMessage;


    @Test
    public void simpleEmailTest() {
   
        sendEmail("这是一封Spring发送的电子邮件!");
    }

    private void sendEmail(String test) {
   
        //根据模版创建一个线程安全的拷贝对象，并对其进行自定义属性
        SimpleMailMessage msg = new SimpleMailMessage(this.templateMessage);
        //设置正文文本
        msg.setText(test);
        //发送邮件
        mailSender.send(msg);
    }
}
```


  使用 JavaMailSender 和 JavaMail的MimeMesage 以创建更复杂的邮件，例如带有附件的邮件、特殊字符编码或HTML文本等。

## 3.1 使用MimeMessagePreparator

  Spring提供了MimeMessagePreparator回调接口，它有一个prepare方法用于准备JavaMail的MIME信件。   发送一封HTML邮件！

```java
/**
 * @author lx
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring-mail.xml")
public class MimeMessagePreparatorTest {
   

    /*
     * 在开发中，这些这些可以直接注入到组件类中
     */

    @Resource
    private JavaMailSender mailSender;

    @Value("${mail.message.from}")
    private String from;
    @Value("${mail.message.to}")
    private String to;
    @Value("${mail.message.cc}")
    private String cc;
    @Value("${mail.message.bcc}")
    private String bcc;


    @Test
    public void emailTest() {
   
        sendEmail();
    }

    private void sendEmail() {
   
        MimeMessagePreparator mimeMessagePreparator = mimeMessage -> {
   
            //设置发信人
            mimeMessage.setFrom(new InternetAddress(from));
            //设置收信人，可以设置多个，采用参数数组或者","分隔
            mimeMessage.addRecipients(TO, to);
            //设置抄送人，可以设置多个，采用参数数组或者","分隔
            mimeMessage.addRecipients(CC, cc);
            //设置暗送人，可以设置多个，采用参数数组或者","分隔
            mimeMessage.addRecipients(BCC, bcc);
            //设置邮件主题（标题）
            mimeMessage.setSubject("mimeMessagePreparator的HTML邮件");
            //设置邮件内容（正文）和类型
            mimeMessage.setContent("<h2><font color=red>2021加油哦!</font></h2>", "text/html;charset =utf-8");
        };
        mailSender.send(mimeMessagePreparator);
    }
}
```

## 3.2 使用MimeMessageHelper

  大部分电子邮件允许附件和内嵌资源。内嵌资源包括要在邮件中使用但不希望显示为附件的图像或样式表格。   可以看到，上面的MimeMessagePreparator仍然是采用了JavaMail的mimeMessage构建消息，mimeMessage的原始API用来处理内联消息或者附件将会比较复杂，此时可以使用Spring提供的 org.springframe.mail.javamail.MimeMessageHelper，它保护您不必使用冗长的 JavaMail API。使用 MimeMessageHelper创建复杂的 MimeMessage 非常简单！   简单的使用如下：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring-mail.xml")
public class MimeMessageHelperTest {
   

    @Resource
    private JavaMailSender mailSender;

    @Value("${mail.message.from}")
    private String from;
    @Value("${mail.message.to}")
    private String to;
    @Value("${mail.message.cc}")
    private String cc;
    @Value("${mail.message.bcc}")
    private String bcc;

    @Test
    public void emailTest() throws MessagingException {
   

        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setCc(cc);
        helper.setBcc(bcc);
        helper.setSubject("MimeMessageHelper测试!");
        helper.setText("MimeMessageHelper测试!");
        mailSender.send(mimeMessage);
    }
}
```

### 3.2.1 发送附件

```java
@Test
public void attachmentsTest() throws MessagingException {
   
    MimeMessage mimeMessage = mailSender.createMimeMessage();
    //使用 true 标志位表示需要使用多部分消息，即multipart
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
    //设置基本属性
    helper.setFrom(from);
    helper.setTo(to);
    helper.setCc(cc);
    helper.setBcc(bcc);
    helper.setSubject("MimeMessageHelper附件测试!");
    //设置正文
    helper.setText("MimeMessageHelper附件!");

    /*
     * 添加图片附件
     * 相对路径必须找正确
     * 如果不知道路径，那么可以先通过File测试，找到绝对路径前缀，然后再传递相对路径
     * File file = new File("");
     * System.out.println(file.getAbsolutePath());
     */

    //附件图片
    FileSystemResource fileSystemResource = new FileSystemResource("src/main/resources/pic.jpg");

    //添加附件名和附件输入流
    helper.addAttachment("pic.jpg", fileSystemResource);

    /*
     * 发送
     */
    mailSender.send(mimeMessage);
}
```


  结果如下： ![img](https://img-blog.csdnimg.cn/20210116195057708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

### 3.2.2 发送内嵌资源

  使用addInline方法使用指定的的Content-ID将内嵌资源添加到 MimeMessage 中。   添加文本和资源的顺序非常重要。请务必先添加文本，然后添加资源。如果顺序相反，那么不会工作。

```java
@Test
public void inlineResourcesTest() throws MessagingException {
   
    MimeMessage mimeMessage = mailSender.createMimeMessage();
    /*
     * 第二个参数 true 标志位，表示需要使用多部分消息，即multipart
     * 第三个参数 encoding，表示设置消息编码的格式
     */
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true, "utf-8");
    /*
     * 设置基本属性
     */
    helper.setFrom(from);
    helper.setTo(to);
    helper.setCc(cc);
    helper.setBcc(bcc);
    helper.setSubject("MimeMessageHelper的内嵌图片测试");

    /*
     * 设置正文
     *
     * 第二个参数 true 标志位，表示设置正文为HTML格式
     * 其中的图片采用<img src='cid:pic'/>形式，指定需要内嵌的图片的Content-ID
     */
    helper.setText("<html><body><h4><font color='red'>这是一张内嵌图片</font>" +
                    "</h4><img src='cid:pic'/></body></html>",true);

    /*
     * 添加图片附件
     *
     * 相对路径必须找正确
     * 如果不知道路径，那么可以先通过File测试，找到绝对路径前缀，然后再传递相对路径
     * File file = new File("");
     * System.out.println(file.getAbsolutePath());
     */

    //附件图片
    FileSystemResource fileSystemResource = new FileSystemResource("src/main/resources/pic.jpg");

    //添加内嵌图片关联的html的Content-ID，即pic
    helper.addInline("pic", fileSystemResource);

    /*
     * 发送
     */
    mailSender.send(mimeMessage);
}
```


  结果如下： ![img](https://img-blog.csdnimg.cn/2021011619482726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)


  Spring提供的发送email的API大大的简化了开发人员发送邮件所需编写的JavaMail代码。另外，在上面的案例中，我们是直接设置的正文，在实际上企业开发中，通常不会这样显示的创建邮件内容：

1. 在 Java 代码中创建基于 HTML 的电子邮件内容既繁琐又容易出错。 
2. 显示逻辑和业务逻辑之间没有明确的分离。 
3. 更改电子邮件内容的显示结构需要编写 Java 代码、重新编译、重新部署等。

  **通常，解决这些问题的方法是使用邮件内容模板（如 FreeMarker、HTML文件）来定义电子邮件内容的显示结构。这样代码的任务只是将要在电子邮件模板中呈现的数据填充到模版中并发送电子邮件。当电子邮件的内容变得比较复杂时，这绝对是一个最佳实践。**

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

