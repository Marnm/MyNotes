---
title: 044Redis的数据备份与恢复
tags: []
---

# Redis的数据备份与恢复

## Linux

1. 启动Redis

进入Redis目录

	redis-cli
	
2. 数据备份

	SAVE
	
该命令将在redis备份目录中创建dump.rdb文件。

3. 恢复数据

		config get dir

以上命令config get dir输出的redis备份目录为/usr/local/redis/bin

停止Redis服务

拷贝备份文件到/usr/local/redis/bin目录下

重启redis

## Windows

	redis-server --service-install redis.windows.conf
	redis-server --service-start
	redis-server --service-stop
	
## 实战研究

### SAVE

	rm -rf /var/lib/redis/*
	redis-cli
	flushdb
	set jaking 666
	get jaking
	save
	quit
	ls /var/lib/redis/
	
	redis-cli
	get jaking
	del jaking
	get jaking
	quit
	ls /var/lib/redis/
	systemctl restart redis
	
	redis-cli
	get jaking
	
### BGSAVE

	rm -rf /var/lib/redis/*
	redis-cli
	flushdb
	
	set jaking 666
	get jaking
	
	bgsave
	quit
	
	ls /var/lib/redis/
	redis-cli
	
	get jaking
	del jaking
	get jaking
	quit
	
	ls /var/lib/redis/
	systemctl restart redis
	redis-cli
	get jaking
	
### BGREWRITEAOF

	rm -rf /var/lib/redis/*
	redis-cli
	
	set jaking 666
	bgrewirteaof
	quit
	
	ls /var/lib/redis/
	sz /var/lib/redis/appendonly.aof
	
	redis-cli
	get jaking
	del jaking
	get jaking
	quit
	
	systemctl restart redis
	redis-cli
	get jaking
	quit
	

**思考：为什么不能恢复数据？**

rdb是快照，定期把redis内存数据刷到dump.rdb

aof是记录命令

主动删除，不能恢复！除非是redis异常、重装、迁移才能恢复！

### RDB

**迁移数据**

	rpm -ivh redis-5.0.11-1.el7.remi.x86_64.rpm 
	
	systemctl start redis
	rm -rf /var/lib/redis/*
	cd /var/lib/redis
	rz
	
	redis-cli
	get jaking
	
**AOF**

	redis-cli
	set abc 123
	get abc
	
	bgrewriteaof
	quit
	ls
	
	sz appendonly.aof
	
**迁移数据**

	cat /etc/redis.conf
	
	chown redis.redis appendonly.aof
