

>  基于最新Spring 5.x，详细介绍了Spring JDBC(JdbcTemplate)框架的使用！

  Spring JDBC是对传统JDBC访问的简单封装，使用Spring JDBC之后，可以省去一部分以前需要开发人员编写的访问数据的底层操作，比如注册驱动、获得连接、执行查询等等。Spring JDBC相当于一个简单封装的持久层框架，原始功能比较简单，使用起来也比较简单，如果开发一些小型项目，那还是可以直接使用的，如果是一些大型项目，由于它并不是真正的orm框架，因此需要自己封装一些工具，如果有能力封装的话，那么Spring JDBC用起来是非常舒服的，性能也很强，不比mybatis差！





  下表展示了Spring JDBC能为我们做了哪些事，我们自己又需要做哪些事。这些事在以前都是需要开发人员做的。


  **Spring Framework 的 JDBC框架由四个不同的包组成：**

1. **core**：org.springframework.jdbc.core包包含**JdbcTemplate**类及其各种回调接口，以及各种相关类。simple子包含**SimpleJdbcInsert**和**SimpleJdbcCall**类。namedparam子包中包含**NamedParameterJdbcTemplate**类和相关的支持类。 
2. **datasource**：org.springframework.jdbc.datasource包包含一个实用程序类，用于轻松访问DataSource和各种简单DataSource实现，比如**SimpleDriverDataSource**、**DriverManagerDataSource**等，可用于在Java EE容器之外测试和运行未修改的JDBC代码。embedded子包支持使用 Java 数据库引擎（如 HSQL、H2 和 Derby）创建嵌入式数据库。 
3. **object**：org.springframework.jdbc.object包包含将RDBMS查询、更新和存储过程表示为线程安全的可重用的对象的类，比如**SqlQuery**、**SqlUpdate**等。这种更高级别的 JDBC 抽象取决于 org.springframe.jdbc.core 包中的较低级别的抽象。 
4. **support**：org.springframework.jdbc.support包提供**SQLException**异常转换功能和一些实用程序类。JDBC处理期间引发的异常将转换为org.springframework.dao包中定义的异常。这意味着使用Spring JDBC抽象层的代码不需要实现JDBC或RDBMS特定的错误处理。所有转换的异常都是非受检异常，因此可以选择捕获异常获取传播到调用方。


  **Spring JDBC采用一些模板类来操作数据库，用以简化数据库操作，其中最著名的就是JdbcTemplate，JdbcTemplate是spring-jdbc包中的核心类，属于Spring Framework核心组件！**   **JdbcTemplate是对原始JDBC API对象的封装、改造之后的操作模板类，它就是用于和数据库交互的，实现对表的CRUD等所有操作。Spring中凡是带有xxxTemplate字样的类，基本上都是对于原生API的二次封装，常见模版比如HibernateTemplate、RedisTemplate、JmsTemplate、RestTemplate等等。**   **JdbcTemplate简化了原始JDBC的使用步骤，而使用JdbcTemplate之后，我们只需要提供sql语句和获取结果集就行了，同时可以很轻松的配置使用Spring的声明式事物。Spring官网就说过JdbcTemplate的功能：“The Spring Framework takes care of all the low-level details that can make JDBC such a tedious API.”，即Spring框架负责处理所有低级的乏味的JDBC API。**   **实际上，Spring JDBC还提供了更多的可以访问数据库的模板类：**

1. **JdbcTemplate**：最经典且最流行的Spring JDBC访问模版，提供了基本的Spring JDBC访问数据的功能！ 
2. **NamedParameterJdbcTemplate**：Spring 2.0新增的模版，内部包装了一个JdbcTemplate对象并进行功能扩展，提供了“named parameters（命名参数）”这个新特性，不再需要使用传统的JDBC 的“？”占位符代表sql参数。 
3. **SimpleJdbcInsert**：Spring 2.5新增的模版，一个多线程的、可重用的对象，为数据库表提供了方便的插入功能。它提供元数据处理，以简化构造基本插入语句所需的编码，只需要提供表的名称和包含列名和列值的映射。实际插入仍是使用JdbcTemplate。 
4. **SimpleJdbcCall**：Spring 2.5新增的模版，一个多线程的、可重用的对象，表示对存储过程（procedure）和函数（function）的调用。它提供元数据处理来简化访问基本存储过程/函数所需的编码，只需要提供存储过程/函数的名称和在执行调用时包含参数的Map。提供的参数的名称将与创建存储过程时声明的输入（in）和输出（out）参数匹配。 
5. **RDBMS对象**：Spring JDBC提供了数据库RDBMS操作的Java抽象，包括MappingSqlQuery、SqlUpdate、StoredProcedure，分别代表可重用的sql查询操作抽象（还支持结果映射）对象、可重用的sql更新操作对象、可重用的存储过程/函数调用对象。它们的特点是可以安全的重复使用！

  **我们主要学习JdbcTemplate的使用！**


## 3.1 JdbcTemplate的概述

  **JdbcTemplate 是 Spring JDBC 的核心类，我们学习Spring JDBC的使用，主要就是学习这个JdbcTemplate核心类的使用！**   **JdbcTemplate主要功能如下：**

1. 数据库CRUD操作以及存储过程/函数执行。 
2. 对结果集ResultSet 执行迭代并提取、封装为指定类型的结果。 
3. 捕获 JDBC 异常并将其转换为 org.springframe.dao 包中定义的通用、信息更丰富的异常层次结构。 
4. JdbcTemplate的参数化sql方法使用PreparedStatement，可以防止sql注入。

  **JdbcTemplate的方法按照名字和作用可以为分几大类：**

1. **update**：执行表数据的新增、修改、删除等语句，batchUpdate则是支持批处理语句； 
2. **query**：执行表数据查询语句。 
3. **call**：执行存储过程、函数等高级语句。 
4. **execute**：自定义执行任何的sql语句，包括对数据库、表、字段本身的DDL操作，包括表数据的增删改查的操作，包括存储过程、函数等高级语句的调用操作。

  **JdbcTemplate必须和DataSource数据源一起使用。常见的Java数据源有：**

