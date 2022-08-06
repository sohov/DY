

>详细介绍了Spring IoC基于XML的DI详细配置全解。

**前面的文章Spring IOC容器的概念以及基于XML的IoC装配中，我们学习了IoC容器的概念以及依赖注入的两种方式，现在我们来看看一些更详细的配置。本文基于Spring5.2.8，案例基于上一篇文章的案例。**







**对于基本类型、String、包装类类型的属性，我们可以直接使用value属性的字符串值来描述具体的值，这样可读性也更强。在最后注入的时候Spring的转换服务会将这些值从 String 转换为属性或参数的实际类型。**

**并且Spring支持使用< value >标签表示具体的字面量值：**

```java
<bean id="simpleSetterBased" class="com.spring.core.SimpleSetterBased">
    <constructor-arg name="property1">
        <value>xxx</value>
    </constructor-arg>
    <constructor-arg name="property2">
        <value>yyy</value>
    </constructor-arg>
    
    <!--setter方法 name表示属性名 value 表示属性值-->
    <property name="property3">
        <value>123</value>
    </property>
    <property name="property4">
        <value>false</value>
    </property>
</bean>
```

## 1.1 Properties快捷转换

**Spring容器支持通过PropertyEditor直接解析value中的特定格式的字符串字面量值，并转换为一个Properties集合。后面我们也会学习集合的注入方式，但是这是一个非常好用的快捷方式！**

我们来测试一下，首先有一个PropertiesDI类，内部有一个Properties（Hashtable是Properties的父类）属性：

```java
/**
 * @author lx
 */
public class PropertiesDI {
   

    private Hashtable properties;

    /**
     * setter
     */
    public void setProperties(Properties properties) {
   
        this.properties = properties;
    }

    @Override
    public String toString() {
   
        return "PropertiesDI{" +
                "properties=" + properties +
                '}';
    }
}
```

配置如下：

```java
<!--properties-->
<bean class="com.spring.core.PropertiesDI" id="propertiesDI">
    <property name="properties">
        <!--直接写配置即可,自动转换为Properties-->
        <value>
            ! 注释
            # 注释
            # “#”“!”开头的一行被算作注释不会解析。
            # key和value可以使用 “=”、“:”、“ ”等符号分割，详见properties说明


            key=value
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
            ccc:ddd
            aaa bbb
            eee    fff
        </value>
    </property>
</bean>
```

测试：

```java
@Test
public void properties() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("propertiesDI", PropertiesDI.class));
}
```

结果如下，成功注入：

```java
[propertiesDI]
PropertiesDI{
   properties={
   jdbc.url=jdbc:mysql://localhost:3306/mydb, jdbc.driver.className=com.mysql.jdbc.Driver, eee=fff, key=value, aaa=bbb, ccc=ddd}}
```


## 2.1 ref引用

**在< constructor-arg >、< property >、< entry >标签中有一个ref属性，用于将bean的指定属性的值设置为对容器管理的另一个bean的引用。这就是引用类型属性依赖的设置方式。被引用的bean是要设置其属性的bean的依赖项，在设置该属性之前，需要对其进行初始化。ref属性的值需要与引用的目标bean的id或者name属性中的一个值相同。**

**当然还有一个< ref >标签，可以作为< constructor-arg >、< property >以及某些集合标签的子标签，通过< ref >标签的bean属性也可以来指定引用的目标bean。< ref >标签允许在同一容器或父容器中创建对任何bean的引用，而不管它是否在同一XML文件中。bean属性的值需要与引用的目标bean的id或者name属性中的一个值相同。**

如下案例，首先有一个RefDI类，用于ref测试：

```java
/**
 * @author lx
 */
public class RefDI {
   

    private HelloSpring helloSpring1;
    private HelloSpring helloSpring2;

    public RefDI(HelloSpring helloSpring1, HelloSpring helloSpring2) {
   
        this.helloSpring1 = helloSpring1;
        this.helloSpring2 = helloSpring2;
    }

    @Override
    public String toString() {
   
        return "RefDI{" +
                "helloSpring1=" + helloSpring1 +
                ", helloSpring2=" + helloSpring2 +
                '}';
    }
}
```

配置文件：

```java
<!--ref-->
<!--定义一个Bean-->
<bean name="helloSpring3" class="com.spring.core.HelloSpring"/>

<bean class="com.spring.core.RefDI" id="refDI">
    <!--使用ref属性引用helloSpring3的bean-->
    <constructor-arg name="helloSpring1" ref="helloSpring3"/>

    <!--使用ref标签引用helloSpring3的bean-->
    <constructor-arg name="helloSpring2">
        <ref bean="helloSpring3"/>
    </constructor-arg>
</bean>
```

测试:

```java
@Test
public void ref() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("refDI", RefDI.class));
}
```

成功注入了其他bean：

```java
初始化
[helloSpring3, refDI]
RefDI{
   helloSpring1=HelloSpring{
   hello='hello'}, helloSpring2=HelloSpring{
   hello='hello'}}
```

## 2.2 parent继承

**< bean >、< ref >等标签中还有一个parent属性，这个属性用于指定目标bean将使用父bean的属性和配置，除了autowire、scope和lazy init属性，parent属性用于相同属性以及值的复用。parent属性的值与目标父bean的id属性或name属性中的一个值相同。**

对于parent继承，这里又分几种情况：

1. 如果父bean有class属性，而子bean没有class属性，那么子bean就是和父bean同一个class类型，相当于创建两个相同的对象。 
2. 如果父bean有class属性，而子bean也有class属性，那么允许它们是不同的类型，但是子bean必须含有父bean中定义的所有的注入方式。 
3. 如果父bean没有class属性，那么子bean必须定义class属性，这个父bean实际上类似于一个属性和值的模版，仅仅被值bean引用，实现配置复用，不能实例化，（这时父bean必须添加abstract="true"属性，表示父bean不会被创建，类似于于抽象类，否则启动容器会尝试父bean，但是由于父bean没有class而抛出异常：No bean class specified on bean definition）。这种情况下，子bean同样必须含有父bean中定义的所有的注入方式。 
4. 这里的父bean和子bean 以及“继承”，并不是Java中的继承关系，仅仅是复用了注入方式，精简了代码！ 
5. 如果子bean和父bean中注入对相同依赖同时注入的值的话，那么可能会相互覆盖对方的值。这根据依赖注入的先后顺序：父bean的构造器注入-&gt;子bean的构造器注入-&gt;父bean的setter注入-&gt;子bean的setter注入，排序在后面的对相同依赖的注入值将会覆盖之前注入的值！

如下案例，首先有三个类：

```java
/**
 * @author lx
 */
public class ParentOne {
   
    private String property1;

    public void setProperty1(String property1) {
   
        this.property1 = property1;
    }

    @Override
    public String toString() {
   
        return "ParentOne{" +
                "property1='" + property1 + '\'' +
                '}';
    }
}
```

**一个有意思的地方是，虽然ParentTwo继承了ParentOne，但是并没有继承私有属性property1，不过由于继承了setProperty1方法，因此仍然能够正常工作，这就是前面说的“子bean必须含有父bean中定义的所有的注入方式”的含义，对于setter方法注入来说，你没这个属性没关系，只要有个同名方法，参数类型能够兼容（从String转为参数类型）就不会报错，与返回值无关（见ParentThree）！**


**< idref >标签通常可以作为< constructor-arg >、< property >以及某些集合标签的子标签，用于将容器中另一个 bean的id或者name的字符串值（并不是引用）传递给< constructor-arg >、< property >以及某些集合标签，同时使用idref容器在部署的时候还会验证这个名称的bean是否真实存在（被定义了），这是一种防止错误的方法。该标签目前用的比较少。**

如下案例，首先有一个IdrefCheck类，用于校验bean是否被定义了：

```java
public class IdrefCheck {
   

    private String targetName;

    public void setTargetName(String targetName) {
   
        this.targetName = targetName;
    }

    @Override
    public String toString() {
   
        return "IdrefDI{" +
                "targetName='" + targetName + '\'' +
                '}';
    }
}
```

配置文件：

```java
<!--idref-->
<!--定义一个Bean-->
<bean name="helloSpring3" class="com.spring.core.HelloSpring" />

<!--idrefCheck校验bean-->
<bean class="com.spring.core.IdrefCheck" name="idrefCheck">
    <!--实际上就等于<property name="targetName" value="helloSpring3">-->
    <!--但是多了bean校验的功能-->
    <property name="targetName">
        <idref bean="helloSpring3"/>
    </property>
</bean>
```

**实际上< idref >的bean属性引用的值就是等于一个String类型的值，都是字符串，但是< idref >多了一个校验对应名称的bean是否存在的功能！**

在idea中，如果idref的bean属性指定的bean名字不存在容器中，那么直接报红，如果运行，那么会抛出：Invalid bean name 'helloSpring3' in bean reference for bean property 'targetName'。

```java
@Test
public void idref() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("idrefDI", IdrefCheck.class));
}
```

结果如下：

```java
org.springframework.beans.factory.BeanCreationException: Error creating 
bean with name 'idrefDI' defined in class path resource [DI.xml]: Initialization of bean failed; nested exception is org.springframework.beans.factory.BeanDefinitionStoreException: Invalid bean name 'helloSpring3' in bean reference for bean property 'targetName'
```

在很久之前（Spring2.0之前的版本中），&lt; idref &gt;标签的一个常用的作用是在用在ProxyFactoryBean中定义的AOP拦截器里。指定拦截器名称时使用&lt; idref &gt;元素可防止你拼写错误拦截器ID。但是目前用的比较少了！


**< bean >标签的内部可以使用< constructor-arg >、< property >以及某些集合标签，表示依赖注入。同样，在< constructor-arg >、< property >以及某些集合标签中也可以使用< bean >子标签，表示一个内部bean。原因很简单，如果我们注入的是一个对象，并且我们不想要通过ref引用其他已存在的bean，那么只有定义自己的内部的bean。**

**和“外部”bean的区别是，内部bean不需要定义id或者name属性，因为这个对象就相当于一个外部bean自己的对象。就算指定了，容器也不会使用这些值作为bean的名字，我们也不能通过IoC容器获取。容器在创建时也会忽略内部bean的scope作用域属性（后面会讲），因为内部 bean 始终是匿名的，并且始终使用外 bean 创建。无法独立访问内部bean，也无法将它们注入其他外部bean中。**

如下案例，首先有一个InnerBean类，用于内部bean测试：

```java
/**
 * 内部bean
 *
 * @author lx
 */
public class InnerBean {
   
    private InnerBeanInner innerBeanInner1;
    private InnerBeanInner innerBeanInner2;

    public void setInnerBeanInner1(InnerBeanInner innerBeanInner1) {
   
        this.innerBeanInner1 = innerBeanInner1;
    }

    public void setInnerBeanInner2(InnerBeanInner innerBeanInner2) {
   
        this.innerBeanInner2 = innerBeanInner2;
    }


    @Override
    public String toString() {
   
        return "InnerBean{" +
                "innerBeanInner1=" + innerBeanInner1 +
                ", innerBeanInner2=" + innerBeanInner2 +
                '}';
    }

    public static class InnerBeanInner {
   
        private String property1;
        private int property2;


        public void setProperty1(String property1) {
   
            this.property1 = property1;
        }

        public void setProperty2(int property2) {
   
            this.property2 = property2;
        }

        @Override
        public String toString() {
   
            return "InnerBeanInner{" +
                    "property1='" + property1 + '\'' +
                    ", property2=" + property2 +
                    '}';
        }
    }
}
```

配置文件：

```java
<!--内部bean-->
<bean id="innerBean" class="com.spring.core.InnerBean">
    <property name="innerBeanInner1">
        <!--内部bean 不需要指定id或者name-->
        <bean class="com.spring.core.InnerBean.InnerBeanInner">
            <property name="property1" value="aaa"/>
            <property name="property2" value="111"/>
        </bean>
    </property>
    <property name="innerBeanInner2">
        <!--内部bean 指定id或者name也没用,不能通过容器获取到-->
        <bean id="innerBeanInner" class="com.spring.core.InnerBean.InnerBeanInner">
            <property name="property1" value="bbb"/>
            <property name="property2" value="222"/>
        </bean>
    </property>
</bean>
```

测试：

```java
@Test
public void innerBean() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    InnerBean innerBean = ac.getBean("innerBean", InnerBean.class);
    System.out.println(innerBean);
}
```

结果：

```java
[innerBean]
InnerBean{
   innerBeanInner1=InnerBeanInner{
   property1='aaa', property2=111}, innerBeanInner2=InnerBeanInner{
   property1='bbb', property2=222}}
```


