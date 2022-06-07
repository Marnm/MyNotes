---
title: 09MySQL传统主从同步
tags:
  - mysql
  - master
  - slave
---

## 环境

| **主机名** | **IP** | **系统/MySQL版本** | 角色 |
|   ---     |   ---  |     ---  |    ---    |
|MySQL_1 | 192.168.10.67 | RHEL7.3/MySQL5.7 | Master |
| MySQL_2 | 192.168.10.68 | RHEL7.3/MySQL5.7 | Slave |


## 原理

![](http://192.168.85.188:8081/uploads/1652922895516766641123.png)

1)、在master机器上的操作：

　  当master上的数据发生变化时，该事件变化会按照顺序写入bin-log中。当slave链接到master的时候，master机器会为slave开启binlog dump线程。当master的binlog发生变化的时候，bin-log dump线程会通知slave，并将相应的binlog内容发送给slave。

2)、在slave机器上操作：

　  当主从同步开启的时候，slave上会创建两个线程：I/O线程。该线程连接到master机器，master机器上的binlog dump 线程会将binlog的内容发送给该I/O线程。该I/O线程接收到binlog内容后，再将内容写入到本地的relay log；sql线程。该线程读取到I/O线程写入的relay log。并且根据replay log。并且根据relay log 的内容对slave数据库做相应的操作。

3)、MySQL主从同步原理图如下:

