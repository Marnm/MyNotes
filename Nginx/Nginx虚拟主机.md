---
title: Nginx虚拟主机
tags: []
---

## 故障

```
[root@ctos74 nginx]# nginx -t
nginx: [emerg] "server" directive is not allowed here in /etc/nginx/default.d/web-server.conf:1
nginx: configuration file /etc/nginx/nginx.conf test failed
```
> xxx.conf不允许放在server里面

**一般是配置参数位置放错了，将参数放到正确的位置**
