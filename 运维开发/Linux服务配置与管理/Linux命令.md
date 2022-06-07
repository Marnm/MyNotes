---
title: Linux命令
tags: []
---

# 增
`touch`创建一个文件
> -a：更新atime
> -c：更新ctime，若该文件不存在则不建立新文件
> -m：更新mtime
> -d：后面可以接更新日期而不使用当前日期，也可以使用 --date="日期或时间"
> -t：后面可以接更新时间不使用当前时间：格式[YYYYMMDDhhmm]}

`mkdir`创建目录
> -m：配置目录权限
> -p：递归创建目录
```bash
mkdir -m 700 /dir # 新建目录并赋予拥有者读写执行权限

mkdir -p /dir/dir2 # 新建多层目录
```

`cp`复制文件，如果复制2个以上文件，则目的文件一定要是目录才行
```bash
-a：相当于 -dr  --preserve=all
-d：若来源文件为链接文件，则复制链接文件属性而非文件本身
-i：若目标文件已经存在时，在覆盖前会先询问
-p：连同文件属性一起复制过去
-r：递归复制
-u：destination比source旧才更新destination，或destination不存在的情况下才复制
-l：创建硬链接
-s：创建软链接
--preserve=all ：除了-p的权限相关参数外，还加入了SELinux的属性，links，xattr等也复制了

```

----

# 删
`rm`删除文件
```bash
-r：递归删除
-f：强制删除
```

# 改
`mv`移动文件 或 重命名文件
```bash
-f：强制，如果目标文件存在，不询问直接覆盖
```

`chmod`修改权限
> 读  写  执行
>  r   w    x
>  4   2    1
> u拥有者user、g所属群组group、o其他人other、a所有人anyone
> +添加权限、-移除权限、=设定权限

```bash
# 所有用户添加写权限
chmod a+w .zshrc
```

# 查
`ls`
> -a：列出全部文件，（包括隐藏文件，
> -A，显示除 . 和 .. 外的全部文件，）
> -d：仅列出目录
> -l：以长数据行列出，包含文件的属性与权限等数据
> -F，给各类型文件添加标志（indicator） */=>@|，目录/、可执行文件\*、符号链接@、套接字socket=、FIFOs|

`history`
> history是一个有用的内建命令，要查看最近用过的命令列表，可以输入不带选项的history命令。
> 通常历史记录会保存最近的500-1000条命令。可以通过修改HISTSIE的环境变量设置保存命令数量
> .bash_history位于用户主目录，保存历史命令的记录文件
> 注意：bash命令的历史记录是先存放在内存中，当shell退出时才写入到历史文件中。要实现强制写入，使用history -a选项

```bash
-a 写入.bash_history文件
-n 强制读取.bash_history文件
!! # 使用刚刚用过的那条命令
!num    # 通过编号选择执行过的历史命令
```

不同颜色代表的文件类型
蓝色：目录
绿色：可执行文件
白色：一般性文件，如文本文件，配置文件等
红色：压缩文件或归档文件
浅蓝色：链接文件
红色闪烁：链接文件存在问题
黄色：设备文件
青黄色：管道文件

```bash
ls -l   # 列出当前目录可见详细信息
ls -hl  # 列出详细信息并以可读大小显示文件信息
ls -al  # 列出全部文件详细信息
ls --human-readable --size -1 -S --classfy  # 按文件大小排序
du -sh * | sort -h  # 按文件大小排序（同上），倒序
###################3
# 显示文件inode信息
# 索引节点（index inode，简称为inode）是Linux中一个特殊的概念，具有相同的索引节点号的两个文本本质上是同一个文件（除文件名不同外）
ls -i
ls -li  

# 最近修改的文件显示在最上面
ls -t
ls -tl

# 显示递归文件
ls -R

# 打印UID和GID
ls -n

# 逆序排序
ls -r
ls -l	列出可见文件详细信息
ls -hl	列出文件详细信息 并以可读大小显示文件大小
ls -al	列出文件的详细信息,包括影藏文件
ls --human-readable --size -1 -S --classify	按文件大小排序
du -sh * | sort -h 	按文件大小排序（从小到大排序）
ls -n 显示uid和gid
```

