---
title: 12MySQL + Atlas - 读写分离实战
tags:
  - mysql
  - atlas
---

## MySQL + Atlas --- 读写分离实战

1. 数据库用户配置
2. 主从数据库配置
3. Atlas配置
4. 读写分离测试

**序章**
Atlas是360团队弄出来的一套基于MySQL-Proxy基础之上的代理，修改了MySQL-Proxy的一些BUG，并且优化了很多东西。而且安装方便。配置的注释很详细，都是中文。英文不好的同学有福了。  
Atlas官方链接： https://github.com/Qihoo360/Atlas/blob/master/README_ZH.md  
Atlas下载链接： https://github.com/Qihoo360/Atlas/releases  

环境：

|    系统   |        IP     |     角色      |
|    ---    |       ---     |     ---      | 
| CentOS7.4 | 192.168.73.20 | Atlas代理服务 |
| RHEL 7.3  | 192.168.73.21 | MySQL_Master |
| RHEL 7.3  | 192.168.73.22 | MySQL_Slave  |
**注意：**三台服务器都要安装MySQL数据！

## MySQL Master

```ini
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

bind-address    = 0.0.0.0

user=mysql

innodb_flush_log_at_trx_commit=1
sync_binlog=1
log-bin=mysql-bin
server-id=1
binlog-do-db=testdb
binlog-ignore-db=mysql
skip-name-resolve

general_log=ON
general_log_file= /var/lib/mysql/rhel_sqlsrv.log
```

```sql
CREATE DATABASE testdb DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

mysql> grant replication slave on *.* to root@'%' identified by 'ccc.456';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> select user,host from mysql.user;
+---------------+--------------+
| user          | host         |
+---------------+--------------+
| root          | %            |
| slave         | 192.168.73.% |
| mysql.session | localhost    |
| mysql.sys     | localhost    |
| root          | localhost    |
+---------------+--------------+
5 rows in set (0.00 sec)

mysql>

mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |     1605 | testdb       | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>
```

```shell
[root@rhel7 ~]& mysqldump -uroot -p -B testdb > testdb.sql
Enter password:
[root@rhel7 ~]& ls
anaconda-ks.cfg  ha.sql  mysql_master_ha  rpm_mysql5.7  testdb.sql
[root@rhel7 ~]& scp testdb.sql root@192.168.73.22:~
testdb.sql                                                 100% 2126     2.1KB/s   00:00
[root@rhel7 ~]&
```


## MySQL Slave

```ini
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

bind-address    = 0.0.0.0

log-bin=mysql-slave2
server-id=3
binlog-ignore-db=mysql
log_slave_updates=1
skip-name-resolve
general_log=ON
general_log_file=/var/lib/mysql/rhel_sqlsrv.log
```

```sql
mysql> change master to master_host='192.168.73.21', master_user='root',master_password='ccc.456',master_port=3306,master_log_file='mysql-bin.000003',master_log_pos=1605,master_connect_retry=10;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.73.21
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 1605
               Relay_Log_File: rhel_sqlsrv-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000003
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
          Exec_Master_Log_Pos: 1605
              Relay_Log_Space: 533
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
             Master_Server_Id: 1
                  Master_UUID: 8f5a42da-cd2d-11ec-8c02-000c29f9275c
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
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

mysql>
```

```shell
[root@rhel_sqlsrv ~]& mysql -uroot -p < testdb.sql
Enter password:
[root@rhel_sqlsrv ~]&
```

## 测试主从同步

**Master**
```sql
Database changed
mysql> insert into book values('abc',123,456);
Query OK, 1 row affected (0.00 sec)

mysql>
```

**Slave**
```sql
mysql> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
| Linux                   |    66 |   666 |
| Unix                    |    77 |   777 |
| abc                     |   123 |   456 |
+-------------------------+-------+-------+
7 rows in set (0.00 sec)

mysql>
```

## Atlas配置


**安装Atlas**
```shell
[root@ctos7mini ~]& rpm -ivh Atlas-2.2.1.el6.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:Atlas-2.2.1-1                    ################################# [100%]
[root@ctos7mini ~]&
```

