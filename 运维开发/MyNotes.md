---
title: MyNotes
tags: []
---

### Redhat Enterprise Linux 7

**修改主机名**
```bash
hostnamectl set-hostname 主机名
````

```bash
# 修改网卡配置
vim /etc/sysconfig/network-scripts/ifcfg-ens33
# 示例：
# 网卡类型
TYPE=Ethernet
# 静态static，动态dhcp，无none
BOOTPROTO=static
# IP地址
IPADDR=192.168.1.2
# 前缀
#也可以使用掩码,如NETMASK=255.255.255.0
PREFIX=24
# 网关
GATEWAY=192.168.1.1
# DNS
DNS1=192.168.1.1
DNS2=8.8.8.8
# 网卡设备名称
DEVICE=ens33
# NetworkManager管理名称
NAME=ens33
```

----

### Ubuntu 20.04下配置静态IP地址
````bash
sudo vim /etc/netplan/**-network-manager-all.yaml
# 配置格式要严格按照YAML格式，否则配置后不会应用

# 示例
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
      ens33:
          dhcp4: no
          addresses:
              - 192.168.73.20/24
          gateway4: 192.168.73.2
          nameservers:
              addresses: [114.114.114.114,192.168.73.2]
````

**保存并应用更改**
```bash
netplan apply
```
参考链接：https://netplan.io/examples/




### 如何让VMware虚拟机走主机代理流量
**Ubuntu设置**
查看本机IP地址，我这里的虚拟机使用的是NAT模式，所以我使用NAT网卡的地址：
![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072103688.png)
Ubuntu设置里面使用本机的NAT地址，端口填你的梯子端口
![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072104906.png)
我这里的socks端口是7890
![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072104351.png)

#### 配置虚拟机终端走代理
即使前面配置了系统代理，但Ubuntu终端默认是不支持socks协议的，如果需要终端支持socks协议，需要用到第三方工具proxychains
```bash
sudo apt-get install proxychains
```
修改proxychains配置文件：
```bash
sudo vim /etc/proxychains.conf
```
添加socks协议，地址和端口
![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072105535.png)
查找libproxychains.so.3文件的路径，并复制该路径，等会儿用得到
```bash
find /usr/lib/ -name libproxychains.so.3 -print
/usr/lib/x86_64-linux-gnu/libproxychains.so.3
```
修改/usr/bin/proxychains文件
```bash
vim /usr/bin/proxychains
```
将export LD_PRELOAD=libproxychains.so.3改为刚才查找到的文件路劲即可
![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072106793.png)

使用方法：
如果需要命令走socks代理，需要在该命令前面运行`proxycahins'
```bash
proxychains snap install code
```
参考链接：https://zvv.me/z/2014.html

**Redhat系列代理配置**
```bash
vim /etc/profile
```
```shell
export PROXY_URL="http://192.168.73.1:7890/"
export http_proxy="$PROXY_URL"
export https_proxy="$PROXY_URL"
export ftp_proxy="$PROXY_URL"
export no_proxy="127.0.0.1, localhost"
```
更新配置
```bash
source /etc/profile
```
**注意：**如果需要取消代理，需要配置如下代码，并注释或删除上面的代码
```bash
unset http_proxy
unset https_proxy
unset ftp_proxy
unset no_proxy
```

### VIM内置主题舒适度
```bash
:colorscheme murphy
```
**`murphy`>`evening`>`darkblue`>`pablo`>`slate`**

### Linux下出现Read-only file system的解决办法
> 参考链接https://support.huawei.com/enterprise/zh/knowledge/EKB1100061604/

```bash
# 以读写方式重新挂载文件系统
mount -o remount rw /
```

### Linux添加语言编码格式
```bash
locale -a
sudo vim /etc/locale.gen    # 去掉需要的语言编码的注释
sudo locale-gen     # 更新系统编码信息
```

### 交换机原理
#### **二层交换机具体的工作流程：**
1. 当交换机从某个端口收到一个数据包，它先读取包头中的源MAC地址，这样它流知道源MAC地址的机器是连在哪个端口上的；
2. 再去读取包头中的目的MAC地址，并在地址表中查找响应的端口  
3. 如表中有与这目的的MAC地址对应的端口，把数据直接复制到这端口上
4. 如表中找不到相应的端口则把数据包广播到所有端口上，当目的季期间对源机器回应时，交换机又可以学习-目的MAC地址与哪个端口对应的，在下次传送数据时就不再需要对所有端口进行广播了。


#### 三层交换机
三层交换机的最重要的目的是加快大型局域网内部的数据交换，所具有的路由功能也是为这目的服务的，能够做到一次路由，多次转发。对于数据包转发等规律性的过程由硬件高速实现，而像路由信息更新、路由表维护、路由计算、路由确定等功能，由软件实现。三件交换技术就是二层交换技术+三层转发技术。


### 获取VIM自带的中文版手册
```bash
iconv -f gb2312 -t utf8 /usr/share/vim/vimxx//tutor/tutor.zh.euc -o ~/vimtutor.txt
```

