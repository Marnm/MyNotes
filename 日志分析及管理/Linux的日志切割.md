---
title: Linux的日志切割
tags: []
---

> 如果日志是静态的，比如没有服务向里面写内容，那么可以利用`split`进行切割
> 如果日志是动态的，可以使用`logrotate` 进行切割


### logrotate
`/etc/logrotate.conf` 预设的配置文件
 


### 示例：对ssh进行日志分割
```bash
vim /etc/logrotate.d/sshd

/var/log/sshd.log{
    missingok
    weekly
    create 0600 root root
    minsize 1M
    rotate 3
}
```

**重启**
```bash
systemctl restart rsyslog
```

**日志切割**
```bash
logrotate -vf /etc/logrotate.d/sshd
```

### 日志集中管理
