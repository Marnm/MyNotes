---
title: rsync数据同步
tags:
  - rsync
  - sync
---

## rsync基础
**rsync命令** 是一个远程数据同步工具，可通过LAN/WAN快速同步多台主机间的文件。rsync使用所谓的“rsync算法”来使本地和远程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。 rsync是一个功能非常强大的工具，其命令也有很多功能特色选项，我们下面就对它的选项一一进行分析说明。

### 一、rsync命令

> **基本格式：** 
>>  `rsync [option] 原始位置 目标位置`

**常用选项：**
	
	-a        归档模式，递归并保留对象属性，等同于-rlptgoD
	-v        显示同步过程详细(verbose)信息
	-z        在传输文件时进行压缩(compress)
	-H        保留硬连接文件
	-A        保留ACL属性
	--delete  删除目标位置有而原始位置没有的文件
	-r        递归模式，包含目录及子目录中所有文件
	-l        对于软链接文件仍然复制为软链接文件
	-p        保留文件的权限标记
	-t        保留文件的时间标记
	-g        保留文件的属组标记（仅超级用户使用
	-o        保留文件属主标记（仅超级用户使用
	-D        保留设备文件及其他特殊文件
	==========================================================
	-q        --quiet精简输出模式
	-c        --checksum 打开校验开关，强制对文件传输进行校验
	-R        --relative 使用相对路径信息。
	-b        --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀
	



## 配置rsync

Linux默认已经安装了rsync工具

### 基于SSH的同步源  

SSH方式是通过系统用户来进行备份的。  

首先，在服务端创建一个用户：

```xml
[root@ctos7mini ~]# useradd rsync_user
[root@ctos7mini ~]# passwd rsync_user
Changing password for user rsync_user.
New password:
BAD PASSWORD: The password fails the dictionary check - it is too simplistic/systematic
Retype new password:
passwd: all authentication tokens updated successfully.
[root@ctos7mini ~]#
```

在客户端创建一个目录，同步服务端的数据：
```xml
[root@rhel7 ~]# mkdir -p /client/ssh
[root@rhel7 ~]# cd !$
cd /client/ssh
[root@rhel7 ssh]#
##################################################

[root@rhel7 ssh]# rsync -avz rsync_user@192.168.73.20:/var/www/html/* /client/ssh/
rsync_user@192.168.73.20's password:
receiving incremental file list
index.html

sent 42 bytes  received 230 bytes  60.44 bytes/sec
total size is 203  speedup is 0.75
[root@rhel7 ssh]#
```
同步客户端数据到服务端：
```xml
[root@rhel7 ssh]# touch {a..c}.txt
[root@rhel7 ssh]# rsync -avz /client/ssh/* rsync_user@192.168.73.20:/var/www/html/
rsync_user@192.168.73.20's password:
sending incremental file list
a.txt
b.txt
c.txt
rsync: mkstemp "/var/www/html/.a.txt.U9FvU4" failed: Permission denied (13)
rsync: mkstemp "/var/www/html/.b.txt.1CnYNT" failed: Permission denied (13)
rsync: mkstemp "/var/www/html/.c.txt.8E9qHI" failed: Permission denied (13)

sent 177 bytes  received 69 bytes  70.29 bytes/sec
total size is 203  speedup is 0.83
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1052) [sender=3.0.9]
[root@rhel7 ssh]#
```
`rsync error：`同步失败，提示对`/var/www/html`没有写入权限

使用`setfacl`对服务端的`/var/www/html`目录设置安全的ACL权限
```xml
[root@ctos7mini ~]# setfacl -m user:rsync_user:rwx /var/www/html/
```

然后再同步一次：
```xml
[root@rhel7 ssh]# rsync -avz /client/ssh/* rsync_user@192.168.73.20:/var/www/html/
rsync_user@192.168.73.20's password:
sending incremental file list
a.txt
b.txt
c.txt

sent 177 bytes  received 69 bytes  98.40 bytes/sec
total size is 203  speedup is 0.83
[root@rhel7 ssh]#
```

查看服务端的/var/www/html/目录，可以看到已经同步过来了
```xml
[root@ctos7mini ~]# ls /var/www/html/
a.txt  b.txt  c.txt  index.html
[root@ctos7mini ~]#
```
#### 配置免密登录

