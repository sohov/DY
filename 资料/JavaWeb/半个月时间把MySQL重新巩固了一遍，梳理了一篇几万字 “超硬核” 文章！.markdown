
- MySQL 是最流行的关系型数据库管理系统，在 WEB 应用方面 MySQL 是最好的 RDBMS(Relational Database Management System：关系数据库管理系统)应用软件之一


![img](https://img-blog.csdnimg.cn/1d08d8edcc7c432cb36ec517d5d859b0.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)








- **MySQL面试题链接：精品MySQL面试题，备战八月99%必问！过不了面试算我的**


![img](https://img-blog.csdnimg.cn/4bdea1132530427e8b3554a2e2b47294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

**评论区评论要资料三个字即可获得MySQL全套资料 ！**




![img](https://img-blog.csdnimg.cn/img_convert/954039e049910d6579e1d32ba995b7ad.png)

## 什么是数据库

- 数据库（Database）是按照数据结构来组织、存储和管理数据的仓库。 
- 每个数据库都有一个或多个不同的 API 用于创建，访问，管理，搜索和复制所保存的数据。 
- 我们也可以将数据存储在文件中，但是在文件中读写数据速度相对较慢。所以，现在我们使用关系型数据库管理系统（RDBMS）来存储和管理大数据量。所谓的关系型数据库，是建立在关系模型基础上的数据库，借助于集合代数等数学概念和方法来处理数据库中的数据。

## MySQL数据库

>MySQL 是一个关系型数据库管理系统，由瑞典 MySQL AB 公司开发，目前属于 Oracle 公司。MySQL 是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

- MySQL 是开源的，目前隶属于 Oracle 旗下产品。 
- MySQL 支持大型的数据库。可以处理拥有上千万条记录的大型数据库。 
- MySQL 使用标准的 SQL 数据语言形式。 
- MySQL 可以运行于多个系统上，并且支持多种语言。这些编程语言包括 C、C++、Python、Java、Perl、PHP、Eiffel、Ruby 和 Tcl 等。 
- MySQL 对PHP有很好的支持，PHP 是目前最流行的 Web 开发语言。 
- MySQL 支持大型数据库，支持 5000 万条记录的数据仓库，32 位系统表文件最大可支持 4GB，64 位系统支持最大的表文件为8TB。 
- MySQL 是可以定制的，采用了 GPL 协议，你可以修改源码来开发自己的 MySQL 系统。

## RDBMS 术语

>在我们开始学习MySQL 数据库前，让我们先了解下RDBMS的一些术语

- 数据库: 数据库是一些关联表的集合。 
- 数据表: 表是数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。 
- 列: 一列(数据元素) 包含了相同类型的数据, 例如邮政编码的数据。 
- 行：一行（=元组，或记录）是一组相关的数据，例如一条用户订阅的数据。 
- 冗余：存储两倍数据，冗余降低了性能，但提高了数据的安全性。 
- 主键：主键是唯一的。一个数据表中只能包含一个主键。你可以使用主键来查询数据。 
- 外键：外键用于关联两个表。 
- 复合键：复合键（组合键）将多个列作为一个索引键，一般用于复合索引。 
- 索引：使用索引可快速访问数据库表中的特定信息。索引是对数据库表中一列或多列的值进行排序的一种结构。类似于书籍的目录。 
- 参照完整性: 参照的完整性要求关系中不允许引用不存在的实体。与实体完整性是关系模型必须满足的完整性约束条件，目的是保证数据的一致性。

>MySQL 为关系型数据库(Relational Database Management System), 这种所谓的<code>关系型</code>可以理解为<code>表格</code>的概念, 一个关系型数据库由一个或数个表格组成, 如图所示的一个表格

## 数据库表的存储位置

>MySQL数据表以文件方式存放在磁盘中：

1. 包括表文件、数据文件以及数据库的选项文件 
2. 位置：MySQL安装目录\data下存放数据表。目录名对应数据库名，该目录下文件名对应数据表

**注：**

InnoDB类型数据表只有一个*. frm文件，以及上一级目录的ibdata1文件 MylSAM类型数据表对应三个文件：

1. *. frm —— 表结构定义文件 
2. *. MYD —— 数据文件 
3. *. MYI —— 索引文件

存储位置：因操作系统而异，可查my.ini



- MySQL提供的数据类型包括数值类型（整数类型和小数类型）、字符串类型、日期类型、复合类型（复合类型包括enum类型和set类型）以及二进制类型 。

## 一. 整数类型


![img](https://img-blog.csdnimg.cn/4672264710334b75b873400c694824bc.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 整数类型的数，默认情况下既可以表示正整数又可以表示负整数（此时称为有符号数）。如果只希望表示零和正整数，可以使用无符号关键字“unsigned”对整数类型进行修饰。 
- 各个类别存储空间及取值范围。


![img](https://img-blog.csdnimg.cn/c9a0e383d2154fa690fe36e3e0cd5c36.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 二. 小数类型


![img](https://img-blog.csdnimg.cn/b340d38bd1c7430e85d5f449920af39e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

-  decimal(length, precision)用于表示精度确定（小数点后数字的位数确定）的小数类型，length决定了该小数的最大位数，precision用于设置精度（小数点后数字的位数）。  
-  例如： decimal (5,2)表示小数取值范围：999.99～999.99 decimal (5,0)表示： -99999～99999的整数。  
-  各个类别存储空间及取值范围。 


![img](https://img-blog.csdnimg.cn/ffaee357193f4dddba9287f189f2d113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 三. 字符串


![img](https://img-blog.csdnimg.cn/a1f9265fd0f4492e8037962e9cded270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- char()与varchar(): 例如对于简体中文字符集gbk的字符串而言，varchar(255)表示可以存储255个汉字，而每个汉字占用两个字节的存储空间。假如这个字符串没有那么多汉字，例如仅仅包含一个‘中’字，那么varchar(255)仅仅占用1个字符（两个字节）的储存空间；而char(255)则必须占用255个字符长度的存储空间，哪怕里面只存储一个汉字。 
- 各个类别存储空间及取值范围。


![img](https://img-blog.csdnimg.cn/08b9e591ef9a4939bdb28caac0f9e340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 四. 日期类型


- date表示日期，默认格式为‘YYYY-MM-DD’； time表示时间，格式为‘HH:ii:ss’； year表示年份； datetime与timestamp是日期和时间的混合类型，格式为’YYYY-MM-DD HH:ii:ss’。 ![img](https://img-blog.csdnimg.cn/8b1dfde80d2d415db59240682d88a016.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70) 
- datetime与timestamp都是日期和时间的混合类型，区别在于： 表示的取值范围不同，datetime的取值范围远远大于timestamp的取值范围。 将NULL插入timestamp字段后，该字段的值实际上是MySQL服务器当前的日期和时间。 同一个timestamp类型的日期或时间，不同的时区，显示结果不同。 
- 各个类别存储空间及取值范围。


![img](https://img-blog.csdnimg.cn/e4b520c7104349b7a8c7dc26e29d8161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 五. 复合类型

- MySQL 支持两种复合数据类型：enum枚举类型和set集合类型。 enum类型的字段类似于单选按钮的功能，一个enum类型的数据最多可以包含65535个元素。 set 类型的字段类似于复选框的功能，一个set类型的数据最多可以包含64个元素。

## 六. 二进制类型

- 二进制类型的字段主要用于存储由‘0’和‘1’组成的字符串，因此从某种意义上将，二进制类型的数据是一种特殊格式的字符串。 
- 二进制类型与字符串类型的区别在于：字符串类型的数据按字符为单位进行存储，因此存在多种字符集、多种字符序；而二进制类型的数据按字节为单位进行存储，仅存在二进制字符集binary。


![img](https://img-blog.csdnimg.cn/8893e04e9e4a4525913ee8f9d498285b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)



- 约束是一种限制，它通过对表的行或列的数据做出限制，来确保表的数据的完整性、唯一性。下面文章就来给大家介绍一下6种mysql常见的约束，希望对大家有所帮助。

## 一. 非空约束(not null)

-  非空约束用于确保当前列的值不为空值，非空约束只能出现在表对象的列上。  
-  Null类型特征：所有的类型的值都可以是null，包括int、float 等数据类型 


![img](https://img-blog.csdnimg.cn/f3e5d6604b524ce8aef15fb405c6553c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 二. 唯一性约束(unique)

- 唯一约束是指定table的列或列组合不能重复，保证数据的唯一性。 
- 唯一约束不允许出现重复的值，但是可以为多个null。 
- 同一个表可以有多个唯一约束，多个列组合的约束。 
- 在创建唯一约束时，如果不给唯一约束名称，就默认和列名相同。 
- 唯一约束不仅可以在一个表内创建，而且可以同时多表创建组合唯一约束。


![img](https://img-blog.csdnimg.cn/3f0ff2876176474fa04f5a14cce677ed.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 三. 主键约束(primary key) PK

-  主键约束相当于 唯一约束 + 非空约束 的组合，主键约束列不允许重复，也不允许出现空值。  
-  每个表最多只允许一个主键，建立主键约束可以在列级别创建，也可以在表级别创建。  
-  当创建主键的约束时，系统默认会在所在的列和列组合上建立对应的唯一索引。 


![img](https://img-blog.csdnimg.cn/8d0cd7d7c03f427ba314e37f87ffb929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 四. 外键约束(foreign key) FK

-  外键约束是用来加强两个表（主表和从表）的一列或多列数据之间的连接的，可以保证一个或两个表之间的参照完整性，外键是构建于一个表的两个字段或是两个表的两个字段之间的参照关系。  
-  创建外键约束的顺序是先定义主表的主键，然后定义从表的外键。也就是说只有主表的主键才能被从表用来作为外键使用，被约束的从表中的列可以不是主键，主表限制了从表更新和插入的操作。 

## 五. 默认值约束 (Default)

- 若在表中定义了默认值约束，用户在插入新的数据行时，如果该行没有指定数据，那么系统将默认值赋给该列，如果我们不设置默认值，系统默认为NULL。


![img](https://img-blog.csdnimg.cn/d72aea4d63814588ac939259ef07e69f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 六. 自增约束(AUTO_INCREMENT)

-  自增约束(AUTO_INCREMENT)可以约束任何一个字段，该字段不一定是PRIMARY KEY字段，也就是说自增的字段并不等于主键字段。  
-  但是PRIMARY_KEY约束的主键字段，一定是自增字段，即PRIMARY_KEY 要与AUTO_INCREMENT一起作用于同一个字段。 


![img](https://img-blog.csdnimg.cn/ba78411f14a64e98943c6770dd95d9c2.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

当插入第一条记录时，自增字段没有给定一个具体值，可以写成DEFAULT/NULL，那么以后插入字段的时候，该自增字段就是从1开始，没插入一条记录，该自增字段的值增加1。当插入第一条记录时，给自增字段一个具体值，那么以后插入的记录在此自增字段上的值，就在第一条记录该自增字段的值的基础上每次增加1。也可以在插入记录的时候，不指定自增字段，而是指定其余字段进行插入记录的操作。



## 登录数据库相关命令

### 一. 启动服务

语法：

```java
mysql> net start mysql
```

### 二. 关闭服务

语法：

```java
mysql> net stop mysql
```

### 三. 链接MySQL

- 语法：mysql -u用户名 -p密码;

```java
mysql -uroot -p123456;
```


- 在以上命令行中，mysql 代表客户端命令，-u 后面跟连接的数据库用户，-p 表示需要输入密码。如果数据库设置正常，并输入正确的密码，将看到上面一段欢迎界面和一个 mysql&gt;提示符。 ![img](https://img-blog.csdnimg.cn/ed219eb883da4bb8937e7deed1aa694c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 四. 退出数据库

- 语法：quit

```java
mysql> quit
```


- 结果： ![img](https://img-blog.csdnimg.cn/e5e9c07eaefa49e29ee523ee0676c0e3.png)


## DDL（Data Definition Languages）语句：即数据库定义语句

>对于数据库而言实际上每一张表都表示是一个数据库的对象，而数据库对象指的就是DDL定义的所有操作，例如：表，视图，索引，序列，约束等等，都属于对象的操作，所以表的建立就是对象的建立，而对象的操作主要分为以下三类语法

- 创建对象：CREATE 对象名称; 
- 删除对象：DROP 对象名称; 
- 修改对象：ALTER 对象名称;

### 数据库相关操作


![img](https://img-blog.csdnimg.cn/f15b87b081ce4f0d938ca1b76e03f2e7.gif#pic_center)

#### 一. 创建数据库

- 语法：create database 数据库名字;

```java
mysql> create database sqltest;
```


- 结果： ![img](https://img-blog.csdnimg.cn/eafc5281bb574243a3f086a31b6f8e5c.png)

#### 二. 查看已经存在的数据库

- 语法：show databases;

```java
mysql> show databases;
```


- 结果： ![img](https://img-blog.csdnimg.cn/73f05f84b5e24eb381f380dc9784cb83.png)

可以发现，在上面的列表中除了刚刚创建的 mzc-test，sqltest，外，还有另外 4 个数据库，它们都是安装MySQL 时系统自动创建的，其各自功能如下。

1. information_schema：主要存储了系统中的一些数据库对象信息。比如用户表信息、列信息、权限信息、字符集信息、分区信息等。 
2. cluster：存储了系统的集群信息。 
3. mysql：存储了系统的用户权限信息。 
4. test：系统自动创建的测试数据库，任何用户都可以使用。

#### 三. 选择数据库

- 语法：use 数据库名;

```java
mysql> use mzc-test;
```


- 返回Database changed代表我们已经选择 sqltest 数据库，后续所有操作将在 sqltest 数据库上执行。 ![img](https://img-blog.csdnimg.cn/95bcb35d5acb40e98ef4d1d6f6d0c64d.png) 
- 有些人可能会问到，连接以后怎么退出。其实，不用退出来，use 数据库后，使用show databases就能查询所有数据库，如果想跳到其他数据库，用use 其他数据库名字。

#### 四. 查看数据库中的表

- 语法：show tables;

```java
mysql> show tables;
```


- 结果： ![img](https://img-blog.csdnimg.cn/2f3976d4973f4f55b17f462ec15b72df.png)

#### 五. 删除数据库

- 语法：drop database 数据库名称;

```java
mysql> drop database mzc-test;
```


- 结果： ![img](https://img-blog.csdnimg.cn/28cd147f278d465cb89cc5d9021d9ea6.png) 
- 注意：删除时，最好用 `` 符号把表明括起来

#### 六. 设置表的类型

- MySQL的数据表类型：MyISAM、InnoDB、HEAP、 BOB、CSV等


![img](https://img-blog.csdnimg.cn/2181dae5b0a046cab1f8355ef81fa039.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70) 语法：

```java
CREATE TABLE 表名(
	#省略代码
）ENGINE= InnoDB;
```

适用场景：

```java
1. 使用MyISAM：节约空间及响应速度快；不需事务，空间小，以查询访问为主
2. 使用InnoDB：安全性，事务处理及多用户操作数据表；多删除、更新操作，安全性高，事务处理及并发控制
```

##### 1. 查看mysql所支持的引擎类型

语法：

```java
SHOW ENGINES
```

结果：


![img](https://img-blog.csdnimg.cn/c99eb52469624d90939bb5f615173aa8.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

##### 2. 查看默认引擎

语法：

```java
SHOW VARIABLES LIKE 'storage_engine';
```


结果： ![img](https://img-blog.csdnimg.cn/988a3bfc75b1473ca3dc197130b8cff7.png)


### 数据库表相关操作


![img](https://img-blog.csdnimg.cn/60b7fe5091904a14aecdc32b79929aa2.gif#pic_center)

#### 一. 创建表

语法：create table 表名 {列名,数据类型,约束条件};

```java
CREATE TABLE `Student`(
	`s_id` VARCHAR(20),
	`s_name` VARCHAR(20) NOT NULL DEFAULT '',
	`s_birth` VARCHAR(20) NOT NULL DEFAULT '',
	`s_sex` VARCHAR(10) NOT NULL DEFAULT '',
	PRIMARY KEY(`s_id`)
);
```


- 结果 ![img](https://img-blog.csdnimg.cn/58f111836ea84134ac9d67f65c728bda.png)

注意：表名还请遵守数据库的命名规则，这条数据后面要进行删除，所以首字母为大写。

#### 二. 查看表定义

- 语法：desc 表名

```java
mysql> desc Student;
```


- 结果： ![img](https://img-blog.csdnimg.cn/0ff854714afb4e6b9a859e98cef55d66.png) 
- 虽然 desc 命令可以查看表定义，但是其输出的信息还是不够全面，为了查看更全面的表定义信息，有时就需要通过查看创建表的 SQL 语句来得到，可以使用如下命令实现 
- 语法：show create table 表名 \G;

```java
mysql> show create table Student \G;
```


- 结果： ![img](https://img-blog.csdnimg.cn/e2ab9e77231a44baad0a51c20d6e550b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70) 
- 从上面表的创建 SQL 语句中，除了可以看到表定义以外，还可以看到表的engine（存储引擎）和charset（字符集）等信息。\G选项的含义是使得记录能够按照字段竖着排列，对于内容比较长的记录更易于显示。

#### 三. 删除表

- 语法：drop table 表名

```java
mysql> drop table Student;
```

- 结果：


![img](https://img-blog.csdnimg.cn/2409c00136ea481294b62aa6c478389b.png)

#### 四. 修改表 (重要)

- 对于已经创建好的表，尤其是已经有大量数据的表，如果需要对表做一些结构上的改变，我们可以先将表删除（drop），然后再按照新的表定义重建表。这样做没有问题，但是必然要做一些额外的工作，比如数据的重新加载。而且，如果有服务在访问表，也会对服务产生影响。因此，在大多数情况下，表结构的更改一般都使用 alter table语句，以下是一些常用的命令。

##### 1. 修改表类型

- 语法：ALTER TABLE 表名 MODIFY [COLUMN] column_definition [FIRST | AFTER col_name] 
- 例如，修改表 student 的 s_name 字段定义，将 varchar(20)改为 varchar(30)

```java
mysql> alter table Student modify s_name varchar(30);
```


- 结果： ![img](https://img-blog.csdnimg.cn/94968663b88c44eeb62cc1d35e0b2aa7.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

##### 2. 增加表字段

- 语法：ALTER TABLE 表名 ADD [COLUMN] [FIRST | AFTER col_name]; 
- 例如，表 student 上新增加字段 s_test，类型为 int(3)

```java
mysql> alter table student add column s_test int(3);
```


- 结果： ![img](https://img-blog.csdnimg.cn/df997c4a64ea49c5a57348fb51313a19.png)

##### 3. 删除表字段

- 语法：ALTER TABLE 表名 DROP [COLUMN] col_name 
- 例如，将字段 s_test 删除掉

```java
mysql> alter table Student drop column s_test;
```


- 结果： ![img](https://img-blog.csdnimg.cn/74afa84ea422434b9cc4844f504f9b37.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

##### 4. 字段改名

- 语法：ALTER TABLE 表名 CHANGE [COLUMN] old_col_name column_definition [FIRST|AFTER col_name] 
- 例如，将 s_sex 改名为 s_sex1，同时修改字段类型为 int(4)

```java
mysql> alter table Student change s_sex s_sex1 int(4);
```


- 结果： ![img](https://img-blog.csdnimg.cn/76a28a1dc8a845919a49dee8f2eb8e99.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

注意：change 和 modify 都可以修改表的定义，不同的是 change 后面需要写两次列名，不方便。但是 change 的优点是可以修改列名称，modify 则不能。

##### 5. 修改字段排列顺序

-  前面介绍的的字段增加和修改语法（ADD/CNAHGE/MODIFY）中，都有一个可选项first|after column_name，这个选项可以用来修改字段在表中的位置，默认 ADD 增加的新字段是加在表的最后位置，而 CHANGE/MODIFY 默认都不会改变字段的位置。  
-  例如，将新增的字段 s_test 加在 s_id 之后  
-  语法：alter table 表名 add 列名 数据类型 after 列名; 

```java
mysql> alter table Student add s_test date after s_id;
```


- 结果： ![img](https://img-blog.csdnimg.cn/fcc52c86f50c4e0988cab7be12d52d4a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70) 
- 修改已有字段 s_name，将它放在最前面

```java
mysql> alter table Student modify s_name varchar(30) default '' first;
```


- 结果： ![img](https://img-blog.csdnimg.cn/66f566832b95427f9d02c1bd9ccf6a84.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

注意：CHANGE/FIRST|AFTER COLUMN 这些关键字都属于 MySQL 在标准 SQL 上的扩展，在其他数据库上不一定适用。

##### 6.表名修改

- 语法：ALTER TABLE 表名 RENAME [TO] new_tablename 
- 例如，将表 Student 改名为 student

```java
mysql> alter table Student rename student;
```


- 结果： ![img](https://img-blog.csdnimg.cn/2773e6372c3f4d969a4192b4a2c2a563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)


## DML（Data Manipulation Language）语句：即数据操纵语句

- 用于操作数据库对象中所包含的数据

### 一. 添加数据：INSERT

Insert 语句用于向数据库中插入数据

#### 1. 插入单条数据（常用）

语法：insert into 表名(列名1,列名2,...) values(值1,值2,...)

特点：

- 插入值的类型要与列的类型一致或兼容。插入NULL可实现为列插入NULL值。列的顺序可以调换。列数和值的个数必须一致。可省略列名，默认所有列，并且列的顺序和表中列的顺序一致。

案例：

```java
-- 插入学生表测试数据
insert into Student(s_id,s_name,s_birth,s_sex) values('01' , '赵信' , '1990-01-01' , '男');
```


![img](https://img-blog.csdnimg.cn/cc0737e6d4e142f4bfb5dad1e973ac95.png)

#### 2. 插入单条数据

语法：INSERT INTO 表名 SET 列名 = 值，列名 = 值

- 这种方式每次只能插入一行数据，每列的值通过赋值列表制定。

案例：

```java
INSERT INTO student SET s_id='02',s_name='德莱厄斯',s_birth='1990-01-01',s_sex='男'
```


![img](https://img-blog.csdnimg.cn/26c40bdc8d0a480db047204ba07a761c.png)

#### 3. 插入多条数据

语法：insert into 表名 values(值1,值2,值3),(值4,值5,值6),(值7,值8,值9);

案例：

```java
INSERT INTO student VALUES('03','艾希','1990-01-01','女'),('04','德莱文','1990-08-06','男'),('05','俄洛依','1991-12-01','女');
```


![img](https://img-blog.csdnimg.cn/b2114cc9e64b4c25ac61135bddde5742.png)

>上面的例子中，值1,值2,值3),(值4,值5,值6),(值7,值8,值9) 即为 Value List，其中每个括号内部的数据表示一行数据，这个例子中插入了三行数据。Insert 语句也可以只给部分列插入数据，这种情况下，需要在 Value List 之前加上 ColumnName List，

例如：

```java
INSERT INTO student(s_name,s_sex) VALUES('艾希','女'),('德莱文','男');
```

- 每行数据只指定了 s_name 和 s_sex 这两列的值，其他列的值会设为 Null。

#### 4. 表数据复制

语法：INSERT INTO 表名 SELECT * from 表名;

案例：

```java
INSERT INTO student SELECT * from student1;
```


![img](https://img-blog.csdnimg.cn/6ed545aab17e4277a643edc89245b188.png)注意：

- 两个表的字段需要一直，并尽量保证要新增的表中没有数据

### 二. 更新数据：UPDATE

Update 语句一共有两种语法，分别用于更新单表数据和多表数据。


![img](https://img-blog.csdnimg.cn/28b77e7ce41f4b23832613ed70f5ecbe.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 注意：没有 WHERE 条件的 UPDATE 会更新所有值！

#### 1. 修改一条数据的某个字段

语法：UPDATE 表名 SET 字段名 =值 where 字段名=值

案例：

```java
UPDATE student SET s_name ='张三' WHERE s_id ='01'
```


![img](https://img-blog.csdnimg.cn/f3ecb27e165b4b0a8e037657fd0c8973.png)

#### 2. 修改多个字段为同一的值

语法：UPDATE 表名 SET 字段名= 值 WHERE 字段名 in ('值1','值2','值3');

案例：

```java
UPDATE student SET s_name = '李四' WHERE s_id in ('01','02','03');
```


![img](https://img-blog.csdnimg.cn/81fc907de8bb4c17a001a13cfa4b1e17.png)

#### 3. 使用case when实现批量更新

语法：update 表名 set 字段名 = case 字段名 when 值1 then '值' when 值2 then '值' when 值3 then '值' end where s_id in (值1,值2,值3)

案例：

```java
update student set s_name = case s_id when 01 then '小王' when 02 then '小周' when 03 then '老周' end where s_id in (01,02,03)
```


![img](https://img-blog.csdnimg.cn/0aa2f860468d449bb1fb240550bcb83a.png)

- 这句sql的意思是，更新 s_name 字段，如果 s_id 的值为 01 则 s_name 的值为 小王，s_id = 02 则 s_name = 小周，如果s_id =03 则 s_name 的值为 老周。 
- 这里的where部分不影响代码的执行，但是会提高sql执行的效率。确保sql语句仅执行需要修改的行数，这里只有3条数据进行更新，而where子句确保只有3行数据执行。

案例 2：

```java
UPDATE student SET s_birth = CASE s_name
	WHEN '小王' THEN
		'2019-01-20'
	WHEN '小周' THEN
		'2019-01-22'
END WHERE s_name IN ('小王','小周');
```


![img](https://img-blog.csdnimg.cn/a6e7eb3292144e74bbf2f92d7e0d8310.png)

### 三. 删除数据：DELETE

- 数据库一旦删除数据，它就会永远消失。 因此，在执行DELETE语句之前，应该先备份数据库，以防万一要找回删除过的数据。

#### 1. 删除指定数据

语法：DELETE FROM 表名 WHERE 列名=值

- 注意：删除的时候如果不指定where条件，则保留数据表结构，删除全部数据行，有主外键关系的都删不了

案例：

```java
DELETE FROM student WHERE s_id='09'
```


![img](https://img-blog.csdnimg.cn/f53d6a06ef8a41c781b71e8f4e4452ea.png)与 SELECT 语句不同的是，DELETE 语句中不能使用 GROUP BY、 HAVING 和 ORDER BY 三类子句，而只能使用WHERE 子句。原因很简单， GROUP BY 和 HAVING 是从表中选取数据时用来改变抽取数据形式的， 而 ORDER BY 是用来指定取得结果显示顺序的。因此，在删除表中数据 时它们都起不到什么作用。`

#### 2. 删除表中全部数据

语法：TRUNCATE 表名;

-  注意：全部删除，内存无痕迹，如果有自增会重新开始编号。  
-  与 DELETE 不同的是，TRUNCATE 只能删除表中的全部数据，而不能通过 WHERE 子句指定条件来删除部分数据。也正是因为它不能具体地控制删除对象， 所以其处理速度比 DELETE 要快得多。实际上，DELETE 语句在 DML 语句中也 属于处理时间比较长的，因此需要删除全部数据行时，使用 TRUNCATE 可以缩短 执行时间。 

案例：

```java
TRUNCATE student1;
```


![img](https://img-blog.csdnimg.cn/c027235fd6bc404b80744a085b7ecddd.png)


## DQL（Data Query Language）语句：即数据查询语句

- 查询数据库中的记录，关键字 SELECT，**这块内容非常重要！**

### 一. wherer 条件语句

语法：select 列名 from 表名 where 列名 =值

where的作用：

1. 用于检索数据表中符合条件的记录 
2. 搜索条件可由一个或多个逻辑表达式组成，结果一般为真或假

**搜索条件的组成：**

- 算数运算符


![img](https://img-blog.csdnimg.cn/f2f094c59f4a442eb114b662fbb77008.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)


- 逻辑操作符（操作符有两种写法） ![img](https://img-blog.csdnimg.cn/66df9e8856414ddeb28f6b1f198ad3de.png) 
- 比较运算符


![img](https://img-blog.csdnimg.cn/d494a96213ce4a7eaa3ceb75575fda00.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

注意：数值数据类型的记录之间才能进行算术运算，相同数据类型的数据之间才能进行比较。

><strong>表数据</strong><br> <img src="https://img-blog.csdnimg.cn/6d314527cf7b49dea82afa129de25908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

案例 1（AND）：

```java
SELECT  * FROM student WHERE s_name ='小王' AND s_sex='男'
```


![img](https://img-blog.csdnimg.cn/0eafa04ce0b046838413f2b118501334.png) 案例 2（OR）：

```java
SELECT  * FROM student WHERE s_name ='崔丝塔娜' OR s_sex='男'
```


![img](https://img-blog.csdnimg.cn/11a73b5670a24309911643dc1aaca875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

案例 3（NOT）：

```java
SELECT  * FROM student WHERE NOT s_name ='崔丝塔娜'
```


![img](https://img-blog.csdnimg.cn/cd621969ff214d75b4ac45902f5c70c3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70) 案例 4（IS NULL）：

```java
SELECT * FROM student WHERE s_name IS NULL;
```


![img](https://img-blog.csdnimg.cn/23ac7490a01a4d5297333d2bdc21a11b.png) 案例 5（IS NOT NULL）：

```java
SELECT * FROM student WHERE s_name IS NOT NULL;
```


![img](https://img-blog.csdnimg.cn/90695dd631f94537a06142512be14be0.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70) 案例 6（BETWEEN）：

```java
SELECT * FROM student WHERE s_birth BETWEEN '2019-01-20' AND '2019-01-22'
```


![img](https://img-blog.csdnimg.cn/c5f7d9ec460d4b13ba2374273a4beff5.png)

案例 7（LINK）：

```java
SELECT * FROM student WHERE s_name LIKE '小%'
```


![img](https://img-blog.csdnimg.cn/2ef87a755b5e4f708e53495cf1b16d77.png)

案例 8（IN）：

```java
SELECT * FROM student WHERE s_name IN ('小王','小周')
```


![img](https://img-blog.csdnimg.cn/2b9fc03c60444a5cb59f32acfd420370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 二. as 取别名

- 表里的名字没有变，只影响了查询出来的结果

案例：

```java
SELECT s_name as `name` FROM student
```


![img](https://img-blog.csdnimg.cn/1f442275ab194b72a7689b01fbd1fb99.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 使用as也可以为表取别名 **（作用：单表查询意义不大，但是当多个表的时候取别名就好操作，当不同的表里有相同名字的列的时候区分就会好区分）**

### 三. distinct 去除重复记录

- 注意：当查询结果中所有字段全都相同时 才算重复的记录

案例

```java
SELECT DISTINCT * FROM student
```


![img](https://img-blog.csdnimg.cn/fbae5772fb744a79adc1adc43aee4d77.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

**指定字段**

1. 星号表示所有字段 
2. 手动指定需要查询的字段

```java
SELECT DISTINCT s_name,s_birth FROM student
```


![img](https://img-blog.csdnimg.cn/3afe0916f1f84530948fd55a33e6285a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

1. 还可也是四则运算 
2. 聚合函数

### 四. group by 分组

- group by的意思是根据by对数据按照哪个字段进行分组，或者是哪几个字段进行分组。

语法：

```java
select 字段名 from 表名 group by 字段名称;
```

#### 1. 单个字段分组

```java
SELECT COUNT(*)FROM student GROUP BY s_sex;
```


![img](https://img-blog.csdnimg.cn/890e8b1fef52458c926ae286fc032d07.png)

#### 2. 多个字段分组

```java
SELECT s_name,s_sex,COUNT(*) FROM student GROUP BY s_name,s_sex;
```


![img](https://img-blog.csdnimg.cn/f6b31c834872470f8416047532692aa6.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 注意：多个字段进行分组时，需要将s_name和s_sex看成一个整体，只要是s_name和s_sex相同的可以分成一组；如果只是s_sex相同，s_sex不同就不是一组。

### 五. having 过滤

- HAVING 子句对 GROUP BY 子句设置条件的方式与 WHERE 和 SELECT 的交互方式类似。WHERE 搜索条件在进行分组操作之前应用；而 HAVING 搜索条件在进行分组操作之后应用。HAVING 语法与 WHERE 语法类似，但 HAVING 可以包含聚合函数。HAVING 子句可以引用选择列表中显示的任意项。

我们如果要查询男生或者女生，人数大于4的性别

```java
SELECT s_sex as 性别,count(s_id) AS 人数 FROM student GROUP BY s_sex HAVING COUNT(s_id)>4
```


![img](https://img-blog.csdnimg.cn/666914d0b8ee4899902a785c57a77b53.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 六. order by 排序

- 根据某个字段排序，默认升序(从小到大)

语法：

```java
select * from 表名 order by 字段名;
```

#### 1. 一个字段，降序（从大到小）

```java
SELECT * FROM student ORDER BY s_id DESC;
```


![img](https://img-blog.csdnimg.cn/845119940241447e9041c8e4180803cc.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

#### 2. 多个字段

```java
SELECT * FROM student ORDER BY s_id DESC, s_birth ASC;
```


![img](https://img-blog.csdnimg.cn/f750c5745edb49659815fb190b578b0d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 多个字段 第一个相同在按照第二个 asc 表示升序

### limit 分页

- 用于限制要显示的记录数量

语法1:

```java
select * from table_name limit 个数;
```

语法2:

```java
select * from table_name limit 起始位置,个数;
```

案例：

- 查询前三条数据

```java
SELECT * FROM student LIMIT 3;
```


![img](https://img-blog.csdnimg.cn/caa9275fa8e0476384a95fed9f289ce6.png)

- 从第三条开始 查询3条

```java
SELECT * FROM student LIMIT 2,3;
```


![img](https://img-blog.csdnimg.cn/caaa4bd61f584abd92ac1c35a758f564.png)

注意：起始位置 从0开始

>经典的使用场景:分页显示

1. 每一页显示的条数 a = 3 
2. 明确当前页数 b = 2 
3. 计算起始位置 c = (b-1) * a

### 子查询

- 将一个查询语句的结果作为另一个查询语句的条件或是数据来源，​ 当我们一次性查不到想要数据时就需要使用子查询。

```java
SELECT
	* 
FROM
	score 
WHERE
	s_id =(
	SELECT
		s_id 
	FROM
		student 
WHERE
	s_name = '赵信')
```


![img](https://img-blog.csdnimg.cn/8610d0e67a424359aec6a5768b107267.png)

#### 1. in 关键字子查询

- 当内层查询 (括号内的) 结果会有多个结果时, 不能使用 = 必须是in ,另外子查询必须只能包含一列数据

子查询的思路:

1. 要分析 查到最终的数据 到底有哪些步骤 
2. 根据步骤写出对应的sql语句 
3. 把上一个步骤的sql语句丢到下一个sql语句中作为条件

```java
SELECT
	* 
FROM
	score 
WHERE
	s_id IN (
	SELECT
		s_id 
	FROM
		student 
WHERE
	s_sex = '男')
```


![img](https://img-blog.csdnimg.cn/695a8cadfe7246488a6d5b19c3ee470d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

#### exists 关键字子查询

- 当内层查询 有结果时 外层才会执行

### 多表查询

#### 1. 笛卡尔积查询

- 笛卡尔积查询的结果会出现大量的错误数据即,数据关联关系错误,并且会产生重复的字段信息 !

#### 2. 内连接查询

- 本质上就是笛卡尔积查询，inner可以省略。


![img](https://img-blog.csdnimg.cn/55ea38eec2914c2abdf62d57701a9eb4.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

语法:

```java
select * from  表1 inner join 表2;
```

#### 3. 左外连接查询

- 左边的表无论是否能够匹配都要完整显示，右边的仅展示匹配上的记录


![img](https://img-blog.csdnimg.cn/6f13463d6f8947bebc4a4c41cd41b341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 注意: 在外连接查询中不能使用where 关键字 必须使用on专门来做表的对应关系

#### 4. 右外连接查询

- 右边的表无论是否能够匹配都要完整显示，左边的仅展示匹配上的记录


![img](https://img-blog.csdnimg.cn/a09cbedbf5e749289b7850802ae06ba9.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)


## DCL（Data Control Language）语句：即数据控制语句

- DCL(Data Control Language)语句：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。

### 关键字

- **GRANT** 
- **REVOKE**

### 查看用户权限

当成功创建用户账户后，还不能执行任何操作，需要为该用户分配适当的访问权限。可以使用SHOW GRANTS FOR语句来查询用户的权限。

例如：

```java
mysql> SHOW GRANTS FOR test;
+-------------------------------------------+
| Grants for test@%                         |
+-------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' |
+-------------------------------------------+
1 row in set (0.00 sec)
```

### GRANT语句

- 对于新建的MySQL用户，必须给它授权，可以用GRANT语句来实现对新建用户的授权。

#### 格式语法

```java
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH {
  GRANT OPTION | resource_option} ...]

GRANT PROXY ON user
    TO user [, user] ...
    [WITH GRANT OPTION]

object_type: {
    TABLE
  | FUNCTION
  | PROCEDURE
}

priv_level: {
    *
  | *.*
  | db_name.*
  | db_name.tbl_name
  | tbl_name
  | db_name.routine_name
}

user:
    (see Section 6.2.4, “Specifying Account Names”)

auth_option: {
    IDENTIFIED BY 'auth_string'
  | IDENTIFIED WITH auth_plugin
  | IDENTIFIED WITH auth_plugin BY 'auth_string'
  | IDENTIFIED WITH auth_plugin AS 'auth_string'
  | IDENTIFIED BY PASSWORD 'auth_string'
}

tls_option: {
    SSL
  | X509
  | CIPHER 'cipher'
  | ISSUER 'issuer'
  | SUBJECT 'subject'
}

resource_option: {
  | MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
}
```

#### 权限类型(priv_type)

- 授权的权限类型一般可以分为数据库、表、列、用户。

##### 授予数据库权限类型

授予数据库权限时，priv_type可以指定为以下值：

- SELECT：表示授予用户可以使用 SELECT 语句访问特定数据库中所有表和视图的权限。 
- INSERT：表示授予用户可以使用 INSERT 语句向特定数据库中所有表添加数据行的权限。 
- DELETE：表示授予用户可以使用 DELETE 语句删除特定数据库中所有表的数据行的权限。 
- UPDATE：表示授予用户可以使用 UPDATE 语句更新特定数据库中所有数据表的值的权限。 
- REFERENCES：表示授予用户可以创建指向特定的数据库中的表外键的权限。 
- CREATE：表示授权用户可以使用 CREATE TABLE 语句在特定数据库中创建新表的权限。 
- ALTER：表示授予用户可以使用 ALTER TABLE 语句修改特定数据库中所有数据表的权限。 
- SHOW VIEW：表示授予用户可以查看特定数据库中已有视图的视图定义的权限。 
- CREATE ROUTINE：表示授予用户可以为特定的数据库创建存储过程和存储函数的权限。 
- ALTER ROUTINE：表示授予用户可以更新和删除数据库中已有的存储过程和存储函数的权限。 
- INDEX：表示授予用户可以在特定数据库中的所有数据表上定义和删除索引的权限。 
- DROP：表示授予用户可以删除特定数据库中所有表和视图的权限。 
- CREATE TEMPORARY TABLES：表示授予用户可以在特定数据库中创建临时表的权限。 
- CREATE VIEW：表示授予用户可以在特定数据库中创建新的视图的权限。 
- EXECUTE ROUTINE：表示授予用户可以调用特定数据库的存储过程和存储函数的权限。 
- LOCK TABLES：表示授予用户可以锁定特定数据库的已有数据表的权限。 
- SHOW DATABASES：表示授权可以使用SHOW DATABASES语句查看所有已有的数据库的定义的权限。 
- ALL或ALL PRIVILEGES：表示以上所有权限。

#### 授予表权限类型

授予表权限时，priv_type可以指定为以下值：

- SELECT：授予用户可以使用 SELECT 语句进行访问特定表的权限。 
- INSERT：授予用户可以使用 INSERT 语句向一个特定表中添加数据行的权限。 
- DELETE：授予用户可以使用 DELETE 语句从一个特定表中删除数据行的权限。 
- DROP：授予用户可以删除数据表的权限。 
- UPDATE：授予用户可以使用 UPDATE 语句更新特定数据表的权限。 
- ALTER：授予用户可以使用 ALTER TABLE 语句修改数据表的权限。 
- REFERENCES：授予用户可以创建一个外键来参照特定数据表的权限。 
- CREATE：授予用户可以使用特定的名字创建一个数据表的权限。 
- INDEX：授予用户可以在表上定义索引的权限。 
- ALL或ALL PRIVILEGES：所有的权限名。

##### 授予列(字段)权限类型

- 授予列(字段)权限时，priv_type的值只能指定为SELECT、INSERT和UPDATE，同时权限的后面需要加上列名列表(column-list)。

##### 授予创建和删除用户的权限

- 授予列(字段)权限时，priv_type的值指定为CREATE USER权限，具备创建用户、删除用户、重命名用户和撤消所有特权，而且是全局的。

#### ON

- 有ON，是授予权限，无ON，是授予角色。如：

```java
-- 授予数据库db1的所有权限给指定账户
GRANT ALL ON db1.* TO 'user1'@'localhost';
-- 授予角色给指定的账户
GRANT 'role1', 'role2' TO 'user1'@'localhost', 'user2'@'localhost';
```

#### 对象类型(object_type)

- 在ON关键字后给出要授予权限的object_type，通常object_type可以是数据库名、表名等。

#### 权限级别(priv_level)

指定权限级别的值有以下几类格式：

- *：表示当前数据库中的所有表。 
-  *.* ：表示所有数据库中的所有表。 
- db_name.*：表示某个数据库中的所有表，db_name指定数据库名。 
- db_name.tbl_name：表示某个数据库中的某个表或视图，db_name指定数据库名，tbl_name指定表名或视图名。 
- tbl_name：表示某个表或视图，tbl_name指定表名或视图名。 
- db_name.routine_name：表示某个数据库中的某个存储过程或函数，routine_name指定存储过程名或函数名。

#### 被授权的用户(user)

```java
'user_name'@'host_name'
```

- Tips：'host_name’用于适应从任意主机访问数据库而设置的，可以指定某个地址或地址段访问。 
- 可以同时授权多个用户。

user表中host列的默认值


host_name格式有以下几种：

- 使用%模糊匹配，符合匹配条件的主机可以访问该数据库实例，例如192.168.2.%或%.test.com； 
- 使用localhost、127.0.0.1、::1及服务器名等，只能在本机访问； 
- 使用ip地址或地址段形式，仅允许该ip或ip地址段的主机访问该数据库实例，例如192.168.2.1或192.168.2.0/24或192.168.2.0/255.255.255.0； 
- 省略即默认为%。

#### 身份验证方式(auth_option)

- auth_option为可选字段，可以指定密码以及认证插件(mysql_native_password、sha256_password、caching_sha2_password)。

#### 加密连接(tls_option)

- tls_option为可选的，一般是用来加密连接。

#### 用户资源限制(resource_option)

- resource_option为可选的，一般是用来指定最大连接数等。


#### 权限生效

- 若要权限生效，需要执行以下语句：

```java
FLUSH PRIVILEGES;
```

### REVOKE语句

- REVOKE语句主要用于撤销权限。

#### 语法格式

- REVOKE语法和GRANT语句的语法格式相似，但具有相反的效果

```java
REVOKE
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    FROM user [, user] ...

REVOKE ALL [PRIVILEGES], GRANT OPTION
    FROM user [, user] ...

REVOKE PROXY ON user
    FROM user [, user] ...
```

- 若要使用REVOKE语句，必须拥有MySQL数据库的全局CREATE USER权限或UPDATE权限; 
- 第一种语法格式用于回收指定用户的某些特定的权限，第二种回收指定用户的所有权限；


## TCL（Transaction Control Language）语句：事务控制语句

**什么是事物？**

- 一个或一组sql语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行

**事务的ACID属性**

-  原子性：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生  
-  一致性：事务必须使数据库从一个一致性状态变换到另外一个一致性状态  
-  隔离性：一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰  
-  持久性：一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响 

**分类**

-  隐式事务：事务没有明显的开启和结束的标记(比如insert，update，delete语句)  
-  显式事务：事务具有明显的开启和结束的标记(autocommit变量设置为0) 

### 事务的使用步骤

#### 开启事务

- 默认开启事务

```java
SET autocommit = 0 ;
```

#### 提交事务

```java
COMMIT;
```

#### 回滚事务

```java
ROLLBACK ;
```

#### 查看当前的事务隔离级别

```java
select @@tx_isolation;
```

#### 设置当前连接事务的隔离级别

```java
set session transaction isolation level read uncommitted;
```

#### 设置数据库系统的全局的隔离级别

```java
set global transaction isolation level read committed ;
```



- MySQL提供了众多功能强大、方便易用的函数，使用这些函数，可以极大地提高用户对于数据库的管理效率，从而更加灵活地满足不同用户的需求。本文将MySQL的函数分类并汇总，以便以后用到的时候可以随时查看。

(这里使用 Navicat Premium 15 工具进行演示)


![img](https://img-blog.csdnimg.cn/87caf7415ffd4a50b2c035de378a8413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)


 *因为内容太多了这里只演示一些常用的* ![img](https://img-blog.csdnimg.cn/3fc4bf79c8eb47d48c94d36333ab175a.gif#pic_center)

## 一. 数学函数

对数值型的数据进行指定的数学运算，如abs()函数可以获得给定数值的绝对值，round()函数可以对给定的数值进行四舍五入。

### 1. ABS(number)

- 作用：返回 number 的绝对值

```java
SELECT
 ABS(s_score)
FROM
	score;
```


![img](https://img-blog.csdnimg.cn/6cadf07104e24fe5b7b4315ae1bb00d6.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/998d03dea4d74686b6f59ea57d593fe9.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

-  ABS(-86) 返回：86  
-  number 参数可以是任意有效的数值表达式。如果 number 包含 Null，则返回 Null；如果是未初始化变量，则返回 0。 

### 2. PI()

-  例1：pi() 返回：3.141592653589793  
-  例2：pi(2) 返回：6.283185307179586  
-  作用：计算圆周率及其倍数 

### 3. SQRT(x)

- 作用：返回非负数的x的二次方根

### 4. MOD(x,y)

- 作用：返回x被y除后的余数

### 5. CEIL(x)、CEILING(x)

- 作用：返回不小于x的最小整数

### 6. FLOOR(x)

- 作用：返回不大于x的最大整数

### 7. FLOOR(x)

- 作用：返回不大于x的最大整数

### 8. ROUND(x)、ROUND(x,y)

- 作用：前者返回最接近于x的整数，即对x进行四舍五入；后者返回最接近x的数，其值保留到小数点后面y位，若y为负值，则将保留到x到小数点左边y位

```java
SELECT ROUND(345222.9)
```


![img](https://img-blog.csdnimg.cn/602b47635e254481a156bf045ea3b6a6.png)

- 参数说明： numberExp 需要进行截取的数据 nExp 整数，用于指定需要进行截取的位置，&gt;0：从小数点往右位移nExp个位数， &lt;0：从小数点往左

nExp个位数 =0：表示当前小数点的位置

### 9. POW(x,y)和、POWER(x,y)

- 作用：返回x的y次乘方的值

### 10. EXP(x)

- 作用：返回e的x乘方后的值

### 11. LOG(x)

- 作用：返回x的自然对数，x相对于基数e的对数

### 12. LOG10(x)

- 作用：返回x的基数为10的对数

### 13. RADIANS(x)

- 作用：返回x由角度转化为弧度的值

### 14. DEGREES(x)

- 作用：返回x由弧度转化为角度的值

### 15. SIN(x)、ASIN(x)

- 作用：前者返回x的正弦，其中x为给定的弧度值；后者返回x的反正弦值，x为正弦

### 16. COS(x)、ACOS(x)

- 作用：前者返回x的余弦，其中x为给定的弧度值；后者返回x的反余弦值，x为余弦

### 17. TAN(x)、ATAN(x)

- 作用：前者返回x的正切，其中x为给定的弧度值；后者返回x的反正切值，x为正切

### 18. COT(x)

- 作用：返回给定弧度值x的余切

## 二. 字符串函数

### 1. CHAR_LENGTH(str)

- 作用：计算字符串字符个数

```java
SELECT CHAR_LENGTH('这是一个十二个字的字符串');
```


![img](https://img-blog.csdnimg.cn/1a7643e3a9e2473ebcc6cc197ddc847e.png)

### 2. CONCAT(s1,s2，…)

- 作用：返回连接参数产生的字符串，一个或多个待拼接的内容，任意一个为NULL则返回值为NULL

```java
SELECT CONCAT('拼接','测试');
```


![img](https://img-blog.csdnimg.cn/5bf70d552d7544378760d172b2ebbbef.png)

### 3. CONCAT_WS(x,s1,s2,…)

- 作用：返回多个字符串拼接之后的字符串，每个字符串之间有一个x

```java
SELECT CONCAT_WS('-','测试','拼接','WS')
```


![img](https://img-blog.csdnimg.cn/2d028c963f2644bda52859b473d0d8f0.png)

### 4. INSERT(s1,x,len,s2)

- 作用：返回字符串s1，其子字符串起始于位置x，被字符串s2取代len个字符

```java
SELECT INSERT('测试字符串替换',2,1,'牛');
```


![img](https://img-blog.csdnimg.cn/dcc86c26d71d4e50b9db799cf3e1fc53.png)

### 5. LOWER(str)和LCASE(str)、UPPER(str)和UCASE(str)

- 作用：前两者将str中的字母全部转换成小写，后两者将字符串中的字母全部转换成大写

```java
SELECT LOWER('JHGYTUGHJGG'),LCASE('HKJHKJHKJHKJ');
```


![img](https://img-blog.csdnimg.cn/e941fbd3b6ee4567b3c8d239c3bc9b88.png)

```java
SELECT UPPER('aaaaaa'),UCASE('vvvvv');
```


![img](https://img-blog.csdnimg.cn/c5ba7e4cda9b473184c2242c7a85bd7a.png)

### 6. LEFT(s,n)、RIGHT(s,n)

- 作用：前者返回字符串s从最左边开始的n个字符，后者返回字符串s从最右边开始的n个字符

```java
SELECT LEFT('左边开始',2),RIGHT('右边开始',2);
```


![img](https://img-blog.csdnimg.cn/6c1381dfedce4d1eb40919059dffad69.png)

### 7. LPAD(s1,len,s2)、RPAD(s1,len,s2)

- 作用：前者返回s1，其左边由字符串s2填补到len字符长度，假如s1的长度大于len，则返回值被缩短至len字符；前者返回s1，其右边由字符串s2填补到len字符长度，假如s1的长度大于len，则返回值被缩短至len字符

```java
SELECT LEFT('左边开始',2),RIGHT('右边开始',2);
```


![img](https://img-blog.csdnimg.cn/446c2f7dedc345cd9ce28dadfa02c198.png)

### 8. LTRIM(s)、RTRIM(s)

- 作用：前者返回字符串s，其左边所有空格被删除；后者返回字符串s，其右边所有空格被删除

```java
SELECT LTRIM('       左边开始'),RTRIM('    右边开始         ');
```


![img](https://img-blog.csdnimg.cn/1772b5d7231d449f9a291e1d6cd80a45.png)

### 9. TRIM(s)

- 作用：返回字符串s删除了两边空格之后的字符串

```java
SELECT TRIM(' 是是 ');
```


![img](https://img-blog.csdnimg.cn/30601d6e89584bad83816c781df75551.png)

### 10. TRIM(s1 FROM s)

- 作用：删除字符串s两端所有子字符串s1，未指定s1的情况下则默认删除空格

### 11. REPEAT(s,n)

- 作用：返回一个由重复字符串s组成的字符串，字符串s的数目等于n

```java
SELECT REPEAT('测试',5);
```


![img](https://img-blog.csdnimg.cn/f4283e94edfc49d39aec633851efffd5.png)

### 12. SPACE(n)

- 作用：返回一个由n个空格组成的字符串

```java
SELECT SPACE(20);
```


![img](https://img-blog.csdnimg.cn/21814d2a9db24d0db563afeba1e0e06f.png)

### 13. REPLACE(s,s1,s2)

- 作用：返回一个字符串，用字符串s2替代字符串s中所有的字符串s1

### 14. STRCMP(s1,s2)

- 作用：若s1和s2中所有的字符串都相同，则返回0；根据当前分类次序，第一个参数小于第二个则返回-1，其他情况返回1

```java
SELECT STRCMP('我我我','我我我');
```


![img](https://img-blog.csdnimg.cn/c3f75f58392c4f30b5bdeb7e3a7289c0.png)

```java
SELECT STRCMP('我我我','是是是');
```


![img](https://img-blog.csdnimg.cn/4383e02348aa4ad5996fd9d061551751.png)

### 15. SUBSTRING(s,n,len)、MID(s,n,len)

- 作用：两个函数作用相同，从字符串s中返回一个第n个字符开始、长度为len的字符串

```java
SELECT SUBSTRING('测试测试',2,2);
```


![img](https://img-blog.csdnimg.cn/4b8ea2c00da1413f9002f36182b505d1.png)

```java
SELECT MID('测试测试',2,2);
```


![img](https://img-blog.csdnimg.cn/4f24022b4da74e299a2dd3b118f42c4b.png)

### 16. LOCATE(str1,str)、POSITION(str1 IN str)、INSTR(str,str1)

- 作用：三个函数作用相同，返回子字符串str1在字符串str中的开始位置（从第几个字符开始）

```java
SELECT LOCATE('字','获取字符串的位置');
```


![img](https://img-blog.csdnimg.cn/a080a19b3f08417180d85c6b6cc4b6eb.png)

### 17. REVERSE(s)

- 作用：将字符串s反转

```java
SELECT REVERSE('字符串反转');
```


![img](https://img-blog.csdnimg.cn/d50310f57e3346efbca5f01dafb5cfa0.png)

### 18. ELT(N,str1,str2,str3,str4,…)

- 作用：返回第N个字符串

```java
SELECT ELT(2,'字符串反转','sssss');
```


![img](https://img-blog.csdnimg.cn/0f25494edecb470093e7d17aedb3a0aa.png)

## 三. 日期和时间函数


 *当前时间*  ![img](https://img-blog.csdnimg.cn/a37a099993a447a9840e7495e28a9ad9.png)

### 1. CURDATE()、CURRENT_DATE()

- 作用：将当前日期按照"YYYY-MM-DD"或者"YYYYMMDD"格式的值返回，具体格式根据函数用在字符串或是数字语境中而定

### 2. CURRENT_TIMESTAMP()、LOCALTIME()、NOW()、SYSDATE()

- 作用：这四个函数作用相同，返回当前日期和时间值，格式为"YYYY_MM-DD HH:MM:SS"或"YYYYMMDDHHMMSS"，具体格式根据函数用在字符串或数字语境中而定

```java
SELECT CURRENT_TIMESTAMP()
```


![img](https://img-blog.csdnimg.cn/ded5b8ae1d4c4a40b28ce18cb3c89078.png)

```java
SELECT LOCALTIME()
```


![img](https://img-blog.csdnimg.cn/f429f4851e5f4ff3bd0062447f7bc54f.png)

```java
SELECT NOW()
```


![img](https://img-blog.csdnimg.cn/e474f8bfcddb4e5bbe342061da69d817.png)

```java
SELECT SYSDATE()
```


![img](https://img-blog.csdnimg.cn/2234c86e346046c5a7cb834cbbf8bcdf.png)

### 3. UNIX_TIMESTAMP()、UNIX_TIMESTAMP(date)

- 作用：前者返回一个格林尼治标准时间1970-01-01 00:00:00到现在的秒数，后者返回一个格林尼治标准时间1970-01-01 00:00:00到指定时间的秒数

```java
SELECT UNIX_TIMESTAMP()
```


![img](https://img-blog.csdnimg.cn/af3e89a5a90d4ec187e6f6a986bdf453.png)

### 4. FROM_UNIXTIME(date)

- 作用：和UNIX_TIMESTAMP互为反函数，把UNIX时间戳转换为普通格式的时间

### 5. UTC_DATE()和UTC_TIME()

- 前者返回当前UTC（世界标准时间）日期值，其格式为"YYYY-MM-DD"或"YYYYMMDD"，后者返回当前UTC时间值，其格式为"YYYY-MM-DD"或"YYYYMMDD"。具体使用哪种取决于函数用在字符串还是数字语境中

```java
SELECT UTC_DATE()
```


![img](https://img-blog.csdnimg.cn/f2f6d4cc08c94919a73da94de6e72eee.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

```java
SELECT UTC_TIME()
```


![img](https://img-blog.csdnimg.cn/895993dde96e4ed0bea9a58e44dfb4c8.png)

### 6. MONTH(date)和MONTHNAME(date)

- 作用：前者返回指定日期中的月份，后者返回指定日期中的月份的名称

```java
SELECT MONTH(NOW())
```


![img](https://img-blog.csdnimg.cn/aa014ca701b745aa830c36f9c732bc2d.png)

```java
SELECT MONTHNAME(NOW())
```


![img](https://img-blog.csdnimg.cn/79ddb6cdb4d9401bba0ae24ed75e80df.png)

### 7. DAYNAME(d)、DAYOFWEEK(d)、WEEKDAY(d)

- 作用：DAYNAME(d)返回d对应的工作日的英文名称，如Sunday、Monday等；DAYOFWEEK(d)返回的对应一周中的索引，1表示周日、2表示周一；WEEKDAY(d)表示d对应的工作日索引，0表示周一，1表示周二

### 8. WEEK(d)

- 计算日期d是一年中的第几周

```java
SELECT WEEK(NOW())
```


![img](https://img-blog.csdnimg.cn/8037d67f871e4503bd90729ff4359d18.png)

### 9. DAYOFYEAR(d)、DAYOFMONTH(d)

- 作用：前者返回d是一年中的第几天，后者返回d是一月中的第几天

```java
SELECT DAYOFYEAR(NOW())
```


![img](https://img-blog.csdnimg.cn/5d7dd09d0e4a45888473bea4e10afc19.png)

```java
SELECT DAYOFMONTH(NOW())
```


![img](https://img-blog.csdnimg.cn/b66b065304e74db7b8cf6fc883732c1a.png)

### 10. YEAR(date)、QUARTER(date)、MINUTE(time)、SECOND(time)

- 作用： YEAR(date)返回指定日期对应的年份，范围是1970~2069；QUARTER(date)返回date对应一年中的季度，范围是1~4；MINUTE(time)返回time对应的分钟数，范围是0~59；SECOND(time)返回制定时间的秒值

```java
SELECT YEAR(NOW())
```


![img](https://img-blog.csdnimg.cn/0a73e72f8b9d4eb48f9e98ff6267d0bf.png)

```java
SELECT QUARTER(NOW())
```


![img](https://img-blog.csdnimg.cn/2131cb741c8042a4b88e902e695ede09.png)

```java
SELECT MINUTE(NOW())
```


![img](https://img-blog.csdnimg.cn/f34b774b7cce4e5bb43e39dfa096cec6.png)

```java
SELECT SECOND(NOW())
```


![img](https://img-blog.csdnimg.cn/6b34137858084608bf359ddc7b2e422e.png)

### 11. EXTRACE(type FROM date)

- 作用：从日期中提取一部分，type可以是YEAR、YEAR_MONTH、DAY_HOUR、DAY_MICROSECOND、DAY_MINUTE、DAY_SECOND

### 12. TIME_TO_SEC(time)

- 作用：返回以转换为秒的time参数，转换公式为"3600 *小时 + 60* 分钟 + 秒"

```java
SELECT TIME_TO_SEC(NOW())
```


![img](https://img-blog.csdnimg.cn/ed1f98799b134bbcb2dfcc3af1114a42.png)

### 13. SEC_TO_TIME()

- 作用：和TIME_TO_SEC(time)互为反函数，将秒值转换为时间格式

```java
SELECT SEC_TO_TIME(530)
```


![img](https://img-blog.csdnimg.cn/350f3b43c9884779b7c72b70c4093d37.png)

### 14. DATE_ADD(date,INTERVAL expr type)、ADD_DATE(date,INTERVAL expr type)

- 作用：返回将起始时间加上expr type之后的时间，比如DATE_ADD(‘2010-12-31 23:59:59’, INTERVAL 1 SECOND)表示的就是把第一个时间加1秒

### 15. DATE_SUB(date,INTERVAL expr type)、SUBDATE(date,INTERVAL expr type)

- 作用：返回将起始时间减去expr type之后的时间

### 16. ADDTIME(date,expr)、SUBTIME(date,expr)

- 作用：前者进行date的时间加操作，后者进行date的时间减操作

## 四. 条件判断函数

### 1. IF(expr,v1,v2)

- 作用：如果expr是TRUE则返回v1，否则返回v2

### 2. IFNULL(v1,v2)

- 作用：如果v1不为NULL，则返回v1，否则返回v2

### 3. CASE expr WHEN v1 THEN r1 [WHEN v2 THEN v2] [ELSE rn] END

- 作用：如果expr等于某个vn，则返回对应位置THEN后面的结果，如果与所有值都不想等，则返回ELSE后面的rn

## 五. 系统信息函数

### 1. VERSION()

- 作用：查看MySQL版本号

```java
SELECT VERSION()
```


![img](https://img-blog.csdnimg.cn/e4da87a383d447b1a574146e98a4dc3e.png)

### 2. CONNECTION_ID()

- 作用：查看当前用户的连接数

```java
SELECT CONNECTION_ID()
```


![img](https://img-blog.csdnimg.cn/20a6700ba7394f18a6b877d65f706154.png)

### 3. USER()、CURRENT_USER()、SYSTEM_USER()、SESSION_USER()

- 作用：查看当前被MySQL服务器验证的用户名和主机的组合，一般这几个函数的返回值是相同的

```java
SELECT USER()
```


![img](https://img-blog.csdnimg.cn/0c8471716cf941998460a657edf32501.png)

```java
SELECT CURRENT_USER()
```


![img](https://img-blog.csdnimg.cn/c6da3feb11184d9791fe0c5d2fb0e4c4.png)

```java
SELECT SYSTEM_USER()
```


![img](https://img-blog.csdnimg.cn/9f524e0c8f904690b5901f419dc07c9e.png)

```java
SELECT SESSION_USER()
```


![img](https://img-blog.csdnimg.cn/bffc0cce25ef47e7b9c6afb553a0f766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 4. CHARSET(str)

- 作用：查看字符串str使用的字符集

```java
SELECT CHARSET(555)
```


![img](https://img-blog.csdnimg.cn/cb0654f72acd4db8aa3f7a625861458c.png)

### 5. COLLATION()

- 作用：查看字符串排列方式

```java
SELECT COLLATION('sssfddsfds')
```


![img](https://img-blog.csdnimg.cn/44af017007fd44d086b91f4df77ca662.png)

## 六. 加密函数

### 1. PASSWORD(str)

- 作用：从原明文密码str计算并返回加密后的字符串密码，注意这个函数的加密是单向的（不可逆），因此不应将它应用在个人的应用程序中而应该只在MySQL服务器的鉴定系统中使用

```java
SELECT PASSWORD('mima')
```


![img](https://img-blog.csdnimg.cn/cfaef03a7f184fc689296f32b2b9f5b9.png)

### 2. MD5(str)

- 作用：为字符串算出一个MD5 128比特校验和，改值以32位十六进制数字的二进制字符串形式返回

```java
SELECT MD5('mima')
```


![img](https://img-blog.csdnimg.cn/99e4fc00c7e140b59d97c4e9036494a1.png)

### 3. ENCODE(str, pswd_str)

- 作用：使用pswd_str作为密码，加密str

```java
SELECT ENCODE('fdfdz','mima')
```


![img](https://img-blog.csdnimg.cn/0c41ce4455864accb008001f4120bb46.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 4. DECODE(crypt_str,pswd_str)

- 作用：使用pswd_str作为密码，解密加密字符串crypt_str，crypt_str是由ENCODE函数返回的字符串

```java
SELECT DECODE('fdfdz','mima')
```


![img](https://img-blog.csdnimg.cn/03c1c54104114674804bf096af5969c9.png)

## 七. 其他函数

### 1. FORMAT(x,n)

- 作用：将数字x格式化，并以四舍五入的方式保留小数点后n位，结果以字符串形式返回

```java
SELECT FORMAT(446.454,2)
```


![img](https://img-blog.csdnimg.cn/a9fca31470374884996657433eb90595.png)

### 2. CONV(N,from_base,to_base)

- 作用：不同进制数之间的转换，返回值为数值N的字符串表示，由from_base进制转换为to_base进制

### 3. INET_ATON(expr)

- 作用：给出一个作为字符串的网络地址的点地址表示，返回一个代表该地址数值的整数，地址可以使4或8比特

### 4. INET_NTOA(expr)

- 作用：给定一个数字网络地址（4或8比特），返回作为字符串的该地址的点地址表示

### 5. BENCHMARK(count,expr)

- 作用：重复执行count次表达式expr，它可以用于计算MySQL处理表达式的速度，结果值通常是0（0只是表示很快，并不是没有速度）。 
- 另一个作用是用它在MySQL客户端内部报告语句执行的时间

### 6. CONVERT(str USING charset)

- 作用：使用字符集charset表示字符串str

更多用法还请参考： [http://www.geezn.com/documents/gez/help/117555-1355219868404378.html](http://www.geezn.com/documents/gez/help/117555-1355219868404378.html)


![img](https://img-blog.csdnimg.cn/fbacd2b6f4f44005b7e98d42372c7861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)


- 题目来自互联网，建议每道题都在本地敲一遍巩固记忆 ！

## 创建数据库


![img](https://img-blog.csdnimg.cn/aa54242663664a1184216e9b88db6e8a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

## 创建表（并初始化数据）

```java
-- 学生表
CREATE TABLE `student`(
`s_id` VARCHAR(20),
`s_name` VARCHAR(20) NOT NULL DEFAULT '',
`s_birth` VARCHAR(20) NOT NULL DEFAULT '',
`s_sex` VARCHAR(10) NOT NULL DEFAULT '',
PRIMARY KEY(`s_id`)
);
-- 课程表
CREATE TABLE `course`(
`c_id` VARCHAR(20),
`c_name` VARCHAR(20) NOT NULL DEFAULT '',
`t_id` VARCHAR(20) NOT NULL,
PRIMARY KEY(`c_id`)
);
-- 教师表
CREATE TABLE `teacher`(
`t_id` VARCHAR(20),
`t_name` VARCHAR(20) NOT NULL DEFAULT '',
PRIMARY KEY(`t_id`)
);
-- 成绩表
CREATE TABLE `score`(
`s_id` VARCHAR(20),
`c_id` VARCHAR(20),
`s_score` INT(3),
PRIMARY KEY(`s_id`,`c_id`)
);

-- 插入学生表测试数据
insert into student values('01' , '赵信' , '1990-01-01' , '男');
insert into student values('02' , '德莱厄斯' , '1990-12-21' , '男');
insert into student values('03' , '艾希' , '1990-05-20' , '男');
insert into student values('04' , '德莱文' , '1990-08-06' , '男');
insert into student values('05' , '俄洛依' , '1991-12-01' , '女');
insert into student values('06' , '光辉女郎' , '1992-03-01' , '女');
insert into student values('07' , '崔丝塔娜' , '1989-07-01' , '女');
insert into student values('08' , '安妮' , '1990-01-20' , '女');
-- 课程表测试数据
insert into course values('01' , '语文' , '02');
insert into course values('02' , '数学' , '01');
insert into course values('03' , '英语' , '03');

-- 教师表测试数据
insert into teacher values('01' , '死亡歌颂者');
insert into teacher values('02' , '流浪法师');
insert into teacher values('03' , '邪恶小法师');

-- 成绩表测试数据
insert into score values('01' , '01' , 80);
insert into score values('01' , '02' , 90);
insert into score values('01' , '03' , 99);
insert into score values('02' , '01' , 70);
insert into score values('02' , '02' , 60);
insert into score values('02' , '03' , 80);
insert into score values('03' , '01' , 80);
insert into score values('03' , '02' , 80);
insert into score values('03' , '03' , 80);
insert into score values('04' , '01' , 50);
insert into score values('04' , '02' , 30);
insert into score values('04' , '03' , 20);
insert into score values('05' , '01' , 76);
insert into score values('05' , '02' , 87);
insert into score values('06' , '01' , 31);
insert into score values('06' , '03' , 34);
insert into score values('07' , '02' , 89);
insert into score values('07' , '03' , 98);
```

## 表结构

- 这里建的表主要用于sql语句的练习，所以并没有遵守一些规范。下面让我们来看看相关的表结构吧

**学生表（student）**


![img](https://img-blog.csdnimg.cn/1503861c9c77405b8e083b6b0595ec55.png)

- s_id = 学生编号，s_name = 学生姓名，s_birth = 出生年月，s_sex = 学生性别

**课程表（course）**


![img](https://img-blog.csdnimg.cn/0c221e9384264e68b0e4760069daa733.png)

- c_id = 课程编号，c_name = 课程名称，t_id = 教师编号

**教师表（teacher）**


![img](https://img-blog.csdnimg.cn/cece47274e3145c2859d7f25e1246b75.png)

- t_id = 教师编号，t_name = 教师姓名

**成绩表（score）**


![img](https://img-blog.csdnimg.cn/22c6f01a00374a48b5f6880764865f3a.png)

- s_id = 学生编号，c_id = 课程编号，s_score = 分数

## 习题

- 开始之前我们先来看看四张表中的数据。


![img](https://img-blog.csdnimg.cn/99fe938a9fa44a1fa2e5ef922c5420ba.png)




![img](https://img-blog.csdnimg.cn/9b0f4f851899495ebd9e73a5ed752ee8.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70) ![img](https://img-blog.csdnimg.cn/0ea368756f5944ff8c3bab6a202ecb96.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)![img](https://img-blog.csdnimg.cn/7544084d59f74803a068583f2c510393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 1. 查询"01"课程比"02"课程成绩高的学生的信息及课程分数

```java
SELECT
	st.*,
	sc.s_score AS '语文',
	sc2.s_score '数学' 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
	AND sc.c_id = '01'
	LEFT JOIN score sc2 ON sc2.s_id = st.s_id 
	AND sc2.c_id = '02'
```


![img](https://img-blog.csdnimg.cn/ed35a1a92c954a7da0db2b7e005ace4b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 2. 查询"01"课程比"02"课程成绩低的学生的信息及课程分数

```java
SELECT
	st.*,
	s.s_score AS 数学,
	s2.s_score AS 语文 
FROM
	student st
	LEFT JOIN score s ON s.s_id = st.s_id 
	AND s.c_id = '01'
	LEFT JOIN score s2 ON s2.s_id = st.s_id 
	AND s2.c_id = '02' 
WHERE
	s.s_score < s2.s_score
```


![img](https://img-blog.csdnimg.cn/c81e1b5d4cb648309ef7af9188dc7e27.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 3. 查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩

```java
SELECT
	st.s_id AS '学生编号',
	st.s_name AS '学生姓名',
	AVG( s.s_score ) AS avgScore 
FROM
	student st
	LEFT JOIN score s ON st.s_id = s.s_id 
GROUP BY
	st.s_id 
HAVING
	avgScore >= 60
```


![img](https://img-blog.csdnimg.cn/d40e8352558144abb70b2ed8b62d3b76.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 4. 查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩

- (包括有成绩的和无成绩的)

```java
SELECT
	st.s_id AS '学生编号',
	st.s_name AS '学生姓名',(
	CASE
			
			WHEN ROUND( AVG( sc.s_score ), 2 ) IS NULL THEN
			0 ELSE ROUND( AVG( sc.s_score ), 2 ) 
		END 
		) 
	FROM
		student st
		LEFT JOIN score sc ON st.s_id = sc.s_id 
	GROUP BY
		st.s_id 
	HAVING
	AVG( sc.s_score )< 60 
	OR AVG( sc.s_score ) IS NULL
```


![img](https://img-blog.csdnimg.cn/5ba40851785d487ea6bfc324017c083b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 5. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩

```java
SELECT
	st.s_id AS '学生编号',
	st.s_name AS '学生姓名',
	COUNT( sc.c_id ) AS '选课总数',
	sum( CASE WHEN sc.s_score IS NULL THEN 0 ELSE sc.s_score END ) AS '总成绩' 
FROM
	student st
	LEFT JOIN score sc ON st.s_id = sc.s_id 
GROUP BY
	st.s_id
```


![img](https://img-blog.csdnimg.cn/2be40b7b279b48beadbba9516c62b79b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 6. 查询"流"姓老师的数量

```java
SELECT COUNT(t_id) FROM teacher WHERE t_name LIKE '流%'
```


![img](https://img-blog.csdnimg.cn/755f943200384ed7b982f18a78d23ac6.png)

### 7. 查询学过"流浪法师"老师授课的同学的信息

```java
SELECT
	st.* 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id
	LEFT JOIN course cs ON cs.c_id = sc.c_id
	LEFT JOIN teacher tc ON tc.t_id = cs.t_id 
	WHERE tc.t_name = '流浪法师'
```


![img](https://img-blog.csdnimg.cn/b9f30c4b1e194dccbf41a4fdbab3d094.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 8. 查询没学过"张三"老师授课的同学的信息

```java
-- 查询流浪法师教的课
SELECT
	cs.* 
FROM
	course cs
	LEFT JOIN teacher tc ON tc.t_id = cs.t_id 
WHERE
	tc.t_name = '流浪法师'



-- 查询有流浪法师课程成绩的学生id
SELECT
	sc.s_id 
FROM
	score sc 
WHERE
	sc.c_id IN (
	SELECT
		cs.c_id 
	FROM
		course cs
		LEFT JOIN teacher tc ON tc.t_id = cs.t_id 
	WHERE
	tc.t_name = '流浪法师')



-- 取反，查询没有学过流浪法师课程的同学信息
SELECT
	st.* 
FROM
	student st 
WHERE
	st.s_id NOT IN (
	SELECT
		sc.s_id 
	FROM
		score sc 
	WHERE
	sc.c_id IN ( SELECT cs.c_id FROM course cs LEFT JOIN teacher tc ON tc.t_id = cs.t_id WHERE tc.t_name = '流浪法师' ) 
	)
```


![img](https://img-blog.csdnimg.cn/bfc3df5291fe4daa9c7ffae37edffb34.png)

### 9. 查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息

- 方法 1

```java
-- 查询学过编号为01课程的同学id
SELECT
	st.s_id 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id
	INNER JOIN course cs ON cs.c_id = sc.c_id 
	AND cs.c_id = '01';
	
	

-- 查询学过编号为02课程的同学id
SELECT
	st2.s_id 
FROM
	student st2
	INNER JOIN score sc2 ON sc2.s_id = st2.s_id
	INNER JOIN course cs2 ON cs2.c_id = sc2.c_id 
	AND cs2.c_id = '02';
	
	

-- 查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息
SELECT
	st.* 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id
	INNER JOIN course cs ON cs.c_id = sc.c_id 
	AND sc.c_id = '01' 
WHERE
	st.s_id IN (
	SELECT
		st2.s_id 
	FROM
		student st2
		INNER JOIN score sc2 ON sc2.s_id = st2.s_id
		INNER JOIN course cs2 ON cs2.c_id = sc2.c_id 
		AND cs2.c_id = '02' 
	);
```


![img](https://img-blog.csdnimg.cn/b0de35338ddb41c297853e16415b799c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 方法 2

```java
SELECT
	a.* 
FROM
	student a,
	score b,
	score c 
WHERE
	a.s_id = b.s_id 
	AND a.s_id = c.s_id 
	AND b.c_id = '01' 
	AND c.c_id = '02';
```


![img](https://img-blog.csdnimg.cn/b0de35338ddb41c297853e16415b799c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 10. 查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息

```java
SELECT
	st.s_id 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id
	INNER JOIN course cs ON cs.c_id = sc.c_id 
	AND cs.c_id = '01' 
WHERE
	st.s_id NOT IN (
	SELECT
		st.s_id 
	FROM
		student st
		INNER JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course cs ON cs.c_id = sc.c_id 
		AND cs.c_id = '02' 
	);
```


![img](https://img-blog.csdnimg.cn/1512bc3e14a148a0b150fbb0a1a10e30.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 11. 查询没有学全所有课程的同学的信息

- 方法 1

```java
SELECT
	* 
FROM
	student 
WHERE
	s_id NOT IN (
	SELECT
		st.s_id 
	FROM
		student st
		INNER JOIN score sc ON sc.s_id = st.s_id 
		AND sc.c_id = '01' 
	WHERE
		st.s_id IN (
		SELECT
			st.s_id 
		FROM
			student st
			INNER JOIN score sc ON sc.s_id = st.s_id 
			AND sc.c_id = '02' 
		WHERE
			st.s_id 
		) 
		AND st.s_id IN (
		SELECT
			st.s_id 
		FROM
			student st
			INNER JOIN score sc ON sc.s_id = st.s_id 
			AND sc.c_id = '03' 
		WHERE
			st.s_id 
		) 
	);
```


![img](https://img-blog.csdnimg.cn/8041a58830fc4c408549b0ff58e83f60.png)

- 方法 2

```java
SELECT
	a.* 
FROM
	student a
	LEFT JOIN score b ON a.s_id = b.s_id 
GROUP BY
	a.s_id 
HAVING
	COUNT( b.c_id ) != '3';
```


![img](https://img-blog.csdnimg.cn/8041a58830fc4c408549b0ff58e83f60.png)

### 12. 查询至少有一门课与学号为"01"的同学所学相同的同学的信息

```java
SELECT DISTINCT
	st.* 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
WHERE
	sc.c_id IN ( SELECT sc2.c_id FROM student st2 LEFT JOIN score sc2 ON sc2.s_id = st2.s_id WHERE st2.s_id = '01' );
```


![img](https://img-blog.csdnimg.cn/93079ce0ebef4b2393d052c6e0c9ed5f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 13. 查询和"01"号的同学学习的课程完全相同的其他同学的信息

```java
SELECT
	st.* 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
GROUP BY
	st.s_id 
HAVING
	GROUP_CONCAT( sc.c_id )=(
	SELECT
		GROUP_CONCAT( sc2.c_id ) 
	FROM
		student st2
		LEFT JOIN score sc2 ON sc2.s_id = st2.s_id 
	WHERE
		st2.s_id = '01' 
	);
```


![img](https://img-blog.csdnimg.cn/71e23426fcbb4f20815aab7d415fd310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 14. 查询没学过"邪恶小法师"老师讲授的任一门课程的学生姓名

```java
SELECT
	* 
FROM
	student 
WHERE
	s_id NOT IN (
	SELECT
		sc.s_id 
	FROM
		score sc
		INNER JOIN course cs ON cs.c_id = sc.c_id
	INNER JOIN teacher t ON t.t_id = cs.t_id 
	AND t.t_name = '邪恶小法师');
```


![img](https://img-blog.csdnimg.cn/060ccf91bfea404cbef40c5991e5a1d3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 15. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```java
SELECT
	st.s_id AS '学号',
	st.s_name AS '姓名',
	AVG( sc.s_score ) AS '平均成绩' 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
WHERE
	sc.s_id IN (
	SELECT
		sc.s_id 
	FROM
		score sc 
	WHERE
		sc.s_score < 60 
		OR sc.s_score IS NULL 
	GROUP BY
		sc.s_id 
	HAVING
		COUNT( 1 )>= 2 
	) 
GROUP BY
	st.s_id
```


![img](https://img-blog.csdnimg.cn/393799bbd8e54effbfb4b9de125f6640.png)

### 16. 检索"01"课程分数小于60，按分数降序排列的学生信息

```java
SELECT
	st.* 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id 
	AND sc.c_id = '01' 
	AND sc.s_score < '60' 
ORDER BY
	sc.s_score DESC;
	
	
SELECT
	st.* 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
WHERE
	sc.c_id = '01' 
	AND sc.s_score < '60' 
ORDER BY
	sc.s_score DESC;
```


![img](https://img-blog.csdnimg.cn/83da236ada9540b9bb0d9c0237f2366e.png)

### 17. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

- 方法 1

```java
SELECT
	st.*,
	AVG( sc4.s_score ) AS '平均分',
	sc.s_score AS '语文',
	sc2.s_score AS '数学',
	sc3.s_score AS '英语' 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
	AND sc.c_id = '01'
	LEFT JOIN score sc2 ON sc2.s_id = st.s_id 
	AND sc2.c_id = '02'
	LEFT JOIN score sc3 ON sc3.s_id = st.s_id 
	AND sc3.c_id = '03'
	LEFT JOIN score sc4 ON sc4.s_id = st.s_id 
GROUP BY
	st.s_id 
ORDER BY
	AVG( sc4.s_score ) DESC;
```


![img](https://img-blog.csdnimg.cn/0b8be12869ba43d984ca7a379135bf93.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

- 方法 2

```java
SELECT
	st.*,
	( CASE WHEN AVG( sc4.s_score ) IS NULL THEN 0 ELSE AVG( sc4.s_score ) END ) AS '平均分',
	( CASE WHEN sc.s_score IS NULL THEN 0 ELSE sc.s_score END ) AS '语文',
	( CASE WHEN sc2.s_score IS NULL THEN 0 ELSE sc2.s_score END ) AS '数学',
	( CASE WHEN sc3.s_score IS NULL THEN 0 ELSE sc3.s_score END ) AS '英语' 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
	AND sc.c_id = '01'
	LEFT JOIN score sc2 ON sc2.s_id = st.s_id 
	AND sc2.c_id = '02'
	LEFT JOIN score sc3 ON sc3.s_id = st.s_id 
	AND sc3.c_id = '03'
	LEFT JOIN score sc4 ON sc4.s_id = st.s_id 
GROUP BY
	st.s_id 
ORDER BY
	AVG( sc4.s_score ) DESC;
```


![img](https://img-blog.csdnimg.cn/5688db7698f14d8d93486bb9ada52d95.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 18. 查询各科成绩最高分、最低分和平均分：

- 以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率 
- 及格为&gt;=60，中等为：70-80，优良为：80-90，优秀为：&gt;=90

```java
SELECT
	cs.c_id,
	cs.c_name,
	MAX( sc1.s_score ) AS '最高分',
	MIN( sc2.s_score ) AS '最低分',
	AVG( sc3.s_score ) AS '平均分',
	((
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			s_score >= 60 
			AND c_id = cs.c_id 
			)/(
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			c_id = cs.c_id 
		)) AS '及格率',
	((
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			s_score >= 70 
			AND s_score < 80 
			AND c_id = cs.c_id 
			)/(
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			c_id = cs.c_id 
		)) AS '中等率',
	((
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			s_score >= 80 
			AND s_score < 90 
			AND c_id = cs.c_id 
			)/(
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			c_id = cs.c_id 
		)) AS '优良率',
	((
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			s_score >= 90 
			AND c_id = cs.c_id 
			)/(
		SELECT
			COUNT( s_id ) 
		FROM
			score 
		WHERE
			c_id = cs.c_id 
		)) AS '优秀率' 
FROM
	course cs
	LEFT JOIN score sc1 ON sc1.c_id = cs.c_id
	LEFT JOIN score sc2 ON sc2.c_id = cs.c_id
	LEFT JOIN score sc3 ON sc3.c_id = cs.c_id 
GROUP BY
	cs.c_id;
```


![img](https://img-blog.csdnimg.cn/23838babb2cf47489e6e855843eac53d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 19. 按各科成绩进行排序，并显示排名(实现不完全)

- mysql没有rank函数 
- 加@score是为了防止用union all 后打乱了顺序

```java
SELECT
	c1.s_id,
	c1.c_id,
	c1.c_name,
	@score := c1.s_score,
	@i := @i + 1 
FROM
	(
	SELECT
		c.c_name,
		sc.* 
	FROM
		course c
		LEFT JOIN score sc ON sc.c_id = c.c_id 
	WHERE
		c.c_id = "01" 
	ORDER BY
		sc.s_score DESC 
	) c1,
	( SELECT @i := 0 ) a UNION ALL
SELECT
	c2.s_id,
	c2.c_id,
	c2.c_name,
	c2.s_score,
	@ii := @ii + 1 
FROM
	(
	SELECT
		c.c_name,
		sc.* 
	FROM
		course c
		LEFT JOIN score sc ON sc.c_id = c.c_id 
	WHERE
		c.c_id = "02" 
	ORDER BY
		sc.s_score DESC 
	) c2,
	( SELECT @ii := 0 ) aa UNION ALL
SELECT
	c3.s_id,
	c3.c_id,
	c3.c_name,
	c3.s_score,
	@iii := @iii + 1 
FROM
	(
	SELECT
		c.c_name,
		sc.* 
	FROM
		course c
		LEFT JOIN score sc ON sc.c_id = c.c_id 
	WHERE
		c.c_id = "03" 
	ORDER BY
		sc.s_score DESC 
	) c3;

SET @iii = 0;
```


![img](https://img-blog.csdnimg.cn/1144b16c68764a99a1703ea2dccd7a3f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 20. 查询学生的总成绩并进行排名

```java
SELECT
	st.s_id,
	st.s_name,
	( CASE WHEN sum( sc.s_score ) IS NULL THEN 0 ELSE SUM( sc.s_score ) END ) 
FROM
	student st
	LEFT JOIN score sc ON st.s_id = sc.s_id 
GROUP BY
	st.s_id 
ORDER BY
	SUM( sc.s_score ) DESC
```


![img](https://img-blog.csdnimg.cn/f7b92b0829ce4d0cab3ab9e6fa3b8d76.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 21. 查询不同老师所教不同课程平均分从高到低显示

```java
SELECT
	t.t_id,
	t.t_name,
	AVG( sc.s_score ) 
FROM
	teacher t
	LEFT JOIN course c ON c.t_id = t.t_id
	LEFT JOIN score sc ON sc.c_id = c.c_id 
GROUP BY
	t.t_id 
ORDER BY
	AVG( sc.s_score ) DESC
```


![img](https://img-blog.csdnimg.cn/6c36531204174ab79005321cb453a53d.png)

### 22. 查询所有课程的成绩第2名到第3名的学生信息及该课程成绩

```java
SELECT
	a.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_id,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON sc.c_id = c.c_id 
		AND c.c_id = '01' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 1,
		2 
	) a UNION ALL
SELECT
	b.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_id,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '02' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 1,
		2 
	) b UNION ALL
SELECT
	c.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_id,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '03' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 1,
		2 
	) c;
```


![img](https://img-blog.csdnimg.cn/c8e471e4980842388c0b8a9daa316f41.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 23. 统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比

```java
SELECT
	c.c_id,
	c.c_name,
	(
	SELECT
		COUNT( 1 ) 
	FROM
		score sc 
	WHERE
		sc.c_id = c.c_id 
		AND sc.s_score <= 100 AND sc.s_score > 80 
		)/(
	SELECT
		COUNT( 1 ) 
	FROM
		score sc 
	WHERE
		sc.c_id = c.c_id 
	) AS '100-85',
	((
		SELECT
			COUNT( 1 ) 
		FROM
			score sc 
		WHERE
			sc.c_id = c.c_id 
			AND sc.s_score <= 85 AND sc.s_score > 70 
			)/(
		SELECT
			COUNT( 1 ) 
		FROM
			score sc 
		WHERE
			sc.c_id = c.c_id 
		)) AS '85-70',
	((
		SELECT
			COUNT( 1 ) 
		FROM
			score sc 
		WHERE
			sc.c_id = c.c_id 
			AND sc.s_score <= 70 AND sc.s_score > 60 
			)/(
		SELECT
			COUNT( 1 ) 
		FROM
			score sc 
		WHERE
			sc.c_id = c.c_id 
		)) AS '70-60',
	((
		SELECT
			COUNT( 1 ) 
		FROM
			score sc 
		WHERE
			sc.c_id = c.c_id 
			AND sc.s_score <= 60 AND sc.s_score >= 0 
			)/(
		SELECT
			COUNT( 1 ) 
		FROM
			score sc 
		WHERE
			sc.c_id = c.c_id 
		)) AS '85-70' 
FROM
	course c 
ORDER BY
	c.c_id
```


![img](https://img-blog.csdnimg.cn/1a3891a301414e599518b497f00fe812.png)

### 24. 查询学生平均成绩及其名次

```java
SET @i = 0;
SELECT
	a.*,
	@i := @i + 1 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		round( CASE WHEN AVG( sc.s_score ) IS NULL THEN 0 ELSE AVG( sc.s_score ) END, 2 ) AS agvScore 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id 
	GROUP BY
		st.s_id 
	ORDER BY
		agvScore DESC 
	) a
```


![img](https://img-blog.csdnimg.cn/3b4a8d71fad841ff8d3c61a2218124c5.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 25. 查询各科成绩前三名的记录

```java
SELECT
	a.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_id,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '01' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 0,
		3 
	) a UNION ALL
SELECT
	b.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_id,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '02' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 0,
		3 
	) b UNION ALL
SELECT
	c.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_id,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '03' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 0,
		3 
	) c
```


![img](https://img-blog.csdnimg.cn/077fa80313d644949d94c7f056da8213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 26. 查询每门课程被选修的学生数

```java
SELECT
	c.c_id,
	c.c_name,
	COUNT( 1 ) 
FROM
	course c
	LEFT JOIN score sc ON sc.c_id = c.c_id
	INNER JOIN student st ON st.s_id = c.c_id 
GROUP BY
	c.c_id
```


![img](https://img-blog.csdnimg.cn/72b5920a123c4cadb5f5d80a9466796e.png)

### 27. 查询出只有两门课程的全部学生的学号和姓名

```java
SELECT
	st.s_id,
	st.s_name 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id
	INNER JOIN course c ON c.c_id = sc.c_id 
GROUP BY
	st.s_id 
HAVING
	COUNT( 1 ) = 2
```


![img](https://img-blog.csdnimg.cn/2017f196335f45c4adaef13d14898ce7.png)

### 28. 查询男生、女生人数

```java
SELECT s_sex, COUNT(1) FROM student GROUP BY s_sex
```


![img](https://img-blog.csdnimg.cn/4ff2b38904894a929f6dcd382bc80d0f.png)

### 29. 查询名字中含有"德"字的学生信息

```java
SELECT * FROM student WHERE s_name LIKE '%德%'
```


![img](https://img-blog.csdnimg.cn/32ec816e45cb47f08d55a9d16d41d2d7.png)

### 30. 查询同名同性学生名单，并统计同名人数

```java
select st.s_name,st.s_sex,count(1) from student st group by st.s_name,st.s_sex having count(1)>1
```


![img](https://img-blog.csdnimg.cn/47093ceb41164544848e975a1185263b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 31. 查询1990年出生的学生名单

```java
SELECT st.* FROM student st WHERE st.s_birth LIKE '1990%';
```


![img](https://img-blog.csdnimg.cn/28e1a946195b46d9a19b3fadacb30675.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 32. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列

```java
SELECT
	c.c_id,
	c_name,
	AVG( sc.s_score ) AS scoreAvg 
FROM
	course c
	INNER JOIN score sc ON sc.c_id = c.c_id 
GROUP BY
	c.c_id 
ORDER BY
	scoreAvg DESC,
	c.c_id ASC;
```


![img](https://img-blog.csdnimg.cn/24c485125f0449338393c344620d9e81.png)

### 33. 查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩

```java
SELECT
	st.s_id,
	st.s_name,
	( CASE WHEN AVG( sc.s_score ) IS NULL THEN 0 ELSE AVG( sc.s_score ) END ) scoreAvg 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
GROUP BY
	st.s_id 
HAVING
	scoreAvg > '85';
```


![img](https://img-blog.csdnimg.cn/b7278687b5794e7787108972a7fff0cb.png)

### 34. 查询课程名称为"数学"，且分数低于60的学生姓名和分数

```java
SELECT
	* 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id 
	AND sc.s_score < 60
	INNER JOIN course c ON c.c_id = sc.c_id 
	AND c.c_name = '数学';
```


![img](https://img-blog.csdnimg.cn/abcd5d870ef844258756ac00a1105220.png)

### 35. 查询所有学生的课程及分数情况

```java
SELECT
	* 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id
	LEFT JOIN course c ON c.c_id = sc.c_id 
ORDER BY
	st.s_id,
	c.c_name;
```


![img](https://img-blog.csdnimg.cn/916e023816ed42ea841bf385e7400342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 36. 查询任何一门课程成绩在70分以上的姓名、课程名称和分数

```java
SELECT
	st.s_id,st.s_name,c.c_name,sc.s_score 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id
	LEFT JOIN course c ON c.c_id = sc.c_id 
WHERE
	st.s_id IN (
	SELECT
		st2.s_id 
	FROM
		student st2
		LEFT JOIN score sc2 ON sc2.s_id = st2.s_id 
	GROUP BY
		st2.s_id 
	HAVING
		MIN( sc2.s_score )>= 70 
	ORDER BY
	st2.s_id 
	)
```


![img](https://img-blog.csdnimg.cn/74de22c390cd43938014f19286fbaa66.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 37. 查询不及格的课程

```java
SELECT
	st.s_id,
	c.c_name,
	st.s_name,
	sc.s_score 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id 
	AND sc.s_score < 60
	INNER JOIN course c ON c.c_id = sc.c_id
```


![img](https://img-blog.csdnimg.cn/fba95ee750ed45929c77c000cb710a74.png)

### 38. 查询课程编号为01且课程成绩在80分以上的学生的学号和姓名

```java
SELECT
	st.s_id,
	st.s_name,
	sc.s_score 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id 
	AND sc.c_id = '01' 
	AND sc.s_score >= 80;
```


![img](https://img-blog.csdnimg.cn/4422f73d076b4feba28ec86078eb1057.png)

### 39. 求每门课程的学生人数

```java
SELECT
	c.c_id,
	c.c_name,
	COUNT( 1 ) 
FROM
	course c
	INNER JOIN score sc ON sc.c_id = c.c_id 
GROUP BY
	c.c_id;
```


![img](https://img-blog.csdnimg.cn/e58272ef494540dd8ed04aec0a5c03c4.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 40. 查询选修"死亡歌颂者"老师所授课程的学生中，成绩最高的学生信息及其成绩

```java
SELECT
	st.*,
	sc.s_score 
FROM
	student st
	INNER JOIN score sc ON sc.s_id = st.s_id
	INNER JOIN course c ON c.c_id = sc.c_id
	INNER JOIN teacher t ON t.t_id = c.t_id 
	AND t.t_name = '死亡歌颂者' 
ORDER BY
	sc.s_score DESC 
	LIMIT 0,1;
```


![img](https://img-blog.csdnimg.cn/8ca25507a1c043a9bcb2b2fb3407f665.png)

### 41. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

```java
SELECT
	st.s_id,
	st.s_name,
	sc.c_id,
	sc.s_score 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id
	LEFT JOIN course c ON c.c_id = sc.c_id 
WHERE
	(
	SELECT
		COUNT( 1 ) 
	FROM
		student st2
		LEFT JOIN score sc2 ON sc2.s_id = st2.s_id
		LEFT JOIN course c2 ON c2.c_id = sc2.c_id 
	WHERE
		sc.s_score = sc2.s_score 
	AND c.c_id != c2.c_id 
	)>1;
```


![img](https://img-blog.csdnimg.cn/2f8bc9d664b24e92a7de116eda95ffd1.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 42. 查询每门功成绩最好的前两名

```java
SELECT
	a.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '01' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 0,
		2 
	) a UNION ALL
SELECT
	b.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '02' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 0,
		2 
	) b UNION ALL
SELECT
	c.* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c.c_name,
		sc.s_score 
	FROM
		student st
		LEFT JOIN score sc ON sc.s_id = st.s_id
		INNER JOIN course c ON c.c_id = sc.c_id 
		AND c.c_id = '03' 
	ORDER BY
		sc.s_score DESC 
		LIMIT 0,
	2 
	) c;
```


![img](https://img-blog.csdnimg.cn/effd88d642f54893a2fae58b6900ae1a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

写法 2

```java
SELECT
	a.s_id,
	a.c_id,
	a.s_score 
FROM
	score a 
WHERE
	( SELECT COUNT( 1 ) FROM score b WHERE b.c_id = a.c_id AND b.s_score > a.s_score ) <= 2 
ORDER BY
	a.c_id;
```


![img](https://img-blog.csdnimg.cn/d8be49657bfa4828b57896875af9dee4.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 43. 统计每门课程的学生选修人数（超过5人的课程才统计）

- 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```java
SELECT
	c.c_id,
	COUNT( 1 ) 
FROM
	score sc
	LEFT JOIN course c ON c.c_id = sc.c_id 
GROUP BY
	c.c_id 
HAVING
	COUNT( 1 ) > 5 
ORDER BY
	COUNT( 1 ) DESC,
	c.c_id ASC;
```


![img](https://img-blog.csdnimg.cn/7baa3c0dc97a436c8d77cd0e3c5830ab.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 44. 检索至少选修两门课程的学生学号

```java
SELECT
	st.s_id 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
GROUP BY
	st.s_id 
HAVING
	COUNT( 1 )>= 2;
```


![img](https://img-blog.csdnimg.cn/0492f350435f4b98a2e4d51aba6a3d5f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 45. 查询选修了全部课程的学生信息

```java
SELECT
	st.* 
FROM
	student st
	LEFT JOIN score sc ON sc.s_id = st.s_id 
GROUP BY
	st.s_id 
HAVING
	COUNT( 1 )=(
	SELECT
		COUNT( 1 ) 
FROM
	course)
```


![img](https://img-blog.csdnimg.cn/9a3a8bef835648bd9ba9bf1eff27097f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 46. 查询各学生的年龄

```java
SELECT
	st.*,
	TIMESTAMPDIFF(
		YEAR,
		st.s_birth,
	NOW()) 
FROM
	student st
```


![img](https://img-blog.csdnimg.cn/ce1c628556a34ed9a98aa9304ff82e68.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

### 47. 查询本周过生日的学生

```java
SELECT
	st.* 
FROM
	student st 
WHERE
	WEEK (
	NOW())+ 1 = WEEK (
	DATE_FORMAT( st.s_birth, '%Y%m%d' ))
```


![img](https://img-blog.csdnimg.cn/b7a1997fbe954fce98050434c956fa73.png)

### 48. 查询下周过生日的学生

```java
SELECT
	st.* 
FROM
	student st 
WHERE
	WEEK (
		NOW())+ 1 = WEEK (
	DATE_FORMAT( st.s_birth, '%Y%m%d' ));
```


![img](https://img-blog.csdnimg.cn/d62ab74216b44a969917970fcaed62a6.png)

### 49. 查询本月过生日的学生

```java
SELECT
	st.* 
FROM
	student st 
WHERE
	MONTH (
	NOW())= MONTH (
	DATE_FORMAT( st.s_birth, '%Y%m%d' ));
```


![img](https://img-blog.csdnimg.cn/2d5d90ae468b4d00a026627ea5f3891d.png)

### 50. 查询下月过生日的学生

```java
SELECT
	st.* 
FROM
	student st 
WHERE
	MONTH (
		TIMESTAMPADD(
			MONTH,
			1,
		NOW()))= MONTH (
	DATE_FORMAT( st.s_birth, '%Y%m%d' ));
```


![img](https://img-blog.csdnimg.cn/b885e7bc8d4e4ffe86af68ad7afa6bc0.png)




![img](https://img-blog.csdnimg.cn/f29529ffbb374ff0800ba28a06f59884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTY5MjcwNQ==,size_16,color_FFFFFF,t_70)

**点击预览在线版: 阿里巴巴开发手册**


**内容偏向基础适合各个阶段人员的学习与巩固，如果对您还有些帮助希望给博主点个赞支持一下，感谢！**

