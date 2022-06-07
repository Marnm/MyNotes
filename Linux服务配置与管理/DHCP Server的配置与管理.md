---
title: DHCP Server的配置与管理
tags:
  - DHCP
---

## 安装配置
```bash
#rhel/centos
yum install -y dhcp
```

> `/etc/dhcp/dhcpd.conf`DHCP主配置文件目录
> `/usr/share/doc/dhcp*/dhcpd.conf.sample`DHCP配置模板
> `/var/lib/dhcpd/dhcpd.leases` dhcp租约数据库文件，保存一系列的租约声明，包含客户端的主机名、MAC地址、分配的IP地址等。

### 配置示例
```bash
vim /etc/dhcp/dhcpd.conf
########################
# 声明一个段：
subnet 192.168.1.0 netmask 255.255.255.0 {
  # 地址池IP地址范围
  range 192.168.1.10 192.168.1.50;
  # 客户端指定DNS服务器地址：
  #option domain-name-servers ns1.internal.example.org; 
  # 为客户端指定所属的域：
  #option domain-name "internal.example.org";
  # 默认网关/路由
  option routers 192.168.1.1;
  # 广播地址
  option broadcast-address 192.168.1.255;
  # 定义IP默认租约时间，（单位：s/秒）：
  default-lease-time 600;  
  # 定义IP最大租约时长，（单位：s/秒）：
  max-lease-time 7200;
  # 定义日志类型
  log-facility local7;
  # IP地址绑定：
  # 指定一个IP地址给指定的客户端，指定的地址不可以不在地址池范围内
  host localhost {
    # 客户端的MAC地址
    hardware ethernet 00:0c:29:10:d9:56;
    # 指定的IP给客户端
    fixed-address 192.168.1.11;
  }
}
```
