

本文是在看了b站视频【尚硅谷Spring框架视频教程（spring5源码级讲解）】后，手动实现及理解整理 视频地址： [https://www.bilibili.com/video/BV1Vf4y127N5?spm_id_from=333.999.0.0](https://www.bilibili.com/video/BV1Vf4y127N5?spm_id_from=333.999.0.0)


Spring是轻量级开源JavaEE框架，可以解决企业应用开发的困难性。 有两个核心部分：

1. **IOC**：控制反转， 把创建对象的过程交给Spring去管理 
2. **AOP**： 面向切面编程，不修改源码进行功能增强。

Spring 特点：

1. 方便解耦、简化开发 
2. AOP编程支持 
3. 方便程序测试 
4. 方便和其他框架整合 
5. 方便进行事务操作 
6. 降低API开发难度



![img](https://img-blog.csdnimg.cn/f45e9d2bf0454a85b20f575d9f083730.png)



1.导入IOC核心jar包 ![img](https://img-blog.csdnimg.cn/d49b61ef91e842d0984b1fe38ba69a69.png) 2.创建对象类

```java
public class Book {
   
    private String bName;
    private String bAuthor;
    
    public void testDemo() {
   
        System.out.println(bName + "::" + bAuthor);
    }
}
```

3.创建bean1.xml文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
<!--    配置对象创建-->
    <bean id="book" class="cn.lych4.spring5.bean.Book"></bean>
</beans>
```

4.创建对象

```java
//1. 加载spring 配置文件
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        //2.获取配置的对象
        Book book = context.getBean("book", Book.class);
        System.out.println(book);
        book.testDemo();
```


## 什么是IOC？

控制反转（IOC)， 就算把对象创建和对象之间的调用过程交给Spring容器进行管理 IOC的目的是为了降低耦合

## IOC过程

第一步： xml配置文件，配置创建的对象

```java
<bean id="userDao" class="cn.lych4.dao.impl.UserDaoImpl"></bean>
```

第二步： 有service 类 和 dao 类， 创建工厂类。进一步降低耦合度

```java
class UserFactory{
   
	public static UserDao getDao() {
   
		String classValue = class属性值; // 1. XML解析
		Class clazz = Class.forName(classValue) // 通过反射创建对象
		return (UserDao)clazz.newInstance();
	}
}
```

## IOC接口

IOC 思想基于IOC容器完成，IOC容器底层就算对象工厂 Spring 提供两种方式实现 IOC 容器： （两个接口）

1. BeanFactory： IOC容器基本实现，是Spring 内部的使用接口，一般开发人员不使用 加载配置文件的时候，不会创建对象，在获取对象的时候才去创建对象 
<li>ApplicationContext： 是BeanFactory的子接口，提供更多强大的功能，一般由开发人员使用。 加载配置文件的时候，就会把对象创建</li>

## IOC的bean管理

### 什么是Bean管理

Bean管理指的是两个操作：

1. Spring 创建对象 
2. Spring 注入属性（set值）

### Bean管理操作有两种方式

①. 基于XML配置文件方式实现， ②.基于注解方式实现 

## IOC的Bean管理操作（基于xml）

### 1.bean标签创建对象

在spring配置文件中，使用bean标签，标签里面添加对应属性，就可以是实现对象创建 *id属性： 对象标识 *class属性： 类全路径 name属性：早期属性，和id类似，只不过可以加特殊符号 默认执行无参构造完成对象创建

#### DI 依赖注入，注入属性

##### ——set注入

对象类必须有对应set方法 配置文件：

```java
<bean id="book" class="cn.lych4.spring5.bean.Book">
        <!--   set方法注入属性     -->
        <property name="bName" value="易筋经"></property>
        <property name="bAuthor" value="达摩老祖"></property>
    </bean>
```

##### ——有参构造注入

对象类必须有对应有参构造方法 配置文件：

```java
<!--    有参构造创建对象-->
    <bean id="orders" class="cn.lych4.spring5.bean.Orders">
       <constructor-arg name="oname" value="电脑"></constructor-arg>
        <constructor-arg name="address" value="China"></constructor-arg>
    </bean>
```

##### xml注入其他类型属性，空值和bean值

1.字面量 **null 空值**

```java
<!--注入空值-->
        <property name="bAuthor">
            <null/>
        </property>
```

**属性值包括特殊符号** 特殊符号转义，或者是把特殊内容写到 **<![CDATA[这里]]**

```java
<!--注入带特殊符号的值,  <[大神]> -->
        <property name="bAuthor">
            <value><![CDATA[<[大神]>]]></value>
        </property>
```

2.注入属性-外部bean 创建一个Service类和Dao类 在Service类中调用Dao类中的方法 在spring配置文件中进行配置

```java
<!--    1.service 和 dao 对象创建-->
    <bean id="userService" class="cn.lych4.spring5.service.UserService">
<!--        2.service 中注入 userDao 对象， name:类里面的属性名称-->
        <property name="userDao" ref="userDaoImpl"></property>
    </bean>
    <bean id="userDaoImpl" class="cn.lych4.spring5.dao.UserDaoImpl"></bean>
```

3.注入属性-内部bean （1）**一对多关系**：部门和员工 （2）在实体类中表示一对多的关系

```java
//部门类,一
public class Dept {
   
    private String name;

    public void setName(String name) {
   
        this.name = name;
    }
}

//员工类，多
public class Emp {
   
    private String ename;
    private String gender;
    //员工属于某一个部分
    private Dept dept;

    public void setEname(String ename) {
   
        this.ename = ename;
    }

    public void setGender(String gender) {
   
        this.gender = gender;
    }

    public void setDept(Dept dept) {
   
        this.dept = dept;
    }
}
```

（3）在spring配置文件中进行配置

```java
<!--    注入内部bean-->
    <bean id="emp" class="cn.lych4.spring5.bean.Emp">
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>
<!--        设置对象类型属性-->
        <property name="dept">
            <bean id="dept" class="cn.lych4.spring5.bean.Dept">
                <property name="name" value="安保部"></property>
            </bean>
        </property>
    </bean>
```

3.注入属性-级联赋值 (1) 第一种

```java
<!--    注入内部bean-->
    <bean id="emp" class="cn.lych4.spring5.bean.Emp">
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>
<!--        级联赋值-->
        <property name="dept" ref="dept"></property>
    </bean>
    <bean id="dept" class="cn.lych4.spring5.bean.Dept">
        <property name="dname" value="财务部"></property>
    </bean>
```

（2）第二种

```java
<!--    注入内部bean-->
    <bean id="emp" class="cn.lych4.spring5.bean.Emp">
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>
<!--        级联赋值-->
        <property name="dept" ref="dept"></property>
        <!--        这里Emp类必须有dept属性的get方法-->
        <property name="dept.dname" value="技术部"></property>
    </bean>
    <bean id="dept" class="cn.lych4.spring5.bean.Dept">
        <property name="dname" value="财务部"></property>
    </bean>
```

##### xml注入集合属性

注入数组、List、Map

```java
<bean id="stu" class="cn.lych4.spring5.collectiontype.Stu">
<!--        数组类型注入-->
        <property name="courses">
            <array>
                <value>java课程</value>
                <value>数据库课程</value>
            </array>
        </property>
<!--        list类型注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>三哥</value>
            </list>
        </property>
<!--        map类型属性注入-->
        <property name="maps">
            <map>
                <entry key="爱好" value="学习"></entry>
            </map>
        </property>
<!--        set类型属性注入-->
        <property name="sets">
            <set>
                <value>Java</value>
                <value>MySQL</value>
            </set>
        </property>
    </bean>
```

设置对象集合

```java
<bean id="stu" class="cn.lych4.spring5.collectiontype.Stu">
<!--        list类型,里面是对象，属性注入-->
        <property name="courseList">
            <list>
                <ref bean="course1"></ref>
                <ref bean="course2"></ref>
                <ref bean="course3"></ref>
            </list>
        </property>
    </bean>
    
    <!--    创建多个course对象-->
    <bean id="course1" class="cn.lych4.spring5.collectiontype.Course" >
        <property name="cname" value="Java"></property>
    </bean>
    <bean id="course2" class="cn.lych4.spring5.collectiontype.Course" >
        <property name="cname" value="Python"></property>
    </bean>
    <bean id="course3" class="cn.lych4.spring5.collectiontype.Course" >
        <property name="cname" value="Go"></property>
    </bean>
```

抽取集合值, 引入util命名空间

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

    <util:list id="bookList" >
        <value>MySQL必知必会</value>
        <value>JVM调优</value>
    </util:list>

    <bean id="book" class="cn.lych4.spring5.collectiontype.Book" >
        <property name="bnames" ref="bookList"></property>
    </bean>
</beans>
```

### 2. FactoryBean

Spring有两种类型的bean，一种是普通的bean 另外一种是工厂bean （FactoryBean）

- 普通bean： 在配置文件种定义bean类型就是返回类型 
- 工厂bean：在配置文件定义bean类型可以和返回类型不一样 第一步：创建类，让这个类作为工厂bean， 实现接口FactoryBean 第二步：实现接口里面的方法，在实现的方法中定义返回的bean类型

创建一个Bean，实现FactoryBean接口

```java
public class MyBean implements FactoryBean<Course> {
   

    //定义返回bean
    public Course getObject() throws Exception {
   
        Course course = new Course();
        course.setCname("abc");
        return course;
    }

    public Class<?> getObjectType() {
   
        return null;
    }

    public boolean isSingleton() {
   
        return false;
    }
}
```

xml配置文件

```java
<bean id="myBean" class="cn.lych4.spring5.factorybean.MyBean"></bean>
```

测试:

```java
@Test
    public void testFactoryBean() {
   
        ApplicationContext context = new ClassPathXmlApplicationContext("factorybean.xml");
//        MyBean myBean = context.getBean("myBean", MyBean.class);
        Course myBean = context.getBean("myBean", Course.class);
        System.out.println(myBean);
    }
```

### 3. bean作用域

在Spring里面可以设置创建的bean实例是单实例还是多实例

在Spring里面默认情况下创建的bean都是单实例bean

scope属性可以设置单实例还是多实例。scope属性值： singleton、 prototype

```java
<bean id="book" class="cn.lych4.spring5.collectiontype.Book"  scope="prototype"></bean>
```

scope值是 singleton 的时候，加载spring配置文件就会创建实例对象 scope值是 prototype 时候, 不是在加载 spring 配置文件时候创建对象，获取对象的时候才会创建

### 4. bean生命周期

①通过构造器创建bean实例（无参构造） ②为bean的属性设置值和对其他bean引用（调用set方法）

**③把bean实例传递bean后置处理器的Before方法**（需要配置） ③调用bean的初始化的方法（需要进行配置初始化的方法） **③把bean实例传递bean后置处理器的After方法** （需要配置）

④bean可以使用了（对象获取到了） ⑤当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

```java
<!--    配置后置处理器， 所有bean都会加上后置处理器-->
    <bean id="myBeanPost" class="cn.lych4.spring5.bean.MyBeanPost"></bean>
```

### 5. xml自动装配

bean标签属性autowire，配置自动装配。 两个属性常用值：

1. byName 按属性名称注入，注入值bean的id值和类属性名得一样 
2. byType 按类型属性类型注入

### 6.xml引入外部属性文件


**① 先直接配置数据库信息， 配置德鲁伊连接池** 记得引入jar包![img](https://img-blog.csdnimg.cn/a784771180d0424fa333e1ff3088017d.png)

```java
<!--    配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
```


**② 引入外部属性文件** 创建外部属性文件 ![img](https://img-blog.csdnimg.cn/b1ab7898892d4d08abe1e65d5433e056.png) 引入context名称空间，把properties属性文件引入到 spring 配置文件中

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
<!--    引入外部属性文件-->
    <context:property-placeholder location="jdbc.properties"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"/>
        <property name="url" value="${prop.url}"/>
        <property name="password" value="${prop.password}"/>
    </bean>
</beans>
```

## IOC的Bean管理操作（基于注解）

### 1.基于注解实现对象创建

注解可以用在类、方法、属性上面

@Component @Service @Controller @Repository


这四个注解功能一样，都可以用来创建bean实例 1.需要额外引入AOP的jar包![img](https://img-blog.csdnimg.cn/2b01fbf504c54bfe8a3d33c348395dd4.png) 2.开启组件扫描

```java
<!-- 开启组件扫描, 多个包逗号隔开, 或者直接扫描上层包 （引入名称空间context）-->
    <context:component-scan base-package="cn.lych4.spring5"/>
```

3.创建类，在类上添加创建对象注解

```java
@Component(value = "userService")   //效果和 <bean id="userService" class="..."/> 一样
public class UserService {
   
    public void add() {
   
        System.out.println("service::add");
    }
}
```

**常用包扫描设置：**

```java
<!-- 示例1：  不使用默认filter， 自己配置filter-->
    <context:component-scan base-package="cn.lych4.spring5" use-default-filters="false">
<!--        只扫描带Controller注解的类-->
        <context:include-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 示例2：  使用默认filter， 并且设置不扫描的注解-->
    <context:component-scan base-package="cn.lych4.spring5">
<!--        不扫描Controller注解的类-->
        <context:exclude-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```

### 2.基于注解实现属性注入

@Autowired : 根据属性类型进行自动装配 @Qualifier: 根据属性名称进行注入 @Resource: 可以根据类型注入，也可以根据名称注入 @Value ： 注入普通类型属性

1. Service类 和Dao类都加上创建对象注解（对象创建了） 
2. Service类需要注入Dao类属性， 先在Service类中定义Dao类型属性，不需要加set方法 
3. 在属性上面添加@Autowired 注解即可

**Qualifier 注解**要和Autowired 一起使用, 因为按类型注入一个接口可能有多个实现类，对应多个对象，按类型的话就找不到。

```java
@Autowired
    @Qualifier(value = "userDao1")
    private UserDao userDao;
```

**Resource 注解** 既可以根据类型直接注入，也可以加个name来指定按名称找

**Value 注解**

```java
@Value(value = "abc")
   private String name;
```

### 完全注解开发

包扫描配置都不要了

1. 创建配置类，替代xml文件 
2. 编写测试类

```java
@Test
    public void testService2() {
   
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
        UserService userService = context.getBean("userService", UserService.class);
        userService.add();
    }
```


## 什么是AOP？

面向切面编程（AOP）， 利用AOP可以对业务进行隔离，降低业务各部分之间的耦合度，提高代码重用性。提高开发效率

通过不修改源码的方式，在主干功能里面添加新的功能

## AOP 底层原理

### AOP底层使用动态代理


两种情况 的动态代理 **①有接口的情况， 使用JDK的动态代理** 一个接口，一个实现类 ![img](https://img-blog.csdnimg.cn/21e5afc676654318b6504f70c2db0e98.png) JDK动态代理， 创建一个UserDao 接口的实现类代理对象，去增强类的方法


②没有接口的情况，使用CGLIB动态代理 ![img](https://img-blog.csdnimg.cn/b3a2839f12bd4313a1bada4ba149faf1.png) 创建当前类的子类的代理对象

### JDK动态代理


使用JDK动态代理，使用Proxy类里面的方法创建代理对象 ![img](https://img-blog.csdnimg.cn/2f26b857d2f840cf985ffc80d46d5d83.png) 三个参数分别是 类加载器、这个类实现的接口、实现InvocationHandler接口，写增强部分 代码：

```java
public class JDKProxy {
   
    public static void main(String[] args) {
   
        //创建接口实现类代理对象
        Class[] interfaces = {
   UserDao.class};
        UserDaoImpl userDao = new UserDaoImpl();
        UserDao proxyUserDao = (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));
        int result = proxyUserDao.add(1, 2);
        System.out.println(result);

    }
}

class  UserDaoProxy implements InvocationHandler {
   
    private Object obj;

    //1. 创建的是谁的代理对象，把这个人传进来
    public UserDaoProxy(Object obj) {
   
        this.obj = obj;
    }

    //增强逻辑
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
   
        //方法之前, 这里还可以根据方法名实现只增强接口中的某些方法
        System.out.println("方法执行之前...方法名：" + method.getName() + "...传递的参数：" + Arrays.toString(args)) ;
        //执行原方法
        Object ret = method.invoke(obj, args); 
        //方法之后
        System.out.println("方法之后执行..." + obj);
        return ret;
    }
}
```

UserDao

```java
public interface UserDao {
   
    public int add(int a, int b);
}
```

UserDaoImpl

```java
public class UserDaoImpl implements UserDao {
   
    public int add(int a, int b) {
   
        System.out.println("add方法执行了");
        return a + b;
    }
}
```

## AOP术语

1. 连接点： 类中哪些方法可以被增强，这些方法就称为连接点 2. 切入点： 实际被真正增强的方法叫做切除点 3. 通知（增强）：实际增强的逻辑部分称为通知。有前置通知、后置通知、环绕通知、异常通知、最终通知 4. 切面 ：是动作， 把通知应用到切入点的过程就是切面

## 实现AOP操作准备操作

Spring 框架中一般基于AspectJ实现AOP操作。 AspectJ是独立的AOP框架， 一般把AspectJ和Spring框架一起使用，进行AOP操作

基于AspectJ 实现AOP操作，两种方式： xml配置方式和注解方式。


1.引入jar包 ![img](https://img-blog.csdnimg.cn/9218096c27284d9db25914fa2bf56d13.png)

**2.切入点表达式**


切入点表达式作用： 知道对哪个类里面的哪个方法进行增强 语法结构： ![img](https://img-blog.csdnimg.cn/5761f31d68554fdbaa20e6bb9800a5f9.png)

```java
//举例1： 对cn.lych4.spring5.dao.BookDao 类里面的add()进行增强
execution(* cn.lych4.spring5.dao.BookDao.add(..))

//举例2： 对cn.lych4.spring5.dao.BookDao 类里面所有方法进行增强
execution(* cn.lych4.spring5.dao.BookDao.*(..))

//举例3： 对cn.lych4.spring5.dao 包里所有类里面所有方法进行增强
execution(* cn.lych4.spring5.dao.*.*(..))
```

## 实现AOP操作（AspectJ 注解）

1.创建类，在类中定义方法

```java
@Component
public class User {
   
    public void add() {
   
        System.out.println("add........");
    }
}
```

2.创建增强类，编写增强逻辑，不同方法代表不同通知

```java
@Component
@Aspect
public class UserProxy {
   

    //前置通知
    @Before(value = "execution(* cn.lych4.spring5.aopanno.User.add(..))")
    public void before() {
   
        System.out.println("before.....");
    }

    // 最终通知， 不管有没有异常都通知
    @After(value = "execution(* cn.lych4.spring5.aopanno.User.add(..))")
    public void after() {
   
        System.out.println("after.....");
    }

    //返回通知， 返回值执行完才通知   //后置通知
    @AfterReturning(value = "execution(* cn.lych4.spring5.aopanno.User.add(..))")
    public void afterReturning() {
   
        System.out.println("afterReturning.....");
    }

    //异常通知
    @AfterThrowing(value = "execution(* cn.lych4.spring5.aopanno.User.add(..))")
    public void afterThrowing() {
   
        System.out.println("afterThrowing.....");
    }

    //环绕通知， 方法前后都执行
    @Around(value = "execution(* cn.lych4.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
   
        System.out.println("around.....环绕之前....");
        proceedingJoinPoint.proceed();
        System.out.println("around.....环绕之后.....");

    }
}
```

3.进行通知配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


<!--    开启组件扫描-->
    <context:component-scan base-package="cn.lych4.spring5"/>

<!--    开启Aspect生成代理对象-->
    <aop:aspectj-autoproxy/>
</beans>
```

4.测试：

```java
@Test
    public void testAopAnno(){
   
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        User user = context.getBean("user", User.class);
        user.add();
    }
```

**5.可以进行公共切入点抽取**

```java
//相同切入点抽取
    @Pointcut(value = "execution(* cn.lych4.spring5.aopanno.User.add(..))")
    public void pointDemo() {
   

    }

    //前置通知
    @Before(value = "pointDemo()")
    public void before() {
   
        System.out.println("before.....");
    }
```

**6.如果有多个增强类对同一方法进行增强，可以设置增强优先级** 给增强类添加一个@Order() 注解， 括号里填入数字，数字越小优先级越高

```java
@Component
@Aspect
@Order(3)
public class PersonProxy {
   
    //前置通知
    @Before(value = "execution(* cn.lych4.spring5.aopanno.User.add(..))")
    public void before() {
   
        System.out.println("PersonProxy before.....");
    }
}
```

## 基于配置文件方式实现AOP操作（了解）

这样配置就欧克了👇👇👇

```java
<!--创建对象-->
    <bean id="user" class="cn.lych4.spring5.aopanno.User"/>
    <bean id="userProxy" class="cn.lych4.spring5.aopanno.UserProxy"/>
    <!--配置aop增强-->
    <aop:config>
        <!--切入点-->
        <aop:pointcut id="p" expression="execution(* cn.lych4.spring5.aopanno.User.add(..))"/>
        <!--配置切面-->
        <aop:aspect ref="userProxy">
            <!--增强作用在具体的方法上-->
            <aop:before method="before" pointcut-ref="p"/>
        </aop:aspect>
    </aop:config>
```

## 完全注解开发

创建配置类，加三个注解

```java
@Configuration  //表明这是配置类
@ComponentScan(basePackages = {
   "cn.lych4.spring5"}) //开启组件扫描 <context:component-scan base-package="cn.lych4.spring5"/>
@EnableAspectJAutoProxy(proxyTargetClass = true)   //开启Aspect生成代理对象 <aop:aspectj-autoproxy/>
```

>重点<br> <font color="red">AOP概念， 底层原理动态代理（有接口、无接口）， AOP术语 ， AspectJ注解实现AOP操作</font>


## JdbcTemplate是什么

JdbcTemplate是Spring框架对JDBC进行封装，使用JdbcTemplate方便实现对数据库的操作

## 准备工作


1.新引入jar包 ![img](https://img-blog.csdnimg.cn/3c4f442e02f24c519d8028c8954c7a44.png) 2.配置数据库连接池，创建JdbcTemplate对象 jdbc.properties：

```java
jdbc.driverClass=com.alibaba.druid.pool.DruidDataSource
jdbc.url=jdbc:mysql:// /user_db
jdbc.username=root
jdbc.password=
```

bean1.xml：

```java
<!--    引入外部属性文件-->
    <context:property-placeholder location="jdbc.properties"/>
    <!-- 配置连接池 -->
    <!-- DruidDataSource dataSource = new DruidDataSource(); -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!-- 获取properties文件内容，根据key获取，使用spring表达式获取 -->
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

	<!--JdbcTemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<!--注入dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

## 小试牛刀

3.创建Service类、Dao类，在Dao类中注入jdbcTemplate对象 BookService

```java
@Service
public class BookService {
   
    //注入Dao
    @Autowired
    private BookDao bookDao;

    public void add(Book book){
   
        bookDao.add(book);
    }
}
```

BookDao

```java
public interface BookDao {
   
    public void add(Book book);
}
```

BookDaoImpl

```java
@Repository
public class BookDaoImpl implements BookDao{
   

    //注入jdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void add(Book book) {
   
        //1.创建sql语句
        String sql = "insert into book values(?,?,?)";
        //2. 调用方法实现
        Object[] args = {
    book.getBookId(), book.getBookName(), book.getBstatus()};
        int update = jdbcTemplate.update(sql,args);
        System.out.println(update);
    }
}
```

5.创建数据库表对应的实体类， 创建数据库表

```java
public class Book {
   
    private String bookId;
    private String bookName;
    private String bstatus;
	// +getAndSet方法，toString方法
}
```


注：为了规范，数据库列名最好是下划线分割命名，而不是大驼峰命名 ![img](https://img-blog.csdnimg.cn/0d9f6a89b707493da917ea47350a7cf4.png)

6.测试

```java
@Test
    public void testJdbcTemplate() {
   
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);

        Book book = new Book();
        book.setBookId("1");
        book.setBookName("Java");
        book.setBstatus("售出");
        bookService.add(book);
    }
```


可以看的数据库添加了一条数据 ![img](https://img-blog.csdnimg.cn/76591642e5ca428bb071bacb96d0003b.png)

报错： 严重: {dataSource-1} init error 解决方案： ①检查sql语句 ②MySQL数据库版本和驱动不匹配，改配置文件中的 jdbc.driverClass=com.mysql.jdbc.Driver

## 修改删除

修改、删除和添加基本一样， 主要sql语句不一样

修改：

```java
public void updata(Book book) {
   
        String sql = "update book set book_name=?,bstatus=? where book_id=?";
        Object[] args = {
   book.getBookName(), book.getBstatus(), book.getBookId()};
        int update = jdbcTemplate.update(sql, args);
        System.out.println(update);
    }
```

删除

```java
public void delete(String  id) {
   
        String sql = "delete from book where book_id=?";
        int update = jdbcTemplate.update(sql, id);
        System.out.println(update);
    }
```

自行添加接口方法，Service方法， 自行进行修改删除测试， 很简单

## 查询

查询返回某个值

```java
public Integer selectCount() {
   
        String sql = "select count(*) from book";
        //queryForObject(String sql, Class<T> requiredType) 第一个参数sql语句， 第二个参数返回类型class 即你查询的结果返回的类型
        Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
        return count;
    }
```

查询返回对象

```java
public Book findOne(String id) {
   
        String sql = "select * from book where book_id=?";
        //queryForObject(String sql, RowMapper<T> rowMapper, Object... args)
        // 第一个参数sql语句， 第二个参数RowMapper是接口，返回不同类型的数据，使用这个接口里的实现类，完成查询到数据的封装
        Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
        return book;
    }
```

查询返回集合

```java
public List<Book> findAllBook() {
   
        String sql = "select * from book";
        //query(String sql, RowMapper<T> rowMapper)  第一个参数sql语句， 第二个参数RowMapper接口
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
        return bookList;
    }
```

>可能报错：<font color="red"><br> org.springframework.dao.IncorrectResultSizeDataAccessException:<br> Incorrect result size: expected 1, actual 2</font><br> 这个错误的意思是要求返回一个对象，你却返回两个对象，一看数据库，查询的id有两条数据（实际业务id一般是主键不可能重复，这里没设置）<br> 解决方案，手动删除id重复的内容

## 批量操作


批量添加及测试 ![img](https://img-blog.csdnimg.cn/befcfadb40ef4c5888eb01196f4115a1.png)

```java
public void batchAddBook(List<Object[]> batchArgs) {
   
        String sql = "insert into book values(?,?,?)";
        //批量添加过程就是把batchArgs遍历，每个进行添加
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

```java
@Test
    public void testBatch() {
   
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        List<Object[]> batchArgs = new ArrayList<>();
        Object[] o1 = {
   "3", "python2", "未售出2"};
        Object[] o2 = {
   "4", "redis2", "已售出2"};
        Object[] o3 = {
   "5", "linux2", "未售出2"};
        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
        bookService.batchAdd(batchArgs);
    }
```

批量修改及测试

```java
@Override
    public void batchUpdate(List<Object[]> batchArgs) {
   
        String sql = "update book set book_name=?, bstatus=? where book_id=?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

```java
@Test
    public void testBatch() {
   
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        List<Object[]> batchArgs = new ArrayList<>();
        Object[] o1 = {
   "python2", "未售出2", "3"};
        Object[] o2 = {
   "redis2", "已售出2", "4"};
        Object[] o3 = {
   "linux2", "未售出2", "5"};
        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
        bookService.batchUpdate(batchArgs);
    }
```

批量删除

```java
@Override
    public void batchDelete(List<Object[]> batchArgs) {
   
        String sql = "delete from book where book_id=?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

```java
@Test
    public void testBatch() {
   
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        List<Object[]> batchArgs = new ArrayList<>();
        Object[] o1 = {
   "3"};
        Object[] o2 = {
   "4"};
        Object[] o3 = {
   "5"};
        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
        bookService.batchDelete(batchArgs);
    }
```


事务是数据库操作最基本单元，一组操作要么都成功，要么都失败

>事务是一个不可分割的数据库操作序列，也是数据库并发控制的基本单位，其执行的结果必须使数据库从一种一致性状态转变为另一种一致性状态。

事务的ACID特性： 原子性、一致性、隔离性、持久性

## 事务操作

举例： 转账业务

### 准备工作

建表 t_account



![img](https://img-blog.csdnimg.cn/5c3379e5ce2940349ce7befe5fbdd17e.png) 加入原始数据： ![img](https://img-blog.csdnimg.cn/bfbe2135730d4e4cbbfc8c03ed5152a1.png)

创建实体类

```java
public class User {
   
    private String id;
    private String username;
    private Integer money;
    // +getAndSet方法 +toString方法
}
```

创建Dao接口、DaoImpl类、 Service类、测试类，注入属性

```java
public interface UserDao {
   
    void addMoney(int money, String username);
    void reduceMoney(int money, String username);
}
```

```java
@Repository
public class UserDaoImpl  implements UserDao{
   

    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * 多钱
     * @param money
     */
    @Override
    public void addMoney(int money, String username) {
   
        String sql = "update t_account set money=money+? where username=?";
        jdbcTemplate.update(sql, money, username);
    }

    /**
     * 少钱
     * @param money
     */
    @Override
    public void reduceMoney(int money, String username) {
   
        String sql = "update t_account set money=money-? where username=?";
        jdbcTemplate.update(sql, money, username);
    }
}
```

```java
@Service
public class UserService {
   

    @Autowired
    private UserDao userDao;

    public void accountMoney(int money, String from, String to) {
   
        //少钱
        userDao.reduceMoney(money, from);
        //多钱
        userDao.addMoney(money, to);
    }
}
```

```java
@Test
    public void testAccount() {
   
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        UserService userService = context.getBean("userService", UserService.class);
        userService.accountMoney(100, "lucy", "mary");
    }
```


执行测试方法后： ![img](https://img-blog.csdnimg.cn/3170d9155e8c408ab5b098572d3723f3.png)

正常情况这样执行业务是没有问题的，但是万一少钱执行完了，突然断电了，多钱方法没执行。这样就不行了。所有需要数据库事务，多钱和少钱是一个整体，要么都成功，要么都不执行进行回滚。

## Spring事务管理操作

一般是把事务添加到Service层 Spring进行事务操作有两种方式：编程式事务管理和**声明式事务管理**

声明式事务管理： **注解方式** 或基于xml配置文件方式

Spring进行声明式事务管理，底层使用的AOP原理


Spring 提供一个接口代表事务管理器，这个接口针对不同的框架提供不同的实现类 ![img](https://img-blog.csdnimg.cn/ca4bf31d04054a93afc15bf963877c31.png)

### 基于注解实现声明式事务管理


**1.在Spring配置文件中配置事务管理器** ![img](https://img-blog.csdnimg.cn/ada6af597da0485783e591c4400617b7.png) 完整配置文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

<!--    开启组件扫描-->
    <context:component-scan base-package="cn.lych4.spring5"/>
    <!--    引入外部属性文件-->
    <context:property-placeholder location="jdbc.properties"/>
    <!-- 配置连接池 -->
    <!-- DruidDataSource dataSource = new DruidDataSource(); -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!-- 获取properties文件内容，根据key获取，使用spring表达式获取 -->
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--    JdbcTemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--        注入dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 开启事务直接-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    
</beans>
```

**2.在Service类（或者方法）上面添加事务注解** @Transactional注解添加到类上面就是类中所有方法都加上了事务,加方法上就是方法添加事务。 UserService 模拟异常，看看添加事务注解和不加注解数据库数据的区别。 【加了事务注解，程序出现异常事务会回滚，数据库数据不变。不加事务数据库钱少了】

```java
@Transactional
@Service
public class UserService {
   
    @Autowired
    private UserDao userDao;

    public void accountMoney(int money, String from, String to) {
   
        //少钱
        userDao.reduceMoney(money, from);
        int i = 1/0;  //模拟异常
        //多钱
        userDao.addMoney(money, to);
    }
}
```

### @Transactional注解参数设置

1.  propagation： 事务传播行为 多事务方法之间进行调用，这个过程中事务是如何进行管理的， 默认是REQUIRED  
2.  ioslation： 事务隔离级别 事务具有隔离性，多事务操作之间不会产生影响。不考虑隔离性会产生很多问题： 脏读：事务A读取到了事务B未提交的数据，然后事务B回滚了，事务A拿到错误数据 不可重复读：事务A读取到事务B已提交的修改数据，事务A中前后两次读这个数据不一样 幻读（虚读）：事务A读取到事务B已提交的添加的数据，事务A之前读没看到这数据，然后又读看到了这数据  
3.  timeout：超时时间 事务在一定时间内必须提交，如果一直没有提交就会回滚。默认值是-1，不会超时  
4.  readOnly：是否只读 readOnly默认值是false，表示可以查询也可以增删改。 true的话就只能进行查询操作  
5.  rollbackFor：回滚 设置出现哪些异常进行事务回滚  
6.  noRollback：不回滚 设置出现哪些异常事务不进行回滚 

### 基于xml声明式事务管理

**在Spring配置文件中进行配置**：

1. 配置事务管理器、

```java
<!--创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

1. 配置通知、

```java
<!--    2.配置通知-->
    <tx:advice id="txadvice">
<!--        配置事务参数-->
        <tx:attributes>
<!--            指定在哪种规则的方法上面添加事务-->
            <tx:method name="accountMoney" propagation="REQUIRED"/>
<!--            <tx:method name="account*"/>-->
        </tx:attributes>
    </tx:advice>
```

1. 配置切入点和切面

```java
<!--    3.配置切入点和切面-->
    <aop:config>
<!--        配置切入点, UserService类中所有方法-->
        <aop:pointcut id="pt" expression="execution(* cn.lych4.spring5.service.UserService.*(..))"/>
<!--        配置切面-->
        <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>
    </aop:config>
```

### 完全注解开发

```java
@Configuration //配置类
@ComponentScan(basePackages = "cn.lych4.spring5") //开启组件扫描
@EnableTransactionManagement //开启事务
public class TxConfig {
   

    //创建数据库连接池
//        <!--    引入外部属性文件-->
//    <context:property-placeholder location="jdbc.properties"/>
//    <!-- 配置连接池 -->
//    <!-- DruidDataSource dataSource = new DruidDataSource(); -->
//    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
//        <!-- 获取properties文件内容，根据key获取，使用spring表达式获取 -->
//        <property name="driverClassName" value="${jdbc.driverClass}"/>
//        <property name="url" value="${jdbc.url}"/>
//        <property name="username" value="${jdbc.username}"/>
//        <property name="password" value="${jdbc.password}"/>
//    </bean>
    @Bean
    public DruidDataSource getDruidDataSource() {
   
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName("com.mysql.jdbc.Driver");
        druidDataSource.setUrl("jdbc:mysql// //user_db");
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("");
        return druidDataSource;
    }

    //创建jdbcTemplate对象
//    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
//        <!--        注入dataSource-->
//        <property name="dataSource" ref="dataSource"/>
//    </bean>
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
   
        //到IOC容器中根据类型找到 dataSource这个对象
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //注入dataSource
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    //创建事务管理器
//    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
//        <property name="dataSource" ref="dataSource"/>
//    </bean>
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
   
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```


1.  整个Spring5 框架基于Java8，运行时兼容JDK9，许多不建议使用的类和方法被删除  
2.  Spring5.0 框架自带了通用的日志封装。 也可以整合其他日志框架 

## 整合Log4j2


**1. 引入jar包** ![img](https://img-blog.csdnimg.cn/54ed6af85e5a4d0eb77a4efaf8843962.png) **2. 创建log4j2.xml 必须是这个名字**

代码示例

```java
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，可以看到log4j2内部各种详细输出-->
<configuration status="INFO">
    <!--先定义所有的appender-->
    <appenders>
        <!--输出日志信息到控制台-->
        <console name="Console" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </console>
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <!--root：用于指定项目的根日志，如果没有单独指定Logger，则会使用root作为默认的日志输出-->
    <loggers>
        <root level="info">
            <appender-ref ref="Console"/>
        </root>
    </loggers>
</configuration>
```

这样执行代码就会自动输出一些日志，也可以在控制台手动输出日志

```java
public class UserLog {
   
    private static final Logger log = LoggerFactory.getLogger(UserLog.class);

    public static void main(String[] args) {
   
        log.info("Hello log4j2");
        log.warn("Hello log4j2");
    }
}
```

## @Nullable 注解

@Nullable 注解可以使用在方法上面、属性上面、参数上面，表示方法返回可以为空、属性值可以为空、参数值可以为空。

## 函数式风格对象注册

## 整合Junit5.0单元测试


>如果发现有错误的地方，欢迎大家提出批评指正

><b> <font size="4" color="#4e2a40">💖致力于分享记录各种知识干货，<font color="#cc5595">关注我，让我们一起进步，互相学习，不断创作更优秀的文章</font>。<br> 💖💖不要忘了三连哦 👍 💬 ⭐️ </font></b>