```bash
[root@localhost log]# ls -hl
total 640K
drwxr-xr-x. 2 root root  232 Jan 14 00:01 anaconda
drwx------. 2 root root   23 Jan 14 00:01 audit
-rw-r--r--. 1 root root 8.5K Jan 14 00:08 boot.log
-rw-------. 1 root utmp    0 Jan 13 23:57 btmp
-rw-------. 1 root root 2.4K Jan 14 03:01 cron
-rw-r--r--. 1 root root 125K Jan 14 00:04 dmesg
-rw-r--r--. 1 root root 125K Jan 14 00:01 dmesg.old
-rw-r--r--. 1 root root   73 Jan 14 00:08 firewalld
drwx------. 2 root root    6 Aug  3  2016 httpd
-rw-r--r--. 1 root root 285K Jan 14 02:40 lastlog
-rw-------. 1 root root  396 Jan 14 00:04 maillog
-rw-------. 1 root root 311K Jan 14 03:10 messages
drwx------. 2 root root    6 Jan 27  2014 ppp
drwxr-xr-x. 2 root root   43 Jan 14 00:06 rhsm
-rw-------. 1 root root 4.3K Jan 14 02:40 secure
-rw-------. 1 root root    0 Jan 13 23:57 spooler
-rw-------. 1 root root    0 Jan 13 23:57 tallylog
drwxr-xr-x. 2 root root   23 Jan 14 00:01 tuned
-rw-r--r--. 1 root root 3.2K Jan 14 00:04 vmware-vmsvc.log
-rw-rw-r--. 1 root utmp 4.9K Jan 14 02:40 wtmp
[-rw-------] [1] [root] [root] [2.2K] [Jan 14 02:50] [yum.log]
[    1     ] [2] [ 3  ] [ 4  ] [ 5  ] [     6      ] [  7    ]
[   权限   ][链接][拥有者][群组][文件大小][文件修改日期][文件名]
[root@localhost log]#
```
.gif
.gif

> 其中文件类型：
> [d]是目录     [-]是文件    
> [l]是链接      [b]是设备文件
> [c]是管道文件

pwd
```bash
pwd -P  # 打印当前目录的物理位置
```

`umask` 设置或查看默认权限
> 使用方法跟chmod类似
> 通常以掩码的形式来表示，例如0022表示组用户和其他用户的权限减去了一个2的权限，也就是写权限，
> 因此建立新文件时默认权限是rwxr-xr-x

### 查看Linux系统版本信息
```bash
lsb_release -a
cat /etc/redhat-release
cat /etc/issue
cat /proc/cpuinfo
cat /proc/version
uname -a
uname -r
```

### 查看内存
```bash
free -m 
free -h
cat /proc/meminfo
```

### 条件查询
`sort`、
> -n # 从小到大排序
> -r # 逆序
> -M # 按月份排序
> -t # 指定关键字区分位置
> -k # 指定列

```bash
# 按":"做分隔符，以第三列，进行正序排序
cort -t ":" -k3 -n /etc/passwd | more
```

cut
```bash
# 以 / 做分隔符，指定第三段的内容
cat a.txt | cut -d "/" -f 3
```

# 网络管理工具
## ip
```bash
ip link set ens38 down  # 关闭ens38网卡
ip link set ens38 name ens34    # 重命名ens38网卡为ens34
ip link set ens34 up    # 启用ens34网卡
```
## nmcli
> NetworkManager client网络管理客户端

```bash
nmcli connection show   # 查看当前连接状态
nmcli connection reload # 重启服务
nmcli connection show -active   # 显示活动的连接
nmcli connection show "lan ens33"   # 显示指定一个网络连接配置
nmcli device status # 显示设备状态
nmcli device show ens33 #显示指定接口属性
nmcli device show # 显示全部接口属性
nmcli connection up static  # 启动static连接配置
nmcli con up default    # 启用default连接配置
nmcli con add help  # 查看帮助
nmcli connection modify dormitory ipv4.dns "114.114.114.114 8.8.8.8"    # 设置dormitory网络的dns地址
nmcli connection add type ethernet con-name home ifname ens33   # 创建名为home的动态连接配置文件
nmcli -p conn show ens33    # -p添加标题和分段
```
### nmcli创建网络会话
通过nmcli命令创建一个网络会话，运行该命令后，系统会在网络配置目录生成一个网卡文件
```bash
nmcli connection add con-name dormitory ifname ens33 autoconnect no type thernet ip4 192.168.1.2/24 gw4 192.168.1.1
#  同时指定IPv6地址
nmcli connection add type ethernet con-name test-lab ifname ens33 ip4 192.168.100.2/24 gw4 192.168.100.1 ip6 
abbe::cafe gw6 2001:db8::1
# 设置DNS服务器地址
nmcli connection mod test-lab ipv4.dns "114.114.114.114 8.8.8.8"
nmcli connection mod test-lab ipv6.dns "001:4860:4860::8888 2001:4860:4860::8844"
# 添加DNS服务器地址
nmcli conn mod test-lab +ipv4.dns 192.168.100.1
```
.png
该配置默认是不激活的，需要手动启用该会话
```bash
nmcli conn up dormitory
```
### nmcli 编辑器
通过nmcli connection edit打开编辑