生成ssh密钥
```xml
[root@rhel7 rsync]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): rsync_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in rsync_rsa.
Your public key has been saved in rsync_rsa.pub.
The key fingerprint is:
d6:7d:cb:44:81:d8:e6:76:e8:a5:98:9a:0b:a4:c7:50 root@rhel7
The key's randomart image is:
+--[ RSA 2048]----+
|           o ..  |
|          . +  . |
|      E    o ..  |
|     .   . .+.o  |
|    . . S .=.+o  |
|     = .  o o+ . |
|    . +  o    o  |
|     . .o        |
|        ..       |
+-----------------+
[root@rhel7 rsync]#
```

添加密钥到rsync服务端

```xml
[root@rhel7 rsync]# ssh-copy-id -i rsync_rsa.pub rsync_user@192.168.73.20
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
rsync_user@192.168.73.20's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'rsync_user@192.168.73.20'"
and check to make sure that only the key(s) you wanted were added.

[root@rhel7 rsync]#
```

上传测试：

```xml
[root@rhel7 client]# touch txt{1..5}
[root@rhel7 client]# ls
txt1  txt2  txt3  txt4  txt5
[root@rhel7 client]# rsync -avz /client/* rsync_user@192.168.73.20:/var/www/html/
sending incremental file list
txt1
txt2
txt3
txt4
txt5

sent 237 bytes  received 107 bytes  688.00 bytes/sec
total size is 0  speedup is 0.00
[root@rhel7 client]#
```


### 后台(daemon)服务方式

启动rsync服务，编辑/etc/xinetd.d/rsync文件，将其中的disable=yes改为disable=no，并重启xinetd服务，如下：

```gherkin
vi /etc/xinetd.d/rsync

#default: off
# description: The rsync server is a good addition to an ftp server, as it \
# allows crc checksumming etc.
service rsync {
disable = no
socket_type = stream
wait = no
user = root
server = /usr/bin/rsync
server_args = --daemon
log_on_failure += USERID
}
```

创建配置文件，默认安装好rsync程序后，并不会自动创建rsync的主配置文件，需要手工来创建，其主配置文件为“/etc/rsyncd.conf”，创建该文件并插入如下内容：
```gherkin
vi /etc/rsyncd.conf

address = 192.168.73.20
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log

[share]
comment = soft
path=/server/rsync
read only = no
dont compress = *.gz *.bz2 *.zip
auth users = works
secrets file = /etc/rsyncd_users.db
```

创建密码文件，采用这种方式不能使用系统用户对客户端进行认证，所以需要创建一个密码文件，其格式为“username:password”，用户名可以和密码可以随便定义，最好不要和系统帐户一致，同时必须要把创建的密码文件权限设置为600。
```shell
echo "works:123456" > /etc/rsyncd_users.db
chmod 600 /etc/rsyncd_users.db
```

启动rsync服务：
```xml
[root@ctos7mini ~]# rsync --daemon
[root@ctos7mini ~]# netstat -pantu | grep 873
tcp        0      0 192.168.73.20:873       0.0.0.0:*               LISTEN      2645/rsync
[root@ctos7mini ~]#
```

创建rsync数据存放目录：
```xml
[root@ctos7mini ~]# mkdir -p /server/rsync
[root@ctos7mini ~]# cd !$
cd /server/rsync
[root@ctos7mini rsync]# touch rsyncd.txt
[root@ctos7mini rsync]#
```

在客户端执行同步操作：
```xml
[root@rhel7 ssh]# rsync -avz works@192.168.73.20::share /client/rsync/
Password:
receiving incremental file list
created directory /client/rsync
./
rsyncd.txt

sent 49 bytes  received 100 bytes  33.11 bytes/sec
total size is 0  speedup is 0.00
[root@rhel7 ssh]# ls /client/rsync/
rsyncd.txt
[root@rhel7 ssh]#
```
在客户端执行上行同步操作：  
先修改服务端同步目录的权限
```xml
[root@ctos7mini rsync]# setfacl -m u:nobody:rwx /server/rsync/
[root@ctos7mini rsync]# setfacl -m u:nobody:rwx /server/rsync
[root@ctos7mini rsync]# systemctl restart rsyncd
[root@ctos7mini rsync]# rsync --daemon
[root@ctos7mini rsync]#
```
然后在客户端执行同步操作：
```xml
[root@rhel7 client]# pwd
/client/rsync
[root@rhel7 rsync]# touch file{1..5}
[root@rhel7 rsync]# ls
file1  file2  file3  file4  file5
[root@rhel7 rsync]# rsync -avz /client/rsync/* works@192.168.73.20::share
Password:
sending incremental file list
file1
file2
file3
file4
file5

sent 234 bytes  received 103 bytes  74.89 bytes/sec
total size is 0  speedup is 0.00
[root@rhel7 rsync]#
```
## rsync托管xinetd

