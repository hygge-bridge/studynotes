# mysql介绍

#### mysql特点

1. 关系型数据库。
2. 基于客户端服务器(C/S)模型。
3. 数据被存储为一个二维表，一行数据被称为记录，列被称为字段或属性。因为是一行一行存储数据，所以mysql又被称为行式数据库。
4. 支持插件式的存储引擎，常见的存储引擎有innodb、myisam、memory等。
5. 服务器模型采用select+可伸缩的线程池来实现网络服务器。

**mysql使用select而不是epoll的原因**：

因为mysql存储数据会涉及磁盘io，速度较慢。此时就算通过epoll来较快的接收网络io，也无法将其快速存储到磁盘。所以接收网络io没有必要过快，只需要将网络io和磁盘io的速度相匹配即可。

#### EDBMS、RDB、table介绍

- mysql其实是关系型数据库管理系统（EDBMS)
- mysql中可以创建许多的关系型数据库(RDB)
- 一个RDB中可以创建多个表(table)---二维表

如下所示，chat其实就是一个RDB，user就是一个表。我们一般说mysql是数据库只是大家的习惯叫法罢了，真正的数据库其实是在mysql里面创建的才叫数据库，而mysql其实是关系型数据库管理系统。

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| chat               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> use chat;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_chat |
+----------------+
| allgroup       |
| friend         |
| groupuser      |
| offlinemessage |
| user           |
+----------------+
5 rows in set (0.00 sec)
```

#### 配置目录介绍

mysql的默认目录在`/var/lib/mysql`

配置文件就在`/etc/my.cnf`，如果没有可以自行创建，只需要在`[mysqld]`下面写配置信息即可。如下为设置mysql的默认编码格式和默认存储引擎：

```mysql
[mysqld]

character-set-server=utf8
default-storage-engine=INNODB
```

# 数据类型

合理定义每个字段的数据类型，可以有效减少数据的存储大小，所以定义类型前必须考虑仔细！

#### 数值类型

![image-20240720215010811](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240720215010811.png)

在使用浮点类型时需要注意精度问题，对于高精度要求建议使用decimal。

- 对于float和double类型，如果发生了数据溢出那么会直接导致数据被截断。但是对于decimal类型，会直接报错。
- float的精度通常由6-7个有效位，double是15-16个有效位，而decimal有28个有效位。

#### 字符串类型

![image-20240720215439268](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240720215439268.png)

**char vs varchar：**char是固定长度，varchar是可变长度的字符串类型。

**类型后括号的数字：**

- 字符串类型后括号的数字表示字符数（字节数）--- 一个字符表示一个字节
- 数值类型后括号的数字表示显示的宽度

```mysql
# 即使存入"hello",仍然会用12个字节来进行存储。如果字符串超过12字节，那么会发生截断
CHAR(12) VARCHAR(12)

# 无论插入什么数字，都只会使用5个字节来进行存储。
# 当数值字符长度小于指定显示宽度时，会用空格或0填充，如插入2那么可能会被填充为00002
# 当数值字符长度大于指定显示宽度时，不会截断而是正常显示，如插入123456那么会正常显示123456
INT(5)
```

ps：mysql中字符串用单引号括起来，从而保持字面值。

#### 日期和时间类型

![image-20240720220243299](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240720220243299.png)

ps：TIMESTAMP会自动更新时间，而DATETIME需要手动更新时间。

实际项目中使用这个类型的次数不多，主要是利用`now()`和`unix_timestamp()`来设置时间。直接把时间戳进行存储方便得多，然后在代码中转换为日期就好了。

```mysql
mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2024-07-20 22:04:53 |
+---------------------+
1 row in set (0.00 sec)

