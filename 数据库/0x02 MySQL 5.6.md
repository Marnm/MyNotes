---
title: 0x02 MySQL 5.6
tags: []
---

## 源码安装MySQL 5.6

**1. 安装依赖包及编译工具**
```
[root@localhost ~]# cd /usr/local/src
[root@localhost src]# yum install ‐y  gcc*  make  ncurses* perl* 
```
**2. 安装CMake编译工具**
```
[root@localhost src]# tar xvf cmake‐2.8.11.2.tar.gz
[root@localhost src]# cd cmake‐2.8.11.2
[root@localhost cmake‐2.8.11.2]# ./configure
[root@localhost cmake‐2.8.11.2]# make
[root@localhost cmake‐2.8.11.2]# make install
```
**3. 配置MySQL**

创建一个用于执行MySQL服务程序的账户
```
[root@localhost cmake‐2.8.11.2]# cd ..
[root@localhost src]# useradd mysql ‐s /sbin/nologin
```
创建用于保存MySQL数据库程序和数据库文件的目录，并把该目录的所有者和所属组身份
修改为mysql  其中，/usr/local/mysql 是用于保存MySQL数据库服务程序的目
录，/usr/local/mysql/var 则是用于保存真实数据库文件的目录
```
[root@localhost src]# mkdir ‐p /usr/local/mysql/var
[root@localhost src]# chown ‐Rf mysql:mysql /usr/local/mysql
```
解压、编译、安装 MySQL 数据库服务程序。在编译数据库时使用的是 cmake 命令，其
中，-DCMAKE_INSTALL_PREFIX 参数用于定义数据库服务程序的保存目录，-
DMYSQL_DATADIR 参数用于定义真实数据库文件的目录，-DSYSCONFDIR 则是定义
MySQL 数据库配置文件的保存目录
```
[root@localhost src]# tar xvf mysql‐5.6.19.tar.gz
[root@localhost src]# cd mysql‐5.6.19
[root@localhost mysql‐5.6.19]# cmake . ‐DCMAKE_INSTALL_PREFIX=/usr/local/mysql ‐DMYSQL_DATADIR=/usr/local/mysql/var ‐DSYSCONFDIR=/etc
[root@localhost mysql‐5.6.19]# make && make install
```
为了让 MySQL 数据库程序正常运转起来，需要先删除 /etc 目录中的默认配置文件，然后在 MySQL 数据库程序的保存目录 scripts 内找到一个名为 mysql_install_db 的脚本程序，执行这个脚本程序并使用 --user 参数指定 MySQL 服务的对应账号名称（在前面步骤已经创建），使用 --basedir 参数指定MySQL 服务程序的保存目录，使用 --datadir 参数
指定 MySQL 真实数据库的文件保存目录，这样即可生成系统数据库文件，也会生成新的MySQL 服务配置文件
```
 [root@localhost mysql‐5.6.19]# rm ‐rf /etc/my.cnf
 [root@localhost mysql‐5.6.19]# cd /usr/local/mysql
 [root@localhost mysql]# ./scripts/mysql_install_db ‐‐user=mysql ‐‐basedir=/usr/local/mysql ‐‐datadir=/usr/local/mysql/var
```
把Linux系统新生成的MySQL数据库配置文件链接到/etc目录中，然后把程序目录中的开机程序文件复制到/etc/rc.d/init.d目录中，以便通过service命令来管理MySQL数据库服务程序。记得把数据库脚本文件的权限修改成755以便于让用户有执行该脚本的权限
```
 [root@localhost mysql]# ls
 bin data include lib my.cnf README share support‐files
 COPYING docs INSTALL‐BINARY man mysql‐test scripts sql‐bench var
 [root@localhost mysql]# ln ‐s my.cnf /etc/my.cnf
 [root@localhost mysql]# cp ./support‐files/mysql.server /etc/rc.d/init.d/mysqld
 [root@localhost mysql]# chmod 755 /etc/rc.d/init.d/mysqld
```
编辑刚刚复制的MySQL数据库脚本文件，把第46、47行的basedir与datadir参数分别修改
为MySQL数据库程序的保存目录和真实数据库的文件内容
```shell
 [root@localhost mysql]# vim /etc/rc.d/init.d/mysqld
  46 basedir=/usr/local/mysql
  47 datadir=/usr/local/mysql/var
```
配置好脚本文件后便可以用service命令启动mysqld数据库服务了。mysqld是MySQL数据
库程序的服务名称，注意不要写错。再使用chkconfig命令把mysqld服务程序加入到开机
启动项中
```
 [root@localhost mysql]# service mysqld start
 Unit mysqld.service could not be found.
 Starting MySQL. SUCCESS!
 [root@localhost mysql]# chkconfig mysqld on
```
在/etc/profile配置文件的最后加入export PATH=$PATH:/usr/local/mysql/bin
```shell
 [root@localhost mysql]# vim /etc/profile
  export PATH=$PATH:/usr/local/mysql/bin
```
MySQL数据库服务程序还会调用到一些程序文件和函数库文件。由于当前是通过源码包方式安装MySQL数据库，因此现在也必须以手动方式把这些文件链接过来
``` 
 [root@localhost mysql]# mkdir  /var/lib/mysql
 [root@localhost mysql]# ln ‐s /usr/local/mysql/lib/mysql  /usr/lib/mysql
 [root@localhost mysql]# ln ‐s /var/lib/mysql/mysql.sock  /tmp/mysql.sock 
 [root@localhost mysql]# ln ‐s /usr/local/mysql/include/mysql  /usr/include/mysql
```
现在，MySQL数据库服务程序已经启动，调用的各个函数文件已经就位，PATH环境变量
中也加入了MySQL数据库命令的所在目录。接下来准备对MySQL数据库进行初始化，这个
初始化的配置过程与MariaDB数据库是一样的，只是最后变成了Thanks for using MySQL!