```mbox
[root@ctos7mini ~]& ll /usr/local/mysql-proxy/ -R
/usr/local/mysql-proxy/:
total 4
drwxr-xr-x. 2 root root   75 May  7 04:33 bin
drwxr-xr-x. 2 root root   22 May  7 04:33 conf
drwxr-xr-x. 3 root root 4096 May  7 04:33 lib
drwxr-xr-x. 2 root root    6 Dec 17  2014 log

/usr/local/mysql-proxy/bin:
total 44
-rwxr-xr-x. 1 root root  9696 Dec 17  2014 encrypt
-rwxr-xr-x. 1 root root 23564 Dec 17  2014 mysql-proxy
-rwxr-xr-x. 1 root root  1552 Dec 17  2014 mysql-proxyd
-rw-r--r--. 1 root root     6 Dec 17  2014 VERSION

/usr/local/mysql-proxy/conf:
total 4
-rw-r--r--. 1 root root 2810 Dec 17  2014 test.cnf

/usr/local/mysql-proxy/lib:
total 9812
-rwxr-xr-x. 1 root root  968655 Dec 17  2014 libevent-2.0.so.5
-rwxr-xr-x. 1 root root 3464668 Dec 17  2014 libglib-2.0.so.0
-rwxr-xr-x. 1 root root   35483 Dec 17  2014 libgmodule-2.0.so.0
-rwxr-xr-x. 1 root root    8656 Dec 17  2014 libgthread-2.0.so.0
-rwxr-xr-x. 1 root root 5021619 Dec 17  2014 libjemalloc.so.1
-rwxr-xr-x. 1 root root  172984 Dec 17  2014 liblua-5.1.so
-rwxr-xr-x. 1 root root   14735 Dec 17  2014 libmysql-chassis-glibext.so.0
-rwxr-xr-x. 1 root root   73823 Dec 17  2014 libmysql-chassis.so.0
-rwxr-xr-x. 1 root root   14446 Dec 17  2014 libmysql-chassis-timing.so.0
-rwxr-xr-x. 1 root root  180107 Dec 17  2014 libmysql-proxy.so.0
-rwxr-xr-x. 1 root root   71566 Dec 17  2014 libsql-tokenizer.so.0
drwxr-xr-x. 4 root root      32 May  7 04:33 mysql-proxy

/usr/local/mysql-proxy/lib/mysql-proxy:
total 0
drwxr-xr-x. 3 root root 162 May  7 04:33 lua
drwxr-xr-x. 2 root root  44 May  7 04:33 plugins

/usr/local/mysql-proxy/lib/mysql-proxy/lua:
total 240
-rw-r--r--. 1 root root   9387 Dec 17  2014 admin.lua
-rwxr-xr-x. 1 root root  15966 Dec 17  2014 chassis.so
-rwxr-xr-x. 1 root root   8988 Dec 17  2014 crc32.so
-rwxr-xr-x. 1 root root   9416 Dec 17  2014 glib2.so
-rwxr-xr-x. 1 root root  18323 Dec 17  2014 lfs.so
-rwxr-xr-x. 1 root root  43879 Dec 17  2014 lpeg.so
-rwxr-xr-x. 1 root root 102944 Dec 17  2014 mysql.so
-rwxr-xr-x. 1 root root   9248 Dec 17  2014 posix.so
drwxr-xr-x. 2 root root    243 May  7 04:33 proxy
-rwxr-xr-x. 1 root root   6626 Dec 17  2014 time.so

/usr/local/mysql-proxy/lib/mysql-proxy/lua/proxy:
total 80
-rw-r--r--. 1 root root   536 Dec 17  2014 auth.lua
-rw-r--r--. 1 root root  6670 Dec 17  2014 auto-config.lua
-rw-r--r--. 1 root root  2807 Dec 17  2014 balance.lua
-rw-r--r--. 1 root root  2873 Dec 17  2014 charset.lua
-rw-r--r--. 1 root root  3477 Dec 17  2014 commands.lua
-rw-r--r--. 1 root root   238 Dec 17  2014 crc32.lua
-rw-r--r--. 1 root root  1906 Dec 17  2014 filter.lua
-rw-r--r--. 1 root root  2706 Dec 17  2014 log.lua
-rw-r--r--. 1 root root  6105 Dec 17  2014 parser.lua
-rw-r--r--. 1 root root 14816 Dec 17  2014 split.lua
-rw-r--r--. 1 root root  6366 Dec 17  2014 test.lua
-rw-r--r--. 1 root root   106 Dec 17  2014 ticker.lua
-rw-r--r--. 1 root root  6215 Dec 17  2014 tokenizer.lua


#日志级别，分为message、warning、critical、error、debug五个级别
/usr/local/mysql-proxy/lib/mysql-proxy/plugins:
total 4920
-rwxr-xr-x. 1 root root   21134 Dec 17  2014 libadmin.so
-rwxr-xr-x. 1 root root 5011799 Dec 17  2014 libproxy.so

/usr/local/mysql-proxy/log:
total 0
```
**bin目录下放的都是可执行文件**

