---
title: LNMP高可用架构配置
tags: []
---

**9台机器IP地址分配（静态）：**：  
`192.168.110.11` ----> Nginx反向代理  
`192.168.110.12` ----> Nginx反向代理  
`192.168.110.13` ----> Nginx Server  
`192.168.110.14` ----> Nginx Server  
`192.168.110.15` ----> PHP 5.6  
`192.168.110.16` ----> PHP 5.6  
`192.168.110.17` ----> 存储服务  
`192.168.110.18` ----> MySQL  
`192.168.110.19` ----> NMySQL 


## 系统基本配置

在每台机器上完成基本配置和优化
```shell
#!/bin/bash

# CentOS 7 Local YUM and EPEL Repository
# local yum and epel repository
function centos7(){
mv -f /etc/yum.repos.d/*.repo /tmp/
cat > /etc/yum.repos.d/local.repo <<EOF
[local-yum]
name=Local Base ISO Repository
baseurl=http://192.168.110.3/iso/centos7
enabled=1
gpgcheck=0
priority=1
EOF

# local epel
cat > /etc/yum.repos.d/epel-local.repo << EOF
[epel-local]
name=Extra Packages for CentOS 7
baseurl=http://192.168.110.3/epel/7/x86_64
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
EOF
yum clean all && yum makecache
yum install -y vim wget
}

# Red Hat Enterprise Linux 7 Local YUM and EPEL Repository
function rhel7(){
mv -f /etc/yum.repos.d/*.repo /tmp
cat > /etc/yum.repos.d/local.repo <<EOF
[local-yum]
name=Local Base ISO Repository
baseurl=http://192.168.110.3/iso/rhel7
enabled=1
gpgcheck=0
priority=1
EOF

# local epel
cat > /etc/yum.repos.d/epel-local.repo << EOF
[epel-local]
name=Extra Packages for Enterprise Linux 7
baseurl=http://192.168.110.3/epel/7Server/x86_64
enabled=1
gpgcheck=0
EOF
yum clean all && yum makecache
yum install -y vim wget
}

# Optimiz Linux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config &> /dev/null
setenforce 0
systemctl stop firewalld
systemctl disable firewalld
iptables -F
systemctl stop NetworkManager &> /dev/null
systemctl disable NetworkManager &> /dev/null

# Change IP Address 
name=`ip a |  grep ens | tr -d : | awk '{print $2}' | head -1`
ip=`ip a | grep "\<inet\>" | grep -v "127.0.0.1" | tr '/' ' ' | awk '{print $2}' | head -1`

cat > /etc/sysconfig/network-scripts/ifcfg-$name << EOF
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
NAME=$name
DEVICE=$name
ONBOOT=yes
IPADDR=$ip
NETMASK=255.255.255.0
GATEWAY=192.168.110.1
DNS1=192.168.110.1
EOF

# Manual replace ip address
read -p "Change New IP Address: " new_ip

sed -i "s/$ip/$new_ip/g" /etc/sysconfig/network-scripts/ifcfg-$name

systemctl restart network

# Change hostname
host_bit=`ip a | grep "\<inet\>" | grep -v "127.0.0.1" | tr '/' ' ' | awk '{print $2}' | head -1 | awk ' BEGIN{ FS="."} { print $4 }'`
hostnamectl set-hostname server$host_bit

# Configuration Local Repository
#OS=`cat /etc/redhat-release  | awk '{ print $1 }'`
if [ `cat /etc/redhat-release  | awk '{ print $1 }'` == "CentOS" ]
    then centos7
else
    rhel7
fi

# shutdown
init 0
```

