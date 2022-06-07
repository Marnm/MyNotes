---
title: 07MySQL 备份与恢复（删库不跑路）
tags:
  - mysql
  - sqldump
  - mysqlcheck
  - mysqlhotcopy
  - xtrabackup
---

### 一：MySQL字符集

**1：字符集介绍**
字符集就是一套文字符号及其编码、比较规则的集合，第一个计算机字符集ASC2；MySQL数据库字符集包括字符集（CHARACTER）和校对规则（COLLATION）两个概念，其中字符集用来定义MySQL数据字符串的存储方式，而校对规则定义比较字符串的方式

**2：MySQL数据库常见字符集介绍**
      
选择字符集建议使用国际标准的utf8

**3：MySQL怎样选择合适的字符集**  
1、如果处理各种各样的文字，发布到不同语言国家地区，应选Unicode字符集，对MySQL来说就是utf8(每个汉字三个字节)  
2、如果只是需要支持中文，并且数据量很大，性能要求也高，可选GBK(定长，每个汉字占双字节，英文也占双字节)，如果是大量运算，比较排序等，定长字符集更快，性能也高  
3、处理移动互联网业务，可能需要使用utf8mb4字符集


**4：查看当前MySQL支持的字符集**

MySQL可以支持多种字符集，同一台服务器，库或表的不同字段都可以指定不同的字符集

    mysql -uroot -pJaking@vip.163.com -e "show character set \G;"  #查看所有的字符集

查看常用的字符集：

    mysql -uroot -pJaking@vip.163.com -e "show character set\G;" |egrep "gbk|utf8|latin1"|awk '{print $0}'

```shell
[root@jaking ~]# mysql -uroot -pJaking@vip.163.com -e "show character set\G;" | egrep "gbk|utf8|latin1"|awk '{print $0}'
mysql: [Warning] Using a password on the command line interface can be insecure.
          Charset: latin1
Default collation: latin1_swedish_ci
          Charset: gbk
Default collation: gbk_chinese_ci
          Charset: utf8
Default collation: utf8_general_ci
          Charset: utf8mb4
Default collation: utf8mb4_general_ci
```
**5：查看MySQL当前的字符集设置情况**
```sql
mysql> show variables like 'character_set%';
+--------------------------+------------------------------------+
| Variable_name            | Value                              |
+--------------------------+------------------------------------+
| character_set_client     | utf8                               |
| character_set_connection | utf8                               |
| character_set_database   | utf8                               |
| character_set_filesystem | binary                             |
| character_set_results    | utf8                               |
| character_set_server     | utf8                               |
| character_set_system     | utf8                               |
| character_sets_dir       | /usr/local/mysql57/share/charsets/ |
+--------------------------+------------------------------------+
8 rows in set (0.00 sec)
mysql> show character set;
+----------+---------------------------------+---------------------+--------+
| Charset  | Description                     | Default collation   | Maxlen |
+----------+---------------------------------+---------------------+--------+
| big5     | Big5 Traditional Chinese        | big5_chinese_ci     |      2 |
| dec8     | DEC West European               | dec8_swedish_ci     |      1 |
| cp850    | DOS West European               | cp850_general_ci    |      1 |
| hp8      | HP West European                | hp8_english_ci      |      1 |
| koi8r    | KOI8-R Relcom Russian           | koi8r_general_ci    |      1 |
| latin1   | cp1252 West European            | latin1_swedish_ci   |      1 |
| latin2   | ISO 8859-2 Central European     | latin2_general_ci   |      1 |
| swe7     | 7bit Swedish                    | swe7_swedish_ci     |      1 |
| ascii    | US ASCII                        | ascii_general_ci    |      1 |
| ujis     | EUC-JP Japanese                 | ujis_japanese_ci    |      3 |
| sjis     | Shift-JIS Japanese              | sjis_japanese_ci    |      2 |
| hebrew   | ISO 8859-8 Hebrew               | hebrew_general_ci   |      1 |
| tis620   | TIS620 Thai                     | tis620_thai_ci      |      1 |
| euckr    | EUC-KR Korean                   | euckr_korean_ci     |      2 |
| koi8u    | KOI8-U Ukrainian                | koi8u_general_ci    |      1 |
| gb2312   | GB2312 Simplified Chinese       | gb2312_chinese_ci   |      2 |
| greek    | ISO 8859-7 Greek                | greek_general_ci    |      1 |
| cp1250   | Windows Central European        | cp1250_general_ci   |      1 |
| gbk      | GBK Simplified Chinese          | gbk_chinese_ci      |      2 |
| latin5   | ISO 8859-9 Turkish              | latin5_turkish_ci   |      1 |
| armscii8 | ARMSCII-8 Armenian              | armscii8_general_ci |      1 |
| utf8     | UTF-8 Unicode                   | utf8_general_ci     |      3 |
| ucs2     | UCS-2 Unicode                   | ucs2_general_ci     |      2 |
| cp866    | DOS Russian                     | cp866_general_ci    |      1 |
| keybcs2  | DOS Kamenicky Czech-Slovak      | keybcs2_general_ci  |      1 |
| macce    | Mac Central European            | macce_general_ci    |      1 |
| macroman | Mac West European               | macroman_general_ci |      1 |
| cp852    | DOS Central European            | cp852_general_ci    |      1 |
| latin7   | ISO 8859-13 Baltic              | latin7_general_ci   |      1 |
| utf8mb4  | UTF-8 Unicode                   | utf8mb4_general_ci  |      4 |
| cp1251   | Windows Cyrillic                | cp1251_general_ci   |      1 |
| utf16    | UTF-16 Unicode                  | utf16_general_ci    |      4 |
| utf16le  | UTF-16LE Unicode                | utf16le_general_ci  |      4 |
| cp1256   | Windows Arabic                  | cp1256_general_ci   |      1 |
| cp1257   | Windows Baltic                  | cp1257_general_ci   |      1 |
| utf32    | UTF-32 Unicode                  | utf32_general_ci    |      4 |
| binary   | Binary pseudo charset           | binary              |      1 |
| geostd8  | GEOSTD8 Georgian                | geostd8_general_ci  |      1 |
| cp932    | SJIS for Windows Japanese       | cp932_japanese_ci   |      2 |
| eucjpms  | UJIS for Windows Japanese       | eucjpms_japanese_ci |      3 |
| gb18030  | China National Standard GB18030 | gb18030_chinese_ci  |      4 |
+----------+---------------------------------+---------------------+--------+
41 rows in set (0.00 sec)
``` 
 
**改字符编码**
```shell 
[root@jaking ~]# echo character_set_server=utf8 >> /etc/my.cnf
[root@jaking ~]# cat /etc/my.cnf
[client]
port=3306
socket=/data/mysql/mysql.sock
 
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci
 
skip-name-resolve
user=mysql
port=3306
basedir=/usr/local/mysql57
datadir=/data/mysql
tmpdir=/tmp
socket=/data/mysql/mysql.sock
 
log-error=/data/mysql/mysqld.log
pid-file=/data/mysql/mysqld.pid 
default-storage-engine=InnoDB
character_set_server=utf8
[root@jaking ~]# systemctl restart mysqld
[root@jaking ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.19 Source distribution
 
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
```sql
mysql> show variables like 'character_set%';
+--------------------------+------------------------------------+
| Variable_name            | Value                              |
+--------------------------+------------------------------------+
| character_set_client     | utf8                               |
| character_set_connection | utf8                               |
| character_set_database   | utf8                               |
| character_set_filesystem | binary                             |
| character_set_results    | utf8                               |
| character_set_server     | utf8                               |
| character_set_system     | utf8                               |
| character_sets_dir       | /usr/local/mysql57/share/charsets/ |
+--------------------------+------------------------------------+
8 rows in set (0.00 sec)

mysql> create database testdb;     
Query OK, 1 row affected (0.00 sec)

mysql> show create database testdb;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| testdb   | CREATE DATABASE `testdb` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```

**6：实战：迁移数据**

 背景：公司业务数据book,由于之前建表没注意字符集的问题，导致之前写入的数据出现乱码；现在要将之前的数据和现在数据的字符集一致，不出现乱码情况；将字符集为latin1已有记录的数据转成utf8，并且已经存在的记录不乱码

步骤
                 1：建库及建表的语句导出，sed批量修改为utf8
                 2：导出之前所有的数据
                 3：修改mysql服务端和客户端编码为utf8
                 4：删除原有的库表及数据
                 5：导入新的建库及建表语句
                 6：导入之前的数据
 
 准备创建测试库和表的SQL语句
```sql
CREATE DATABASE testbookdb;
#CREATE DATABASE `testbookdb` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
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

