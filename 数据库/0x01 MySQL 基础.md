---
title: 0x01 MySQL 基础
tags: []
---

## 一、SQL语句

DDL	数据库定义语言	create drop alter  
DML	数据库操作语言	insert delete update  
DCL	数据库控制语言	grant revoke  
DQL	数据库查询语言	select   

本章要求掌握sql语句的基本用法，包括create database,create table, drop database,drop table,insert into,update,delete from,grant,revoke; 其他sql语句作为拓展。	

MySQL中的语法格式  
1. 默认以 `;` 提交
2. 关键字大小写不敏感；库名、表名、数据都是区分大小写；
3. 使用空格

- DQL	
	- 数据库查询语言 select
	- 查询数据库show databases;
	- 查询表
		show tables;
		show tables from test;
	- 查看表的结构	      desc mysql.user;
	- 查看表中的数据	   select host,user,password from mysql.user where user='root';

\------------------------------------------------------  
- DDL	
	- 数据库定义语言 create drop alter
	- 创建库 create database dbname;
	- 删除库 drop database dbname;
	- 创建表 create table tbname (id int,name char(20),age int);
	- 删除表 drop table tbname;

- 库名表名命名规则
	- 英文字母、数字、下划线
	- 以英文字母开头
	- <=32个字符
	
切换到执行数据库：
```sql
mysql> use dbname
Database changed
```
\---------------------------------------------------------------  
- DML	数据库操作语言	 insert delete update
	- 插入一行	      
		- insert into dbname.tbname values (1,'superman',21);
		- insert into dbname.tbname set id=1,name='superman',age=21;
	- 插入多行         
		- insert into dbname.tbname values (2,'batman1',21),(3,'batman2',22),(4,'batman3',23),(5,'batman4',24);
		- insert into dbname.tbname (id,age) values (6,26);	
	- update	             update dbname.tbname set age=14 where id=2 or id=3;
	- delete	             delete from dbname.tbname where id=2;

\------------------------------------------------------------------------

- DCL	数据库控制语言	grant 授权 revoke 收回授权
	- `grant all on *.* to 'batman'@'172.25.0.12' identified by 'uplooking';`
		- grant 关键字
		- all	所有权限除了grant之外
		- on *.*  所有的库所有的表
		- to user@host 权限是给谁的，从哪里来的
		- identified by 密码
	- `flush privileges;` 刷新授权表

## 二、表的存储引擎

存储引擎  数据库读写数据的方式，决定了读写数据的快慢


|            事务	 |  锁机制	 |  适用场景 |
| ---  | --- | --- |
|myisam | no 	|   表锁	 |     分析型 |
innodb |  yes	|   行锁	 |     金钱+线上高并发 |

## 三、事务

事务(Transaction)是并发控制的基本单位。

`所谓事务,它是一个操作序列,这些操作要么都执行,要么都不执行, 它是一个不可分割的工作单位。`例如,银行转帐工作:从一个帐号扣款并 使另一个帐号增款,这两个操作要么都执行,要么都不执行。

事务的四大特性ACID  
1. 原子性（Atomicity）事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。 
2. 一致性（Consistency）事务在完成时，必须是所有的数据都保持一致状态。
3. 隔离性（Isolation）并发事务执行之间无影响，在一个事务内部的操作对其他事务是不产生影响，这需要事务隔离级别来指定隔离性。
4. 持久性（Durability）一旦事务完成，数据库的改变必须是持久化的。

查看所有的存储引擎	  show engines;  
创建表指定存储引擎	  create table tbname (id int,name char(20),age int) engine=myisam;  
查看表的属性	use dbname; show table status where name='tbname';   

## 四、如何获得MySQL相关资源

为了学习更多的MySQL知识,请访问MySQL官网 https://www.mysql.com/

## 五、MySQL在企业中的应用场景

MySQL 拥有庞大的用户群,国外的有 Facebook、Flickr、eBay 等,国内的有阿里、腾讯、新浪、百度等。而这些互联网和大部分传统公司的服务需要7×24小时连续工作。当此类型网站的部分数据库服务器宕机时,就需要高可用技术将流量牵引至备份主机,从而此时这些公司需要通过备份和恢复手段来产生备机,并通过复制来同步主备机间的状态,同时部署各种监控软件来监控服务器状态。当异常数据库服务器宕机时,通过手工或自动化手段将主机流量切换至备机,这个动作叫作failover。而一些大型公司在面对成千上万台 MySQL 服务器时,通常使用`自动化运维脚本`或程序完成上述种种动作。
