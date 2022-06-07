---
title: 01Redis概述
tags: []
---

# Redis概述

## 什么是Redis

Redis的出现时间并不长，是NoSQL的一种，基于键-值型的存储，与memcache类似，但是memcache中只是内存的缓存，而Redis不仅是内存中的缓存，还提供持久存储，在2009年第一次发布Redis。

Redis全称（Remote Dictionary Server）远程字典服务器，而这个字典服务器从本质上来讲，主要是提供数据结构的远程存储功能的，可以理解为Redis是一个高级的K-V存储，和数据结构存储，因为Redis除了能够存储K-V这种简单的数据之外，还能够存储列表、字典、hash表等对应的数据结构。

在性能上Redis不比memcache差，因为redis整个运行通通都是在内存中实现的，它的所有的数据集都是保存在内存中的，内存中的数据会周期性的写入到磁盘上，以实现数据的持久功能，而这种写磁盘并不是用于访问，而仅是冗余功能，所以redis所有功能都在内存中完成，因为此性能也是可想而知。

redis与mamcache不同之处在于redis有一个周期性的将数据保存到磁盘上的机制，而且不只一种，有两种机制，这也是redis持久化的一种实现，**另外与memcache有所区别的是，redis是单线程服务器，只有一个线程来响应所有的请求**
redis支持主从模式，但是redis的主从模式默认就有一个sentinel工具，从而实现主从架构的高可用，也就是说，redis能够借助于sentinel工具来监控主从节点，当主节点发生故障时，会自己提升另外一个从节点成为新的主节点

在redis 3.0版本发布，开始支持redis集群，从而可以实现分布式，可以将用户的请求分散至多个不同节点


## Redis所支持的数据类型

支持存储的数据类型有、String（字符串，包含整数）, List（列表）, Hash（关联数组）, Sets（集合）, Sorted Sets（有序集合）, Bitmaps（位图）, HyperLoglog 


## Redis性能评估

1、100万较小的键存储字符串，大概消耗100M内存

2、由于redis是单线程，如果服务器主机上有多个CPU，只有一个能够使用，但并不意味着CPU会成为瓶颈，因为redis是一个比较简单的K-V数据存储，CPU通常不会成为瓶颈的

3、在常见的linux服务器上，500K（50万）的并发，只需要一秒钟处理，如果主机硬件较好的情况下，每秒钟可以达到上百万的并发



## Redis与memcache对比：

* Memcache是一个分布式的内存对象缓存系统
* 而redis是可以实现持久存储
* Memcache是一个LRU的缓存
* redis支持更多的数据类型
* Memcache是多线程的
* redis是单线程的
* 二者性能几乎不相上下，实际上redis会受到硬盘持久化的影响，但是性能仍然保持在与Memcache不相上下，是非常了不起的



## Redis的优势

* 丰富的（资料形态）操作  
string（字符串，包含整数）, List（列表）, Hash（关联数组）, Sets（集合）, Sorted Sets（有序集合）, Bitmaps（位图）, HyperLoglog
        
* 内建Replication和culster（自身支持复制及集群功能）  
* 支持就地更新（in-place update）操作，直接可以在内存中完成更新操作  
* 支持持久化（磁盘）  
避免雪崩效应，万一出现雪崩效应，所有的数据都无法恢复，但redis由于有持久性的数据，可以实现恢复



## Memcached的优势

* 多线程
* 善用多核CPU
* 更少的阻塞操作
* 更少的内存开销
* 更少的内存分配压力
* 可能有更少的内存碎片



# 安装Redis

https://redis.io  官网  
https://download.redis.io/releases/  下载地址

## 源码编译安装

**获取源码包**
```shell
[root@ctos7mini ~]& wget https://download.redis.io/releases/redis-3.2.5.tar.gz
--2022-05-09 04:27:32--  https://download.redis.io/releases/redis-3.2.5.tar.gz
Connecting to 192.168.73.1:7890... connected.
Proxy request sent, awaiting response... 200 OK
Length: 1544040 (1.5M) [application/octet-stream]
Saving to: ‘redis-3.2.5.tar.gz’

100%[==================================================================================================================>] 1,544,040   1.10MB/s   in 1.3s

2022-05-09 04:27:35 (1.10 MB/s) - ‘redis-3.2.5.tar.gz’ saved [1544040/1544040]
```

**解压文件到`/usr/local/`**
```shell
[root@ctos7mini ~]& tar zxvf redis-3.2.5.tar.gz -C /usr/local/
redis-3.2.5/
redis-3.2.5/.gitignore
redis-3.2.5/00-RELEASENOTES
redis-3.2.5/BUGS
redis-3.2.5/CONTRIBUTING
redis-3.2.5/COPYING
redis-3.2.5/INSTALL
redis-3.2.5/MANIFESTO
redis-3.2.5/Makefile
redis-3.2.5/README.md
redis-3.2.5/deps/
redis-3.2.5/deps/Makefile
redis-3.2.5/deps/README.md
redis-3.2.5/deps/geohash-int/
redis-3.2.5/deps/geohash-int/Makefile
redis-3.2.5/deps/geohash-int/geohash.c
redis-3.2.5/deps/geohash-int/geohash.h
redis-3.2.5/deps/geohash-int/geohash_helper.c
..................省略..........................
```

**安装编译环境**
```shell
[root@ctos7mini redis-3.2.5]& yum install -y gcc gcc-c++ kernel-devel
```

**编译Redis**
```shell
[root@ctos7mini src]& make && make install
```

**创建redis.service启动文件**
```shell
[root@ctos7mini ~]& vim /usr/lib/systemd/system/redis.service
```