1. **DBCP**：老数据源了，来自Apache Commons，现在维护的还挺正常的。https://github.com/apache/commons-dbcp。 
2. **c3p0**：老数据源了，也还在用，但是维护的没有DBCP多。https://github.com/swaldman/c3p0。 
3. **Tomcat-jdbc**：来自tomcat， Apache Commons DBCP 连接池的一种替换或备选方案，提供了更多功能比DBCP更多的功能，现在维护的还挺正常的。 
4. **Proxool**：老古董了，作者在2016年声明已放弃维护，声明说他本人甚至在2006年的时候就不使用Java语言了……。https://github.com/proxool/proxool。 
5. **BoneCP**：老古董了，性能击败了DBCP、c3p0，但作者在2015年声明已放弃维护，还推荐使用HikariCP……。https://github.com/wwadge/bonecp。 
6. **Druid**：后起之秀。alibaba的，大品牌，值得信赖。在国内，目前的新项目应该是使用的最多的数据源吧。速度非常快，并且除了管理数据库连接这个老本行之外，还提供了监控功能，比如日志监控、sql性能监控等，人家自己介绍的就是“为监控而生的数据库连接池”，维护的很不错。项目地址：https://github.com/alibaba/druid。官方数据：https://github.com/alibaba/druid/wiki/Druid%E8%BF%9E%E6%8E%A5%E6%B1%A0%E4%BB%8B%E7%BB%8D。 
7. **HikariCP**：后起之秀。这里的Hikari来源日语，翻译过来就是“光”，就像“光”一样快？看来作者是个老二次元了。人家自己介绍的就是“Fast, simple, reliable”，就是一个存粹的非常非常快的连接池，老牛逼了，维护的也很好。https://github.com/brettwooldridge/HikariCP 
<li>Spring Boot 2.0开始，默认数据源就是**HikariCP**，其他的框架中，HikariCP使用的也比Druid多，国外更多的是使用HikariCP，但是在国内，Druid用的更多。关于Druid和HikariCP的“官方撕逼”1：https://github.com/brettwooldridge/HikariCP/issues/232。“官方撕逼”2：https://www.zhihu.com/question/53892749/answer/518733788。**所以说，现在的新项目应该用什么数据源连接池，不用我说了吧？**</li>

  另外，某些开发者使用Spring提供的**DriverManagerDataSource**或者**SimpleDriverDataSource**数据源，然而该类**只不过实现相同的标准接口，它们确实是一个Spring框架自带的数据源，但是不支持连接池管理，每次连接数据库都是创建新的连接对象**，这个类自己去看看源码甚至注释就知道了，这些数据源仅仅用于测试！

## 3.2 配置JdbcTemplate

### 3.2.1 配置依赖

  **本文我们将使用Druid数据源，同时采用注解开发！maven依赖如下，其中spring-context提供Spring核心机制的支持，spring-jdbc提供Spring JDBC的操作支持。**

```java
<properties>
    <spring-framework.version>5.2.8.RELEASE</spring-framework.version>
    <mysql-connector-java>8.0.16</mysql-connector-java>
    <druid>1.2.3</druid>
    <lombok>1.18.12</lombok>
    <junit>4.12</junit>
</properties>
<dependencies>
    <!--spring 核心组件所需依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring-framework.version}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
    <!--Spring 测试-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring-framework.version}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <!--spring-jdbc-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring-framework.version}</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <!--mysql数据库驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql-connector-java}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
    <!--druid数据源-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid}</version>
    </dependency>
    <!--单元测试-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit}</version>
    </dependency>
</dependencies>
```

### 3.2.2 配置JdbcTemplate

  **JdbcTemplate是线程安全的，它内部的状态是dataSource，不会影响会话状态，因此可以将其交给IoC容器管理，然后在DAO中直接注入单例bean实例即可。如果应用程序访问多个数据库，这需要多个数据源，随后需要多个配置不同数据源的 JdbcTemplate 实例。**   **可以使用XML配置：**

```java
<bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url"
              value="jdbc:mysql://xxx"/>
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg name="dataSource" ref="druidDataSource"/>
</bean>
```

  **也可以使用Java Config方式配置，这种方式目前更流行，我们下面的案例都是使用Java Config方式配置，舍弃了XML文件！**

```java
@ComponentScan
@Configuration
public class JdbcStart {
   

    /**
     * 配置JdbcTemplate
     */
    @Bean
    public JdbcTemplate jdbcTemplate() {
   
        return new JdbcTemplate(druidDataSource());
    }

    /**
     * 配置Druid数据源
     */
    @Bean
    public DruidDataSource druidDataSource() {
   
        DruidDataSource druidDataSource = new DruidDataSource();
        //为了方便，直接硬编码了，如果使用Spring boot就更简单了
        //简单的配置数据库连接信息，其他连接池信息采用默认配置
        druidDataSource.setUrl("jdbc:mysql://xxx");
        druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("123456");
        return druidDataSource;
    }
}
```

### 3.2.3 数据库表

  本人数据库是MySql 8版本。数据库表：

```java
CREATE TABLE `jt_study` (
	`id` INT ( 11 ) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR ( 200 ) DEFAULT NULL COMMENT '姓名',
	`age` INT ( 11 ) DEFAULT NULL COMMENT '年龄',
	`create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
PRIMARY KEY ( `id` ) 
) ENGINE = INNODB AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8;
```

  插入一些数据：

```java
INSERT INTO `jt_study`
VALUES
	( NULL, 'Google', 12, '2019-04-21 15:55:15' ),
	( NULL, '淘宝', 11, CURRENT_TIMESTAMP() ),
	( NULL, '百度', 1, '2018-04-21 15:55:15' ),
	( NULL, '微博', 5, CURRENT_TIMESTAMP() ),
	( NULL, 'Facebook', 5, '2020-04-21 15:55:15' );
```

  实体：

```java
public class JtStudy {
   

    private Date createTime;
    private Integer id;
    private String name;
    private Integer age;


    public Date getCreateTime() {
   
        return createTime;
    }

    public void setCreateTime(Date createTime) {
   
        this.createTime = createTime;
    }

    public Integer getId() {
   
        return id;
    }

    public void setId(Integer id) {
   
        this.id = id;
    }

    public String getName() {
   
        return name;
    }

    public void setName(String name) {
   
        this.name = name;
    }

    public Integer getAge() {
   
        return age;
    }

    public void setAge(Integer age) {
   
        this.age = age;
    }

    @Override
    public String toString() {
   
        return "JtStudy{" +
                "createTime=" + createTime +
                ", id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public JtStudy() {
   }

    public JtStudy(String name, Integer age) {
   
        this.name = name;
        this.age = age;
    }
}
```

### 3.2.4 测试类

  我们使用spring-test进行测试！测试类如下：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = JdbcStart.class)
public class JdbcTest {
   

    @Resource
    private JdbcTemplate jdbcTemplate;
    
}
```

  我们首先看看配置成功没有：

```java
@Test
public void testJdbcTemplate() {
   
    System.out.println(jdbcTemplate);
    System.out.println(Objects.requireNonNull(jdbcTemplate.getDataSource()).getClass());
}
```

  结果如下：