## 配置Nginx Server
```shell
#!/bin/bash

# Install compiler requirment env
yum install -y gcc gcc-c++ pcre pcre-devel openssl openssl-libs openssl-devel zlib zlib-devel

# source build for Nginx 
cd /usr/local/src/
curl -O --progress http://192.168.110.3/src/nginx-1.12.0.tar.gz
tar -zxf nginx-1.12.0.tar.gz
cd nginx-1.12.0/

./configure --with-http_v2_module --with-http_ssl_module --with-http_sub_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre

make
make install

ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx

cd /usr/local/nginx/conf/
cp nginx.conf nginx.conf.bak

# Change Nginx Config
cat > /usr/local/nginx/conf/nginx.conf <<EOF
user nobody;
worker_processes auto;
worker_cpu_affinity 0001;
worker_rlimit_nofile 50000;

error_log /usr/local/nginx/logs/error.log warn;
pid /usr/local/nginx/logs/nginx.pid;

events {
    worker_connections 50000;
    use epoll;
    multi_accept on;
}
 
http {
    include mime.types;
    default_type application/octet-stream;

	log_format  main  '\$remote_addr - \$remote_user [\$time_local] "\$request" '
                      '\$status \$body_bytes_sent "\$http_referer" '
                      '"\$http_user_agent" "\$http_x_forwarded_for"';
    
    access_log /usr/local/nginx/logs/access.log main;
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    
    gzip on;
    gzip_min_length 2K;
    gzip_buffers 4 16k;
    gzip_comp_level 5;
    gzip_types text/plain application/x-javascript text/css application/xml;
    
    include /usr/local/nginx/conf/user/*.conf;
}

EOF

nginx

mkdir -p /usr/local/nginx/conf/user
cat > /usr/local/nginx/conf/user/uplooking.conf <<EOF
server {
    listen 80;
    server_name www.uplookinglinux.com;
    
    charset utf-8;
    access_log /usr/local/nginx/logs/uplookinglinux.access.log main;
    
    location / {
        root /var/www/web;
        index index.html index.htm;
    }
    
    error_page 500 502 503 504 /50x.html;
    location = /50x.thml {
        root /usr/local/nginx/html;
    }
}
EOF

nginx -s reload

mkdir -p /var/www/web
echo "Welcome to Nginx" > /var/www/web/index.html
chown -Rf nobody:nobody /var/www/web

echo -e "* soft nproc 65535\n* hard nproc 65535\n* soft nofile 65535\n* hard nofile 65535" >> /etc/security/limits.conf


# 测试Nginx
CURL='/usr/bin/curl'
RVMHTTP="127.0.0.1"
CURLARGS="-s -I"

#raw="$($CURL $CURLARGS $RVMHTTP)"

# curl -s -I 127.0.0.1 > /tmp/stat_code
$CURL $CURLARGS $RVMHTTP > /tmp/stat_code

# 获取http状态码
http_code=`/usr/bin/sed -n '1,1p' /tmp/stat_code | /usr/bin/awk '{ print $2 }'`

if [[ $http_code -ge 200 && $http_code -le 299 ]]
then
    echo -e "\e[1;32mNginx OK\e[0m"
else
    echo -e "\e[1;31mServer Error !\e[0m"
fi

nginx -v

# 挂载NFS目录到静态Nginx Server上
yum install -y nfs-utils && mount -t nfs 192.168.110.17:/webData /var/www/web/
echo -e "192.168.110.17:/webData\t/var/www/web\tnfs\tdefaults\t0 0" >> /etc/fstab

mount -a

echo "请退出当期会话并重新登录！"
```

## 配置Nginx代理和负载均衡
```shell
#!/bin/bash

# 安装依赖环境
yum install -y gcc gcc-c++ pcre pcre-devel openssl openssl-libs openssl-devel zlib zlib-devel

# 获取Nginx
cd /usr/local/src/
curl -O --progress http://192.168.110.3/src/nginx-1.12.0.tar.gz
tar -zxf nginx-1.12.0.tar.gz
cd nginx-1.12.0/

# 编译Nginx
./configure --with-http_v2_module --with-http_ssl_module --with-http_sub_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre

make
make install

ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx

nginx

cd /usr/local/nginx/conf/
cp nginx.conf nginx.conf.bak

mkdir -p /usr/local/nginx/conf/user
cat > /usr/local/nginx/conf/nginx.conf << EOF
user nobody;
worker_processes auto;

worker_cpu_affinity 0001;
worker_rlimit_nofile 50000;

error_log /usr/local/nginx/logs/error.log warn;
pid /usr/local/nginx/logs/nginx.pid;

events {
    worker_connections 50000;
    use epoll;
    multi_accept on;
}


http {
    include mime.types;
    default_type application/octet-stream;
    

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /usr/local/nginx/logs/access.log main;
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    
    gzip on;
    gzip_min_length 2K;
    gzip_buffers 4 16k;
    gzip_comp_level 5;
    gzip_types text/plain application/x-javascript text/css application/xml;
    
    upstream webServer {
        server 192.168.110.13:80 weight=1 max_fails=3 fail_timeout=10s;
        server 192.168.110.14:80 weight=1 max_fails=3 fail_timeout=10s;
    }
    
    include /user/local/nginx/conf/user/*.conf;
}
EOF

cat > /usr/local/nginx/conf/user/proxy.conf << EOF
server {
    listen 80;
    server_name www.uplookinglinux.com;

    charset utf-8;
    access_log /usr/local/nginx/logs/proxy.access.log main;

    location / {
        root /var/www/web;
        index index.html index.htm;
        #指定代理到对应的后端服务器
        proxy_pass http://webServer;

        #传递客户的ip信息给这台服务器 在后台服务通过参数$http_x_real_ip取得该传递的IP信息
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_redirect off;
        proxy_connect_timeout 30;
        proxy_send_timeout 15;
        proxy_read_timeout 15;
    }
}
EOF

nginx -s reload
```