1. “encrypt”是用来对MySQL密码进行加密的，在配置的时候会用到
2. “mysql-proxy”是MySQL自己的读写分离代理
3. “mysql-proxyd”是360弄出来的，后面有个“d”，服务的启动、重启、停止，都是用他来执行的

**conf目录下放的是配置文件**

1. “test.cnf”只有一个文件，用来配置代理的，可以使用vim来编辑
2. lib目录下放的是一些包，以及Atlas的依赖
2. log目录下放的是日志，如报错等错误信息的记录

进入bin目录，使用encrypt来对数据库的密码进行加密，我的MySQL数据的用户名是root，密码是123456，我需要对密码进行加密
```shell
[root@ctos7mini ~]& cd /usr/local/mysql-proxy/
[root@ctos7mini mysql-proxy]& ./bin/encrypt 123456
/iZxz+0GRoA=

```

**配置Atlas，使用vim进行编辑**
```shell
[root@ctos7mini ~]& cat /usr/local/mysql-proxy/conf/test.cnf
[mysql-proxy]

#带#号的为非必需的配置项目

#管理接口的用户名
admin-username = root

#管理接口的密码
admin-password = pwd

#Atlas后端连接的MySQL主库的IP和端口，可设置多项，用逗号分隔
proxy-backend-addresses = 192.168.73.21:3306

#Atlas后端连接的MySQL从库的IP和端口，@后面的数字代表权重，用来作负载均衡，若省略则默认为1，可设置多项，用逗号分隔
proxy-read-only-backend-addresses = 192.168.73.22:3306@1

#用户名与其对应的加密过的MySQL密码，密码使用PREFIX/bin目录下的加密程序encrypt加密，下行的user1和user2为示例，将其替换为你的MySQL的用户名和加密密码！
pwds = root:/iZxz+0GRoA=

#设置Atlas的运行方式，设为true时为守护进程方式，设为false时为前台方式，一般开发调试时设为false，线上运行时设为true,true后面不能有空格。
daemon = true

#设置Atlas的运行方式，设为true时Atlas会启动两个进程，一个为monitor，一个为worker，monitor在worker意外退出后会自动将其重启，设为false时只有worker，没有monitor，一般开发调试时设为false，线上运行时设为true,true后面不能有空格。
keepalive = true

#工作线程数，对Atlas的性能有很大影响，可根据情况适当设置
event-threads = 8

#日志级别，分为message、warning、critical、error、debug五个级别
log-level = message

#日志存放的路径
log-path = /usr/local/mysql-proxy/log

#SQL日志的开关，可设置为OFF、ON、REALTIME，OFF代表不记录SQL日志，ON代表记录SQL日志，REALTIME 代表记录SQL日志且实时写入磁盘，默认为OFF
#sql-log = OFF

#慢日志输出设置。当设置了该参数时，则日志只输出执行时间超过sql-log-slow（单位：ms)的日志记录 。不设置该参数则输出全部日志。
#sql-log-slow = 10

#实例名称，用于同一台机器上多个Atlas实例间的区分
#instance = test

#Atlas监听的工作接口IP和端口
proxy-address = 0.0.0.0:1234

#Atlas监听的管理接口IP和端口
admin-address = 0.0.0.0:2345

#分表设置，此例中person为库名，mt为表名，id为分表字段，3为子表数量，可设置多项，以逗号分隔， 若不分表则不需要设置该项
#tables = person.mt.id.3

#默认字符集，设置该项后客户端不再需要执行SET NAMES语句
#charset = utf8

#允许连接Atlas的客户端的IP，可以是精确IP，也可以是IP段，以逗号分隔，若不设置该项则允许所有IP 连接，否则只允许列表中的IP连接
#client-ips = 127.0.0.1, 192.168.1

#Atlas前面挂接的LVS的物理网卡的IP(注意不是虚IP)，若有LVS且设置了client-ips则此项必须设置，否 则可以不设置
#lvs-ips = 192.168.1.1
[root@ctos7mini ~]&
```

**启动Atlas**