mysql> select unix_timestamp(now());
+-----------------------+
| unix_timestamp(now()) |
+-----------------------+
|            1721484308 |
+-----------------------+
1 row in set (0.00 sec)
```

#### enum和set

限制字段只能取固定的值。不同点在于enum只能取唯一一个值，而set可以取任意多个值。

比如性别字段，就可以设置为enum，因为全世界的人也只有男女两个性别。（不过好像美丽国有几十种性别😢）

# 运算符

#### 算术运算符

![image-20240720220833880](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240720220833880.png)

举例：将用户的年龄加1`update user set age=age+1;`

#### 逻辑运算符

建议使用mysql定义的`NOT AND OR`，而不是`! && ||`。

![image-20240720220847759](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240720220847759.png)

举例：查询性别为男且成绩大于90分的学生`select * from student where sex='M' and score>='90.0';`

#### 比较运算符

![image-20240720220947557](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240720220947557.png)

如果字段的完整性约束没有设置为`not null`，那么该字段的取值有可能为null。如果要判断其值是否为null，必须使用`is (not) null`，而不能使用`== null`。

使用like时需要注意是否会用到索引。假设字段具有索引，如果通配符在字符串的中间或末尾那么可以用到，如果通配符在字符串的前面那么无法用到索引。

# 完整性约束

一共有6大约数。

#### 主键约束(primary key)

不可以取空值(null)，不可以重复，且一个表中只能有一个主键。

innodb要求表必须具有主键，如果没有主键，那么它会默认添加一列整型字段作为主键，当然这和它的索引底层实现有关，后续介绍。

#### 自增键约束(auto_increment)

大部分的整型主键都会设置为自增键，从而可以不用手动设置主键id，因为mysql会自动地顺序生成主键。

#### 唯一键约束(unique)

可以取空值(null)，不可以重复，且一个表中可以有多个唯一键。（和主键对比）

#### 默认值约束(default)

主要是用于添加一个默认值。

#### 外键约束(foreign key)

外键的作用是让多个表的字段产生关联。

假设有一张学生表和学生考试表，有一天学生信息被移除出学生表了，但是学生考试表中仍然有该学生，那么这个数据就是脏数据，这会导致资源的浪费。外键的作用就类似于告诉用户这个数据还关联了一些其他表，你需要把其他表的数据也删除掉才可以。

现代项目开发中，外键、存储函数、存储过程、触发器等几乎不会再使用了。由于这些限制逻辑操作不应交给mysql处理，mysql会涉及磁盘io是很有可能达到性能瓶颈的，所以我们应该让mysql尽可能处理核心操作（增删查改）。比如这些表与表的关系处理交给业务代码，从而减轻mysql的工作压力。

#### 举例

```mysql
mysql> CREATE TABLE user(
    -> id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT'用户主键id',
    -> age TINYINT UNSIGNED NOT NULL UNIQUE DEFAULT 18 COMMENT'用户年龄默认为18',
    -> sex ENUM('male', 'famale') );
Query OK, 0 rows affected (0.03 sec)

mysql> desc user;
+-------+-----------------------+------+-----+---------+----------------+
| Field | Type                  | Null | Key | Default | Extra          |
+-------+-----------------------+------+-----+---------+----------------+
| id    | int unsigned          | NO   | PRI | NULL    | auto_increment |
| age   | tinyint unsigned      | NO   | UNI | 18      |                |
| sex   | enum('male','famale') | YES  |     | NULL    |                |
+-------+-----------------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)

mysql> CREATE TABLE user(
    -> id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT'用户主键id',
    -> age TINYINT UNSIGNED NOT NULL UNIQUE DEFAULT 18 COMMENT'用户年龄默认为18',
    -> sex ENUM('male', 'famale') );
Query OK, 0 rows affected (0.03 sec)

mysql> desc user\G
*************************** 1. row ***************************
  Field: id
   Type: int unsigned
   Null: NO
    Key: PRI
Default: NULL
  Extra: auto_increment
*************************** 2. row ***************************
  Field: age
   Type: tinyint unsigned
   Null: NO
    Key: UNI
Default: 18
  Extra: 
*************************** 3. row ***************************
  Field: sex
   Type: enum('male','famale')
   Null: YES
    Key: 
Default: NULL
  Extra: 
3 rows in set (0.00 sec)

