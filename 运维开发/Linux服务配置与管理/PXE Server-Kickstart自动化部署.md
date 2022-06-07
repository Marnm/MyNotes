---
title: PXE Server/Kickstart自动化部署
tags:
  - PXE
  - Kickstart
  - Cobbler
  - DHCP
  - DNS
  - FTP
  - TFTP
---

## Linux
### PXE/Kickstart

#### BIOS模式

配置网络服务器（NFS，HTTPS，HTTP，FTP）用于存储ISO镜像
**配置DHCP**

```bash
yum install -y dhcp
cp /usr/share/doc/dhcp*/dhcpd.conf.example /etc/dhcp/dhcpd.conf
vim /etc/dhcp/dhcpd.conf
```

```bash
allow booting;
allow bootp;
ddns-update-style interim;
ignore client-updates;
subnet 192.168.10.0 netmask 255.255.255.0 {
    option subnet-mask 255.255.255.0;
    option domain-name-servers 192.168.10.10;
    range dynamic-bootp 192.168.10.100 192.168.10.200;
    default-lease-time 21600;
    max-lease-time 43200;
    next-server 192.168.10.10;
    filename "pxelinux.0";
 }
```

```bash
systemctl restart dhcpd && systemctl enable dhcpd
```

**配置tftp 提供引导文件**

```bash
yum install -y tftp-server xinetd
vim /etc/xinet.d/tftp

service tftp {
    socket_type = dgram
    protocol = udp
    wait = yes
    user = root
    server = /usr/sbin/in.tftpd
    server_args = -s /var/lib/tftpboot
    disable = no
    per_source = 11
    cps = 100 2
    flags = IPv4
 
systemctl restart xinetd && systemctl enable xinetd
firewall-cmd --permanent --add-port=69/udp
firewall-cmd --reload
```

获取pxelinux.0文件

```bash
# 安装syslinux包
yum install -y syslinux
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
# 或
# 直接加压rpm包
mount -t iso9660 /dev/cdrom /media/cdrom -o loop,ro
cp -pr /media/cdrom/Packages/syslinux-x.xx-xx.x86_64.rpm ./
rpm2cpio syslinux-x.xx-xx.x86_64.rpm | cpio -dimv
# 复制pxelinux.0到/var/lib/tftpboot/pxelinux/中 
cp pxelinux.0 /var/lib/tftpboot/
```

复制引导文件到tftp根目录中

```bash
cp /media/cdrom/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/
cp /media/cdrom/isolinux/{vesamenu.c32,boot.msg} /var/lib/tftpboot/
```

引导菜单default文件

```bash
# 创建目录
mkdir /var/lib/tftpboot/pxelinux.cfg
# 创建配置文件
vim /var/lib/tftpboot/pxelinux.cfg/default
# 或
cp /media/cdrom/isolinux/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default
```

**配置FTP**
复制镜像安装文件到ftp目录

```bash
yum install -y vsftpd
cp -pvr /media/cdrom/* /var/ftp/pub/

firewall-cmd --permanent --add-service=ftp
firewall-cmd --reload
setsebool -P ftpd_connect_all_unreserved=on

systemctl restart vsftpd && systemctl enable vsftpd
```