```ini
[Unit]
Description=Redis persistent key-value database
After=network.target
 
[Service]
ExecStart=/usr/local/redis-3.2.5/src/redis-server /usr/local/redis-3.2.5/redis.conf --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
 
[Install]
WantedBy=multi-user.target
```

创建redis-shutdown文件
```shell
[root@ctos7mini src]& vim /usr/libexec/redis-shutdown

##=========================================================##

#!/bin/bash
#
# Wrapper to close properly redis and sentinel
test x"$REDIS_DEBUG" != x && set -x

REDIS_CLI=/usr/local/redis-3.2.5/src/redis-cli

# Retrieve service name
SERVICE_NAME="$1"
if [ -z "$SERVICE_NAME" ]; then
   SERVICE_NAME=redis
fi

# Get the proper config file based on service name
CONFIG_FILE="/etc/$SERVICE_NAME.conf"

# Use awk to retrieve host, port from config file
HOST=`awk '/^[[:blank:]]*bind/ { print $2 }' $CONFIG_FILE | tail -n1`
PORT=`awk '/^[[:blank:]]*port/ { print $2 }' $CONFIG_FILE | tail -n1`
PASS=`awk '/^[[:blank:]]*requirepass/ { print $2 }' $CONFIG_FILE | tail -n1`
SOCK=`awk '/^[[:blank:]]*unixsocket\s/ { print $2 }' $CONFIG_FILE | tail -n1`

# Just in case, use default host, port
HOST=${HOST:-127.0.0.1}
if [ "$SERVICE_NAME" = redis ]; then
    PORT=${PORT:-6379}
else
    PORT=${PORT:-26739}
fi

# Setup additional parameters
# e.g password-protected redis instances
[ -z "$PASS"  ] || ADDITIONAL_PARAMS="-a $PASS"

# shutdown the service properly
if [ -e "$SOCK" ] ; then
    $REDIS_CLI -s $SOCK $ADDITIONAL_PARAMS shutdown
else
    $REDIS_CLI -h $HOST -p $PORT $ADDITIONAL_PARAMS shutdown
fi

##=========================================================##

[root@ctos7mini src]& chmod 755 /usr/libexec/redis-shutdown
```


```shell
[root@ctos7mini ~]& useradd redis -s /sbin/nologin
[root@ctos7mini ~]& systemctl daemon-reload
[root@ctos7mini ~]& systemctl start redis
[root@ctos7mini ~]& netstat -pantul | grep 6379
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      1282/redis-server *
tcp6       0      0 :::6379                 :::*                    LISTEN      1282/redis-server *
[root@ctos7mini ~]&
```

```mbox
[root@ctos7mini ~]& redis-server &
[1] 1282
[root@ctos7mini ~]& 1282:C 09 May 04:53:34.532 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1282:M 09 May 04:53:34.533 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.5 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1282
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

1282:M 09 May 04:53:34.540 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1282:M 09 May 04:53:34.540 # Server started, Redis version 3.2.5
1282:M 09 May 04:53:34.541 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1282:M 09 May 04:53:34.541 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1282:M 09 May 04:53:34.541 * DB loaded from disk: 0.000 seconds
1282:M 09 May 04:53:34.541 * The server is now ready to accept connections on port 6379

[root@ctos7mini ~]&
```

```mbox
[root@ctos7mini ~]& redis-cli
127.0.0.1:6379>
```

## rpm安装Redis

```shell
[root@ctos7mini ~]& yum install https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/r/redis-3.2.12-2.el7.x86_64.rpm
Loaded plugins: fastestmirror
redis-3.2.12-2.el7.x86_64.rpm                                                                                                                        | 544 kB  00:00:01
Examining /var/tmp/yum-root-OuFo3f/redis-3.2.12-2.el7.x86_64.rpm: redis-3.2.12-2.el7.x86_64
Marking /var/tmp/yum-root-OuFo3f/redis-3.2.12-2.el7.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package redis.x86_64 0:3.2.12-2.el7 will be installed
--> Processing Dependency: libjemalloc.so.1()(64bit) for package: redis-3.2.12-2.el7.x86_64
Loading mirror speeds from cached hostfile
 * base: centos.nethub.com.hk
 * epel: mirrors.neusoft.edu.cn
 * extras: centos.nethub.com.hk
 * updates: centos.nethub.com.hk
--> Running transaction check
---> Package jemalloc.x86_64 0:3.6.0-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================================
 Package                             Arch                              Version                                  Repository                                             Size
============================================================================================================================================================================
Installing:
 redis                               x86_64                            3.2.12-2.el7                             /redis-3.2.12-2.el7.x86_64                            1.4 M
Installing for dependencies:
 jemalloc                            x86_64                            3.6.0-1.el7                              epel                                                  105 k

Transaction Summary
============================================================================================================================================================================
Install  1 Package (+1 Dependent package)

Total size: 1.5 M
Total download size: 105 k
Installed size: 1.7 M
Is this ok [y/d/N]: y
Downloading packages:
jemalloc-3.6.0-1.el7.x86_64.rpm                                                                                                                      | 105 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : jemalloc-3.6.0-1.el7.x86_64                                                                                                                              1/2
  Installing : redis-3.2.12-2.el7.x86_64                                                                                                                                2/2
  Verifying  : redis-3.2.12-2.el7.x86_64                                                                                                                                1/2
  Verifying  : jemalloc-3.6.0-1.el7.x86_64                                                                                                                              2/2

Installed:
  redis.x86_64 0:3.2.12-2.el7

Dependency Installed:
  jemalloc.x86_64 0:3.6.0-1.el7

Complete!

[root@ctos7mini ~]& systemctl start redis
[root@ctos7mini ~]&
```
![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205082109184.png)
注意：使用RPM包安装redis会依赖jemalloc程序包，jemalloc是内存分配工具，此包在epel源中
