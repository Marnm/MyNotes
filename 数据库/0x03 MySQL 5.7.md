---
title: 0x03 MySQL 5.7
tags: []
---

## 源码安装MySQL 5.7

### 1. 安装依赖 Boost C++ 库

从MySQL5.7版本开始，安装MySQL需要依赖 Boost 的C++扩展，而且只能是 1.59.0 版本。  
Boost 下载地址（任一地址都可下载）：  
[http://www.boost.org/users/history](http://www.boost.org/users/history) ；  
[https://www.boost.org/users/history/version_1_59_0.html](https://www.boost.org/users/history/version_1_59_0.html)  
[https://sourceforge.net/projects/boost/files/boost/1.59.0/](https://sourceforge.net/projects/boost/files/boost/1.59.0/)  
[https://downloads.mysql.com/archives/get/p/23/file/mysql-boost-5.7.19.tar.gz](https://downloads.mysql.com/archives/get/p/23/file/mysql-boost-5.7.19.tar.gz)  
选择1.59.0版本下载，在编译是填写相应参数，指定Boost源码位置即可。  
Boost库是一个可移植、提供源代码的C++库，作为标准库的后备，是C++标准化进程的开发引擎之一。  
Boost库由C++标准委员会库工作组成员发起，其中有些内容有望成为下一代C++标准库内容。  
在C++社区中影响甚大，是不折不扣的“准”标准库。Boost由于其对跨平台的强调，对标准C++的强调，与编写平台无关。  

**下载解压Boost C++库**  

    wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
    # 解压到/usr/local/boost
    tar -zxvf boost_1_59_0.tar.gz 
    mv boost_1_59_0 /usr/local/boost

### 2. 安装相关编译工具和依赖包 

    yum install -y gcc-c++ cmake ncurses-devel




### 3. 创建MySQL用户来运行MySQL数据库

    groupadd -g 27 mysql
    useradd mysql -u 27 -g mysql -M -s /sbin/nologin

### 4. 创建存放数据的目录

    mkdir -p /data/mysql
    chown -Rf mysql:mysql /data/mysql
    
### 5. 源码编译安装MySQL 5.7
MySQL官方下载地址：
[https://downloads.mysql.com/archives/community/](https://downloads.mysql.com/archives/community/)

**下载**

    wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.19.tar.gz
    tar -zxvf mysql-5.7.19.tar.gz -C /usr/local/src
    
**编译安装**

    cd /usr/local/src/mysql-5.7.19
    
    # 预编译
    cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql57 -DMYSQL_DATADIR=/data/mysql -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DMYSQL_TCP_PORT=3306 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DMYSQL_USER=mysql -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/boost -DWITH_INNODB_MEMCACHED=ON
    
    ## 编译安装
    # 默认编译速度非常慢，不推荐
    # 加速编译方法看文末操作
    make && make install
    
### 6. 创建配置文件

    mv /etc/my.cnf /etc/my.cnf.bak

```bash
cat > /etc/my.cnf <<EOF
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
EOF
```

### 7. 配置MySQL环境变量

```bash
echo "export PATH=\$PATH:/usr/local/mysql57/bin" >> /etc/profile
source /etc/profile
```

### 8. 初始化MySQL 5.7

    /usr/local/mysql57/bin/mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql
    
### 9. 运行MySQL

    /usr/local/mysql57/support-files/mysql.server start
    
    /usr/local/mysql57/support-files/mysql.server status

### 10. 查找MySQL root用户初始密码

    cd /data/mysql
    
    cat mysqld.log | grep root

### 11. 修改默认密码

**使用root默认密码登录MySQL**

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'New_password';
FLUSH PRIVILEGES;
```

### 12. 创建MySQL 5.7 systemd启动文件

### 13. 编译优化

**安装ccache加快编译速度**

    
    yum install -y epel-release
    yum install -y ccache
    
    # 使用ccache接管gcc/g++
    cp ccache /usr/local/bin/
    ln -s ccache /usr/local/bin/gcc
    ln -s ccache /usr/local/bin/g++
    ln -s ccache /usr/local/bin/cc
    ln -s ccache /usr/local/bin/c++

**make -j并行编译**
CPU是影响编译速度的一个重要因素
`make -j4` 让make最多允许4个编译命令同时执行

在多核CPU上，适当的进行并行编译还是可以明显提高编译速度的。但并行的任务不宜太多，**make -j 一般是以CPU的核心数目的两倍**为宜。

**注意！** 在使用并行编译时，系统内存和交换分区给大一些，内存过小时，编译会出错

ccache官方文档
[https://ccache.dev/manual/4.3.html](https://ccache.dev/manual/4.3.html)

------

------

## 使用rpm安装MySQL 5.7

### 1. 优化Linux系统

    sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config &> /dev/null
    setenforce 0
    systemctl stop firewalld
    systemctl disable firewalld
    iptables -F
    systemctl stop NetworkManager &> /dev/null
    systemctl disable NetworkManager &> /dev/null

### 2. 获取MySQL 5.7的rpm包

获取方式一：
MySQL官网下载
[https://downloads.mysql.com/archives/community/](https://downloads.mysql.com/archives/community/)
获取方式二：copy镜像安装包

**分别下载这5个rpm包：**
MySQL服务端程序RPM Package, MySQL Server：
[mysql-community-server-5.7.24-1.el7.x86_64.rpm](https://downloads.mysql.com/archives/get/p/23/file/mysql-community-server-5.7.24-1.el7.x86_64.rpm)
MySQL客户端工具RPM Package, Client Utilities：
[mysql-community-client-5.7.24-1.el7.x86_64.rpm](https://downloads.mysql.com/archives/get/p/23/file/mysql-community-client-5.7.24-1.el7.x86_64.rpm)
MySQL开发库RPM Package, Development Libraries：
[mysql-community-devel-5.7.24-1.el7.x86_64.rpm](https://downloads.mysql.com/archives/get/p/23/file/mysql-community-devel-5.7.24-1.el7.x86_64.rpm)
MySQL配置程序RPM Package, MySQL Configuration：
[mysql-community-common-5.7.24-1.el7.x86_64.rpm](https://downloads.mysql.com/archives/get/p/23/file/mysql-community-common-5.7.24-1.el7.x86_64.rpm)
MySQL共享库RPM Package, Shared Libraries：
[mysql-community-libs-5.7.24-1.el7.x86_64.rpm](https://downloads.mysql.com/archives/get/p/23/file/mysql-community-libs-5.7.24-1.el7.x86_64.rpm)

### 3. 安装MySQL

**卸载系统中默认安装的mariadb-libs数据库包**

    rpm -e --nodeps mariadb-libs

**安装rpm包**
安装顺序：

    rpm -ivh mysql-community-common-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-libs-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-client-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-devel-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-server-5.7.24-1.el7.x86_64.rpm

### 4. 初始化数据库

    vim /etc/my.cnf
    
    # 末行添加skip-name-resolv，跳过主机名解析，提高连接速度
    skip-name-resolv

**启动数据库**

    systemctl start mysqld
    systemctl status mysqld
    systemctl enable mysqld

### 5. 查看MySQL初始密码

    cat /var/log/mysqld.log | grep "password"
    
### 6. 修改MySQL初始密码

**使用MySQL root默认密码登录数据库**

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_PASSWORD';
FLUSH PRIVILEGES;
# 或者
update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost';
# 或者
set password for 'root'@'localhost'=password('123456');
```
#### 扩展：修改MySQL密码策略
在使用rpm安装MySQL 5.7的时候，默认密码强度策略是MEDIUM，使用初始密码登录后，必须修改默认密码，不然很多功能都不能用，修改密码后才能正常操作。
**方法一：关闭密码验证插件**
修改MySQL配置文件，添加如下参数

    validate_password=off
    
**方法二：修改密码强度策略**（推荐）
```sql
-- 查看密码策略
SHOW VARIABLES like "%password%";
SET global validate_password_length=6;
SET global validate_password_policy=0;
# 0 或 LOW
# 1 或 MEDIUM
# 2 或 STRONG
FLUSH PRIVILEGES;
```


### MySQL 5.7多实例配置
**安装MySQL 5.7 rpm包**

    rpm -e --nodeps mariadb-libs
    
    rpm -ivh mysql-community-common-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-libs-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-client-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-devel-5.7.24-1.el7.x86_64.rpm
    rpm -ivh mysql-community-server-5.7.24-1.el7.x86_64.rpm
    
**创建数据存放目录**

    mkdir -p /data/mysql/{3306, 3307}
    mkdir -p /data/mysql/3306/{data, binlog, logs}
    mkdir -p /data/mysql/3307/{data, binlog, logs}
    
**创建用户，设置属主属组**

    chown -Rf mysql:mysql /data/mysql
    mv /etc/my.cnf /etc/my.cnf.bak
    
**新增配置文件my3306.cnf**
```json
cat>>/etc/my3306.cnf<<EOF
[mysqld]
user = mysql
port = 3306
server_id = 3306
datadir = /data/mysql/3306/data
socket = /data/mysql/3306/mysql3306.sock
symbolic-links = 0
log-error = /data/mysql/3306/logs/mysqld3306.log
pid-file = /data/mysql/3306/mysqld3306.pid
EOF
```
**新增配置文件my3307.cnf**
```json

cp /etc/my3306.cnf /etc/my3307.cnf
sed -i 's/3306/3307/g' /etc/my3307.cnf

[root@jaking mysql5.7]# cat /etc/my3307.cnf
[mysqld]
user = mysql
port = 3307
server_id = 3307
datadir = /data/mysql/3307/data
socket = /data/mysql/3307/mysql3307.sock
symbolic-links = 0
log-error = /data/mysql/3307/logs/mysqld3307.log
pid-file = /data/mysql/3307/mysqld3307.pid
```

**备份mysql启动服务文件**

    mv /usr/lib/systemd/system/mysqld.service /usr/lib/systemd/system/mysqld.service.bak
    
**新增mysqld3306.service启动文件**
```json

cat>>/usr/lib/systemd/system/mysqld3306.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
Type=forking
PIDFile=/data/mysql/3306/mysqld3306.pid
TimeoutSec=0
PermissionsStartOnly=true
ExecStart=/usr/sbin/mysqld --defaults-file=/etc/my3306.cnf --daemonize --pid-file=/data/mysql/3306/mysqld3306.pid $MYSQLD_OPTS
EnvironmentFile=-/etc/sysconfig/mysql
LimitNOFILE = 5000
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false
EOF
```
**新增mysqld3307.service启动文件**
```json
cp /usr/lib/systemd/system/mysqld3306.service /usr/lib/systemd/system/mysqld3307.service
sed -i 's/3306/3307/g' /usr/lib/systemd/system/mysqld3307.service

[root@jaking mysql5.7]# cat /usr/lib/systemd/system/mysqld3307.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
Type=forking
PIDFile=/data/mysql/3307/mysqld3307.pid
TimeoutSec=0
PermissionsStartOnly=true
ExecStart=/usr/sbin/mysqld --defaults-file=/etc/my3307.cnf --daemonize --pid-file=/data/mysql/3307/mysqld3307.pid 
EnvironmentFile=-/etc/sysconfig/mysql
LimitNOFILE = 5000
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false
```
**初始化多实例3306、3307**

    mysqld --defaults-file=/etc/my3306.cnf --initialize --user=mysql --datadir=/data/mysql/3306/data
    mysqld --defaults-file=/etc/mysql3307.cnf --initialize --user=mysql --datadir=/data/mysql/3307/data
    
**启动多实例3306、3307**

    systemctl daemon-reload
    systemctl start mysqld3306
    systemctl start mysqld3307
    
    netstat -pantul | grep mysql
    
    ps aux | grep mysql
    
**查看MySQL root初始密码**

    cat /data/mysql/3306/logs/mysqld3306.log | grep root
    cat /data/mysql/3307/logs/mysqld3307.log | grep root
    
**登录多实例并修改root默认密码**

    # mysql3306实例
    mysql -S /data/mysql/3306/mysql3306.sock -uroot -p 

```sql
# 修改默认密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newPassword';
FLUSH PRIVILEGES;
```
    
    # mysql3307实例
    mysql -S /data/mysql/3307/mysql3307.sock -uroot -p
    
```sql
# 修改默认密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'New_password';
FLUSH PRIVILEGES;
```

#### 增加实例
**创建目录，设置属主属组**

    mkdir -p /data/mysql/3308/{data, binlog, logs}
    chown -Rf mysql:mysql /data/mysql/3308
    
**新增配置文件my3308.cnf**

    cp /etc/my3306.cnf /etc/my3308.cnf
    sed -i 's/3306/3308/g' /etc/my3308.cnf
    
```sh
cat /etc/my3308.cnf
[mysqld]
user = mysql
port = 3308
server_id = 3308
datadir = /data/mysql/3308/data
socket = /data/mysql/3308/mysql3308.sock
symbolic-links = 0
log-error = /data/mysql/3308/logs/mysqld3308.log
pid-file = /data/mysql/3308/mysqld3308.pid
```
**添加mysqld3308.service启动文件**

    cp /usr/lib/systemd/system/mysqld3306.service /usr/lib/systemd/system/mysqld3308.service
    sed -i 's/3306/3308/g' /usr/lib/systemd/system/mysqld3308.service

```sh
cat /usr/lib/systemd/system/mysqld3308.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
Type=forking
PIDFile=/data/mysql/3308/mysqld3308.pid
TimeoutSec=0
PermissionsStartOnly=true
ExecStart=/usr/sbin/mysqld --defaults-file=/etc/my3308.cnf --daemonize --pid-file=/data/mysql/3308/mysqld3308.pid 
EnvironmentFile=-/etc/sysconfig/mysql
LimitNOFILE = 5000
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false
```
**初始化新增实例3308**

    mysqld --defaults-file=/etc/my3308.cnf --initialize --user=mysql --datadir=/data/mysql/3308/data
    
**启动新增实例3308**

    systemctl start mysqld3308
    
    netstat -nutpl | grep mysql
    
**获取mysql3308 root初始化密码**

    cat /data/mysql/3308/logs/mysqld3308.log | grep root
    
**登录mysql3308修改默认密码**

    mysql -S /data/mysql/3308/mysql3308.sock -uroot -p
    
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY "NewPassword";
FLUSH PRIVILEGES;
```

------

------


## 修改MySQL端口

    vim /etc/my.cnf
    
    [mysqld]
    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock
    symbolic-links=0
    port=3307
    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid
    
    netstat -pantul | grep 3307
    

------

------

## MySQL 5.7 忘记root密码

**修改配置文件，绕过密码验证**

    echo skip-grant-tables >> /etc/my.cnf
    /etc/init.d/mysqld restart
    
**修改数据库root密码**

    mysql

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_Password';
FLUSH PRIVILEGES;
```

**修改配置文件，删除skil-grant-tables，恢复密码验证**

**重启MySQL**