```java
org.springframework.jdbc.core.JdbcTemplate@5e5d171f
class com.alibaba.druid.pool.DruidDataSource
```

  可以看到数据源确实是使用的DruidDataSource数据源，这说明配置成功，下面正式使用。

## 3.3 query操作

  **JdbcTemplate 提供的一系列query方法用来查询数据！**

### 3.3.1 query通用查询

  常用的方法就是query(String sql, RowMapper&lt; T &gt; rowMapper, @Nullable Object… args)。该方法查询给定的 SQL语句，使用PreparedStatement和要绑定到Statement的args参数列表，并且通过 RowMapper 将每一行查询到的数据映射到结果对象。将返回映射之后的结果对象list集合，没有查询到数据将返回一个空集合。   **sql**：要执行的 SQL 查询，对于参数使用“？”占位符表示   **rowMapper**：将每一个查询结果映射到指定对象的回调函数，我们可以自己指定，也可以使用预定义映射函数！**这实际上就是此前传统JDBC编程中对ResultSet的处理！**   **args**：要绑定到查询sql的参数数组。

  查询jt_study表中age为5的数据！

```java
@Test
public void query() {
   
    String sql = "select * from jt_study where age = ?";
    List<JtStudy> jtStudies = jdbcTemplate.query(sql, (rs, rowNum) -> {
   
        /*
         * mapRow映射函数
         * @param rs 要映射的ResultSet结果集
         * @param rowNum 当前行的编号
         * @return 当前行的结果对象
         */
        JtStudy jtStudy = new JtStudy();
        jtStudy.setId(rs.getInt("id"));
        jtStudy.setName(rs.getString("name"));
        jtStudy.setAge(rs.getInt("age"));
        jtStudy.setCreateTime(rs.getDate("create_time"));
        return jtStudy;
    }, 5);

    System.out.println(jtStudies);
}
```

  结果如下：

```java
[JtStudy{
   id=4, name='微博', age=5, createTime=2020-11-29}, JtStudy{
   id=5, name='Facebook', age=5, createTime=2020-04-21}]
```

  可以看到，我们只需要一个方法query方法，第一个参数是query语句，然后传递一个rowMapper映射实现将查询结果封装为JtStudy对象（我这里传递的是lambda对象），第三个参数传递sql的参数，即可获取结果。   在DAO开发中，我们可以为每个实例单独实现一个RowMapper并单独提取出来，这样可以被其他sql复用，代码更加优化。

```java
private final RowMapper<JtStudy> jtStudyRowMapper = (rs, rowNum) -> {
   
    JtStudy jtStudy = new JtStudy();
    jtStudy.setId(rs.getInt("id"));
    jtStudy.setName(rs.getString("name"));
    jtStudy.setAge(rs.getInt("age"));
    jtStudy.setCreateTime(rs.getDate("create_time"));
    return jtStudy;
};


public List<JtStudy> findAllJdbcTests() {
   
    return jdbcTemplate.query("select * from jt_study ", jtStudyRowMapper);
}

public List<JtStudy> findJdbcTestsByAge(int age) {
   
    return jdbcTemplate.query("select * from jt_study where age = ?", jtStudyRowMapper, age);
}


@Test
public void daoQuery() {
   
    //service调用
    List<JtStudy> jtStudies = findJdbcTestsByAge(5);
    System.out.println(jtStudies);
    //service调用
    List<JtStudy> allJtStudies = findAllJdbcTests();
}
```

##### 3.3.1.1 使用预定义的RowMapper

  **相比于传统JDBC编程，我们看到上面的代码确实更加方便了，但是第二个rowMapper参数仍然需要很多的代码去将查询结果映射为Java实例对象，比较麻烦。实际上spring JDBC提供了一个默认映射规则的rowMapper实现BeanPropertyRowMapper，我们只需要传递一个BeanPropertyRowMapper实现，并且执行映射对象的class即可实现自动映射：**

```java
@Test
public void beanPropertyRowMapper() {
   
    String sql = "select * from jt_study where age = ?";
    List<JtStudy> jtStudies = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(JtStudy.class), 5);
    //也可以使用BeanPropertyRowMapper.newInstance静态方法免去我们手动new对象
    //List<JtStudy> jtStudies = jdbcTemplate.query(sql, BeanPropertyRowMapper.newInstance(JtStudy.class), 5);
    System.out.println(jtStudies);
}
```

  是不是发现代码突然清爽了好多^_^，没错**BeanPropertyRowMapper**使用了默认的映射规则，将查询结果与实例对应的属性一一映射，它的映射规则如下：

1. 数据库字段名和实体的属性名相等，或者仅仅存在大小写的区别，那么可以映射； 
2. 如果数据库字段名和实体的属性名不相等，数据库字段名有下滑线_，实体的属性名下划线_，那么可以映射，这要求数据库字段名在对应下划线位置后的第一个字符为大写，其他字符必须为小写（开头第一个字符除外），这就是**驼峰映射**！如果数据库字段名开头和结尾有下滑线_，这要求实体的属性名的开头或者结尾同样必须有下划线！ 
3. 如果一个数据库字段映射了多个实体的属性，那么选择最匹配的那一个填充。

  所以，如果查询结果不符合映射规则，那么仍然需要实现RowMapper手动指定映射规则，或者在sql语句中指定as别名！Spring JDBC还提供了其他的rowMapper实现，比如：

1. **ColumnMapRowMapper**：它将每一行数据转换为一个Map，key为字段名，value为字段值。 
2. **SingleColumnRowMapper**：对每一行数据提取值并返回，仅会对单个列的结果进行处理。

### 3.3.2 queryForObject查询单行结果

  要求sql语句只会返回一行结果，比如一条数据、统计结果等，但可能包含多个列。

#### 3.3.2.1 查询一行多列

  查询jt_study表中age为11的数据，查询结果为一行多列！可以使用**queryForObject(String sql, RowMapper< T > rowMapper, @Nullable Object… args)** 方法：

```java
@Test
public void queryForObject() {
   
    String sql = "select * from jt_study where age = ?";
    JtStudy jtStudy = jdbcTemplate.queryForObject(sql, BeanPropertyRowMapper.newInstance(JtStudy.class), 11);
    System.out.println(jtStudy);
}
```

  如果没有查找到数据或者不是单行数据，那么将抛出异常。

#### 3.3.2.2 查询一行一列

  查询jt_study表中的数据行数，很明显查询结果为一行一列！可以使用**queryForObject(String sql, RowMapper< T > rowMapper, @Nullable Object… args)** 方法，更简单！

```java
@Test
public void queryCount() {
   
    String sql = "select count(*) from jt_study ";
    Long count=jdbcTemplate.queryForObject(sql, Long.class, 11);
    System.out.println(count);
}
```

  如果没有查找到数据或者不是一行一列的数据，那么将抛出异常。

