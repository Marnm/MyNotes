---
title: swap交换分区
tags:
  - swap
---

### 扩展交换分区
**方法一：创建交换分区**
> 使用fdisk/gdisk等分区工具创建一个文件系统
> 使用mkswap创建交换分区

```bash
fdisk /dev/sdb
n
...
+1G
...
w
# 设置交换分区
mkswap /dev/sdb1

# 启用交换分区
swapon /dev/sdb1

# 开机挂载
vim /etc/fstab
#添加如下参数
/dev/sdb1   swap swap   defaults 0 0
```

**方法二：创建交换文件**
```bash
# 以创建一个512MB大小的文件为例：
dd if=/dev/zero of=/swapfile bs=1024 count=524288

# 创建swap格式文件
mkswap /swapfile

# 启用交换分区
swapon /swapfile

# 开机挂载
vim /etc/fstab
/swapfile   swap    swap    defaults 0 0

```

### 删除交换空间
```bash
# 禁用交换分区/交换文件
swapoff /dev/sdb1
#swapoff /swapfile
```
从`/etc/fstab`删除swap项目，使用fdisk或yast工具删除分区
