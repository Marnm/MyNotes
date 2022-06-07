---
title: 无法挂载/sysroot并自动进入紧急模式
tags: []
---

如果系统出现非正常关机之后；
如意外断电之后，使文件系统出现错误状态，导致系统无法正常开机

开机后会自动进入紧急模式：

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072148052.png)

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072149556.png)

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072150205.png)


当尝试进行挂载的`/sysroot`目录的时候，会出现下面的报错：

    mount: can't find /sysroot in /etc/fstab

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072150291.png)


使用`journalctl -xb`查看系统报错信息

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072150012.png)

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072150628.png)

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072151724.png)

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072151364.png)

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072151116.png)

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072151396.png)


**解决方法**
需要使用`xfs_repair`命令修复文件系统

```shell
[sudo] xfs_repair -v /dev/dm-0
# 或：
# -L选项可能会损坏文件系统
[sudo] xfs_repair -v -L /dev/dm-0
```
