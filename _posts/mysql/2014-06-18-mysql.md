---
layout: post
title: "mysql主从数据库不一致数据恢复"
description: ""
category: MySQL
tags: [mysql, 主从数据库，数据恢复]
---
{% include JB/setup %}

在一些情况下，我们可能会出现 MySQL 主从数据库数据冲突的情形，这个时候我们需要在从数据库中进行数据恢复，以保证主从数据库的一致性。在本文中，我们将一步一步地记录下如何进行数据备份和恢复。


一、 主数据库备份	
	使用 mysqlbackup 备份
	
~~~
mysqlbackup --defaults-file=/etc/mysql/my.cnf --user=root --compress --backup-dir=$BAKDIR  backup
~~~

	$BAKDIR 是备份文件的目录。

二、从 master 机器上把备份数据传到 slave 机器上，这个可以使用`scp`命令或者其他传输命令

三、停掉slave

进到 slave 数据库执行：
	
~~~
stop slave;
~~~

四、停掉 slave 的 mysql

~~~
/etc/init.d/mysql stop
~~~

五、在 slave 机器上执行 apply log

~~~
mysqlbackup --defaults-file=/etc/mysql/my.cnf --user=root --uncompress --backup-dir=$BACKUPDIR apply-log
~~~

`$BACKUPDIR`备份文件所在的目录。因为之前压缩的时候是有 `--compress` 参数的，所以现在必须有 `--uncompress` 参数。
    
       apply-log
              The command takes existing backup directory snapshot as input
              and expands/transforms innodb data and log files as needed to
              prepare for restore. If the original backup had been taken
              in compressed format, the --uncompress option should be used
              during apply-log operation. In order to do apply-log for
              backup image file, it must be extracted to backup directory
              first.

六、在 slave 机器上执行copy back

~~~
mysqlbackup --defaults-file=/etc/mysql/my.cnf --user=root --backup-dir=$BACKUPDIR copy-back	
~~~

执行这个命令时必须保证 mysql 已经停掉了。
	
	   copy-back
              The command restores data, index, log files from backup
              directory to server repository. The restore operation
              assumes the server is offline. Use of this command when
              server is running is not supported.
              
七、开启 slave 上的 mysql

**在默认的情况下，从数据库在启动的时候就默认开了 slave 复制线程，所以必须先把slave 复制线程停掉，否则就自动同步，然后数据又会有冲突了（等于这次数据恢复失败了），所以在启动之前，要确认一下mysql配置文件`/etc/mysql/my.cnf`文件是否有 `skip-slave-start` 选项，没有则必须加上。**
	
`skip-slave-start`：如果要mysql启动时不自动启动slave复制线程，该选项必须配置，防止slave的mysql重启时，自动复制，造成数据冲突问题。

~~~
/etc/init.d/mysql start
~~~

八、检查是否已经停掉 slave

~~~
show slave status\G
~~~

如果出现`Seconds_Behind_Master: NULL`，表示 slave 已停掉。

九、 检查备份日志上面的binlog 和 position，然后在 slave 机器上执行：

~~~
CHANGE MASTER TO MASTER_LOG_FILE="", MASTER_LOG_POS=""
~~~

在 slave 数据库上执行这一条命令，`MASTER_LOG_FILE`为备份日志上面的 binlog，`MASTER_LOG_POS`为备份日志上面的 position。

十、从数据库恢复备份之后，(如果是 debian 的机器)可能会出现

~~~
ERROR 1045 (28000): Access denied for user 'debian-sys-maint'@'localhost' (using password: YES)
~~~

这是因为Debian会有一个默认的 MySQL 用户`debian-sys-maint`， 专门用于开关和检查状态。这个用户对应的密码存放在`/etc/mysql/debian.cnf`文件。

解决方法：
	
~~~
GRANT ALL PRIVILEGES on *.* TO 'debian-sys-maint'@'localhost' IDENTIFIED BY PASSWORD 'your password';
FLUSH PRIVILEGES;
~~~
	
十一、 开启 slave

在 slave 数据库上执行：
	
~~~
start slave;
~~~

十二、 检查同步

在 master 数据库执行：

~~~
show master status\G
~~~

在 slave 数据库上执行：
	
~~~
show slave status\G
~~~

检查是否正在同步，检查同步是否成功。

十三、 把 slave 的 `/etc/mysql/my.conf` 文件改回来

把`skip-slave-start`选项从`/etc/mysql/my.cnf`去掉。