```shell
[root@ctos7mini bin]& cd /usr/local/mysql-proxy/bin/
[root@ctos7mini bin]& ./mysql-proxyd test status
MySQL-Proxy of test is NOT running
[root@ctos7mini bin]& ./mysql-proxyd test start
OK: MySQL-Proxy of test is started
[root@ctos7mini bin]& ./mysql-proxyd test status
MySQL-Proxy of test is running (13844)
MySQL-Proxy of test is running (13846)
[root@ctos7mini bin]& systemctl stop mysqld
```

```sql
[root@ctos7mini bin]& mysql -h127.0.0.1 -uroot -P2345 -uroot -ppwd
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.0.99-agent-admin

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
ERROR 1105 (07000): use 'SELECT * FROM help' to see the supported commands
mysql> show databases;
ERROR 1105 (07000): use 'SELECT * FROM help' to see the supported commands
mysql> help

For information about MySQL products and services, visit:
   http://www.mysql.com/
For developer information, including the MySQL Reference Manual, visit:
   http://dev.mysql.com/
To buy MySQL Enterprise support, training, or other products, visit:
   https://shop.mysql.com/

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.
resetconnection(\x) Clean session context.

For server side help, type 'help contents'

mysql> select * from help;
+----------------------------+---------------------------------------------------------+
| command                    | description                                             |
+----------------------------+---------------------------------------------------------+
| SELECT * FROM help         | shows this help                                         |
| SELECT * FROM backends     | lists the backends and their state                      |
| SET OFFLINE $backend_id    | offline backend server, $backend_id is backend_ndx's id |
| SET ONLINE $backend_id     | online backend server, ...                              |
| ADD MASTER $backend        | example: "add master 127.0.0.1:3306", ...               |
| ADD SLAVE $backend         | example: "add slave 127.0.0.1:3306", ...                |
| REMOVE BACKEND $backend_id | example: "remove backend 1", ...                        |
| SELECT * FROM clients      | lists the clients                                       |
| ADD CLIENT $client         | example: "add client 192.168.1.2", ...                  |
| REMOVE CLIENT $client      | example: "remove client 192.168.1.2", ...               |
| SELECT * FROM pwds         | lists the pwds                                          |
| ADD PWD $pwd               | example: "add pwd user:raw_password", ...               |
| ADD ENPWD $pwd             | example: "add enpwd user:encrypted_password", ...       |
| REMOVE PWD $pwd            | example: "remove pwd user", ...                         |
| SAVE CONFIG                | save the backends to config file                        |
| SELECT VERSION             | display the version of Atlas                            |
+----------------------------+---------------------------------------------------------+
16 rows in set (0.00 sec)

mysql> ^DBye
```

```sql
[root@ctos7mini bin]& mysql -h127.0.0.1 -uroot -P1234 -uroot -pccc.456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.0.81-log MySQL Community Server (GPL)

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
| ha                 |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
+--------------------+
6 rows in set (0.00 sec)

mysql> ^DBye
[root@ctos7mini bin]& mysql -h127.0.0.1 -uroot -P1234 -uroot -pccc.456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.0.81-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ^DBye
[root@ctos7mini bin]& mysql -h127.0.0.1 -uroot -P1234 -uroot -pccc.456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.0.81-log MySQL Community Server (GPL)

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
| ha                 |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
+--------------------+
6 rows in set (0.00 sec)

mysql> show variables like "%general_log%";
+------------------+--------------------------------+
| Variable_name    | Value                          |
+------------------+--------------------------------+
| general_log      | OFF                            |
| general_log_file | /var/lib/mysql/rhel_sqlsrv.log |
+------------------+--------------------------------+
2 rows in set (0.00 sec)

mysql> ^DBye
[root@ctos7mini bin]& mysql -h127.0.0.1 -uroot -P1234 -uroot -pccc.456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.0.81-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like "%general_log%";
+------------------+--------------------------------+
| Variable_name    | Value                          |
+------------------+--------------------------------+
| general_log      | OFF                            |
| general_log_file | /var/lib/mysql/rhel_sqlsrv.log |
+------------------+--------------------------------+
2 rows in set (0.01 sec)

mysql> ^DBye
[root@ctos7mini bin]& mysql -h127.0.0.1 -uroot -P2345 -uroot -pccc.456
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1045 (28000): password doesn't match
[root@ctos7mini bin]& mysql -h127.0.0.1 -uroot -P1234 -uroot -ppwd
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1045 (28000): Access denied for user 'root'@'127.0.0.1:37884' (using password: YES)
[root@ctos7mini bin]& mysql -h127.0.0.1 -P2345 -uroot -ppwd
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.0.99-agent-admin

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
ERROR 1105 (07000): use 'SELECT * FROM help' to see the supported commands
mysql> ^DBye
[root@ctos7mini bin]& mysql -h127.0.0.1 -P1234 -uroot -pccc.456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.0.81-log MySQL Community Server (GPL)

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
| ha                 |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
+--------------------+
6 rows in set (0.00 sec)

mysql> ^DBye
```