如果`disable = no`，rsync默认使用xinetd服务，修改为`disable=yes`即可。
```xml
[root@ctos7mini ~]# cat /etc/xinetd.d/rsync
service rsync
{
        disable = yes
        socket_type = stream
        wait = no
        user = root
        server = /usr/bin/rsync
        server_args = --daemon
        log_on_failure += USERID
}
[root@ctos7mini ~]#
```


## 系统服务脚本

### inotify实时监控同步信息
```shell
#!/bin/bash
#filename watchdir.sh
path=$1
/usr/bin/inotifywait -mrq --timefmt '%d/%m/%y/%H:%M' --format '%T %w %f' -e modify,delete,create,attrib $path
```

```xml
[root@rhel7 ~]# ls /client/
txt1  txt2  txt3  txt4  txt5
[root@rhel7 ~]# rsync -avz --password-file=/etc/rsync.pass /client/ rsync_user@192.168.73.20::backup
sending incremental file list
./
txt1
txt2
txt3
txt4
txt5

sent 246 bytes  received 106 bytes  140.80 bytes/sec
total size is 0  speedup is 0.00
[root@rhel7 ~]#
```

```xml
[root@ctos7mini development]# ./watchdir.sh /server/
05/05/22/21:50 /server/
05/05/22/21:50 /server/ .txt1.oo2ezf
05/05/22/21:50 /server/ .txt1.oo2ezf
05/05/22/21:50 /server/ .txt2.s76Fka
05/05/22/21:50 /server/ .txt2.s76Fka
05/05/22/21:50 /server/ .txt3.7IR754
05/05/22/21:50 /server/ .txt3.7IR754
05/05/22/21:50 /server/ .txt4.zm3zRZ
05/05/22/21:50 /server/ .txt4.zm3zRZ
05/05/22/21:50 /server/ .txt5.Rv05KU
05/05/22/21:50 /server/ .txt5.Rv05KU
05/05/22/21:50 /server/

```

### 实时同步

```shell
#!/bin/bash

#功能：利用inotify和rsync实现数据的实时同步
#作者：传棋
#邮箱：Jaking@vip.163.com
#时间：2022年4月11日

SRC='/data/'  
DEST='rsyncuser@192.168.10.11::backup'    
LOG='/var/log/inotify_rsync.log' 
 
/usr/bin/inotifywait -mrq --timefmt '%Y-%m-%d %H:%M' --format '%T %w %f' -e create,delete,moved_to,close_write,attrib ${SRC} | while read DATE TIME DIR FILE

do
 
   FILEPATH=${DIR}${FILE}
   /usr/bin/ssh -l root 192.168.10.12 /usr/bin/rsync -az --delete --password-file=/etc/rsync.pass  $DEST $SRC && /usr/bin/echo "At ${TIME} on ${DATE}, file $FILEPATH was backuped up via rsync" >> ${LOG}
 
done
```


## 结合inotify实时同步

inotify 一种强大的、细粒度的、异步文件系统监控机制，它满足各种各样的文件监控需要，可以监控文件系统的访问属性、读写属性、权限属性、删除创建、移动等操作，也就是可以监控文件发生的一切变化。

inotify-tools是一个C库和一组命令行的工作提供Linux下inotify的简单接口。inotify-tools安装后会得到`inotifywait`和`inotifywatch`这两条命令：

- **`inotifywait` 命令** 可以用来收集有关文件访问信息，Linux发行版一般没有包括这个命令，需要安装inotify-tools，这个命令还需要将inotify支持编译入Linux内核，好在大多数Linux发行版都在内核中启用了inotify。
- **`inotifywatch` 命令**  用于收集关于被监视的文件系统的统计数据，包括每个inotify事故发生多少次。

开始之前需要检测系统内核是否支持inotify：

使用`uname -r`命令检查linux内核，如果低于2.6.13，就需要重新编译内核加入inotify支持。  
使用`ll /proc/sys/fs/inotify`命令，是否有以下三条信息输出，如果没有表示不支持。  

