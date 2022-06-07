---
title: 0x06 MySQL GTID 主从同步
tags: []
---

## MySQL GTID 主从同步

**概念**

从 MySQL 5.6.5 开始新增了一种基于 GTID 的复制方式。通过 GTID保证了每个在主库上提交的事务在集群中有一个唯一的ID。  
这种方式强化了数据库的主备一致性，故障恢复以及容错能力。  
在原来基于二进制日志的复制，从库需要告知主库要从哪个偏移量进行增量同步，如果指定错误会造成数据的遗漏，从而造成数据的不一致。  
借助GTID，在发生主备切换的情况下，MySQL的其它从库可以自动在新主库上找到正确的复制位置，这大大简化了复杂复制拓扑下集群的维护，
也减少了人为设置复制位置发生误操作的风险。另外，基于GTID的复制可以忽略已经执行过的事务，减少了数据发生不一致的风险。

## 什么是GTID？

GTID (Global Transaction ID) 是对于一个已提交事务的编号，并且是一个全局唯一的编号。 GTID 实际上 是由UUID+TID 组成的。  
其中 UUID 是一个 MySQL 实例的唯一标识。TID代表了该实例上已经提交的事务数量，并且随着事务提交单调递增。

下面是一个GTID的具体形式：03a1eb63-c21a-11ec-b07f-000c2987bea6:1-25，冒号分割前边为UUID，后边为TID。  

GTID 集合可以包含来自多个 MySQL 实例的事务，它们之间用逗号分隔。  

如果来自同一MySQL实例的事务序号有多个范围区间，各组范围之间用冒号分隔。  
例如： 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-6,03a1eb63-c21a-11ec-b07f-000c2987bea6:1-25  
	可以使用show master status实时查看当前事务执行数。  


## GTID的作用

GTID采用了新的复制协议，旧协议是，首先从服务器上在一个特定的偏移量位置连接到主服务器上一个给定的二进制日志文件，然后主服务器再从给定的连接点开始发送所有的事件。  
新协议有所不同，支持以全局统一事务ID (GTID)为基础的复制。当在主库上提交事务或者被从库应用时，可以定位和追踪每一个事务。  
GTID复制是全部以事务为基础，使得检查主从一致性变得非常简单。如果所有主库上提交的事务也同样提交到从库上，一致性就得到了保证。  


## GTID的工作原理

① 当一个事务在主库端执行并提交时，产生GTID，一同记录到binlog日志中。  
② binlog传输到slave,并存储到slave的relaylog后，读取这个GTID的这个值设置GTID_next变量，即告诉Slave，下一个要执行的GTID值。  
③ sql线程从relay log中获取GTID，然后对比slave端的binlog是否有该GTID。  
④ 如果有记录，说明该GTID的事务已经执行，slave会忽略。  
⑤ 如果没有记录，slave就会执行该GTID事务，并记录该GTID到自身的binlog，在读取执行事务前会先检查其他session持有该GTID，确保不被重复执行。  
⑥ 在解析过程中会判断是否有主键，如果没有就用二级索引，如果没有就用全部扫描。  


## GTID比传统复制的优势

- 更简单的实现failover，不需要找log_file和log_Pos  
- 更简单的搭建主从复制
- 比传统复制更加安全
- GTID是连续没有空洞的，因此主从库出现数据冲突时，可以用添加空事物的方式进行跳过


## 配置一主一从GTID

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
数据库角色           IP         系统与MySQL版本         有无数据
主数据库	    192.168.10.11    RHEL7 MySQL5.7        无数据
从数据库	    192.168.10.12    RHEL7 MySQL5.7        无数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



在主数据库里创建一个同步账户授权给从数据库使用

```xml
[root@server11 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.24 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> grant replication slave on *.* to 'Jaking'@'192.168.10.%' identified by 'Jaking@vip.163.com';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```



**配置主库文件**

```xml
[root@server11 ~]# vim /etc/my.cnf
[mysqld]
port=3306
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
skip-name-resolve
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
skip-name-resolve

log-bin=mysql-bin                        // 开启二进制日志
server-id=11                             // 服务器ID，必须唯一
gtid-mode=on                             #开启gtid模式
enforce-gtid-consistency=on       //强制gtid一致性，开启后对特定的create table不支持
binlog-format=row                       //默认为mixed混合模式，更改成row复制，为了数据一致性
log-slave-updates=1                    //从库binlog记录主库同步的操作日志
skip-slave-start=1                         //跳过slave复制线程
```



```xml
[root@server11 ~]# systemctl restart mysqld
[root@server11 ~]# systemctl status mysqld 
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-04-22 05:28:12 EDT; 6s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 11752 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 11730 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 11755 (mysqld)
    Tasks: 27
   CGroup: /system.slice/mysqld.service
           └─11755 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld...

Apr 22 05:28:11 server11 systemd[1]: Stopped MySQL Server.
Apr 22 05:28:11 server11 systemd[1]: Starting MySQL Server...
Apr 22 05:28:12 server11 systemd[1]: Started MySQL Server.
```


**查看GTID模式状态**

