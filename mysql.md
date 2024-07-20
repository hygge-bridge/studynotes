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