**创建kickstart配置文件**
生成kickstart文件
[使用在线生成器（需要注册登录）](https://access.redhat.com/labs/kickstartconfig/)
生成后下载命名存放到/var/ftp/pub/ks.cfg
或
使用用户目录下的默认模板anaconda-ks.cfg

```bash
cp /root/anaconda-ks.cfg /var/ftp/pub/ks.cfg
chmod +r /var/ftp/pub/ks.cfg
```

```
---------------------dhcpd.confg--------------------------
 allow booting;
 allow bootp;
 ddns-update-style interim;
 ignore client-updates;
 subnet 192.168.10.0 netmask 255.255.255.0 {
     option subnet-mask 255.255.255.0;
     option domain-name-servers 192.168.10.10;
     range dynamic-bootp 192.168.10.100 192.168.10.200;
     default-lease-time 21600;
     max-lease-time 43200;
     next-server 192.168.10.10;
     filename "pxelinux.0";
 }
```

```
---------------------tftp-----------------------
disable = no
```

```
---------------------default--------------------
default linux
...
# 64 line 地址要准确，镜像文件放在哪个目录就填到哪个目录，除非ftp设置了默认目录
append initrd=initrd.img inst.stage2=ftp://192.168.128.120/pub ks=ftp://192.168.128.120/pub/ks.cfg quiet
```

```
---------------------ks.cfg---------------------
# 系统认证设置
auth --enableshadown --passalgo=sha512
# 文件服务地址

# 地址要准确，镜像文件放在哪个目录就填到哪个目录
url --url=ftp://192.168.128.120/pub
firstboot --enable
ignoredisk --only-use=sda
keyboard --vckeymap=us --xlayouts='us'
lang en_US.UTF-8
network --bootproto=dhcp --device=eno16777728 --onboot=off --ipv6=auto
network --hostname=localhost.localdomain
rootpw --iscrypted $6$pDjJf42g8C6pL069$iI.PX/yFaqpo0ENw2pa7MomkjLyoae2zjMz2UZJ7b H3UO4oWtR1.Wk/hxZ3XIGmzGJPcs/MgpYssoi8hPCt8b/
# 时区设置
timezone Asia/Shanghai --isUtc
# 分区方式
clearpart --all --initlabel
# 安装rpm包
 31 %packages
 32 @base
 33 @core
 34 @desktop-debugging
 35 @dial-up
 36 @fonts
 37 @gnome-desktop
 38 @guest-agents
 39 @guest-desktop-agents
 40 @input-methods
 41 @internet-browser
 42 @multimedia
 43 @print-client
 44 @x11
 45 
 46 %end
```

检查服务是否启动
systemctl status xinetd dhcpd

#### UEFI模式

在BOIS模式的基础上更新如下配置：

更新DHCP的配置文件，参考模板

```bash
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp*/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
allow booting;
allow bootp;
ddns-update-style interim;
ignore client-updates;
option rfc3442-classless-static-routes code 121 = array of integer 8;
option ms-classless-static-routes code 249 = array of integer 8;
option space pxelinux;
option pxelinux.magic code 208 = string;
option pxelinux.configfile code 209 = text;
option pxelinux.pathprefix code 210 = text;
option pxelinux.reboottime code 211 = unsigned integer 32;
option architecture-type code 93 = unsigned integer 16;
subnet 192.168.128.0 netmask 255.255.255.0 {
    option subnet-mask 255.255.255.0;
    option domain-name-servers 192.168.128.120;
    range dynamic-bootp 192.168.128.128 192.168.128.200;
    default-lease-time 21600;
    max-lease-time 43200;

    class "pxeclients"{
        match if substring(option vendor-class-identifier, 0, 9)="PXEClient";
        next-server 192.168.128.120;
        if option architecture-type = 00:07 {
            filename "shim.efi";
        } else {
            filename "pxelinux.0";
        }
    }
}
```

复制uefi引导文件

```bash
cp -pr /media/cdrom/Packages/shim-0.9-2.el7.x86_64.rpm 
cp -pr /media/cdrom/Packages/grub2-efi-2.02-0.44.el7.x86_64.rpm
rpm2cpio shim-0.9-2.el7.x86_64.rpm | cpio -dimv
rpm2cpio grub2-efi-2.02-0.44.el7.x86_64.rpm

# 复制相关引导文件到tftpboot/目录下
cp ./bootloaders/boot/efi/EFI/redhat/shim.efi /var/lib/tftpboot/
cp ./bootloaders/boot/efi/EFI/redhat/grubx64.efi /var/lib/tftpboot/
```

在/var/lib/tftpboot/下添加grub.cfg配置文件

```bash
vim /var/lib/tftpboot/uefi/grub.cfg：
set timeout=60
# menuentry 'RHEL 7'是引导的菜单名称
menuentry 'RHEL 7' {
	# vmlinuz的存放目录,如果文件存放在/var/lib/tftpboot/目录下，直接先vmlinuz
    linuxefi vmlinuz ip=dhcp
    # 镜像安装文件的存放目录
    inst.repo=ftp://192.168.73.120/pub/
    # initrd.img文件的存放发目录
    initrdefi initrd.img
}
```

> 启动所有服务systemctl restart xinetd dhcpd

复制镜像文件到uefi/目录下

```bash
cp /media/cdrom/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/
```

**查看LInux处于BIOS模式还是UEFI模式**

在UEFI模式下Linux系统在`/sys/firmware/`目录下存在`efi`目录，BIOS模式是没有这个目录的

```bash
ls -ld /sys/firmware/efi/

# BIOS模式下grub2引导文件为/etc/grub2.cfg
# EFI模式下grub2引导文件为/etc/grub2-efi.cfg
[root@pxelab tftpboot]# ls -lrt /etc/grub2.cfg
lrwxrwxrwx. 1 root root 22 Jan 22 17:31 /etc/grub2.cfg -> ../boot/grub2/grub.cfg
[root@pxelab ~]# ls -lrt /etc/grub2-efi.cfg
lrwxrwxrwx. 1 root root 31 Jan 24 07:18 /etc/grub2-efi.cfg -> ../boot/efi/EFI/redhat/grub.cfg




```

如果是EFI模式的系统，有如下目录/文件：

```sh
[root@pxelab ~]# ls -lrt /etc/grub2-efi.cfg
lrwxrwxrwx. 1 root root 31 Jan 24 07:18 /etc/grub2-efi.cfg -> ../boot/efi/EFI/redhat/grub.cfg
[root@pxelab ~]# ls -ld /boot/efi/EFI/
drwx------. 4 root root 4096 Jan 24 07:19 /boot/efi/EFI/
[root@pxelab ~]# ls -lrt /boot/efi/EFI/
total 8
drwx------. 2 root root 4096 Jan 24 07:19 BOOTdrwx------. 3 root root 4096 Jan 24 07:21 redhat
[root@pxelab ~]# ls -lrt /boot/efi/EFI/*
/boot/efi/EFI/BOOT:
total 1340
-rwx------. 1 root root   71784 Jul 20  2015 fallback.efi
-rwx------. 1 root root 1295704 Jul 20  2015 BOOTX64.EFI

/boot/efi/EFI/redhat:
total 5812
-rwx------. 1 root root 1289544 Jul 20  2015 shim-redhat.efi
-rwx------. 1 root root 1295704 Jul 20  2015 shim.efi
-rwx------. 1 root root 1282496 Jul 20  2015 MokManager.efi
-rwx------. 1 root root     176 Jul 20  2015 BOOT.CSV
-rwx------. 1 root root 1024464 Aug 29  2016 grubx64.efi
-rwx------. 1 root root 1024464 Aug 29  2016 gcdx64.efi
drwx------. 2 root root    4096 Jan 24 07:18 fonts
-rwx------. 1 root root    1024 Jan 24 07:21 grubenv
-rwx------. 1 root root    4245 Jan 24 07:21 grub.cfg
[root@pxelab ~]#
```




### Kickstart

生成kickstart文件
使用在线生成器（需要注册登录）https://access.redhat.com/labs/kickstartconfig/
或
使用用户目录下的默认模板anaconda-ks.cfg

> 【注意！】
> 配置的每部分必须按顺序指定，每部分内不必按需排序
> %addon、%packages、%pre、%post必须以%end结尾
> #注释

安装kickstart文件验证器

```bash
yum install pykickstart
```

使用ksvalidator命令验证ks文件的正确性

```bash
ksvalidator kickstart.ks
```

**BIOS**

```bash
append initrd=initrd.img inst.ks=[http/ftp/tftp/nfs]://192.168.73.120/var/lib/tftpboot/ks.cfg
```

**UEFI**

```bash
kernel vmlinuz inst.ks=[http/ftp/tftp/nfs]://192.168.73.120/var/tftpboot/ks.cfg
```

kickstart文件语法变化

```bash
ksverdiff -f RHEL6 -t RHEL7
# -f指定比较第一个发行版，-t指定比较最后一个发行版
```

使用SSL协议时，禁用SSLv2和SSLv3，该选项触发CVE-2014-3566漏洞

### Cobbler+Kickstart
关闭SELinux和防火墙，重启Linux

```bash
systemctl stop firewalld
systemctl disable firewalld
iptables -F
setenforce 0

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/configf
```



更新yum源和epel源

```bash
# 直接使用默认源
yum install -y epel-release
```

安装其他服务

```bash
yum install -y dhcp pykickstart httpd dhcp rsync tftp-server xinetd syslinux
```

安装Cobbler

```bash
yum install cobbler cobbler-web
```

修改服务配置文件

> vim /etc/cobbler/settings

```bash
问题 1：修改 server 地址为 192.168.1.7
改：390 server: 127.0.1
为：390 server: 192.168.1.7
问题 2：修改 next_server 地址为 192.168.1.7
改：278 next_server: 127.0.1
为：278 next_server: 192.168.1.7
改：242 manage_dhcp: 0
为：242 manage_dhcp: 1
```

> vim /etc/cobbler/dhcp.template

```bash
改：22 option routers 192.168.1.5; #修改默认网关地址
为：22 option routers 192.168.1.1; #以实际的网关为准
改：23 option domain-name-servers 192.168.1.1; #修改 DNS 地址
为：23 option domain-name-servers 114.114.114.114;
如下：

 21 subnet 192.168.1.0 netmask 255.255.255.0 {
 22      option routers             192.168.1.1;
 23      option domain-name-servers 114.114.114.114;
 24      option subnet-mask         255.255.255.0;
 25      range dynamic-bootp        192.168.1.100 192.168.1.254;
 26      default-lease-time         21600;
 27      max-lease-time             43200;
 28      next-server                $next_server;
```

启动服务

```bash
systemctl start httpd cobblerd
cobbler sync
systemctl enable httpd cobblerd
```

检查cobbler是否存在错误

```bash
cobbler check
systemctl restart cobblerd
cobbler sync
```

挂载光驱导入镜像

```bash
mount /dev/cdrom /mnt
cobbler import --path=/mnt/ --name=CentOS7.6 --arch=x86_64
```

查看cobbler镜像配置文件

```bash
cobbler list
```


## Windows
### PXE Server

### 修改Windows镜像