## 配置存储服务

```shell
# 安装RAID管理工具
yum install -y mdadm

# 创建RAID10
mdadm -Cv /dev/md10 -a yes -l 10 -n 4 /dev/sd[b-e]
mdadm -Q /dev/md10
mdadm -q /dev/md10

# 制作LVM分区
pvcreate /dev/md10
vgcreate webData /dev/md10
lvcreate -n lv_web -L 20G webData
mkfs.xfs /dev/webData/lv_web

# 挂载磁盘
mkdir /webData
echo -e "/dev/webData/lv_web\t/webData\t xfs\tdefaults\t0 0" >> /etc/fstab
mount -a

# 配置NFS Server
yum install -y rpcbind nfs-utils
systemctl start rpcbind &> /dev/null
systemctl enable rpcbind &> /dev/null
systemctl status rpcbind

echo -e "/webData *(rw,sync,all_squash,anonuid=99,anongid=99)" > /etc/exports

systemctl start nfs
systemctl status nfs

```

## 配置PHP FastCGI
```shell
#!/bin/bash

# filename : php-config.sh
# 配置PHP
cd /usr/local/src/
curl -O --progress http://192.168.110.3/src/php5.6_auto_install.tar.gz
tar zxf php5.6_auto_install.tar.gz
cd php5.6/
chmod a+x install.sh
. ./install.sh

# 创建站点目录
mkdir -p /var/www/web
echo -e "<?php\n\tphpinfo();\n?>" > /var/www/web/index.php
```

### 在Nginx代理服务器上配置FastCGI
```shell
#!/bin/bash

# filename : ngxin-fastcgi.sh

cat >> /usr/local/nginx/conf/nginx.conf <<EOF
upstream phpServer {
	# 指定负载算法：特定的客户端由固定的服务器服务，解决session共享问题
	ip_hash;
	server 192.168.110.15:9000 max_fails=3 fail_timeout=10s;
	server 192.168.110.16:9000 max_fails=3 fail_timeout=10s;
}
EOF

cat >> /usr/local/nginx/conf/user/proxy.conf << EOF
fastcgi_connect_timeout 30;
fastcgi_send_timeout 30;
fastcgi_read_timeout 30;
fastcgi_buffer_size 64k;
fastcgi_busy_buffers_size 128k;
fastcgi_temp_file_write_size 128k;
location ~ \.(php|php5)$ {
	root /var/www/web;
	#指定代理到phpServer集群
	fastcgi_pass phpServer;
	fastcgi_index index.php;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	include fastcgi_params;
}
EOF

nginx -t
nginx -s reload

```

## Nginx代理服务器高可用——Keepalived

```shell
#!/bin/bash

echo -e "192.168.110.11\tserver11\n192.168.110.12\tserver12" >> /etc/hosts

# 安装Keepalived
yum install -y keepalived

mv -f /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.confback
cat > /etc/keepaliaved/keepalived.conf << EOF
! Configuration File for keepalived
 
global_defs {
    notification_email {
        fnaichu@qq.com
    }
    notification_email_from keepalvied@proxy-1
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id proxy-1
    vrrp_mcast_group4 224.4.4.4
}
 
vrrp_script chk_proxy {
    script "/etc/keepalived/proxy_check.sh"
     
    interval 2
     
    weight -2
}
 
vrrp_instance VI_1 {
    state BACKUP
     
    nopreempt
     
    virtual_router_id 11
     
    priority 100
     
    advert_int 1
     
    interface ens33
     
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.110.10/24 dev ens33 label ens33:1
    }
    track_script {
        chk_proxy
    }
    smtp_alert
}
EOF

# 代理检测脚本
cat > /etc/keepaliaved/proxy_check.sh << EOF
#!/bin/bash

netstat -nlpt | grep -w "80" &>> /dev/null

if [ $? -ne 0 ]
then
	systemctl stop keepalived
fi

exit 0
EOF

chmod +x /etc/keepalived/proxy_check.sh

systemctl start keepalived
systemctl enable keepalived
```

## 配置MySQL
```shell
#!/bin/bash

yum remove -y mariadb-libs

# 安装MySQL
rpm -ivh http://192.168.110.3/other/mysql-community-common-5.7.24-1.el7.x86_64.rpm
rpm -ivh http://192.168.110.3/other/mysql-community-libs-5.7.24-1.el7.x86_64.rpm
rpm -ivh http://192.168.110.3/other/mysql-community-client-5.7.24-1.el7.x86_64.rpm
rpm -ivh http://192.168.110.3/other/mysql-community-server-5.7.24-1.el7.x86_64.rpm
rpm -ivh http://192.168.110.3/other/mysql-community-libs-compat-5.7.32-1.el7.x86_64.rpm

systemctl start mysqld
#systemctl enable mysqld
systemctl status mysqld

echo "skip-name-resolve" >> /etc/my.cnf
grep "temporary password" /var/log/mysqld.log | awk '{ print $NF }'

echo "上面是MySQL初始密码，请手动修改密码"
# mysql_secure_installation
# show global variables like "%password%";
# set global validate_password_length=6;
# set global validate_password_policy=LOW;
# ALTER USER 'root'@'localhost' IDENTIFIED BY 'ccc.456';
# FLUSH PRIVILEGES;
```

