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

## 表

1. 创建表

   `CREATE TABLE table_name (field_name1 datatype, field_name2 datatype, ... )`

   - `CHARACTER SET charset_name` - 指定表采用的字符集，默认为所在数据库的字符集
   - `COLLATE collation_name` - 指定表字符集的校对规则，默认为所在数据库的字符集校对规则
   - `ENGINE engine_name` - 指定存储引擎

2. 修改表

3. 删除表

## 字段数据类型

1. 数值类型

   | 类型            | 说明                                 |
   | --------------- | ------------------------------------ |
   | `TINYINT`       | 1个字节                              |
   | `SMALLINT`      | 2个字节                              |
   | `MEDIUMINT`     | 3个字节                              |
   | `INT`           | 4个字节                              |
   | `BIGINT`        | 8个字节                              |
   | `FLOAT`         | 4个字节                              |
   | `DOUBLE`        | 8个字节                              |
   | `DECIMAL(M, D)` | 定点数，M指定长度，D指定小数点的位数 |

2. 文本类型

3. 二进制类型

4. 时间日期类型

时间日期类型

## 增删改查

## 函数

## 内连接

## 外连接