## 5.1 注入方式

**Spring提供了详细的集合注入方式。< list >、< set >、< map >、< props >、< array >标签分别用来注入Java中的List、Set、Map、Properties、array类型的集合，主要是用于集合类型的依赖项的注入。因为集合的元素既可以是基本类型也可以是对象甚至集合，因此集合注入非常灵活。另外集合注入支持泛型转换，注入的时候会自动将value的字符串值转换为对应泛型类型！**

如下案例，首先有一个CollectionDI类，用于集合注入测试：

```java
/**
 * 集合注入
 *
 * @author lx
 */
public class CollectionDI {
   

    //集合属性注入

    private List list;
    private Set set;
    private Map map;
    private Properties properties;
    private Object[] array;

    public CollectionDI(List list, Set set, Map map, Properties properties, Object[] array) {
   
        this.list = list;
        this.set = set;
        this.map = map;
        this.properties = properties;
        this.array = array;
    }

    static class CollectionInner {
   
        private String property1;
        private int property2;


        public void setProperty1(String property1) {
   
            this.property1 = property1;
        }

        public void setProperty2(int property2) {
   
            this.property2 = property2;
        }

        @Override
        public String toString() {
   
            return "CollectionInner{" +
                    "property1='" + property1 + '\'' +
                    ", property2=" + property2 +
                    '}';
        }
    }

    @Override
    public String toString() {
   
        return "CollectionDI{" +
                "\n" + "list=" + list +
                "\n" + ", set=" + set +
                "\n" + ", map=" + map +
                "\n" + ", properties=" + properties +
                "\n" + ", array=" + Arrays.toString(array) +
                '}';
    }
}
```

配置文件：

```java
<!--Collection注入-->
<bean id="collectionInner" class="com.spring.core.CollectionDI.CollectionInner">
    <property name="property1" value="refs"/>
    <property name="property2" value="111"/>
</bean>

<bean id="collectionDI" class="com.spring.core.CollectionDI">
    <!--list只有一个元素时，可以使用value属性赋值或者ref引用就行了-->
    <constructor-arg name="list">
        <!--list标签表示list集合，用于定义多个元素-->
        <list>
            <!--value标签用于定义字面量的值作为集合元素-->
            <value>111</value>
            <!--也可以使用Bean标签定义一个bean(对象)作为集合元素-->
            <bean class="com.spring.core.CollectionDI.CollectionInner">
                <property name="property1" value="list"/>
                <property name="property2" value="1"/>
            </bean>
            <!--也可以引用外部bean-->
            <ref bean="collectionInner"/>
            <value>null</value>
            <!--当然集合元素也可以定义集合-->

        </list>
    </constructor-arg>
    <!--set只有一个元素时，可以使用value属性赋值或者ref引用就行了-->
    <constructor-arg name="set">
        <!--set标签表示set集合，用于定义多个元素-->
        <set>
            <!--value标签用于定义字面量的值作为集合元素-->
            <value>111</value>
            <!--也可以使用Bean标签定义一个bean(对象)作为集合元素-->
            <bean class="com.spring.core.CollectionDI.CollectionInner">
                <property name="property1" value="set"/>
                <property name="property2" value="2"/>
            </bean>
            <!--也可以引用外部bean-->
            <ref bean="collectionInner"/>
            <value>null</value>
            <!--也可以使用idref，仅作为字符串-->
            <idref bean="collectionInner"/>
            <!--当然集合元素也可以定义集合-->
        </set>
    </constructor-arg>
    <constructor-arg name="map">
        <!--map标签表示map集合-->
        <map>
            <!--map标签中首先需要定义entry标签，表示一个键值对-->
            <entry key="key" value="value"/>
            <!--key和value都可以使用标签-->
            <entry key-ref="collectionInner" value-ref="collectionInner"/>
            <!--key和value都可以引用外部bean-->
            <entry>
                <!--key可以使用内部bean，或者集合等等-->
                <key>
                    <bean class="com.spring.core.CollectionDI.CollectionInner">
                        <property name="property1" value="mapkey"/>
                        <property name="property2" value="3"/>
                    </bean>
                </key>
                <!--注意value标签只能是注入字面量值，如果想要对象类型的value，那么直接使用Bean标签就行了-->
                <bean class="com.spring.core.CollectionDI.CollectionInner">
                    <property name="property1" value="mapvalue"/>
                    <property name="property2" value="3"/>
                </bean>

                <!--value也可以是集合等等类型，但是只能有一个大标签-->
                <!--<map>-->
                <!--    <entry key="innermap" value="innermap"/>-->
                <!--    <entry key="inner2map" value="inner2map"/>-->
                <!--</map>-->
                <!--也可以使用idref，仅作为字符串-->
                <!--<idref bean="collectionInner"/>-->
            </entry>
        </map>
    </constructor-arg>
    <constructor-arg name="properties">
        <!--props标签表示properties集合-->
        <props>
            <!--props标签中首先需要定义prop标签，表示一个String键值对-->
            <prop key="111">111</prop>
            <prop key="111">222</prop>
            <prop key="222">222</prop>
            <prop key="null">null</prop>
        </props>

        <!--实际上也可以放map,但是要求String类型的key和value-->
        <!--<map>-->
        <!--    <entry key="key" value="value"/>-->
        <!--</map>-->
    </constructor-arg>
    <!--数组只有一个元素时，可以使用value属性赋值或者ref引用就行了-->
    <constructor-arg name="array">
        <!--array标签表示array数组，用于定义多个元素-->
        <array>
            <!--value标签用于定义字面量的值作为集合元素-->
            <value>111</value>
            <!--也可以使用Bean标签定义一个bean(对象)作为集合元素-->
            <bean class="com.spring.core.CollectionDI.CollectionInner">
                <property name="property1" value="array"/>
                <property name="property2" value="4"/>
            </bean>
            <!--也可以引用外部bean-->
            <ref bean="collectionInner"/>
            <value>null</value>
            <!--也可以使用idref，仅作为字符串-->
            <idref bean="collectionInner"/>
            <!--当然集合元素也可以定义为集合-->
        </array>
    </constructor-arg>
</bean>
```

测试：

```java
@Test
public void collectionDI() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("collectionDI", CollectionDI.class));
}
```

结果如下：

```java
[collectionInner, collectionDI]
CollectionDI{
   
list=[111, CollectionInner{
   property1='list', property2=1}, CollectionInner{
   property1='refs', property2=111}, null]
, set=[111, CollectionInner{
   property1='set', property2=2}, CollectionInner{
   property1='refs', property2=111}, null]
, map={
   key=value, CollectionInner{
   property1='refs', property2=111}=CollectionInner{
   property1='refs', property2=111}, CollectionInner{
   property1='mapkey', property2=3}=CollectionInner{
   property1='mapvalue', property2=3}}
, properties={
   null=null, 222=222, 111=222}
, array=[111, CollectionInner{
   property1='array', property2=4}, CollectionInner{
   property1='refs', property2=111}, null]}
```

## 5.2 集合的继承与合并

**Spring容器还支持集合的继承和合并。我们可以定义父< list >、< map >、< set >、< props >、< array >集合bean，并且支持子< list >、< map >、< set >、< props >、< array >集合bean从父集合bean继承值，当然子集合bean也可以重写值。实际上这里的集合继承这就是上面讲的parent继承中的父bean和子bean的延伸，集合也是一种bean，用法都是一样的，都是使用parent属性，只不过集合多了合并的选项！**

**子集合bean定义的依赖值默认会覆盖父集合bean的依赖值，但是< list >、< map >、< set >、< props >、< array >标签支持集合元素的合并**。通过在子集合bean标签中（在父集合bean标签中设置无效）设置属性**merge=true**，子集合bean的值最终就是合并父集合bean和子集合bean的元素的结果。**根据集合的性质，map会替换相同key的value，set则会去重，他们都是无序的，list、array则会累计并且有序（子集合bean的元素会合并到父集合bean的元素之后）。**

**采用merge合并时，子、父集合bean注入的集合元素将也可能相互覆盖而不是合并。其次，构造器注入的方式则会被setter注入的方式完全替换，父集合bean的setter注入则会被子集合bean的setter注入完全替换，因为setter注入相当于重新设置了一个新集合，因此这里所说的集合合并仅针对两个子、父集合bean都是构造器注入的方式！**

**注意**：使用merge合并的时候，父集合bean对相同依赖的注入即使只注入一个值也不能使用value或者ref快捷注入！

如下案例，首先有一个CollectionDI类，用于集合继承合并测试：

```java
/**
 * 集合继承合并
 *
 * @author lx
 */
public class CollectionMerging {
   

    //集合继承合并

    private List list;
    private Set set;
    private Properties properties;
    private Map map;
    private Object[] array;

    public CollectionMerging(List list, Set set, Properties properties, Map map, Object[] array) {
   
        this.list = list;
        this.set = set;
        this.properties = properties;
        this.map = map;
        this.array = array;
    }

    public void setProperties(Properties properties) {
   
        this.properties = properties;
    }

    @Override
    public String toString() {
   
        return "CollectionMerging{" +
                "list=" + list +
                ", set=" + set +
                ", properties=" + properties +
                ", map=" + map +
                ", array=" + Arrays.toString(array) +
                '}';
    }
}
```

配置文件：

```java
<!--    集合继承合并-->
<bean name="collectionMergingParent" abstract="true">
    <constructor-arg name="list">
        <list>
            <value>1111</value>
        </list>
    </constructor-arg>
    <constructor-arg name="set">
        <set>
            <value>2222</value>
            <value>3333</value>
        </set>
    </constructor-arg>
    <constructor-arg name="properties">
        <props>
            <prop key="1">1</prop>
            <prop key="2">2</prop>
            <prop key="3">3</prop>
            <prop key="5">5</prop>
        </props>
    </constructor-arg>
    <property name="properties">
        <props>
            <prop key="1">1</prop>
            <prop key="2">10</prop>
            <prop key="3">3</prop>
        </props>
    </property>
    <constructor-arg name="map">
        <map>
            <entry key="1" value="1"/>
            <entry key="2" value="1"/>
            <entry key="3" value="1"/>
        </map>
    </constructor-arg>
    <!--相同依赖使用merge合并时不可以使用value或者ref简单注入-->
    <!--<constructor-arg name="array" value="111"/>-->
    <constructor-arg name="array">
        <array merge="true">
            <value>111</value>
            <value>121</value>
        </array>
    </constructor-arg>
</bean>


<bean id="collectionMergingChild" class="com.spring.core.CollectionMerging" parent="collectionMergingParent">
    <!--list不会去重-->
    <constructor-arg name="list">
        <list merge="true">
            <value>1111</value>
            <value>2222</value>
        </list>
    </constructor-arg>
    <!--set会去重-->
    <constructor-arg name="set">
        <set merge="true">
            <value>1111</value>
            <value>2222</value>
        </set>
    </constructor-arg>
    <!--KV集合会替换value，由于对于properties依赖最终有一个父bean的setter注入方式，
    因此最终父bean的setter注入会覆盖前面的全部注入  -->
    <constructor-arg name="properties">
        <props merge="true">
            <prop key="1">0</prop>
            <prop key="2">2</prop>
            <prop key="3">4</prop>
            <prop key="4">4</prop>
        </props>
    </constructor-arg>
    <!--<property name="properties">-->
    <!--    <props merge="true">-->
    <!--        <prop key="1">0</prop>-->
    <!--        <prop key="2">2</prop>-->
    <!--        <prop key="3">4</prop>-->
    <!--        <prop key="4">4</prop>-->
    <!--    </props>-->
    <!--</property>-->
    <constructor-arg name="array">
        <array merge="true">
            <value>111</value>
            <value>111</value>
        </array>
    </constructor-arg>
</bean>
```

测试：

```java
@Test
public void collectionMerging() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("collectionMergingChild", CollectionMerging.class));
}
```

结果如下：

```java
[collectionMergingParent, collectionMergingChild]
CollectionMerging{
   list=[1111, 1111, 2222], set=[2222, 3333, 1111], properties={
   3=3, 2=10, 1=1}, map={
   1=1, 2=1, 3=1}, array=[111, 121, 111, 111]}
```


**Spring将默认值为空的属性的值视为空字符串。另外，就算value字面量值设置为null，那也看做是“null”字符串。如果真的想要设置为null而不是空字符串值，那么应该使用< null/>标签，这个标签才表示真正的注入null值。**

如下案例，首先有一个NullDI类，用于null注入测试：

