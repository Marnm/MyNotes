---
title: iproute2工具包
tags: []
---

# iproute2 工具

| net-tools | iproute2 |
|   ---     |    ---   |
| arp -na   | ip neigh |
| ifconfig  | ip link  |
| ifconfig -a | ip addr show |
| ifconfig --help | ip help |
| ifconfig -eth0 up | ip link set eth0 up |
| ipmaddr | ip maddr |
| iptunnel | ip tunnel |
| netstat | ss |
| netstat -i | ip -s link |
| netstat -g | ip maddr |
| netstat -l | ss -l |
| netstat -r | ip route |
| route add | ip route add |
| route del | ip route del |
| route -n | ip route show |
| vconfig | ip link |

| iproute2 | net-tools | Notes |
| --- | --- | --- |
| ip link show | ifconfig -a | 显示所有网卡|
| ip link set down/up eth0 | ifconfig eth0 up/down | 启动/关闭网卡 |
| ip addr add 192.168.0.10/24 dev eth0 | ifconfig eth0 192.168.0.10/24 | 在eth0上添加IP地址 |
| ip addr del 192.168.0.10/24 dev eth0 | ifconfig eth0 0 | 删除eth0的IP地址 |
| ip addr show dev eth0 | ifconfig eth0 | 查看eth0的网络配置信息 |
| ip -6 addr add fe80::f0b7:57ff:fe2f:5f0d/64 dev eth1 | ifconfig eth1 inet6 add fe80::f0b7:57ff:fe2f:5f0d/64 | 在eth1上添加IPv6地址 |
| ip -6 addr show dev eth0 | ifconfig eth0 | 查看IPv6地址 |
| ip link set dev eth0 address 02:42:20:d2:28:36 | ifconfig eth0 hw ether 02:42:20:d2:28:36 | 配置MAC地址 |
| ip route show | route -n | 查看路由表信息 |
| ip route add default via 192.168.0.1 dev eth0 | route add default gw 192.168.0.1 eth0 | 添加默认路由 |
| ip roue replace default via 192.168.0.1 dev ens33 | route del default gw 192.168.0.1 ens33 | 删除默认路由 |
| ip route add 10.24.32.0/24 via 192.168.0.1 dev ens33 | route add -net 10.24.32.0/24 gw 192.168.0.1 dev ens33 | 添加静态路由 |
| ip route del 192.168.10.0/24 | route del -net 192.168.10.0/24 | 删除静态路由 |
| ss | netstat | 查看tcp/udp的scoket监听信息 |
| arp -an | ip neigh | 查看ARP表 |
| bridge | brctl | 管理bridge地址和device |