`注意：如果 mysql_secure_installation 找不到，则需要执行 source /etc/profile 后，再执行 mysql_secure_installation 来初始化MySQL数据库，或者重新连接 Linux 系统后再使用 mysql_secure_installation 来初始化MySQL数据库。`

```
 [root@localhost ~]# mysql_secure_installation
 NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
 SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!
 In order to log into MySQL to secure it, we'll need the current
 password for the root user. If you've just installed MySQL, and
 you haven't set the root password yet, the password will be blank,
 so you should just press enter here.
 Enter current password for root (enter for none): 此处只需按下回车键
 OK, successfully used password, moving on...
 Setting the root password ensures that nobody can log into the MySQL
 root user without the proper authorisation.
 Set root password? [Y/n] y （要为root管理员设置数据库的密码）
 New password: 输入要为root管理员设置的数据库密码
 Re‐enter new password: 再输入一次密码
 Password updated successfully!
 Reloading privilege tables..
 ... Success!
 By default, a MySQL installation has an anonymous user, allowing anyone
 to log into MySQL without having to have a user account created for
 them. This is intended only for testing, and to make the installation
 go a bit smoother. You should remove them before moving into a
 production environment.
 Remove anonymous users? [Y/n] y （删除匿名账户）
 ... Success!
 Normally, root should only be allowed to connect from 'localhost'. This
 ensures that someone cannot guess at the root password from the network.
 Disallow root login remotely? [Y/n] y （禁止root管理员从远程登录）
 ... Success!
 By default, MySQL comes with a database named 'test' that anyone can
 access. This is also intended only for testing, and should be removed
 before moving into a production environment.
 Remove test database and access to it? [Y/n] y （删除test数据库并取消对其的访问权限）
 ‐ Dropping test database...
 ... Success!
 ‐ Removing privileges on test database...
 ... Success!
 Reloading the privilege tables will ensure that all changes made so far
 will take effect immediately.
 Reload privilege tables now? [Y/n] y （刷新授权表，让初始化后的设定立即生效）
 ... Success!
 All done! If you've completed all of the above steps, your MySQL
 installation should now be secure.
 Thanks for using MySQL!
 Cleaning up...
```

登录MySQL数据库，做简单操作，验证数据库是否正常
```sql
 [root@localhost ~]# mysql ‐u root ‐p
 Enter password:
 Welcome to the MySQL monitor. Commands end with ; or \g.
 Your MySQL connection id is 12
 Server version: 5.6.19 Source distribution

 Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserv
ed.

 Oracle is a registered trademark of Oracle Corporation and/or its
 affiliates. Other names may be trademarks of their respective
 owners.

 Type 'help;' or '\h' for help. Type '\c' to clear the current input stat
ement.

 mysql> show databases;
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | Database |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | information_schema |
 | mysql |
 | performance_schema |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 3 rows in set (0.00 sec)

 mysql> create database test;
 Query OK, 1 row affected (0.00 sec)

 mysql> show databases;
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | Database |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | information_schema |
 | mysql |
 | performance_schema |
 | test |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 4 rows in set (0.00 sec)

 mysql> create database test2;
 Query OK, 1 row affected (0.00 sec)

 mysql> show databases;
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | Database |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | information_schema |
 | mysql |
 | performance_schema |
 | test |
 | test2 |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 5 rows in set (0.00 sec)

 mysql> drop database test2;
 Query OK, 0 rows affected (0.00 sec)

 mysql> show databases;
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | Database |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 | information_schema |
 | mysql |
 | performance_schema |
 | test |
 +‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐‐+
 4 rows in set (0.00 sec)

 mysql>
 ```
 