```java
/**
 * null注入
 *
 * @author lx
 */
public class NullDI {
   

    //null注入

    public String property1;
    public String property2;
    public NullDI nullDI;

    public void setProperty1(String property1) {
   
        this.property1 = property1;
    }

    public void setProperty2(String property2) {
   
        this.property2 = property2;
    }

    public void setNullDI(NullDI nullDI) {
   
        this.nullDI = nullDI;
    }

    @Override
    public String toString() {
   
        return "NullDI{" +
                "property1='" + property1 + '\'' +
                ", property2='" + property2 + '\'' +
                ", nullDI=" + nullDI +
                '}';
    }
}
```

配置文件：

```java
<bean class="com.spring.core.NullDI" id="nullDI">
    <!--空的value属性被当做空字符串-->
    <property name="property1" value=""/>
    <!--只有null标签才表示真正的注入null-->
    <property name="property2">
        <null/>
    </property>
    <property name="nullDI">
        <null/>
    </property>
</bean>
```

测试：

```java
@Test
public void nullDI() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    NullDI nullDI = ac.getBean("nullDI", NullDI.class);
    System.out.println(nullDI.property1 == null);
    System.out.println(nullDI.property2 == null);
    System.out.println(nullDI.nullDI == null);
    System.out.println(nullDI);
}
```

结果如下：

```java
[nullDI]
false
true
true
NullDI{
   property1='', property2='null', nullDI=null}
```


**该知识点了解即可。**

**p-namespace（p-命名空间）允许你使< bean >元素的属性而不是嵌套的< property >子标签来描述依赖，实际上是简化setter方式注入的配置！该知识点了解即可。**

如下案例，首先有一个PNameSpaceDI类，用于p-namespace注入测试：

```java
/**
 * p-namespace属性注入
 *
 * @author lx
 */
public class PNameSpaceDI {
   

    //p-namespace属性注入

    private String property1;
    private boolean property2;
    private PNameSpaceDIInner pNameSpaceDIInner;

    public void setProperty1(String property1) {
   
        this.property1 = property1;
    }

    public void setProperty2(boolean property2) {
   
        this.property2 = property2;
    }

    public void setpNameSpaceDIInner(PNameSpaceDIInner pNameSpaceDIInner) {
   
        this.pNameSpaceDIInner = pNameSpaceDIInner;
    }



    public static class PNameSpaceDIInner {
   
        private String property1;
        private int property2;

        public void setProperty1(String property1) {
   
            this.property1 = property1;
        }

        public void setProperty2(int property2) {
   
            this.property2 = property2;
        }

        @Override
        public String toString() {
   
            return "InnerBeanInner{" +
                    "property1='" + property1 + '\'' +
                    ", property2=" + property2 +
                    '}';
        }
    }

    @Override
    public String toString() {
   
        return "PNameSpaceDI{" +
                "property1='" + property1 + '\'' +
                ", property2=" + property2 +
                ", pNameSpaceDIInner=" + pNameSpaceDIInner +
                '}';
    }
}
```

配置文件： p-namespace是Spring2.x版本后提供的方式。p-namespace没有在xsd文件中默认定义，需要我们手动引入：

```java
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
```

bean配置：

```java
<!--p-namespace注入-->
<!--普通setter注入-->
<bean id="pNameSpaceDI1" class="com.spring.core.PNameSpaceDI">
    <property name="property1" value="vv"/>
    <property name="property2" value="true"/>
    <property name="pNameSpaceDIInner">
        <bean class="com.spring.core.PNameSpaceDI.PNameSpaceDIInner">
            <property name="property1" value="p1"/>
            <property name="property2" value="11"/>
        </bean>
    </property>
</bean>

<!--p-namespace注入-->
<bean id="pNameSpaceDI2" class="com.spring.core.PNameSpaceDI" p:property1="xxx" p:property2="true"
      p:pNameSpaceDIInner-ref="pNameSpaceDIInner"/>
<!--p-namespace引用的对象，只能是外部对象-->
<bean id="pNameSpaceDIInner" class="com.spring.core.PNameSpaceDI.PNameSpaceDIInner" p:property1="p1" p:property2="11"/>
```

**将p-namespace引入进来之后就可以使用了。可以看到，用法很简单：对于String、基本类型及其包装类，我们使用 p:属性名=“xxx”即可；对于对象类型，我们应该使用p:属性名-ref=“xxx”，表示引用要给一个bean。**

**p-namespace在一定程度上简化了setter注入的方式，但是也有缺点，那就是对于引用类型的注入只能引用外部容器的bean，不能定义内部bean。**

测试：

```java
@Test
public void pNameSpaceDI() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    PNameSpaceDI pNameSpaceDI = ac.getBean("pNameSpaceDI2", PNameSpaceDI.class);
    System.out.println(pNameSpaceDI);
}
```

结果如下，成功注入：

```java
[pNameSpaceDI1, pNameSpaceDI2, pNameSpaceDIInner]
PNameSpaceDI{
   property1='xxx', property2=true, pNameSpaceDIInner=InnerBeanInner{
   property1='p1', property2=11}}
```


**该知识点了解即可**。

**和前面的p-namespace类似，Spring 3.1继续引入了c-namespace快捷注入，允许你使< bean >元素的属性而不是嵌套的< constructor-arg >子标签来描述依赖，实际上是简化构造器方式注入的配置！**

如下案例，首先有一个PNameSpaceDI类，用于c-namespace注入测试：

```java
/**
 * c-namespace属性注入
 *
 * @author lx
 */
public class CNameSpaceDI {
   

    //c-namespace属性注入

    private String property1;
    private boolean property2;
    private CNameSpaceDIInner cNameSpaceDIInner;

    /**
     * 需要构造器
     */
    public CNameSpaceDI(String property1, boolean property2, CNameSpaceDIInner cNameSpaceDIInner) {
   
        this.property1 = property1;
        this.property2 = property2;
        this.cNameSpaceDIInner = cNameSpaceDIInner;
    }

    public static class CNameSpaceDIInner {
   
        private String property1;
        private int property2;

        public CNameSpaceDIInner(String property1, int property2) {
   
            this.property1 = property1;
            this.property2 = property2;
        }

        @Override
        public String toString() {
   
            return "CNameSpaceDIInner{" +
                    "property1='" + property1 + '\'' +
                    ", property2=" + property2 +
                    '}';
        }
    }

    @Override
    public String toString() {
   
        return "CNameSpaceDI{" +
                "property1='" + property1 + '\'' +
                ", property2=" + property2 +
                ", cNameSpaceDIInner=" + cNameSpaceDIInner +
                '}';
    }
}
```

配置文件： c-namespace没有在xsd文件中默认定义，同样需要我们手动引入：

```java
xmlns:c="http://www.springframework.org/schema/c"
```

bean配置：

```java
<!--c-namespace注入-->
<!--普通构造器注入-->
<bean id="cNameSpaceDI" class="com.spring.core.CNameSpaceDI">
    <constructor-arg name="property1" value="yy"/>
    <constructor-arg name="property2" value="true"/>
    <constructor-arg name="cNameSpaceDIInner">
        <bean class="com.spring.core.CNameSpaceDI.CNameSpaceDIInner">
            <constructor-arg name="property1" value="c1"/>
            <constructor-arg name="property2" value="22"/>
        </bean>
    </constructor-arg>
</bean>


<!--c-namespace注入-->

<!--通过参数名-->
<bean id="cNameSpaceDI2" class="com.spring.core.CNameSpaceDI" c:property1="xxx" c:property2="true"
      c:cNameSpaceDIInner-ref="cNameSpaceDIInner"/>
<!--通过参数位置索引-->
<bean id="cNameSpaceDI3" class="com.spring.core.CNameSpaceDI" c:_0="xxxx" c:_1="true" c:_2-ref="cNameSpaceDIInner"/>

<!--c-namespace引用的对象，只能是外部对象-->
<bean id="cNameSpaceDIInner" class="com.spring.core.CNameSpaceDI.CNameSpaceDIInner">
    <constructor-arg name="property1" value="c1"/>
    <constructor-arg name="property2" value="22"/>
</bean>
```

**将c-namespace引入进来之后就可以使用了。可以看到，用法很简单：对于String、基本类型及其包装类，我们使用 c:属性名=“xxx”即可；对于对象类型，我们应该使用c:属性名-ref=“xxx”，表示引用一个外部bean。另外，c-namespace可以使用参数索引来代替参数名，索引从0开始，格式为c:_index=“xxx”或者c:_index-ref=“xxx”。**

**c-namespace在一定程度上简化了构造器注入的方式，但是也有缺点，那就是对于引用类型的注入只能引用外部容器的bean，不能定义内部bean。**

测试：

```java
@Test
public void cNameSpaceDI() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("cNameSpaceDI2", CNameSpaceDI.class));
    System.out.println(ac.getBean("cNameSpaceDI3", CNameSpaceDI.class));
}
```

结果如下，成功注入：

```java
[cNameSpaceDI, cNameSpaceDI2, cNameSpaceDI3, cNameSpaceDIInner]
CNameSpaceDI{
   property1='xxx', property2=true, cNameSpaceDIInner=CNameSpaceDIInner{
   property1='c1', property2=22}}
CNameSpaceDI{
   property1='xxxx', property2=true, cNameSpaceDIInner=CNameSpaceDIInner{
   property1='c1', property2=22}}
```


**该知识点了解即可。**

**设置bean的< property >依赖时，可以对name属性使用复合属性名称或嵌套属性名称来为内层的属性设置依赖项，这要求除最终属性名称外，路径上的所有依赖组件都不为null。如果中途有为null的依赖，那么抛出NullPointerException。注意：除了setter方法之外，我们一定要有对于路径上依赖的组件的getter方法，否则不能实现复合属性注入！**

1. 对于对象属性，属性名之间使用“.”分隔； 
2. 对于集合或者数组索引，在属性名之后使用“[index]”； 
3. 对于Map，在属性名之后使用“[key]”； 
4. 支持多重集合嵌套，比如&lt; property name=“property5[0][1]” value=“33”/&gt;，表示设置property5属性集合内部的第一个集合元素内部的第二个元素值为33； 
5. 当然支持更复杂的对象属性+集合嵌套，按照语法来写就行了。

如下案例，首先有一个CompoundPropertyDI类，具有多层嵌套内部类，用于复合属性名称注入测试：

```java
/**
 * 复合属性注入
 *
 * @author lx
 */
public class CompoundPropertyDI {
   

    //复合属性注入

    public String property1;
    public boolean property2;
    public CompoundPropertyDIInner compoundPropertyDIInner;

    public void setProperty1(String property1) {
   
        this.property1 = property1;

    }

    public void setProperty2(boolean property2) {
   
        this.property2 = property2;
    }

    public void setCompoundPropertyDIInner(CompoundPropertyDIInner compoundPropertyDIInner) {
   
        this.compoundPropertyDIInner = compoundPropertyDIInner;
    }
    
    /**
     * getter是复合属性名称注入所必须的方法
     */
    public CompoundPropertyDIInner getCompoundPropertyDIInner() {
   
        return compoundPropertyDIInner;
    }

    public static class CompoundPropertyDIInner {
   
        public String property1;
        public int property2;
        public int property3;
        public CompoundPropertyDIInnerInner compoundPropertyDIInnerInner;

        public void setProperty1(String property1) {
   
            this.property1 = property1;
        }

        public void setProperty2(int property2) {
   
            this.property2 = property2;
        }

        public void setProperty3(int property3) {
   
            this.property3 = property3;
        }

        public void setCompoundPropertyDIInnerInner(CompoundPropertyDIInnerInner compoundPropertyDIInnerInner) {
   
            this.compoundPropertyDIInnerInner = compoundPropertyDIInnerInner;
        }

        /**
         * getter是复合属性名称注入所必须的方法
         */
        public CompoundPropertyDIInnerInner getCompoundPropertyDIInnerInner() {
   
            return compoundPropertyDIInnerInner;
        }

        public static class CompoundPropertyDIInnerInner {
   
            public String property1;
            public int property2;


            public void setProperty1(String property1) {
   
                this.property1 = property1;
            }

            public void setProperty2(int property2) {
   
                this.property2 = property2;
            }

            @Override
            public String toString() {
   
                return "CompoundPropertyDIInnerInner{" +
                        "property1='" + property1 + '\'' +
                        ", property2=" + property2 +
                        '}';
            }
        }

        @Override
        public String toString() {
   
            return "CompoundPropertyDIInner{" +
                    "property1='" + property1 + '\'' +
                    ", property2=" + property2 +
                    ", property3=" + property3 +
                    ", compoundPropertyDIInnerInner=" + compoundPropertyDIInnerInner +
                    '}';
        }
    }

    @Override
    public String toString() {
   
        return "CompoundPropertyDI{" +
                "property1='" + property1 + '\'' +
                ", property2=" + property2 +
                ", compoundPropertyDIInner=" + compoundPropertyDIInner +
                '}';
    }
}
```