## 读写分离测试
 
**在MySQL主库和从库开启general_log日志，注意主库和从库general_log日志名称不同！**

临时生效
```sql
mysql> show variables like '%general_log%';
+------------------+--------------------------------+
| Variable_name    | Value                          |
+------------------+--------------------------------+
| general_log      | OFF                            |
| general_log_file | /var/lib/mysql/rhel_sqlsrv.log |
+------------------+--------------------------------+
2 rows in set (0.00 sec)

mysql>
mysql> set global general_log=ON;
Query OK, 0 rows affected (0.00 sec)
```
永久生效
```shell
[root@rhel7 ~]& echo "general_log=ON" >> /etc/my.cnf
[root@rhel7 ~]& echo "general_log_file=/var/lib/mysql/rhel.log" >> /etc/my.cnf

[root@rhel_sqlsrv ~]& echo "general_log=ON" >> /etc/my.cnf
[root@rhel_sqlsrv ~]& echo "general_log_file=/var/lib/mysql/rhel_sqlsrv.log" >> /etc/my.cnf
[root@rhel_sqlsrv ~]& systemctl restart mysqld
```

```sql
mysql> show variables like "%general_log%";
+------------------+--------------------------+
| Variable_name    | Value                    |
+------------------+--------------------------+
| general_log      | ON                       |
| general_log_file | /var/lib/mysql/rhel7.log |
+------------------+--------------------------+
2 rows in set (0.00 sec)

mysql> ^DBye
```

**清空数据库日志**：

```shell
[root@rhel7 ~]& echo > /var/lib/mysql/rhel7.log

[root@rhel_sqlsrv ~]& echo > /var/lib/mysql/rhel_sqlsrv.log
```

**测试**：

Master（rw）:
```shell
[root@rhel7 ~]& tail -f /var/lib/mysql/rhel7.log
```

Slave（ro）:
```shell
[root@rhel_sqlsrv ~]& tail -f /var/lib/mysql/rhel_sqlsrv.log
```

Atlas:
```sql
[root@ctos7mini mysql-proxy]# mysql -h127.0.0.1 -P1234 -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.0.81-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
ro:  
slave检测到读操作
```mbox
[root@rhel_sqlsrv ~]& tail -f /var/lib/mysql/rhel_sqlsrv.log

2022-05-07T09:07:23.252345Z        13 Connect   root@192.168.73.20 on  using TCP/IP
2022-05-07T09:07:23.252973Z        13 Query     SET CHARACTER_SET_CONNECTION=utf8
2022-05-07T09:07:23.254053Z        13 Query     SET CHARACTER_SET_RESULTS=utf8
2022-05-07T09:07:23.258100Z        13 Query     SET CHARACTER_SET_CLIENT=utf8
2022-05-07T09:07:23.259321Z        13 Query     select @@version_comment limit 1

```
rw:
```mbox
[root@rhel7 ~]# tail -f /var/lib/mysql/rhel7.log


```

在Atlas进行读操作:
```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| ha                 |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
+--------------------+
6 rows in set (0.00 sec)

mysql> use testdb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_testdb |
+------------------+
| book             |
+------------------+
1 row in set (0.00 sec)

mysql> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
| Linux                   |    66 |   666 |
| Unix                    |    77 |   777 |
| abc                     |   123 |   456 |
| def                     |    83 |   364 |
+-------------------------+-------+-------+
8 rows in set (0.01 sec)

mysql>
```

Slave检测到读操作：
```mbox
[root@rhel_sqlsrv ~]# tail -f /var/lib/mysql/rhel_sqlsrv.log

