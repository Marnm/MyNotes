---
title: Linux网卡聚合及bond模式的原理
tags:
  - bonding
---

## Linux网卡聚合配置

### bonding
#### Redhat系列
添加2块物理网卡
修改网卡配置文件
> 默认网卡以ifcfg-ens33为例
```bash
cd /etc/sysconfig/network-scripts/
cp ifcfg-ens33 ifcfg-bond_static
cp ifcfg-ens33 ifcfg-ens38
echo > ifcfg-bond_static
```
修改物理网卡配置文件
```bash
vim ifcfg-ens33
TYPE=Ethernet
BOOTPROT=none
DEVICE=ens33
NAME=ens33
ONBOOT=yes
MASTER=bond_static  # 主网卡=bong_static
SLAVE=yes   # 从网卡=yes
```
----
```bash
vim ifcfg-ens38
TYPE=Ethernet
BOOTPROTO=none
DEVICE=ens38
NAME=ens38
ONBOOT=yes
MASTER=bond_static
SLAVE=yes
```
修改bond配置
```bash
vim ifcfg-bond_static
TYPE=Ethernet
BOOTPROTO=static
DEVICE=bond_static
NAME=bond_static
IPADDR=192.168.x.x
PREFIX=24
GATEWAY=192.168.X.X
DNS1=X.X.X.X
DNS2=X.X.X.X
ONBOOT=yes
BONDING_MASTER=yes  # 是否主网卡=yes
BONDING_OPTS="miimon=100 mode=6"    # MII监控时间间隔=100毫秒，绑定模式=6
```

#### Debian系列
> 以Ubuntu 20.04为例
**添加2块物理网卡**
修改配置文件：
> sudo vim /etc/netplan/01-network-manager-all.yaml
```yaml
version: 2
  renderer: NetworkManager
  ethernets:
          ens33:
                  dhcp4: no
          ens38:
                  dhcp4: no

  bonds:
      # bond名称
      bond-stat:
              interfaces: [ens33, ens38]        # 添加的2块物理网卡
              addresses: [192.168.73.20/24]     # 地址
              gateway4: 192.168.73.2        # 网关
              nameservers:      # DNS服务器
                addresses: [114.114.114.114, 192.168.73.2]
              routes:   # 路由
                      - to: default
                        via: 192.168.73.2
              parameters:   
                      mode: active-backup       # 绑定模式
                      mii-monitor-interval: 1       # MII监控时间间隔
```
应用配置
```bash
sudo netplan apply
```
 
 #### **bonding的7种模式：**
 - **0，(balance-rr)Round-robin-policy，平衡轮询策略**，
特点：传输数据包顺序是依次传输（即：第1个包走eth0，下一个包就走eth1….一直循环下去，直到最后一个传输完毕），此模式提供负载平衡和容错能力；但是我们知道如果一个连接或者会话的数据包从不同的接口发出的话，中途再经过不同的链路，在客户端很有可能会出现数据包无序到达的问题，而无序到达的数据包需要重新要求被发送，这样网络的吞吐量就会下降
- **1，(active-backup)Active-backup policy，主-备份策略，**
特点：只有一个设备处于活动状态，当一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得，从外面看来，bond的MAC地址是唯一的，以避免switch(交换机)发生混乱。此模式只提供了容错能力；由此可见此算法的优点是可以提供高网络连接的可用性，但是它的资源利用率较低，只有一个接口处于工作状态，在有 N 个网络接口的情况下，资源利用率为1/N
- **2，(balance-xor)XOR policy，平衡策略**，基于指定的传输HASH策略传输数据包。缺省的策略是(源MAC地址 XOR 目标MAC地址)%slave数量，其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力）
- **3，broadcast广播策略**，在每个slave接口上传输每个数据包，此模式提供了容错能力
- **4，(802.3ad)IEEE 802.3ad Dynamic link aggregation（IEEE 802.3ad动态链路聚合）**，创建一个聚合组，它们共享同样的的速率和双工设定。根据802.3ad规范将多个slave工作咋同一个激活的聚合体下，外出流量的slave选举是基于传输hash策略，该策略可以通过xmit_hash_policy选线从缺省的XOR策略改变到其他策略，需要注意的是，并不是所有的除数策略都是802.3ad适应的，尤其考虑到在802.3ad标准的43.2.4章节提及的包乱序问题。不同的是实现可能会有不同的适应性。
    - 必须要条件：
        - 条件1：ethtool支持获取每个slave的速率和双工设定
        - 条件2：switch交换机支持IEEE 802.3ad Dynamic link aggregation
        - 条件3：大多数switch需要经过特定配置才能支持802.3ad模式
- **5，(balance-tlb)Adaptive transmit load balancing适配器传输负载均衡**，不需要任何特别的switch支持的通道bonding，在每个slave上根据当前的负载（根据速度计算）分配外出流量，如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址
    - 该模式的必要条件：ethtool支持获取每个slave的速率
- **6，(balance-alb)Adaptive load balancing适配器适应性负载均衡**，该模式包含了balance=tlb模式，同时加上针对IPv4流量的接收负载均衡（receiveload balance，rlb），而且不需要任何switch的支持，接收负载均衡是通过ARP协商实现的，bonding驱动截获本机的ARP应答，并把源硬件地址改写为bood中某个salve的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。
来自服务器端的接收流量也会被均衡。当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。当ARP应答从对端到达时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。使用ARP协商进行负载均衡的一个问题是：每次广播 ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。这个问题可以通过给所有的对端发送更新（ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。当新的slave加入到bond中时，或者某个未激活的slave重新 激活时，接收流量也要重新分布。接收的负载被顺序地分布（roundrobin）在bond中最高速的slave上当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答 不会被switch(交换机)阻截。
    必要条件：条件1：ethtool支持获取每个slave的速率；条件2：底层驱动支持设置某个设备的硬件地址，从而使得总是有个slave(curr_active_slave)使用bond的硬件地址，同时保证每个 bond 中的slave都有一个唯一的硬件地址。如果curr_active_slave出故障，它的硬件地址将会被新选出来的 curr_active_slave接管其实mod=6与mod=0的区别：mod=6，先把eth0流量占满，再占eth1，….ethX；而mod=0的话，会发现2个口的流量都很稳定，基本一样的带宽。而mod=6，会发现第一个口流量很高，第2个口只占了小部分流量
    
----
参考链接：
https://www.jianshu.com/p/793b0138a3ef
https://chegva.com/2506.html
