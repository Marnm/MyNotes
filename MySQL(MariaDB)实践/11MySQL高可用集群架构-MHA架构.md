---
title: 11MySQL高可用集群架构-MHA架构
tags:
  - mysql
  - mha
---

# MySQL高可用集群——MHA架构

**简介：**
 
MHA（Master High Availability）目前在MySQL高可用方面是一个相对成熟的解决方案，它由日本DeNA公司youshimaton（现就职于Facebook公司）开发，是一套优秀的作为MySQL高可用性环境下故障切换和主从提升的高可用软件。在MySQL故障切换过程中，MHA能做到在0~30秒之内自动完成数据库的故障切换操作，并且在进行故障切换的过程中，MHA能在最大程度上保证数据的一致性，以达到真正意义上的高可用。
 
该软件由两部分组成：MHA Manager（管理节点）和MHA Node（数据节点）。  

MHA Manager可以单独部署在一台独立的机器上管理多个master-slave集群，也可以部署在一台slave节点上。  
MHA Node运行在每台MySQL服务器上，MHA Manager会定时探测集群中的master节点，当master出现故障时，它可以自动将最新数据的slave提升为新的master，然后将所有其他的slave重新指向新的master。整个故障转移过程对应用程序完全透明。
 
在MHA自动故障切换过程中，MHA试图从宕机的主服务器上保存二进制日志，最大程度的保证数据的不丢失，但这并不总是可行的。例如，如果主服务器硬件故障或无法通过ssh访问，MHA没法保存二进制日志，只进行故障转移而丢失了最新的数据。使用MySQL 5.5的半同步复制，可以大大降低数据丢失的风险。MHA可以与半同步复制结合起来。如果只有一个slave已经收到了最新的二进制日志，MHA可以将最新的二进制日志应用于其他所有的slave服务器上，因此可以保证所有节点的数据一致性。
 
目前MHA主要支持一主多从的架构，要搭建MHA,要求一个复制集群中必须最少有三台数据库服务器，一主二从，即一台充当master，一台充当备用master，另外一台充当从库，因为至少需要三台服务器，出于机器成本的考虑，淘宝也在该基础上进行了改造，目前淘宝TMHA已经支持一主一从。MHA 适合任何存储引擎, 只要能主从复制的存储引擎它都支持，不限于支持事物的 innodb 引擎。
 
官方介绍：https://code.google.com/p/mysql-master-ha/
 