配置文件：

```java
<!--复合属性注入-->
<!--多层嵌套bean-->
<bean id="compoundPropertyDI" class="com.spring.core.CompoundPropertyDI">
    <property name="property1" value="com"/>
    <property name="property2" value="false"/>
    <property name="compoundPropertyDIInner">
        <bean class="com.spring.core.CompoundPropertyDI.CompoundPropertyDIInner">
            <property name="property1" value="111"/>
            <property name="property2" value="222"/>
            <property name="compoundPropertyDIInnerInner">
                <!--超过两层的内部类嵌套需要使用$分隔-->
                <bean class="com.spring.core.CompoundPropertyDI$CompoundPropertyDIInner$CompoundPropertyDIInnerInner">
                    <property name="property1" value="111"/>
                    <property name="property2" value="222"/>
                </bean>
            </property>
        </bean>
    </property>


    <!--复合属性注入 在外部bean中直接为内部bean的属性赋值-->
    <!--注意：除了setter方法之外，我们一定要有对于路径上依赖的组件的getter方法，否则不能实现复合属性注入-->
    <property name="compoundPropertyDIInner.property3" value="333"/>
    <property name="compoundPropertyDIInner.compoundPropertyDIInnerInner.property1" value="444"/>
</bean>
```

测试：

```java
@Test
public void compoundPropertyDI() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("compoundPropertyDI", CompoundPropertyDI.class));
}
```

结果如下，成功实现复合属性名称注入：

```java
[compoundPropertyDI]
CompoundPropertyDI{
   property1='com', property2=false, compoundPropertyDIInner=CompoundPropertyDIInner{
   property1='111', property2=222, property3=333, compoundPropertyDIInnerInner=CompoundPropertyDIInnerInner{
   property1='444', property2=222}}}
```


该知识点了解即可。

**如果某个bean是另一个bean的依赖项，则通常意味着这个bean被设置为另一个 bean的属性。但是，有时bean之间的依赖关系不太直接，他们之间没有直接的引用关系。例如，web应用中的Database的bean必须先于DAO的bean初始化，但是DAO的bean不需要持有Database的bean实例。**

**< bean >标签的depends-on属性可以显式强制在初始化使用此元素的bean之前初始化一个或多个指定bean。depends-on的值是先初始化的bean的id或者name，需要指定多个depends-on的bean时，可以使用英文的“,”、“;”“ ”符号分隔，Spring会按照depend-on中定义的顺序来初始化bean。**

**另外，depends-on属性也可以指定相应依赖项的销毁时间顺序，最先初始化的bean将最后被销毁。无论是指定初始化顺序还是销毁顺序，都要求设置该属性的bean的作用范围是singleton，否则depends-on属性无效！**

如下案例，首先有一个DependsOnDI类，用于depends-on注入测试：

```java
/**
 * depends-on
 *
 * @author lx
 */
public class DependsOnDI {
   

    public static class DependsOnDIA {
   
        public DependsOnDIA() {
   
            System.out.println("DependsOnDIA初始化");
        }

        public void init() {
   
            System.out.println("DependsOnDIA初始化回调");
        }

        public void destroy() {
   
            System.out.println("DependsOnDIA销毁回调");
        }

    }

    public static class DependsOnDIB {
   
        public DependsOnDIB() {
   
            System.out.println("DependsOnDIB初始化");
        }

        public void init() {
   
            System.out.println("DependsOnDIB初始化回调");
        }

        public void destroy() {
   
            System.out.println("DependsOnDIB销毁回调");
        }
    }

    public static class DependsOnDIC {
   
        public DependsOnDIC() {
   
            System.out.println("DependsOnDIC初始化");
        }

        public void init() {
   
            System.out.println("DependsOnDIC初始化回调");
        }

        public void destroy() {
   
            System.out.println("DependsOnDIC销毁回调");
        }
    }
}
```

配置文件，要求在dependsOnDIA初始化之前，先顺序初始化dependsOnDIC和dependsOnDIB：

```java
<!--depends-on  如果不是scope=singleton(默认)，那么depends-on属性无效-->
<bean class="com.spring.core.DependsOnDI.DependsOnDIA" depends-on="dependsOnDIC dependsOnDIB" id="dependsOnDIA" destroy-method="destroy" init-method="init"/>
<bean class="com.spring.core.DependsOnDI.DependsOnDIB" id="dependsOnDIB" destroy-method="destroy" init-method="init"/>
<bean class="com.spring.core.DependsOnDI.DependsOnDIC" id="dependsOnDIC" destroy-method="destroy" init-method="init"/>
```

测试：

```java
@Test
public void dependsOnDI() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    ac.close();
}
```

结果如下，确实是按照顺序初始化、销毁的：

```java
DependsOnDIC初始化
DependsOnDIC初始化回调
DependsOnDIB初始化
DependsOnDIB初始化回调
DependsOnDIA初始化
DependsOnDIA初始化回调
[dependsOnDIA, dependsOnDIB, dependsOnDIC]
com.spring.core.DependsOnDI$DependsOnDIA@7a9273a8
DependsOnDIA销毁回调
DependsOnDIB销毁回调
DependsOnDIC销毁回调
```


该知识点了解即可。

**默认情况下，ApplicationContext在初始化的时候，就会创建所有定义好的作用范围是singleton的bean及其依赖项。这样做的好处是配置中的错误能够被立即发现，而不是等到使用时才发现。**

**< bean >标签的lazy-init属性可以设置bean为延迟初始化，在首次请求这个bean的时后才创建 bean 实例，而不是在启动容器时就创建 bean 实例。** **另外，当一个延迟初始化bean是一个非延迟初始化的singleton bean的依赖项时，ApplicationContext会在启动时就创建延迟初始化bean，因为它必须满足singleton bean的依赖项的要求。延迟初始化bean被立即初始化并注入到其他非延迟初始化的singleton bean中。**

**lazy-init属性只有在scope="singleton"时才会有效，如果scope=“pototype”，那么即使设置了lazy-init=“false”（默认就是false），也会延迟初始化。**

**我们也可以在< beans >标签上设置default-lazy-init="true"来为该标签下面的全部bean统一设置是否延迟初始化，默认false，非延迟初始化！**

如下案例，首先有一个LazyInitDI类，用于lazy-init注入测试：

```java
/**
 * lazy-init
 *
 * @author lx
 */
public class LazyInitDI {
   

    public static class LazyInitDIA {
   
        public LazyInitDIA() {
   
            System.out.println("LazyInitDIA初始化");
        }
    }

    /**
     * 依赖LazyInitDIC
     */
    public static class LazyInitDIB {
   
        private LazyInitDIC lazyInitDIC;

        public LazyInitDIB(LazyInitDIC lazyInitDIC) {
   
            this.lazyInitDIC = lazyInitDIC;
            System.out.println("LazyInitDIB初始化");

        }
    }

    public static class LazyInitDIC {
   
        public LazyInitDIC() {
   
            System.out.println("LazyInitDIC初始化");
        }
    }
}
```

配置文件：

```java
<!--lazy-init 延迟初始化-->

<!--lazyInitDIA延迟初始化-->
<bean class="com.spring.core.LazyInitDI.LazyInitDIA" lazy-init="true" id="lazyInitDIA"/>
<!--lazyInitDIB属于singleton的非延迟初始化-->
<bean class="com.spring.core.LazyInitDI.LazyInitDIB" id="lazyInitDIB">
    <!--依赖lazyInitDIC-->
    <constructor-arg name="lazyInitDIC" ref="lazyInitDIC"/>
</bean>
<!--lazyInitDIC属于延迟初始化，但是被singleton的lazyInitDIB依赖，因此会立即初始化-->
<bean class="com.spring.core.LazyInitDI.LazyInitDIC" lazy-init="true" id="lazyInitDIC"/>
```

测试：

```java
@Test
public void lazyInit() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    //使用lazyInitDIA的bean实例时才会初始化
    //System.out.println(ac.getBean("lazyInitDIA", LazyInitDI.LazyInitDIA.class));
}
```

结果如下：

```java
LazyInitDIC初始化
LazyInitDIB初始化
[lazyInitDIA, lazyInitDIB, lazyInitDIC]
```


该知识点了解即可。

**这里的“自动注入”，不是前面说的“依赖注入DI”，而是一种对于XML配置文件的省略写法。前面的案例中，我们需要手写配置文件指明依赖和注入方式。Spring支持某些情况下对于依赖自动注入的情况，通过检查ApplicationContext的内容，可以让Spring自动解析、配置bean的依赖。让我们不需要手写配置文件。**

**自动注入的优点如下：**

1. 自动注入可以显著减少手动指定属性或构造函数参数的需要。（前面讲的parent继承也有这个功能）。 
2. 自动注入可以自动更新对象的配置。例如，如果您需要向类添加依赖项，则无需修改配置文件即可自动装配该依赖。

使用基于XML的配置元数据时，可以通过设置&lt; bean &gt;标签的autowire属性来设置自动注入模式，共有四种：

1.  no：默认值。不启用自动装配。bean必须引用由ref定义的元素。虽然配置要多一点，但是这样可以让系统结构和bean之间的关系更加清晰。  
2.  byName：通过属性的名字的方式查找bean依赖的对象并为其注入。比如说某个bean的autowire属性为byName，并且有个属性名为a，有个setA方法，那么IoC容器会在配置文件中查找id/name的值包含a的bean，然后使用setter方法模式为其注入。注意这里的自动setter相比前面讲的手动setter注入有限制，即setXXX方法的XXX一定要对应一个容器中实例的name才行，否则不会注入。  
3.  byType：通过属性的类型查找bean依赖的对象并为其注入。比如说某个bean的autowire属性为byType，并且有个属性名为a，有个setA方法，这个属性a的类型为com.A，那么IoC容器会在配置文件中查找class为com.A的bean，然后使用setter方法模式为其注入。注意这里的自动setter注入相比手动setter配注入有限制，即setXXX方法的XXX一定是属性名才行（第一个字母可以改变大小写）。如果容器有多个同类型的bean，那么抛出异常：expected single matching bean but found x: xxx。  
4.  constructor：同byType一样，也是通过类型查找依赖对象。与byType的区别在于它不是使用setter方法注入，而是使用构造器注入。如果容器有多个同类型的bean，并且不能找到最合适的依赖，那么同样会抛出异常：expected single matching bean but found x: xxx。 

另外，&lt; beans &gt;标签的default-autowire属性同样有上面几个选项，可以指定内部的全部&lt; bean &gt;的通用自动注入方式，但是如果某些&lt; bean &gt;主动设置了自己的自动注入方式，那么会覆盖&lt; beans &gt;标签设置的方式！

如下案例，首先有一个AutowireDI类，用于autowire注入测试：

```java
/**
 * autowire注入
 *
 * @author lx
 */
public class AutowireDI {
   

    /**
     * 依赖AutowireDIB和AutowireDIC
     */
    public static class AutowireDIA {
   
        private AutowireDIB autowireDIB;
        private AutowireDIC autowireDIC;

        public void setAutowireDIC(AutowireDIC autowireDIC) {
   
            this.autowireDIC = autowireDIC;
        }

        public AutowireDIA() {
   
        }

        public AutowireDIA(AutowireDIB autowireDIB) {
   
            this.autowireDIB = autowireDIB;
        }

        @Override
        public String toString() {
   
            return "AutowireDIA{" +
                    "autowireDIB=" + autowireDIB +
                    ", autowireDIC=" + autowireDIC +
                    '}';
        }
    }

    public static class AutowireDIB {
   
        private String property1;
    }

    public static class AutowireDIC {
   
        private String property1;
    }
}
```

配置文件：

```java
<!--autowire-->
<!--用于构造器注入的依赖-->
<bean id="autowireDIB" class="com.spring.core.AutowireDI.AutowireDIB"/>
<bean id="autowireDIB2" class="com.spring.core.AutowireDI.AutowireDIB"/>
<!--用于setter注入的依赖-->
<bean id="autowireDIC" class="com.spring.core.AutowireDI.AutowireDIC"/>
<!--<bean id="autowireDIC2" class="com.spring.core.AutowireDI.AutowireDIC"/>-->

<!--默认，不自动注入-->
<bean id="autowireDIA0" class="com.spring.core.AutowireDI.AutowireDIA" />

<!--byType 将自动setter注入autowireDIC 如果容器有多个同类型的bean，并且找不到最合适的bean，那么抛出异常：-->
<!--expected single matching bean but found 2: autowireDIC,autowireDIC2-->
<bean id="autowireDIA1" class="com.spring.core.AutowireDI.AutowireDIA" autowire="byType"/>

<!--byName 将自动setter注入autowireDIC-->
<bean id="autowireDIA2" class="com.spring.core.AutowireDI.AutowireDIA" autowire="byName"/>

<!--constructor 将自动构造器注入autowireDIC 如果容器有多个同类型的bean，并且找不到最合适的bean，那么抛出异常：-->
<bean id="autowireDIA3" class="com.spring.core.AutowireDI.AutowireDIA" autowire="constructor"/>
```

