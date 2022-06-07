---
title: Shell脚本工具
tags:
  - shell
  - bash
---

**查看有多少远程的IP在连接本机**
```sh
#!/bin/bash

netstat -atn | awk '{print $5}' | awk '{print $1}' | sort -nr | uniq -c
```

**Get Tomcat PID**
```shell
#!/bin/bash

v1="Hello"
v2="world"
v3=${v1}${v2}
echo $v3

pidlist=`ps -ef|grep apache-tomcat-7.0.75|grep -v "grep"|awk '{print $2}'`
echo $pidlist
echo "tomcat id list :$pidlist"  # 显示PID
```

**源码安装memcached**
```shell
#!/bin/bash

# 一键部署memcached

# 脚本用源码来安装memcached服务器
wget http://www.memcached.org/files/memcached-1.5.1.tar.gz
yum -y install gcc
tar -xf memcached-1.5.1.tar.gz
cd memcached-1.5.1
./configure
make && make install
```

**检测当前用户是否为管理员**
```shell
#!/bin/bash

# 检测本机当前用户是否为超级管理员，如果是管理员，则使用yum安装vsftpd，
# 如果不是，则提示您非管理员用户
if [ $USER == "root" ]
then
	yum -y install vsftpd
else
	echo "您不是管理员，没有权限安装软件"
	
fi
```

**杀掉tomcat并重新启动**
```shell
#!/bin/bash

# kill tomcat pid

# 找到tomcat的PID
pidlist=`ps -ef|grep apache-tomcat-7.0.75|grep -V "grep"|awk '{print $2}'`

echo "tomcat ID LIST : $pidlist"

kill -9 $pidlist

echo "KILL $pidlist:"
echo "service stop success"
echo "start tomcat"

cd /opt/apache-tomcat-7.0.75
pwd
rm -rf work/*
cd bin
./startup.sh 
```

**统计当前登录系统的用户数**
```shell
#!/bin/bash

# 统计当前Linux系统中可以登录计算机的账户有多少个

# 方法1；
grep "bash$" /etc/passwd | wc -l

# 方法2；
awk -f : '/bash$/{x++}end{print x}' /etc/passwd

```

**备份MySQL表数据**
```shell
#!/bin/bash

source /etc/profile
dbName=mysql
tableName=db

echo [`date+'%Y-%m-%d %H:%M:%S'`]' start loading data....'
mysql -uroot -P3306 ${ddbName} -e "LOAD DATA LOCAL INFILE '# /home/wenmin/wenxing.txt' INTO TABLE ${tableName} FIELDS TERMINATED BY ';'"
echo [`date +'%Y-%m-%d %H:%M:%S'`]'end loading data...'
exit

```

**使用死循环实时显示eth0网卡发送的数据包流量**
```shell
#!/bin/bash

# 使用死循环实时显示eth0网卡发送的数据包流量

while :
do
	echo '本地网卡 eth0 流量信息如下：'
	ifconfig eth0 | grep "RX pack" | awk '{print $5}'
	ifconfig eth0 | grep "TX pack" | awk '{print $5}'
done
```

**测试192.168.4.0/24整个网段中哪些主机处于开机状态，哪些主机处于关机状态**
```shell
for i in {1..254}
do
	# 每隔0.3秒ping一次，一共ping2次，并以1毫秒为单位设置ping的超时时间
	ping -c 2 -i 0.3 -W 1 192.168.1.$i $> /dev/null
	if [ $? -eq 0 ]; then
		echo "192.168.4.$i is up"
	else
		echo "192.168.168.4.$i is down"
	fi
done
```

**提示用户输入用户名和密码，脚本自动创建相应的账户及配置密码**
```shell
#!/bin/bash

# 编写脚本：提示用户输入用户名和密码，脚本自动创建相应的账户及配置密码
# 如果用户不输入账户名，则提示必须输入账户名并退出脚本
# 如果用户不输入密码，则统一使用默认的123456作为默认密码

read -p "请输入用户名：" user

# 使用-z可以判断一个变量是否为空，如果为空，提示用户必须输入账户名，并退出脚本，退出码为2
# 没有输入用户名脚本退出后，使用$?查看的返回码为2

if [ -z $user ]; then
	echo " 您必须输入账户名"
	exit 2
fi

# 使用stty -echo关闭shell的回显功能
stty -echo
read -p "请输入密码："pass
stty echo

pass=${pass:-123456}
useradd "$user"
echo "$pass" | passwd --stdin "$user"
```