## 用脚本安装MySQL 5.6

 ```shell
 [root@localhost src]# vim mysql.sh 
#!/bin/bash
#Explain:This is a script to quickly deploy the MySQL service.
#Author:Jaking
#Email:Jaking@vip.163.com
#Date:2019-10-09
#注意:把所需源码包放到/usr/local/src目录下
#推荐安装包：mysql-5.6.19.tar.gz cmake-2.8.11.2.tar.gz

#优化环境
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config &> /dev/null
setenforce 0
systemctl stop firewalld
systemctl disable firewalld
iptables -F
systemctl stop NetworkManager &> /dev/null
systemctl disable NetworkManager &> /dev/null

#检查yum源是否配置好，如果没有配置好则配置yum源
yum clean all 1> /dev/null 2>&1 && yum makecache &> /dev/null
if [ $? -ne 0 ];then

cd /etc/yum.repos.d

cat >rhel7.repo<<EOF
[rhel7]
name=rhel7
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
EOF

  mkdir -p /media/cdrom
  mount /dev/cdrom /media/cdrom &>/dev/null
  sed -i -e '/iso9660/d' /etc/fstab
  echo "/dev/cdrom /media/cdrom iso9660 defaults 0 0" >> /etc/fstab
  yum clean all &>/dev/null && yum makecache &>/dev/null

fi

#安装依赖包及编译工具
yum install -y gcc* make ncurse* perl*

#安装CMake编译工具
cd /usr/local/src
tar xzvf cmake-2.8.11.2.tar.gz
cd cmake-2.8.11.2
./configure
make && make install


#部署MySQL数据库

#创建一个名为mysql的用户
useradd mysql -s /sbin/nologin

#创建一个用于保存MySQL数据库程序和数据库文件的目录，并把该目录的所有者和所属组身份修改为mysql。
mkdir -p /usr/local/mysql/var
chown -Rf mysql:mysql /usr/local/mysql

#解压、编译、安装MySQL数据库服务程序。在编译数据库时使用的是cmake命令，其中，-DCMAKE_INSTALL_PREFIX参数用于定义数据库服务程序的保存目录，-DMYSQL_DATADIR参数用于定义真实数据库文件的目录，-DSYSCONFDIR则是定义MySQL数据库配置文件的保存目录。
cd /usr/local/src
tar xzvf mysql-5.6.19.tar.gz
cd mysql-5.6.19
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/var -DSYSCONFDIR=/etc
make && make install

#为了让MySQL数据库程序正常运转起来，需要先删除/etc目录中的默认配置文件，然后在MySQL数据库程序的保存目录scripts内找到一个名为mysql_install_db的脚本程序，执行这个脚本程序并使用--user参数指定MySQL服务的对应账号名称（在前面步骤已经创建），使用--basedir参数指定MySQL服务程序的保存目录，使用--datadir参数指定MySQL真实数据库的文件保存目录，这样即可生成系统数据库文件，也会生成出新的MySQL服务配置文件。
rm -rf /etc/my.cnf
cd /usr/local/mysql
./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var

#把系统新生成的MySQL数据库配置文件链接到/etc目录中，然后把程序目录中的开机程序文件复制到/etc/rc.d/init.d目录中，以便通过service命令来管理MySQL数据库服务程序，把数据库脚本文件的权限修改成755以便于让用户有执行该脚本的权限。
ln -s my.cnf /etc/my.cnf 
cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
chmod 755 /etc/rc.d/init.d/mysqld

#编辑刚刚复制的MySQL数据库脚本文件，把第46、47行的basedir与datadir参数分别修改为MySQL数据库程序的保存目录和真实数据库的文件内容。
sed -i '46d' /etc/rc.d/init.d/mysqld && sed -i '45 abasedir=/usr/local/mysql' /etc/rc.d/init.d/mysqld 
sed -i '47d' /etc/rc.d/init.d/mysqld && sed -i '46 adatadir=/usr/local/mysql/var' /etc/rc.d/init.d/mysqld


#设置PATH变量
sed -i '74d' /etc/profile && sed -i '73 aexport PATH=$PATH:/usr/local/mysql/bin' /etc/profile

#创建MySQL默认的数据文档存储目录
mkdir /var/lib/mysql

#创建链接文件
ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock 
ln -s /usr/local/mysql/include/mysql /usr/include/mysql

#启动MySQL,把mysqld加入开机启动项中。
service mysqld start
chkconfig mysqld on
echo "MySQL已安装成功，请执行 source /etc/profile 后，再执行 mysql_secure_installation 来完成MySQL数据库的初始化！"
```
执行脚本安装MySQL 5.6
```
[root@localhost src]# chmod 755 mysql.sh
[root@localhost src]# ./mysql.sh 
Starting MySQL. SUCCESS! 
MySQL已安装成功，请执行 source /etc/profile 后，再执行 mysql_secure_installation 来完成MySQL数据库的初始化！
[root@localhost src]# mysql_secure_installation
-bash: mysql_secure_installation: command not found
[root@localhost src]# 
[root@localhost src]# source /etc/profile
Attempting to create directory /root/perl5
```

```
[root@localhost src]# mysql_secure_installation


NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MySQL to secure it, we'll need the current
password for the root user.  If you've just installed MySQL, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 直接回车
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!




All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!


Cleaning up...
```

```sql
[root@localhost src]# mysql -uroot -puplooking
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.6.19 Source distribution

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

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
+--------------------+
3 rows in set (0.00 sec)

mysql> exit
Bye
[root@localhost src]#
```