测试：

```java
@Test
public void autowire() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    System.out.println(ac.getBean("autowireDIA0", AutowireDI.AutowireDIA.class));
    System.out.println(ac.getBean("autowireDIA1", AutowireDI.AutowireDIA.class));
    System.out.println(ac.getBean("autowireDIA2", AutowireDI.AutowireDIA.class));
    System.out.println(ac.getBean("autowireDIA3", AutowireDI.AutowireDIA.class));
}
```

结果如下，成功自动注入：

```java
[autowireDIB, autowireDIB2, autowireDIC, autowireDIA0, autowireDIA1, autowireDIA2, autowireDIA3]
AutowireDIA{
   autowireDIB=null, autowireDIC=null}
AutowireDIA{
   autowireDIB=null, autowireDIC=com.spring.core.AutowireDI$AutowireDIC@782663d3}
AutowireDIA{
   autowireDIB=null, autowireDIC=com.spring.core.AutowireDI$AutowireDIC@782663d3}
AutowireDIA{
   autowireDIB=com.spring.core.AutowireDI$AutowireDIB@1990a65e, autowireDIC=null}
```

## 12.1 XML自动注入的问题

自动注入确实在一定程度上能够节省配置文件的编写，然而还有自己的问题：

1. 对于byName注入，没找到该name的依赖就不会注入。 
2. 对于byType注入，如果容器有多个同类型的bean，并且不能找到最合适的依赖，那么抛出异常：expected single matching bean but found x: xxx。 
3. 对于constructor注入，如果容器有多个同类型的bean，并且不能找到最合适的依赖，那么同样会抛出异常。expected single matching bean but found x: xxx。 
4. 使用自动注入之后，不容易明确bean之间的依赖关系！ 
5. **实际上它们的依赖注入规则都比较复杂，后面讲源码的时候会讲解它的具体规则。比如构造器注入时如果有多个同类型的实例，则可以根据参数名进行进一步匹配，但是setter注入时却不会按照参数名匹配。**

**对于这些的问题，常见的解决办法是：**

1. 最常见的选择是放弃自动注入（默认就是不开启），虽然多了一些配置代码，但是保证不出问题，并且bean的关系更加明显。 
2. 将&lt; bean &gt;的autowire-candidate属性设置为false，该bean从自动注入候选bean中排除。 
3. 将&lt; bean &gt;的primary属性设置为true，该bean设置为自动注入的主要候选bean。 
4. 使用注解方式注入（后面会讲，实际上注解就是一种自动注入）。

## 12.2 选择自动注入候选bean

首先，&lt; beans &gt;标签具有一个**default-autowire-candidates**属性，用于统一筛选当前容器下具有哪些id/name的bean可以作为主动注入的候选bean。default-autowire-candidates的值可以使用模式匹配。例如，要将候选bean的名称限制为以Repository结尾的任何bean，可以使用*Repository。可以提供多个模式，使用逗号分隔。default-autowire-candidates属性只会影响通过类型的自动注入，它不会影响显式的byName的自动注入。

另外，将&lt; bean &gt;标签的**autowire-candidate**属性设置为false（默认为true），也会将该bean从自动注入候选bean中排除，则这个bean在自动注入时将会不可用（包括注解样式的注入，如@Autowired）。如果autowire-candidate属性显示设置了值为true或者false，那么default-autowire-candidates属性将对该bean的筛选失效！需要注意的是，autowire-candidate属性只会影响通过类型的自动注入，它不会影响显式的byName的自动注入。

当然还可以反过来，将&lt; bean &gt;标签的**primary**属性设置为true（默认为fasle），将该bean指定为主要候选对象，一般指定一个就行了，可以指定多个，但是那样的话又不能确定是注入哪一个bean了。注意，即使显示设置了primary属性的值为true或者false，那么default-autowire-candidates属性将对该bean的筛选仍然有效！并且优先级小于autowire-candidate的显示定义的属性！

配置文件：

```java
<!--default-autowire-candidates设置模式，筛选指定规则的name、id的bean-->
<!--只会影响通过类型的自动注入，它不会影响显式的byName的自动注入。-->
<beans default-autowire-candidates="*DIB1*">
    <!--用于构造器注入的依赖   即使设置了primary="true"，仍然受到default-autowire-candidates的影响-->
    <bean id="autowireDIB" class="com.spring.core.AutowireDI.AutowireDIB" primary="true"/>
    <!--autowire-candidate显示设置的属性不会受到default-autowire-candidates的影响，且优先级大于primary设置的属性-->
    <bean id="autowireDIB2" class="com.spring.core.AutowireDI.AutowireDIB" primary="false"
          autowire-candidate="true"/>
    <!--用于setter注入的依赖  autowire-candidate="false"不会影响通过byName注入  -->
    <bean id="autowireDIC" class="com.spring.core.AutowireDI.AutowireDIC" autowire-candidate="false"/>
    <!--<bean id="autowireDIC2" class="com.spring.core.AutowireDI.AutowireDIC"/>-->

    <!--默认，不自动注入-->
    <bean id="autowireDIA0" class="com.spring.core.AutowireDI.AutowireDIA"/>

    <!--byType 将自动setter注入autowireDIC 如果容器有多个同类型的bean，那么抛出异常：-->
    <!--expected single matching bean but found 2: autowireDIC,autowireDIC2-->
    <bean id="autowireDIA1" class="com.spring.core.AutowireDI.AutowireDIA" autowire="byType"/>

    <!--byName 将自动setter注入autowireDIC-->
    <bean id="autowireDIA2" class="com.spring.core.AutowireDI.AutowireDIA" autowire="byName"/>

    <!--constructor 将自动构造器注入autowireDIC 如果容器有多个同类型的bean，那么不能确定注入哪一个-->
    <bean id="autowireDIA3" class="com.spring.core.AutowireDI.AutowireDIA" autowire="constructor"/>
</beans>
```

测试：

```java
@Test
    public void selectAutowire() {
   
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
        System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
        System.out.println("autowireDIB：" + ac.getBean("autowireDIB", AutowireDI.AutowireDIB.class));
        System.out.println("autowireDIB2：" + ac.getBean("autowireDIB2", AutowireDI.AutowireDIB.class));
        System.out.println("autowireDIA0：" + ac.getBean("autowireDIA0", AutowireDI.AutowireDIA.class));
        System.out.println("autowireDIA1；" + ac.getBean("autowireDIA1", AutowireDI.AutowireDIA.class));
        System.out.println("autowireDIA2：" + ac.getBean("autowireDIA2", AutowireDI.AutowireDIA.class));
        System.out.println("autowireDIA3：" + ac.getBean("autowireDIA3", AutowireDI.AutowireDIA.class));
    }
```

结果如下，成功选择注入：

```java
[autowireDIB, autowireDIB2, autowireDIC, autowireDIA0, autowireDIA1, autowireDIA2, autowireDIA3]
autowireDIB：com.spring.core.AutowireDI$AutowireDIB@64485a47
autowireDIB2：com.spring.core.AutowireDI$AutowireDIB@25bbf683
autowireDIA0：AutowireDIA{
   autowireDIB=null, autowireDIC=null}
autowireDIA1；AutowireDIA{
   autowireDIB=null, autowireDIC=null}
autowireDIA2：AutowireDIA{
   autowireDIB=null, autowireDIC=com.spring.core.AutowireDI$AutowireDIC@6ec8211c}
autowireDIA3：AutowireDIA{
   autowireDIB=com.spring.core.AutowireDI$AutowireDIB@25bbf683, autowireDIC=null}
```


该知识点了解即可。

**IoC容器中的bean之间的依赖关系通常是通过属性来确定的，这样就可能由于bean的声明周期不同而造成某些问题。比如某个bean a是单例（singleton）的，只会在岂容启动时创建一次，它依赖了一个bean b，但是这个bean b是非单例的，业务要求是，每一个访问bean a，都需要注入最新的bean b实例。**

显然，单单设置bean b的scope属性为prototype是不能解决问题的，因为容器只创建一次singleton的bean a，因此只获得一次设置属性bean b的机会，以后每次访问都是bean a访问自己保存的bean b的引用，而不是向IoC容器要bean b（getBean），所以造成了bean b相当于单例的情况。

如下案例，首先有一个MethodIn类，用于上面的问题的测试：

```java
/**
 * @author lx
 */
public class MethodIn {
   

    private MethodInInner methodInInner1;
    private MethodInInner methodInInner2;

    public MethodIn(MethodInInner methodInInner1) {
   
        this.methodInInner1 = methodInInner1;
    }

    public void setMethodInInner2(MethodInInner methodInInner2) {
   
        this.methodInInner2 = methodInInner2;
    }

    public MethodInInner getMethodInInner1() {
   
        return methodInInner1;
    }

    public MethodInInner getMethodInInner2() {
   
        return methodInInner2;
    }

    public static class MethodInInner {
   
        public MethodInInner() {
   
            System.out.println("MethodInInner初始化：" + this);
        }
    }
}
```

配置文件：

```java
<!--bean methodIn 每一个调用时需要获取最新的methodInInner对象--><bean id="methodIn" class="com.spring.core.MethodIn">    <constructor-arg name="methodInInner1" ref="methodInInner"/>    <property name="methodInInner2" ref="methodInInner"/></bean><!--methodInInner的scope设置为prototype，看看行不行--><bean class="com.spring.core.MethodIn.MethodInInner" id="methodInInner" scope="prototype"/>
```

测试：

```java
@Testpublic void methodNi() {
       ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));    MethodIn methodIn = ac.getBean("methodIn", MethodIn.class);    //多次获取methodInInner，发现都是同一个对象    System.out.println("methodInInner1：" + methodIn.getMethodInInner1());    System.out.println("methodInInner1：" + methodIn.getMethodInInner1());    System.out.println("methodInInner2：" + methodIn.getMethodInInner2());    System.out.println("methodInInner2：" + methodIn.getMethodInInner2());}
```

结果如下，发现MethodInInner设置为prototype并没有解决问题，仍然只是在第一次初始化methodIn的时候初始化一次：

```java
MethodInInner初始化：com.spring.core.MethodIn$MethodInInner@4d41ceeMethodInInner初始化：com.spring.core.MethodIn$MethodInInner@631330c[methodIn, methodInInner]methodIn：com.spring.core.MethodIn@12c8a2c0methodIn：com.spring.core.MethodIn@12c8a2c0methodIn：com.spring.core.MethodIn@12c8a2c0
```

这种情况，我们可以使用方法注入来解决！

## 13.1 查找方法注入

查找方法注入（Lookup Method Injection），是指IoC容器重写配置文件中指定 的方法，注入方法的返回结果，将会返回容器中另一个命名的bean。

**原理比较简单，在解析< bean>标签的时候，会对具有< lookup-method >标签的bean利用cglib的动态代理技术，生成该类的动态子类，该代理类将会代理< lookup-method >的name属性指定的方法，最终返回bean属性指定名称的bean对象，这个返回是向容器要对象。那么，如果这个bean对象是prototype类型，必然每一次生成一个新的对象并返回，如果这个bean对象是singleton类型，必然每一次返回同一个对象！**

如下案例，首先有一个LookupMethodIn类，用于查找方法注入的测试：

```java
public class LookupMethodIn {
   

    public static class LookupMethodInA {
   
        private LookupMethodInB lookupMethodInB;
        /**
         * 实际上lookupMethodInC属性根本你没有注入过
         */
        private LookupMethodInC lookupMethodInC;

        public LookupMethodInB getLookupMethodInB() {
   
            return lookupMethodInB;
        }

        public void setLookupMethodInB(LookupMethodInB lookupMethodInB) {
   
            this.lookupMethodInB = lookupMethodInB;
        }

        /**
         * 实际上每一个都是找容器要对象
         */
        public LookupMethodInC getLookupMethodInC() {
   
            //调用createLookupMethodInC方法
            return createLookupMethodInC();
        }

        /**
         * 将会被动态代理替换的方法，找容器要对象
         */
        public LookupMethodInC createLookupMethodInC() {
   
            return lookupMethodInC;
        }

        public void setLookupMethodInC(LookupMethodInC lookupMethodInC) {
   
            this.lookupMethodInC = lookupMethodInC;
        }

        @Override
        public String toString() {
   
            return "LookupMethodInA{" +
                    "lookupMethodInB=" + lookupMethodInB +
                    ", lookupMethodInC=" + lookupMethodInC +
                    '}';
        }
    }

    public static class LookupMethodInB {
   
    }

    public static class LookupMethodInC {
   
    }
}
```

