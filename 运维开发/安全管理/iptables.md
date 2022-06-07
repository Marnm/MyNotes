---
title: iptables
tags:
  - iptables
---

# iptables
## iptables
### iptables的四表五链

> iptables的结构是由talbes组成，而tables是由链组成，链又是由具体的规则组成。
> 因此我们在编写iptables的规则时，要先指定表，再指定链。
> tables的作用是区分不同功能的规则，并且存储这些规则。

| 表名 | 功能 | 优先级（低-->高） | 内建链 |
| --- | --- | --- | --- |
| filter | 负责数据包的过滤功能，防火墙；内核模块：iptables_filter | 4 | INPUT、 OUTPUT、 FORWARD |
| nat | 网络地址转换功能；内核模块：iptable_nat | 3 | PREROUTING 、POSTROUTING、 OUTPUT |
| mangle | 拆解报文，做出修改，并重新组装成数据包的功能，iptable_mangle | 2 | PREROUTING、 OUTPUT、 FORWARD、 INPUT、 POSTROUTING |
| raw | 数据跟踪，关闭nat表上启用的连接追踪机制；table_raw | 1 | PREROUTING 、OUTPUT |

*规则链*

| 链类型 | 作用域 |
| --- | --- |
| PREROUTING | 数据包进入路由表之前 |
| INPUT | 通过路由表后目的地为本机 |
| OUTPUT | 由本机产生，向外转发 |
| FORWARD | 通过路由表后，目的地不为本机 |
| POSTROUTING | 发送到网卡接口之前 |

**链表的优先级顺序：**
- 表间的优先顺序：raw > mangle > nat > filter
- 链间的优先顺序：
    - 入站数据：PREROUTING > INPUT
    - 出站数据：OUTPUT > POSTROUTING
    - 转发数据：PREROUTING > FORWARD > POSTROUTING
- 链内的匹配顺序：
    - 自顶向下按顺序依次进行检查，找到匹配的规则即停止（LOG选项表示记录相关日志）
    - 若在该链内找不到相匹配的规则，则按该链的默认策略处理（未修改的状况下，默认策略为允许）

**匹配条件**
- 源地址Source IP
- 目标地址Destination IP

**处理动作**
- ACCEPT：允许数据包通过
- DROP：直接丢弃数据包，不给任何回应信息
- REJECT：拒绝数据包通过，必要时会给数据发送端一个响应信息，客户端刚请求就会收到拒绝的信息。
- SNAT：源地址转换，解决内网用户用同一个公网地址上网的问题
- MASQUERADE：是SNAT的一种特殊形式，适用于动态、临时会变的IP上
- DNAT：目标地址转换
- REDIRECT：在本机做端口映射
- LOG：在`/var/log/messages`文件中记录日志信息，然后将数据包传递给下一条规则，也就是说除了记录以外不对数据包做任何其他操作，仍然让下一条规则去匹配。

**选项**

    -t，--table：对指定的表table进行操作，table必须是raw，nat，filter，mangle中的一个。如果不指定-t，默认filter表
    
    # 通用匹配：源地址，目标地址的匹配
    -p：指定要匹配的协议类型
    -s，--souce [!] address[/mask]：把指定的 一个/一组 地址作为源地址，按此规则进行过滤。当后面没有maks时，address是一个地址，比如192.168.1.1；当mask指定时，可以表示一组范围内的地址，比如192.168.1.0/255.255.255.0
    -d，--destination [!] address[/mask]：指定目的地址，按此进行过滤
    -i，--in-interface [!] <网络接口名称>：指定数据包的来自的网络接口，比如最常见的eht0、ens33。该选项只对INPUT、FORWARD、PREROUTING这三个链起作用
    -o，--out-interface [!] <网络接口名称>：指定数据包出去的网络接口。只对OUTPUT、FORWARD、POSTROUTING三个链起作用。
    
    # 查看管理命令
    -L， --list [chain] ：列出链chain上面的所有规则，如果没有指定链，列出表的所有链所有规则。
    
    # 规则管理命令
    -A，--append chain rule-specification：在指定链chain的末尾插入指定的规则，这条规则会被放到最后，最后才执行。
    -I，--insert chain [rulenum] rule-specification：在链chain中的指定位置插入一条或多条规则。如果指定的规则号是1，则在链的头部插入。这也是默认的情况，如果没有指定规则号。
    -D，--delete chain rule-specification/--delete chain rulenum ：在指定的链chain中删除一个或多个指定规则
    -R num，replays：替换/修改第几条规则
    
    # 链管理命令
    -P，--policy chain target：为指定的链chain设置策略target。注意，只有内置的链才允许有策略，用户自定义的是不允许的。
    -F，--flush [cahin]：清空指定链chain上面的所有规则。如果没有指定链，则默认清空所有链上的规则
    -N，--new-chain chain：用指定的名字创建一个新的链
    -X，--delete-chain [chain]：删除指定的链，这个链必须没有被其他任何规则引用，而且这条上必须没有任何规则。如果没有指定链名，默认删除所有用户链
    -E，--rename-chain old-chain new-chain：重命名链。这并不会对内部链造成任何影响
    -Z，--zero [chain]：把指定链，或表中的所有链的所有计数器清零。
    
    -j，--jump target <指定目标>：及满足某条件时该执行什么样的动作。target可以是内置的目标，比如ACCEPT，也可以是用户自定义的链。
    
**格式**

    iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网>  --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作
    



## iptables-restore
## iptables-save
## iptstate
