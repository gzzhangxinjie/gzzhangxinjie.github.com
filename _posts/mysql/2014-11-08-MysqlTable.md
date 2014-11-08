---
layout: post
title: "MySQL数据表操作"
description: ""
category: mysql
tags: [mysql, 数据表]
---
{% include JB/setup %}


## 数据表的创建
MySQL 数据表的创建可以通过以下语句格式创建：

~~~
CREATE TABLE tbl_name (column_specs);
~~~
这里介绍几种创建数据表的变体：

* 改变存储引擎。
* 只在数据表不存在时才创建。
* 临时数据表，服务器会在客户会话结束时自动删除它们。
* 从另一个数据表或是从一次 SELECT 查询的结果来创建数据表。
* 使用 MERGE 数据表、分区数据表、FEDERATED 数据表。

### 指定存储引擎

* 创建数据表时可以在 `CREATE TABLE` 语句后面指定存储引擎, 存储引擎的名字不区分大小写。

~~~
CREATE TABLE tbl_name (column_specs) ENGINE = engine_name;
~~~
例如，创建一个 InnoDB 和 MyISAM 的数据表，创建语句如下：

~~~
CREATE TABLE tbl_name (column_specs) ENGINE = InnoDB;

CREATE TABLE tbl_name (column_specs) ENGINE = MyISAM;

~~~

如果没有指定存储引擎，则使用系统的默认存储引擎来创建数据表。内建的默认存储引擎是 MyISAM。在服务器运行期间，也可以通过设置系统选项 `storage-engine` 改变默认的存储引擎。

* 可以通过 `SHOW CREATE TABLE tbl_name` 来查看数据表使用的是哪一种引擎。

~~~
mysql> CREATE TABLE test(name varchar(30)) ENGINE = InnoDB;
Query OK, 0 rows affected (0.04 sec)

mysql> SHOW CREATE TABLE test\G;
*************************** 1. row ***************************
       Table: test
Create Table: CREATE TABLE `test` (
  `name` varchar(30) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
~~~

### 只创建当前不存在的数据表

我们可以通过 `CREATE TABLE IF NOT EXIST` 语句来创建当前不存在的数据表。如果当前已经存在同名的数据表，那么将不会再创建数据表。这种语句特别适合在脚本中使用，因为多次执行创建语句不会报错。（如果没有`IF NOT EXIST` 语句，创建一次数据表之后，如果再执行相同的语句，会报错。）

注意，使用 `CREATE TABLE IF NOT EXIST` 语句时，mysql 并不会检查已有的数据表的表结构与现在要创建的数据表结构是否一致。只要是相同名字了，mysql 就不会重新建表了。这样的话，有可能会因为数据表结构的不一致导致后面处理时会出现字段错乱的问题。如果想要避免这种情况，可以使用 `DROP TABLE IF EXIST`语句删除表，然后再使用 `CREATE TABLE` 创建表。

### 创建临时数据表

我们可以在创建数据表时加上`TEMPORARY` 关键字，

~~~
CREATE TEMPORARY TABLE tbl_name... ;
~~~
服务器将会创建一个临时的数据表，它在我们与服务器断开时自动消失。

临时数据表可以与已有的数据表使用同一个名字。当使用相同名字时，已有的数据表将会被隐藏。我们只能访问到临时数据表；当我们与服务器断开，重连时，我们又可以继续使用已有的数据表了；或者当我们将临时数据表删除掉时，我们也可以继续使用已有的数据表。

不过，并不建议创建与已有数据表相同名字的临时数据表。这个容易给自己留下隐藏的问题：如果我们与服务器断开了，但是又使用了某些机制会重连服务器；那么按照机制，此时呈现在我们面前的并不是临时数据表，但假如我们并不清楚这种情况，而又删除了数据表（以为删的是临时数据表），那么就容易出现问题。

### 从其他数据表或查询结果创建数据表

~~~
CREATE TABLE new_tbl_name LIKE tbl_name;
~~~

这个语句将创建与`tbl_name`相同的格式的新表`new_tbl_name`，新表`new_tbl_name`的属性和索引都与`tbl_name`一样。不过新表的内容是空的，如果需要把`tbl_name`的内容复制到`new_tbl_name`，可以使用下面语句。

~~~
INSERT INTO new_tbl_name SELECT * from tbl_name;
~~~

我们也可以通过`SELECT`语句创建数据表：

~~~
CREATE TABLE ... SELECT...
~~~
例如，

~~~
CREATE TABLE new_tbl_name1 SELECT * FROM tbl_name;
CREATE TABLE new_tbl_name2 SELECT column1, column2 FROM tbl_name;
~~~
通过查询语句创建数据表时，不仅可以创建指定的数据列，还可以拷贝相应的内容。不过这种创建方式并不会将`tbl_name`的属性（例如,`auto_increment`属性）及索引复制到`new_tbl_name`中。

###创建 MERGE 数据表

MERGE 存储引擎把一组 MyISAM 数据表当做一个逻辑单元来对待，让我们可以同时对他们进行查询。构成一个 MERGE 数据表的各成员 MyISAM 表必须具备完全一样的结构。每个成员表的的数据列必须按照相同的顺序定义相同的名字和类型，索引也必须按照同样的顺序和方式定义。

创建 merge 表的结构如下：

~~~
CREATE TABLE tbl_name
(...)
ENGINE = MERGE UNION = (tbl_name1, tbl_name2, ...);
~~~
其中，

1. 括号中的属性必须与每个 `MyISAM` 表一样；
2. 存储引擎为`MERGE`;
3. `UNION`选项指定了被选中的`MyISAM`表。


###使用分区数据表
略

###使用 FEDERATED 数据表
略


## 数据表的删除

删除一个数据表:

~~~
DROP TABLE tbl_name;
~~~

删除多个数据表：

~~~
DROP TABLE tbl_name1, tbl_name2, ...;
~~~

判断表是否存在，如果存在，删除这个表：

~~~
DROP TABLE IF EXIST tbl_name;
~~~

删除临时数据表：

~~~
DROP TEMPORARY TABLE tbl_name;
~~~