2022-05-07T09:07:23.252345Z        13 Connect   root@192.168.73.20 on  using TCP/IP
2022-05-07T09:07:23.252973Z        13 Query     SET CHARACTER_SET_CONNECTION=utf8
2022-05-07T09:07:23.254053Z        13 Query     SET CHARACTER_SET_RESULTS=utf8
2022-05-07T09:07:23.258100Z        13 Query     SET CHARACTER_SET_CLIENT=utf8
2022-05-07T09:07:23.259321Z        13 Query     select @@version_comment limit 1
2022-05-07T09:13:04.634499Z        13 Query     show databases
2022-05-07T09:13:09.313174Z        13 Query     SELECT DATABASE()
2022-05-07T09:13:09.314061Z        13 Init DB   testdb
2022-05-07T09:13:09.316061Z        13 Init DB   testdb
2022-05-07T09:13:09.316710Z        13 Query     show databases
2022-05-07T09:13:09.317592Z        13 Init DB   testdb
2022-05-07T09:13:09.318360Z        13 Query     show tables
2022-05-07T09:13:09.319468Z        13 Init DB   testdb
2022-05-07T09:13:09.320217Z        13 Field List        book
2022-05-07T09:13:16.960823Z        13 Init DB   testdb
2022-05-07T09:13:16.962628Z        13 Query     show tables
2022-05-07T09:13:20.799873Z        13 Init DB   testdb
2022-05-07T09:13:20.800737Z        13 Query     select * from book

```

在Atlas进行写操作：
```sql
mysql> insert into book values('test', 67, 252);
Query OK, 1 row affected (0.01 sec)

mysql>
```

Master检测到写操作：
```mbox
[root@rhel7 ~]# tail -f /var/lib/mysql/rhel7.log

2022-05-07T10:59:33.029685Z         4 Connect   root@192.168.73.20 on  using TCP/IP
2022-05-07T10:59:33.030609Z         4 Query     SET CHARACTER_SET_CONNECTION=utf8
2022-05-07T10:59:33.031735Z         4 Query     SET CHARACTER_SET_RESULTS=utf8
2022-05-07T10:59:33.032656Z         4 Query     SET CHARACTER_SET_CLIENT=utf8
2022-05-07T10:59:33.033919Z         4 Init DB   testdb
2022-05-07T10:59:33.035018Z         4 Query     insert into book values('test', 67, 252)

```


## 故障解决

**故障一**

**Atlas**访问工作接口时，数据库只有一个`information_schema`，不显示其他库：

```mbox
[root@ctos7mini mysql-proxy]# mysql -h192.168.73.21 -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
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
+--------------------+
1 row in set (0.00 sec)

mysql> ^DBye
```

故障分析：

1. 主从不同步：  
查看Slave的状态：
```sql
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: 192.168.73.21
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 154
               Relay_Log_File: rhel_sqlsrv-relay-bin.000012
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Connecting
            Slave_SQL_Running: Yes
.........................................省略...................................................................
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 2003
                Last_IO_Error: error connecting to master 'root@192.168.73.21:3306' - retry-time: 10  retries: 1
 .........................................省略...................................................................
1 row in set (0.00 sec)
```

2. `Atlas`没有连接到主从  
访问Atlas管理接口查看主从服务是否都在线
```sql
[root@ctos7mini mysql-proxy]# mysql -h127.0.0.1 -P2345 -uroot -ppwd
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.0.99-agent-admin

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> select * from backends;
+-------------+--------------------+-------+------+
| backend_ndx | address            | state | type |
+-------------+--------------------+-------+------+
|           1 | 192.168.73.21:3306 | up    | rw   |
|           2 | 192.168.73.22:3306 | down  | ro   |
+-------------+--------------------+-------+------+
2 rows in set (0.00 sec)

mysql>

```
根据实际环境检查，主从是否同步，MySQL是否允许远程连接，数据库端口是否开放。

**故障二：**

数据库能够查询，不能进行写操作：

```sql
mysql> use testdb
Database changed
mysql> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
| Linux                   |    66 |   666 |
| Unix                    |    77 |   777 |
| abc                     |   123 |   456 |
| def                     |    83 |   364 |
+-------------------------+-------+-------+
8 rows in set (0.00 sec)

mysql> insert into `book` values('test',24,62);
ERROR 1046 (3D000): No database selected
mysql>
```

Master日志：
```mbox
2022-05-07T09:24:33.993274Z        17 Query     insert into `book` values('test',24,62)
2022-05-07T09:24:39.346670Z        17 Init DB   Access denied for user 'root'@'%' to database 'testdb'
```

> 重启操作系统