### 3.3.3 特性化查询

  其他特性化查询操作，比如：

1. **queryForList**：每一行的结果映射为一个Map，key为查询返回的字段名，value为字段值，随后将所有map置于list中返回。没有查询到结果将会报错。 
2. **queryForMap**：只针对单行查询结果，将单行的结果映射为一个Map，key为查询返回的字段名，value为字段值，随后返回。没有查询到结果或者不符合要求将会报错。

```java
@Test
public void queryOther() {
   
    List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from jt_study  where age = 5 ");
    System.out.println(maps);
    Map<String, Object> stringObjectMap = jdbcTemplate.queryForMap("select * from jt_study where age = 11");
    System.out.println(stringObjectMap);
}
```

## 3.4 update操作

  **JdbcTemplate 提供的一系列update方法用来插入、修改、删除数据！**

### 3.4.1 插入操作

  插入一行数据！采用**update(String sql, @Nullable Object… args)** 通用方法即可！

```java
@Test
public void insert() {
   
    //插入的sql
    String sql = "insert into jt_study (id,name,age) values (?,?,?)";
    //调用update，返回受影响的行数
    int insert = jdbcTemplate.update(sql, null, "insert1", 14);
    System.out.println(insert);

    //查询该插入数据
    String querySql = "select * from jt_study where age = ?";
    JtStudy jtStudy = jdbcTemplate.queryForObject(querySql, BeanPropertyRowMapper.newInstance(JtStudy.class), 14);
    System.out.println(jtStudy);
}
```

#### 3.4.1.1 自增主键返回

  **插入后自增主键返回！这里需要采用另一个update方法，传递一个PreparedStatementCreator和keyHolder，PreparedStatementCreator用于创建PreparedStatement，此时需要设置返回主键，而keyHolder而用于在插入完毕之后获取返回的主键！**   **可以看到，jdbcTemplate返回自增主键还是比较麻烦的，如果不是Java 8的lambda表达式，那么代码更加复杂！**

```java
@Test
public void insertReturn() {
   
    //插入的sql
    String sql = "insert into jt_study (name,age) values (?,?)";
    //主键持有者
    KeyHolder keyHolder = new GeneratedKeyHolder();
    //调用另一个update方法，传递PreparedStatementCreator和KeyHolder
    jdbcTemplate.update(con -> {
   
        //设置返回自增主键
        PreparedStatement ps = con.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
        //或者如下样式
        //PreparedStatement ps = con.prepareStatement(sql, new String[] { "id" });
        //传统JDBC操作
        ps.setString(1, "insertReturn");
        ps.setInt(2, 15);
        return ps;
    }, keyHolder);
    //现在可以从keyHolder中获取主键
    long id = Objects.requireNonNull(keyHolder.getKey()).longValue();
    System.out.println("id: " + id);
}
```

### 3.4.2 修改操作

  修改操作是同样的逻辑！

```java
@Test
public void update() {
   
    //修改的sql
    String updateSql = "update jt_study set name = ? where age = ?";
    //调用update，返回受影响的行数
    jdbcTemplate.update(updateSql, "update", 14);

    //查询该修改的数据
    String querySql = "select * from jt_study where age = ?";
    JtStudy jtStudy = jdbcTemplate.queryForObject(querySql, BeanPropertyRowMapper.newInstance(JtStudy.class), 14);
    System.out.println(jtStudy);
}
```

### 3.4.3 删除操作

  删除操作是同样的逻辑！

```java
@Test
public void delete() {
   
    //修改的sql
    String deleteSql = "delete from jt_study  where age = ?";
    //调用update，返回受影响的行数
    jdbcTemplate.update(deleteSql, 14);

    //查询剩下的数据
    String querySql = "select * from jt_study ";
    List<JtStudy> query = jdbcTemplate.query(querySql, BeanPropertyRowMapper.newInstance(JtStudy.class));
    System.out.println(query);
}
```

## 3.5 batchUpdate批处理操作

  JdbcTemplate 提供的一系列batchUpdate方法用来批量插入、修改、删除数据！批处理可以通过一个连接执行多个sql语句，执行效率高，资源利用率好！

### 3.5.1 基本批处理

**int[ ] batchUpdate(String sql, final BatchPreparedStatementSetter pss)**   **sql**：固定格式的sql语句。   **BatchPreparedStatementSetter**：用于设置批处理的次数以及每一次批处理的参数。   **int[ ]**：返回一个int数组，里面的值表示对应位置的sql语句影响的数据条数。如果计数不可用，JDBC 驱动程序将返回值 -2。

  如下案例，批量插入一批数据，更新和删除的语法一样：

```java
@Test
public void basicBatch() {
   
    //参数数组集合，这表示将会批量插入三条数据，在web项目中，这个集合可以通过service传递给dao
    List<JtStudy> batchArgs = new ArrayList<>();
    batchArgs.add(new JtStudy("basicBatch1", 0));
    batchArgs.add(new JtStudy("basicBatch2", 0));
    batchArgs.add(new JtStudy("basicBatch3", 0));

    //service调用dao的方法，返回每条sql影响的数据条数数组
    int[] ints = basicBatchDao(batchArgs);

    //查询全部的数据
    String querySql = "select * from jt_study";
    List<JtStudy> query = jdbcTemplate.query(querySql, BeanPropertyRowMapper.newInstance(JtStudy.class));
    System.out.println(query);
}


private int[] basicBatchDao(List<JtStudy> batchArgs) {
   
    //插入的sql
    String insertSql = "insert into jt_study (name,age) values (?,?)";

    //调用batchUpdate
    return jdbcTemplate.batchUpdate(insertSql, new BatchPreparedStatementSetter() {
   
        /**
         * 设置sql参数值
         * @param ps 当前sql预处理对象
         * @param i 当前处理sql的位置
         */
        @Override
        public void setValues(PreparedStatement ps, int i) throws SQLException {
   
            ps.setString(1, batchArgs.get(i).getName());
            ps.setInt(2, batchArgs.get(i).getAge());
        }

        /**
         * @return 返回批处理的大小（执行次数）
         */
        @Override
        public int getBatchSize() {
   
            return batchArgs.size();
        }
    });
}
```

### 3.5.2 便捷批处理

  上面的操作需要自己设置参数，如果参数位置一一对应，那么可以调用简化版本的方法！**int[] batchUpdate(String sql, List<Object[ ]> batchArgs)**   **sql**：固定格式的sql语句。   **batchArgs**：一个Object[]数组的list集合，每一个Object[]数组表示一条sql语句的参数，list集合中有多少个Object[]数组就表示本批次会执行多少个sql语句。   **int[ ]**：返回一个int数组，里面的值表示对应位置的sql语句影响的数据条数。如果计数不可用，JDBC 驱动程序将返回值 -2。