### VMware
#### workstation虚拟机启用UEFI支持
> 在部分较新的系统或workstation版本是默认开启UEFI支持的，手动修改可以通过虚拟机文件或workstation设置来修改
> 创建一个新的虚拟机后，打开该虚拟机的存放目录
> 使用文本编辑打开***.vmx文件
> 在该文件中添加内容：firmware="efi"
> 
![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205072106354.png)

#### 修改BIOS启动等待时间
> 如果要实现启动界面延迟可选择的效果可添加：
> "bios.bootdelay = 5000"        #5000毫秒

### `/etc/`下主要配置文件解释
`/etc/hosts/`主机名到IP地址的映射关系文件
`/etc/resolv.conf`DNS服务器的配置文件，（设置DNS的）
`/etc/gateways`定义网络服务的端口，（ 设定路由器的）
`/etc/services`定义了网络服务的端口
```bash
touch /etc/nolgoin  # 禁止所有普通用户登录
```


----
```bash
dd if=/dev/zero of=/mnt/sw1;swapon /mnt/sw1
# dd用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换
# if=文件名；输入文件名，，缺省为标准输入。即指定源文件。<if=input file>
# of=文件名；输出文件名，缺省为标准输出。即指定目的文件。<of=output file>
# dev/zero：是一个输入设备，你可你用它来初始化文件。
# /mnt/sw1：Linux操作系统下的挂载目录下的交换分区目录
# swapon：开启交换分区
# mkswap：设置交换分区
# mkfs：建立 linux 文件系统在特定的分区上
```

### 如何识别系统是使用UEFI安装还是BIOS
> How to identify if the system is installed with UEFI or with Legacy only boot

```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```

---

### Linux退出状态码
> 按照惯例，一个成功结束的命令的退出状态码为0；
> 如果一个命令结束时有错误，退出状态码就是一个正数值

```bash
echo $?
```

| 状态码 | 描述 |
| --- | --- |
| 0 | 命令成功结束 |
| 1 | 一般性未知错误 |
| 2 | 不适合的shell命令 |
| 126 | 命令不可执行 |
| 127 | 没找到命令 |
| 128 | 无效的退出参数 |
| 128+x | 与Linux信号x相关的严重错误 |
| 130 | 通过Ctrl+C终止的命令 |
| 255 | 正常范围之外的退出状态码 |

**如下：**
```bash
# 退出状态码126表名用户没有执行命令的正确权限
[root@rhelsrv shell]# ./expr_calc.sh
-bash: ./expr_calc.sh: Permission denied
[root@rhelsrv shell]# echo $?
126
[root@rhelsrv shell]#

# 另一个会碰到的常见错误是给某个命令提供了无效参数
# 这会产生一般性的退出状态码1，表明在命令中发生了未知错误
[root@rhelsrv shell]# date %t
date: invalid date ‘%t’
[root@rhelsrv shell]# echo $?
1
[root@rhelsrv shell]#


```

**这种默认行为是可以手动改变的**
> `exit` 命令允许你在脚本结束时指定一个退出状态码

```bash
var2=100
var3=45
echo "-----------"
echo "保留4位小数"
var4=$(echo "scale=4; $var2/$var3" | bc)
echo 100/45=$var4

exit 255

[root@rhelsrv shell]# echo $?
255
```
**注意：**退出状态码最大只能是255
> 退出状态码被缩减到了0\~255的区间。
> 一个值的模就是被除后的余数。最终结果是**指定的数值除以256后得到的余数，该余数就是最后的状态退出码**
> 例如：指定的退出码为`exit 300`，则300 % 256，余数44，最后这个余数成了最后的状态退出码


### 获取vimtutor中文版
```bash
iconv -f gb2312 -t utf8 /usr/share/vim/vim**/tutor/tutor.zh.euc -o ~/vimtutor.txt
```

### 删除旧内核
`RHEL/CentOS`
```
rpm -qa | grep kernel

yum remove kernel-***
```

`Debian/Ubuntu`
```
dpkg --list | grep linux-image

apt purge linux-image-***
```


### Windows UWP应用设置代理

> Win 10 的 UWP 应用 (应用商店下载的 APP)，默认是不走代理的 (沙盒的网络隔离特性：禁止 APP 访问 localhost)，也就无法在中国使用像 Facebook 一样无法访问或者直接链接特别慢的 APP。

```ps
foreach ( $Obj in Get-ChildItem "HKCU:\Software\Classes\Local Settings\Software\Microsoft\Windows\CurrentVersion\AppContainer\Mappings\" -name ) { CheckNetIsolation.exe LoopbackExempt -a -n=$Oj }
```
或
```cmd
FOR /F "tokens=11 delims=\" %p IN ('REG QUERY "HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\CurrentVersion\AppContainer\Mappings"') DO CheckNetIsolation.exe LoopbackExempt -a -p=%p
```
参考：[https://noif.cc/2019/11/uwp-proxy/#%E4%B8%80checknetisolation](https://noif.cc/2019/11/uwp-proxy/#%E4%B8%80checknetisolation)

### 修改Linux命令提示符

```bash
echo $PS1
# 关键字记得加转义符`\`，如\$
PS1="[\u@\h \W]& "
```