```bash
nmcli connection edit type ethernet con-name company
nmcli> set ipv4.method auto
nmcli> save     # 保存修改
nmcli> quit     # 退出编辑器
####################################
nmcli connection edit type ethernet con-name company
nmcli>set ipv4.addresses 192.168.3.2/24 # 设置静态IP地址
nmcli>save temporary    # 临时保存，（暂时存储到内存中），系统重启后配置消失
nmcli>quit
```
参考链接：https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_networkmanager_command_line_tool_nmcli


## netplan


## ifconfig
> 该命令是net-tools工具包的一个查看网络信息的命令

## route
> 该命令是net-tools工具的一个查看网络路由信息的命令
**选项**
```bash
-A：设置地址类型
-C：打印将Linux核心的路由缓存
-v：详细信息模式
-n：执行DNS反向查找，直接显示数字形式的IP地址
-e：netstat哥特式显示路由表
-net：到一个网络的路由表
-host：到一个主机的路由表
```
**参数**
```bash
add：添加路由
del：删除路由
target：目的网络或目标主机
gw：设置默认网关
mss：设置TCP最大区块长度（MSS），单位MB
window：指定通过路由表的TCP连接的TCP窗口大小
dev：指定网络接口
```
**示例**
> 格式：
> route [add|del] [-net|-host|default] target [netmask num] [gw num] [dev if]

**\***一些系统的内核配置已经包含了路由功能，但默认并没有在系统启动时启用此功能，开启Linux的路由功能可以通过调整内核的网络参数来实现，要配置和调整内核参数可以使用`sysctl`命令。
> sysctl -w net.ipv4.ip_forward=1


```bash
route       # 查看系统当前路由
route -n       # -n直接显示数字形式的IP地址
```
> - U，up表示此路由当前为启动状态
> - H，host，表示此网关为一个主机
> - G，Gateway，表示此网关为一个路由器
> - R，Reinstate Route使用动态路由重新初始化的路由
> - D，Modified此路由是由路由守护程序或导向动态修改
> - ！，表示此路由当前为关闭状态
```bash
route add -net 192.168.128.0 netmask 255.255.255.0 dev ens33    # 在ens33网络接口增加一条到192.168.128.0的路由
route add -net 192.168.128.0 netmask 255.255.255.0 reject       # 增加一条屏蔽的路由
route del -net 192.168.128.0 netmask 255.255.255.0  # 删除一条路由，删除的时候可以不用谢网关
route del -net 192.168.128.0 netmask 255.255.255.0 reject   # 删除一条屏蔽路由
route del default gw 192.168.128.11 # 删除设置的默认网关
route add default gw 192.168.128.12 # 添加默认网关
route add -net 192.168.128.0 netmask 255.255.255.0 gw 192.168.128.1     # 添加一条路由，并设置该网段全部经过该网关
route add -host 192.168.128.11 dev ens38    # 添加一个主机到
```
> getty、
> smbd(samba daemon)、
> telnet，远程登录主机和管理
> httpd、
> ip、网络配置工具
> ifconfig、配置和显示系统网卡的网络参数
> mesg、设置当前终端的写权限
> nc、用于设置路由器的强大网络管理工具，（行内称为网络工具中的瑞士军刀），全称netcat，
> netconf、
> netconfig、
> netstat、打印网络状态信息
> ping、使用ICMP传输协议测试主机之间网络的连通性
> pppstats、
> samba、
> setserial、
> shapecfg(shaper configuration)、
> smbd(samba daemon)、
> statserial(status ofserial port)、
> talk、终端聊天工具
> tcpdump、sniffer嗅探抓包工具
> testparm(test parameter)、
> traceroute、追踪数据包再网络上的传输时的全部路径
> tty(teletypewriter)、显示连接到当前标准输入的终端设备文件名
> uuname、
> wall(write all)、向系统当前所有打开的终端广播信息
> write、向指定登录用户终端上发送信息
> ytalk、
> arpwatch、
> apachectl、
> smbclient(samba client)、
> pppsetup