CREATE TABLE book1 (bTypeId int,bName char(16),price int,publishing char(16));
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('1','Linux','66','DZ');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('2','CLD','68','RM');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('3','OS','90','JX');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('4','MySQL1','71','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('5','MySQL2','72','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('6','MySQL3','73','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('7','MySQL4','74','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('8','MySQL5','75','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('9','MySQL6','76','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('10','MySQL7','77','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('11','MySQL8','78','QH');
INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('12','MySQL9','79','QH');

mysql> create database book;
Query OK, 1 row affected (0.00 sec)

mysql> show create database book;
+----------+---------------------------------------------------------------+
| Database | Create Database                                               |
+----------+---------------------------------------------------------------+
| book     | CREATE DATABASE `book` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+---------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> use book;
Database changed

mysql> CREATE TABLE book1 (bTypeId int,bName char(16),price int,publishing char(16)) character set latin1;
Query OK, 0 rows affected (0.00 sec)
 
mysql> show CREATE TABLE book1;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                          |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book1 | CREATE TABLE `book1` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.02 sec)
```

**修改数据库字符集**
```sql
mysql> show create database book;
+----------+---------------------------------------------------------------+
| Database | Create Database                                               |
+----------+---------------------------------------------------------------+
| book     | CREATE DATABASE `book` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+---------------------------------------------------------------+
1 row in set (0.00 sec)
 
mysql> alter database book default character set latin1;
Query OK, 1 row affected (0.00 sec)
 
mysql> show create database book;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| book     | CREATE DATABASE `book` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+----------+-----------------------------------------------------------------+
1 row in set (0.01 sec)
```

**修改表的字符集**
 ```sql
mysql> show create table book1;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                        |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book1 | CREATE TABLE `book1` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
 
mysql> alter table book1 default character set latin1;
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0
 
mysql> show create table book1;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book1 | CREATE TABLE `book1` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) CHARACTER SET utf8 DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) CHARACTER SET utf8 DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

**深入研究字符集：**
```sql
mysql> drop table book1;
Query OK, 0 rows affected (0.00 sec)
mysql> CREATE TABLE book1 (bTypeId int,bName char(16),price int,publishing char(16)) character set latin1;
Query OK, 0 rows affected (0.01 sec)
 
mysql> INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('1','《Linux从入门到精通》','66','电子工业出版社');
ERROR 1366 (HY000): Incorrect string value: '\xE3\x80\x8ALin...' for column 'bName' at row 1
mysql> drop table book1;
Query OK, 0 rows affected (0.00 sec)
 
mysql>  CREATE TABLE book1 (bTypeId int,bName char(16),price int,publishing char(16));
Query OK, 0 rows affected (0.00 sec)
 
mysql> INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('1','《Linux从入门到精通》','66','电子工业出版社');
ERROR 1366 (HY000): Incorrect string value: '\xE3\x80\x8ALin...' for column 'bName' at row 1
mysql> show create table book1;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                          |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book1 | CREATE TABLE `book1` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
 
mysql> drop table book1;                                                        
Query OK, 0 rows affected (0.01 sec)
 
mysql>  CREATE TABLE book1 (bTypeId int,bName char(16),price int,publishing char(16)) character set utf8;
Query OK, 0 rows affected (0.01 sec)
 
mysql> INSERT INTO book1(bTypeId,bName,price,publishing) VALUES('1','《Linux从入门到精通》','66','电子工业出版社');
Query OK, 1 row affected (0.00 sec)
 
mysql> select * from book1;
+---------+-------------------------------+-------+-----------------------+
| bTypeId | bName                         | price | publishing            |
+---------+-------------------------------+-------+-----------------------+
|       1 | 《Linux从入门到精通》         |    66 | 电子工业出版社        |
+---------+-------------------------------+-------+-----------------------+
1 row in set (0.00 sec)

mysql> show tables;
+-----------------------+
| Tables_in_testbookdb2 |
+-----------------------+
| book1                 |
+-----------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE book2 (bTypeId int,bName char(16),price int,publishing char(16)) character set utf8 COLLATE utf8_general_ci;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO book2(bTypeId,bName,price,publishing) VALUES('1','《Linux从入门到精通》','66','电子工业出版社');
Query OK, 1 row affected (0.00 sec)

mysql> show create table book2;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                        |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book2 | CREATE TABLE `book2` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> show create table book1;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                  |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book1 | CREATE TABLE `book1` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) CHARACTER SET latin1 DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) CHARACTER SET latin1 DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> 
```

**1）：导出表结构**

        mysqldump -uroot -pJaking@vip.163.com --default-character-set=latin1 -d  testbookdb > booktable.sql 

**2）: 编辑booktable.sql 将latin1修改成utf8**

        vim booktable.sql 修改所有latin1为utf8

**3）: 确保数据库不再更新，导出指定库的所有数据**
```shell
 mysqldump -uroot -pJaking@vip.163.com --quick --no-create-info --extended-insert --default-character-set=latin1 --databases  testbookdb > bookdata.sql
 
 mysqldump -uroot -pJaking@vip.163.com --databases  testbookdb > bookdata.sql         
 
 mysqldump -uroot -pJaking@vip.163.com --quick --no-create-info --extended-insert --default-character-set=latin1 -B  testbookdb > bookdata.sql
 
 mysqldump -uroot -pJaking@vip.163.com --quick --no-create-info --extended-insert --default-character-set=latin1 -t testbookdb > bookdata.sql
 
 mysqldump -uroot -pJaking@vip.163.com --databases  testbookdb > bookdata.sql
 
 mysqldump -uroot -pJaking@vip.163.com -t testbookdb > bookdata.sql
 
 mysqldump -uroot -pJaking@vip.163.com --quick --no-create-info --extended-insert --default-character-set=latin1  -A  > ALL.sql
 
 mysqldump -uroot -pJaking@vip.163.com --quick --no-create-info --extended-insert  -A  > ALL.sql
  
 mysqldump -uroot -pJaking@vip.163.com --quick --no-create-info --extended-insert --default-character-set=utf8  --all-databases  > ALL.sql
 
 mysqldump -uroot -pJaking@vip.163.com --all-databases  > ALL.sql
 
 mysql -uroot -pJaking@vip.163.com < ALL.sql
```

**参数说明：**
`--quick`:用于转储大的表，强制mysqldump从服务器一次一行的检索数据而不是检索所有行，并输出当前cache到内存中
`--no-create-info`:不要创建create table语句
`--extended-insert`:使用包括几个values列表的多行insert语法，这样文件更小，IO也小，导入数据时会非常快
`--default-character-set=latin1`:按照原有字符集导出数据，这样导出的文件中，所有中文都是可见的，不会保存成乱码
**4）：打开bookdata.sql 将SET NAME latin1 修改成SET NAME utf8**

                 vim bookdata.sql 
                 /*!40101 SET NAMES utf8 */;

**5）: 重新建库**
```SQL
mysql> create database  testdb default charset utf8;
```

**6）：建立表，导入我们之前导出的表的表结构**
```Shell
mysql -uroot -pJaking@vip.163.com testdb < booktable.sql
```
**7）: 导入数据**

        mysql -uroot -pJaking@vip.163.com testdb < bookdata.sql
        注意：选择目标字符集时，要注意最好大于等于原字符集（字库更大），否则可能会丢失不被支持的数据

**二：MySQL日常维护**

**1：mysqlcheck mysql修复工具**

mysqlcheck客户端工具可以检查和修复MyISAM表，还可以优化和分析表。
实际上，它集成了mysql工具中check、repair、analyze、tmpimize的功能。
/usr/local/mysql/bin/mysqlcheck   #源码编译安装位置

```Shell
rpm -qf `which mysqlcheck`  yum安装查看
[root@jaking ~]# rpm -qf `which mysqlcheck`
file /usr/local/mysql57/bin/mysqlcheck is not owned by any package
[root@jaking ~]# yum provides mysqlcheck
Loaded plugins: product-id, search-disabled-repos, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
1:mariadb-5.5.52-1.el7.x86_64 : A community developed branch of MySQL
Repo        : rhel7
Matched from:
Filename    : /usr/bin/mysqlcheck
[root@jaking ~]# yum install -y mariadb-5.5.52-1.el7.x86_64
[root@jaking ~]# rpm -qf `which mysqlcheck`
mariadb-5.5.52-1.el7.x86_64
[root@jaking ~]# which mysqlcheck
/usr/bin/mysqlcheck
```

参数选项：
mysqlcheck --help 查看帮助
`-c`, --check (检查表)；
`-r`, --repair（修复表)；
`-a`, --analyze (分析表)；
`-o`, --tmpimize(优化表)； //其中，默认选项是-c(检查表)
`-u`， 使用mysql中哪个用户进行操作

mysqlcheck使用语法：
使用以下3种方式来调用mysqlcheck：
mysqlcheck[tmpions] db_name [tables]
mysqlcheck[tmpions] ---database DB1 [DB2 DB3...]
mysqlcheck[tmpions] --all--databases
如果没有指定任何表或使用---databases(-B)或--all--databases(-A)选项，则检查整个数据库。

举例说明：