```java
@Test
public void batchInsert() {
   
    //插入的sql
    String insertSql = "insert into jt_study (name,age) values (?,?)";
    //参数数组集合，这表示将会批量插入三条数据
    List<Object[]> batchArgs = new ArrayList<>();
    batchArgs.add(new Object[]{
   "batchInsert1", -1});
    batchArgs.add(new Object[]{
   "batchInsert2", -2});
    batchArgs.add(new Object[]{
   "batchInsert3", -3});

    //调用batchUpdate
    jdbcTemplate.batchUpdate(insertSql, batchArgs);
    //查询全部的数据
    String querySql = "select * from jt_study";
    List<JtStudy> query = jdbcTemplate.query(querySql, BeanPropertyRowMapper.newInstance(JtStudy.class));
    System.out.println(query);
}
```

### 3.5.3 批处理分批

  如果批处理的数据量过大，那么可能导致长期占用连接，吞吐量降低，发生意想不到的问题，因此我们可以将批处理数据再次分批处理。

**int[ ][ ] batchUpdate(String sql, final Collection< T > batchArgs, final int batchSize, final ParameterizedPreparedStatementSetter< T > pss)**

  **sql**：固定格式的sql语句。   **batchArgs**：一个list集合，每一个集合元素包含一条sql语句的参数，list集合中有多少个元素就表示会执行多少个sql语句。   **batchSize**：每一批sql语句的最大执行数量   **ParameterizedPreparedStatementSetter**：用于为每一个sql语句设置参数。   **int[ ][ ]**：返回一个int的二维数组，顶级二维数组的长度表示运行的批处理数，二级数组的长度指示该批处理中的更新数，每个批处理中的更新数应该是所有批处理提供的批处理大小（除了最后一个可能较少的批处理），具体取决于提供的更新对象总数。二级数组的每个元素值表示对应每个更新语句的更新计数，返回的是 JDBC 驱动程序报告的更新计数，如果计数不可用，JDBC 驱动程序将返回值 -2。

```java
@Test
public void batchBatch() {
   
    //参数数组集合，这表示将会批量插入五条数据，在web项目中，这个集合可以通过service传递给dao
    List<JtStudy> batchArgs = new ArrayList<>();
    batchArgs.add(new JtStudy("batchBatch1", -4));
    batchArgs.add(new JtStudy("batchBatch2", -4));
    batchArgs.add(new JtStudy("batchBatch3", -4));
    batchArgs.add(new JtStudy("batchBatch4", -4));
    batchArgs.add(new JtStudy("batchBatch5", -4));

    //service调用dao的方法
    //返回一个int[][]二维数组，第一个数值表示第几批，第二个数值表示某一批处理中的某条sql影响的数据条数
    int[][] ints = batchBatchDao(batchArgs);
    System.out.println(Arrays.deepToString(ints));
}

private int[][] batchBatchDao(List<JtStudy> batchArgs) {
   
    //插入的sql
    String insertSql = "insert into jt_study (name,age) values (?,?)";

    //调用batchUpdate，设置每一批最多处理2个sql，因此五条数据将分为3批
    //返回一个int[][]二维数组，第一个数值表示第几批，第二个数值表示某一批处理中的某条sql影响的数据条数
    return jdbcTemplate.batchUpdate(insertSql,
            batchArgs,
            2,
            (ps, argument) -> {
   
                ps.setString(1, argument.getName());
                ps.setInt(2, argument.getAge());
            });
}
```

## 3.6 其他 JdbcTemplate 操作

  **我们可以使用 execute ()方法运行任何任意 SQL。因此，该方法通常用于 DDL 语句。**   下面的示例创建一个表：

```java
@Test
public void execute() {
   
    jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
}
```

  你甚至可以使用update调用一个存储过程，但是没有返回值，只能自己查找！   创建一个存过：

```java
CREATE PROCEDURE demo_in_parameter ( IN ageNum INT, OUT flag INT ) BEGIN
	DECLARE
		num INT;
	
	SET flag = 1;
	SELECT
		count( * ) INTO num 
	FROM
		jt_study 
	WHERE
		age = ageNum;
	IF
		num != 1 THEN
			
			SET flag = 0;
		
	END IF;
END;
```

  调用：

```java
@Test
public void procedure() {
   
    jdbcTemplate.update("call test.demo_in_parameter(?,@flag)", 12);
    System.out.println(jdbcTemplate.queryForObject("select @flag", Integer.class));

    jdbcTemplate.update("call test.demo_in_parameter(?,@flag)", 5);
    System.out.println(jdbcTemplate.queryForObject("select @flag", Integer.class));
}
```


  **NamedParameterJdbcTemplate 类使用命名参数增加了对 JDBC 语句编程的支持，而不是仅使用经典占位符 "？"参数对 JDBC 语句进行编程。这样的好处就是不需要参数顺序一致，只需要保证命名参数唯一即可！**   NamedParameterJdbcTemplate类包装了一个JdbcTemplate，并委托到包装的 JdbcTemplate上完成大部分工作，可以使用getJdbcOperations()方法获取内部包装的 JdbcTemplate。   这里仅介绍NamedParameterJdbcTemplate类中与 JdbcTemplate 本身不同的地方，即使用命名参数对 JDBC 语句进行编程。

## 4.1 配置NamedParameterJdbcTemplate

  **NamedParameterJdbcTemplate是线程安全的模板类，因此我们可以直接在JdbcStart配置类中加入一个NamedParameterJdbcTemplate的@Bean方法，并且传递以前配置的jdbcTemplate对象！**

```java
/**
 * 配置NamedParameterJdbcTemplate
 */
@Bean
public NamedParameterJdbcTemplate namedParameterJdbcTemplate() {
   
    return new NamedParameterJdbcTemplate(jdbcTemplate());
}
```

  随后在测试类JdbcTest中引入NamedParameterJdbcTemplate即可使用：

```java
@Resource
private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
```

## 4.2 基本使用

  **命名参数的使用方法很简单，在分配给 sql 变量的值中使用命名参数表示法，即使用“：xxx”作为占位符，来表示引用名为xxx的变量，替代“?”占位符，并且在命名参数变量（MapSqlParameterSource）中的设置的相应值，随后传递给query方法即可！**   比如查询jt_study表中age为5的数据！