**每周5使用tar命令备份/var/log下的所有日志文件**
```shell
#!/bin/bash

# 每周 5 使用 tar 命令备份/var/log 下的所有日志文件
# vim  /root/logbak.sh
# 编写备份脚本,备份后的文件名包含日期标签,防止后面的备份将前面的备份数据覆盖
# 注意 date 命令需要使用反引号括起来,反引号在键盘<tab>键上面


tar -czf log-`date +%Y%m%d`.tar.gz /var/log

# crontab -e #编写计划任务，执行备份脚本
00 03 * * 5 /home/wenming/datas/logbak.sh
```


**定义要监控的页面地址，对 tomcat 状态进行重启或维护**
```shell
#!/bin/bash

# function:自动监控tomcat进程，挂了就执行重启操作
# author：huanghong
# DEFINE

# 获取tomcat PPID
tomcatID=$(ps -ef |grep tomcat | grep -w 'apache-tomcat-7.0.75' |grep -v 'grep' |awk '{print $2}')

# tomcat_startup
StartTomcat=/opt/apache-tomcat-7.0.75/bin/startup.sh

# TomcatCache=/user/apache-tomcat-5.5.23/work

#定义要监控的页面地址
WebURL=http://192.168.242.113:8080/

# 日志输出
GetPageInfo=/dev/null
TomcatMonitorLog=/tmp/TomcatMonitor.log

Monitor() {
	echo "[info]开始监控tomcat...[$(date +'%F %H:%M:%S')]"
	if [$TomcatID]
	then
		echo "[info]tomcat进程ID为:$tomcatID."
		# 获取返回状态码
		TomcatServiceCode=$(curl -s -o $GetPageInfo -m 10 --connect-timeout 10 $WebURL -w %{http_code})
		if [ $TomcatServiceCode -eq 200 ]; then
			echo ""[info]返回码为$TomcatServiceCode,tomcat启动成功,页面正常.""
		else
			echo "[error]访问出错，状态码为$TomcatServiceCode,错误日志已输出到$GetPageInfo"  
			echo "[error]开始重启tomcat" 
			kill -9 $tomcatID # 杀掉tomcat进程
			sleep 3
			# rm -rf $TomcatCache
			$StartTomcat
		fi
		else
			echo "[error]进程不存在！tomcat自动重启..."
			echo "[info]$StartTomcat, 请稍侯......."
			$StartTomcat
		fi
		echo "---------------------------------------"
}

Monitor >> $TomCatMonitorLog
```

**通过位置变量创建Linux系统账户及密码**
```shell
#!/bin/bash

# 通过位置变量创建Linux系统账户及密码

# $1是执行脚本的第一个参数，$2是执行脚本的第二个参数

useradd "$1"
echo "$2" | passwd --stdin "$1"

```

**实时监控本机内存和硬盘剩余空间,剩余内存小于500M、根分区剩余空间小于1000M时,发送报警邮件给root管理员**
```shell
#!/bin/bash
# 实时监控本机内存和硬盘剩余空间,剩余内存小于500M、根分区剩余空间小于1000M时,发送报警邮件给root管理员

# 提取根分区剩余空间
dis_sike=$(free | awk '/Mem/{print $4}')
while:
do
	# 注意内存和磁盘提取空间大小都是以kb为单位
	if [ $disk_size -le 512000 -a $mem_size -le 1024000 ]
	then
		mail -s "Warning" root <<EOF
		Insufficient resource,资源不足
		EOF
	fi
done
```

**一键部署LAMP（rpm）**
```shell
#!/bin/bash


# 一键部署 LNMP(RPM 包版本)
# 使用 yum 安装部署 LNMP,需要提前配置好 yum 源,否则该脚本会失败
# 本脚本使用于 centos7.2 或 RHEL7.2

yum install -y httpd
yum install -y mariadb mariadb-devel mariadb-server
yum install -y httpd
yum install -y php php-mysql

systemctl start httpd mariadb
systemctl enable httpd mariadb
```