1：检查表(check)
```shell
[root@jaking ~]# mysqlcheck -uroot -pJaking@vip.163.com -c testdb book1
mysqlcheck: [Warning] Using a password on the command line interface can be insecure.
testdb.book1                                       OK
```
2:  修复表（repair）
```shell
[root@jaking ~]# mysqlcheck -uroot -pJaking@vip.163.com -r testdb book1
mysqlcheck: [Warning] Using a password on the command line interface can be insecure.
testdb.book1
note     : The storage engine for the table doesn't support repair

[root@jaking ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.7.19 Source distribution
 
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
```sql 
mysql> show create table testdb.book1;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                        |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book1 | CREATE TABLE `book1` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
 
mysql> alter table testdb.book1 engine = MyISAM;
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0
 
mysql> show create table testdb.book1;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                        |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| book1 | CREATE TABLE `book1` (
  `bTypeId` int(11) DEFAULT NULL,
  `bName` char(16) DEFAULT NULL,
  `price` int(11) DEFAULT NULL,
  `publishing` char(16) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
mysql> exit
Bye
```
```shell
[root@jaking ~]# mysqlcheck -uroot -pJaking@vip.163.com -r testdb book1
mysqlcheck: [Warning] Using a password on the command line interface can be insecure.
testdb.book1                                       OK
 
3:修复指定的数据库
[root@jaking ~]# mysqlcheck -uroot -pJaking@vip.163.com -r testdb book1
mysqlcheck: [Warning] Using a password on the command line interface can be insecure.
testdb.book1                                       OK                                 

[root@jaking ~]# mysqlcheck -uroot -p  -r --databases testdb
Enter password: 
testdb.book1                                       OK
testdb.book2
note     : The storage engine for the table doesn't support repair
```

扩展： 修复文件系统，容易掉失数据

    fsck -y -f /dev/sdbx

4:  检查修复所有数据库
```shell
[root@jaking ~]# mysqlcheck -uroot -pJaking@vip.163.com -A -r
book.book1                                         OK
book2.book1
note     : The storage engine for the table doesn't support repair
market.order
note     : The storage engine for the table doesn't support repair
market.order1
note     : The storage engine for the table doesn't support repair
market.user
note     : The storage engine for the table doesn't support repair
mysql.columns_priv                                 OK
mysql.db                                           OK
mysql.engine_cost
note     : The storage engine for the table doesn't support repair
mysql.event                                        OK
mysql.func                                         OK
mysql.gtid_executed
Warning  : Please do not modify the gtid_executed table. This is a mysql internal system table to store GTIDs for committed transactions. Modifying it can lead to an inconsistent GTID state.
note     : The storage engine for the table doesn't support repair
mysql.help_category
note     : The storage engine for the table doesn't support repair
mysql.help_keyword
note     : The storage engine for the table doesn't support repair
mysql.help_relation
note     : The storage engine for the table doesn't support repair
mysql.help_topic
note     : The storage engine for the table doesn't support repair
mysql.innodb_index_stats
note     : The storage engine for the table doesn't support repair
mysql.innodb_table_stats
note     : The storage engine for the table doesn't support repair
mysql.ndb_binlog_index                             OK
mysql.plugin
note     : The storage engine for the table doesn't support repair
mysql.proc                                         OK
mysql.procs_priv                                   OK
mysql.proxies_priv                                 OK
mysql.server_cost
```

参数-A 等于  --all-databases  

5：每天定时对mysql数据库进行优化

使用计划任务
```shell
crontab -e
#0 1 * * * /usr/bin/mysqlcheck -A -o -r  -u你的用户名 -p你的密码 > /dev/null 2>&1

0 3 * * *  /usr/bin/mysqlcheck -uroot -pJaking@vip.163.com -r -o -A > /dev/null 2>&1 #每天3点优化
```

2：MySQL备份与恢复

1)MySQL备份的类型
按照备份时对数据库的影响范围分为：
Hot backup(热备)  Cold Backup（冷备）Warm Backup（温备）
Hot backup：热备指在数据库运行中直接备份，对正在运行的数据库没有任何影响，（Online Backup）官方手册为在线备份
Cold Backup：冷备指在数据库停止的情况下进行备份，(OfflineBackup) 官方手册称为离线备份
Warm Backup：温备指备份同样在数据库运行时进行，但是会对当前数据库的操作有所影响，例如加一个全局读锁以保证备份数据的一致性

按照备份后文件内容：

逻辑备份 -->
指备份后的文件内容是可读的，通常为文本文件，内容一般是SQL语句，或者是表内的实际数据，如mysqldump和SELECT * INTO OUTFILE的方法，一般适用于数据库的升级和迁移，恢复时间较长

裸文件备份-->
拷贝数据库的物理文件，数据库既可以处于运行状态（mysqlhotcopy 、ibbackup、xtrabackup这类工具），也可以处于停止状态，恢复时间较短。

按照备份数据库的内容来分，又可以分为：
完全备份：对数据库完整的备份
增量备份：在上一次完全备份基础上，对更新的数据进行备份（xtrabackup）
日志备份：二进制日志备份，主从复制

2)逻辑备份工具mysqldump
使用的时候MySQL当要导入或者导出数据量大的库的时候,用PHPMYADMIN甚至MySQL Administrator这些工具都会力不从心,这时只能使用MySQL所提供的命令行工具mysqldump进行备份恢复；数据量大的时候不推荐使用，可支持MyISAM,InnoDB

MySQL数据的导出和导入工具: mysqldump

导出数据：
语法： mysqldump [TMPIONS] database [tables] >导出的文件名.sql

A:导出所有数据库
```shell
mysqldump -uroot -pJaking@vip.163.com -A >all.sql
mysqldump -uroot -pJaking@vip.163.com --all-databases >all2.sql
```
参数-A代表所有，等同于 --all-databases

B:导出某个数据库
```shell
mysqldump -u 用户名 -p密码 数据库名 > 导出的文件名.sql  
mysqldump -uroot -pJaking@vip.163.com testdb > testdb.sql
```
C:导出单张表
```shell
mysqldump -uroot -pJaking@vip.163.com testdb book1 > testdb.book1.sql  #导出testdb库book1表
```
D:导出库的表结构
```shell
mysqldump -uroot -pJaking@vip.163.com -d testdb > testdbtable.sql  #只导出testdb库的表结构
```
E:只导出数据
```shell
mysqldump -uroot -pJaking@vip.163.com -t testdb > testdbdata.sql  #只导出testdb库中的数据
```
F:导出数据库，并自动生成库的创建语句
```shell
mysqldump -uroot -pJaking@vip.163.com -B testdb > testdb.sql  #生成建库语句
mysql -uroot -pJaking@vip.163.com < testdb.sql  #导入不用指定数据名
```

导入数据：
A：导入所有数据库

    mysql -uroot -pJaking@vip.163.com < all.sql

B：导入数据库
```shell
mysql -uroot -pJaking@vip.163.com testdb < testdb.sql  #如果导入时，没有对应的数据库，需要你手动创建一下
```
mysql> create database testdb;

使用source导入
mysql> drop database testdb;
mysql> create database testdb;
mysql> use testdb;
mysql> source /tmp/testdb.sql
 
C:导入表
mysql> drop table testdb.book1;
mysql> source /tmp/testdb.book1.sql;   #导入表时，不需要重新创建表，要先进到相应的数据库中
mysql> select * from testdb.book1;

D：导入表结构和数据
mysql> create database testdb;
mysql -uroot -pJaking@vip.163.com testdb < testdbtable.sql
mysql -uroot -pJaking@vip.163.com testdb < testdbdata.sql

mysqldump远程备份与恢复数据

文档：10 mysqldump远程备份与恢复数据.md
链接：http://note.youdao.com/noteshare?id=7af3e7cda297defe01228ed798e72c02&sub=E2F44DF8E1BC4B9287EA434DEA3B670C

远程备份
mysqldump -h ip_addr -u db_user -p db_name > file_name.sql
远程恢复
mysql -h ip_addr -u db_user -p < db_file.sql


3) mysqlhotcopy 裸文件备份    

注意：5.7版本已经去掉此命令，请在5.5及以下版本测试！
mysqlhotcopy使用lock tables、flush tables和cp或scp来快速备份数据库，它是备份数据库或单个表最快的途径,完全属于物理备份,但只能用于备份MyISAM存储引擎和运行在数据库目录所在的机器上，与mysqldump备份不同,mysqldump属于逻辑备份,恢复时是执行的sql语句。

mysqlhotcopy本质是使用锁表语句后再使用cp或scp拷贝数据库

1.安装mysqlhotcopy

yum provides mysqlhotcopy
yum install -y mariadb-server
 
2.查看mysqlhotcopy帮助信息

mysqlhotcopy --help
  --allowold   don't abort if target dir already exists (rename it _old) --不覆盖以前备份的文件
  --addtodest  don't rename target dir if it exists, just add files to it      --属于增量备份
  --noindices   don't include full index files in copy          --不备份索引文件
  --debug     enable debug                                 --启用调试输出
  --regexp=#   copy all databases with names matching regexp  --使用正则表达式
  --checkpoint=#  insert checkpoint entry into specified db.table    --插入检查点条目
  --flushlog      flush logs once all tables are locked          --所有表锁定后刷新日志
  --resetmaster    reset the binlog once all tables are locked     --一旦锁表重置binlog文件
  --resetslave  reset the master.info once all tables are locked  --一旦锁表重置master.info文件 
 
举例说明：
A：备份一个数据库到一个目录中
mysqlhotcopy -u root -p Jaking@vip.163.com BOOK /tmp  #-u -p 后面都有空格

[root@jaking ~]# mysqlhotcopy -u root -p Jaking@vip.163.com BOOK /tmp
Flushed 1 tables with read lock (`BOOK`.`book`) in 0 seconds.
Locked 0 views () in 0 seconds.
Copying 2 files...
Copying indices for 0 files...
Unlocked tables.
mysqlhotcopy copied 1 tables (2 files) in 0 seconds (0 seconds overall).
[root@jaking ~]# ls /tmp
BOOK
[root@jaking ~]# ls /tmp/BOOK/
book.frm  db.opt

对比下大小

[root@jaking ~]# du -sh /var/lib/mysql/BOOK/ /tmp/BOOK/
16K     /var/lib/mysql/BOOK/
16K     /tmp/BOOK/
 
B：备份多个数据库到一个目录
```shell
[root@jaking ~]# rm -rf /tmp/*
[root@jaking ~]# mysqlhotcopy -u root -p Jaking@vip.163.com BOOK mysql /tmp/
mysqlhotcopy -u root -p Jaking@vip.163.com BOOK mysql /tmp/
Flushed 23 tables with read lock (`BOOK`.`book`, `mysql`.`columns_priv`, `mysql`.`db`, `mysql`.`event`, `mysql`.`func`, `mysql`.`help_category`, `mysql`.`help_keyword`, `mysql`.`help_relation`, `mysql`.`help_topic`, `mysql`.`host`, `mysql`.`ndb_binlog_index`, `mysql`.`plugin`, `mysql`.`proc`, `mysql`.`procs_priv`, `mysql`.`proxies_priv`, `mysql`.`servers`, `mysql`.`tables_priv`, `mysql`.`time_zone`, `mysql`.`time_zone_leap_second`, `mysql`.`time_zone_name`, `mysql`.`time_zone_transition`, `mysql`.`time_zone_transition_type`, `mysql`.`user`) in 0 seconds.
Locked 0 views () in 0 seconds.
Copying 2 files...
Copying indices for 0 files...
Copying 72 files...
Copying indices for 0 files...
Unlocked tables.
mysqlhotcopy copied 24 tables (74 files) in 0 seconds (0 seconds overall).

[root@jaking ~]# ls /tmp
BOOK  mysql
[root@jaking ~]# du -h /tmp 
0       /tmp/.X11-unix
0       /tmp/.font-unix
0       /tmp/.ICE-unix
0       /tmp/.XIM-unix
0       /tmp/.Test-unix
16K     /tmp/BOOK
1004K   /tmp/mysql
1020K   /tmp
[root@jaking ~]# du -h /var/lib/mysql/
1004K   /var/lib/mysql/mysql
212K    /var/lib/mysql/performance_schema
16K     /var/lib/mysql/BOOK
30M     /var/lib/mysql/
```
C：备份数据库中的某个表
语法：mysqlhotcopy -u 用户 -p 密码 数据库名./要备份的表名/  要备份的路径

mysqlhotcopy -u root -p Jaking@vip.163.com BOOK./book/ /tmp/   #注意最后的 / 一定要有！
实际上是把对应的表文件复制到/tmp目录下
```shell
[root@jaking ~]# rm -rf *
[root@jaking ~]# mysqlhotcopy -u root -p Jaking@vip.163.com BOOK./book/ /tmp/   Flushed 1 tables with read lock (`BOOK`.`book`) in 0 seconds.
Locked 0 views () in 0 seconds.
Copying 1 files...
Copying indices for 0 files...
Unlocked tables.
mysqlhotcopy copied 1 tables (1 files) in 0 seconds (0 seconds overall).
[root@jaking ~]# ls BOOK/
book.frm
[root@jaking ~]# ll /tmp/BOOK/
total 12
-rw-rw---- 1 mysql mysql 8624 Apr 22 14:31 book.frm
```
D:恢复数据

破坏数据
```shell
[root@jaking tmp]# cd /var/lib/mysql/
[root@jaking mysql]# ls
aria_log.00000001  BOOK     ib_logfile0  mysql       performance_schema
aria_log_control   ibdata1  ib_logfile1  mysql.sock
[root@jaking mysql]# rm -rf BOOK/
[root@jaking mysql]# ls
aria_log.00000001  ibdata1      ib_logfile1  mysql.sock
aria_log_control   ib_logfile0  mysql        performance_schema
[root@jaking mysql]#

[root@jaking ~]# mysql -uroot -pJaking@vip.163.com
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 5.5.52-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```sql
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]>
```
开始恢复
```shell
[root@jaking mysql]# cp -ra /tmp/BOOK/ /var/lib/mysql/
[root@jaking mysql]# ll /var/lib/mysql/
total 28700
-rw-rw---- 1 mysql mysql    16384 Apr 22 14:29 aria_log.00000001
-rw-rw---- 1 mysql mysql       52 Apr 22 14:29 aria_log_control
drwxr-x--- 2 mysql mysql       22 Apr 22 14:53 BOOK
-rw-rw---- 1 mysql mysql 18874368 Apr 22 14:53 ibdata1
-rw-rw---- 1 mysql mysql  5242880 Apr 22 14:53 ib_logfile0
-rw-rw---- 1 mysql mysql  5242880 Apr 22 14:29 ib_logfile1
drwx------ 2 mysql mysql     4096 Apr 22 14:29 mysql
srwxrwxrwx 1 mysql mysql        0 Apr 22 14:29 mysql.sock
drwx------ 2 mysql mysql     4096 Apr 22 14:29 performance_schema
[root@jaking mysql]# chown -Rf mysql:mysql /var/lib/mysql/BOOK/

[root@jaking ~]# mysql -uroot -pJaking@vip.163.com
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 40
Server version: 5.5.52-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```sql
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> use BOOK;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [BOOK]> show tables;
+----------------+
| Tables_in_BOOK |
+----------------+
| book           |
+----------------+
1 row in set (0.00 sec)

MariaDB [BOOK]> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
+-------------------------+-------+-------+
4 rows in set (0.00 sec)

MariaDB [BOOK]> 
```

总结：
mysqldump和mysqlhotcopy的比较：
1、mysqldump 是采用SQL级别的备份机制，它将数据表导成 SQL 脚本文件，数据库大时，占用系统资源较多，支持常用的MyISAM，Innodb
2、mysqlhotcopy只是简单的缓存写入和文件复制的过程，占用资源和备份速度比mysqldump快很多；特别适合大的数据库
3、mysqlhotcopy只能运行在数据库数据目录所在的机器上，mysqldump可以用在远程客户端
4、相同的地方都是在线执行LOCK TABLES 以及 UNLOCK TABLES
5、mysqlhotcopy恢复只需要COPY备份文件到源目录覆盖即可，mysqldump需要导入SQL文件到原来库中
 

实战：写个自动备份MySQL数据库shell脚本
vim  mysql-autoback.sh 
#!/bin/bash   
export LANG=en_US.UTF-8  
mkdir /database_back &>/dev/null
savedir=/database_back
cd "$savedir"  
time="$(date +"%Y-%m-%d")"  
mysqldump -u root -pJaking@vip.163.com book > book-"$time".sql  

再添加计划任务 crontab 就行
 

xtrabackup备份工具使用

1、xtrabackup简介

Xtrabackup包括两个主要工具：Xtrabackup和innobackupex：
Xtrabackup只能备份InnoDB和XtraDB两种引擎表，而不能备份MyISAM数据表

我们知道，针对InnoDB存储引擎，MySQL本身没有提供合适的热备工具，ibbackup虽是一款高效的首选热备方式，但它是是收费的。好在Percona公司给大家提供了一个开源、免费的Xtrabackup热备工具，它可实现ibbackup的所有功能，并且还扩展支持真正的增量备份功能，是商业备份工具InnoDB Hotbackup的一个很好的替代品。

2、Percona-XtraBackup 安装 （mysql5.7.20需安装XtraBackup2.4.9）

下载安装包
 [root@jaking ~]# wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.9/binary/redhat/7/x86_64/Percona-XtraBackup-2.4.9-ra467167cdd4-el7-x86_64-bundle.tar 

解压包
[root@jaking ~]# tar -xvf Percona-XtraBackup-2.4.9-ra467167cdd4-el7-x86_64-bundle.tar 

yum安装并解决依赖：
yum -y localinstall percona-xtrabackup-24-2.4.9-1.el7.x86_64.rpm 

或者
yum install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm

或者
rpm -ivh percona-release-latest.noarch.rpm

yum install -y percona-xtrabackup-24.x86_64

如果出现以下报错：
Error: Package: percona-xtrabackup-24-2.4.22-1.el7.x86_64 (percona-release-x86_64)
           Requires: libev.so.4()(64bit)

解决方法：
```shell
[root@jaking ~]# wget ftp://rpmfind.net/linux/atrpms/el6-x86_64/atrpms/stable/libev-4.04-2.el6.x86_64.rpm
[root@jaking ~]# rpm -ivh libev-4.04-2.el6.x86_64.rpm 
[root@jaking ~]# rz
rz waiting to receive.
 zmodem trl+C ȡ

  100%    7805 KB 7805 KB/s 00:00:01       0 Errors2.4.22-1.el7.x86_64.rpm...
 
[root@jaking ~]# ls -hl
total 7.7M
-rw-r--r--  1 root root  38K May 13 09:36 libev-4.04-2.el6.x86_64.rpm
-rwxr-xr-x. 1 root root  937 Apr 23 03:57 named.sh
-rw-r--r--  1 root root 7.7M Mar 10 04:56 percona-xtrabackup-24-2.4.22-1.el7.x86_64.rpm

或者
[root@jaking ~]# yum install -y percona-xtrabackup #备选方案

或者

[root@jaking ~]# rpm -ivh mysql-community-libs-compat-5.7.20-1.el7.x86_64.rpm
[root@jaking ~]# yum install -y cmake gcc gcc-c++ libaio libaio-devel automake autoconf bzr rsync
[root@jaking ~]# yum  install -y perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL  perl-Digest-MD5
[root@jaking ~]# rpm -ivh percona-xtrabackup-24-2.4.22-1.el7.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:percona-xtrabackup-24-2.4.22-1.el################################# [100%]
[root@jaking ~]# which xtrabackup
/usr/bin/xtrabackup
[root@jaking ~]# xtrabackup --version
xtrabackup: recognized server arguments: --datadir=/var/lib/mysql 
xtrabackup version 2.4.22 based on MySQL server 5.7.32 Linux (x86_64) (revision id: c99a781)

注意：my.cnf配置文件里datadir指定的MySQL数据路径

[root@jaking ~]#cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
skip-name-resolve

[root@jaking ~]# systemctl restart mysqld  
```


<span style="color:orange">fnaichu:</span>
先安装epel源
yum install -y epel-release

安装MySQL，确保至少安装有以下5个包：

安装xtrabackup:
参考官方文档：https://docs.percona.com/percona-xtrabackup/2.4/installation/yum_repo.html

安装xtrabackup依赖包
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm

下载xtrabackup安装包
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.4/\
binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.4-1.el7.x86_64.rpm

安装xtrabackup
yum localinstall percona-xtrabackup-24-2.4.4-1.el7.x86_64.rpm


3、xtrbackup使用
一般使用innobackupex脚本
innobackupex是perl脚本对xtrabackup的封装和功能扩展

备份准备工作：
权限和连接
xtrabackup需要连接到数据库和datadir操作权限
xtrabackup或者innobackupex在使用过程中涉及到两类用户权限

1.系统用户，用来调用innobackupex或者xtrabackup

  

2.数据库用户，数据库内使用的用户
   
      其他连接选项：

| 点击这里 | 点击这里 |
|    Tmpion
    |    Description
    |
|   –port
   |   The port to use when connecting to the database   server with TCP/IP.
   |
|   –socket
   |   The socket to use when   connecting to the local database.
   |
|   –host
   |   The host to use when connecting   to the database server with TCP/IP.
   |




可以单独创建用来备份数据库的用户，安全，并赋予对应的权限

4、全备和全备还原

使用innobackupex创建全备

创建全备：
innobackupex --user=root --password=Jaking@vip.163.com /tmp/db_backup/
innobackupex --user=root --password=Jaking@vip.163.com /tmp/db_backup/ &>>/tmp/db_backup.log
&>>/tmp/db_backup.log   #不显示输出信息，输出信息重定向到db_backup.log

[root@jaking ~]# ls /tmp
db_backup  db_backup.log  percona-version-check
[root@jaking ~]# ls /tmp/db_backup
2021-05-14_08-08-38
[root@jaking ~]# ls /tmp/db_backup/2021-05-14_08-08-38/
backup-my.cnf   ibdata1             sys                     xtrabackup_info
book1           market              testbookdb              xtrabackup_logfile
book2           mysql               testdb
ib_buffer_pool  performance_schema  xtrabackup_checkpoints 

内部机制：在备份的时候innobackupex会调用xtrabackup来备份innodb表，并复制所有的表定义，其他引擎的表(MyISAM,MERGE,CSV,ARCHIVE)
innobackupex是一个封装了xtrabackup的脚本，能同时处理innodb和myisam，但在处理myisam时需要加一个读锁

其他选项:
--no-timestamp，指定了这个选项备份会直接备份在BACKUP-DIR，不再创建时间戳文件夹
--default-file，指定配置文件，用来配置innobackupex的选项，该选项指定了从哪个文件读取MySQL配置

innobackupex --user=root --password=Jaking@vip.163.com --no-timestamp /tmp/db_backup/full (使用--no-timestamp时，后面的这个full目录必须跟上且不能提前自己建立，它由innobackupex自动建立，否则可能会报错innobackupex: Error: Failed to create backup directory) (Errcode: 17 - File exists)


[root@jaking ~]# innobackupex --default-file=/etc/my.cnf --user=root --password=Jaking@vip.163.com --no-timestamp /tmp/db_backup/full
[root@jaking ~]# ls /tmp/db_backup/
full

使用innobackupex还原备份

使用innobackupex --copy-back 来还原备份

停止MySQL服务

systemctl stop mysqld

删除数据：（危险动作）
```shell
[root@jaking ~]# systemctl stop mysqld
[root@jaking ~]# ls /var/lib/mysql
auto.cnf    client-cert.pem  ib_logfile0         private_key.pem  sys
BOOK        client-key.pem   ib_logfile1         public_key.pem   testbookdb
ca-key.pem  ib_buffer_pool   mysql               server-cert.pem
ca.pem      ibdata1          performance_schema  server-key.pem
[root@jaking ~]# rm -rf /var/lib/mysql/*
[root@jaking ~]# ls /var/lib/mysql

还原:
[root@jaking ~]# innobackupex --copy-back /tmp/db_backup/full/ 
[root@jaking ~]# ls /var/lib/mysql
BOOK            ibdata1  performance_schema  testbookdb
ib_buffer_pool  mysql    sys                 xtrabackup_info
 
查看权限
[root@jaking ~]# ll /var/lib/mysql
total 12324
drwxr-x--- 2 root root       52 Apr 23 04:53 BOOK
-rw-r----- 1 root root      310 Apr 23 04:53 ib_buffer_pool
-rw-r----- 1 root root 12582912 Apr 23 04:53 ibdata1
drwxr-x--- 2 root root     4096 Apr 23 04:53 mysql
drwxr-x--- 2 root root     8192 Apr 23 04:53 performance_schema
drwxr-x--- 2 root root     8192 Apr 23 04:53 sys
drwxr-x--- 2 root root       88 Apr 23 04:53 testbookdb
-rw-r----- 1 root root      472 Apr 23 04:53 xtrabackup_info

重新授权
[root@jaking ~]# chown -Rf mysql:mysql /var/lib/mysql   #要不然MySQL服务不能启动
[root@jaking ~]# ll /var/lib/mysql                   
total 12324
drwxr-x--- 2 mysql mysql       52 Apr 23 04:53 BOOK
-rw-r----- 1 mysql mysql      310 Apr 23 04:53 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 Apr 23 04:53 ibdata1
drwxr-x--- 2 mysql mysql     4096 Apr 23 04:53 mysql
drwxr-x--- 2 mysql mysql     8192 Apr 23 04:53 performance_schema
drwxr-x--- 2 mysql mysql     8192 Apr 23 04:53 sys
drwxr-x--- 2 mysql mysql       88 Apr 23 04:53 testbookdb
-rw-r----- 1 mysql mysql      472 Apr 23 04:53 xtrabackup_info
[root@jaking ~]# systemctl start mysqld

注：datadir必须是为空的，innobackupex –copy-back不会覆盖已存在的文件，还要注意，还原时需要先关闭服务，如果服务是启动的，那么就不能还原到datadir

[root@jaking ~]# ls /var/lib/mysql
auto.cnf  mysql  performance_schema  sys

[root@jaking ~]# innobackupex --copy-back /tmp/db_backup/full/
xtrabackup: recognized server arguments: --datadir=/var/lib/mysql 
xtrabackup: recognized client arguments: 
220423 05:01:30 innobackupex: Starting the copy-back operation

IMPORTANT: Please check that the copy-back run completes successfully.
           At the end of a successful copy-back run innobackupex
           prints "completed OK!".

innobackupex version 2.4.22 based on MySQL server 5.7.32 Linux (x86_64) (revision id: c99a781)
Original data directory /var/lib/mysql is not empty!

[root@jaking ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.20 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

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
| testbookdb         |
+--------------------+
6 rows in set (0.00 sec)

mysql> use BOOK;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_BOOK |
+----------------+
| book           |
+----------------+
1 row in set (0.00 sec)

mysql> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
+-------------------------+-------+-------+
4 rows in set (0.00 sec)

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

mysql> 
```
5：创建增量备份与还原
增量备份作用：减少备份数据重复，节省磁盘空间，缩短备份时间
增量备份的实现，依赖于innodb页上面的LSN（log sequence number），每次对数据库的修改都会导致LSN自增，增量备份会复制指定LSN<日志序列号>之后的所有数据页

创建增量备份
1. 首先创建全备
在创建增量备份之前需要一个全备，不然增量备份是没有意义的
```shell
innobackupex --user=root --password=Jaking@vip.163.com /tmp/db_backup/
[root@jaking ~]# ls /tmp
db_backup  percona-version-check
[root@jaking ~]# ls /tmp/db_backup/
2022-04-24_21-23-58
 ```
插入一些数据到表里面
```shell
[root@jaking ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.20 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

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
| testbookdb         |
+--------------------+
6 rows in set (0.00 sec)


CREATE DATABASE TESTDB;
show databases;
use TESTDB;
CREATE TABLE TABLE1 (bTypeId int,bName char(16),price int,publishing char(16));
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('1','Linux','66','DZ');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('2','CLD','68','RM');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('3','SYS','90','JX');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('4','MySQL1','71','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('5','MySQL2','72','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('6','MySQL3','73','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('7','MySQL4','74','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('8','MySQL5','75','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('9','MySQL6','76','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('10','MySQL7','77','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('11','MySQL8','78','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('12','MySQL9','79','QH');

CREATE TABLE TABLE2 (name char(16),price int,pages int);
INSERT INTO TABLE2(name,price,pages) VALUES('Linux','30','666');
INSERT INTO TABLE2(name,price,pages) VALUES('CLD','60','666');
INSERT INTO TABLE2(name,price,pages) VALUES('SYS','80','666');
INSERT INTO TABLE2(name,price,pages) VALUES('MySQL1','166','666');

show tables;
select * from TABLE1;
select * from TABLE2;

mysql> show tables;
+------------------+
| Tables_in_TESTDB |
+------------------+
| TABLE1           |
| TABLE2           |
+------------------+
2 rows in set (0.00 sec)

mysql> select * from TABLE1;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
|       7 | MySQL4 |    74 | QH         |
|       8 | MySQL5 |    75 | QH         |
|       9 | MySQL6 |    76 | QH         |
|      10 | MySQL7 |    77 | QH         |
|      11 | MySQL8 |    78 | QH         |
|      12 | MySQL9 |    79 | QH         |
+---------+--------+-------+------------+
12 rows in set (0.00 sec)

mysql> select * from TABLE2;
+--------+-------+-------+
| name   | price | pages |
+--------+-------+-------+
| Linux  |    30 |   666 |
| CLD    |    60 |   666 |
| SYS    |    80 |   666 |
| MySQL1 |   166 |   666 |
+--------+-------+-------+
4 rows in set (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| TESTDB             |
| mysql              |
| performance_schema |
| sys                |
| testbookdb         |
+--------------------+
7 rows in set (0.00 sec)

4 rows in set (0.00 sec)
 ```
2. 使用 --incremental创建增量备份
```shell
innobackupex --user=root --password=Jaking@vip.163.com --incremental /增量1路径 --incremental-basedir=全备路径，指定在哪个全备上进行增量备份
[root@jaking ~]# ls /tmp/db_backup/
2021-05-14_08-33-57
[root@jaking ~]# innobackupex --user=root --password=Jaking@vip.163.com --incremental /tmp/db_backup/ --incremental-basedir=/tmp/db_backup/2022-04-24_21-23-58/
[root@jaking ~]# ls /tmp/db_backup/
2022-04-24_21-23-58  2022-04-24_21-31-38
[root@jaking ~]# ls /tmp/db_backup/2022-04-24_21-23-58/
backup-my.cnf   mysql               xtrabackup_checkpoints
BOOK            performance_schema  xtrabackup_info
ib_buffer_pool  sys                 xtrabackup_logfile
ibdata1         testbookdb
[root@jaking ~]# ls /tmp/db_backup/2022-04-24_21-31-38/
backup-my.cnf   mysql               xtrabackup_checkpoints
BOOK            performance_schema  xtrabackup_info
ib_buffer_pool  sys                 xtrabackup_logfile
ibdata1.delta   testbookdb
ibdata1.meta    TESTDB
[root@jaking ~]# du -sh /tmp/db_backup/2022-04-24_21-23-58/
26M     /tmp/db_backup/2022-04-24_21-23-58/
[root@jaking ~]# du -sh /tmp/db_backup/2022-04-24_21-31-38/ 
5.1M    /tmp/db_backup/2022-04-24_21-31-38/
```
再查看LSN<日志序列号>
[root@jaking ~]# cat /tmp/db_backup/2022-04-24_21-23-58/xtrabackup_checkpoints
backup_type = full-backuped
from_lsn = 0
to_lsn = 2566100
last_lsn = 2566109
compact = 0
recover_binlog_info = 0
flushed_lsn = 2566109

[root@jaking ~]# cat /tmp/db_backup/2022-04-24_21-31-38/xtrabackup_checkpoints
backup_type = incremental
from_lsn = 2566100
to_lsn = 2588592
last_lsn = 2588601
compact = 0
recover_binlog_info = 0
flushed_lsn = 2588601

3. 增量备份创建的替代方法
可以使用指定 --incremental-lsn来代 --incremental-basedir的方法创建增量备份
innobackupex --user=root --password=Jaking@vip.163.com --incremental /tmp/db_backup/ --incremental-lsn=<日志序列号>  #从这个编号点开始备份
```shell
[root@jaking ~]# ls /tmp/db_backup/
2022-04-24_21-23-58  2022-04-24_21-31-38
[root@jaking ~]# innobackupex --user=root --password=Jaking@vip.163.com --incremental /tmp/db_backup/ --incremental-lsn=2566109
[root@jaking ~]# ls /tmp/db_backup/
2022-04-24_21-23-58  2022-04-24_21-31-38  2022-04-24_21-41-49
[root@jaking ~]# du -sh /tmp/db_backup/2022-04-24_21-23-58
26M     /tmp/db_backup/2022-04-24_21-23-58
[root@jaking ~]# du -sh /tmp/db_backup/2022-04-24_21-31-38
5.1M    /tmp/db_backup/2022-04-24_21-31-38
[root@jaking ~]# du -sh /tmp/db_backup/2022-04-24_21-41-49
5.1M    /tmp/db_backup/2022-04-24_21-41-49
```
注意：xtrabackup只会影响xtradb或者innodb的表，其他引擎的表在增量备份的时候只会复制整个文件

还原

停止MySQL服务,删除数据 rm -rf /var/lib/mysql/* 

还原增量备份 
增量备份的恢复比全备要复杂一点，第一步是在所有备份目录下重做已提交的日志，如：
innobackupex --apply-log --redo-only BASE-DIR  

innobackupex --apply-log --redo-only BASE-DIR --incremental-dir=INCREMENTAL-DIR-1  
innobackupex --apply-log --redo-only BASE-DIR --incremental-dir=INCREMENTAL-DIR-2  
...
...
...
innobackupex --apply-log BASE-DIR --incremental-dir=INCREMENTAL-DIR-last-num

注意：如果仅有一份增量备份，第3条语句忽略
其中BASE-DIR是指全备目录，INCREMENTAL-DIR-1是指第一次的增量备份，INCREMENTAL-DIR-2是指第二次的增量备份，以此类推。

这里要注意的是：最后一步的增量备份并没有--redo-only选项！

以上语句执行成功之后，最终数据在BASE-DIR（即全备目录）下

第一步完成之后，我们开始第二步：回滚未完成的日志：
innobackupex --apply-log BASE-DIR

上面执行完之后，BASE-DIR里的备份文件已完全准备就绪，最后一步是拷贝：
innobackupex --copy-back BASE-DIR

恢复mysql权限 chown -Rf mysql:mysql /var/lib/mysql/

最后启动 systemctl start mysqld 

检验数据是否恢复正常
注意：innobackupex还原增量备份容易出错，不建议使用！

实战：
```shell
[root@jaking ~]# ls /var/lib/mysql
auto.cnf        ibdata1      mysql               testbookdb
BOOK            ib_logfile0  performance_schema  TESTDB
ib_buffer_pool  ib_logfile1  sys                 xtrabackup_info
[root@jaking ~]# mkdir /mysql-all-data
[root@jaking ~]# mv /var/lib/mysql/* /mysql-all-data/
[root@jaking ~]# ls /var/lib/mysql                   
[root@jaking ~]# ls /mysql-all-data/
auto.cnf        ibdata1      mysql               testbookdb
BOOK            ib_logfile0  performance_schema  TESTDB
ib_buffer_pool  ib_logfile1  sys                 xtrabackup_info
[root@jaking ~]# ls /tmp/db_backup/
2022-04-24_21-23-58  2022-04-24_21-31-38  2022-04-24_21-41-49
[root@jaking ~]# rm -rf /tmp/db_backup/2022-04-24_21-41-49
[root@jaking ~]# ls /tmp/db_backup/                       
2022-04-24_21-23-58  2022-04-24_21-31-38
[root@jaking ~]# innobackupex --apply-log --redo-only /tmp/db_backup/2022-04-24_21-23-58
[root@jaking ~]# innobackupex --apply-log /tmp/db_backup/2022-04-24_21-23-58 --incremental-dir=/tmp/db_backup/2022-04-24_21-31-38
fnaich:！增量合并必须使用绝对路径，否则报错
[root@jaking ~]# ls /tmp/db_backup/2022-04-24_21-23-58/
backup-my.cnf   ib_logfile0  performance_schema  xtrabackup_checkpoints
BOOK            ib_logfile1  sys                 xtrabackup_info
ib_buffer_pool  ibtmp1       testbookdb          xtrabackup_logfile
ibdata1         mysql        TESTDB              xtrabackup_master_key_id
[root@jaking ~]# innobackupex --apply-log /tmp/db_backup/2022-04-24_21-23-58/
[root@jaking ~]# innobackupex --copy-back /tmp/db_backup/2022-04-24_21-23-58/
[root@jaking ~]# chown -Rf mysql:mysql /var/lib/mysql/
[root@jaking ~]# ll /var/lib/mysql/
total 122920
drwxr-x--- 2 mysql mysql       88 Apr 24 21:59 BOOK
-rw-r----- 1 mysql mysql      315 Apr 24 21:59 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 Apr 24 21:59 ibdata1
-rw-r----- 1 mysql mysql 50331648 Apr 24 21:59 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr 24 21:59 ib_logfile1
-rw-r----- 1 mysql mysql 12582912 Apr 24 21:59 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Apr 24 21:59 mysql
drwxr-x--- 2 mysql mysql     8192 Apr 24 21:59 performance_schema
drwxr-x--- 2 mysql mysql     8192 Apr 24 21:59 sys
drwxr-x--- 2 mysql mysql       88 Apr 24 21:59 testbookdb
drwxr-x--- 2 mysql mysql       92 Apr 24 21:59 TESTDB
-rw-r----- 1 mysql mysql      504 Apr 24 21:59 xtrabackup_info
-rw-r----- 1 mysql mysql        1 Apr 24 21:59 xtrabackup_master_key_id
[root@jaking ~]# systemctl start mysqld
[root@jaking ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.20 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| TESTDB             |
| mysql              |
| performance_schema |
| sys                |
| testbookdb         |
+--------------------+
7 rows in set (0.00 sec)

mysql> use BOOK;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_BOOK |
+----------------+
| book           |
+----------------+
2 rows in set (0.00 sec)

mysql> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
+-------------------------+-------+-------+
4 rows in set (0.00 sec)

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

mysql> use TESTDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_TESTDB |
+------------------+
| TABLE1           |
| TABLE2           |
+------------------+
2 rows in set (0.00 sec)

mysql> select * from TABLE1;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
|       7 | MySQL4 |    74 | QH         |
|       8 | MySQL5 |    75 | QH         |
|       9 | MySQL6 |    76 | QH         |
|      10 | MySQL7 |    77 | QH         |
|      11 | MySQL8 |    78 | QH         |
|      12 | MySQL9 |    79 | QH         |
+---------+--------+-------+------------+
12 rows in set (0.00 sec)

mysql> select * from TABLE2;
+--------+-------+-------+
| name   | price | pages |
+--------+-------+-------+
| Linux  |    30 |   666 |
| CLD    |    60 |   666 |
| SYS    |    80 |   666 |
| MySQL1 |   166 |   666 |
+--------+-------+-------+
4 rows in set (0.00 sec)

mysql> 
```

拓展：还原数据
```shell
[root@server12 ~]# rm -rf /tmp/db_backup/*
[root@server12 ~]# innobackupex --default-file=/etc/my.cnf --user=root --password=Jaking@vip.163.com --no-timestamp /tmp/db_backup/all-mysql-data
[root@server12 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.20 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| TESTDB             |
| mysql              |
| performance_schema |
| sys                |
| testbookdb         |
+--------------------+
7 rows in set (0.00 sec)

CREATE DATABASE TESTDB666;
show databases;
use TESTDB666;
CREATE TABLE TABLE1 (bTypeId int,bName char(16),price int,publishing char(16));
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('1','Linux','66','DZ');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('2','CLD','68','RM');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('3','SYS','90','JX');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('4','MySQL1','71','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('5','MySQL2','72','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('6','MySQL3','73','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('7','MySQL4','74','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('8','MySQL5','75','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('9','MySQL6','76','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('10','MySQL7','77','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('11','MySQL8','78','QH');
INSERT INTO TABLE1(bTypeId,bName,price,publishing) VALUES('12','MySQL9','79','QH');

CREATE TABLE TABLE2 (name char(16),price int,pages int);
INSERT INTO TABLE2(name,price,pages) VALUES('Linux','30','666');
INSERT INTO TABLE2(name,price,pages) VALUES('CLD','60','666');
INSERT INTO TABLE2(name,price,pages) VALUES('SYS','80','666');
INSERT INTO TABLE2(name,price,pages) VALUES('MySQL1','166','666');

show tables;
select * from TABLE1;
select * from TABLE2;

mysql> show tables;
+---------------------+
| Tables_in_TESTDB666 |
+---------------------+
| TABLE1              |
| TABLE2              |
+---------------------+
2 rows in set (0.00 sec)

mysql> select * from TABLE1;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
|       7 | MySQL4 |    74 | QH         |
|       8 | MySQL5 |    75 | QH         |
|       9 | MySQL6 |    76 | QH         |
|      10 | MySQL7 |    77 | QH         |
|      11 | MySQL8 |    78 | QH         |
|      12 | MySQL9 |    79 | QH         |
+---------+--------+-------+------------+
12 rows in set (0.00 sec)

mysql> select * from TABLE2;
+--------+-------+-------+
| name   | price | pages |
+--------+-------+-------+
| Linux  |    30 |   666 |
| CLD    |    60 |   666 |
| SYS    |    80 |   666 |
| MySQL1 |   166 |   666 |
+--------+-------+-------+
4 rows in set (0.00 sec)

mysql> 

mysql> ^DBye
```

```shell
[root@server12 ~]# innobackupex --user=root --password=Jaking@vip.163.com --no-timestamp --incremental /tmp/db_backup/incremental-mysql-data --incremental-basedir=/tmp/db_backup/all-mysql-data/
[root@server12 ~]# ls /tmp/db_backup/
all-mysql-data  incremental-mysql-data
[root@server12 ~]# du -sh /tmp/db_backup/all-mysql-data/
27M     /tmp/db_backup/all-mysql-data/
[root@server12 ~]# du -sh /tmp/db_backup/incremental-mysql-data/
4.8M    /tmp/db_backup/incremental-mysql-data/

[root@server12 ~]# systemctl stop mysqld
[root@server12 ~]# rm -rf /var/lib/mysql/*
[root@server12 ~]# ls /tmp/db_backup/all-mysql-data/
backup-my.cnf   ibdata1             sys         xtrabackup_checkpoints
BOOK            mysql               testbookdb  xtrabackup_info
ib_buffer_pool  performance_schema  TESTDB      xtrabackup_logfile
[root@server12 ~]# ls /tmp/db_backup/incremental-mysql-data/
backup-my.cnf   mysql               TESTDB666
BOOK            performance_schema  xtrabackup_checkpoints
ib_buffer_pool  sys                 xtrabackup_info
ibdata1.delta   testbookdb          xtrabackup_logfile
ibdata1.meta    TESTDB
[root@server12 ~]# cp -ra /tmp/db_backup/incremental-mysql-data/TESTDB666 /tmp/db_backup/all-mysql-data/
[root@server12 ~]# cp -ra /tmp/db_backup/incremental-mysql-data/ibdata1* /tmp/db_backup/all-mysql-data/
[root@server12 ~]# ls /tmp/db_backup/all-mysql-data/
backup-my.cnf   ibdata1.delta       sys         xtrabackup_checkpoints
BOOK            ibdata1.meta        testbookdb  xtrabackup_info
ib_buffer_pool  mysql               TESTDB      xtrabackup_logfile
ibdata1         performance_schema  TESTDB666
[root@server12 ~]# ls /tmp/db_backup/incremental-mysql-data/
backup-my.cnf   mysql               TESTDB666
BOOK            performance_schema  xtrabackup_checkpoints
ib_buffer_pool  sys                 xtrabackup_info
ibdata1.delta   testbookdb          xtrabackup_logfile
ibdata1.meta    TESTDB
[root@server12 ~]# innobackupex --copy-back /tmp/db_backup/all-mysql-data/
[root@server12 ~]# ll /var/lib/mysql
total 13912
drwxr-x--- 2 root root       88 Apr 24 22:26 BOOK
-rw-r----- 1 root root      315 Apr 24 22:26 ib_buffer_pool
-rw-r----- 1 root root 12582912 Apr 24 22:26 ibdata1
-rw-r----- 1 root root  1622016 Apr 24 22:26 ibdata1.delta
-rw-r----- 1 root root       60 Apr 24 22:26 ibdata1.meta
drwxr-x--- 2 root root     4096 Apr 24 22:26 mysql
drwxr-x--- 2 root root     8192 Apr 24 22:26 performance_schema
drwxr-x--- 2 root root     8192 Apr 24 22:26 sys
drwxr-x--- 2 root root       88 Apr 24 22:26 testbookdb
drwxr-x--- 2 root root       92 Apr 24 22:26 TESTDB
drwxr-x--- 2 root root      150 Apr 24 22:26 TESTDB666
-rw-r----- 1 root root      482 Apr 24 22:26 xtrabackup_info
[root@server12 ~]# chown -Rf mysql:mysql /var/lib/mysql
[root@server12 ~]# ll /var/lib/mysql                   
total 13912
drwxr-x--- 2 mysql mysql       88 Apr 24 22:26 BOOK
-rw-r----- 1 mysql mysql      315 Apr 24 22:26 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 Apr 24 22:26 ibdata1
-rw-r----- 1 mysql mysql  1622016 Apr 24 22:26 ibdata1.delta
-rw-r----- 1 mysql mysql       60 Apr 24 22:26 ibdata1.meta
drwxr-x--- 2 mysql mysql     4096 Apr 24 22:26 mysql
drwxr-x--- 2 mysql mysql     8192 Apr 24 22:26 performance_schema
drwxr-x--- 2 mysql mysql     8192 Apr 24 22:26 sys
drwxr-x--- 2 mysql mysql       88 Apr 24 22:26 testbookdb
drwxr-x--- 2 mysql mysql       92 Apr 24 22:26 TESTDB
drwxr-x--- 2 mysql mysql      150 Apr 24 22:26 TESTDB666
-rw-r----- 1 mysql mysql      482 Apr 24 22:26 xtrabackup_info
[root@server12 ~]# systemctl start mysqld
[root@server12 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.20 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| TESTDB             |
| TESTDB666          |
| mysql              |
| performance_schema |
| sys                |
| testbookdb         |
+--------------------+
8 rows in set (0.00 sec)

mysql> use BOOK;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_BOOK |
+----------------+
| book           |
+----------------+
2 rows in set (0.00 sec)

mysql> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
+-------------------------+-------+-------+
4 rows in set (0.00 sec)

mysql> use TESTDB666;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------+
| Tables_in_TESTDB666 |
+---------------------+
| TABLE1              |
| TABLE2              |
+---------------------+
2 rows in set (0.00 sec)

mysql> select * from TABLE1;
ERROR 1146 (42S02): Table 'TESTDB666.TABLE1' doesn't exist
mysql> select * from TABLE2;
ERROR 1146 (42S02): Table 'TESTDB666.TABLE2' doesn't exist
mysql> use TESTDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_TESTDB |
+------------------+
| TABLE1           |
| TABLE2           |
+------------------+
2 rows in set (0.00 sec)

mysql> select * from TABLE1;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
|       7 | MySQL4 |    74 | QH         |
|       8 | MySQL5 |    75 | QH         |
|       9 | MySQL6 |    76 | QH         |
|      10 | MySQL7 |    77 | QH         |
|      11 | MySQL8 |    78 | QH         |
|      12 | MySQL9 |    79 | QH         |
+---------+--------+-------+------------+
12 rows in set (0.00 sec)

mysql> select * from TABLE2;
+--------+-------+-------+
| name   | price | pages |
+--------+-------+-------+
| Linux  |    30 |   666 |
| CLD    |    60 |   666 |
| SYS    |    80 |   666 |
| MySQL1 |   166 |   666 |
+--------+-------+-------+
4 rows in set (0.00 sec)

mysql> ^DBye
```

```shell
[root@server12 ~]#

[root@server12 ~]# ll /mysql-all-data/
total 110632
-rw-r----- 1 mysql mysql       56 Apr 23 05:15 auto.cnf
drwxr-x--- 2 mysql mysql       88 Apr 24 21:26 BOOK
-rw-r----- 1 mysql mysql      343 Apr 24 21:51 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 Apr 24 21:51 ibdata1
-rw-r----- 1 mysql mysql 50331648 Apr 24 21:51 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr 23 05:15 ib_logfile1
drwxr-x--- 2 mysql mysql     4096 Apr 23 05:14 mysql
drwxr-x--- 2 mysql mysql     8192 Apr 23 05:14 performance_schema
drwxr-x--- 2 mysql mysql     8192 Apr 23 05:14 sys
drwxr-x--- 2 mysql mysql       88 Apr 23 05:14 testbookdb
drwxr-x--- 2 mysql mysql       92 Apr 24 21:28 TESTDB
-rw-r----- 1 mysql mysql      472 Apr 23 05:14 xtrabackup_info
[root@server12 ~]# systemctl stop mysqld
[root@server12 ~]# rm -rf /var/lib/mysql/*
[root@server12 ~]# mv /mysql-all-data/* /var/lib/mysql
[root@server12 ~]# ll /var/lib/mysql
total 110632
-rw-r----- 1 mysql mysql       56 Apr 23 05:15 auto.cnf
drwxr-x--- 2 mysql mysql       88 Apr 24 21:26 BOOK
-rw-r----- 1 mysql mysql      343 Apr 24 21:51 ib_buffer_pool
-rw-r----- 1 mysql mysql 12582912 Apr 24 21:51 ibdata1
-rw-r----- 1 mysql mysql 50331648 Apr 24 21:51 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Apr 23 05:15 ib_logfile1
drwxr-x--- 2 mysql mysql     4096 Apr 23 05:14 mysql
drwxr-x--- 2 mysql mysql     8192 Apr 23 05:14 performance_schema
drwxr-x--- 2 mysql mysql     8192 Apr 23 05:14 sys
drwxr-x--- 2 mysql mysql       88 Apr 23 05:14 testbookdb
drwxr-x--- 2 mysql mysql       92 Apr 24 21:28 TESTDB
-rw-r----- 1 mysql mysql      472 Apr 23 05:14 xtrabackup_info
[root@server12 ~]# systemctl start mysqld
[root@server12 ~]# mysql -uroot -pJaking@vip.163.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.20 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| BOOK               |
| TESTDB             |
| mysql              |
| performance_schema |
| sys                |
| testbookdb         |
+--------------------+
7 rows in set (0.01 sec)

mysql> use BOOK;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_BOOK |
+----------------+
| book           |
+----------------+
2 rows in set (0.00 sec)

mysql> select * from book;
+-------------------------+-------+-------+
| name                    | price | pages |
+-------------------------+-------+-------+
| Linux                   |    30 |   666 |
| Cloud Computing         |    60 |   666 |
| Operation System        |    80 |   666 |
| Artificial Intelligence |   166 |   666 |
+-------------------------+-------+-------+
4 rows in set (0.00 sec)

mysql> use TESTDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_TESTDB |
+------------------+
| TABLE1           |
| TABLE2           |
+------------------+
2 rows in set (0.00 sec)

mysql> select * from TABLE1;
+---------+--------+-------+------------+
| bTypeId | bName  | price | publishing |
+---------+--------+-------+------------+
|       1 | Linux  |    66 | DZ         |
|       2 | CLD    |    68 | RM         |
|       3 | SYS    |    90 | JX         |
|       4 | MySQL1 |    71 | QH         |
|       5 | MySQL2 |    72 | QH         |
|       6 | MySQL3 |    73 | QH         |
|       7 | MySQL4 |    74 | QH         |
|       8 | MySQL5 |    75 | QH         |
|       9 | MySQL6 |    76 | QH         |
|      10 | MySQL7 |    77 | QH         |
|      11 | MySQL8 |    78 | QH         |
|      12 | MySQL9 |    79 | QH         |
+---------+--------+-------+------------+
12 rows in set (0.00 sec)

mysql> select * from TABLE2;
+--------+-------+-------+
| name   | price | pages |
+--------+-------+-------+
| Linux  |    30 |   666 |
| CLD    |    60 |   666 |
| SYS    |    80 |   666 |
| MySQL1 |   166 |   666 |
+--------+-------+-------+
4 rows in set (0.00 sec)

mysql> ^DBye
[root@server12 ~]#
```

由以上操作可知innobackupex还原增量数据不稳定！

6：官方文档
https://www.percona.com/doc/percona-xtrabackup/2.4/index.html

文档：10 mysqldump远程备份与恢复数据.md
链接：http://note.youdao.com/noteshare?id=7af3e7cda297defe01228ed798e72c02&sub=E2F44DF8E1BC4B9287EA434DEA3B670C

文档：16 innobackupex备份MySQL数据遇到Can'...
链接：http://note.youdao.com/noteshare?id=75c497bba7496b9ec1c61c766e03cca8&sub=F4A9B0EB5E554DB5BB6EF40A12A25512

练习：1. 把数据字典导入MySQL数据库 2. MySQL5.7.20在线升级到MySQL5.7.24