配置文件：

```java
<bean class="com.spring.core.LookupMethodIn.LookupMethodInA" 
id="lookupMethodInA">
    <!--普通setter注入-->
    <property name="lookupMethodInB" ref="lookupMethodInB"/>
    <!--查找方法注入 name表示要被动态替换的方法名，bean表示容器中的一个bean的名字，没有过i嗯都会从容器返回该bean的实例-->
    <lookup-method name="createLookupMethodInC" bean="lookupMethodInC"/>
</bean>
<!--需要被注入的bean-->
<bean class="com.spring.core.LookupMethodIn.LookupMethodInB" id="lookupMethodInB" scope="prototype"/>
<bean class="com.spring.core.LookupMethodIn.LookupMethodInC" id="lookupMethodInC" scope="prototype"/>
```

测试：

```java
@Test
public void lookupmethodNi() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    LookupMethodIn.LookupMethodInA lookupMethodInA = ac.getBean("lookupMethodInA", LookupMethodIn.LookupMethodInA.class);
    System.out.println("获取lookupMethodInB，同一个对象");
    System.out.println("lookupMethodInB：" + lookupMethodInA.getLookupMethodInB());
    System.out.println("lookupMethodInB：" + lookupMethodInA.getLookupMethodInB());
    System.out.println("lookupMethodInB：" + lookupMethodInA.getLookupMethodInB());
    System.out.println("获取lookupMethodInC，不同的对象");
    System.out.println("lookupMethodInC：" + lookupMethodInA.getLookupMethodInC());
    System.out.println("lookupMethodInC：" + lookupMethodInA.getLookupMethodInC());
    System.out.println("lookupMethodInC：" + lookupMethodInA.getLookupMethodInC());

    //实际上lookupMethodInA的lookupMethodInC属性根本没有被注入过，每一次的getLookupMethodInC都是直接找容器要对象
    //而由于lookupMethodInC被设置为prototype，因此每一次都会获取新的对象
    System.out.println("lookupMethodInA：" + lookupMethodInA);
}
```

结果如下，查找方法注入能够保证对于prototype的属性每一次返回一个新对象。

```java
[lookupMethodInA, lookupMethodInB, lookupMethodInC]
获取lookupMethodInB，同一个对象
lookupMethodInB：com.spring.core.LookupMethodIn$LookupMethodInB@ba8d91c
lookupMethodInB：com.spring.core.LookupMethodIn$LookupMethodInB@ba8d91c
lookupMethodInB：com.spring.core.LookupMethodIn$LookupMethodInB@ba8d91c
获取lookupMethodInC，不同的对象
lookupMethodInC：com.spring.core.LookupMethodIn$LookupMethodInC@7364985f
lookupMethodInC：com.spring.core.LookupMethodIn$LookupMethodInC@5d20e46
lookupMethodInC：com.spring.core.LookupMethodIn$LookupMethodInC@709ba3fb
lookupMethodInA：LookupMethodInA{
   lookupMethodInB=com.spring.core.LookupMethodIn$LookupMethodInB@ba8d91c, lookupMethodInC=null}
```

另外，由于查找方法注入使用的cglib动态代理技术，那么肯定有些限制。如果方法是抽象的，动态生成的子类会自动实现该方法，否则会将其覆盖。要注意的是因为采用的是继承，bean和代理的方法都不能是final修饰的。

如果我们使用的注解开发，那么查找方法注入的注解就是@Lookup，标注在某个方法上即可。注解的value属性可以不写，会根据类型自动匹配容器中同类型的一个bean，但是如果有多个同类型bean，那么抛出异常：expected single matching bean but found 2:xxx。（前提是annotation-config开启注解配置，后面会讲）。

```java
/**
 * 将会被动态代理替换的方法，找容器要对象。
 * <p>
 * 参数注解配置时，注解的value属性指定bean的名字，可以不写，会根据类型自动匹配容器中同类型的一个bean
 * 但是如果有多个同类型bean，那么抛出异常：expected single matching bean but found 2:xxx
 */
@Lookup("lookupMethodInC")
public LookupMethodInC createLookupMethodInC() {
   
    return lookupMethodInC;
}
```

## 13.2 任意方法替换

**任意方法替换（Arbitrary Method Replacement）：可以实现方法主体和返回结果的替换，相当于运行时使用一个方法替换另一个方法。**

如下案例，首先有一个ArbitraryMethodRe类，用于任意方法替换的测试，内部有一个TimeConverter，它的convert方法本来只有两种格式化逻辑，现在需要增加一种，那么我们可以使用任意方法替换，来修改代码！任意方法替换需要定义一个实现MethodReplacer方法替换器接口的方法替换器，并重写reimplement方法，这个方法中代码以及返回值就是要替换的方法的代码和返回值。

```java
/**
 * @author lx
 */
public class ArbitraryMethodRe {
   

    /**
     * 时间格式化器
     */
    public static class TimeConverter {
   
        /**
         * 需要被替换的方法
         */
        public String convert(byte type) {
   
            DateTimeFormatter dateTimeFormater;
            if (type == 1) {
   
                dateTimeFormater = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH时mm分ss秒");
            } else {
   
                dateTimeFormater = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH时mm分ss秒 SSS毫秒");
            }
            LocalDateTime localDateTime = LocalDateTime.now();
            return dateTimeFormater.format(localDateTime);
        }
    }

    /**
     * 方法替换器
     */
    public static class ReplaceTimeConverter implements MethodReplacer {
   

        /**
         * 重新实现给定的方法，增加功能。还是走的cglib的逻辑
         *
         * @param obj    重新实现的方法的实例
         * @param method 需要重新实现的方法
         * @param args   方法的参数数组
         * @return 方法的返回值
         * @throws Throwable 异常
         */
        @Override
        public Object reimplement(Object obj, Method method, Object[] args) throws Throwable {
   
            byte type = (byte) args[0];
            //增加一种格式化逻辑
            DateTimeFormatter dateTimeFormater;
            switch (type) {
   
                case 1:
                    dateTimeFormater = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH时mm分ss秒");
                    break;
                case 2:
                    dateTimeFormater = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH时mm分ss秒 SSS毫秒");
                    break;
                default:
                    dateTimeFormater = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss SSS");
            }
            LocalDateTime localDateTime = LocalDateTime.now();
            return dateTimeFormater.format(localDateTime);
        }
    }
}
```

配置文件：

```java
<!--任意方法替换-->
<!--方法替换器-->
<bean class="com.spring.core.ArbitraryMethodRe.ReplaceTimeConverter" id="replaceTimeConverter"/>
<!--需要替换的bean-->
<bean class="com.spring.core.ArbitraryMethodRe.TimeConverter" id="timeConverter">
    <!--任意方法替换 name表示要替换的方法 replacer表示方法替换器的引用-->
    <replaced-method name="convert" replacer="replaceTimeConverter">
        <!--在方法重载的情况下标识替换方法的参数，只有当方法被重载并且类中存在多个变量时，该标签才是必需的。-->
        <!-- <arg-type match="byte"/>-->
    </replaced-method>
</bean>
```

可以在元素中使用多个&lt; arg-type &gt;标签来指示被重写的方法的方法签名。只有当方法重载且类中存在多个变体时，参数的签名才是必要的。为方便起见，参数的类型字符串可以是完全限定的类型名称的子字符串。例如，以下所有内容都是匹配 java.lang.String：

```java
java.lang.String
String
Str
```

由于参数数量通常足以区分每个可能的选择，只需键入与参数类型匹配的最短字符串，即可节省大量输入。测试：

```java
@Test
public void arbitraryMethodRe() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println(Arrays.toString(ac.getBeanDefinitionNames()));
    ArbitraryMethodRe.TimeConverter timeConverter = ac.getBean("timeConverter", ArbitraryMethodRe.TimeConverter.class);
    System.out.println(timeConverter.convert((byte) 3));
}
```

结果如下，成功实现了方法替换：

```java
[replaceTimeConverter, timeConverter]
2020/09/02 00:03:04 396
```


## 14.1 作用域分类

**< bean >标签的scope属性用于定义该bean的作用域（作用范围），这是一个非常强大的属性，我们在前面的学习内容中就已经见过很多次了。**

**所谓作用域，实际上应该是“用来声明容器中的对象应该出于的限定场景或者说该对象的特定存活时间（范围），即容器在对象进入其相应的scope之前生成并装配这些对象，在该对象不再处于这些scope的限定之后，容器通常会销毁这些对象”——《Spring揭秘》。**

**Spring 5.x框架支持六个作用域，其中后面四个作用域仅在使用web开发的ApplicationContext中可用，比如XmlWebApplicationContext。当然，Spring支持创建自定义作用域。**

1. singleton： 
 <ol> 
  1. 默认值，单例。单个bean定义绑定到单个bean实例，每个IoC容器仅保存单个对象实例，所有随后的请求和对这个名为bean的请求都返回缓存的对象。 
  1. 当应用加载，创建容器时，对象就被创建了。只要容器在，对象一直活着，并且不会再创建。 当应用卸载，销毁容器时，对象就被销毁了。 
 </ol>  
4. prototype： 
 <ol> 
  4. 原型，单个bean定义绑定到多个bean实例。当具有prototype作用域的bean被注入到另一个即将要被初始化的bean中，或者通过对容器的getBean（）方法调用获取该bean时，都会创建一个新的bean实例。通常来说，应该为所有有状态bean使用Prototype作用域，为无状态bean使用Singleton作用域。大部分bean都是无状态的。 
  4. Spring不负责prototype bean的完整生命周期。容器实例化、配置或以其他方式组装原型对象，然后将其交给客户端，之后就不再记录该原型实例。因此，初始化生命周期回调方法会在所有对象上被调用，不管生命周期如何，但在原型的情况下，不会调用配置的销毁生命周期回调。 
 </ol>  
7. request：单个bean定义被绑定到单个的Http请求生命周期。表示该针对每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。 
8. session：单个bean的定义被绑定到一个Http Session的生命周期。表示该针对每个独立的session都会产生一个新的bean，它仅仅比request scope的bean会存活更长的时间，在其他的方面真是没什么区别。 
9. application：单个bean的定义被绑定到ServletContext的生命周期中（一个web程序中可能有多个IoC容器）。 
10. websocket：单个bean的定义被绑定到WebSocket的生命周期中。

据本人经验，Spring Boot基于注解开发的web项目中其他作用域都用的比较少，一般都是默认的singleton。

## 14.2 Singleton依赖Prototype

当一个prototype bean是一个singleton bean的依赖项时，ApplicationContext在创建singleton bean是就会立即创建一次prototype bean，因为它必须满足singleton bean的依赖项的要求：prototype bean需要非null的被立即初始化并注入到singleton bean中。

如果希望singleton bean在运行时重复获取prototype bean的新实例。则不能将prototype bean通过传统方式（构造器或者setter）注入到singleton bean中，因为当Spring容器实例化singleton bean并解析和注入其依赖项时，该注入只发生一次。如果在运行时需要多个原型bean的新实例，应该使用前面讲的方法注入。


Spring框架提供了许多bean相关的回调接口，用于实现不同的功能。

## 15.1 bean生命周期回调

生命周期回调接口（Lifecycle Callbacks），是与容器对 bean 生命周期的管理进行交互的接口，有两个：初始化之后和销毁之前。对于自己创建的bean，即容器不能管理的bean的生命周期则接口定义无效。比如prototype bean由于创建之后bean的生命周期就不归容器管理，因此只能调用初始化回调方法，并且当具有prototype作用域的bean被注入到另一个即将要被初始化的bean中，或者通过对容器的getBean（）方法调用获取该bean时都会调用，但永远不会调用销毁回调方法。而对于singleton bean，则仅仅调用一次初始化回调方法，因为仅仅初始化一次，并且在容器关闭时才会调用销毁回调方法。注意：这里的“销毁”是从容器中移除该bean实例，至于这个对象到底会不会立即被GC回收则不一定！

要与容器中bean的生命周期的管理进行交互，最原始的办法是对应的bean实现Spring提供的InitializingBean和DisposableBean接口。容器为前者调用afterPropertiesSet()方法，为后者调用destroy()方法，以便bean在被容器管理的初始化之后和销毁之前执行某些操作。

