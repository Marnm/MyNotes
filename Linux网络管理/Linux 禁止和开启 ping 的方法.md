---
title: Linux 禁止和开启 ping 的方法
tags: []
---

## ping 原理

网络上的机器都有唯一确定的IP地址，我们给目标IP地址发送一个数据包，对方就要返回一个同样大小的数据包，根据返回的数据包我们可以确定目标主机的存在，可以初步判断目标主机的操作系统等。

ping命令一般用于检测网络通与不通，也叫时延（网络一端传送到另一端需要的时间），其值越大，速度越慢 ping (Packet Internet Grope)，因特网包探索器，用于测试网络连接量的程序。

ping发送一个ICMP回声请求消息给目的地并报告是否收到所希望的ICMP回声应答。它是用来检查网络是否通畅或者网络连接速度的命令。

## ping的工作流程

1、在同一网段内

	ping

主机A--------------------->主机B  
ICMP请求包  
- 在本机(主机A)查找ARP缓存表查找主机B的IP与其对应的MAC
- 没有找到主机B的IP与其MAC的映射关系，则发送一个arp (根据IP地址获取物理地址的一个TCP/IP协议) 请求广播
- 主机B接收到arp请求包后，回复一个arp应答包(里面包含本机MAC)

主机A<----------------------主机B  
ICMP应答包


2、不在同一网段

在主机A上运行“ping主机C(不在同一网段)”后，开始跟上面一样，到了怎样得到MAC地址时，IP协议通过计算发现C机与自己不在同一网段内，就直接交给路由器处理，也就是将路由的MAC取过来，至于怎样得到路由的MAC，跟上面一样，**先在ARP缓存表找，找不到就广播包。路由得到这个数据帧后，再跟主机C进行联系，如果找不到，就向主机A返回一个超时的信息**。

Linux默认是允许Ping响应的，系统是否允许Ping由2个因素决定的：A、内核参数，B、防火墙，需要2个因素同时允许才能允许Ping，2个因素有任意一个禁Ping就无法Ping。
 

具体的配置方法
 
内核参数设置

       
1、允许PING设置
       
A. 临时允许PING操作的命令为：`echo 0 >/proc/sys/net/ipv4/icmp_echo_ignore_all`
         
B. 永久允许PING配置方法。
  在 /etc/sysctl.conf 中增加一行 
  net.ipv4.icmp_echo_ignore_all=0
  如果已经有net.ipv4.icmp_echo_ignore_all这一行了，直接修改=号后面的值即可的（0表示允许，1表示禁止）。
   修改完成后执行sysctl -p使新配置生效。
        
       
2、禁止Ping设置     
         
A.临时禁止PING的命令为：echo 1 >/proc/sys/net/ipv4/icmp_echo_ignore_all     
       
B.永久禁止PING配置方法。
   /etc/sysctl.conf 中增加一行
   net.ipv4.icmp_echo_ignore_all=1
   如果已经有net.ipv4.icmp_echo_ignore_all这一行了，直接修改=号后面的值即可的。（0表示允许，1表示禁止）
   修改完成后执行sysctl -p使新配置生效。
         
防火墙设置

注：此处的方法的前提是内核配置是默认值，也就是没有禁止ping
这里以iptables防火墙为例，其他防火墙操作方法可参考防火墙的官方文档。

     
1、允许PING设置      

     iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
     iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
      
 
     
2、禁止PING设置
     
     iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
     iptables -A OUTPUT -p icmp --icmp-type echo-reply -j DROP
     
     iptables -A INPUT -p icmp --icmp-type 8 -s 0/0 -j DROP
     
     --icmp-type 8 echo request——回显请求（ping请求）

     0/0 所有 IP
     
     iptables -A INPUT -p icmp --icmp-type 8 -j DROP
     iptables -A INPUT -p icmp --icmp-type 0 -j DROP
     
     0  Echo Reply——回显应答（Ping应答）
     8  Echo request——回显请求（Ping请求）