### 配置主从
```shell
#!/bin/bash

#mysql_replica.sh
mysql -uroot -p"ccc.456"
GRANT REPLICATION SLAVE ON *.* TO 'root'@'192.168.110.%' IDENTIFIED BY 'ccc.456';
FLUSH PRIVILEGES;
quit

cat >> /etc/my.cnf <<EOF
server-id=1
log_bin=mysql-bin
query_cache_type=OFF
binlog-ignore-db=performance_schema
binlog-ignore-db=mysql
binlog-ignore-db=sys
EOF

systemctl restart mysqld

```

### MySQL的Keepalived高可用
```shell
echo -e "192.168.110.18\tserver18\n192.168.110.19\tserver19\n" >> /etc/hosts

yum install -y keepalived

mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.confbak
# KeepAlived
cat > /etc/keepalived/keepalived.conf << EOF
! Configuration File for keepalived
 
global_defs {
    notification_email {
        Jaking@vip.163.com
    }
    
    notification_email_from keepalived@server18
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id server18
    vrrp_mcast_group5 225.5.5.5
}
 
    vrrp_script chk_proxy {
        script "/etc/keepalived/proxy_check.sh"
        interval 2
        weight -2
    }
     
    vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    virtual_router_id 11
    priority 100
    advert_int 1
    interface ens33
    
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    
    virtual_ipaddress {
        192.168.110.20/24 dev ens33 label ens33:1
    }
    
    track_script {
        chk_proxy
    }
    smtp_alert
}
EOF

cat > /etc/keepalived/proxy_check.sh <<EOF
echo > /etc/keepalived/proxy_check.sh << EOF
#!/bin/bash
netstat -nlpt | grep -w "3306" &>> /dev/null
 
if [ $? -ne 0 ]
then
    systemctl stop keepalived
fi

exit 0
EOF

chmod a+x /etc/keepalived/proxy_check.sh
systemctl restart keepalived
```

## 项目上线
```shell
yum install -y unzip
unzip Cloudreve_v1.0.3云盘.zip

ssh root@192.168.110.18 "mysql -uroot -p"ccc.456" -e 'create database cloud;'"
sql_user="mysql -uroot -p"ccc.456" -e 'GRANT ALL ON cloud.* TO "abc"@"192.168.110.%" IDENTIFIED BY "Abc.abc.1";'"
ssh root@192.168.110.18 "$sql_user"

chown -R nobody:nobody /webData/
chmod u+w /webData/runtime/
chmod u+w /webData/public/

cp /webData/application/database_sample.php /webData/application/database.php


```

### 修改Nginx代理配置
```nginx
cat /usr/local/nginx/conf/user/proxy.conf 
server {
    listen       80;
    server_name  www.uplookinglinux.com;
 
    charset utf-8;
    access_log  /var/log/nginx/proxy.access.log  main;
 
    #################################################
    fastcgi_connect_timeout 30;
    fastcgi_send_timeout 30;
    fastcgi_read_timeout 30;
    fastcgi_buffer_size  64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    #uri 匹配 .php 或者 .php5 结尾
    location ~ \.(php|php5)$ {
        root   /var/www/web;
        #指定代理到 phpServer集群
        fastcgi_pass  phpServer;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include   fastcgi_params;
    }
    #匹配根目录，因为根目录下的主页为index.php动态页面
    location = / {
        root   /var/www/web;
        #指定代理到 phpServer集群
        fastcgi_pass  phpServer;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include   fastcgi_params;
 
    }
    #################################################
 
    location / {
        #设置伪静态转发规则
        if (!-e $request_filename) {
            rewrite  ^(.*)$  /index.php?s=/$1  last;
            break;
        }
 
        root   /var/www/web;
        index  index.html index.htm;
        proxy_pass http://webServer;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
 
        proxy_redirect off;
        proxy_connect_timeout 30;
        proxy_send_timeout 15;
        proxy_read_timeout 15;
    }
}
yum install -y nfs-utils
mkdir -p /var/www/web
chown nobody:nobody /var/www/web/
mount -tnfs 192.168.110.17:/webData /var/www/web/
echo -e "192.168.10.17:/webData\t/var/www/web\tnfs\tdefaults\t0 0" >> /etc/fstab
mount -a
```