图01展示了如何通过MHA Manager管理多组主从复制。可以将MHA工作原理总结为如下：
![](http://192.168.85.188:8081/uploads/1652923038589086640mha_str.png)

（1）从宕机崩溃的master保存二进制日志事件（binlog events）;
 
（2）识别含有最新更新的slave；
 
（3）应用差异的中继日志（relay log）到其他的slave；
 
（4）应用从master保存的二进制日志事件（binlog events）；
 
（5）提升一个slave为新的master；
 
（6）使其他的slave连接新的master进行复制；
 
MHA软件由两部分组成，Manager工具包和Node工具包，具体的说明如下。

**MHA Manager**工具包由两部分组成，Manager工具包和Node工具包，具体的说明如下。

Manager工具包主要包括以下几个工具：

|     **名 称**       |     **作 用**       |
|        ---          |        ---         |
| masterha_check_ssh  | 检查MHA的SSH配置状况 |
| masterha_check_repl | 检查MySQL复制状况   |
| masterha_manager    | 启动MHA             |
| masterha_check_status | 检测当前MHA运行状态 |
| masterha_master_monitor | 检测master是否宕机 |
| masterha_master_switch | 控制故障转移（自动或手动） |
| masterha_conf_host | 添加或删除配置的server信息 |

**MHA Node**工具包（这些工具通常有MHA Manager的脚本触发，无需人为操作）主要包括以下几个工具：

|    **名 称**          |                     **作 用**                      |
|       ---             |                            ---                    |
| save_binary_logs      | 保存和复制master的二进制日志                         |
| apply_diff_relay_logs | 识别差异的中继日志事件并将其差异的事件应用于其他的slave |
| filter_mysqlbinlog    | 去除不必要的ROLLBACK事件（已废弃）                    |
| purge_relay_logs      | 清除中继日志（不会阻塞SQL线程）                       |

_注意：_
为了尽可能的减少主库硬件损坏宕机造成的数据丢失，因此在配置MHA的同时建议配置成MySQL 5.5的半同步复制。关于半同步复制原理各位自己进行查阅。（不是必须）

## 1. 安装部署MHA

接下来部署MHA，具体的搭建环境如下（操作系统为2台CentOS7.4 64bit，2台RHEL7.3 64bit）

|  角色                  |    IP地址       |   主机名     |        工具                       |    操作系统    |
|  ---                   |      ---       |    ---       |        ---                        |     ---       |
| manager                | 192.168.73.20  |  ctos7mini   |  mha4mysql-manager,mha4mysql-node | CentOS7.4 x64 |
| master                 | 192.168.73.21  |  rhel7       |  mha4mysql-node                   | RHEL7.3 x64   |
| slave,candicate master | 192.168.73.22  |  rhel_sqlsrv |  mha4mysql-node                   | RHEL7.3 x64   |
| slave                  | 192.168.73.23  |  ctos_sqlsrv |  mha4mysql-node                   | CentOS7.4 x64 |

其中master对外提供写服务，备选Candicate master（实际为slave1）提供读服务，slave2也提供读服务，一旦master宕机，将会把备选master提升为新的master，slave指向新的master

**（1）在安装MHA工具之前，先要安装epel源，解决依赖包问题：**

Centos6安装源：
	rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm（已失效，官方不再支持）  
Centos7安装源：
	rpm -ivh https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-14.noarch.rpm  
Centos8安装源：8以上需要安装2个：  
	rpm -ivh https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/e/epel-release-8-15.el8.noarch.rpm  
	rpm -ivh https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/e/epel-next-release-8-15.el8.noarch.rpm  
                               
Centos9安装源：  
	rpm -ivh https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/Packages/e/epel-release-9-2.el9.noarch.rpm  
	rpm -ivh https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/Packages/e/epel-next-release-9-2.el9.noarch.rpm  
                               
RHEL7安装源：yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm  
RHEL8安装源：dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm  
RHEL9-Beta安装源：dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm  

MHA Manager中主要包括了几个管理员的命令行工具，例如master_manger，master_master_switch等；MHA Manger也依赖于perl模块，具体如下：
 
（1）安装MHA Node软件包之前需要安装依赖，我这里使用yum完成，首先epel源要安装注意：刚才已经配置epel源
 
（2）安装MHA Manager 首先安装MHA Manger依赖的perl模块（我这里使用yum安装）：
 
在所有机器上使用yum安装全部依赖

	yum install perl-DBD-MySQL perl-Config-Tiny perl-Log-Dispatch perl-Parallel-ForkManager perl-Time-HiRes perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker perl-CPAN -y

**注！**在RHEL系统中安装了epel源之后依旧会出现缺少依赖包问题


**（2）获取MHA相关包，在所有的节点安装mha-node：**

相关下载地址：  
阿里云盘分享：https://www.aliyundrive.com/s/cMNxKg6srT6  
mha_manager_0.55 (RHEL/CentOS 4,5)  
https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/mysql-master-ha/mha4mysql-manager-0.55-1.el5.noarch.rpm  
mha_manager_0.55 (RHEL/CentOS 6)  
https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/mysql-master-ha/mha4mysql-manager-0.55-0.el6.noarch.rpm  

mha_manager_0.54 (RHEL/CentOS 4,5)  
https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/mysql-master-ha/mha4mysql-manager-0.54-0.el5.noarch.rpm  
mha_manager_0.54 (RHEL/CentOS 6)  
https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/mysql-master-ha/mha4mysql-manager-0.54-0.el6.noarch.rpm  

mha_node_0.54 (RHEL/CentOS 4,5)  
https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/mysql-master-ha/mha4mysql-node-0.54-1.el5.noarch.rpm  
mha_node_0.54 (RHEL/CentOS 6)  
https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/mysql-master-ha/mha4mysql-node-0.54-0.el6.noarch.rpm  

 
mha_node安装完成后会在/usr/bin/目录下生成以下脚本文件：  
```shell
[root@ctos7mini ~]& rpm -ivh mha4mysql-node-0.54-0.el6.noarch.rpm
[root@ctos7mini ~]& cd /usr/bin/
[root@ctos7mini bin]& ll app* filter* purge* save*
-rwxr-xr-x. 1 root root 15977 Dec  1  2012 apply_diff_relay_logs
-rwxr-xr-x. 1 root root  4807 Dec  1  2012 filter_mysqlbinlog
-rwxr-xr-x. 1 root root  7401 Dec  1  2012 purge_relay_logs
-rwxr-xr-x. 1 root root  7263 Dec  1  2012 save_binary_logs
[root@ctos7mini bin]&
```
**（3）安装MHA Manager**
 
安装MHA Manager软件包：
```xml
[root@ctos7mini mysql_master_ha]# rpm -ivh mha4mysql-manager-0.54-0.el6.noarch.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:mha4mysql-manager-0.54-0.el6     ################################# [100%]
[root@ctos7mini mysql_master_ha]#
```
**注意！** 如果在RHEL上可能出现无法安装MHA Manager，出现缺少依赖，提示需要安装`perl-Config-Tiny`  
需要在rhel系统手动下载安装：  

	rpm -ivh http://mirror.centos.org/centos/7/os/x86_64/Packages/perl-Config-Tiny-2.14-7.el7.noarch.rpm

MHA Manager安装完成后会在/usr/bin目录下面生成以下脚本文件，前面已经说过这些脚本的作用，这里不再重复
```shell
[root@ctos7mini mysql_master_ha]&ll /usr/bin/mast*
-rwxr-xr-x. 1 root root 1995 Dec  1  2012 /usr/bin/masterha_check_repl
-rwxr-xr-x. 1 root root 1779 Dec  1  2012 /usr/bin/masterha_check_ssh
-rwxr-xr-x. 1 root root 1865 Dec  1  2012 /usr/bin/masterha_check_status
-rwxr-xr-x. 1 root root 3201 Dec  1  2012 /usr/bin/masterha_conf_host
-rwxr-xr-x. 1 root root 2517 Dec  1  2012 /usr/bin/masterha_manager
-rwxr-xr-x. 1 root root 2165 Dec  1  2012 /usr/bin/masterha_master_monitor
-rwxr-xr-x. 1 root root 2373 Dec  1  2012 /usr/bin/masterha_master_switch
-rwxr-xr-x. 1 root root 3879 Dec  1  2012 /usr/bin/masterha_secondary_check
-rwxr-xr-x. 1 root root 1739 Dec  1  2012 /usr/bin/masterha_stop
[root@ctos7mini mysql_master_ha]& 
```

**（4）配置所有主机相互SSH登录无密码验证（使用key登录，工作中常用）**  
有一点需要注意：不能禁止 password 登陆，否则会出现错误

在所有主机上配置ssh免密码登录：
```mbox
[root@rhel7 ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 回车
Enter passphrase (empty for no passphrase): 回车
Enter same passphrase again: 回车
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c2:95:fa:7e:f5:ef:07:3f:70:a2:ec:bb:3a:5a:9f:d9 root@rhel7
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         .       |
|        o        |
|     . o         |
|      + S        |
|       o    .o.. |
|        ......+o |
|       ...oo+ ..o|
|       .oo+BoE o=|
+-----------------+
[root@rhel7 ~]#
```
```shell
[root@rhel7 ~]& ssh-copy-id root@192.168.73.20
[root@rhel7 ~]& ssh-copy-id root@192.168.73.22
[root@rhel7 ~]& ssh-copy-id root@192.168.73.23
```
其他主机重复以上操作

**ssh配置扩展**

> 解决rhel连接ssh等待时间过久的问题

```shell
[root@rhel7 ~]& vim /etc/ssh/sshd_config

# 去掉注释，改为no
UseDNS no
```

## 2. 搭建MySQL主从复制环境

注意：binlog-do-db 和 replicate-ignore-db 设置必须相同；MHA 在启动时候会检测过滤规则，如果过滤规则不同，MHA 不启动监控和故障转移

### 在192.168.73.21上配置MySQL master

**创建需要同步的数据库**
```sql
mysql -uroot -p123456
mysql> CREATE DATABASE ha DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> use HA;
mysql> CREATE TABLE test(id int,name varchar(20));
```

**配置`my.cnf`，添加需要的参数：**
```ini
# 允许远程访问
bind-address    = 0.0.0.0

# 启用二进制日志
log-bin=mysql-bin-master
# 本机数据库ID标识
server-id=1
# 可以被从服务器复制的库，二进制需要同步的数据库名
binlog-do-db=ha
# 不可以被从服务器复制的库
binlog-ignore-db=mysql

# [可选]自MySQL5.6版本，引入了新密码校验插件validate_password, 用于管理用户密码长度、强度等，保障账号的安全性
validate-password=off 
```
文档：安装validate_password插件.md  
http://note.youdao.com/noteshare?id=3808ed864a33805181a8cbea885a6784&sub=02140C7F57F9404499D06E2EF7006296

重启MySQL

**授权**
```sql
mysql> grant replication slave on *.* to slave@'192.168.73.%' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
查看状态信息
```sql
mysql> show master status;
+-------------------------+----------+--------------+------------------+-------------------+
| File                    | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------------+----------+--------------+------------------+-------------------+
| mysql-bin-master.000001 |      154 | ha           | mysql            |                   |
+-------------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>
```
### 在192.168.73.22上配置MySQL slave

**配置`my.cnf`，添加需要的参数：**

```ini
bind-address    = 0.0.0.0

log-bin=mysql-slave1
server-id=2
binlog-do-db=ha
binlog-ignore-db=mysql
#只有开启log_slave_updates，从库binlog才会记录主库同步的操作日志
log_slave_updates=1
```
重启MySQL

**授权并建立主从关系**
```sql
mysql> grant replication slave on *.* to slave@'192.168.73.%' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> change master to master_host='192.168.73.21',master_user='slave',master_password='123456';
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> show master status;
+---------------------+----------+--------------+------------------+-------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------------+----------+--------------+------------------+-------------------+
| mysql-slave1.000002 |      154 | ha           | mysql            |                   |
+---------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

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

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.73.21
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin-master.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: ctos_sqlsrv-relay-bin.000011
                Relay_Log_Pos: 381
        Relay_Master_Log_File: mysql-bin-master.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
.................................................省略.......................................................
                  Master_UUID: 8f5a42da-cd2d-11ec-8c02-000c29f9275c
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
.................................................省略........................................................
1 row in set (0.00 sec)

mysql>
```

### 在192.168.73.23上配置MySQL slave

**配置`my.cnf`，添加需要的参数：**
```ini
bind-address    = 0.0.0.0

log-bin=mysql-slave2
server-id=3
binlog-do-db=ha
binlog-ignore-db=mysql
log_slave_updates=1
```

重启MySQL

**授权并建立主从关系**
```sql
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

mysql> show master status;
+---------------------+----------+--------------+------------------+-------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------------+----------+--------------+------------------+-------------------+
| mysql-slave2.000002 |      601 | ha           | mysql            |                   |
+---------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> change master to master_host='192.168.73.21',master_user='slave',master_password='123456';
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.73.21
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin-master.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: rhel_sqlsrv-relay-bin.000003
                Relay_Log_Pos: 381
        Relay_Master_Log_File: mysql-bin-master.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
.................................................省略.......................................................
             Master_Server_Id: 1
                  Master_UUID: 8f5a42da-cd2d-11ec-8c02-000c29f9275c
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
.................................................省略.......................................................
1 row in set (0.00 sec)

mysql>
```

**两台slave服务器设置read_only（从库对外提供读服务，只所以没有写进配置文件，是因为slave随时会提升为master）**

```sql
mysql> set global read_only=1;
Query OK, 0 rows affected (0.00 sec)

mysql>
```

**创建监控用户（在主从上都执行）：**
```sql
grant all privileges on *.* to 'root'@'192.168.10.%' identified  by '123456';

flush privileges;
```
到这里整个集群环境已经搭建完毕，剩下的就是配置MHA软件了

## 3. 配置MHA

**（1）创建MHA的工作目录，并且创建相关配置文件（在软件包解压后的目录里面有样例配置文件）**

```shell
[root@ctos7mini ~]& mkdir -p /etc/masterha
[root@ctos7mini ~]& mkdir -p /var/log/masterha/app1
[root@ctos7mini ~]& vim /etc/masterha/app1.cnf
```
修改app1.cnf配置文件，修改后的文件内容如下（注意，配置文件中的注释需要去掉，我这里是为了解释清楚）：

**参数说明**
```c
[server default]
manager_workdir=/var/log/masterha/app1            //设置manager的工作目录
manager_log=/var/log/masterha/app1/manager.log          //设置manager的日志
master_binlog_dir=/data/mysql                         //设置master 保存binlog的位置，以便MHA可以找到master的日志，我这里的也就是mysql的数据目录
master_ip_failover_script= /usr/local/bin/master_ip_failover    //设置自动failover时候的切换脚本
master_ip_online_change_script= /usr/local/bin/master_ip_online_change  //设置手动切换时候的切换脚本
password=123456         //设置mysql中root用户的密码，这个密码是前文中创建监控用户的那个密码
user=root               设置监控用户root
ping_interval=1         //设置监控主库，发送ping包的时间间隔，默认是3秒，尝试三次没有回应的时候自动进行railover
remote_workdir=/tmp     //设置远端mysql在发生切换时binlog的保存位置
repl_password=123456    //设置复制用户的密码
repl_user=repl          //设置复制环境中的复制用户名
report_script=/usr/local/send_report    //设置发生切换后发送的报警的脚本
shutdown_script=""      //设置故障发生后关闭故障主机脚本（该脚本的主要作用是关闭主机放在发生脑裂,这里没有使用）
ssh_user=root           //设置ssh的登录用户名
 
[server1]
hostname=192.168.10.12
port=3306
 
[server2]
hostname=192.168.10.13
port=3306
candidate_master=1   #设置为候选master，如果设置该参数以后，发生主从切换以后将会将此从库提升为主库，即使这个主库不是集群中事件最新的slave
check_repl_delay=0   #默认情况下如果一个slave落后master 100M的relay logs的话，MHA将不会选择该slave作为一个新的master，因为对于这个slave的恢复需要花费很长时间，通过设置check_repl_delay=0,MHA触发切换在选择一个新的master的时候将会忽略复制延时，这个参数对于设置了candidate_master=1的主机非常有用，因为这个候选主在切换的过程中一定是新的master
 
[server3]
hostname=192.168.10.14
port=3306
```

**参考配置：**
```ini
[server default]
manager_workdir=/var/log/masterha/app1
manager_log=/var/log/masterha/app1/manager.log
master_binlog_dir=/var/lib/mysql
master_ip_failover_script= /usr/local/bin/master_ip_failover
master_ip_online_change_script= /usr/local/bin/master_ip_online_change 
password=123456
user=root
ping_interval=1
remote_workdir=/tmp
repl_password=123456
repl_user=slave
#report_script=/usr/local/send_report
shutdown_script=""
ssh_user=root

[server1]
hostname=192.168.73.21
port=3306

[server2]
hostname=192.168.73.22
port=3306
candidate_master=1
check_repl_delay=0

[server3]
hostname=192.168.73.23
port=3306
```
**（2）设置relay log的清除方式（在每个slave节点上）：**

```sql
mysql> set global relay_log_purge=0;
Query OK, 0 rows affected (0.00 sec)

mysql>
```
<span style="color:red">注意：</span>  
MHA在发生切换的过程中，从库的恢复过程中依赖于relay log的相关信息，所以这里要将relay log的自动清除设置为OFF，采用手动清楚relay log的方式。在默认情况下，slave上的中继日志会在SQL线程执行完毕后被自动删除。**但是在MHA环境中，这些中继日志在恢复其它slave时可能会被用到，因此需要禁用中继日志的自动删除功能。**定期清楚中继日志需要考虑到复制延时的问题。在ext3文件系统下，删除大的文件需要一定的时间，会导致严重的复制延时。为了避免复制延时，需要暂时为中继日志创建硬链接，因为在Linux系统中通过硬链接删除大文件速度会很快。（在MySQL数据库中，删除大表时，通常也采用建立硬链接的方式）

**（3）.检查SSH配置**

检查MHA Manger到所有MHA Node的SSH连接状态：

```mbox
[root@ctos7mini ~]& masterha_check_ssh --conf=/etc/masterha/app1.cnf
Fri May  6 22:31:51 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri May  6 22:31:51 2022 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Fri May  6 22:31:51 2022 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Fri May  6 22:31:51 2022 - [info] Starting SSH connection tests..
Fri May  6 22:31:54 2022 - [debug]
Fri May  6 22:31:51 2022 - [debug]  Connecting via SSH from root@192.168.73.21(192.168.73.21:22) to root@192.168.73.22(192.168.73.22:22)..
Fri May  6 22:31:52 2022 - [debug]   ok.
Fri May  6 22:31:52 2022 - [debug]  Connecting via SSH from root@192.168.73.21(192.168.73.21:22) to root@192.168.73.23(192.168.73.23:22)..
Fri May  6 22:31:53 2022 - [debug]   ok.
Fri May  6 22:31:55 2022 - [debug]
Fri May  6 22:31:52 2022 - [debug]  Connecting via SSH from root@192.168.73.23(192.168.73.23:22) to root@192.168.73.21(192.168.73.21:22)..
Fri May  6 22:31:53 2022 - [debug]   ok.
Fri May  6 22:31:53 2022 - [debug]  Connecting via SSH from root@192.168.73.23(192.168.73.23:22) to root@192.168.73.22(192.168.73.22:22)..
Fri May  6 22:31:54 2022 - [debug]   ok.
Fri May  6 22:31:55 2022 - [debug]
Fri May  6 22:31:52 2022 - [debug]  Connecting via SSH from root@192.168.73.22(192.168.73.22:22) to root@192.168.73.21(192.168.73.21:22)..
Fri May  6 22:31:53 2022 - [debug]   ok.
Fri May  6 22:31:53 2022 - [debug]  Connecting via SSH from root@192.168.73.22(192.168.73.22:22) to root@192.168.73.23(192.168.73.23:22)..
Fri May  6 22:31:54 2022 - [debug]   ok.
Fri May  6 22:31:55 2022 - [info] All SSH connection tests passed successfully.
[root@ctos7mini ~]&
```
各个节点ssh验证都是ok才正常

**（4）.检查整个复制环境状况**

配置VIP
```shell
[root@ctos7mini ~]& ifconfig ens33
[root@rhel7 ~]& ip -4 addr show dev ens33
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    inet 192.168.73.21/24 brd 192.168.73.255 scope global ens33
       valid_lft forever preferred_lft forever

[root@ctos7mini ~]& ip addr add 192.168.73.73/24 brd 192.168.10.255 dev ens33 label ens33:1

[root@ctos7mini ~]& ip -4 addr show dev ens33
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    inet 192.168.73.20/24 brd 192.168.73.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet 192.168.73.73/24 brd 192.168.10.255 scope global secondary ens33:1
       valid_lft forever preferred_lft forever
[root@ctos7mini ~]&
```

**创建自动切换脚本**
```shell
[root@ctos7mini ~]& vim /usr/local/bin/master_ip_failover
```

```perl
#!/usr/bin/env perl
use strict;
use warnings FATAL => 'all';

use Getopt::Long;

my (
    $command,          $ssh_user,        $orig_master_host, $orig_master_ip,
    $orig_master_port, $new_master_host, $new_master_ip,    $new_master_port
);

my $vip = '192.168.73.73';
my $brdc = '192.168.73.255';
my $ifdev = 'ens33';
my $key = '1';
my $ssh_start_vip = "/usr/sbin/ip addr add $vip/24 brd $brdc dev $ifdev label $ifdev:$key;/usr/sbin/arping -q -A -c 1 -I $ifdev:$key $vip;iptables -F;";
my $ssh_stop_vip = "/usr/sbin/ip addr del $vip/24 dev $ifdev label $ifdev:$key";

GetOptions(
    'command=s'          => \$command,
    'ssh_user=s'         => \$ssh_user,
    'orig_master_host=s' => \$orig_master_host,
    'orig_master_ip=s'   => \$orig_master_ip,
    'orig_master_port=i' => \$orig_master_port,
    'new_master_host=s'  => \$new_master_host,
    'new_master_ip=s'   => \$new_master_ip,
    'new_master_port=i'  => \$new_master_port,
);

exit &main();

sub main {

    print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";

    if ( $command eq "stop" || $command eq "stopssh" ) {

        my $exit_code = 1;
        eval {
            print "Disabling the VIP on old master: $orig_master_host \n";
            &stop_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn "Got Error: $@\n";
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "start" ) {

        my $exit_code = 10;
        eval {
            print "Enabling the VIP - $vip on the new master - $new_master_host \n";
            &start_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn $@;
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "status" ) {
        print "Checking the Status of the script.. OK \n";
        exit 0;
    }
    else {
        &usage();
        exit 1;
    }
}
sub start_vip() {
    `ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
# A simple system call that disable the VIP on the old_master
sub stop_vip() {
    `ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}

sub usage {
    print
    "Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}
```

```shell
[root@ctos7mini ~]& vim /usr/local/bin/master_ip_online_change
```

```perl
#!/usr/bin/env perl
use strict;
use warnings FATAL =>'all';

use Getopt::Long;

#my $vip = '172.25.0.100/24';  # Virtual IP
my $vip = '192.168.73.73/24';  # Virtual IP
my $key = "1";
my $ssh_start_vip = "/sbin/ifconfig ens33:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig ens33:$key down";
my $exit_code = 0;

my (
  $command,              $orig_master_is_new_slave, $orig_master_host,
  $orig_master_ip,       $orig_master_port,         $orig_master_user,
  $orig_master_password, $orig_master_ssh_user,     $new_master_host,
  $new_master_ip,        $new_master_port,          $new_master_user,
  $new_master_password,  $new_master_ssh_user,
);
GetOptions(
  'command=s'                => \$command,
  'orig_master_is_new_slave' => \$orig_master_is_new_slave,
  'orig_master_host=s'       => \$orig_master_host,
  'orig_master_ip=s'         => \$orig_master_ip,
  'orig_master_port=i'       => \$orig_master_port,
  'orig_master_user=s'       => \$orig_master_user,
  'orig_master_password=s'   => \$orig_master_password,
  'orig_master_ssh_user=s'   => \$orig_master_ssh_user,
  'new_master_host=s'        => \$new_master_host,
  'new_master_ip=s'          => \$new_master_ip,
  'new_master_port=i'        => \$new_master_port,
  'new_master_user=s'        => \$new_master_user,
  'new_master_password=s'    => \$new_master_password,
  'new_master_ssh_user=s'    => \$new_master_ssh_user,
);


exit &main();

sub main {

#print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";

if ( $command eq "stop" || $command eq "stopssh" ) {

        # $orig_master_host, $orig_master_ip, $orig_master_port are passed.
        # If you manage master ip address at global catalog database,
        # invalidate orig_master_ip here.
        my $exit_code = 1;
        eval {
            print "\n\n\n***************************************************************\n";

            print "Disabling the VIP - $vip on old master: $orig_master_host\n";
            print "***************************************************************\n\n\n\n";

&stop_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn "Got Error: $@\n";
            exit $exit_code;
        }
        exit $exit_code;
}
elsif ( $command eq "start" ) {

        # all arguments are passed.
        # If you manage master ip address at global catalog database,
        # activate new_master_ip here.
        # You can also grant write access (create user, set read_only=0, etc) here.
my $exit_code = 10;
        eval {
            print "\n\n\n***************************************************************\n";

            print "Enabling the VIP - $vip on new master: $new_master_host \n";
            print "***************************************************************\n\n\n\n";

&start_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn $@;
            exit $exit_code;
        }
        exit $exit_code;
}
elsif ( $command eq "status" ) {
        print "Checking the Status of the script.. OK \n";
        `ssh $orig_master_ssh_user\@$orig_master_host \" $ssh_start_vip \"`;
        exit 0;
}
else {
&usage();
        exit 1;
}
}

# A simple system call that enable the VIP on the new master
sub start_vip() {
`ssh $new_master_ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
# A simple system call that disable the VIP on the old_master
sub stop_vip() {
`ssh $orig_master_ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}

sub usage {
print
"Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}
[root@ctos7mini ~]#
```

修改文件权限
```shell
[root@ctos7mini ~]& chmod 755 /usr/local/bin/master_ip_failover
[root@ctos7mini ~]& chmod 755 /usr/local/bin/master_ip_online_change
[root@ctos7mini ~]& grep /usr/local/bin/master_ip_failover /etc/masterha/app1.cnf
master_ip_failover_script= /usr/local/bin/master_ip_failover
[root@ctos7mini ~]&
```

**通过masterha_check_repl脚本查看整个集群的状态**

```mbox
[root@ctos7mini ~]& masterha_check_repl --conf=/etc/masterha/app1.cnf
Fri May  6 22:54:08 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri May  6 22:54:08 2022 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Fri May  6 22:54:08 2022 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Fri May  6 22:54:08 2022 - [info] MHA::MasterMonitor version 0.54.
Fri May  6 22:54:09 2022 - [info] Dead Servers:
Fri May  6 22:54:09 2022 - [info] Alive Servers:
Fri May  6 22:54:09 2022 - [info]   192.168.73.21(192.168.73.21:3306)
Fri May  6 22:54:09 2022 - [info]   192.168.73.22(192.168.73.22:3306)
Fri May  6 22:54:09 2022 - [info]   192.168.73.23(192.168.73.23:3306)
Fri May  6 22:54:09 2022 - [info] Alive Slaves:
Fri May  6 22:54:09 2022 - [info]   192.168.73.22(192.168.73.22:3306)  Version=5.7.24-log (oldest major version between slaves) log-bin:enabled
Fri May  6 22:54:09 2022 - [info]     Replicating from 192.168.73.21(192.168.73.21:3306)
Fri May  6 22:54:09 2022 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri May  6 22:54:09 2022 - [info]   192.168.73.23(192.168.73.23:3306)  Version=5.7.24-log (oldest major version between slaves) log-bin:enabled
Fri May  6 22:54:09 2022 - [info]     Replicating from 192.168.73.21(192.168.73.21:3306)
Fri May  6 22:54:09 2022 - [info] Current Alive Master: 192.168.73.21(192.168.73.21:3306)
Fri May  6 22:54:09 2022 - [info] Checking slave configurations..
Fri May  6 22:54:09 2022 - [info] Checking replication filtering settings..
Fri May  6 22:54:09 2022 - [info]  binlog_do_db= ha, binlog_ignore_db= mysql
Fri May  6 22:54:09 2022 - [info]  Replication filtering check ok.
Fri May  6 22:54:09 2022 - [info] Starting SSH connection tests..
Fri May  6 22:54:12 2022 - [info] All SSH connection tests passed successfully.
Fri May  6 22:54:12 2022 - [info] Checking MHA Node version..
Fri May  6 22:54:14 2022 - [info]  Version check ok.
Fri May  6 22:54:14 2022 - [info] Checking SSH publickey authentication settings on the current master..
Fri May  6 22:54:14 2022 - [info] HealthCheck: SSH to 192.168.73.21 is reachable.
Fri May  6 22:54:15 2022 - [info] Master MHA Node version is 0.54.
Fri May  6 22:54:15 2022 - [info] Checking recovery script configurations on the current master..
Fri May  6 22:54:15 2022 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql --output_file=/tmp/save_binary_logs_test --manager_version=0.54 --start_file=mysql-bin-master.000002
Fri May  6 22:54:15 2022 - [info]   Connecting to root@192.168.73.21(192.168.73.21)..
  Creating /tmp if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /var/lib/mysql, up to mysql-bin-master.000002
Fri May  6 22:54:15 2022 - [info] Master setting check done.
Fri May  6 22:54:15 2022 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Fri May  6 22:54:15 2022 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.73.22 --slave_ip=192.168.73.22 --slave_port=3306 --workdir=/tmp --target_version=5.7.24-log --manager_version=0.54 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Fri May  6 22:54:15 2022 - [info]   Connecting to root@192.168.73.22(192.168.73.22:22)..
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to rhel_sqlsrv-relay-bin.000003
    Temporary relay log file is /var/lib/mysql/rhel_sqlsrv-relay-bin.000003
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri May  6 22:54:26 2022 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='root' --slave_host=192.168.73.23 --slave_ip=192.168.73.23 --slave_port=3306 --workdir=/tmp --target_version=5.7.24-log --manager_version=0.54 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Fri May  6 22:54:26 2022 - [info]   Connecting to root@192.168.73.23(192.168.73.23:22)..
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to ctos_sqlsrv-relay-bin.000011
    Temporary relay log file is /var/lib/mysql/ctos_sqlsrv-relay-bin.000011
    Testing mysql connection and privileges..mysql: [Warning] Using a password on the command line interface can be insecure.
 done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Fri May  6 22:54:37 2022 - [info] Slaves settings check done.
Fri May  6 22:54:37 2022 - [info]
192.168.73.21 (current master)
 +--192.168.73.22
 +--192.168.73.23

Fri May  6 22:54:37 2022 - [info] Checking replication health on 192.168.73.22..
Fri May  6 22:54:37 2022 - [info]  ok.
Fri May  6 22:54:37 2022 - [info] Checking replication health on 192.168.73.23..
Fri May  6 22:54:37 2022 - [info]  ok.
Fri May  6 22:54:37 2022 - [info] Checking master_ip_failover_script status:
Fri May  6 22:54:37 2022 - [info]   /usr/local/bin/master_ip_failover --command=status --ssh_user=root --orig_master_host=192.168.73.21 --orig_master_ip=192.168.73.21 --orig_master_port=3306


IN SCRIPT TEST====/usr/sbin/ip addr del 192.168.73.73/24 dev ens33 label ens33:1==/usr/sbin/ip addr add 192.168.73.73/24 brd 192.168.73.255 dev ens33 label ens33:1;/usr/sbin/arping -q -A -c 1 -I ens33:1 192.168.73.73;iptables -F;===

Checking the Status of the script.. OK
Fri May  6 22:54:37 2022 - [info]  OK.
Fri May  6 22:54:37 2022 - [warning] shutdown_script is not defined.
Fri May  6 22:54:37 2022 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.
```
`MySQL Replication Health is NOT OK!` 如果提示这个不OK，说明有问题。（解决方法参考最后章节）
`MySQL Replication Health is OK.`显示Ok ，正常！

参数调试：
```shell
ln -s /usr/local/mysql57/bin/mysqlbinlog /usr/local/bin/mysqlbinlog
ln -s /usr/local/mysql57/bin/mysql /usr/local/bin/mysql
[root@ctos7mini ~]& grep master_ip_failover /etc/masterha/app1.cnf
#master_ip_failover_script= /usr/local/bin/master_ip_failover
```

**（5）检查MHA Manager的状态：**

通过master_check_status脚本查看Manager的状态：
 
```shell
[root@ctos7mini ~]& masterha_check_status --conf=/etc/masterha/app1.cnf
app1 is stopped(2:NOT_RUNNING).
[root@ctos7mini ~]&
```
注意：如果正常，会显示"PING_OK"，否则会显示"NOT_RUNNING"，这代表MHA监控没有开启

**（6）开启MHA Manager监控**

	nohup masterha_manager --conf=/etc/masterha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/masterha/app1/manager.log 2>&1 &

启动参数介绍：
```
--remove_dead_master_conf    该参数代表当发生主从切换后，老的主库的ip将会从配置文件中移除
--manger_log                 日志存放位置
--ignore_last_failover       在缺省情况下，如果MHA检测到连续发生宕机，且两次宕机间隔不足8小时的话，则不会进行Failover，之所以这样限制是为了避免ping-pong效应。该参数代表忽略上次MHA触发切换产生的文件，默认情况下，MHA发生切换后会在日志目录，也就是上面我设置的/data产生app1.failover.complete文件，下次再次切换的时候如果发现该目录下存在该文件将不允许触发切换，除非在第一次切换后收到删除该文件，为了方便，这里设置为--ignore_last_failover
```

继续检查MHA Manager监控是否正常：
```shell
[root@ctos7mini ~]& masterha_check_status --conf=/etc/masterha/app1.cnf
app1 (pid:2803) is running(0:PING_OK), master:192.168.73.21
[6]-  Exit 1                  nohup masterha_manager --conf=/etc/masterha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/masterha/app1/manager.log 2>&1
[7]+  Exit 1                  nohup masterha_manager --conf=/etc/masterha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/masterha/app1/manager.log 2>&1
[root@ctos7mini ~]& jobs
[1]+  Running                 nohup masterha_manager --conf=/etc/masterha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/masterha/app1/manager.log 2>&1 &
[root@ctos7mini ~]&
```
可以看见已经在监控了，而且master的主机为192.168.73.21

**（7）查看启动日志**
```mbox
[root@ctos7mini ~]& tail -n 10 /var/log/masterha/app1/manager.log
Fri May  6 23:16:28 2022 - [info]  OK.
Fri May  6 23:16:28 2022 - [warning] shutdown_script is not defined.
Fri May  6 23:16:28 2022 - [info] Set master ping interval 1 seconds.
Fri May  6 23:16:28 2022 - [warning] secondary_check_script is not defined. It is highly recommended setting it to check master reachability from two or more routes.
Fri May  6 23:16:28 2022 - [info] Starting ping health check on 192.168.73.21(192.168.73.21:3306)..
Fri May  6 23:16:32 2022 - [warning] Got error when monitoring master:  at /usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm line 446.
Fri May  6 23:16:32 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln448] Target master's advisory lock is already held by someone. Please check whether you monitor the same master from multiple monitoring processes.
Fri May  6 23:16:32 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln468] Error happened on health checking.  at /usr/bin/masterha_manager line 50.
Fri May  6 23:16:32 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln482] Error happened on monitoring servers.
Fri May  6 23:16:32 2022 - [info] Got exit code 1 (Not master dead).
[root@ctos7mini ~]&
```
清空日志，然后关闭监控
```shell
[root@ctos7mini ~]& echo > /var/log/masterha/app1/manager.log

[root@ctos7mini ~]& masterha_stop --conf=/etc/masterha/app1.cnf
Stopped app1 successfully.
[1]+  Exit 1                  nohup masterha_manager --conf=/etc/masterha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/masterha/app1/manager.log 2>&1
[root@ctos7mini ~]&
```
继续开启监控

	nohup masterha_manager --conf=/etc/masterha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/masterha/app1/manager.log 2>&1 &
	
观看日志
```shell
[root@ctos7mini ~]& tail -n 10 /var/log/masterha/app1/manager.log

IN SCRIPT TEST====/usr/sbin/ip addr del 192.168.73.73/24 dev ens33 label ens33:1==/usr/sbin/ip addr add 192.168.73.73/24 brd 192.168.73.255 dev ens33 label ens33:1;/usr/sbin/arping -q -A -c 1 -I ens33:1 192.168.73.73;iptables -F;===

Checking the Status of the script.. OK
Fri May  6 23:30:53 2022 - [info]  OK.
Fri May  6 23:30:53 2022 - [warning] shutdown_script is not defined.
Fri May  6 23:30:53 2022 - [info] Set master ping interval 1 seconds.
Fri May  6 23:30:53 2022 - [warning] secondary_check_script is not defined. It is highly recommended setting it to check master reachability from two or more routes.
Fri May  6 23:30:53 2022 - [info] Starting ping health check on 192.168.73.21(192.168.73.21:3306)..
Fri May  6 23:30:53 2022 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
[root@ctos7mini ~]&
```
其中`Ping(SELECT) succeeded, waiting until MySQL doesn't respond..`说明整个系统已经开始监控了


**（9）模拟故障**

停掉MySQL主库

	[root@rhel7 ~]# systemctl stop mysqld

查看日志，观察集群状况
```mbox
[root@ctos7mini ~]& tail -n 20 /var/log/masterha/app1/manager.log

----- Failover Report -----

app1: MySQL Master failover 192.168.73.21 to 192.168.73.22 succeeded

Master 192.168.73.21 is down!

Check MHA Manager logs at ctos7mini:/var/log/masterha/app1/manager.log for details.

Started automated(non-interactive) failover.
Invalidated master IP address on 192.168.73.21.
The latest slave 192.168.73.22(192.168.73.22:3306) has all relay logs for recovery.
Selected 192.168.73.22 as a new master.
192.168.73.22: OK: Applying all logs succeeded.
192.168.73.22: OK: Activated master IP address.
192.168.73.23: This host has the latest relay log events.
Generating relay diff files from the latest slave succeeded.
192.168.73.23: OK: Applying all logs succeeded. Slave started, replicating from 192.168.73.22.
192.168.73.22: Resetting slave info succeeded.
Master failover to 192.168.73.22(192.168.73.22:3306) completed successfully.
[root@ctos7mini ~]&
```

查看192.168.73.22的slave状态：
```sql
mysql> show slave status\G
Empty set (0.00 sec)

mysql>
```

查看192.168.73.23的slave状态
```sql
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.73.22
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-slave2.000002
          Read_Master_Log_Pos: 1048
               Relay_Log_File: ctos_sqlsrv-relay-bin.000002
                Relay_Log_Pos: 323
        Relay_Master_Log_File: mysql-slave2.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
.................................................省略.......................................................
             Master_Server_Id: 3
                  Master_UUID: 90e5e492-cd2d-11ec-88d4-000c299254eb
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
.................................................省略.......................................................
1 row in set (0.00 sec)

mysql>
```
主库变成了192.168.73.22！

查看备用主库的IP地址：
```mbox
[root@ctos_sqlsrv ~]& ip addr show dev ens33
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:27:5c:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.73.23/24 brd 192.168.73.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet 192.168.73.73/24 brd 192.168.73.255 scope global secondary ens33:1
       valid_lft forever preferred_lft forever
    inet6 fe80::412f:a64f:e2ff:e154/64 scope link
       valid_lft forever preferred_lft forever
[root@ctos_sqlsrv ~]&
```
ip漂移到了备用master上


开启down掉的master，手动进行切换主库：
```mbox
 [root@ctos7mini ~]& masterha_master_switch --conf=/etc/masterha/app1.cnf --master_state=alive --new_master_host=192.168.73.21 --new_master_port=3306 --orig_master_is_new_slave --running_updates_limit=10000
Sat May  7 03:08:13 2022 - [info] MHA::MasterRotate version 0.54.
Sat May  7 03:08:13 2022 - [info] Starting online master switch..
Sat May  7 03:08:13 2022 - [info]
Sat May  7 03:08:13 2022 - [info] * Phase 1: Configuration Check Phase..
Sat May  7 03:08:13 2022 - [info]
Sat May  7 03:08:13 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sat May  7 03:08:13 2022 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Sat May  7 03:08:13 2022 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Sat May  7 03:08:14 2022 - [info] Current Alive Master: 192.168.73.21(192.168.73.21:3306)
Sat May  7 03:08:14 2022 - [info] Alive Slaves:
Sat May  7 03:08:14 2022 - [info]   192.168.73.22(192.168.73.22:3306)  Version=5.7.24-log (oldest major version between slaves) log-bin:enabled
Sat May  7 03:08:14 2022 - [info]     Replicating from 192.168.73.21(192.168.73.21:3306)
Sat May  7 03:08:14 2022 - [info]     Primary candidate for the new Master (candidate_master is set)
Sat May  7 03:08:14 2022 - [info]   192.168.73.23(192.168.73.23:3306)  Version=5.7.24-log (oldest major version between slaves) log-bin:enabled
Sat May  7 03:08:14 2022 - [info]     Replicating from 192.168.73.21(192.168.73.21:3306)

It is better to execute FLUSH NO_WRITE_TO_BINLOG TABLES on the master before switching. Is it ok to execute on 192.168.73.21(192.168.73.21:3306)? (YES/no): yes
Sat May  7 03:08:17 2022 - [info] Executing FLUSH NO_WRITE_TO_BINLOG TABLES. This may take long time..
Sat May  7 03:08:17 2022 - [info]  ok.
Sat May  7 03:08:17 2022 - [info] Checking MHA is not monitoring or doing failover..
Sat May  7 03:08:17 2022 - [info] Checking replication health on 192.168.73.22..
Sat May  7 03:08:17 2022 - [info]  ok.
Sat May  7 03:08:17 2022 - [info] Checking replication health on 192.168.73.23..
Sat May  7 03:08:17 2022 - [info]  ok.
Sat May  7 03:08:17 2022 - [info] 192.168.73.21 can be new master.
Sat May  7 03:08:17 2022 - [info]
From:
192.168.73.21 (current master)
 +--192.168.73.22
 +--192.168.73.23

To:
192.168.73.21 (new master)
 +--192.168.73.22
 +--192.168.73.23
 +--192.168.73.21

Starting master switch from 192.168.73.21(192.168.73.21:3306) to 192.168.73.21(192.168.73.21:3306)? (yes/NO): yes
Sat May  7 03:08:19 2022 - [info] Checking whether 192.168.73.21(192.168.73.21:3306) is ok for the new master..
Sat May  7 03:08:19 2022 - [info]  ok.
Sat May  7 03:08:19 2022 - [info] 192.168.73.21(192.168.73.21:3306): SHOW SLAVE STATUS returned empty result. To check replication filtering rules, temporarily executing CHANGE MASTER to a dummy host.
Sat May  7 03:08:19 2022 - [info] 192.168.73.21(192.168.73.21:3306): Resetting slave pointing to the dummy host.
Sat May  7 03:08:19 2022 - [info] ** Phase 1: Configuration Check Phase completed.
Sat May  7 03:08:19 2022 - [info]
Sat May  7 03:08:19 2022 - [info] * Phase 2: Rejecting updates Phase..
Sat May  7 03:08:19 2022 - [info]
Sat May  7 03:08:19 2022 - [info] Executing master ip online change script to disable write on the current master:
Sat May  7 03:08:19 2022 - [info]   /usr/local/bin/master_ip_online_change --command=stop --orig_master_host=192.168.73.21 --orig_master_ip=192.168.73.21 --orig_master_port=3306 --orig_master_user='root' --orig_master_password='ccc.456' --new_master_host=192.168.73.21 --new_master_ip=192.168.73.21 --new_master_port=3306 --new_master_user='root' --new_master_password='ccc.456'



***************************************************************
Disabling the VIP - 192.168.73.73/24 on old master: 192.168.73.21
***************************************************************



Got Error: Use of uninitialized value $orig_master_ssh_user in concatenation (.) or string at /usr/local/bin/master_ip_online_change line 101.

Sat May  7 03:08:19 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ManagerUtil.pm, ln178] Got ERROR:  at /usr/bin/masterha_master_switch line 53.
[root@ctos7mini ~]& 
```

## masterha故障解决

**故障一：**
```xml
Fri May  6 22:43:11 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri May  6 22:43:11 2022 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Fri May  6 22:43:11 2022 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Fri May  6 22:43:11 2022 - [info] MHA::MasterMonitor version 0.54.
Fri May  6 22:43:11 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln255] Got MySQL error when connecting 192.168.73.22(192.168.73.22:3306) :1045:Access denied for user 'root'@'192.168.73.20' (using password: YES), but this is not mysql crash. Check MySQL server settings.
 at /usr/share/perl5/vendor_perl/MHA/ServerManager.pm line 251.
Fri May  6 22:43:11 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln255] Got MySQL error when connecting 192.168.73.23(192.168.73.23:3306) :1045:Access denied for user 'root'@'192.168.73.20' (using password: YES), but this is not mysql crash. Check MySQL server settings.
 at /usr/share/perl5/vendor_perl/MHA/ServerManager.pm line 251.
Fri May  6 22:43:11 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln255] Got MySQL error when connecting 192.168.73.21(192.168.73.21:3306) :1045:Access denied for user 'root'@'192.168.73.20' (using password: YES), but this is not mysql crash. Check MySQL server settings.
 at /usr/share/perl5/vendor_perl/MHA/ServerManager.pm line 251.
Fri May  6 22:43:12 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln263] Got fatal error, stopping operations
Fri May  6 22:43:12 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln386] Error happend on checking configurations.  at /usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm line 300.
Fri May  6 22:43:12 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln482] Error happened on monitoring servers.
Fri May  6 22:43:12 2022 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!
```
> mha配置文件的数据库用户密码不正确，或没开启远程连接

**故障二：**
```xml
Fri May  6 22:47:47 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Fri May  6 22:47:47 2022 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Fri May  6 22:47:47 2022 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Fri May  6 22:47:47 2022 - [info] MHA::MasterMonitor version 0.54.
Fri May  6 22:47:48 2022 - [info] Dead Servers:
Fri May  6 22:47:48 2022 - [info] Alive Servers:
Fri May  6 22:47:48 2022 - [info]   192.168.73.21(192.168.73.21:3306)
Fri May  6 22:47:48 2022 - [info]   192.168.73.22(192.168.73.22:3306)
Fri May  6 22:47:48 2022 - [info]   192.168.73.23(192.168.73.23:3306)
Fri May  6 22:47:48 2022 - [info] Alive Slaves:
Fri May  6 22:47:48 2022 - [info]   192.168.73.22(192.168.73.22:3306)  Version=5.7.24-log (oldest major version between slaves) log-bin:enabled
Fri May  6 22:47:48 2022 - [info]     Replicating from 192.168.73.21(192.168.73.21:3306)
Fri May  6 22:47:48 2022 - [info]     Primary candidate for the new Master (candidate_master is set)
Fri May  6 22:47:48 2022 - [info]   192.168.73.23(192.168.73.23:3306)  Version=5.7.24-log (oldest major version between slaves) log-bin:enabled
Fri May  6 22:47:48 2022 - [info]     Replicating from 192.168.73.21(192.168.73.21:3306)
Fri May  6 22:47:48 2022 - [info] Current Alive Master: 192.168.73.21(192.168.73.21:3306)
Fri May  6 22:47:48 2022 - [info] Checking slave configurations..
Fri May  6 22:47:48 2022 - [info] Checking replication filtering settings..
Fri May  6 22:47:48 2022 - [info]  binlog_do_db= ha, binlog_ignore_db= mysql
Fri May  6 22:47:48 2022 - [info]  Replication filtering check ok.
Fri May  6 22:47:48 2022 - [info] Starting SSH connection tests..
Fri May  6 22:47:52 2022 - [info] All SSH connection tests passed successfully.
Fri May  6 22:47:52 2022 - [info] Checking MHA Node version..
Fri May  6 22:47:52 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ManagerUtil.pm, ln122] Got error when getting node version. Error:
Fri May  6 22:47:52 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ManagerUtil.pm, ln123]
bash: apply_diff_relay_logs: command not found
Fri May  6 22:47:52 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ManagerUtil.pm, ln151] node version on 192.168.73.22 not found! Maybe MHA Node package is not installed?
 at /usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm line 346.
Fri May  6 22:47:52 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln386] Error happend on checking configurations. node version on 192.168.73.22 not found! Maybe MHA Node package is not installed?
 at /usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm line 346.
        ...propagated at /usr/share/perl5/vendor_perl/MHA/ManagerUtil.pm line 152.
Fri May  6 22:47:52 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln482] Error happened on monitoring servers.
Fri May  6 22:47:52 2022 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!
```

> MySQL服务器没有安装mha node工具


**故障三：**

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205061455441.png)

> 导致该故障的原因有很多，大部分是依赖包关系没有解决，具体需要根据实时环境分析解决

- 参考思路：  
确保MySQL的`libs`和`libs-compat`包安装了，它提供了MySQL需要的动态链接库
```
[root@ctos7mini ~]& rpm -qa |grep mysql-community
mysql-community-common-5.7.24-1.el7.x86_64
mysql-community-libs-compat-5.7.32-1.el7.x86_64
mysql-community-libs-5.7.24-1.el7.x86_64
mysql-community-server-5.7.24-1.el7.x86_64
mysql-community-client-5.7.24-1.el7.x86_64
```

**故障四：**
```mbox
Sat May  7 00:13:00 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sat May  7 00:13:00 2022 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Sat May  7 00:13:00 2022 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Sat May  7 00:13:00 2022 - [info] MHA::MasterMonitor version 0.54.
Sat May  7 00:13:01 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln622] Master 192.168.73.21:3306 from which slave 192.168.73.23(192.168.73.23:3306) replicates is not defined in the configuration file!
Sat May  7 00:13:01 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln386] Error happend on checking configurations.  at /usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm line 300.
Sat May  7 00:13:01 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln482] Error happened on monitoring servers.
Sat May  7 00:13:01 2022 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!
```

```mbox
Sat May  7 00:35:16 2022 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sat May  7 00:35:16 2022 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Sat May  7 00:35:16 2022 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Sat May  7 00:35:16 2022 - [info] MHA::MasterMonitor version 0.54.
Sat May  7 00:35:17 2022 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln193] There is no alive slave. We can't do failover
Sat May  7 00:35:17 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln386] Error happend on checking configurations.  at /usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm line 300.
Sat May  7 00:35:17 2022 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln482] Error happened on monitoring servers.
Sat May  7 00:35:17 2022 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!
```