```java
@Test
public void namedParameterQuery() {
   
    String sql = "select * from jt_study where age = :age";
    //建立一个SqlParameterSource，只有一个参数
    SqlParameterSource namedParameters = new MapSqlParameterSource("age", 5);
    //调用查询
    List<JtStudy> query = namedParameterJdbcTemplate.query(sql, namedParameters, BeanPropertyRowMapper.newInstance(JtStudy.class));
    System.out.println(query);

    //如果有多个参数，那么可以传递一个map
    //HashMap<String, Object> objectObjectHashMap = new HashMap<>();
    //objectObjectHashMap.put("age",5);
    //MapSqlParameterSource mapSqlParameterSource = new MapSqlParameterSource(objectObjectHashMap);
}
```

  **如果觉得使用MapSqlParameterSource多此一举，那么支持更简单的Map参数，NamedParameterJdbcTemplate会自动创建一个MapSqlParameterSource。**

```java
@Test
public void namedParameterMapQuery() {
   
    String sql = "select * from jt_study where age = :age";
    HashMap<String, Object> stringObjectHashMap = new HashMap<>();
    stringObjectHashMap.put("age",5);
    //调用查询，直接传递一个map
    List<JtStudy> query = namedParameterJdbcTemplate.query(sql, stringObjectHashMap, BeanPropertyRowMapper.newInstance(JtStudy.class));
    System.out.println(query);
}
```

  **SqlParameterSource**表示sql命名参数值的来源抽象，上面我们就见过了它的一个实现MapSqlParameterSource，MapSqlParameterSource简单的包装了一个Map作为参数来源，另外提供的一个实现就是BeanPropertySqlParameterSource，此类包装任意 JavaBean（即符合 JavaBean 约定的类的实例：https://www.oracle.com/java/technologies/javase/javabeans-spec.html），并使用包装的 JavaBean 的属性作为命名参数值的来源。   有了**BeanPropertySqlParameterSource**，我们就不再封装map，而是直接传递JavaBean实例即可，NamedParameterJdbcTemplate会自动查找与sql命名参数变量一致的JavaBean的属性名，将属性值作为该sql参数值。   web开发中常见的一种BeanPropertySqlParameterSource使用形式如下：

```java
@Test
public void beanPropertySqlParameterSource() {
   
    //service中的对象
    JtStudy jtStudy = new JtStudy();
    jtStudy.setAge(5);
    //调用dao的方法，传递一个对象即可
    List<JtStudy> jtStudies = beanPropertySqlParameterSourceDao(jtStudy);
    System.out.println(jtStudies);
}

private List<JtStudy> beanPropertySqlParameterSourceDao(JtStudy jtStudy) {
   
    String sql = "select * from jt_study where age = :age";
    BeanPropertySqlParameterSource bpsps = new BeanPropertySqlParameterSource(jtStudy);
    return namedParameterJdbcTemplate.query(sql, bpsps, BeanPropertyRowMapper.newInstance(JtStudy.class));
}
```

## 4.3 便捷批处理

  **NamedParameterJdbcTemplate的批处理和JdbcTemplate差不多，但是没有了批处理分批的功能。另外，参数不是传递List<Object[]>，而是传递一个Map<String, ?>[]数组，即数组元素为Map，Map里面包含了当前执行的sql的命名参数及其对象的值的映射，数组长度就是批处理数量。**   **这里的传递的Map数组将会被SqlParameterSourceUtils通过createBatch转换为SqlParameterSource[ ]数组！**

```java
@Test
public void namedParameterBatch() {
   
    String sql = "insert into jt_study (name,age) values (:name , :age )";

    HashMap<String, Object> stringObjectHashMap1 = new HashMap<>();
    stringObjectHashMap1.put("age", 20);
    stringObjectHashMap1.put("name", "namedParameterBatch1");
    HashMap<String, Object> stringObjectHashMap2 = new HashMap<>();
    stringObjectHashMap2.put("name", "namedParameterBatch2");
    stringObjectHashMap2.put("age", 20);
    //参数数组
    Map<String, Object>[] maps = new Map[]{
   stringObjectHashMap1, stringObjectHashMap2};

    int[] ints = namedParameterJdbcTemplate.batchUpdate(sql, maps);
    System.out.println(Arrays.toString(ints));
}
```

  很明显上面的方式太麻烦了，对此我们可以直接使用**SqlParameterSourceUtils.createBatch**传递一个对象集合，来快速创建**SqlParameterSource[ ]** 数组：

```java
@Test
public void sqlParameterSourceUtilsBatch() {
   
    String sql = "insert into jt_study (name,age) values ( :name , :age )";

    List<JtStudy> batchArgs = new ArrayList<>();
    batchArgs.add(new JtStudy("SqlParameterSource1", 21));
    batchArgs.add(new JtStudy("SqlParameterSource2", 21));
    batchArgs.add(new JtStudy("SqlParameterSource3", 21));

    int[] ints = namedParameterJdbcTemplate.batchUpdate(sql, SqlParameterSourceUtils.createBatch(batchArgs));
    System.out.println(Arrays.toString(ints));
}
```


  Spring JDBC提供了一系列的SimpleJdbc类，用于进一步简化 JDBC 操作。   SimpleJdbcInsert 和 SimpleJdbcCall 类利用可以通过 JDBC 驱动程序检索的数据库元数据来提供简化的配置，这意味着我们可以更少地进行前期配置。

