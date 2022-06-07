---
title: LNMP
tags: []
---

```shell
#!/bin/bash
#Explain: This is a script to quickly deploy the LNMP service
#Author: Jaking
#Email: Jaking@vip.163.com
#Date: 2019-06-06
#注意: 运行这个脚本前，请配置好yum源，把所需源码包放到/usr/local/src目录下。
#推荐安装包：
#cmake-2.8.11.2.tar.gz   libmcrypt-2.5.8.tar.gz  yasm-1.2.0.tar.gz    libpng-1.6.12.tar.gz    
#freetype-2.5.3.tar.gz   libvpx-v1.3.0.tar.bz2   php-5.5.14.tar.gz    jpegsrc.v9a.tar.gz      
#mysql-5.6.19.tar.gz     t1lib-5.1.2.tar.gz      libgd-2.1.0.tar.gz   nginx-1.6.0.tar.gz      tiff-4.0.3.tar.gz

#部署Linux

#安装相关包
yum install -y apr* autoconf automake bison bzip2 bzip2* compat* cpp curl curl-devel fontconfig fontconfig-devel freetype freetype* freetype-devel gcc gcc-c++ gd gettext gettext-devel glibc kernel kernel-headers keyutils keyutils-libs-devel krb5-devel libcom_err-devel libpng libpng-devel libjpeg* libsepol-devel libselinux-devel libstdc++-devel libtool* libgomp libxml2 libxml2-devel libXpm* libtiff libtiff* make mpfr ncurses* ntp openssl openssl-devel patch pcre-devel perl php-common php-gd policycoreutils telnet t1lib t1lib* nasm nasm* unzip wget zlib-devel

#安装CMake编译工具
cd /usr/local/src
tar xzvf cmake-2.8.11.2.tar.gz
cd cmake-2.8.11.2
./configure
make && make install

#部署Nginx

#停掉Apache服务
systemctl  stop httpd

#安装编译工具和库文件
yum install -y  gcc*  zlib  zlib-devel   pcre pcre-devel  openssl  openssl-devel


#创建一个用于执行Nginx服务程序的账户
useradd nginx -s /sbin/nologin

#安装nginx源码包
cd /usr/local/src
tar xzvf nginx-1.6.0.tar.gz
cd nginx-1.6.0
./configure  --user=nginx  --group=nginx  --prefix=/usr/local/nginx  --with-http_stub_status_module   --with-http_sub_module   --with-http_ssl_module  --with-pcre
make && make install

#设置selinux
setenforce 0

#配置防火墙
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables-save > /etc/sysconfig/iptables
 
#验证nginx是否配置有误
/usr/local/nginx/sbin/nginx -t

#启动nginx
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

#查看nginx进程
ps -ef|grep nginx


#部署MySQL

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

#启动MySQL,把mysqld加入开机启动项中。
service mysqld start
chkconfig mysqld on

#设置PATH变量
sed -i '74d' /etc/profile && sed -i '73 aexport PATH=$PATH:/usr/local/mysql/bin' /etc/profile
source /etc/profile

#创建MySQL默认的数据文档存储目录
mkdir /var/lib/mysql

#链接文件
ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock
ln -s /usr/local/mysql/include/mysql /usr/include/mysql

#部署PHP

#安装相关源码包
cd /usr/local/src
tar zxvf yasm-1.2.0.tar.gz
cd yasm-1.2.0
./configure
make && make install

cd /usr/local/src
tar zxvf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure
make && make install

cd /usr/local/src
tar xjvf libvpx-v1.3.0.tar.bz2
cd libvpx-v1.3.0
./configure --prefix=/usr/local/libvpx --enable-shared --enable-vp9
make && make install

cd /usr/local/src
tar zxvf tiff-4.0.3.tar.gz
cd tiff-4.0.3
./configure --prefix=/usr/local/tiff --enable-shared
make && make install

cd /usr/local/src
tar zxvf libpng-1.6.12.tar.gz
cd libpng-1.6.12
./configure --prefix=/usr/local/libpng --enable-shared
make && make install

cd /usr/local/src
tar zxvf freetype-2.5.3.tar.gz
cd freetype-2.5.3
./configure --prefix=/usr/local/freetype --enable-shared
make && make install

cd /usr/local/src
tar zxvf jpegsrc.v9a.tar.gz
cd jpeg-9a
./configure --prefix=/usr/local/jpeg --enable-shared
make && make install

cd /usr/local/src
tar zxvf libgd-2.1.0.tar.gz
cd libgd-2.1.0
./configure --prefix=/usr/local/libgd --enable-shared --with-jpeg=/usr/local/jpeg --with-png=/usr/local/libpng --with-freetype=/usr/local/freetype --with-fontconfig=/usr/local/freetype --with-xpm=/usr/ --with-tiff=/usr/local/tiff --with-vpx=/usr/local/libvpx
make && make install

cd /usr/local/src
tar zxvf t1lib-5.1.2.tar.gz
cd t1lib-5.1.2
./configure --prefix=/usr/local/t1lib --enable-shared
make without_doc && make install
ln -s /usr/lib64/libltdl.so /usr/lib/libltdl.so
cp -frp /usr/lib64/libXpm.so* /usr/lib/

#此时终于把编译php服务源码包的相关软件包都已经安装部署妥当了。在开始编译php源码包之前，先定义一个名为LD_LIBRARY_PATH的全局环境变量，该环境变量的作用是帮助系统找到指定的动态链接库文件，这些文件是编译php服务源码包的必须元素之一。编译php服务源码包时，除了定义要安装到的目录以外，还需要依次定义配置php服务程序配置文件的保存目录、MySQL数据库服务程序所在目录、MySQL数据库服务程序配置文件所在目录，以及libpng、jpeg、freetype、libvpx、zlib、t1lib等服务程序的安装目录路径，并通过参数启动php服务程序的诸多默认功能。
cd /usr/local/src
tar -zvxf php-5.5.14.tar.gz
cd php-5.5.14
export LD_LIBRARY_PATH=/usr/local/libgd/lib
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-mysql=/usr/local/mysql --with-mysqli=/usr/local/mysql/bin/mysql_config --with-mysql-sock=/tmp/mysql.sock --with-pdo-mysql=/usr/local/mysql --with-gd --with-png-dir=/usr/local/libpng --with-jpeg-dir=/usr/local/jpeg --with-freetype-dir=/usr/local/freetype --with-xpm-dir=/usr/ --with-vpx-dir=/usr/local/libvpx/ --with-zlib-dir=/usr/local/zlib --with-t1lib=/usr/local/t1lib --with-iconv --enable-libxml --enable-xml --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-opcache --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt --with-curl --enable-ctype
make && make install

#在php源码包程序安装完成后，需要删除当前默认的配置文件，然后将php服务程序目录中相应的配置文件复制过来。
rm -rf /etc/php.ini
ln -s /usr/local/php/etc/php.ini /etc/php.ini
cp php.ini-production /usr/local/php/etc/php.ini
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
ln -s /usr/local/php/etc/php-fpm.conf /etc/php-fpm.conf

#php-fpm.conf是php服务程序重要的配置文件之一，我们需要启用该配置文件中第25行的pid文件保存目录，然后分别将第148和149行的user与group参数分别修改为nginx账户和用户组名称。
sed -i '25d' /usr/local/php/etc/php-fpm.conf && sed -i '24 apid = run/php-fpm.pid' /usr/local/php/etc/php-fpm.conf 
sed -i '148d' /usr/local/php/etc/php-fpm.conf && sed -i '147 auser = nginx' /usr/local/php/etc/php-fpm.conf
sed -i '149d' /usr/local/php/etc/php-fpm.conf && sed -i '148 agroup = nginx' /usr/local/php/etc/php-fpm.conf

#配置妥当后便可把用于管理php服务的脚本文件复制到/etc/rc.d/init.d中了。为了能够执行脚本，请记得为脚本赋予755权限，最后把php-fpm服务程序加入到开机启动项中。
cp /usr/local/src/php-5.5.14/sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
chmod 755 /etc/rc.d/init.d/php-fpm
chkconfig php-fpm on

#禁用高危功能
sed -i '305d' /usr/local/php/etc/php.ini && sed -i '304 adisable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restor e,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,g etservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd,posix_getegid,posix_geteuid,posix_getgid,po six_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid,posix_getppid,posix_getpwnam,posix_ getpwuid,posix_getrlimit,posix_getsid,posix_getuid,posix_isatty,posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid,posix_ setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname' /usr/local/php/etc/php.ini

#这样就把php服务程序配置妥当了。最后，还需要编辑Nginx服务程序的主配置文件，把第2行的井号（#）删除，然后在后面写上负责运行Nginx服务程序的账户名称和用户组名称；在第45行的index参数后面写上网站的首页名称。最后是将第65～71行参数前的井号（#）删除来启用参数，主要是修改第69行的脚本名称路径参数，其中$document_root变量即为网站信息存储的根目录路径，若没有设置该变量，则Nginx服务程序无法找到网站信息，因此会提示“404页面未找到”的报错信息。在确认参数信息填写正确后便可重启Nginx服务与php-fpm服务。
sed -i '2d' /usr/local/nginx/conf/nginx.conf && sed -i '1 auser nginx nginx;' /usr/local/nginx/conf/nginx.conf 
sed -i '45d' /usr/local/nginx/conf/nginx.conf && sed -i '44 aindex index.html index.htm index.php;' /usr/local/nginx/conf/nginx.conf 
sed -i '65d' /usr/local/nginx/conf/nginx.conf && sed -i '64 alocation ~ \.php$ {' /usr/local/nginx/conf/nginx.conf 
sed -i '66d' /usr/local/nginx/conf/nginx.conf && sed -i '65 aroot html;' /usr/local/nginx/conf/nginx.conf 
sed -i '67d' /usr/local/nginx/conf/nginx.conf && sed -i '66 afastcgi_pass 127.0.0.1:9000;' /usr/local/nginx/conf/nginx.conf 
sed -i '68d' /usr/local/nginx/conf/nginx.conf && sed -i '67 afastcgi_index index.php;' /usr/local/nginx/conf/nginx.conf 
sed -i '69d' /usr/local/nginx/conf/nginx.conf && sed -i '68 afastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;' /usr/local/nginx/conf/nginx.conf
sed -i '70d' /usr/local/nginx/conf/nginx.conf && sed -i '69 ainclude fastcgi_params;' /usr/local/nginx/conf/nginx.conf 
sed -i '71d' /usr/local/nginx/conf/nginx.conf && sed -i '70 a}' /usr/local/nginx/conf/nginx.conf

#重新加载nginx，重启php-fpm
/usr/local/nginx/sbin/nginx -s reload
systemctl restart php-fpm

echo "恭喜！LNMP动态网站环境架构的配置已完成！"
echo "MySQL已安装成功，请执行 source /etc/profile 后，再执行 mysql_secure_installation 来完成MySQL数据库的初始化！"
```