```

ps：mysql中`:`表示以表示形式显示结果，'\G'表示以一行一行的形式显示

# 表的设计原则

在进行项目开发前，都会进行需求分析并进行场景设计，其中表的设计就是不可或缺的部分。至少需要明确的是：

1. 有哪些实体需要被抽象成表？
2. 表中有哪些字段？
3. 字段应该是什么类型？
4. 表与表之间的关系是什么样的？

表与表之间的关系有三种---一对一，一对多和多对多。

#### 一对一

假设有如下两个一对一关系的表。

用户表(User)：

| uid  | name  | age  |
| ---- | ----- | ---- |
| 1001 | tom   | 22   |
| 1002 | hery  | 11   |
| 2000 | helen | 33   |

身份信息表(Info)：

| cardid    | addr     |
| --------- | -------- |
| 111222333 | chengdu  |
| 821734692 | shanghai |
| 121412344 | beijing  |

假如两个表之间是如上的设计，那么在进行查询时，我们是无法获取到用户的身份信息的。解决这种一对一关系的表的方法就是在子表中添加一列来关联父表的主键。

此时子表就是Info，父表就是User，通过在Info中新添加一列uid就可以解决上述问题。比如此时可以用sql语句来查询某人的身份信息`select * from Info where uid = 1001`。子表中新添加的这列就是外键，此列名可以和父表的不同但是数据类型必须相同(建议列名还是相同)。

| uid  | cardid    | addr     |
| ---- | --------- | -------- |
| 1001 | 111222333 | chengdu  |
| 1002 | 821734692 | shanghai |
| 2000 | 121412344 | beijing  |

故一对一的关联方式为---**在子表中添加一列，用于关联父表的主键**。

#### 一对多

假如此时需要做一个电商系统，有三个实体。用户、商品和订单。显而易见的是，用户和商品是没有关系的，用户和订单是一对多的关系，商品和订单是多对多的关系。表的设计如下：

用户表(User):

| uid  | name  | age  |
| ---- | ----- | ---- |
| 1001 | tom   | 22   |
| 1002 | hery  | 11   |
| 2000 | helen | 33   |

商品表(Product)：

| pid  | pname  | price |
| ---- | ------ | ----- |
| 1    | 手机   | 5000  |
| 2    | 笔记本 | 10000 |
| 3    | 手表   | 100   |

订单表(Order)：

| orderid | pid  | number | money | totalprice | addr     |
| ------- | ---- | ------ | ----- | ---------- | -------- |
| O1000   | 1    | 1      | 600   | 1000       | chengdu  |
| O1000   | 3    | 4      | 400   | 1000       | chengdu  |
| O2000   | 2    | 2      | 20000 | 20000      | shanghai |

此时没有办法查询到用户的订单信息，所以需要添加一列uid作为外键。

| orderid | uid  | pid  | number | money | totalprice | addr     |
| ------- | ---- | ---- | ------ | ----- | ---------- | -------- |
| O1000   | 1001 | 1    | 1      | 600   | 1000       | chengdu  |
| O1000   | 1001 | 3    | 4      | 400   | 1000       | chengdu  |
| O2000   | 2000 | 2    | 2      | 20000 | 20000      | shanghai |

故一对多的关联方式为---**添加外键**(和一对一相同)

#### 多对多

观察上诉电商系统的三个表，可以发现，订单表中有很多数据都冗余了，比如totalprice、addr、orderid、uid。假如此时商品价格出现变化，那么我的订单表也会大面积的修改。

此时需要添加一个中间表，从而避免数据冗余存储。添加一个订单内容表(OrderList)：

| orderid | pid  | number | money |
| ------- | ---- | ------ | ----- |
| O1000   | 1    | 1      | 6000  |
| O1000   | 3    | 4      | 400   |
| O2000   | 2    | 2      | 20000 |

订单表如下:

| orderid | uid  | totalprice | addr     |
| ------- | ---- | ---------- | -------- |
| O1000   | 1001 | 1000       | chengdu  |
| O2000   | 2000 | 20000      | shanghai |

此时订单表的冗余问题得到了很多的解决。注意订单内容表可以使用orderid和pid作为联合主键，因为单独一个都可能重复，但是联合起来就不可能重复了。

故多对多的关联方式为---**添加中间表**。

# 范式设计

范式设计其实和表的设计原则同一个目的，都是为了减少数据的冗余。

#### 第一范式

每一列都必须保持原子性。比如地址字段可以再细分为省、市、区等，最后的字段必须是不可以再分割的字段。

第一范式都不满足的数据库就不叫关系型数据库。因为假如字段不保持原子性，那么就是kv数据库了，因为值可以存放一个字符串，比如json字符串，然后把所有的字段全部包含到这个json字符串里面。

#### 第二范式

非主属性完全依赖于主键---针对联合主键。也就是说非主键必须依赖联合主键的所有字段。

比如有个表中字段有学号、姓名、课程、学分。联合主键为学号和课程，但是很明显学分不依赖与学号，所以这就不满足第二范式。这也会导致数据的冗余，同一个表里学分都是一样的，假如此时需要修改学分，修改到一半失败了，就会导致不同同学选修的同一门课程但是学分不一样的情况。

其实根据上诉表的设计原则，学生表和课程表是多对多关系，添加一个中间表就好了。

#### 第三范式

表中的属性不能依赖于其他非主属性。也就是说表的字段必须依赖于主键。

比如有个学生表，字段有学号、姓名、学院、学院地点、学院电话。主键为学号，但是学院地点显然依赖于学院，所以这不满足第三范式。解决方法就是拆表，把学生表和学员表拆分开。

#### 第四范式

消除表中的多值依赖---多值依赖就是说多个值可能表示的是一个结果。

比如有个技能字段，java和JAVA表示同一个意思，但是值的表现形式不一样。解决方案就是把技能字段单独拆分到一个表里面，这样查询的时候我是通过技能表的外键去查询的，就不用担心值的表现形式不一样了。

#### BC范式

每一个表只有一个候选键---候选键就是一个数据库中每行的值都不同。

比如用户有个email字段，虽然这是候选键，但是没有必要单独拆分成一个表。

#### 范式的优缺点

优点：减少数据的冗余存储。冗余存储会导致增删查改的时候会对多条相同的数据进行修改，可能会出问题的（比如修改到一半失败了，那么就会导致数据不一致）

缺点：

- 范式越高，表越多
- 查询时需要连接多个表，增加了sql查询的复杂度，降低了查询效率。

项目开发经验：

一般情况下，满足第三范式就够了，不然表太多了，会导致sql查询太复杂且效率底下。而且有时为了保证查询效率，会故意保留一些数据冗余。