## 5.1 SimpleJdbcInsert简化插入


  **SimpleJdbcInsert用于简化插入操作！它的操作都是execute方法完成的，都是insert操作！** ![img](https://img-blog.csdnimg.cn/20201211165657859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70#pic_center)

### 5.1.1 初始化SimpleJdbcInsert

  SimpleJdbcInsert对象需要使用dataSource或者jdbcTemplate初始化，因此我们需要引入dataSource或者jdbcTemplate实例。   一个SimpleJdbcInsert对象还需要和一个数据库表绑定，通过withTableName方法指定一个数据库表的名字，随后就可以操作这个数据库表。因此，我们通常需要为一个DAO创建一个SimpleJdbcInsert对象。   在web项目的DAO中常见初始化策略为：

```java
@Resource
private JdbcTemplate jdbcTemplate;

private SimpleJdbcInsert simpleJdbcInsert;

@PostConstruct
public void postConstruct() {
   
    //初始化SimpleJdbcInsert，绑定一个数据库表
    simpleJdbcInsert = new SimpleJdbcInsert(jdbcTemplate).withTableName("jt_study");
}
```

  随后我们就可以使用simpleJdbcInsert了！

### 5.1.2 基本插入数据

  基本的插入数据方法采用execute方法就可以了，我们只需要传递一个Map，key为对性的表的字段名，value就是插入的该字段的值，不需要其他sql语句，即可直接插入数据，是不是很神奇！   如下案例，插入一条数据到jt_study表中！

```java
@Test
public void simpleJdbcInsert() {
   
    //插入一条数据到jt_study中
    HashMap<String, Object> paramHashMap = new HashMap<>();
    paramHashMap.put("name", "simpleJdbcInsert");
    paramHashMap.put("age", 100);
    simpleJdbcInsert.execute(paramHashMap);

    System.out.println(jdbcTemplate.queryForObject("select count(*) from jt_study where age = ?", Integer.class,
            100));
}
```

### 5.1.3 默认值与主键返回

  在上面插入数据之后，我们会发现数据库的数据虽然id自增了，但是create_time却是null，我们设置的是create_time为CURRENT_TIMESTAMP，即插入时自动填充当前时间，但是并没有填充。   对于这种取默认值的字段，我们可以在初始化simpleJdbcInsert的时候，通过调用usingGeneratedKeyColumns设置自动生成的字段名数组，此后对于自动生成的字段，如果不在map中传递该字段，那么就会采用自动生成的值！   **我们添加如下设置，添加两个字段，id和create_time：**

```java
@PostConstruct
public void postConstruct() {
   
    //初始化SimpleJdbcInsert，绑定一个数据库表
    simpleJdbcInsert = new SimpleJdbcInsert(jdbcTemplate).withTableName("jt_study");
    simpleJdbcInsert.usingGeneratedKeyColumns("id","create_time");
}
```

  **再次尝试插入，该设置还可以配合executeAndReturnKey方法返回生成的主键，该方法将会返回一个Number类型的队形爱过，可以从里面获取返回的主键：**

```java
@Test
public void simpleJdbcInsertReturn() {
   
    HashMap<String, Object> paramHashMap = new HashMap<>();
    paramHashMap.put("name", "simpleJdbcInsertReturn");
    paramHashMap.put("age", 102);

    //插入一条数据到jt_study中，返回自增的主键
    Number number = simpleJdbcInsert.executeAndReturnKey(paramHashMap);
    System.out.println("id: " + number.longValue());

    System.out.println(jdbcTemplate.queryForObject("select count(*) from jt_study where age = ?", Integer.class,
            102));
}
```

  **如果有多个自增的主键字段或者不是Number类型的主键，那么可以使用executeAndReturnKeyHolder方法，该方法返回一个KeyHolder，内部持有返回的key集合，使用keyHolder.getKeyList()即可获取！**

```java
public void executeAndReturnKeyHolder() {
   
    HashMap<String, Object> paramHashMap = new HashMap<>();
    paramHashMap.put("name", "executeAndReturnKeyHolder");
    paramHashMap.put("age", 104);

    //插入一条数据到jt_study中
    KeyHolder keyHolder = simpleJdbcInsert.executeAndReturnKeyHolder(paramHashMap);

    System.out.println(keyHolder.getKeyList());
}
```

  使用 UsingColumns 方法指定列名称列表来限制插入的列，此时，其他的列将同样会使用默认值。我们加入如下配置，并且没有配置create_time：

```java
@PostConstruct
public void postConstruct() {
   
    //初始化SimpleJdbcInsert，绑定一个数据库表
    simpleJdbcInsert = new SimpleJdbcInsert(jdbcTemplate).withTableName("jt_study");
    simpleJdbcInsert.usingColumns("name","age");
}
```

  尝试插入数据，我们会发现create_time使用了默认值：

```java
@Test
public void usingColumns() {
   
    HashMap<String, Object> paramHashMap = new HashMap<>();
    paramHashMap.put("name", "usingColumns");
    paramHashMap.put("age", 105);

    //插入一条数据到jt_study中
    simpleJdbcInsert.execute(paramHashMap);
}
```

### 5.1.4 SqlParameterSource参数源

  **前面我们都是使用Map提供参数值，这是没问题的可以正常工作，但却不是最方便的方法。类似于NamedParameterJdbcTemplate，这里的SimpleJdbcInsert同样可以使用SqlParameterSource来代理Map提供参数。**   我们可以使用SqlParameterSource的默认实现之一**BeanPropertySqlParameterSource**来作为参数源，该实现包装一个JavaBean对象来提供参数值，如下案例：

```java
@Test
public void sqlParameterSource() {
   
    //参数对象
    JtStudy jtStudy = new JtStudy("sqlParameterSource", 106);
    //参数源
    BeanPropertySqlParameterSource bpsps = new BeanPropertySqlParameterSource(jtStudy);

    //插入一条数据到jt_study中，传递一个参数源
    simpleJdbcInsert.execute(bpsps);
}
```

  **另一个选择是类似于 Map 的 MapSqlParameterSource参数源实现，它提供了一种可以链式编程的更方便的 addValue 方法。**

```java
@Test
public void mapSqlParameterSource() {
   
    //参数源
    MapSqlParameterSource msps = new MapSqlParameterSource();
    //链式的添加参数
    msps.addValue("name", "mapSqlParameterSource").addValue( "age", 107);

    //插入一条数据到jt_study中，传递一个参数源
    simpleJdbcInsert.execute(msps);
}
```

## 5.2 SimpleJdbcCall简化存储过程/函数

  **我们可以以类似于声明 SimpleJdbcInsert 的方式声明 SimpleJdbcCall。SimpleJdbcCall 类提供了对存储过程/函数的简化调用方式，它使用数据库中的元数据查找输入（in）和退出（out）参数的名称，因此不必显式声明它们。**

### 5.2.1 初始化SimpleJdbcCall

  SimpleJdbcCall对象需要使用dataSource或者jdbcTemplate初始化，因此我们需要引入dataSource或者jdbcTemplate实例。   一个SimpleJdbcCall对象还需要和一个存储过程/函数绑定，可以通过withProcedureName方法指定一个存储过程或者通过withFunctionName指定一个函数的名字，注意一个SimpleJdbcCall对象只能绑定其中一个。   在web项目的DAO中常见初始化策略为：

```java
private SimpleJdbcCall simpleJdbcCall;

@PostConstruct
public void simpleJdbcCall() {
   
    //初始化SimpleJdbcInsert，通过withProcedureName绑定一个存储过程
    //或者通过withFunctionName绑定一个函数名
    simpleJdbcCall = new SimpleJdbcCall(jdbcTemplate)
            .withProcedureName("myProcedure");
}
```

### 5.2.2 执行存储过程

  **创建一个存储过程（mysql 8），该存储过程，输入一个ageNum的age值，返回一个int类型的falg变量。判断给定的ageNum在数据库jt_study表中是否有且只有一条数据，如果是那么返回1，否则返回0。**

```java
CREATE PROCEDURE `myProcedure`( IN ageNum INT, OUT flag INT )
BEGIN
	DECLARE
		num INT;
	
	SET flag = 1;
	SELECT
		count( * ) INTO num 
	FROM
		jt_study 
	WHERE
		age = ageNum;
	IF
		num != 1 THEN
			
			SET flag = 0;
		
	END IF;

END
```

  **参数名就是存储过程的的IN参数名，使用execut方法即可执行，返回一个map，里面就是执行的返回结果，key就是OUT参数名：**

```java
@Test
public void executeProcedure() {
   
    //IN 参数源
    MapSqlParameterSource msps = new MapSqlParameterSource("ageNum", 0);

    //执行存储过程，获取返回值
    Map<String, Object> result = simpleJdbcCall.execute(msps);

    System.out.println(result);
}
```

### 5.2.3. 执行函数

  **另外初始化一个simpleJdbcCall，通过withFunctionName绑定一个函数名：**

```java
private SimpleJdbcCall simpleJdbcCall2;

@PostConstruct
public void simpleJdbcCall2() {
   
    //初始化simpleJdbcCall2，通过withFunctionName绑定一个函数名
    simpleJdbcCall2 = new SimpleJdbcCall(jdbcTemplate).withFunctionName("myFun");
}
```

  创建一个函数（mysql 8），计算两个输入参数ia+ib的和并返回：

```java
CREATE FUNCTION `myFun`( ia INT, ib INT ) RETURNS int
    DETERMINISTIC
BEGIN
	RETURN ia + ib;

END
```

  **参数名就是函数的参数名，使用execut方法即可执行函数，返回一个map，里面的key固定为“return”，value就是执行的返回结果。更简单的，可以使用executeFunction方法，直接返回预期类型函数结果：**

```java
@Test
public void executeFun() {
   
    //IN 参数源
    MapSqlParameterSource msps = new MapSqlParameterSource();
    msps.addValue("ia", 1).addValue("ib", 4);
    //通过execute执行存储过程，获取返回值
    Map<String, Object> result = simpleJdbcCall2.execute(msps);
    System.out.println(result);

    //更简单的，通过executeFunction方法还行函数，返回对应类型的返回值
    Integer integer = simpleJdbcCall2.executeFunction(Integer.class, msps);
    System.out.println(integer);
}
```

### 5.2.4 封装结果集

  对于返回结果集的存储过程或函数，我们可以使用返回ResultSet方法，指定返回的结果集的名称，并声明要用于特定参数的 RowMapper 实现（指定对结果集的封装操作）。   如下案例，定义一个存储过程（mysql 8），该存储过程首先对于接收到的ageNum参数加1的值赋给var，随后查找全部age等于var的jt_study表数据。   这种存储过程并没有主动设置返回值，而是将随后的第一查询结果作为返回值：

```java
CREATE PROCEDURE `select_jt_study`(IN ageNum int)
BEGIN
 DECLARE  var int;
 set var = ageNum +1;
 SELECT * FROM jt_study where age = var;
END
```

  **创建一个SimpleJdbcCall绑定该存储过程，指定返回的结果集名为"resultSet"，由于存储过程中的查询语句返回的是多行队列的结果，因此结果集的处理方式为BeanPropertyRowMapper（将每一行结果映射到JtStudy对象中）。**

```java
private SimpleJdbcCall simpleJdbcCall3;

@PostConstruct
public void simpleJdbcCall3() {
   
    //初始化simpleJdbcCall3，通过withProcedureName绑定一个函数名
    simpleJdbcCall3 = new SimpleJdbcCall(jdbcTemplate)
            .withProcedureName("select_jt_study")
            //指定返回的结果集名为"resultSet"，处理方式为将每一条结果映射到JtStudy对象中
            .returningResultSet("resultSet", BeanPropertyRowMapper.newInstance(JtStudy.class));
}
```

  执行，即可获得封装后的结果值：

```java
@Test
public void executeResultSet() {
   
    //IN 参数源
    MapSqlParameterSource msps = new MapSqlParameterSource("ageNum", 4);

    //执行存储过程，获取返回值
    Map<String, Object> result = simpleJdbcCall3.execute(msps);

    System.out.println(result);
}
```

  结果如下：

```java
{
   resultSet=[JtStudy{
   createTime=2020-11-29 15:29:41.0, id=4, name='微博', age=5}, JtStudy{
   createTime=2020-04-21 23:55:15.0, id=5, name='null', age=5}], #update-count-1=0}
```


   [Spring JDBC官方文档](https://docs.spring.io/spring-framework/docs/5.2.8.RELEASE/spring-framework-reference/data-access.html#jdbc)。   **Spring JDBC是Spring提供了的简单框架，它对传统JDBC访问进行了的简单封装，其核心类就是JdbcTemplate，在上面我们已经简单的学习了它的基本使用和一些高级技巧，我们能够明显的感受到，它相比于传统JDBC编程有了很大的改进，使用起来也很简单，可以轻松入门。**   **JdbcTemplate也能够自动解决sql注入，但这需要我们调用参数化sql的方法（前面的讲的所有方法），如果是我们自己拼接sql，那么可能带来安全隐患！**   **相比于MyBatis、Hibernate这种重量级框架，JdbcTemplate作为轻量级框架，它的功能并不是很完善，相比于Hibernate它仍然需要写sql语句，相比于MyBatis它不支持动态sql，另外可能需要大量的rowmapper对象，对于查询操作也很容易抛出各种异常（比如没查询到数据或者返回的数据格式不满足），对于一些大型项目如果需要使用JdbcTemplate，那么可能需要我们自己封装一些工具类来增强。不过由于这个框架非常的底层，仅仅是对JDBC操作做了简单的封装，因此sql执行效率更高，对于复杂sql的效果更加明显。**   **通常，如果不想引入外部数据框架，那么可以使用Spring Data JPA + JdbcTemplate，简单的操作使用JPA，不需要写sql语句，可以真正的实现面对对象，这正是领域驱动设计(DDD:Domain-Driven Design)的追求，而复杂操作可以使用JdbcTemplate直接写sql语句，如果一个系统的需要大量的复杂sql语句，不同领域模型的组合，或许是数据库设计的问题（比如为了节省某个字段造成多表联查）！或者，也可以直接使用简单的Spring Data JDBC来代替二者。**    **后面有机会，我们在慢慢介绍Spring Data下面的常用数据库框架。**

**相关文章：**    [https://spring.io/](https://spring.io/)    [Spring Framework 5.x 学习](https://blog.csdn.net/weixin_43767015/category_10402193.html)    [Spring Framework 5.x 源码](https://blog.csdn.net/weixin_43767015/category_10402194.html)

><font color="red" size="4">如有需要交流，或者文章有误，请直接留言。另外希望点赞、收藏、关注，我将不间断更新各种Java学习博客！</font>

