---
title: 02MongoDB安装部署
tags: []
---

# MongoDB安装部署

**官方文档**

https://docs.mongodb.com/manual/


```shell
[root@server11 ~]& cat >/opt/mongo_cluster/mongo_28017/conf/mongo_28017.conf <<EOF
systemLog:
  destination: file
  logAppend: true
  path: /opt/mongo_cluster/mongo_28017/logs/mongodb.log

storage:
  journal:
    enabled: true
  dbPath: /data/mongo_cluster/mongo_28017
  directoryPerDB: true
  wiredTiger:
     engineConfig:
        cacheSizeGB: 1
        directoryForIndexes: true
     collectionConfig:
         blockCompressor: zlib
     indexConfig:
        prefixCompression: true

processManagement:
  fork: true
  pidFilePath: /opt/mongo_cluster/mongo_28017/pid/mongod.pid

net:
  port: 28017
  bindIp: 127.0.0.1,192.168.10.11

replication:
  oplogSizeMB: 1024
  replSetName: dba666
EOF
```

```shell
[root@rhel7 ~]& mongod -f /opt/mongo_cluster/mongo_28017/conf/mongo_28017.conf
about to fork child process, waiting until server is ready for connections.
forked process: 2463
child process started successfully, parent exiting
[root@rhel7 ~]& mongod -f /opt/mongo_cluster/mongo_28018/conf/mongo_28018.conf
about to fork child process, waiting until server is ready for connections.
forked process: 2492
child process started successfully, parent exiting
[root@rhel7 ~]& mongod -f /opt/mongo_cluster/mongo_28019/conf/mongo_28019.conf
about to fork child process, waiting until server is ready for connections.
forked process: 2521
child process started successfully, parent exiting
[root@rhel7 ~]&
```

```shell
[root@rhel7 ~]& netstat -pantul |grep mongo
tcp        0      0 0.0.0.0:28017           0.0.0.0:*               LISTEN      2463/mongod
tcp        0      0 0.0.0.0:28018           0.0.0.0:*               LISTEN      2492/mongod
tcp        0      0 0.0.0.0:28019           0.0.0.0:*               LISTEN      2521/mongod
tcp        0      0 0.0.0.0:27017           0.0.0.0:*               LISTEN      1410/mongod
```
