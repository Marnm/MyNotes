---
title: DNS Server的配置与管理
tags:
  - DNS
---

## 配置实现
> 修改/etc/named.conf配置文件

```
options {
    listen-on port 53 { any; };
    listen-on-v6 port 53 { any; };
    ...
    allow-query { any; };
    ...
    recursion yes;  # 是否启用递归查询
    
    dnssec-enable yes;
    dnssec-validation yes;
    dnssec-lookaside auto;
}

...
##------------------------------------------------
## 该配置建议添加到/etc//etc/named.rfc1912.zones
# 新增如下
zone "abc.cn" IN {
    type master;
    file "abc.cn.zone";
};

zone "73.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.73.arpa";
};
```

**添加正反向解析**
```shell
# 复制正向解析配置模板
cp -p /var/named/named.localhost /var/named/abc.cn.zone
# 复制反向解析配置模板
cp -p /var/named/named.empty /var/named/192.168.73.arpa
##############################################
# 添加正向解析配置
vim /var/named/abc.cn.zone
```

```ini
$TTL 1D
abc.cn. IN SOA  dns.abc.cn. root.abc.cn. (
                    0   ; serial
                    1D  ; refresh
                    1H  ; retry
                    1W  ; expire
                    3H )    ; minimum
abc.cn. NS  dns.abc.cn.
dns.abc.cn. A   192.168.73.5
www.abc.cn. A   192.168.73.5
www1.abc.cn.    CNAME   www.abc.cn.

# 添加反向解析配置
vim /var/named/192.168.73.arpa

$TTL 3H
@   IN SOA  abc.cn. root.abc.cn. (
                    0   ; serial
                    1D  ; refresh
                    1H  ; retry
                    1W  ; expire
                    3H )    ; minimum
    NS  ns.abc.cn.
ns  A   192.168.73.5
5   PTR ns.abc.cn.
5   PTR www.abc.cn.

```
### 配置DNS转发
```ini
vim /etc/named.conf
################
options{
    forward first;
    # 多个DNS用;隔开
    forwarders {
        192.168.73.2;
        114.114.114.114;
        8.8.8.8;
        8.8.4.4;
        1.1.1.1;
    };
}
```


## 配置主从DNS服务
> 主从机器进行时间同步


```bash
ntpd
```
**修改主DNS服务器配置**
```
vim /etc/named.conf
###################
zone "abc.cn" IN {
    type master;
    file "abc.cn.zone";
    allow-transfer { 192.168.73/24; };  # 添加该参数
};

zone "73.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.73.arpa";
    allow-transfer { 192.168.73/24; };  # 添加该参数
};
```
**修改从DNS服务器配置**

```ini
vim /etc/named.conf
options{
    listen-on port 53 { any; };
    listen-on-v6 port 53 { any; };
    ...
    allow-query     { any; };
    
    recursion yes;
    ...
    ...
    
    dnssec-validation no;
}

zone "abc.cn" IN {
    type slave;
    file "slaves/abc.cn.zone.file";
    masters { 192.168.73.5; };
};

zone "73.168.192.in-addr.arpa" IN {
    type slave;
    file "slaves/192.168.73.arpa.file";
    masters { 192.168.73.5; };
};

```
> 重启从DNS服务器，
> 查看/var/named/slaves/目录下的配置文件
> 从DNS服务能够从主DNS复制配置文件过来

### DNS主从密钥认证
> ntpd主从时间同步

**主DNS服务器配置**
```bash
# 生成密钥
cd /var/named/chroot/etc/
dnssec-keygen -a hmac-md5 -b 128 -n HOST 密钥名称
# 查看密钥的密码
cat k****.+**+***.key
cat Kdnskey.+157+23495.key
dnskey. IN KEY 512 3 157 Rg0mKvMMQ06YAoN0FcOCXQ==
复制密码串
########################################
# 修改主DNS服务器配置文件
vim /etc/named.conf
# 添加如下配置
key "dnskey" {
    algorithm hmac-md5;
    secret "粘贴生成的key密码串";
};

zone "abc.cn" IN {
    type master;
    file "abc.cn.zone";
    allow-transfer { key 密钥名称 };
};

zone "73.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.73.apra";
    allow-transfer { key 密钥名称 };
};
```