```xml
[root@rhel7 ~]# ll /proc/sys/fs/inotify
total 0
-rw-r--r--. 1 root root 0 May  5 19:29 max_queued_events
-rw-r--r--. 1 root root 0 May  5 19:29 max_user_instances
-rw-r--r--. 1 root root 0 May  5 19:29 max_user_watches
[root@rhel7 ~]#
```
- **max_queued_events**：表示调用inotify_init时分配给inotify instance中可排队的event的数目的最大值，超出这个值的事件被丢弃，但会触发IN_Q_OVERFLOW事件。事件队列最大长度，如值太小会出现 Event Queue Overflow 错误，默认值: 16384
- **max_user_instances**：每个用户(real user id)创建inotify实例最大值，默认值: 128
- **max_user_watches**：表示每个inotify instatnces可监控的最大目录数量。如果监控的文件数目巨大，需要根据情况，适当增加此值的大小。可以监视的文件数量(单进程)，默认值: 8192  


根据需求实时调节大小：
```shell
echo 104857600 > /proc/sys/fs/inotify/max_user_watches
echo 'echo 104857600 > /proc/sys/fs/inotify/max_user_watches' >> /etc/rc.local
```

如果遇到以下错误：  
```xml
inotifywait: error while loading shared libraries: libinotifytools.so.0: cannot open shared object file: No such file or directory 
 **解决方法：** 
32位系统：ln -s /usr/local/lib/libinotifytools.so.0 /usr/lib/libinotifytools.so.0
64位系统：ln -s /usr/local/lib/libinotifytools.so.0 /usr/lib64/libinotifytools.so.0
```

**安装inotify-tools**

- inotify-tools项目地址：https://github.com/rvoicilas/inotify-tools
- inotify-tools下载地址：http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz

```xml
tar zxvf inotify-tools-3.14.tar.gz
cd inotify-tools-3.14
./configure
make
make install
```
其他发行版的安装方法：https://github.com/inotify-tools/inotify-tools/wiki#getting

**相关命令**  
`inotifywait`
	
	常用选项：
	-m, --monitor       持续监视变化
	-d, --daemon        以守护进程方式执行，和-m相似，配合-o使用
	-r, --recursive     递归监控目录数据信息变化
	-q, --quiet         减少冗余信息，只打印出需要的信息。
	--exclude <pattern> 指定排除文件或目录，使用扩展的正则表达式匹配模式实现
	--excluddi <pattern> 指定排除文件或目录，不区分大小写
	-o, --outfile <file> 打印事件到文件中，相当于标准正确输出
	-s, --syslogOutput   发送错误到syslog相当于标准错误输出
	--timefmt <fmt>      指定时间输出格式
	--format <fmt>       指定的输出格式；即实际监控输出内容
	-e                   指定要监视的事件列表。
	--timefmt <fms> 时间格式，参考man 3 strftime 例如：--timefmt "%Y-%m-%d %H:%M"
	
	%Y        年份，包含世纪信息
	%y        年份，不包含世纪信息
	%m        月份，范围01-12
	%d        日，范围01-31
	%H        小时，24小时制，范围00-23
	%M        分钟，范围00-59
	
	--format <fmt> 格式定义，例如 --format "%T %w%f event: %;e"  与  --format '%T %w %f'

　　　　%T 输出时间格式中定义的时间格式信息，通过 --timefmt option 语法格式指定时间信息
　　　　%w 事件出现时，监控文件或目录的名称信息
　　　　%f 事件出现时，将显示监控目录下触发事件的文件或目录信息，否则为空
　　　　%e 显示发生的事件信息，不同的事件默认用逗号分隔
　　　　%Xe显示发生的事件信息，不同的事件指定用X进行分隔
	

`-e`选项可监听的事件,例如：-e create,delete,moved_to,clise_write,attrib

| **事件** | **描述** |
| --- | --- |
| create | **访问** ，读取文件。 |
| modify | **修改** ，文件内容被修改。|
| attrib | **属性** ，文件元数据被修改。|
| move | **移动** ，对文件进行移动操作。|
| create | **创建** ，生成新文件 |
| open | **打开** ，对文件进行打开操作。 |
| close | **关闭** ，对文件进行关闭操作。 |
| delete | **删除** ，文件被删除。 |
| access | 文件或目录内容被读取 |
| moved_to | 文件或目录从监控的目录中被移动 |
| close_write | 文件或目录关闭，在写入模式打开之后关闭的 |
| close_nowrite | 文件或目录关闭，在只读模式打开之后关闭 |
| delete_self | 文件或目录被删除，目录本身被删除 |
| unmount | 取消挂载 |


