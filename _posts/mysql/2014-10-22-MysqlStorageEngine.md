---
layout: post
title: "MySQL存储引擎"
description: ""
category: MySQL
tags: [Mysql, 存储引擎, storage engine]
---
{% include JB/setup %}


MySQL 支持多种存储引擎（storage engine）。如下表

|存储引擎|说明|
|:---:|:---|
|ARCHIVE|用于数据存档的引擎(数据航被插入后就不能修改了)|
|BLACKHOLE|这种存储引擎的写操作是删除数据，读操作是返回空白记录|
|CSV|这种引擎在存储数据时以逗号作为数据项之间的分隔符|
|EXAMPLE|示例存储引擎|
|FEDERATED|用于访问远程数据表的存储引擎|
|InnoDB|具备外键支持功能的事务处理引擎|
|MEMORY|内存里的数据表|
|MERGE <br> 或者叫做 MERG_MyISAM|用来管理由多个 MyISAM 数据表构成的数据表集合|
|MyISAM|默认的存储引擎|
|NDB <br> 又名 NDBCLUSTER|MySQL Cluster 专用存储引擎|
|Falcon|用来进行事务处理的存储引擎|

### MyISAM 存储引擎

### MERGE 存储引擎

### CSV 存储引擎

CSV 存储引擎在存储数据时以逗号作为数据项之间的分隔符。它会在数据库子目录里为每个数据表创建`.csv`文件。这是一个普通文本文件，每个数据行占用一个文本行。CSV 存储引擎不支持索引。


