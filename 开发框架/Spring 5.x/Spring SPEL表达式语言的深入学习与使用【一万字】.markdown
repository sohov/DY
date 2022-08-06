

>基于最新Spring 5.x，详细介绍了SPEL表达式语言的概念、语法，以及结合Spring的使用方式。

  **前两篇文章中我们都接触过SPEL，但是都讲得比较笼统，本文则全面的介绍了SPEL的大部分核心语法，相当于对前文的补充。**   **本文内容较多，实际开发过程中大部分知识点可能都用不到，建议酌情学习。**







  Spring 表达式语言（Spring Expression Language，简称“SPEL”）是一种功能强大的表达式语言，SPEL语法类似于EL表达式（使用过JSP开发的程序员应该不陌生），支持在程序运行时查询和操作数据，可以为我们节省大量Java代码。   SPEL表达式语言是Spring专门创建的一种表达式语言，该语言可用于 Spring的所有产品。然而，SPEL和Spring框架没有过于耦合的关系，SPEL可以单独使用，那就像是一种独立的表达式语言。当然，独立使用时，需要创建一些引导基础类，如parser解析器，而如果在Spring框架中使用SPEL表达式，由于Spring提供了大量支持，我们并不需要创建基础类，只需要编写核心表达式字符串即可，由Spring帮助我们解析。   **SPEL有三种使用方式：XML配置、注解配置、独立代码中使用SPEL。对于大部分开发者，99%的可能性接触到的都是基于Spring的前两种使用场景。**   使用SPEL，可以进行求值、正则匹配、创建对象、调用方法、引用bean、集合转换……等等丰富的操作，下面我们一点点的来学习吧！



  首先，我们同样需要引入SPEL表达式的maven依赖，在前两篇文章中，我们只引入了一个spring-context依赖，然后我们就可以使用spel表达式。很明显，spring-context已经帮我们引入了SPEL的依赖，实际上就是spring-expression的jar包，这个包专门用于支持SPEL表达式语言！ ![img](https://img-blog.csdnimg.cn/20200910154755415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)   因此，对于本文，我们同样只需要引入spring-context的依赖就行了：

```java
<properties>
    <spring-framework.version>5.2.8.RELEASE</spring-framework.version>
</properties>
<dependencies>
    <!--spring 核心组件所需依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring-framework.version}</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

   甚至，如果仅仅单独使用SPEL，那么只需要spring-expression的依赖就行了，但是后面要结合Spring，因此建议就使用spring-context的依赖：

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-expression</artifactId>
    <version>${spring-framework.version}</version>
    <scope>compile</scope>
</dependency>
```


  随后，我们新建一个HelloSpel类，用于编写SPEL表达式案例。 ![img](https://img-blog.csdnimg.cn/20200910155743490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

```java
public class HelloSpel {
   

    public static void main(String[] args) {
   
        //SPEL解析器  实例是可重用的，线程安全。
        ExpressionParser parser = new SpelExpressionParser();

        //分析表达式字符串并返回可用于重复计算的 Expression 对象。
        Expression exp = parser.parseExpression("'Hello SPEL'");
        //在默认标准上下文中计算此表达式，并返回计算结果
        String message = (String) exp.getValue();
        System.out.println(message);
    }
}
```

  运行结果如下：

```java
Hello SPEL
```

  SPEL第一例到此结束！   如果我们要单独使用SPEL的API，那么最常用的SPEL类和接口都位于org.springframework.expression包，及其子包之中。   ExpressionParser接口被称为“表达式解析器”接口，负责分析表达式字符串，上面案例中我们传递的字符串参数就被称为“表达式字符串”。ExpressionParser的实现就是一个具体的表达式解析器。分析完毕之后将会返回一个Expression实例。   Expression接口负责计算此前定义的表达式字符串，并通过getValue返回最终计算结果，getValue的参数中可以指定期望返回的结果类型，不指定时返回Object类型。   在分别调用parseExpress方法和getValue方法时，可以抛出两个异常，ParseException 和EvaluationException。   另外，案例中我们并没有进行初始化IoC容器、加载配置之类的操作，这说明SPEL可以脱离Spring其他组件单独工作。


  SPEL也有自己的容器EvaluationContext，又被称为“计算上下文”，用于计算表达式以解析属性、方法或字段并帮助执行类型转换，案例中的getValue的计算实际上就是在默认上下文容器中执行的，Spring提供了两种容器实现：

1. SimpleEvaluationContext：EvaluationContext的简单实现，侧重于基本 SpEL 功能，以简单条件计算和特定数据绑定方案为目标。某些功能比如Java 类型、构造函数和 bean 引用等功能不支持。 
2. StandardEvaluationContext：具有全部SPEL语言的功能和配置选项。你可以使用它来指定一个默认的根对象，并配置每个可用的与评估相关的策略。

  创建SimpleEvaluationContext实例时，需要指定在SpEL表达式中选择属性访问所需的支持级别：

1. forPropertyAccessors：自定义的PropertyAccessor（无反射） 
2. forReadOnlyDataBinding：只读访问的数据绑定属性，访问公共属性。 
3. forReadWriteDataBinding：读写数据绑定属性，访问公共属性。

  通常，我们使用SimpleEvaluationContext即可，默认也是使用的SimpleEvaluationContext。我们可以向容器中存入对象并指定name，然后通过#name在SPEL表达式中引用对象。对于变量的更多操作，下面会讲到！

```java
@Test
public void context() {
   
    ExpressionParser parser = new SpelExpressionParser();

    String rootObject = "rootObject";

    //手动设置上下文，获取上下文容器
    StandardEvaluationContext context = new StandardEvaluationContext();
    //向容器设置变量
    context.setVariable("rootObject", rootObject);
    context.setVariable("xxx", "字符串变量");
    //传递容器，通过#引用变量作为一次计算表达式的根对象RootObject
    System.out.println(parser.parseExpression("#xxx").getValue(context));
    System.out.println(parser.parseExpression("#rootObject.length()").getValue(context));

    //可以为一个容器设置RootObject，引用它的属性时，可以不加#前缀，一个容器只有一个RootObject
    context.setRootObject(rootObject);
    System.out.println(parser.parseExpression("isEmpty()").getValue(context));


    //getValue方法如果不传递容器中将使用默认计算上下文容器，传递的rootObject变量就是容器要使用来计算表达式的根对象，这种方法也不需要#引用变量
    System.out.println(parser.parseExpression("length()").getValue(rootObject,Integer.class));
}
```


  这部分介绍在独立代码中使用SPEL表达式。随后讲解在XML和注解中的应用，实际上如果这部分弄明白了，那么基于XML和注解的应用就很简单了。

## 4.1 字面量表达式

  SPEL支持字面量表达式（Literal Expressions），支持：

1. 字符串：字符串放在外层参数字符串中时，应该使用’’包裹。字符串中的单引号应该使用两个’'表示，其他特殊符号应该使用相应的转义字符。 
2. 数值：int范围整数、double浮点数、E科学（指数）计数法、十六进制。需要注意的是，转换的类型都是包装类型，不能直接进行基本类型的转换，并且整数类型不可超出int范围。 
3. boolean以及null。

```java
@Test
public void literalExpressions () {
   
    ExpressionParser parser = new SpelExpressionParser();
    //字符串 包裹在''中
    System.out.println(parser.parseExpression("'Hello World'").getValue());
    //字符串中的单引号应该使用两个''表示
    System.out.println(parser.parseExpression("'Hello'' World'").getValue());
    //字符串中的其他特殊符号应该使用相应的转义字符
    System.out.println(parser.parseExpression("'\" \t \\ \377 \u0024'").getValue());


    //数值
    System.out.println(parser.parseExpression("111111111").getValue());
    System.out.println(parser.parseExpression("111.1").getValue());
    //转换的类型都是包装类型，不能直接进行基本类型的转换。
    //System.out.println((int)parser.parseExpression("111.1").getValue());
    //整数类型不可超出int范围。
    //System.out.println((int)parser.parseExpression("-1111111111111111").getValue());
    //科学计数法 默认转换为Double
    System.out.println((double)parser.parseExpression("6.0221415E+23").getValue());
    //16进制 默认转换为Integer
    System.out.println((int)parser.parseExpression("0x7FFFFFFF").getValue());


    //boolean
    System.out.println((boolean)parser.parseExpression("true").getValue());
    //null
    System.out.println(parser.parseExpression("null").getValue() == null);
    System.out.println((int)11.1);
}
```

## 4.2 属性导航

  SPEL支持对象属性导航（property Navigating），这类似于EL的属性导航，对于多层嵌套属性，可以使用“.”来连接，同时支持获取属性值和设置属性值。   导航的属性，要求必须是public修饰的或者提供了相应的getter方法：getXxx，同时，路径上的属性值必须不能为null。   SPEL表达式中，属性的第一个字母可以不区分大小写。

```java
@Test
public void propertyNavigating() {
   
    ExpressionParser parser = new SpelExpressionParser();
    SpelBean spelBean = new SpelBean(new Birth("china"));
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("spelBean", spelBean);
    //支持属性导航 要求属性必须是public修饰的或者提供了相应的getter方法
    System.out.println(parser.parseExpression("#spelBean.birth.birthplace").getValue(context));
    //当然支持设置属性值
    parser.parseExpression("#spelBean.Birth.Birthplace").setValue(context,"wenwen");
    //属性的第一个字母可以不区分大小写
    System.out.println(parser.parseExpression("#spelBean.Birth.Birthplace").getValue(context));
}
```

### 4.2.1 使用到的类


![img](https://img-blog.csdnimg.cn/20200910165844271.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

```java
public class SpelBean {
   
    private String property1;
    private int property2;
    private Birth birth;
    private String[] strings;
    private List<String> stringList;
    private Set<String> stringSet;
    private Map<Object, Object> objectObjectMap;
    private Properties properties;

    public SpelBean() {
   
    }

    public SpelBean(Birth birth) {
   
        this.birth = birth;
    }

    public SpelBean(String property1, Birth birth) {
   
        this.property1 = property1;
        this.birth = birth;
    }

    public String getProperty1() {
   
        return property1;
    }

    public void setProperty1(String property1) {
   
        this.property1 = property1;
    }

    public int getProperty2() {
   
        return property2;
    }

    public void setProperty2(int property2) {
   
        this.property2 = property2;
    }

    public Birth getBirth() {
   
        return birth;
    }

    public void setBirth(Birth birth) {
   
        this.birth = birth;
    }

    public String[] getStrings() {
   
        return strings;
    }

    public void setStrings(String[] strings) {
   
        this.strings = strings;
    }

    public List<String> getStringList() {
   
        return stringList;
    }

    public void setStringList(List<String> stringList) {
   
        this.stringList = stringList;
    }

    public Set<String> getStringSet() {
   
        return stringSet;
    }

    public void setStringSet(Set<String> stringSet) {
   
        this.stringSet = stringSet;
    }

    public Map<Object, Object> getObjectObjectMap() {
   
        return objectObjectMap;
    }

    public void setObjectObjectMap(Map<Object, Object> objectObjectMap) {
   
        this.objectObjectMap = objectObjectMap;
    }

    public Properties getProperties() {
   
        return properties;
    }

    public void setProperties(Properties properties) {
   
        this.properties = properties;
    }

    @Override
    public String toString() {
   
        return "SpelBean{" +
                "property1='" + property1 + '\'' +
                ", property2=" + property2 +
                ", birth=" + birth +
                ", strings=" + Arrays.toString(strings) +
                ", stringList=" + stringList +
                ", stringSet=" + stringSet +
                ", objectObjectMap=" + objectObjectMap +
                ", properties=" + properties +
                '}';
    }
}

//……………………

public class Birth {
   
    private String birthplace;
    private LocalDate birthDate;
    private LocalTime birthTime;

    @Override
    public String toString() {
   
        return "Birth{" +
                "birthplace='" + birthplace + '\'' +
                ", birthDate=" + birthDate +
                ", birthTime=" + birthTime +
                '}';
    }

    public String getBirthplace() {
   
        return birthplace;
    }

    public void setBirthplace(String birthplace) {
   
        this.birthplace = birthplace;
    }

    public LocalDate getBirthDate() {
   
        return birthDate;
    }

    public void setBirthDate(LocalDate birthDate) {
   
        this.birthDate = birthDate;
    }

    public LocalTime getBirthTime() {
   
        return birthTime;
    }

    public void setBirthTime(LocalTime birthTime) {
   
        this.birthTime = birthTime;
    }

    public Birth(String birthplace) {
   
        this.birthplace = birthplace;
    }

    public Birth(String birthplace, LocalDate birthDate, LocalTime birthTime) {
   
        this.birthplace = birthplace;
        this.birthDate = birthDate;
        this.birthTime = birthTime;
    }
}
```

## 4.3 集合导航

  SPEL支持集合元素导航，array数组和Collection集合中元素内容是使用[index]表示法获得的，index表示索引。Map集合的value是通过[key]获取的，注意这里的key如果是字符串，那么需要加上’’包裹。也可以引用其他变量作为key。   array数组和Collection集合需要注意索引越界，将会抛出异常，map如果指定的key错误，那么会返回null。   集合元素导航同时支持获取值和设置值。

```java
@Test
public void collectionNavigating() {
   
    Object o = new Object();
    SpelBean spelBean = getSpelBean(o);
    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("spelBean", spelBean);
    context.setVariable("o", o);
    //支持集合导航

    //array和Collection使用[index]
    System.out.println(parser.parseExpression("#spelBean.strings[2]").getValue(context));
    System.out.println(parser.parseExpression("#spelBean.stringList[2]").getValue(context));
    System.out.println(parser.parseExpression("#spelBean.stringSet[2]").getValue(context));

    //map使用[key]
    //注意，我们的map中key是一个为"1"的字符串，因此这里需要带上''
    System.out.println(parser.parseExpression("#spelBean.objectObjectMap['1']").getValue(context));
    System.out.println(parser.parseExpression("#spelBean.objectObjectMap[2]").getValue(context));
    //key也可以引用内部变量作为key   返回结果如果在参数中指定为Optional类型，就不需要我们强转了
    Optional value = parser.parseExpression("#spelBean.objectObjectMap[#o]").getValue(context, Optional.class);
    System.out.println(value);

    System.out.println(parser.parseExpression("#spelBean.properties['1']").getValue(context));
    System.out.println(parser.parseExpression("#spelBean.properties['2']").getValue(context));

    System.out.println("------------设置属性值------------");

    parser.parseExpression("#spelBean.strings[2]").setValue(context,"newValue1");
    System.out.println(parser.parseExpression("#spelBean.strings[2]").getValue(context));
    parser.parseExpression("#spelBean.objectObjectMap['1']").setValue(context,"newValue2");
    System.out.println(parser.parseExpression("#spelBean.objectObjectMap['1']").getValue(context));
}

private SpelBean getSpelBean(Object o) {
   
    SpelBean spelBean = new SpelBean();
    String[] strings = new String[]{
   "1", "2", "4", "4"};
    List<String> stringList = Arrays.asList(strings);
    Set<String> stringSet = new HashSet<>(stringList);
    Map<Object, Object> objectObjectMap = new HashMap<>();
    objectObjectMap.put("1", 11);
    objectObjectMap.put(2, 22);
    objectObjectMap.put(3, 33);
    objectObjectMap.put(o, Optional.empty());
    Properties properties = new Properties();
    properties.setProperty("1", "111");
    properties.setProperty("2", "222");
    properties.setProperty("3", "333");
    properties.setProperty("4", "444");
    spelBean.setStrings(strings);
    spelBean.setStringList(stringList);
    spelBean.setStringSet(stringSet);
    spelBean.setObjectObjectMap(objectObjectMap);
    spelBean.setProperties(properties);
    return spelBean;
}
```

## 4.4 内联list

  可以使用{}和特定拆分符号,直接在表达式中直接表示列表（Inline Lists）。另外{}本身就是代表一个空的list集合，因此可以实现集合嵌套。   一定要注意，对于常量数据组成的集合（即集合的元素没有引用到容器中的变量），如果我们设置预期返回类型为List，或者不设置预期返回类型，那么实际上返回是一个java.util.Collections.UnmodifiableList的实例，该集合是不可变的，包括新增、删除、修改操作，都将直接抛出UnsupportedOperationException，只能用来遍历，这么做是为了提升SPEL的解析性能。因此，如果需要后续对集合进行操作，建议使用具体的集合类型接收，比如ArrayLsit.class。

```java
@Test
public void inlineList() {
   
    ExpressionParser parser = new SpelExpressionParser();
    List value = parser.parseExpression("{1,2,3,4}").getValue(List.class);
    System.out.println(value);
    System.out.println(value.getClass());

    //集合嵌套
    List value1 = parser.parseExpression("{
   {},{'a','b'},{'x','y'}}").getValue(ArrayList.class);
    System.out.println(value1);
    System.out.println(value1.getClass());
    //{}本身就是一个集合，可以实现集合嵌套
    for (Object o : value1) {
   
        System.out.println(o.getClass());
        List list0= (List) o;
        System.out.println(list0);
    }

    //一定要注意，如果我们设置预期类型为List，或者不设置返回类型，那么实际上返回是一个java.util.Collections.UnmodifiableList的实例
    //该集合是不可变的，包括新增、删除、修改操作，都将直接抛出UnsupportedOperationException，只能用来遍历
    //因此，如果需要后续对集合进行操作，建议使用具体的集合类型接收，比如ArrayLsit
    //value.set(1, 2);
}
```

## 4.5 内联map

  我们还可以使用{key：value}表示法直接在表达式中直接表示map（Inline Maps），元素之间使用,分隔。另外{:}本身就是代表一个空的map集合，因此可以实现集合嵌套，map的key或者value也可以是一个map。   一定要注意，对于常量数据组成的集合（即集合的key或者value没有引用到容器中的变量），如果我们设置预期返回类型为List，或者不设置预期返回类型，那么实际上返回是一个java.util.Collections.UnmodifiableMap的实例，该集合是不可变的，包括新增、删除、修改操作，都将直接抛出UnsupportedOperationException，只能用来遍历，这么做是为了提升SPEL的解析性能。因此，如果需要后续对集合进行操作，建议使用具体的集合类型接收，比如HashMap.class（实际上返回LinkedHashMap）。

```java
@Test
public void inlineMap() {
   
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("value", "value");
    ExpressionParser parser = new SpelExpressionParser();
    //常量数据集合
    System.out.println(parser.parseExpression("{1:2,2:3,3:4}").getValue(context, Map.class).getClass());
    //引用到容器变量的集合
    Map value = parser.parseExpression("{1:#value,2:3,3:4}").getValue(context, Map.class);
    System.out.println(value);
    System.out.println(value.getClass());

    System.out.println("--------map嵌套----------");
    //嵌套
    //指定HashMap类型，实际上返回的是一个LinkedHashMap类型
    HashMap value1 = parser.parseExpression("{
   {1:1,2:2}:2,2:3,3:{'xx':'y',{1,2}:4}}").getValue(HashMap.class);
    System.out.println(value1);
    System.out.println(value1.getClass());
    //{}本身就是一个集合，可以实现集合嵌套
    value1.forEach((k, v) -> {
   
        System.out.println(k);
        System.out.println(k.getClass());
        System.out.println(v);
        System.out.println(v.getClass());
    });

    // 一定要注意，对于常量数据组成的集合（即集合的key或者value没有引用到容器中的变量），如果我们设置预期返回类型为List，
    // 或者不设置预期返回类型，那么实际上返回是一个java.util.Collections.UnmodifiableMap的实例，该集合是不可变的，
    // 包括新增、删除、修改操作，都将直接抛出UnsupportedOperationException，只能用来遍历，这么做是为了提升SPEL的解析性能。
    // 因此，如果需要后续对集合进行操作，建议使用具体的集合类型接收，比如HashMap.class。
    value.put(1, 2);
}
```

## 4.6 构造数组

  在SPEL表达式中，我们可以使用Java代码的方式来构造数组（Array Construction），即new一个数组。**另外，数组可以自动转换为集合，集合也可以自动转换为数组。**

```java
@Test
public void arrayConstruction() {
   
    ExpressionParser parser = new SpelExpressionParser();
    //初始化数组
    int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue();
    System.out.println(Arrays.toString(numbers1));
    //数组可以自动的转换为集合
    System.out.println(parser.parseExpression("new int[4]").getValue(List.class));
    //集合也可以自动的转换为数组
    System.out.println(parser.parseExpression("{1,2,3,4}").getValue(int[].class));

    //初始化数组并赋值
    int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3,3}").getValue();
    System.out.println(Arrays.toString(numbers2));

    //数组可以自动的转换为集合
    System.out.println(parser.parseExpression("new int[]{1,2,3,3}").getValue(List.class));
    System.out.println(parser.parseExpression("new int[]{1,2,3,3}").getValue(Set.class));

    //二维数组
    int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue();
    System.out.println(Arrays.deepToString(numbers3));
    //目前不支持初始化生成多维数组并赋值
    //int[][] numbers4 = (int[][]) parser.parseExpression("new int[][]{
   {1,2},{3,4},{3,4,5,6}}").getValue();
}
```

## 4.7 方法调用

  我们可以在SPEL表达式中使用典型的Java编程语法调用方法，还可以对字符串调用方法。方法参数也支持变量。

```java
@Test
public void methodInvoke() {
   
    ExpressionParser parser = new SpelExpressionParser();
    //对字符串调用方法
    //截取
    System.out.println(parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class));
    //拆分数组
    System.out.println(Arrays.toString(parser.parseExpression("'a,b,c,c'.split(',')").getValue(String[].class)));
    //虽然split方法返回一个数组，实际上也可以转换为集合，自动转换机制
    System.out.println(parser.parseExpression("'a,b,c,c'.split(',')").getValue(Set.class));

    SpelBean spelBean = new SpelBean();
    spelBean.setProperty1("property1");
    spelBean.setProperty2(11);
    //默认Context调用方法
    System.out.println(parser.parseExpression("getProperty1()").getValue(spelBean, String.class));

    //指定Context调用方法
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("spelBean", spelBean);
    context.setRootObject(spelBean);
    System.out.println(parser.parseExpression("getProperty1()").getValue(context, String.class));
}
```

## 4.8 运算符支持

  SPEL支持非常多多种的运算符（Operators）操作。既包括Java语言的，还包括groovy语言的。

### 4.8.1 算数运算符

  SPEL支持在数值之间使用算数运算符（Mathematical Operators）。加法（+）、减法（-）、乘法（*）、除法（/）、指数（^）、取模（%）运算符。   也可以在数字和字符串上使用+运算符，表示连接字符串。

```java
@Test
public void mathematicalOperators() {
   
    ExpressionParser parser = new SpelExpressionParser();
    System.out.println(parser.parseExpression("6+2").getValue());
    System.out.println(parser.parseExpression("6-2").getValue());
    System.out.println(parser.parseExpression("6*2").getValue());
    System.out.println(parser.parseExpression("6/2").getValue());
    System.out.println(parser.parseExpression("6%2").getValue());
    System.out.println(parser.parseExpression("6^2").getValue());
    System.out.println(parser.parseExpression("6+2/2-1*3").getValue());
    System.out.println(parser.parseExpression("2+6^2").getValue());
    //支持括号
    System.out.println(parser.parseExpression("(2+6)^2").getValue());
    //支持其它数值类型计算
    System.out.println(parser.parseExpression("1000.00 - 1e4").getValue());
    System.out.println(parser.parseExpression("2.0 * 3e0 * 4").getValue());
    System.out.println(parser.parseExpression("0x111 * 4").getValue());
    //连接字符串
    System.out.println(parser.parseExpression("'6'+2").getValue());
}
```

### 4.8.2 关系运算符

  SPEL可以在数值之间使用关系运算符（Relational Operators）：&lt; &gt; &lt;= &gt;= == !=。   null算作“不存在”，与null比较大小时，任何实数都大于null，null等于null，null不能参与数值计算。

```java
@Test
public void relationalOperators() {
   
    ExpressionParser parser = new SpelExpressionParser();
    System.out.println(parser.parseExpression("6>2").getValue());
    System.out.println(parser.parseExpression("6>=2").getValue());
    System.out.println(parser.parseExpression("6<2").getValue());
    System.out.println(parser.parseExpression("6<=2").getValue());
    System.out.println(parser.parseExpression("6==2").getValue());
    System.out.println(parser.parseExpression("6!=2").getValue());
    //结合算数运算符
    System.out.println(parser.parseExpression("6!=2*3").getValue());
    System.out.println(parser.parseExpression("6+1!=2*3").getValue());
    System.out.println(parser.parseExpression("6+1>2*3").getValue());

    System.out.println("----null----");
    System.out.println(parser.parseExpression("0 gt null").getValue());
    System.out.println(parser.parseExpression("-11111>null").getValue());
    System.out.println(parser.parseExpression("null==null").getValue());
}
```

  另外，每个运算符符号也可以指定为纯字母的等效实体引用。这样可以避免使用特殊符号对嵌入表达式的文档具有特殊含义的问题（例如在 XML 文档中有时候不能正常解析&gt; &lt;等特殊符号）。


### 4.8.3 逻辑运算符

  SPEL支持逻辑运算符（Logical Operators）: and(&amp;&amp;) or(||) not(!)

```java
@Test
public void logicalOperators() {
   
    ExpressionParser parser = new SpelExpressionParser();
    System.out.println(parser.parseExpression("'srt'.isEmpty() and ''.isEmpty()").getValue());
    System.out.println(parser.parseExpression("'srt'.isEmpty() && ''.isEmpty()").getValue());

    System.out.println(parser.parseExpression("'srt'.isEmpty() or ''.isEmpty()").getValue());
    System.out.println(parser.parseExpression("'srt'.isEmpty() || ''.isEmpty()").getValue());

    System.out.println(parser.parseExpression("not 'srt'.isEmpty()").getValue());
    System.out.println(parser.parseExpression("!'srt'.isEmpty()").getValue());

    System.out.println(parser.parseExpression("true || false ").getValue());
}
```

### 4.8.4 条件（三目）运算符

  SPEL支持三目运算符（Ternary Operator）。语法和Java语法类似：关系运算?运算1(或者直接返回结果):运算2(或者直接返回结果)。支持嵌套。   另外还支持三元运算符的简写形式，ELVIS运算符（Elvis Operator），这种方式Java语言不支持，在groovy语言中支持使用。运算1(或者返回的结果)可以不写，简化格式如下：关系运算?:运算2（或者直接返回结果）。如果满足关系，那么结果返回true，否则进入运算2（或者直接返回结果）。支持嵌套。

```java
@Test
public void elvisOperator() {
   
    ExpressionParser parser = new SpelExpressionParser();
    System.out.println(parser.parseExpression("'srt'.isEmpty()?'yes':'no'").getValue());
    System.out.println(parser.parseExpression("''.isEmpty()?'yes':'no'").getValue());
    //ELVIS运算符，如果满足关系，那么返回true，否则返回"no"
    System.out.println(parser.parseExpression("''.isEmpty()?:'no'").getValue());
    //支持嵌套
    System.out.println(parser.parseExpression("''.isEmpty()?''.isEmpty()?'是':'否':'no'").getValue());
}
```

### 4.8.5 赋值运算符

  SPEL使用赋值运算符（Assignment Operator）：=   通常我们在setValue中为变量或者变量的属性赋值，当然也可以在SPEL表达式中使用=来为变量或者变量的属性赋值。

```java
@Test
public void assignmentOperator() {
   
    SpelBean spelBean = new SpelBean(new Birth("china"));
    spelBean.setProperty1("property1");
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("spelBean", spelBean);
    context.setVariable("xxx", "yyy");
    context.setRootObject(spelBean);
    ExpressionParser parser = new SpelExpressionParser();


    System.out.println(parser.parseExpression("birth.birthplace").getValue(context));
    //通常在setValue方法中为变量/属性赋值
    parser.parseExpression("birth.birthplace").setValue(context, "usa");
    System.out.println(parser.parseExpression("birth.birthplace").getValue(context));

    System.out.println(parser.parseExpression("property1").getValue(context));
    //也可以在SPEL表达式中为属性赋值
    System.out.println(parser.parseExpression("property1='ppp'").getValue(context));
    System.out.println(parser.parseExpression("property1").getValue(context));

    System.out.println(parser.parseExpression("#xxx").getValue(context));
    //也可以在SPEL表达式中为变量赋值
    System.out.println(parser.parseExpression("#xxx='zzz'").getValue(context));
    System.out.println(parser.parseExpression("#xxx").getValue(context));
}
```

### 4.8.6 instanceof运算符

  instanceof在Java语法中也被支持，用来测试某个对象是否属于某个类型。对于基本类型的字面量值，将会立即星星装箱操作，因此匹配的类型不能是基本类型，而应该是包装类型，即1 instanceof t(int)的计算结果将为false，而1 instanceof t(Integer)的计算结果为true。

```java
@Test
public void instanceofOperator() {
   
    ExpressionParser parser = new SpelExpressionParser();
    //是否属于Integer
    System.out.println(parser.parseExpression("'xyz' instanceof T(java.lang.Integer)").getValue());
    //Java的基础类（java.lang包）支持类名简写
    System.out.println(parser.parseExpression("'xyz' instanceof T(Integer)").getValue());
    //是否属于String
    System.out.println(parser.parseExpression("'xyz' instanceof T(String)").getValue());
    System.out.println(parser.parseExpression("'xyz' instanceof T(Object)").getValue());


    System.out.println(parser.parseExpression("11 instanceof T(Integer)").getValue());
    //不能使用基本类型当作类型判断，因为前面的1被立即装箱成为Integer对戏那个
    System.out.println(parser.parseExpression("11 instanceof T(int)").getValue());
}
```

### 4.8.7 matches运算符

  matches运算符就是类似于Java中的String的matches方法。用于判断字符串是否匹配某个正则表达式。

```java
@Test
public void matchesOperator() {
   
    ExpressionParser parser = new SpelExpressionParser();
    System.out.println(parser.parseExpression("'xyz' matches '\\b\\w{3}\\b'").getValue());
    System.out.println(parser.parseExpression("'。。' matches '(.)\\1+'").getValue());
}
```

### 4.8.8 Safe Navigation运算符

  safe navigation operator来自groovy语言，用于避免抛出惹人烦的NullPointerException，实现安全失败的导航操作。   当引用一个对象时，可能需要在访问该对象的方法或属性之前验证它不是空的，如果忘记了，那么会导致NullPointerException。使用Safe Navigation操作符之后，如果引用的对象为null，那么将返回null而不是抛出异常。   safe navigation operator的语法很简单，就是在导航的.之前加上一个?即可。

```java
@Test
public void safeNavigationOperator() {
   
    Birth birth = new Birth("Birth safeNavigationOperator");
    ExpressionParser parser = new SpelExpressionParser();
    //safe navigation operator的语法很简单，就是在导航的.之前加上一个?即可
    System.out.println(parser.parseExpression("birthplace?.replace(' ', '')?.split('a')").getValue(birth));
    System.out.println(parser.parseExpression("birthDate?.getYear()").getValue(birth));

    //下面的没有使用safe navigation operator的操作将会抛出异常
    System.out.println(parser.parseExpression("birthDate.getYear()").getValue(birth));
}
```

## 4.9 T 类型

  SPEL支持使用T(ClassPath)的样式来表示Java的某个类的class对象，同时又可调用该类的静态方法和字段。ClassPath一般填写类的全限定类名，java.lang包中的类只需要写简单类名即可。

```java
@Test
public void tClass() {
   
    ExpressionParser parser = new SpelExpressionParser();
    //调用String的join静态方法
    System.out.println(parser.parseExpression("T(String).join('_','@','@')").getValue());
    //调用Math的random静态方法
    System.out.println(parser.parseExpression("T(Math).random()*100").getValue());
    //调用Integer的MAX_VALUE静态字段
    System.out.println(parser.parseExpression("T(Integer).MAX_VALUE").getValue());
}
```

## 4.10 new 构造器

  SPEL支持使用new 运算符调用构造器，除了Java.lang包中的类之外，其它类型都应该使用全限定类名。构造器中也可以new其他对象作为参数，或者使用其他语法。

```java
@Test
public void constructors() {
   
    ExpressionParser parser = new SpelExpressionParser();
    //java.lang 包中的类使用简单类名即可
    System.out.println(parser.parseExpression("new String('new 一个字符串')").getValue());
    System.out.println(parser.parseExpression("new Exception('new 一个异常')").getValue());

    //其他包中的类使用全路径类名
    //new 一个IoC对象
    System.out.println(parser.parseExpression("new org.springframework.context.support.ClassPathXmlApplicationContext()").getValue());
    //构造器中也可以new其他对象作为参数，使用其他语法
    System.out.println(parser.parseExpression("new com.spring.spel.SpelBean('new 一个普通对象',new com.spring.spel.Birth('new对象',T(java.time.LocalDate).now(),T(java.time.LocalTime).now()))").getValue());
}
```

## 4.11 变量

  SPEL支持使用#variableName语法引用计算上下文容器中的变量（Variables）。变量是通过在EvaluationContext的实现并调用setVariable方法设置的。   变量命名可以由小写字母（a-z）、大写字母（A-Z）、数字（0-9）、下划线（_）、美刀（$）这里几部分组成，第一个字符不能是数字。具有同名变量时，后定义的变量值将会覆盖之前定义的变量值。   引用到变量后，就可以利用变量执行前面所学的操作了，比如调用方法、属性等等操作。

```java
@Test
public void variables() {
   
    Birth birth = new Birth("china");

    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("_variable1", "variable1");
    context.setVariable("variable2", "222");
    context.setVariable("variable2", "variable2");
    context.setVariable("birth", birth);

    System.out.println(parser.parseExpression("#_variable1").getValue(context));
    System.out.println(parser.parseExpression("#variable2").getValue(context));
    System.out.println(parser.parseExpression("#birth").getValue(context));
    System.out.println(parser.parseExpression("#birth.birthplace").getValue(context));
    System.out.println(parser.parseExpression("#birth.birthDate=T(java.time.LocalDate).now()").getValue(context));
    System.out.println(parser.parseExpression("#birth").getValue(context));
    System.out.println(parser.parseExpression("#this").getValue(context, birth));
}
```

### 4.11.1 #this和#root

  #this和#root是预定义的变量。#this表示引用当前被计算的对象。   #root表示上下文容器的根对象rootObject，默认为null。我们可以通过context的setRootObject方法为一个容器设置一个rootObject，随后对该容器引用它的相关属性、方法时，可以不使用#variableName引用，一个容器只有一个rootObject。设置了rootObject之后，直接使用#root和#this都是表示容器的根对象的引用。   另外，如果在getValue中手动设置了当前被评估的对象时，#root表示当前被评估的对象。

```java
@Test
public void thisRoot() {
   
    Birth birth = new Birth("china");
    Birth birth2 = new Birth("usa");
    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();

    //#this表示引用当前被评估的对象。
    System.out.println(parser.parseExpression("#this").getValue(birth));
    //#root默认为null
    System.out.println(parser.parseExpression("#root").getValue());
    //手动设置了当前被评估的对象时，#root表示当前被评估的对象。
    System.out.println(parser.parseExpression("#root").getValue(birth));
    System.out.println(parser.parseExpression("#root").getValue(birth2));


    context.setVariable("birth", birth);
    context.setVariable("birth2", birth2);
    //设置容器的root根对象。随后对该容器引用它的相关属性、方法时，可以不使用#variableName引用，#root就表示引用根对象
    context.setRootObject(birth);
    System.out.println(parser.parseExpression("#root").getValue(context));
    System.out.println(parser.parseExpression("birthplace").getValue(context));
    System.out.println(parser.parseExpression("birthDate=T(java.time.LocalDate).now()").getValue(context));
    System.out.println(parser.parseExpression("#root").getValue(context));


    //设置了rootObject之后，直接使用#this表示容器的根对象的引用
    System.out.println(parser.parseExpression("#this").getValue(context));
    //手动设置了当前被评估的对象时，#root表示当前被评估的对象。
    System.out.println(parser.parseExpression("#root").getValue(birth2));

    //#this更常被用来进行集合的筛选
    context.setVariable("primes", new ArrayList<>(Arrays.asList(2, 3, 5, 7, 11, 13, 17)));
    //使用selection 集合选择的语法 后面会讲到
    System.out.println(parser.parseExpression("#primes.?[#this>10]").getValue(context, List.class));
}
```

## 4.12 函数

  除了注册变量之外，SPEL还支持通过EvaluationContext注册函数（Functions）并进行调用。   **这里所说的函数，实际上就是Java中的Method类实例，是对方法的抽象，利用到了反射的技术，这个emethod实例被称为“函数引用”。注意，只能是静态的方法才能通过函数引用方式调用。**

```java
private static class TimeConverter {
   
        /**
         * 需要被注册的"函数"
         * 注意：只能是静态的方法才能通过函数引用方式调用
         */
        private static String convert(byte type) {
   
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

    @Test
    public void functions() throws NoSuchMethodException {
   
        StandardEvaluationContext context = new StandardEvaluationContext();
        //通过反射需要被注册的"函数"
        Method convert = TimeConverter.class.getDeclaredMethod("convert", byte.class);
        //注册到计算上下文容器中 这两方法都行
        //context.setVariable("convert", convert);
        context.registerFunction("convert", convert);
        //调用函数 传入参数即可获取结果
        ExpressionParser parser = new SpelExpressionParser();
        System.out.println(parser.parseExpression("#convert(1)").getValue(context));
        System.out.println(parser.parseExpression("#convert(2)").getValue(context));
        System.out.println(parser.parseExpression("#convert(3)").getValue(context));
    }
```

## 4.13 Bean引用

  SPEL支持使用“@beanname”来引用IoC容器中的bean。这需要在计算上下文（EvaluationContext）中配置一个bean解析器（BeanResolver）实例，在计算其间将通过该实例的resolve方法查找bean，Spring提供默认的BeanFactoryResolver实现，它接受一个IoC容器实例，在resolve方法中将会调用容器的getBean方法查找该容器中的所有bean。   找到bean之后就可以对bean进行各种计算、设置等操作。

```java
@Test
public void beanReferences() {
   
    //初始化Spring容器，这里手动添加两个bean进取
    Birth birth = new Birth("Birth Bean References");
    SpelBean spelBean = new SpelBean("SpelBean Bean References", birth);
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext();
    ac.refresh();
    ConfigurableListableBeanFactory beanFactory = ac.getBeanFactory();
    beanFactory.registerSingleton("birth", birth);
    beanFactory.registerSingleton("spelBean", spelBean);


    StandardEvaluationContext context = new StandardEvaluationContext();
    //设置Bean解析器实例，这个BeanFactoryResolver就是Spring默认的解析器，
    //它接受一个IoC容器实例，在resolve方法中将会调用容器的getBean方法查找该容器中的所有bean。
    context.setBeanResolver(new BeanFactoryResolver(ac));

    
    //尝试通过 @beanname 引用bean
    ExpressionParser parser = new SpelExpressionParser();
    System.out.println(parser.parseExpression("@birth").getValue(context));
    System.out.println(parser.parseExpression("@birth.birthplace").getValue(context));
    System.out.println(parser.parseExpression("@spelBean").getValue(context));

    System.out.println("--------通过SPEL设置bean的值----------");
    System.out.println(ac.getBean("birth"));
    parser.parseExpression("@birth.birthDate").setValue(context, LocalDate.now());
    parser.parseExpression("@birth.birthDate").setValue(context, LocalDate.now());
    //容器中获取bean的实例并输出
    System.out.println(ac.getBean("birth"));
}
```

## 4.14 集合筛选

  SPEL支持集合元素的筛选操作（Collection Selection），允许通过从源集合的元素中选择符合条件的元素转换为另一个集合。这类似于Java8的lambda的filter操作。语法格式如下：

1. .?[selectionExpression]，它将检索与选择表达式匹配的集合全部元素，返回这些匹配的原始元素的新集合。 
2. .^ [selectionExpression]，它将检索并获取与选择表达式匹配的第一个元素。对于本身无序的集合不能保证返回某一个确定的元素。对于map，返回的是一个map。 
3. .$[selectionExpression]，它将检索并获取与选择表达式匹配的最后一个元素。对于本身无序的集合不能保证返回某一个确定的元素。对于map，返回的是一个map。 
4. selectionExpression应该返回一个boolean类型的结果。array和Collection的[] 
5. 中的#this表示集合元素本身，map的[]中的#this同样表示集合元素，不过是Map.Entry，我们可以获取key或者value进行筛选。访问集合元素的属性、方法时，可以省略#this引用前缀。

```java
@Test
public void collectionSelection() {
   
    SpelBean spelBean = new SpelBean();
    String[] strings = new String[]{
   "1", "2", "4", "4"};
    List<String> stringList = Arrays.asList(strings);
    Set<String> stringSet = new HashSet<>(stringList);
    Map<Object, Object> objectObjectMap = new HashMap<>();
    objectObjectMap.put("1", 11);
    objectObjectMap.put(2, 22);
    objectObjectMap.put(3, 33);
    objectObjectMap.put("4", Optional.empty());
    Properties properties = new Properties();
    properties.setProperty("1", "111");
    properties.setProperty("2", "222");
    properties.setProperty("3", "333");
    properties.setProperty("4", "444");
    spelBean.setStrings(strings);
    spelBean.setStringList(stringList);
    spelBean.setStringSet(stringSet);
    spelBean.setObjectObjectMap(objectObjectMap);
    spelBean.setProperties(properties);
    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("spelBean", spelBean);

    /*
     * array和Collection的筛选
     * array和Collection的[]中的#this表示集合元素本身
     * 访问集合元素的属性、方法时，可以省略#this引用前缀
     */

    //直接使用集合
    System.out.println(parser.parseExpression("#this.?[#this.equals('4')]").getValue(strings, List.class));
    //直接使用对象的集合  省略引用元素的#this前缀
    System.out.println(parser.parseExpression("stringList.?[equals('2')]").getValue(spelBean));
    //容器的变量中的集合
    System.out.println(parser.parseExpression("#spelBean.strings.?[#this.equals('4')]").getValue(context));

    System.out.println(parser.parseExpression("#spelBean.stringSet.?[T(Integer).parseInt(#this) < 3]").getValue(context));

    /*
     * map的筛选
     * map的[]中的#this同样表示集合元素,不过是Map.Entry
     * 我们可以获取key或者value进行筛选
     * 访问集合元素的属性、方法时，可以省略#this引用前缀
     */
    //直接使用map 根据key筛选
    System.out.println(parser.parseExpression("#this.?[T(Integer).parseInt(#this.key)<3]").getValue(objectObjectMap));
    //直接使用map 根据value筛选  省略引用Entry元素的#this前缀
    System.out.println(parser.parseExpression("#this.?[T(Integer).parseInt(value)<300]").getValue(properties));
    //直接使用对象的map  省略引用元素的#this前缀
    System.out.println(parser.parseExpression("objectObjectMap.?[key.equals('4')]").getValue(spelBean));
    //容器的变量中使用map
    System.out.println(parser.parseExpression("#spelBean.properties.?[key.equals('3')]").getValue(context));



    System.out.println("检索第一个匹配的元素");
    //map是无序的
    System.out.println(parser.parseExpression("#this.^[true]").getValue(properties));
    System.out.println(parser.parseExpression("#this.^[true]").getValue(objectObjectMap));
    //list是有序的
    System.out.println(parser.parseExpression("#this.^[true]").getValue(stringList));
    System.out.println("检索最后一个匹配的元素");
    System.out.println(parser.parseExpression("#this.$[true]").getValue(properties));
    System.out.println(parser.parseExpression("#this.$[true]").getValue(objectObjectMap));
    System.out.println(parser.parseExpression("#this.$[true]").getValue(stringSet));
    System.out.println(parser.parseExpression("#this.$[true]").getValue(strings));
}
```

## 4.15 集合投影

  SPEL支持集合投影（Collection Projection）。**简单的说，就是对集合元素进行计算，根据返回的结果生成新的集合，这类似于Java8的Stream的map操作。**

1. 集合投影语法格式如下：.![projectionExpression]。 
2. projectionExpression必须返回一个结果，返回的结果将整合为一个list列表。 
3. array和Collection的[]中的#this表示集合元素本身，map的[]中的#this同样表示集合元素，不过是Map.Entry，我们可以获取key或者value进行筛选。访问集合元素的属性、方法时，可以省略#this引用前缀。

```java
@Test
public void collectionProjection() {
   
    SpelBean spelBean = new SpelBean();
    String[] strings = new String[]{
   "a", "b", "c", "d"};
    List<String> stringList = Arrays.asList(strings);
    Set<String> stringSet = new HashSet<>(stringList);
    Map<Object, Object> objectObjectMap = new HashMap<>();
    objectObjectMap.put("1", 11);
    objectObjectMap.put(2, 22);
    objectObjectMap.put(3, 33);
    objectObjectMap.put("4", Optional.empty());
    Properties properties = new Properties();
    properties.setProperty("1", "111");
    properties.setProperty("2", "222");
    properties.setProperty("3", "333");
    properties.setProperty("4", "444");
    spelBean.setStrings(strings);
    spelBean.setStringList(stringList);
    spelBean.setStringSet(stringSet);
    spelBean.setObjectObjectMap(objectObjectMap);
    spelBean.setProperties(properties);
    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("spelBean", spelBean);

    /*array和Collection的投影*/

    //直接使用集合 省略引用元素的#this前缀
    System.out.println(parser.parseExpression("#this.![equals('B')]").getValue(strings, List.class));
    //直接使用对象的集合
    System.out.println(parser.parseExpression("stringList.![#this+'--']").getValue(spelBean));
    //容器的变量中的集合
    System.out.println(parser.parseExpression("#spelBean.strings.![toUpperCase()]").getValue(context, List.class));

    System.out.println(parser.parseExpression("#spelBean.stringSet.![toUpperCase()]").getValue(context));

    /*map的投影*/
    //直接使用map 根据key筛选
    System.out.println(parser.parseExpression("#this.![T(Integer).parseInt(#this.key)+3]").getValue(objectObjectMap));

    //直接使用map 根据value筛选  省略引用Entry元素的#this前缀
    System.out.println(parser.parseExpression("#this.![T(Integer).parseInt(value)+300]").getValue(properties));
    //直接使用对象的map  省略引用元素的#this前缀
    System.out.println(parser.parseExpression("objectObjectMap.![key.equals('4')]").getValue(spelBean));
    //容器的变量中使用map
    System.out.println(parser.parseExpression("#spelBean.properties.![key.equals('3')]").getValue(context));
}
```

## 4.16 表达式模版化

  表达式模板（Expression templates）允许将文本与一个或多个计算表达式组合在一起。每个计算表达式都用可定义的前缀和后缀字符分隔，常见的选择是使用“#{”作为表达式的前缀，“}”作为表达式的后缀，中间的部分就是计算表达式。   在计算时我们需要传递一个ParserContext接口的实例，用于定义表达式模版的解析方式，即表达式的前缀和后缀样式。Spring为我们提供了默认实现TemplateParserContext以及ParserContext.TEMPLATE_EXPRESSION，它们都是默认以#{}分隔计算表达式的。   对所有表达式计算完毕之后,将会把结果填充到原计算表达式的位置，和文本拼接起来，然后一起输出。

```java
@Test
public void expressionTemplates() {
   
    ExpressionParser parser = new SpelExpressionParser();
    ParserContext parserContext=ParserContext.TEMPLATE_EXPRESSION;
    System.out.println(parser.parseExpression("My First Expression Template name is #{#this}",
            parserContext).getValue(
            "Gaia"));

    System.out.println(parser.parseExpression("random number is #{T(java.lang.Math).random()}",
            parserContext).getValue());

    System.out.println(parser.parseExpression("There are two random numbers: #{T(java.lang.Math).random()},#{T(java.lang.Math).random()}",
            parserContext).getValue());

    System.out.println("-----复杂模版-----");

    StandardEvaluationContext context = new StandardEvaluationContext();
    context.setVariable("num", 5);
    context.setVariable("cost", 3214);
    context.setVariable("unitPrice", 5000);
    context.setVariable("totalPrices", 0);

    System.out.println(parser.parseExpression("The total price before the calculation: #{#totalPrices}",parserContext).getValue(context));
    System.out.println(parser.parseExpression("The car cost #{#cost}, sold #{#num} units, sold at a unit price of " +
                    "#{#unitPrice}, and delivered revenue of #{#totalPrices=(#unitPrice - #cost)*#num}.",
            parserContext).getValue(context));

    System.out.println(parser.parseExpression("The calculated total price: #{#totalPrices}",parserContext).getValue(context));
}
```


  现在，我们将SpEL表达式与Spring的基于XML或基于注释的配置元数据一起使用。如果我们学习了前面的SPEL语法，我们可以定义出精美的SPEL表达式，同时我们会发现，此前的文章中对于一些表达式的定义显得过于复杂了，比如对于数组、集合的定义，根本不需要split方法拆分。   **在基于XML或基于注释这两种情况下，定义表达式的语法都是#{表达式字符串} 。**   **表达式字符串的语法和单独使用SPEL表达式的语法都差不多。一点小小的区别就是通过bean name引用容器中的bean的不需要加上#或者@前缀了，按bean name引用即可。并且，在SPEL表达式字符串中，也可以使用${key}引用外部属性的配置，注意如果是字符串，建议加上’ '限制。**

## 5.1 XML配置


  在XML配置中，可以使用SPEL表达式设置属性或构造函数参数值。   ApplicationContext中的所有bean都可作为预定义的变量，可以使用bean name引用。这还包括一些内置的标准上下文 bean，比如访问系统属性的systemProperties（一个map），以及用于访问运行时环境的systemEnvironment。   首先加入一个spring-config.xml和一个properties文件，因为Spring的SPEL表达式中可以通过${key}引用外部配置。 ![img](https://img-blog.csdnimg.cn/2020091017182283.png#pic_center)   kv.properties属性如下：

```java
birthplace=100.0
property1=one
property2=10
strings=strings
stringList=stringList
stringSet={1,2,3,3,'${strings}'}
objectObjectMap.key=new Object()
objectObjectMap.value={1,2,3}
properties={1:2,2:3,3:4}
```

  Spring核心配置文件的配置如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--引入配置文件 SPEL中可以使用${key}获取配置文件的属性 注意如果是字符串需要加上'' 。如果value中仅仅注入属性文件的字面量值，那么使用${key}就行了-->
    <context:property-placeholder location="classpath:kv.properties"/>

    <!--XML使用SPEL表达式-->
    <bean class="com.spring.spel.Birth" name="birth">
        <constructor-arg name="birthplace" value="#{T(java.lang.Math).random() * ${birthplace}}"/>
        <constructor-arg name="birthDate" value="#{T(java.time.LocalDate).now()}"/>
        <constructor-arg name="birthTime" value="#{T(java.time.LocalTime).now()}"/>

        <!--引用预定义bean systemProperties的属性，systemProperties是一个map-->
        <property name="birthplace" value="#{systemProperties['sun.desktop']}"/>
    </bean>

    <bean class="com.spring.spel.SpelBean" name="spelBean">
        <!--SPEL引用 容器中的bean-->
        <constructor-arg name="birth" value="#{birth}"/>
        <!--引用bean 属性-->
        <constructor-arg name="property1" value="#{birth.birthplace+'${property1}'}"/>

        <property name="property2"
                  value="#{T(java.util.concurrent.ThreadLocalRandom).current().nextInt(${property2})}"/>
        <!--{}list集合实际上可以自动转换为数组-->
        <property name="strings" value="#{
    {1,2,3,'${strings}'}}"/>
        <!--collection list 集合-->
        <property name="stringList" value="#{
    {1,2,3,3,'${strings}'}}"/>
        <!--collection set 集合 自动去重-->
        <property name="stringSet" value="#{${stringSet}}"/>

        <!--map 集合-->
        <property name="objectObjectMap"
                  value="#{
    {${property1}:2,'2 -2':3,${objectObjectMap.key}:${objectObjectMap.value},'xx':'yy'}}"/>
        <!--map 集合-->
        <property name="properties" value="#{${properties}}"/>
    </bean>
</beans>
```

  测试：

```java
@Test
public void spelXml() {
   
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
    System.out.println(ac.getBean("birth", Birth.class));
    System.out.println(ac.getBean("spelBean", SpelBean.class));
}
```

  结果如下，完美注入：

```java
Birth{
   birthplace='windows', birthDate=2020-09-10, birthTime=14:56:49.277}
SpelBean{
   property1='windowsone', property2=3, birth=Birth{
   birthplace='windows', birthDate=2020-09-10, birthTime=14:56:49.277}, strings=[1, 2, 3, strings], stringList=[1, 2, 3, 3, strings], stringSet=[1, 2, 3, strings], objectObjectMap={
   one=2, 2 -2=3, java.lang.Object@327b636c=[1, 2, 3], xx=yy}, properties={
   3=4, 2=3, 1=2}}
```

## 5.2 注解配置

  注解配置和XML配置中的SPEL语法都是差不多的！**注解配置使用@Value注解，@Value注解可以标注在字段，方法、方法/构造器的参数上。**   基于注解配置的类如下，这些注解我们在上一篇文章中已经讲过了：

```java
@Configuration
//引入配置文件
@PropertySource(value = "classpath:kv.properties", encoding = "UTF-8")
@ComponentScan("com.spring.spel")
public class AnnotationDemo {
   
    private String property1;
    @Value("#{T(java.util.concurrent.ThreadLocalRandom).current().nextInt(${property2})}")
    private int property2;
    private AnnotationDemo2 annotationDemo2;
    @Value("#{
   {1,2,3,'${strings}'}}")
    private String[] strings;
    private List<String> stringList;
    private Set<String> stringSet;
    @Value("#{
   {${property1}:2,'2 -2':3,${objectObjectMap.key}:${objectObjectMap.value},'xx':'yy'}}")
    private Map<Object, Object> objectObjectMap;
    @Value("#{${properties}}")
    private Properties properties;

    @Value("#{
   {'1','0','1','1'}}")
    private char[] chars;
    @Value("#{
   {1,2,3,0,11,2}}")
    private byte[] bytes;
    @Value("#{
   {1,0,1,1}}")
    private float[] floats;
    @Value("#{
   {1,0,1,1}}")
    private double[] doubles;
    @Value("#{
   {'1','0','1','1'}}")
    private boolean[] booleans;


    public AnnotationDemo(@Value("#{annotationDemo2.birthplace+'${property1}'}") String property1, @Value(
            "#{annotationDemo2}") AnnotationDemo2 annotationDemo2) {
   
        this.property1 = property1;
        this.annotationDemo2 = annotationDemo2;
    }

    @Value("#{
   {1,2,3,3,'${strings}'}}")
    public void setStringList(List<String> stringList) {
   
        this.stringList = stringList;
    }

    public void setStringSet(@Value("#{${stringSet}}") Set<String> stringSet) {
   
        this.stringSet = stringSet;
    }

    @Override
    public String toString() {
   
        return "AnnotationDemo{" +
                "property1='" + property1 + '\'' +
                ", property2=" + property2 +
                ", annotationDemo2=" + annotationDemo2 +
                ", strings=" + Arrays.toString(strings) +
                ", stringList=" + stringList +
                ", stringSet=" + stringSet +
                ", objectObjectMap=" + objectObjectMap +
                ", properties=" + properties +
                ", chars=" + Arrays.toString(chars) +
                ", bytes=" + Arrays.toString(bytes) +
                ", floats=" + Arrays.toString(floats) +
                ", doubles=" + Arrays.toString(doubles) +
                ", booleans=" + Arrays.toString(booleans) +
                '}';
    }
}
//…………………………
@Component
public class AnnotationDemo2 {
   
    private String birthplace;
    private LocalDate birthDate;
    @Value("#{T(java.time.LocalTime).now()}")
    private LocalTime birthTime;

    @Value("#{T(java.time.LocalDate).now()}")
    public void setBirthDate(LocalDate birthDate) {
   
        this.birthDate = birthDate;
    }

    public AnnotationDemo2(@Value("#{T(java.lang.Math).random() * ${birthplace}}")
                                   String birthplace) {
   
        this.birthplace = birthplace;
    }

    /**
     * 导航的属性，要求必须是public修饰的或者提供了相应的getter方法
     */
    public String getBirthplace() {
   
        return birthplace;
    }

    @Override
    public String toString() {
   
        return "AnnotationDemo2{" +
                "birthplace='" + birthplace + '\'' +
                ", birthDate=" + birthDate +
                ", birthTime=" + birthTime +
                '}';
    }
}
```

  测试：

```java
@Test
public void spelAnnotation () {
   
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AnnotationDemo.class);
    System.out.println(ac.getBean("annotationDemo", AnnotationDemo.class));
    System.out.println(ac.getBean("annotationDemo2", AnnotationDemo2.class));
}
```

  结果如下，成功注入：

```java
AnnotationDemo{
   property1='81.27838326783505one', property2=9, annotationDemo2=AnnotationDemo2{
   birthplace='81.27838326783505', birthDate=2020-09-10, birthTime=15:37:47.023}, strings=[1, 2, 3, strings], stringList=[1, 2, 3, 3, strings], stringSet=null, objectObjectMap={
   one=2, 2 -2=3, java.lang.Object@51399530=[1, 2, 3], xx=yy}, properties={
   3=4, 2=3, 1=2}, chars=[1, 0, 1, 1], bytes=[1, 2, 3, 0, 11, 2], floats=[1.0, 0.0, 1.0, 1.0], doubles=[1.0, 0.0, 1.0, 1.0], booleans=[true, false, true, true]}
AnnotationDemo2{
   birthplace='81.27838326783505', birthDate=2020-09-10, birthTime=15:37:47.023}
```


  前两篇文章中我们都接触过SPEL，但是都讲得比较笼统，本文则全面的介绍了SPEL的大部分核心语法，相当于对前文的补充。   Spring表达式语言（简称“SpEL”）是一种功能强大的表达式语言，我们一般用的最多的功能就是结合外部属性文件注入字面量属性，或者注入集合属性，能为我们的Spring项目开发节省大量代码，提升了编程效率。

**相关文章：**   Spring官网： [https://docs.spring.io/](https://docs.spring.io/)   XML： [Spring 5.x 学习(2)—两万字的IoC入门以及基于XML的IoC配置全解](https://blog.csdn.net/weixin_43767015/article/details/108380043)   注解： [Spring 5.x 学习(3)—两万字的基于注解的IoC配置全解](https://blog.csdn.net/weixin_43767015/article/details/108467731)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

