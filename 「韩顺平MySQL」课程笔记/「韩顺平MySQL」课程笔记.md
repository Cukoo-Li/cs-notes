# 「韩顺平MySQL」课程笔记

## MySQL

### 简介

- MySQL是一种关系型数据库管理系统。
- 数据库管理系统包含多个数据库，每个数据库中包含多张表。
- MySQL中的一个普通表本质上是一个文件。

### 相关终端命令

- 启动服务 - `net start mysql`

- 停止服务 - `net stop mysql`

- 连接到MySQL服务器进程 -  `mysql -u root -p [-h 主机IP] [-P 端口号]`

- 备份数据库 - `mysqldump -u root -p -B db_name  ... > xxx.sql`

  > 恢复数据库 - 在MySQL终端中执行`source xxx.sql`

## 数据库操作

1. 创建数据库

    `CREATE DATABASE [IF NOT EXISTS] db_name`

   - `CHARACTER SET charset_name` - 指定数据库采用的字符集，默认为`utf8`。
   - `COLLATE collation_name` - 指定数据库字符集的校对规则，常用的是`utf8_bin`和`utf8_general_ci`。

2. 删除数据库

    `DROP DATABASE [IF EXISTS] db_name`

3. 显示现有的数据库

   `SHOW DATABASES`

## 表操作

1. 创建表

   `CREATE TABLE table_name (field_name datatype, ... )`

   - `CHARACTER SET charset_name` - 指定表采用的字符集，默认为所在数据库的字符集
   - `COLLATE collation_name` - 指定表字符集的校对规则，默认为所在数据库的字符集校对规则
   - `ENGINE engine_name` - 指定存储引擎

2. 修改表

   - `ALTER TABLE table_name ADD (field_name datatype [NOT NULL] [DEFAULT expr], ...)` - 添加字段
   - `ALTER TABLE table_name MODIFY (field_name datatype [NOT NULL] [DEFAULT expr], ...)` - 修改字段
   - `ALTER TABLE table_name DROP (field_name, ...)` - 删除字段

3. 删除表

## 字段数据类型

> 常用的有`int`、`double`、`decimal`、`char`、`varchar`、`text`。

1. 数值类型

   | 类型            | 说明                                                        |
   | --------------- | ----------------------------------------------------------- |
   | `tinyint`       | 1个字节                                                     |
   | `smallint`      | 2个字节                                                     |
   | `mediumint`     | 3个字节                                                     |
   | `int`           | 4个字节                                                     |
   | `bigint`        | 8个字节                                                     |
   | `float`         | 4个字节                                                     |
   | `double`        | 8个字节                                                     |
   | `decimal(M, D)` | 定点数，M指定长度（默认为10），D指定小数点的位数（默认为0） |
   | `bit(n)`        | n个字节，以二进制数形式显示                                 |

2. 文本类型

   | 类型         | 说明                                                         |
   | ------------ | ------------------------------------------------------------ |
   | `char(n)`    | 定长字符串，固定占用`n`个字符的空间，`n`最大为255            |
   | `varchar(n)` | 变长字符串，最多存放`n`个字符，占用空间由实际存放的字符数和头部字节决定<br />最大为65532个字节，在 utf8编码下`n`最大为21844（头部1-3个字节用于记录大小） |
   | `text`       | 2个字节                                                      |
   | `longtext`   | 4个字节                                                      |

3. 二进制类型

   | 类型       | 说明    |
   | ---------- | ------- |
   | `blob`     | 2个字节 |
   | `longblob` | 4个字节 |

4. 时间日期类型

   | 类型        | 说明         |
   | ----------- | ------------ |
   | `date`      | 年月日       |
   | `time`      | 时分秒       |
   | `datetime`  | 年月日时分秒 |
   | `timestamp` | 时间戳       |
   | `year`      | 年           |

## 增删改查

## 函数

## 内连接

## 外连接