```xml
[root@server11 ~]# mysql -uroot -pJaking@vip.163.com -e 'show variables like "%GTID%";'
mysql: [Warning] Using a password on the command line interface can be insecure.
+----------------------------------+-----------+
| Variable_name                    | Value     |
+----------------------------------+-----------+
| binlog_gtid_simple_recovery      | ON        |
| enforce_gtid_consistency         | ON        |
| gtid_executed_compression_period | 1000      |
| gtid_mode                        | ON        |
| gtid_next                        | AUTOMATIC |
| gtid_owned                       |           |
| gtid_purged                      |           |
| session_track_gtids              | OFF       |
+----------------------------------+-----------+
```


**配置从库文件**

```xml
[root@server12 ~]# vim /etc/my.cnf
[mysqld]
port=3306
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
skip-name-resolve

log-bin=mysql-bin
server-id=12
gtid-mode=on  
enforce-gtid-consistency=on          
binlog-format=row
log-slave-updates=1
skip-slave-start=1
[root@server12 ~]# systemctl restart mysqld
[root@server12 ~]# systemctl status mysqld 
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-04-22 18:08:39 CST; 8s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 2749 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 2731 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 2753 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─2753 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/...

Apr 22 18:08:38 server12 systemd[1]: Starting MySQL Server...
Apr 22 18:08:39 server12 systemd[1]: Started MySQL Server.
```


**查看GTID模式状态**

```xml
[root@server12 ~]# mysql -uroot -pJaking@vip.163.com -e 'show variables like "%GTID%";'
mysql: [Warning] Using a password on the command line interface can be insecure.
+----------------------------------+-----------+
| Variable_name                    | Value     |
+----------------------------------+-----------+
| binlog_gtid_simple_recovery      | ON        |
| enforce_gtid_consistency         | ON        |
| gtid_executed_compression_period | 1000      |
| gtid_mode                        | ON        |
| gtid_next                        | AUTOMATIC |
| gtid_owned                       |           |
| gtid_purged                      |           |
| session_track_gtids              | OFF       |
+----------------------------------+-----------+
```


**开启主从同步**

```sql
[root@server12 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.24-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> change master to
     master_host='192.168.10.11',
     master_user='Jaking',
     master_password='Jaking@vip.163.com',
     master_auto_position=1;

Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.10.11
                  Master_User: Jaking
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: server12-relay-bin.000002
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 577
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 11
                  Master_UUID: 03a1eb63-c21a-11ec-b07f-000c2987bea6
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```


查看主从库的信息是否同步

查看主库

```sql
[root@server11 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.24-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```


查看从库

```sql
[root@server12 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.24-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```


## 测试主从同步

在主库执行以下SQL语句

```sql
CREATE DATABASE BOOK;
USE BOOK;
CREATE TABLE book (name char(66),price int,pages int);
INSERT INTO book(name,price,pages) VALUES('Linux','30','666');
INSERT INTO book(name,price,pages) VALUES('Cloud Computing','60','666');
INSERT INTO book(name,price,pages) VALUES('Operation System','80','666');
INSERT INTO book(name,price,pages) VALUES('Artificial Intelligence','166','666');
select * from book;
------------------------------------------------------------------------------------
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> select * from BOOK.book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
+-------------------------+-------+-------+
4 rows in set (0.00 sec)
```


在从库查看数据是否同步过来

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> select * from BOOK.book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
+-------------------------+-------+-------+
4 rows in set (0.00 sec)
```


查看主库状态

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000002 |     1620 |              |                  | 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-6 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)
```


