---
title: "MySQL基础"
date: 2017-10-13T14:54:28+08:00
lastmod: 2021-02-24T14:54:28+08:00
draft: false
tags: ["mysql", "sql"]
categories: ["language", "database"]
---

MySQL是最流行的关系型数据库管理系统，本文主要介绍MySQL的基本知识。
<!--more-->

另外SQL是结构化查询语言(Structured Query Language)，适用于多种数据库。\
本文示例主要采用标准SQL，尽量不涉及非标准的SQL。

# 安装

在 [官网下载地址](https://dev.mysql.com/downloads/mysql/)
根据你的系统选择社区版下载、安装即可。
```shell
# centos 8
yum install -y mysql-server

# 启动mysql服务
systemctl start mysqld
systemctl status mysqld

# 开机启动
systemctl enable mysqld

# mysql_secure_installation
```

# 连接

通过以下命令可以连接到MySQL数据库。

``` shell
mysql -u root -p
```

如果连接不成功，可能是MySQL服务还没启动。

# 数据库创建/删除

## 创建

CREATE DATABASE 用于创建数据库。

``` sql
CREATE DATABASE database_name;
```

比如说建立名为my_db的数据库并切换：

``` sql
CREATE DATABASE my_db;

USE my_db;
```

## 删除

DROP语句可以删除数据库。

``` sql
DROP DATABASE database_name;
```

# 表的创建/删除

## 创建

CREATE TABLE 语句用于创建数据库中的表。

``` sql
CREATE TABLE table_name
    (
    column1_name column2_type,
    column2_name column2_type,
    ...
    );
```

比如说建立名为Persons的表：

``` sql
CREATE TABLE Persons
    (
    Id        int,
    LastName  varchar(255),
    FirstName varchar(255),
    Address   varchar(255)
    );
```

另外通过 `SHOW CREATE TABLE` 命令可以显示创建表的语句。  

## 删除

DROP TABLE语句可用于删除表。

``` sql
DROP TABLE table_name;
```

# 插入数据

INSERT INTO语句可用于向表中插入新的一行。

``` sql
INSERT INTO table_name (column1, column2, ..., columnN)
    VALUES (value1, value2, ..., valueN);
```

比如说在Persons表插入新的一行：

``` sql
INSERT INTO Persons VALUES (1, 'Wang', 'Xiaoming', 'Xixiang Avenue');
```

又或者只在指定列中插入数据(其他列数据将为空)：

``` sql
INSERT INTO Persons (LastName, FirstName, Address)
    VALUES ('Li', 'Daren', 'Xixiang Avenue');
```

# 查询数据

SELECT 语句用于从表中选取数据。\
其中 `SELECT *` 代表选取所有列。

``` sql
SELECT columnA_name,columnB_name,... FROM table_name;
SELECT * FROM table_name;
```

比如说从Persons表选取LastName和FirstName两列的结果：

``` sql
SELECT LastName,FirstName FROM Persons;
```

又或者从Persons表选取所有列的结果：

``` sql
SELECT * FROM Persons;
```
另外在字段比较多的时候，可以加上 `\G` 按列显示。  

# 修改数据

UPDATE语句用于修改表中的数据，一般与WHERE子句配合使用。

``` sql
UPDATE table_name SET columnA=newValueA, columnB=newValueB, ...;
```

比如说修改王小明的地址：

``` sql
UPDATE Persons SET Address='No.5 Avenue'
    WHERE LastName='Wang' AND FirstName='Xiaoming';
```

# 删除数据

DELETE 语句用于删除表中的行，一般与WHERE子句配合使用。

``` sql
DELETE FROM table_name [WHERE Clause];
```

比如说从Persons表中删除王小明的数据：

``` sql
DELETE FROM Persons WHERE LastName='Wang' AND FirstName='Xiaoming';
```

# WHERE

如需有条件地从表中选取数据进行操作(查询、修改、删除等)，可将WHERE子句添加到对应SQL语句。
比如说从Persons表中删除王小明的数据：

``` sql
DELETE FROM Persons WHERE LastName='Wang' AND FirstName='Xiaoming';
```

# AND、OR

AND和OR可在WHERE子语句中把多个条件结合起来进行逻辑运算。\
AND是逻辑与运算，需要所有条件都成立。\
OR是逻辑或运算，只需要其中一个条件成立即可。\
比如说从Persons表中删除王小明的数据：

``` sql
DELETE FROM Persons WHERE LastName='Wang' AND FirstName='Xiaoming';
```

# ORDER BY

ORDER BY语句可用于根据指定的列对结果集进行排序。\
ORDER BY语句默认排序为升序。\
如果您希望按照降序对结果进行排序，可以使用 `DESC` 关键字。\
比如说以逆字母顺序显示公司名称，并以数字顺序显示订单号：

``` sql
SELECT * FROM Orders ORDER BY Company DESC, OrderNumber ASC;
```

# DISTINCT

符合条件的结果可能存在重复项，如需去除重复项可以使用 `DISTINCT` 关键字。

``` sql
SELECT DISTINCT LastName,FirstName FROM Persons;
```

# 别名Alias

通过使用 `AS` 关键字，可以为列和表指定别名(Alias)。

## 表的别名

``` sql
table_name AS alias_name
```

假设我们有两个表分别是：\"Persons\"和\"Orders\"，分别指定别名为\"p\"和\"o\"。\
通过以下SQL语句可以列出\"John Adams\"的所有订单：

``` sql
SELECT o.Id, p.LastName, p.FirstName
    FROM Persons AS p, Orders AS o
        WHERE p.LastName='Adams' AND p.FirstName='John';
```

不使用别名则需要这么写：

``` sql
SELECT Orders.Id, Persons.LastName, Persons.FirstName
    FROM Persons, Orders
        WHERE Persons.LastName='Adams' AND Persons.FirstName='John';
```

另外当过滤条件涉及到同一表中某些列之间的关系，使用别名可以很好地解决。\
比如说列出比领导赚得更多的员工：

  Id  | Name | Salary | ManagerId
  -- | - | - | -
  1 |  Joe   |  7000  |   3
  2 |  Henry |  8000  |   4
  3 |  Sam   |  6000  |  NULL
  4 |  Max   |  9000  |  NULL

```sql
SELECT E1.Name AS Employee
    FROM Employee AS E1, Employee AS E2
        WHERE E1.ManagerId=E2.Id AND E1.Salary>E2.Salary;
```

## 列的别名

``` sql
column_name AS alias_name
```

比如说LastName取别名为Family，FirstName取别名为Name：

``` sql
SELECT LastName AS Family, FirstName AS Name FROM Persons;
```

# JOIN

JOIN用于根据多个表中列之间的关系，从这些表中查询数据。

# UNION

UNION用于合并多个SELECT语句的结果集。\
需要注意的是这些SELECT语句必须拥有相同列数、列数据类型、列定义顺序。
比如说列出所有在中国和美国的不同的雇员名：

``` sql
SELECT Name FROM Employees_China
    UNION SELECT Name FROM Employees_USA；
```

# 授权
```sql
grant privileges on 库名.表名 to '用户名'@'远端服务器域名或IP' IDENTIFIED BY '密码';
grant all privileges on *.* to '用户名'@'%' identified by '密码';

flush privileges;
```
授权可以通过 `grant privileges` 命令操作，一般建议细粒度授权，具体到用户、表名等。  

# 导出、导入
```sql
mysqldump --all-databases > mysql_dump.sql
mysql < mysql_dump.sql
```
