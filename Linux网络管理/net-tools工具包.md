---
title: net-tools工具包
tags: []
---

# net-tools工具包

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

## ifconfig

`eth0`、`eth0:1`、`eth0.1`之间的关系：  
**三者之间的关系对应于物理网卡、子网卡、虚拟VLAN网卡**  

* **物理网卡** 物理网卡这里指的是服务器上实际的网络接口设备
* **子网卡** 并不是实际上的网络接口设备，但是可以作为网络接口在系统中出现，如`eth0:1`、`eth0:2`。它们必须要依赖于物理网卡，虽然可以与物理网卡的网络接口同时在系统中存在并使用不同的IP地址，而且也拥有它们自己的网络配置文件。但是当它依赖的物理网卡不启动（Down）时，这些子网卡将不能工作
* **虚拟VLAN网卡** 这些虚拟的VLAN网卡也不是实际上的网络接口设备，也可以作为网络接口在系统中出现，但是与子网卡不同的是，它们没有自己的配置文件。他们只是通过将物理网加入不同的VLAN而生成的VLAN虚拟网卡。如果将物理网卡通过vconfig命令添加到多个VLAN中的话，就会有多个VLAN虚拟网卡出现，他们的信息以及相关的VLAN信息都是保存在/proc/net/vlan/config这个临时文件中，而没有独自的配置文件。  
**！** 注意，当需要启用VLAN虚拟网卡工作的时候，关联的物理网卡不能配置IP地址，并且，这些主物理网卡上的子网卡也不能被启用和配置IP地址信息。（除了802.1q配置下，在该配置下可以配置IP地址等信息）
