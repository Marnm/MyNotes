---
title: 03Redis基本操作
tags: []
---

# Redis基本操作

```mbox
[root@ctos7mini ~]& redis-cli -h
redis-cli 3.2.5

Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      指定主机IP (default: 127.0.0.1).
  -p <port>          指定端口socket文件进行通信 (default: 6379).
  -s <socket>        指定socket文件，如果客户端和服务端都在同一台主机，可以指定socket文件进行通信
  -a <password>      指定认证密码.
  -r <repeat>        连接成功后指定运行的命令N次.
  -i <interval>      连接成功后每个命令执行完成等待时间，使用-i选项指定.
                     
  -n <db>            Database number.
.......................省略...........................................

[root@ctos7mini ~]&
```

## 登录Redis
```mbox
[root@ctos7mini ~]& redis-cli
127.0.0.1:6379> exit
[root@ctos7mini ~]&

或者
[root@ctos7mini ~]& redis-cli -h 127.0.0.1
127.0.0.1:6379>
```

## 获取Redis帮助
```mbox
127.0.0.1:6379> help
redis-cli 3.2.5
To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit

To set redis-cli perferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
127.0.0.1:6379>

或者,获取指定数据结构的帮助
127.0.0.1:6379> help @string

  APPEND key value
  summary: Append a value to a key
  since: 2.0.0

  BITCOUNT key [start end]
  summary: Count set bits in a string
  since: 2.6.0

  BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]
  summary: Perform arbitrary bitfield integer operations on strings
  since: 3.2.0

  BITOP operation destkey key [key ...]
  summary: Perform bitwise operations between strings
  since: 2.6.0
 .......................................................省略...............................................................
 
 
```

**获取单个命令的使用方法**
```mbox
127.0.0.1:6379> help APPEND

  APPEND key value                         //命令方法
  summary: Append a value to a key
  since: 2.0.0                             //说明此命令在哪个版本中引入的 
  group: string                            //该命令所属哪一个组

127.0.0.1:6379>
```

带`@`的为组，不带则为命令

**切换库（名称空间）：**
```sql
127.0.0.1:6379> select 1 --表示切换到1号库中，默认为0号库，共16个，0-15
OK
127.0.0.1:6379[1]>
```

**set**
```sql
127.0.0.1:6379> help set

  SET key value [EX seconds] [PX milliseconds] [NX|XX] -- 命令 键 值 [EX过期时间，单位秒s]
  summary: Set the string value of a key
  since: 1.0.0
  group: string

127.0.0.1:6379>
-- NX 如果一个键不存在，才创建并设定值，否则不允许设定
-- XX 如果一个键存在则设置键的值，如果不存在则不创建并不设置其值


-- 定义一个键并设置过期时间为10秒
127.0.0.1:6379> set value_1 404 EX 10
OK
127.0.0.1:6379> get value_1
"404"
127.0.0.1:6379> get value_1
"404"
127.0.0.1:6379> get value_1
"404"
127.0.0.1:6379> get value_1
"404"
127.0.0.1:6379> get value_1
"404"
127.0.0.1:6379> get value_1
(nil)
127.0.0.1:6379>
```

**获取键中的值**
```sql
127.0.0.1:6379> help get

  GET key
  summary: Get the value of a key
  since: 1.0.0
  group: string

127.0.0.1:6379>
```

**添加键中的值**
```sql
127.0.0.1:6379> help append

  APPEND key value
  summary: Append a value to a key
  since: 2.0.0
  group: string

127.0.0.1:6379>


127.0.0.1:6379> append value_1 502
(integer) 3
127.0.0.1:6379> get value_1
"502"
127.0.0.1:6379>
```

**获取值的字符串长度**
```sql
127.0.0.1:6379> help strlen

  STRLEN key
  summary: Get the length of the value stored in a key
  since: 2.2.0
  group: string

127.0.0.1:6379>

127.0.0.1:6379> strlen value_1
(integer) 3
127.0.0.1:6379>
```
