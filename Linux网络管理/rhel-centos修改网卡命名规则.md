---
title: rhel/centos修改网卡命名规则
tags: []
---

## 修改网卡名称

```shell
mv /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/ifcfg-eth0
vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

```ini
TYPE=Ethernet
....
NAME=eth0
DEVICE=eth0
...
```

## 禁用网卡命名规则

> 此功能通过`/etc/default/grub`文件来控制，要禁用此功能，在文件中GRUB_CMDLINE_LINUX行的末尾加入`net.ifnames=0  biosdevname=0`即可

	[root@ctos74 ~]$ cat /etc/default/grub

```ini
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

```shell
sed -i 's/swap rhgb quiet/& net.ifnames=0 biosdevname=0/' /etc/default/grub
```

## 添加udev网卡规则
rhel7中可以添加到默认配置文件`70-persistent-ipoib.rules`中
CentOS直接新建一个`70-persistent-net.rules`文件

```shell
SUBSYSTEM=="net",ACTION=="add",DRIVERS=="?*",ATTR{address}=="需要修改名称的网卡MAC地址",ATTR｛type｝=="1" ,KERNEL=="eth*",NAME="eth0"
```

## 更新grub配置

	grub2-mkconfig -o /boot/grub2/grub.cfg

******

## 扩展： shell脚本实现以上全部操作

```shell
#!/bin/bash

NetInterfaceNameRules(){
# 修改网卡名称
cp /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/backupens33
echo -e "\e[1;32m网卡配置文件备份在/etc/sysconfig/network-scripts/backupens33\e[0m"
/usr/bin/sed -i 's/NAME=ens33/NAME=eth0/g' /etc/sysconfig/network-scripts/ifcfg-ens33
/usr/bin/sed -i 's/DEVICE=ens33/DEVICE=eth0/g' /etc/sysconfig/network-scripts/ifcfg-ens33

# 禁用网卡命名规则
cp /etc/default/grub /etc/default/backupgrub
echo -e "\e[1;32m备份文件：backupgrub\e[0m"
/usr/bin/sed -i 's/swap rhgb quiet/& net.ifnames=0 biosdevname=0/' /etc/default/grub

# 添加udev网卡规则
MAC_ADDR=`ip link show dev ens33 | awk 'END{ print $NR }'`
if [ -f /etc/udev/rules.d/70-persistent-ipoib.rules ]
then
    /usr/bin/sed -i '$a SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{addresss}=='\""$MAC_ADDR"\"',ATTR{type}=="1",KERNEL=="eth*",NAME="eth0"' /etc/udev/rules.d/70-persistent-*
else
    /usr/bin/echo 'SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{addresss}=='\""$MAC_ADDR"\"',ATTR{type}=="1",KERNEL=="eth*",NAME="eth0"' >> /etc/udev/rules.d/70-persistent-net.rules
fi

sudo /usr/sbin/grub2-mkconfig -o /boot/grub2/grub.cfg
}

NetInterfaceNameRules
```