**inofitywait示例：**
```xml
# 监控一次性事件
inotifywait /data

# 持续监控
inotifywait -mrq /data

# 持续后台监控，并记录日志
inotifywait -o /root/inotify.log -drq /data --timefmt "%Y-%m-%d %H:%M" --format "%T %w%f event: %e"

# 持续后台监控特定事件
inotifywait -mrq /data --timefmt "%F %H:%M" --format "%T %w%f event: %;e" -e create,delete,moved_to,close_write,attrib
```

### inotify配置示例

**服务端生成验证文件**
```shell
echo "rsync_user:123456" > /etc/rsync.pass
chmod 600 /etc/rsync.pass
```

**在服务端创建rsync备份存放目录**

```xml
[root@ctos7mini ~]# mkdir /server|
```

**修改rsync配置文件**

```xml
[root@ctos7mini ~]# vim /etc/rsyncd.conf

uid=root
gid=root
address = 192.168.73.20
port 874
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
host allow = 192.168.73.10

[backup]
comment=backup
path=/server
read only=no
dont compress = *.gz *.bz2 *.zip *.tgz
auth user = rsync_user
secrets file = /etc/rsync.pass
```
**启动rsync守护程序**
```xml
[root@ctos7mini ~]# rsync --daemon
[root@ctos7mini ~]# netstat -pantu | grep 873
tcp        0      0 192.168.73.20:873       0.0.0.0:*               LISTEN      1806/rsync
```
**配置客户端验证文件**
```shell
echo "123456" > /etc/rsync_user.pass
chmod 600 /etc/rsync_user.pass
```

**测试**  
```xml
# 客户端目录
[root@rhel7 ~]# ls /client/
txt1  txt2  txt3  txt4  txt5

# 服务端目录
[root@ctos7mini ~]# ls /server/
[root@ctos7mini ~]#

# 客户端数据同步到服务端
[root@rhel7 ~]# rsync -avz --password-file=/etc/rsync.pass /client/ rsync_user@192.168.73.20::backup
sending incremental file list
./
txt1
txt2
txt3
txt4
txt5

sent 246 bytes  received 106 bytes  140.80 bytes/sec
total size is 0  speedup is 0.00
[root@rhel7 ~]#
```




## 故障解决

**rsync报错：Operation not permitted (1)**

从客户端推送到服务器报错，但文件是传过去了

```xml
[root@rhel7 rsync]# rsync -avz /client/rsync/* works@192.168.73.20::share
Password:
sending incremental file list
file1
file2
file3
file4
file5
rsync: chgrp "/.file1.Nu6Xxv" (in share) failed: Operation not permitted (1)
rsync: chgrp "/.file2.nym0yl" (in share) failed: Operation not permitted (1)
rsync: chgrp "/.file3.SRp6zb" (in share) failed: Operation not permitted (1)
rsync: chgrp "/.file4.EA4dB1" (in share) failed: Operation not permitted (1)
rsync: chgrp "/.file5.CnzwCR" (in share) failed: Operation not permitted (1)

sent 234 bytes  received 103 bytes  96.29 bytes/sec
total size is 0  speedup is 0.00
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1052) [sender=3.0.9]
[root@rhel7 rsync]#

###服务端：
[root@ctos7mini rsync]# ls
file1  file2  file3  file4  file5
[root@ctos7mini rsync]#
```

在检查目录权限的时候，是没有问题的：

```xml
[root@ctos7mini rsync]# getfacl /server/rsync/
getfacl: Removing leading '/' from absolute path names
# file: server/rsync/
# owner: root
# group: root
user::rwx
user:nobody:rwx
group::r-x
mask::rwx
other::r-x
```

**解决方法一：**  
不加`-a`参数，因为-a是归档模式传输，并保持所有文件属性，等价于-rtopgDl（还没有具体深入研究），可以使用下面这个命令替代：
```xml
[root@rhel7 rsync]# rsync -rltDvz /client/rsync/* works@192.168.73.20::share
```

**解决方法二：**  
配置文件不完善，导致没有root权限，需要重新修改配置文件，添加如下参数:  
```xml
vim /etc/rsyncd.conf

# 赋予目录root权限
uid=root        
gid=root        
address = 192.168.73.20
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log

[share]
comment = soft
path=/server/rsync
read only = no
dont compress = *.gz *.bz2 *.zip
auth users = works
secrets file = /etc/rsyncd_users.db
[root@ctos7mini rsync]#
```
重启rsync  
```xml
[root@ctos7mini rsync]# systemctl restart rsyncd
[root@ctos7mini rsync]# pkill rsync
[root@ctos7mini rsync]# rsync --daemon
[root@ctos7mini rsync]#
```