![](http://192.168.85.188:8081/uploads/1652922926865746299theory.png)


从库生成两个线程，一个I/O线程，一个SQL线程；  
I/O线程去请求主库的binlog，并将得到的binlog日志写到relay log（中继日志） 文件中；  
主库会生成一个 log dump 线程，用来给从库I/O线程传binlog；  
SQL 线程，会读取relay log文件中的日志，并解析成具体操作，来实现主从的操作一致，而最终数据一致；  

<span style="color:red">*注意：搭建MySQL主从同步前，需要分别在主数据和从数据上安装并配置好MySQL数据库！  *</span>

## 配置 Master 192.168.10.67

### 配置 /etc/my.cnf

```xml
[root@MySQL1 mysql‐5.7.19]# vim /etc/my.cnf
[client]
port=3306
socket=/data/mysql/mysql.sock

[mysqld]  
character‐set‐server=utf8
collation‐server=utf8_general_ci

skip‐name‐resolve
user=mysql
port=3306
basedir=/usr/local/mysql57
datadir=/data/mysql
tmpdir=/tmp
socket=/data/mysql/mysql.sock

log‐error=/data/mysql/mysqld.log
pid‐file=/data/mysql/mysqld.pid

log‐bin=mysql‐bin‐master #启用二进制日志
binlog_format=mixed
server‐id=1 #本机数据库ID 标示
binlog‐do‐db=TESTDB #可以被从服务器复制的库, 二进制需要同步的数据库名
binlog‐ignore‐db=mysql #不可以被从服务器复制的库
lower_case_table_names=1
#注意：Linux下部署安装MySQL，默认不忽略表名大小写，需要手动到/etc/my.cnf 下配置 lower_case_table_names=1 使Linux环境下MySQL忽略表名大小写，否则使用MyCAT的时候会提示找不到表的错误！
 ~
 ~
"/etc/my.cnf" 24L, 556C written
```

```xml
[root@MySQL1 ~]# service mysqld restart
Shutting down MySQL........... SUCCESS!
Starting MySQL. SUCCESS!
```

创建要同步的数据库并创建表
```sql
[root@MySQL1 mysql‐5.7.19]# mysql -uroot -pJaking@vip.163.com
mysql>
CREATE DATABASE TESTDB;
show databases;
use TESTDB;
CREATE TABLE TABLE666 (bTypeId int,bName char(16),price int,publishing char(16));

mysql> CREATE DATABASE TESTDB;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
+--------------------+
5 rows in set (0.00 sec)

mysql> use TESTDB;
Database changed

mysql> CREATE TABLE TABLE666 (bTypeId int,bName char(16),price int,publishing char(16));
Query OK, 0 rows affected (0.01 sec)
```

授权
```sql
[root@MySQL1 mysql‐5.7.19]# mysql -uroot -pJaking@vip.163.com
#如果提示 mysql 命令不存在，请尝试用 /usr/local/mysql57/bin/mysql ‐uroot ‐p"Jaking@vip.163.com" 来登录MySQL数据库
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.19‐log Source distribution

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> grant replication slave on *.* to slave@"192.168.10.%" identified by "Jaking@vip.163.com";
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> show master status;
+-------------------------+----------+--------------+------------------+-------------------+
| File                    | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------------+----------+--------------+------------------+-------------------+
| mysql-bin-master.000002 |      154 | TESTDB       | mysql            |                   |
+-------------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> show binlog events\G
*************************** 1. row ***************************
   Log_name: mysql-bin-master.000001
        Pos: 4
 Event_type: Format_desc
  Server_id: 11
End_log_pos: 123
       Info: Server ver: 5.7.24-log, Binlog ver: 4
*************************** 2. row ***************************
   Log_name: mysql-bin-master.000001
        Pos: 123
 Event_type: Previous_gtids
  Server_id: 11
End_log_pos: 154
       Info: 
*************************** 3. row ***************************
   Log_name: mysql-bin-master.000001
        Pos: 154
 Event_type: Stop
  Server_id: 11
End_log_pos: 177
       Info: 
3 rows in set (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
[root@MySQL1 ~]# ls /var/lib/mysql
auto.cnf         ib_logfile1              performance_schema
ca-key.pem       ibtmp1                   private_key.pem
ca.pem           mysql                    public_key.pem
client-cert.pem  mysql-bin-master.000001  server-cert.pem
client-key.pem   mysql-bin-master.000002  server-key.pem
ib_buffer_pool   mysql-bin-master.index   sys
ibdata1          mysql.sock               testdb
ib_logfile0      mysql.sock.lock
[root@MySQL1 ~]# 
```

导出数据库并传给从服务器

```xml
[root@MySQL1 mysql‐5.7.19]# mysqldump -uroot -pJaking@vip.163.com -B TESTDB > TESTDB.sql
mysqldump: [Warning] Using a password on the command line interface can be insecure.

[root@MySQL1 mysql‐5.7.19]# ls
BUILD CPackConfig.cmake install_manifest.txt mysys storage
client CPackSourceConfig.cmake libbinlogevents mysys_ssl strings
cmake CTestTestfile.cmake libbinlogstandalone packaging support‐files
CMakeCache.txt dbug libevent plugin testclients
CMakeFiles Docs libmysql rapid unittest
cmake_install.cmake Doxyfile‐perfschema libmysqld README VERSION
CMakeLists.txt extra libservices regex VERSION.dep
cmd‐line‐utils TESTDB.sql make_dist.cmake scripts vio
config.h.cmake include Makefile source_downloads win
configure.cmake info_macros.cmake man sql zlib
COPYING INSTALL mysql‐test sql‐common

[root@MySQL1 mysql‐5.7.19]# scp TESTDB.sql 192.168.10.68:/root
The authenticity of host '192.168.10.68 (192.168.10.68)' can't be established.
ECDSA key fingerprint is 57:3d:7f:91:5a:bc:6f:a1:88:4f:d2:fc:17:0e:51:8b.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.68' (ECDSA) to the list of knownhosts.
root@192.168.10.68's password:
TESTDB.sql 100% 1765 1.7KB/s 00:00
```


## 配置 Slave 192.168.10.68

### 配置 /etc/my.cnf
```xml
[root@MySQL2 ~]# vim /etc/my.cnf
[client]
port=3306
socket=/data/mysql/mysql.sock

[mysqld]
cTESTDBracter‐set‐server=utf8
collation‐server=utf8_general_ci

skip‐name‐resolve
user=mysql
port=3306
basedir=/usr/local/mysql57
datadir=/data/mysql
tmpdir=/tmp
socket=/data/mysql/mysql.sock

log‐error=/data/mysql/mysqld.log
pid‐file=/data/mysql/mysqld.pid

server‐id=2
#从服务器ID号，不要和主ID相同 ，如果设置多个从服务器，每个从服务器必须有一个唯一的server‐id值，必须与主服务器的以及其它从服务器的不相同。可以认为server‐id值类似于IP地址：这些ID值能唯一识别复制服务器群集中的每个服务器实例。
lower_case_table_names=1
#注意：Linux下部署安装MySQL，默认不忽略表名大小写，需要手动到/etc/my.cnf 下配置 lower_case_table_names=1 使Linux环境下MySQL忽略表名大小写，否则使用MyCAT的时候会提示找不到表的错误！
~
~
"/etc/my.cnf" 20L, 649C written
```

```xml
[root@MySQL2 ~]# service mysqld restart
Shutting down MySQL.. [ OK ]
Starting MySQL. [ OK ]
```

*两台数据库服务器mysql版本要一致*

```sql
[root@MySQL1 ~]# mysql ‐uroot ‐pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 21
Server version: 5.7.19‐log Source distribution

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like '%version%';
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | Variable_name | Value |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | innodb_version | 5.7.19 |
 | protocol_version | 10 |
 | slave_type_conversions | |
 | tls_version | TLSv1,TLSv1.1 |
 | version | 5.7.19‐log |
 | version_comment | Source distribution |
 | version_compile_machine | x86_64 |
 | version_compile_os | Linux |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 8 rows in set (0.01 sec)

 mysql>

[root@MySQL2 mysql]# mysql ‐uroot ‐pJaking@vip.163.com
 mysql: [Warning] Using a password on the command line interface can be insecure.
 Welcome to the MySQL monitor. Commands end with ; or \g.
 Your MySQL connection id is 6
 Server version: 5.7.19 Source distribution

 Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

 Oracle is a registered trademark of Oracle Corporation and/or its
 affiliates. Other names may be trademarks of their respective
 owners.

 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

 mysql> show variables like '%version%';
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | Variable_name | Value |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | innodb_version | 5.7.19 |
 | protocol_version | 10 |
 | slave_type_conversions | |
 | tls_version | TLSv1,TLSv1.1 |
 | version | 5.7.19 |
 | version_comment | Source distribution |
 | version_compile_machine | x86_64 |
 | version_compile_os | Linux |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 8 rows in set (0.02 sec)

 mysql> exit
 Bye
```

测试连接到主数据库是否成功

```sql
 [root@MySQL2 mysql]# mysql -uslave -pJaking@vip.163.com -h 192.168.10.67
 mysql: [Warning] Using a password on the command line interface can be insecure.
 Welcome to the MySQL monitor. Commands end with ; or \g.
 Your MySQL connection id is 5
 Server version: 5.7.19‐log Source distribution

 Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

 Oracle is a registered trademark of Oracle Corporation and/or its
 affiliates. Other names may be trademarks of their respective
 owners.

 Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

 mysql> show databases;
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | Database |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | information_schema |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 1 row in set (0.00 sec)
```
 #此时看不到 TESTDB 数据库属正常现象

向TESTDB数据库导入数据

```sql
[root@MySQL2 ~]# mysql -uroot -pJaking@vip.163.com < TESTDB.sql 
mysql: [Warning] Using a password on the command line interface can be insecure.

[root@MySQL2 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.7.19 Source distribution

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
| Database |
+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
| information_schema |
| TESTDB |
| mysql |
| performance_schema |
| sys |
+‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
5 rows in set (0.00 sec)

mysql> use TESTDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with ‐A

Database changed
mysql> show tables;
+‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
| Tables_in_TESTDB |
+‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
| table666 |
+‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
1 row in set (0.00 sec)

mysql> exit
Bye
```
                                 
指定主数据库，重启slave
```sql
[root@MySQL2 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.19 Source distribution

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> stop slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> change master to master_host='192.168.10.67',master_user='slave',master_password='Jaking@vip.163.com';
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
Slave_IO_State: Waiting for master to send event
Master_Host: 192.168.10.67
Master_User: slave
Master_Port: 3306
Connect_Retry: 60
Master_Log_File: mysql‐bin‐master.000002
Read_Master_Log_Pos: 601
Relay_Log_File: MySQL2‐relay‐bin.000003
Relay_Log_Pos: 828
Relay_Master_Log_File: mysql‐bin‐master.000002
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

Slave_IO_Running ：一个负责与主机的IO通信  
Slave_SQL_Running：负责自己的slave mysql进程  
两个为 Yes 就成功了！  

再到主数据库上查看状态  

```sql
1 [root@MySQL1 mysql‐5.7.19]# mysql ‐uroot ‐pJaking@vip.163.com
2 mysql: [Warning] Using a password on the command line interface can be in
secure.
3 Welcome to the MySQL monitor. Commands end with ; or \g.
4 Your MySQL connection id is 7
5 Server version: 5.7.19‐log Source distribution
6
7 Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserv
ed.
8
9 Oracle is a registered trademark of Oracle Corporation and/or its
10 affiliates. Other names may be trademarks of their respective
11 owners.
12
13 Type 'help;' or '\h' for help. Type '\c' to clear the current input stat
ement.
14
15 mysql> show processlist\G
16 *************************** 1. row ***************************
17  Id: 6
18  User: slave
19  Host: 192.168.10.68:54656
20  db: NULL
21 Command: Binlog Dump
22  Time: 165
23  State: Master TESTDBs sent all binlog to slave; waiting for more updates
24  Info: NULL
25 *************************** 2. row ***************************
26  Id: 7
27  User: root
28  Host: localhost
29  db: NULL
30 Command: Query
31  Time: 0
32  State: starting
33  Info: show processlist
34 2 rows in set (0.00 sec)
35
36 mysql>
```

测试主从同步

```sql
[root@MySQL1 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.7.19-log MySQL Community Server (GPL)

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
| testdb             |
+--------------------+
5 rows in set (0.00 sec)

mysql> use testdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_testdb |
+------------------+
| table666         |
+------------------+
1 row in set (0.00 sec)

mysql> select * from table666;
Empty set (0.00 sec)

mysql> 

INSERT INTO table666(bTypeId,bName,price,publishing) VALUES('1','Linux','66','DZ');
INSERT INTO table666(bTypeId,bName,price,publishing) VALUES('2','CLD','68','RM');
INSERT INTO table666(bTypeId,bName,price,publishing) VALUES('3','SYS','90','JX');
INSERT INTO table666(bTypeId,bName,price,publishing) VALUES('4','MySQL1','71','QH');
INSERT INTO table666(bTypeId,bName,price,publishing) VALUES('5','MySQL2','72','QH');
INSERT INTO table666(bTypeId,bName,price,publishing) VALUES('6','MySQL3','73','QH');

mysql> select * from table666;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
+---------+--------+-------+------------+
6 rows in set (0.00 sec)

mysql> ^DBye
[root@MySQL1 ~]#
```

```sql
[root@MySQL2 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select * from testdb.table666;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
+---------+--------+-------+------------+
6 rows in set (0.00 sec)

mysql> 
```

主库

```sql
mysql> select * from table666;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
+---------+--------+-------+------------+
6 rows in set (0.00 sec)

mysql> delete from table666 where bName="MySQL3";
Query OK, 1 row affected (0.00 sec)

mysql> select * from table666;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
+---------+--------+-------+------------+
5 rows in set (0.00 sec)

mysql> 
```

从库

```sql
mysql> select * from testdb.table666;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
+---------+--------+-------+------------+
5 rows in set (0.00 sec)
```

主库

```sql
mysql> select * from table666;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
+---------+--------+-------+------------+
5 rows in set (0.00 sec)

mysql> INSERT INTO table666(bTypeId,bName,price,publishing) VALUES('666','Linux666','666','QH');
Query OK, 1 row affected (0.00 sec)

mysql> select * from table666;                                                 
+---------+----------+-------+------------+
| bTypeId | bName    | price | publishing |
+---------+----------+-------+------------+
|       1 | Linux    |    66 | DZ         |
|       2 | CLD      |    68 | RM         |
|       3 | SYS      |    90 | JX         |
|       4 | MySQL1   |    71 | QH         |
|       5 | MySQL2   |    72 | QH         |
|     666 | Linux666 |   666 | QH         |
+---------+----------+-------+------------+
6 rows in set (0.00 sec)

mysql>
```

从库

```sql
mysql> select * from testdb.table666;
+---------+----------+-------+------------+
| bTypeId | bName    | price | publishing |
+---------+----------+-------+------------+
|       1 | Linux    |    66 | DZ         |
|       2 | CLD      |    68 | RM         |
|       3 | SYS      |    90 | JX         |
|       4 | MySQL1   |    71 | QH         |
|       5 | MySQL2   |    72 | QH         |
|     666 | Linux666 |   666 | QH         |
+---------+----------+-------+------------+
6 rows in set (0.00 sec)

mysql>
```

## 至此，MySQL传统主从同步搭建成功！


## 练习：1. 配置互为主从  2. 先配置常规主从，然后增加一台从库

修改配置文件  

master_1:

	bind-address    = 0.0.0.0   # 启用远程连接

	log-bin=mysql-bin   
	# 步进值，如果有n台master，就填n：
	auto_increment_increment=2
	# 起始值，一般填第n台master，这里是第1台主MySQL
	auto_increment_offset=1
	server-id=1
	lower_case_table_names=1

master_2:

	auto_increment_increment=2
	auto_increment_offset=2
	log-bin=mysql-bin
	binlog_format=mixed
	server-id=2
	lower_case_table_names=1


\# 导出数据到master_2
master_1:

	mysqldump -uroot -p -A > all_db.sql
	scp all_db.sql master_2_user@master_2_ipaddr:~/

master_2:
	
	mysql -uroot -p < all.db.sql

\# 记录bin-log文件跟位置，并将其授权给对方
#例如：
master_1:

	show master status;


#将自己的bin-log文件授权给其他master：
```sql
GRANT REPLICATION SLAVE ON *.* TO slave@'master_2_ipaddr' IDENTIFIED BY ‘slave_password’;
STOP SLAVE;
CHANGE MASTER TO master_host='master_2_ipaddr',master_user='slave',master_password='slave_password',master_log_file='mysql-bin.000005',master_log_pos=13756331;
FLUSH PRIVILEGES;
START SLAVE;
SHOW SLAVE STATUS\G
```

master_2:

# 将自己的bin-log文件授权给其他maser
```sql
GRANT REPLICATION SLAVE ON *.* TO slave@'master_1_ipaddr' IDENTIFIED BY ‘slave_password’;
FLUSH PRIVILEGES;
STOP SLAVE;
CHANGE MASTER TO master_host='master_1_ipaddr',master_user='slave',master_password='slave_password',master_log_file='mysql-bin.000001',mysql_log_pos=1022;
FLUSH PRIVILEGES;
START SLAVE;
SHOW SLAVE STATUS\G
```
参考：https://www.smister.com/post-41/mysql-master-master.html

\# 在原有主从上 新增从库




MySQL传统主从同步之不停机重建主从同步（实战）
https://note.youdao.com/s/bHvxKKyK
