---
layout: post
title: "MySQL MyISAM引擎和InnoDB引擎对比"
description: ""
category: MySQL
tags: [Mysql, 存储引擎, storage engine, MyISAM, InnoDB]
---
{% include JB/setup %}

###MyISAM 和 InnoDB 的区别

* MyISAM是 MySQL 的默认存储引擎，基于传统的 ISAM 类型，支持全文索引，但不是事务安全的，而且不支持外键。
*  InnoDB是事务型引擎，支持回滚、崩溃恢复能力、多版本并发控制、ACID事务，支持行级锁定。

MyISAM和InnoDB的区别：

* MyISAM支持全文索引，InnoDB 不支持。
* MyISAM锁的粒度是表级，而 InnoDB 支持行级锁定。
* MyISAM不支持事务处理和外键索引，而InnoDB支持。
* MyISAM 在效率上要高于 InnoDB。
* MyISAM 表是保存成文件的形式，在跨平台的数据转移中适用 MyISAM 存储会省去不少的麻烦。
* MyISAM 保存表的具体行数，而 InnoDB 不保存表的具体行数。也就是说，执行select count(\*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(\*)语句包含 where条件时，两种表的操作是一样的。


#### MyISAM 优缺点
MyISAM优点：

* 表的创建和设计非常简单。
* 高性能。
* 支持全文搜索。    

MyISAM缺点：

* 数据完整性较差，不支持事务处理和外键处理。
* 表锁定，当插入或者更新一行时，需要将整个表锁定，不能同时更新多行。

MyISAM适用场景：

* MyISAM管理非事务表，它提供高速存储和检索、以及全文搜索能力；如果应用需要比较高的性能，读多写少，同时不需要进行事务处理，则 MyISAM 适合使用；一般情况来说，小型的应用或者项目，MyISAM 会更适合。


#### InnoDB 优缺点
InnoDB优点：
    
* 数据完整性和外键支持。
* 支持事务处理。
* 支持行级别的锁定。

InnoDB缺点：

* 表的设计相对麻烦一点，需要考虑外键等情形。
* 不支持全文索引。
* 相比 MyISAM，性能稍差。

InnoDB适用场景：

* InnoDB非常适合对数据完整性有要求的场景，例如在线商城、财务应用等；如果应用中需要执行大量的 insert 或者 update 操作，则应该使用 InnoDB，这样可以提高多用户并发操作的性能；一般来说，超大数据量的项目，而且需要事务处理或者外键支持的话，InnoDB 更适合。
