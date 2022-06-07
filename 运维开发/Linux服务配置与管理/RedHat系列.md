---
title: RedHat系列
tags: []
---

配置网络dhcp自动分配IP静态IP通过nmtui工具配置网络nmtui是NetworkManager的一个网络管理工具在终端输入nmtui命令打开网络管理界面通过方向键或Tab键选择菜单，Enter键与Esc键来进行确认与取消操作# 查看IP地址ifconfig    ip add show# 查看路由表netstat -rnroute -n# 重启网卡# rhel 6/centos 6service network restart/etc/init.d/network restart# rhel7/CentOS7systemctl restart networknmcli connection reload/etc/init.d/network restart配置YUM源
	1. 挂载ISO镜像

mkdir /media/cdrommount /dev/cdrom /media/cdrom
	1. 设置光盘自动挂载

echo "/dev/cdrom /media/cdrom iso9660 defaults 0 0" >> /etc/fstab
	1. 修改yum配置文件

vim /etc/yum.repos.d/yum.repo[yum name]name=Local Yum repository baseurl=file:///media/cdromenabled=1gpgcheck=0禁用网卡命名规则vim /etc/sysconfig/grub或编辑软链接文件vim /etc/default/grub添加net.ifname=0 biosdevname=0参数重新生成grub配置文件：

grub2-mkconfig -o /boot/grub2/grub.cfg
重启后网卡编程eth*格式

修改udev网卡命名规则

cd /etc/udev/rules.d/
vim 70-persistent-ipoib.rules
在文件末尾新增一行配置，（默认网卡的MAC地址）：

SUBSYSTEM=="net",ACTION=="add",DRIVERS=="?*",ATTR{address}=="00:50:56:29:0e:ae",ATTR｛type｝=="1" ,KERNEL=="eth*",NAME="eth0"
重新生成grub配置文件

grub2-mkconfig -o /boot/grub2/grub.cfg
重启系统