查看从库状态

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000002 |     1584 |              |                  | 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-6 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)
```


从库停掉主从同步

```sql
mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.10.11
                  Master_User: Jaking
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 1620
               Relay_Log_File: server12-relay-bin.000002
                Relay_Log_Pos: 1833
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1620
              Relay_Log_Space: 2043
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 11
                  Master_UUID: 03a1eb63-c21a-11ec-b07f-000c2987bea6
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-6
            Executed_Gtid_Set: 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-6
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```


在主库创建数据后从库开启主从同步查看是否同步

```sql
CREATE DATABASE `testbookdb` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
show databases;
use testbookdb;
CREATE TABLE book1 (bTypeId int,bName char(16),price int,publishing char(16));
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('1','《Linux从入门到精通》','66','电子工业出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('2','《云计算趋势》','68','人民邮电出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('3','《操作系统设计与实现》','90','机械工业出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('4','《高性能MySQL》','71','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('5','《高性能MySQL2》','72','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('6','《高性能MySQL3》','73','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('7','《高性能MySQL4》','74','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('8','《高性能MySQL5》','75','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('9','《高性能MySQL6》','76','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('10','《高性能MySQL7》','77','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('11','《高性能MySQL8》','78','清华大学出版社');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('12','《高性能MySQL9》','79','清华大学出版社');

CREATE TABLE book2 (name char(16),price int,pages int);
INSERT INTO book2(name,price,pages) VALUES('《Linux从入门到精通》','30','666');
INSERT INTO book2(name,price,pages) VALUES('《云计算趋势》','60','666');
INSERT INTO book2(name,price,pages) VALUES('《操作系统设计与实现》','80','666');
INSERT INTO book2(name,price,pages) VALUES('《高性能MySQL》','166','666');

show tables;
select * from book1;
select * from book2;
```



```sql
mysql> select * from book1;
+---------+-----------------------------------+-------+-----------------------+
| bTypeId | bName                             | price | publishing            |
+---------+-----------------------------------+-------+-----------------------+
|       1 | 《Linux从入门到精通》             |    66 | 电子工业出版社        |
|       2 | 《云计算趋势》                    |    68 | 人民邮电出版社        |
|       3 | 《操作系统设计与实现》            |    90 | 机械工业出版社        |
|       4 | 《高性能MySQL》                   |    71 | 清华大学出版社        |
|       5 | 《高性能MySQL2》                  |    72 | 清华大学出版社        |
|       6 | 《高性能MySQL3》                  |    73 | 清华大学出版社        |
|       7 | 《高性能MySQL4》                  |    74 | 清华大学出版社        |
|       8 | 《高性能MySQL5》                  |    75 | 清华大学出版社        |
|       9 | 《高性能MySQL6》                  |    76 | 清华大学出版社        |
|      10 | 《高性能MySQL7》                  |    77 | 清华大学出版社        |
|      11 | 《高性能MySQL8》                  |    78 | 清华大学出版社        |
|      12 | 《高性能MySQL9》                  |    79 | 清华大学出版社        |
+---------+-----------------------------------+-------+-----------------------+
12 rows in set (0.00 sec)

mysql> select * from book2;
+-----------------------------------+-------+-------+
| name                              | price | pages |
+-----------------------------------+-------+-------+
| 《Linux从入门到精通》             |    30 |   666 |
| 《云计算趋势》                    |    60 |   666 |
| 《操作系统设计与实现》            |    80 |   666 |
| 《高性能MySQL》                   |   166 |   666 |
+-----------------------------------+-------+-------+
4 rows in set (0.00 sec)
```


查看主库状态

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                         |
+------------------+----------+--------------+------------------+-------------------------------------------+
| mysql-bin.000002 |     7389 |              |                  | 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-25 |
+------------------+----------+--------------+------------------+-------------------------------------------+
1 row in set (0.00 sec)
```


查看从库状态

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000002 |     1584 |              |                  | 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-6 |
+------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)
```


从库开启主从同步查看是否同步

```sql
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.10.11
                  Master_User: Jaking
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 7389
               Relay_Log_File: server12-relay-bin.000003
                Relay_Log_Pos: 6223
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 7389
              Relay_Log_Space: 8112
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 11
                  Master_UUID: 03a1eb63-c21a-11ec-b07f-000c2987bea6
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-25
            Executed_Gtid_Set: 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-25
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| mysql              |
| performance_schema |
| sys                |
| testbookdb         |
+--------------------+
6 rows in set (0.00 sec)

mysql> use testbookdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------------+
| Tables_in_testbookdb |
+----------------------+
| book1                |
| book2                |
+----------------------+
2 rows in set (0.00 sec)

mysql> select * from book1;
+---------+-----------------------------------+-------+-----------------------+
| bTypeId | bName                             | price | publishing            |
+---------+-----------------------------------+-------+-----------------------+
|       1 | 《Linux从入门到精通》             |    66 | 电子工业出版社        |
|       2 | 《云计算趋势》                    |    68 | 人民邮电出版社        |
|       3 | 《操作系统设计与实现》            |    90 | 机械工业出版社        |
|       4 | 《高性能MySQL》                   |    71 | 清华大学出版社        |
|       5 | 《高性能MySQL2》                  |    72 | 清华大学出版社        |
|       6 | 《高性能MySQL3》                  |    73 | 清华大学出版社        |
|       7 | 《高性能MySQL4》                  |    74 | 清华大学出版社        |
|       8 | 《高性能MySQL5》                  |    75 | 清华大学出版社        |
|       9 | 《高性能MySQL6》                  |    76 | 清华大学出版社        |
|      10 | 《高性能MySQL7》                  |    77 | 清华大学出版社        |
|      11 | 《高性能MySQL8》                  |    78 | 清华大学出版社        |
|      12 | 《高性能MySQL9》                  |    79 | 清华大学出版社        |
+---------+-----------------------------------+-------+-----------------------+
12 rows in set (0.00 sec)

mysql> select * from book2;
+-----------------------------------+-------+-------+
| name                              | price | pages |
+-----------------------------------+-------+-------+
| 《Linux从入门到精通》             |    30 |   666 |
| 《云计算趋势》                    |    60 |   666 |
| 《操作系统设计与实现》            |    80 |   666 |
| 《高性能MySQL》                   |   166 |   666 |
+-----------------------------------+-------+-------+
4 rows in set (0.00 sec)

mysql> show master status;
+------------------+----------+--------------+------------------+-------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                         |
+------------------+----------+--------------+------------------+-------------------------------------------+
| mysql-bin.000002 |     7113 |              |                  | 03a1eb63-c21a-11ec-b07f-000c2987bea6:1-25 |
+------------------+----------+--------------+------------------+-------------------------------------------+
1 row in set (0.00 sec)
```
