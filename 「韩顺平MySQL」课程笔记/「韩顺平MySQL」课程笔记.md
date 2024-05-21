# 「韩顺平MySQL」课程笔记

## MySQL

### 简介

- MySQL是一种关系型数据库管理系统。
- 数据库管理系统包含多个数据库，每个数据库中包含多张表。
- MySQL中的一个普通表本质上是一个文件。

### 安装与配置

- 启动服务 - `net start mysql`
- 停止服务 - `net stop mysql`
- 连接到MySQL服务器进程 -  `mysql -u root -p [-h 主机IP] [-P 端口号]`

## 数据库操作

- 创建数据库 - `CREATE DATABASE [IF NOT EXISTS] db_name`
  - `CHARACTER SET charset_name` - 指定数据库采用的字符集，默认为`utf8`。
  - `COLLATE collation_name` - 指定数据库字符集的校对规则，常用的是`utf8_bin`和`utf8_general_ci`。
- 显示现有的数据库 - `SHOW DATABASES`
- 删除数据库 - `DROP DATABASE [IF EXISTS] db_name`
- 

## 表

## 数据类型

## 增删改查

## 函数

## 内连接

## 外连接