但是，JSR-250注解规范中的@PostConstruct（初始回调）和@PreDestroy（销毁回调）注解是目前Spring 应用程序中定义生命周期回调方法的最佳做法。使用这些注解意味着 bean 不会耦合到特定于 Spring 的接口（不需要向上面那样继承接口）。当然如果不想使用JSR-250的注解，但仍要删除耦合，我们可以考虑基于XML配置文件的配置元数据方式：init-method和destroy-method属性，分别定义初始化之后的回调和销毁之前的回调。

如下案例，首先有一个LifecycleCallback类，用于生命周期回调的测试：

```java
/**
 * bean生命周期回调
 *
 * @author lx
 */
public class LifecycleCallback {
   

    /**
     * 基于实现InitializingBean和DisposableBean接口的回调
     */
    public static class LifecycleCallbackImp implements InitializingBean, DisposableBean {
   
        public LifecycleCallbackImp() {
   
            System.out.println("LifecycleCallbackImp构造器调用");
        }

        @Override
        public void afterPropertiesSet() {
   
            System.out.println("LifecycleCallbackImp初始化回调");
        }

        @Override
        public void destroy() {
   
            System.out.println("LifecycleCallbackImp销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("LifecycleCallbackImp销毁");
        }
    }

    /**
     * 基于XML的回调
     */
    public static class LifecycleCallbackXml {
   
        public LifecycleCallbackXml() {
   
            System.out.println("LifecycleCallbackXml构造器调用");
        }

        public void afterPropertiesSet() {
   
            System.out.println("LifecycleCallbackXml初始化回调");
        }

        public void destroy() {
   
            System.out.println("LifecycleCallbackXml销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("LifecycleCallbackXml销毁");
        }
    }

    /**
     * 基于注解的回调
     */
    public static class LifecycleCallbackAnn {
   
        public LifecycleCallbackAnn() {
   
            System.out.println("LifecycleCallbackAnn构造器调用");
        }

        @PostConstruct
        public void afterPropertiesSet() {
   
            System.out.println("LifecycleCallbackAnn初始化回调");
        }

        @PreDestroy
        public void destroy() {
   
            System.out.println("LifecycleCallbackAnn销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("LifecycleCallbackAnn销毁");
        }
    }

    /**
     * prototype回调测试
     */
    public static class LifecycleCallbackPro {
   
        public LifecycleCallbackPro() {
   
            System.out.println("prototype LifecycleCallbackPro构造器调用");
        }

        @PostConstruct
        public void afterPropertiesSet() {
   
            System.out.println("prototype LifecycleCallbackPro初始化回调");
        }

        @PreDestroy
        public void destroy() {
   
            System.out.println("prototype LifecycleCallbackPro销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("prototype LifecycleCallbackPro销毁");
        }
    }
}
```

配置文件：

```java
<!--bean生命周期回调-->

<!--基于实现接口-->
<bean class="com.spring.core.LifecycleCallback.LifecycleCallbackImp" id="lifecycleCallbackImp"/>
<!--基于xml配置 init-method值为初始化回调方法名  destroy-method值为销毁回调方法名  -->
<bean class="com.spring.core.LifecycleCallback.LifecycleCallbackXml" id="lifecycleCallbackXml"
      init-method="afterPropertiesSet" destroy-method="destroy"/>
<!--基于注解  需要开启注解支持，后面会讲-->
<context:annotation-config/>
<bean class="com.spring.core.LifecycleCallback.LifecycleCallbackAnn" id="lifecycleCallbackAnn"/>

<!--prototype bean在每次从容器中获取实例时都会调用初始化回调，但是永远不会调用销毁回调-->
<bean class="com.spring.core.LifecycleCallback.LifecycleCallbackPro" id="lifecycleCallbackPro" scope="prototype"/>
```

测试：

```java
@Test
public void lifecycleCallback() {
   
    //开启容器
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    //主动获取bean才会回调
    ac.getBean("lifecycleCallbackPro", LifecycleCallback.LifecycleCallbackPro.class);
    ac.getBean("lifecycleCallbackPro", LifecycleCallback.LifecycleCallbackPro.class);
    //close关闭容器，一定是close才会触发
    ac.close();
    //gc不一定发生，对象不一定被彻底销毁
    System.gc();
}
```

结果如下：

```java
LifecycleCallbackImp构造器调用
LifecycleCallbackImp初始化回调
LifecycleCallbackXml构造器调用
LifecycleCallbackXml初始化回调
LifecycleCallbackAnn构造器调用
LifecycleCallbackAnn初始化回调
prototype LifecycleCallbackPro构造器调用
prototype LifecycleCallbackPro初始化回调
prototype LifecycleCallbackPro构造器调用
prototype LifecycleCallbackPro初始化回调
LifecycleCallbackAnn销毁回调
LifecycleCallbackXml销毁回调
LifecycleCallbackImp销毁回调
prototype LifecycleCallbackPro销毁
prototype LifecycleCallbackPro销毁
LifecycleCallbackAnn销毁
LifecycleCallbackXml销毁
LifecycleCallbackImp销毁
```

### 15.1.1 统一默认回调

如果我们基于XML或者注解方式设置回调方法，如果一个项目有自己的规范，那么通常初始化方法和回调方法在不同类中的方法声明都是一致的，即统一命名为init()、initialize()、dispose()等等。

如果方法命名有统一的规范，那么可以配置Spring容器去“查找”特定名字的初始化和销毁方法。通过配置&lt; beans &gt;的default-init-method=”xxx”，表明该beans下面所有的所有的bean在IOC容器进行初始化的时候，都会在合适的时间调用该bean中名称为xxx的方法。通过配置&lt; beans &gt;的default-destroy-method =”yyy”，表明该beans下面所有的所有的bean在IOC容器进行关闭的时候，都会在合适的时间调用该bean中名称yyy的方法。如果想覆盖默认的回调函数，可以在&lt; bean &gt;中定义init-method或者destroy-method。

如下案例，首先有一个DefaultLifecycleCallback类，用于默认回调的测试：

```java
/**
 * @author lx
 */
public class DefaultLifecycleCallback {
   

    public static class DefaultLifecycleCallbackA {
   
        public DefaultLifecycleCallbackA() {
   
            System.out.println("DefaultLifecycleCallbackA构造器调用");
        }

        public void init() {
   
            System.out.println("DefaultLifecycleCallbackA初始化回调");
        }

        public void destroy() {
   
            System.out.println("DefaultLifecycleCallbackA销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("DefaultLifecycleCallbackA销毁");
        }
    }

    public static class DefaultLifecycleCallbackB {
   
        public DefaultLifecycleCallbackB() {
   
            System.out.println("DefaultLifecycleCallbackB构造器调用");
        }

        public void init() {
   
            System.out.println("DefaultLifecycleCallbackB初始化回调");
        }

        public void destroy() {
   
            System.out.println("DefaultLifecycleCallbackB销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("DefaultLifecycleCallbackB销毁");
        }
    }

    public static class DefaultLifecycleCallbackC {
   
        public DefaultLifecycleCallbackC() {
   
            System.out.println("DefaultLifecycleCallbackC构造器调用");
        }

        public void initialize() {
   
            System.out.println("DefaultLifecycleCallbackC初始化回调");
        }

        public void dispose() {
   
            System.out.println("DefaultLifecycleCallbackC销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("DefaultLifecycleCallbackC销毁");
        }
    }

    public static class DefaultLifecycleCallbackD {
   
        public DefaultLifecycleCallbackD() {
   
            System.out.println("DefaultLifecycleCallbackD构造器调用");
        }

        public void initialize() {
   
            System.out.println("DefaultLifecycleCallbackD初始化回调");
        }

        public void dispose() {
   
            System.out.println("DefaultLifecycleCallbackD销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("DefaultLifecycleCallbackD销毁");
        }
    }

    public static class DefaultLifecycleCallbackE {
   
        public DefaultLifecycleCallbackE() {
   
            System.out.println("prototype DefaultLifecycleCallbackE构造器调用");
        }

        public void initialize() {
   
            System.out.println("prototype DefaultLifecycleCallbackE初始化回调");
        }

        public void dispose() {
   
            System.out.println("prototype DefaultLifecycleCallbackE销毁回调");
        }

        @Override
        protected void finalize() {
   
            System.out.println("prototype DefaultLifecycleCallbackE销毁");
        }
    }
}
```

配置文件：

```java
<!--默认生命周期回调-->
<beans default-init-method="init" default-destroy-method="destroy">
    <bean class="com.spring.core.DefaultLifecycleCallback.DefaultLifecycleCallbackA"
          id="defaultLifecycleCallbackA"/>
    <bean class="com.spring.core.DefaultLifecycleCallback.DefaultLifecycleCallbackB"
          id="defaultLifecycleCallbackB"/>
</beans>
<beans default-init-method="initialize" default-destroy-method="dispose">
    <bean class="com.spring.core.DefaultLifecycleCallback.DefaultLifecycleCallbackC"
          id="defaultLifecycleCallbackC"/>
    <bean class="com.spring.core.DefaultLifecycleCallback.DefaultLifecycleCallbackD"
          id="defaultLifecycleCallbackD"/>
    <bean class="com.spring.core.DefaultLifecycleCallback.DefaultLifecycleCallbackE" id="defaultLifecycleCallbackE"
          scope="prototype"/>
</beans>
```

测试：

```java
@Test
public void defaultlifecycleCallback() {
   
    //开启容器
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    //主动获取bean才会回调
    ac.getBean("defaultLifecycleCallbackE", DefaultLifecycleCallback.DefaultLifecycleCallbackE.class);
    ac.getBean("defaultLifecycleCallbackE", DefaultLifecycleCallback.DefaultLifecycleCallbackE.class);
    //close关闭容器，一定是close才会触发
    ac.close();
    //gc不一定发生，对象不一定被彻底销毁
    System.gc();
}
```

结果如下：

```java
DefaultLifecycleCallbackA构造器调用
DefaultLifecycleCallbackA初始化回调
DefaultLifecycleCallbackB构造器调用
DefaultLifecycleCallbackB初始化回调
DefaultLifecycleCallbackC构造器调用
DefaultLifecycleCallbackC初始化回调
DefaultLifecycleCallbackD构造器调用
DefaultLifecycleCallbackD初始化回调
prototype DefaultLifecycleCallbackE构造器调用
prototype DefaultLifecycleCallbackE初始化回调
prototype DefaultLifecycleCallbackE构造器调用
prototype DefaultLifecycleCallbackE初始化回调
DefaultLifecycleCallbackD销毁回调
DefaultLifecycleCallbackC销毁回调
DefaultLifecycleCallbackB销毁回调
DefaultLifecycleCallbackA销毁回调
prototype DefaultLifecycleCallbackE销毁
prototype DefaultLifecycleCallbackE销毁
DefaultLifecycleCallbackD销毁
DefaultLifecycleCallbackC销毁
DefaultLifecycleCallbackB销毁
DefaultLifecycleCallbackA销毁
```

### 15.1.2 bean回调总结

在new 容器时会调用refresh方法，该方法会销毁已存在的bean（执行销毁回调），并且重新初始化单例的bean（执行初始化回调），调用close方法时，则同样会销毁已存在bean（执行销毁回调）。

**自Spring2.5开始，有3种方式来控制bean的生命周期回调：**

1. 实现InitializingBean和DisposableBean回调接口； 
2. 自定义init() 和destroy() 方法，使用XML配置init-method和destroy-method；自定义init() 和destroy() 
3. 方法，使用@PostConstruct 和 @PreDestroy注解；

**可以将3种方式结合使用。如果多种方式一起使用，每种方式都配置了一个不同的方法名，那么他们的执行顺序将会如下面的顺序所示。但是如果他们都配置了同一个方法名，那么该方法只会执行一次。**

如果为初始化方法配置了多个不同的方法，那么执行顺序如下：

1. PostConstruct 注解修饰的方法； 
2. InitializingBean接口里面的afterPropertiesSet() 方法； 
3. 基于XML的自定义的初始化方法。

如果为销毁方法配置了多个不同的方法，那么执行顺序如下：

1. @PreDestroy 注解修饰的方法； 
2. DisposableBean接口里面的destroy() 方法； 
3. 基于XML的自定义的销毁方法。

## 15.2 容器状态回调

下面的容器回调知识点了解即可！

**ApplicationContext容器也有自己的状态，Spring提供了一些接口，我们可以注册监听不同的状态变化，并调用不同的回调方法。**

