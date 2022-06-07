---
title: SSH原理与应用
tags:
  - SSH
  - openssh
---

## SSH原理

## SSH安装与配置

## SSH基本用法

## SSH口令登录与公钥登录
### 口令登录

### 公钥认证登录
```shell
ssh-keygen -t rsa -C "your_mail@example.com" -f github_rsa
```


#### Windows与Linux之间实现免密登录

    # Windows客户端：
    ssh-keygen
    ...id_rsa_ubuntu
    ...
    ...
    scp keygen_file.pub linux@addree:~/.ssh/
    # Linux服务器:
    cat .ssh/id_rsa_ubuntu.pub > authorized_keys
    
    # Windows客户端:
    notepad C:\Users\Administrator\.ssh\config
    Host 192.168.73.30
    HostName 192.168.73.30
    User echo
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_ubuntu

## SSH端口转发

### 本地端口转发

**以MySQL为例**
> MySQL服务器不允许MySQL远程访问
> 在客户端建立与MySQL服务器的ssh端口转发连接

**三种格式（任意一种写法都行）：**

    ssh -L 127.0.0.1:3306:127.0.0.1:3306 echo@192.168.73.30

本地网卡地址可以省略：

    ssh -L 3306:127.0.0.1:3306 root@192.168.73.30

若两台机器的登录用户名相同，可以省略登录用户名：

    ssh -L 3306:127.0.0.1:3306 192.168.73.30
    
远程命令

    ssh user@ip 'command'
    
    ssh root@192.168.10.12 'uname -a'
    tar -cz test | ssh root@192.168.10.12 'tar -xz'

*【注意！】客户端用于建立映射的端口不能被占用*

**【测试】**

> 在MySQL服务器查看数据库如下：

.png

> 在另一台机器上可以登录到MySQL服务器：

.png

> 关闭端口转发连接，输入密码无法登录：

.png

**注意：** 如果你一直访问着远程数据库，那么ssh端口转发连接无法关闭

-----

**Apache/Nginx的端口转发：**

    ssh -L 192.168.73.10:80:127.0.0.1:80 echo@192.168.73.30

> 把本地的某个端口转发到远程服务器上：

**Nginx服务器**
.png

**客户端**
.png


ssh -f -N -L 192.168.73.10:80:192.168.73.30:80 echo@192.168.73.30

.png
.png


### 远程转发

> 远程转发是通过端口映射间内网端口映射到公网端口上
> 使用远程转发需要先修改配置文件
> 允许端口转发和允许端口绑定

    vim /etc/ssh/sshd_config
    AllowTcpForwarding yes
    GatewayPorts yes


#### **例子：**

CentOS 7：192.168.110.20
RHEL 7: 192.168.73.10
Ubuntu 18.04(Nginx Server): 192.168.73.30

-------------------------------------------------------

Ubuntu:
将本地80端口映射到rhel 7上

    ssh -R 0.0.0.0:80:localhost:80 root@192.168.73.10
rhel 7:
将本地端口映射到CentOS 7上

    ssh -R 0.0.0.0:80:localhost:80 root@192.168.110.20

### 动态转发

ssh -D [本地地址]:本地端口号 远程用户@远程地址

**端口转发维持**
动态转发 ssh -D 1080 user@server1_ip
维持链接 ssh -o ServerAliveInterval=time user@ip

**文档**[https://www.ssh.com/academy/ssh/tunneling/example](https://www.ssh.com/academy/ssh/tunneling/example)

## 中间人攻击

## 配置文件参数解释
