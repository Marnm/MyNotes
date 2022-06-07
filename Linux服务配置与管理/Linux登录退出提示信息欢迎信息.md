---
title: Linux登录退出提示信息欢迎信息
tags:
  - motd
---

Ubuntu
```bash
vim /etc/update.motd.d/xxxx
```
目录中的文件名，数字越小的文件越先加载
用户手动创建的文件记得给权限
```bahs
chmod +x xxx
```

centos
```bash
vim /etc/motd
```