### 15.2.1 Lifecycle回调

Lifecycle接口，一般用于一些后台组件活动的开启和关闭。当ApplicationContext收到一个start（start方法）或者stop（stop、close方法方法）信号时，它会将该信号传递给所有的Lifecycle接口的实现，判断并尝试调用相应的方法。

除了bean初始化和销毁回调之外，Spring管理的bean还可以实现Lifecycle接口，以便这些bean可以参与由容器自身生命周期驱动的启动和关闭过程。

```java
/**
 * Lifecycle 生命周期回调
 * 监听容器的start和stop事件
 */
public interface Lifecycle {
   

    /**
     * 启动当前组件
     */
    void start();

    /**
     *停止该组件，当该方法执行完成后,该组件会被完全停止。当需要异步停
     * 止行为时，考虑实现SmartLifecycle 和它的 stop(Runnable) 方法变体。
     */
    void stop();

    /**
     * 检查此组件是否正在运行，调用start或者stop方法前都会调用该方法
     * 1. 只有该方法返回false时，start方法才会被执行。
     * 2. 只有该方法返回true时，stop(Runnable callback)或stop()方法才会被执行。
     *
     * @return 当前组件是否正在运行
     */
    boolean isRunning();
}
```

如下案例，首先有一个StartShutCallback类，用于Lifecycle测试：

```java
/**
 * @author lx
 */
public class StartShutCallback {
   

    public static class StartShutCallbackA implements Lifecycle {
   

        private boolean running;

        @Override
        public void start() {
   
            System.out.println("StartShutCallbackA start");
            running = true;
        }

        @Override
        public void stop() {
   
            System.out.println("StartShutCallbackA stop");
            running = false;
        }

        @Override
        public boolean isRunning() {
   
            return running;
        }


    }

    public static class StartShutCallbackB implements Lifecycle{
   

        private boolean running;

        @Override
        public void start() {
   
            System.out.println("StartShutCallbackB start");
            running = true;
        }

        @Override
        public void stop() {
   
            System.out.println("StartShutCallbackB stop");
            running = false;
        }

        @Override
        public boolean isRunning() {
   
            return running;
        }

        public StartShutCallbackB() {
   
            LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(2));
        }
    }
}
```

配置文件：

```java
<!--Startup and Shutdown Callbacks-->
<bean class="com.spring.core.StartShutCallback.StartShutCallbackA" id="startShutCallbackA"/>
<bean class="com.spring.core.StartShutCallback.StartShutCallbackB" id="startShutCallbackB"/>
```

测试：

```java
@Test
public void startShutCallback() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    ac.start();
    //ac.stop();
    ac.close();
}
```

结果如下：

```java
StartShutCallbackA start
StartShutCallbackB start
StartShutCallbackA stop
StartShutCallbackB stop
```

org.springframe.context.Lifecycle接口仅仅响应start和stop行为，但是并没有实现上下文刷新时自动启动的功能。想要监听这个行为，可以使用LifecycleProcessor接口。实际上Lifecycle也是委托LifecycleProcessor来实习的。

LifecycleProcessor接口继承了Lifecycle接口，是Lifecycle的扩展。它还添加了对正在刷新和关闭的容器做出反应的另外两个方法。对于组件的控制更加详细！

```java
/**
 * 生命周期处理器，继承了Lifecycle的功能
 * 新增监听容器的refresh和close事件
 */
public interface LifecycleProcessor extends Lifecycle {
   

    /**
     * 容器在refresh时的回调
     */
    void onRefresh();

    /**
     * 容器在close时的回调
     */
    void onClose();

}
```

如果要对特定 bean 的自动启动进行更加细粒度控制（包括启动等级），应该实现org.springframework.context.SmartLifecycle接口。

组件启动和关闭调用的顺序可能很重要。如果任何两个对象之间存在“依赖”关系，则依赖方在依赖之后开始，在依赖之前停止。然而，有时，直接依赖性是未知的。你可能只知道某个类型的对象应该先于另一个类型的对象开始。在这种情况下，应该使用SmartLifecycle接口。在其父接口Phased上定义的getPhase()方法。

```java
/**
 * 启动时期
 */
public interface Phased {
   

    /**
     * 返回对象启动时期int值
     */
    int getPhase();
}
```

SmartLifecycle提供了一些方法的默认实现：

```java
/**
 * 智能周期回调
 */
public interface SmartLifecycle extends Lifecycle, Phased {
   

    /**
     * 默认启动时期，Integer.MAX_VALUE
     */
    int DEFAULT_PHASE = Integer.MAX_VALUE;


    /**
     * 如果容器在调用refresh方法时，希望能够自己自动进行回调（即调用start方法），则返回true，默认返回true
     * 返回false的表示组件打算通过显式的调用start()来启动，类似于普通的Lifecycle实现。
     */
    default boolean isAutoStartup() {
   
        return true;
    }

    /**
     * SmartLifecycle自定义的Stop方法接受回调函数，将在stop完成之后执行
     *
     * @param callback 回调函数
     */
    default void stop(Runnable callback) {
   
        stop();
        callback.run();
    }

    /**
     * 返回当前bean组件的启动时间，默认返回DEFAULT_PHASE
     */
    @Override
    default int getPhase() {
   
        return DEFAULT_PHASE;
    }
}
```

getPhase()方法返回的int值表示启动顺序，具有最低phase的对象首先启动，停止时，则按相反顺序执行。默认的实现返回Integer.MAX_VALUE，表示组件将在最后启动并首先停止。不实现SmartLifecycle的任何其他Lifecycle对象的默认阶段是0。另外，任何负phase值都表示一个对象应该在这些标准组件开始之前开始（并在它们停止之后停止）。

另外，SmartLifecycle自己定义的Stop方法接受一个回调线程任务。任何实现的对象都必须在stop方案完成后执行该任务。在需要的时候，可以实现异步的关闭，因为LifecycleProcessor接口的默认实现DefaultLifecycleProcessor在每个phase中的对象中执行该回调任务时，可以等待一个超时值。每个phase默认超时为30秒。可以在xml中配置一个名为lifecycleProcessor的bean来覆盖默认的LifecycleProcessor实例。如果只想修改超时，那么修改默认的LifecycleProcessor实例定义就足够了：

```java
<bean id="lifecycleProcessor" 
class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

#### 15.2.1.1 案例

如下案例，首先有一个StartShutCallback类，用于Lifecycle测试：

```java
/**
 * @author lx
 */
public class SmartLifecycleCallback {
   

    /**
     * 智能组件，phase=Integer.MAX_VALUE AutoStartup=true
     */
    public static class SmartLifecycleCallbackA implements SmartLifecycle {
   
        private boolean running;
        private static int count;


        /**
         * refresh或者start的时候会调用，同样根据isRunning返回值判断
         */
        @Override
        public void start() {
   
            System.out.println("SmartLifecycleCallbackA start");
            if (++count == 2) {
   
                running = true;
            }
        }

        /**
         * close或者stop的时候会调用，同样根据isRunning返回值判断
         */
        @Override
        public void stop() {
   
            System.out.println("SmartLifecycleCallbackA stop");
            if (++count == 4) {
   
                running = false;
            }
        }

        @Override
        public boolean isRunning() {
   
            return running;
        }

        //也可以是重写isAutoStartup、getPhase、stop默认方法来实现自己的逻辑

        @Override
        public int getPhase() {
   
            return 1;
        }
    }

    /**
     * 智能组件，phase=-1 AutoStartup=false
     */
    public static class SmartLifecycleCallbackB implements SmartLifecycle {
   
        private boolean running;
        private static int count;


        /**
         * refresh或者start的时候会调用，同样根据isRunning返回值判断
         */
        @Override
        public void start() {
   
            System.out.println("SmartLifecycleCallbackB start");
            if (++count == 1) {
   
                running = true;
            }
        }

        /**
         * close或者stop的时候会调用，同样根据isRunning返回值判断
         */
        @Override
        public void stop() {
   
            System.out.println("SmartLifecycleCallbackB stop");
            if (++count == 3) {
   
                running = false;
            }
        }

        @Override
        public boolean isRunning() {
   
            return running;
        }

        //也可以是重写isAutoStartup、getPhase、stop默认方法来实现自己的逻
        @Override
        public int getPhase() {
   
            return -1;
        }

        @Override
        public boolean isAutoStartup() {
   
            return false;
        }
    }

    /**
     * 普通组件，用作对比 默认值： phase=0 AutoStartup=false
     */
    public static class SmartLifecycleCallbackC implements Lifecycle {
   

        private boolean running;

        @Override
        public void start() {
   
            System.out.println("SmartLifecycleCallbackC start");
            running = true;
        }

        @Override
        public void stop() {
   
            System.out.println("SmartLifecycleCallbackC stop");
            running = false;
        }

        @Override
        public boolean isRunning() {
   
            return running;
        }
    }
}
```

配置文件：

```java
<bean class="com.spring.core.SmartLifecycleCallback.SmartLifecycleCallbackA" id="smartLifecycleCallbackA"/>
<bean class="com.spring.core.SmartLifecycleCallback.SmartLifecycleCallbackB" id="smartLifecycleCallbackB"/>
<bean class="com.spring.core.SmartLifecycleCallback.SmartLifecycleCallbackC" id="smartLifecycleCallbackC"/>
```

测试：

```java
@Test
public void smartLifecycleCallback() {
   
    System.out.println("new容器的代码内部实际上会调用一次refresh操作，因此会自动启动");
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    System.out.println("start调用");
    ac.start();
    System.out.println("stop调用");
    ac.stop();
    System.out.println("close调用");
    ac.close();
}
```

结果如下：

```java
new 容器的代码内部实际上会调用一次refresh操作，因此会自动启动
SmartLifecycleCallbackA start
start调用
SmartLifecycleCallbackB start
SmartLifecycleCallbackC start
SmartLifecycleCallbackA start
close调用
SmartLifecycleCallbackA stop
SmartLifecycleCallbackC stop
SmartLifecycleCallbackB stop
```

A组件实现了SmartLifecycle并且设置了AutoStartup=true，因此new容器时就会调用一次start启动。随后的容器的start方法则对三个组件都有效，按照phase从小到大启动：B-&gt;C-&gt;A，随后的close方法同样对三个组件都有效，按照phase从大到小停止：A-&gt;C-&gt;B。

## 15.3 非web应用中优雅的关闭容器

目前，我们的案例都是非web应用案例，基本上容器随着JVM的结束而结束，并没有安全关闭的过程。我们可以为容器注册一个shutdown hook钩子方法到JVM中，当JVM关闭时，将会安全的关闭容器，让所有的资源得到释放（实际上是回调close方法的逻辑），这实际上也是一个回调方法。

测试：

```java
@Test
public void shutdownHook() {
   
    System.out.println("new容器的代码内部实际上会调用一次refresh操作，因此会自动启动");
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("DI.xml");
    //添加回调方法，在JVM关闭时会调用容器close方法的逻辑，如果不加那么容器并不会正常关闭
    //ac.registerShutdownHook();
    System.out.println("start调用");
    ac.start();
}
```

结果如下，没有开启关闭回调时：

```java
new容器的代码内部实际上会调用一次refresh操作，因此会自动启动
SmartLifecycleCallbackA start
start调用
SmartLifecycleCallbackB start
SmartLifecycleCallbackC start
SmartLifecycleCallbackA start
```

开启关闭回调时：

```java
new容器的代码内部实际上会调用一次refresh操作，因此会自动启动
SmartLifecycleCallbackA start
start调用
SmartLifecycleCallbackB start
SmartLifecycleCallbackC start
SmartLifecycleCallbackA start
SmartLifecycleCallbackA stop
SmartLifecycleCallbackC stop
SmartLifecycleCallbackB stop
```

可以看到，容器正常关闭！


本文侧重于Spring的入门以及IoC的XML核心配置，并没有讲源码，适合Spring初学者。先学会用，然后再看源码，是本系列文章的核心思想，后面会有专门的文章对核心原理进行源码解析！

关于IoC或者DI，我觉得初学者不要看太多理论的文章，先大概了解是什么，然后跟着案例练习，在不断的实践中自然就能明白IoC和DI到底是什么，以及相比于传统的代码到底带来了什么的好处。

本次我们只是介绍了基于XML的配置方法，实际上现在的新的Spring项目基本是基于注解和Java代码的配置方式，XML方式一般在一些老项目中才会用到，这些东西后面我们都会一一介绍，到时候就会知道基于注解配置的好处！Spring 5.x作为一系列文章，将会不断更新，可以随时关注！

**相关文章**

1.  [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html) 
2.  [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html) 
3. https://spring.io/

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

