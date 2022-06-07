---
title: RedHat系列
tags: []
---

配置网络dhcp自动分配IP静态IP通过nmtui工具配置网络nmtui是NetworkManager的一个网络管理工具在终端输入nmtui命令打开网络管理界面通过方向键或Tab键选择菜单，Enter键与Esc键来进行确认与取消操作

 查看IP地址
ifconfig    ip add show

 查看路由表

netstat -rn
route -n

 重启网卡  
 rhel 6/centos 6  
service network restart  
/etc/init.d/network restart  

 rhel7/CentOS7  
systemctl restart network  
nmcli connection reload  

/etc/init.d/network restart  

配置YUM源  
	1. 挂载ISO镜像  
 
mkdir /media/cdrommount /dev/cdrom /media/cdrom  
	1. 设置光盘自动挂载  

echo "/dev/cdrom /media/cdrom iso9660 defaults 0 0" >> /etc/fstab  

1. 修改yum配置文件  

vim /etc/yum.repos.d/yum.repo  

[yum name]  
name=Local Yum repository   
baseurl=file:///media/cdrom  
enabled=1  
gpgcheck=0  

禁用网卡命名规则  

vim /etc/sysconfig/grub或编辑软链接文件vim /etc/default/grub添加net.ifname=0 biosdevname=0参数  

重新生成grub配置文件：  

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


## 搭建本地yum和epel仓库

> 本实例在rhel 7.3系统上实现

**先使用光盘配置本地Yumy源**

```shell
# 挂载光盘
mount /dev/cdrom /media/cdrom

# 配置本地yum源
cat > /etc/yum.repos.d/yum.repo << EOF
[local]
name=Local YUM Repository
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
EOF

yum clean all && yum makecache
```

**安装软件包**

	yum install -y createrepo yum-utils wget


**下载软件包并配置本地仓库**

```
mkdir -p /var/www/html/iso
mkdir /tmp/iso
mount -o loop /root/CentOS-7-x86_64-xxx.iso /tmp/iso
cp -r /tmp/iso/ /var/www/html/iso
```

**epel源**

```
mkdir -p /var/www/html/epel
# 执行下载，根据你的网速解决完成速度
wget -np -H --cut-dirs=0 -r -c -L https://dl.fedoraproject.org/pub/epel/7/x86_64 -P /var/www/html/epel
# -np 不要追溯到父目录
# -H 当递归时转到外部主机
# --cut-dirs=0 忽略0层远程目录
# -r 递归下载－－慎用!
# -c  接着下载没下载完的文件
# -L 仅仅跟踪相对链接
# -P 将文件保存到目录
```

**使用reposync**
```
createrepo -v /var/www/html/epel
reposync -r epel -p /var/www/html/
```

**每当新增rpm包后需更新本地仓库**
```
createrepo -p -d -o /var/www/html/iso /var/www/html/iso
createrepo -p -d -o /var/www/html/epel /var/www/html/epel
或者
createrepo --update /var/www/html/iso
createrepo --update /var/www/html/epel
```

## 配置yum源优先级

**安装yum 的优先级插件。**

	rpm -ivh yum-plugin-priorities-1.1.31-40.el7.noarch.rpm

**修改yum配置文件**
```
[local-yum]
name=local iso yum
baseurl=file:///var/www/html/iso/rhel7
enabled=1
gpgcheck=0
priority=1
```
注意此处有`priority=1` ,代表优先级为1.

**重建缓存。**

使用命令

	 yum clean all
	 yum makecache

**可以将epel本地仓库制作成iso镜像文件**

```
yum install -y genisoimage
mkisofs -r -o /root/centos7-epel.iso /var/www/html/epel
```