# 文件传输
> ftp、
> scp

# 备份解压缩
> ar、
> tar
> bunzip2、
> bzip2、
> bzip2recover、
> cpio、
> gunzip、
> gzexe、
> dump、
> gzip、
> unzip、
> zip、
> zipinfo

# 文件管理
diff、diffstat、file、find、git、ln、locate、mv、rm、split、lsattr、mattrib、mcopy、mdel、mktemp、mmove、mren、mshowfat、mtools、od、patch、rc、tee、touch、umask、whereis、which、cat、chattr、chgrp、chmod、chown、cksum、cmp、cp、cut

# 磁盘管理/维护
> df，显示磁盘的相关信息，用于显示磁盘分区上的可使用磁盘空间。默认显示单位KB。
> du，对文件和目录磁盘使用空间的查看
> eject、用来退出抽取式设备，若设备已挂载，则eject命令可将该设备卸载退出
> mcd、
> mdeltree、
> mdu、
> mkdir、
> mlabel、
> mmd、
> mmount、
> mrd、
> mzip、
> rmt、
> stat、显示文件的状态信息
> tree、树状列出目录的内容
> umount，用于卸载已挂载的文件系统/设备
> badblocks、查找磁盘中损坏的区块
> cfdisk、
> dd、复制文件并对源文件的内容进行转换和格式化处理
> e2fsck、用于检查第二扩展文件系统的完整性
> ext2ed、
> fdisk、查看磁盘使用情况/信息，用于磁盘分区
> fsck.ext2、
> fsck、检查并视图修复文件系统中的错误
> fsck.minix、
> hdparm、显示与设定硬盘的参数
> losetup、设定与控制循环（loop）设备
> mbadblocks、
> mformat、
> mkdosfs、
> mke2fs、 创建磁盘分区上的"ext2/ext3"文件系统
> mkfs.ext2、
> mkfs、用于在设备上创建linux文件系统
> mkfs.minix、
> mkfs.msdos、
> mkinitrd、建立要载入ramdisk的映像文件，以供Linux开机时载入ramdisk
> mkisofs、用来将指定的目录与文件做成ISO 9660格式的镜像文件，以供刻录光盘
> mkswap、用于在一个文件或者设备上建立交换分区。在加建立完之后使用swapon命令开始使用这个交换分区。最后一个选择性参数制定了交换区的大小
> mpartition、
> sfdisk、
> swapoff、关闭指定的交换空间（包括交换文件和交换分区）
> swapon、用于激活交换空间
> sync，用于强制被改变的内容立刻写入磁盘，更新super block信息

# 系统设置/管理
alias、bind、chkconfig、chroot、clock、crontab、declare、depmod、dircolors、dmesg、enable、eval、export、grpconv、grpunconv、hwclock、insmod、kbdconfig、lsmod、minfo、modinfo、modprobe、passwd、pwconv、pwunconv、rmmod、rpm、set、sndconfig、ulimit、unalias、unset

adduser、chfn、chsh、date、exit、finger、free、groupdel、groupmod、halt、id、kill、last、lastb、login、logname、logout、logrotate、newgrp、nice、ps、pstree、reboot、renice、rlogin、rsh、screen、shutdown、su、sudo、suspend、tload、top、uname、useradd、userdel、usermod、vlock、w、who、whoami

# 文本操作
awk、col、colrm、comm、csplit、ed、egrep、ex、fgrep、fmt、fold、grep、join、look、mtype、pico、rgrep、sed、sort、tr、uniq、vi、wc

# 设备管理
dumpkeys、loadkeys、MAKEDEV、rdev、setleds

# 邮件

# 其他命令
yes