**从DNS服务器配置**
```bash
# 修改dns配置文件
vim /etc/named.conf
# 添加如下配置
key "dnskey" {
    algorithm hmac-md5;
    secret "粘贴生成的密钥到该处";
};

zone "abc.cn" IN {
    type slave;
    file "slaves/abc.cn.zone.file";
    masters { 192.168.73.5 key dnskey; };
};

zone "73.168.192.in-addr.arpa" IN {
    type slave;
    file "slaves/192.168.73.arpa.file";
    masters { 192.168.73.5 key dnskey; };
};

```




## 常见问题解决
**启动状态报network unreachable问题**
```
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving './DNSKEY/IN': 2001:503:ba3e::2:30#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving './NS/IN': 2001:503:ba3e::2:30#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'dlv.isc.org/DNSKEY/IN': 2001:500:48::1#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'dlv.isc.org/DNSKEY/IN': 2001:4f8:0:2::19#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns.isc.afilias-nst.info/A/IN': 2001:500:2f::f#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns.isc.afilias-nst.info/AAAA/IN': 2001:500:2f::f#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns.isc.afilias-nst.info/A/IN': 2001:500:1::803f:235#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns.isc.afilias-nst.info/AAAA/IN': 2001:500:1::803f:235#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns.isc.afilias-nst.info/A/IN': 2001:503:c27::2:30#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns.isc.afilias-nst.info/AAAA/IN': 2001:503:c27::2:30#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns.isc.afilias-nst.info/A/IN': 2001:500:1a::1#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'dlv.isc.org/DNSKEY/IN': 2001:4f8:0:2::20#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'dlv.isc.org/DNSKEY/IN': 2001:500:60::29#53
Oct 23 11:39:03 server named[1585]: error (network unreachable) resolving 'ns1.isc.ultradns.net/A/IN': 2001:7fd::1#53
```
> 配置递归时，转发数据优先IPv6迭代

修改`/etc/sysconfig/named`
```
# 添加OPTIONS
#/etc/sysconfig/named 是bind守护进程启动时传递参数的配置文件，

OPTIONS="-4"
# BIND named process options
# ~~~~~~~~~~~~~~~~~~~~~~~~~~
```
参考：[https://www.cnblogs.com/zy09/p/14596993.html](https://www.cnblogs.com/zy09/p/14596993.html)

**配置主从DNS，客户端无法读取从DNS配置**
> 修改从DNS服务器配置文件
```bash
vim /etc/named.conf
dnssec-enable yes;  
dnssec-validation no;   #关闭验证
```



## DNS配置解析
> An example of a simple resource record (RR):
    > example.com.    86400    IN    A    192.0.2.1

example.com, 资源记录
86400,  time to live(TTL)
IN  "the Internet system", 指明RR的的类别
A   指明RR的类型，a host address
192.0.2.1   指明RR包含的IP地址


> /etc/named.conf，bind的主要配置文件
> /var/named/，主要配置文件的辅助目录

SOA ，Start of Authority Record，起始授权记录，每个区在区的开始都包含一个
NS，Name Server，域名服务器记录，用来指定该域名由哪个DNS服务器来进行解析，每个区至少包含一个NS
A，地址资源记录，把FQDN（完全合格域名，或完整的计算机名）映射到IP地址，该记录能够解析FQDN域名对应的IP地址。
PTR，指针记录，把IP地址映射到FQDN，用于反向查询，通过IP地址找到域名。
CNAME，别名记录，创建特定的FQDN别名
MX，邮件交换记录，指定邮件交换服务器。


**/etc/named.conf常用配置选项**

| Option | Description |
| --- | --- |
| allow-query |  |
| allow-query-cache |  |
| blackhole |  |
| directory |  |
| disable-empty-zone |  |
| dnssec-enable |  |
| dnssec-validation |  |
| empty-zones-enable |  |
| forwarders |  |
| forward |  |
| listen-on |  |
| listen-on-v6 |  |
| max-cache-size |  |
| notify |  |
| pid-file |  |
| recursion |  |
| statistics-file |  |
