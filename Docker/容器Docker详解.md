---
title: 容器Docker详解
tags: []
---

# 容器Docker详解

## 概述

**1.1 基本概念：**

Docker是一个开源的应用容器引擎，基于Go语言 并遵从Apache2.0协议开源。Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口（类似iPhone的app），更重要的的是容器性能开销极低。

**1.2 优势：**

Docker让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，便可以实现虚拟化。Docker改变了虚拟化的方式，使开发者可以直接将自己的成果放入Docker中进行管理。方便快捷已经是Docker的最大优势，过去需要用数天乃至数周的任务，在Docker容器的处理下，只需要数秒就能完成。

节省开支：  
一方面，云计算时代到来，使开发者不必为了追求效果而配置高额的硬件，Docker改变了高性能必然高价格的思维定势。Docker与云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题，也改变了虚拟化的方式。

**1.3 与传统VM特性对比：**

作为一种轻量级的虚拟化方式，Docker在运行应用上跟传统的虚拟机方式相比具有显著的优势：
Docker容器很快，启动和停止可以在数秒级实现，这相比传统的虚拟机方式要快得多。  
Docker容器对系统资源需求很少，一台主机上可以同时运行数千个Docker容器。  
Docker通过类似Git的操作来方便用户获取、分支和更新应用镜像，指令简明，学习成本较低。  
Docker通过Dockerfile配置文件来支持灵活的自动化创建和部署机制，提高工作效率。  
Docker容器除了运行其中的应用之外，基本不消耗额外的系统资源，保证应用性能的同时，尽量减小系统开销。  
Docker利用Linux系统上的多种防护机制实现了严格可靠的隔离。从1.3版本开始，Docker引入了安全选项和镜像签名机制，极大地提高了使用Docker的安全性。  

| 特性 | 容器 | 虚拟机 |
| ---  | --- | ---   |
| 启动速度 | 秒级  | 分钟级 |
| 硬盘使用 | 一般为MB | 一般给GB |
| 性能 | 接近原生 | 弱于原生 |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |
| 隔离性 | 安全隔离 | 安全隔离 |

**1.4 基础架构**

Docker使用客户端-服务器(C/S)架构模式，使用远程API来管理和创建Docker容器。

Docker容器通过Docker镜像来创建

容器与镜像的关系类似于面向对象编程中的对象与类。

| Docker | 面向对象 |
| ---   | ---     |
| 容器 | 对象 |
| 镜像 | 类 |

![](.png)

**1.5 Docker技术的基础：**

- namespace， 容器隔离的基础，保证A容器看不到B容器。6个命名空间：User, Mnt, Network, UTS, IPC, PID
- cgroups, 容器资源统计和隔离。主要用到的cgroups子系统：CPU, blkio, device, freezer, memory
- unionfs， 典型：aufs/overlayfs，分层镜像实现的基础

**1.6 Docker组件：**

- docker Client客户端————>向docker服务器进程发起请求，如创建、停止、销毁容器等操作
- docker Server服务器进程————>处理所有docker的请求，管理所有容器
- docker Registry镜像仓库————>镜像存放的中央仓库，可看作是存放二进制的scm


## 二、安装部署

**2.1 准备条件**

目前，CentOS仅发行版本中的内核支持Docker。  
Docker运行在CentOS 7上，要求系统为64位、系统内核版本为3.10以上。  
Docker运行在CentOS-6.5或更高的版本的CentOS上，要求系统为64位、系统内核版本2.6.32-431或者更高版本

**2.2 安装docker**
```shell
yum install docker -y
systemctl start docker
systemctl enable docker
```

**2.3 基本命令**
```dockerfile
docker search centos    # 搜索镜像
```

默认从国外拉取，速度很慢，可以使用daocloud配置加速
```shell
# 脚本写入
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://d6f11267.m.daocloud.io

echo "{\"registry-mirrors\": [\"http://d6f11267.m.daocloud.io\"]}"> /etc/docker/daemon.json
systemctl restart docker  # 重启生效
```

根据需求拉取镜像

	docker pull docker.io/ansible/centos7-ansible
	
拉取search到的全部镜像  

```shell
for i in `docker search centos | awk '!/NAME/{print $2}'`; do docker pull $i; done
```

查看本地镜像

	docker images

**2.4 命令整理**

容器操作：
```shell
docker create    # 创建一个容器但是不启动
docker run       # 创建一个容器并启动
docker stop      # 停止容器运行，发送信号SIGTERM
docker start     # 启动一个停止状态的容器
docker rm        # 删除一个容器
docker kill      # 发送信号给容器，默认SIGKILL
docker attach    # 连接（进入）到一个正在运行的容器
docker wait      # 阻塞一个容器，知道容器停止运行
```

获取容器信息：  
```shell
docker ps        # 显示状态为运行（up）的容器
docker ps -a     # 显示所有容器，包括运行中(up)的和退出的(exited）
docker inspect   # 深入容器内部获取容器所有信息
docker logs      # 查看容器日志(stdout/stderr)
docker events    # 得到docker服务器的实时事件
docker port      # 显示容器的端口映射
docker top       # 显示容器的进程信息
docker diff      # 显示容器文件系统的前后变化
```

导出容器：
```shell
docker cp # 从容器里向外拷贝文件或目录
docker export # 将容器整个文件系统导出一个tar包，不带layers、tag等信息
```

执行：

	docker exec  # 在容器里执行一个命令，可以执行bash进入交互式

镜像操作：
```shell
docker images    # 显示本地所有的镜像列表
docker import    # 从一个tar包创建一个镜像，往往和export结合使用
docker build     # 使用Dockerfile创建镜像（推荐）
docker commit    # 从容器创建镜像
docker rmi       # 删除一个镜像
docker load      # 从一个tar包创建一个镜像，和save配合使用
docker history   # 将一个镜像保存为一个tar，带layers和tag信息
docker history   # 显示生成一个镜像的历史命令
docker tag       # 为镜像起一个别名
```

镜像仓库（registry）操作：
```shell
docker login    # 登录到一个registry
docker search   # 从registry仓库搜索镜像
docker pull     # 从仓库下载镜像到本地
docker push     # 将一个镜像push到registry仓库中
```

**2.5 简单实践操作**

运行并进入容器操作：
```dockerfile
docker run -i -t docker.io/1832990/centos6.5 /bin/bash
```
- -t表示在新容器内指定一个伪终端或终端
- -i表示允许我们对容器内的(STDIN)进行交互
- -d表示将容器在后台运行；
- /bin/bash在容器内启动bash shell

所以当容器（container）启动之后，我们会获取到一个命令提示符：

在容器内我们安装mysql并设置开机自启动，将修改后的镜像提交：
```
docker ps -l  #查询容器ID
docker commit -m "功能" -a "用户信息" ID tag  #提交修改后的镜像
```

```shell
docker inspect ID    # 查看详细信息
docker push ID     # 上传docker镜像
```

**利用DockerFile创建镜像**   
使用docker  build，需要创建一个dockerfile文件，其中包含一组指令来告诉docker如何构建镜像
```shell
mkdir DockerFile
cd DockerFile
cat > Dockerfile <<EOF
FROM 603dd3515fcc
MAINTAINER Dcoker xuel
RUN yum install mysql mysql-server -y
RUN mkdir /etc/sysconfig/network
RUN /etc/init.d/mysqld start
EOF
```

	docker build -t "centos6.8:mysqld" 。

- -t指定repository与tag
- .指定Dockerfile的路径

注意一个镜像不能超过127层

此外，还可以利用ADD命令复制本地文件到镜像；  
用EXPOSE命令来向外部开放端口；  
用CMD命令来描述容器启动后运行的程序等。  
CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]

**2.6 Docker文件详解**

Dockerfile的指令根据作用可以分为两种，构建指令和设置指令。  
构建指令，用于构建image，其指定的操作不会再运行image的容器上执行；  
设置指令，用于设置image属性，其指定的操作将在运行image的容器中执行。

- FROM （指定基础image）  
构建指令，必须指定且需要在Dockerfile其他指令的前面。后续的指令版本都依赖于该指令的image。  
FROM指令指定的基础image可以是官方远程仓库中的，也可以位于本地仓库

该指令有两种格式：
```dockerfile
FROM <iamge>        # 指定基础image为该image的最后修改版本
FROM <image>:<tag>  # 指定基础为该image的一个tag版本
```

- MAINTAINER（用来指定镜像创建者信息）  
构建指令，用于将image的制作者相关的信息写入到image中。当我们对该image执行docker inspect命令时，输出中有相应的字段记录该信息。

	MAINTAINER <name>

- RUN(安装软件用)  
构建指令，RUN可以运行任何被基础image支持的命令。如基础image选择了ubuntu，那么软件管理部分只能使用ubuntu的命令。
```dockerfile
RUN <command> (the command is run in a shell - `/bin/sh -c`)
RUN ["executable", "param1", "param2" ...] (exec form)
```

- CMD(设置container启动时执行的操作)
设置指令，用于container启动时指定的操作。该操作可以是执行自定义脚本，也可以是执行系统命令。该指令只能在文件中红存在一次，如果有多个，则执行最后一条。
```dockerfile
CMD ["executablel", "param1", "param2"] (like an exec, this is the preferred form)
CMD command param1 para2 (as a shell)
```

ENTRYPOINT指定的是一个可执行脚本或者程序的路径，该指定的脚本或者程序将会以param1和param2作为参数执行。所以如果CMD指令使用上面的形式，那么Dockerfile中必须要有配套的ENTRYPOINT。当Dockerfile制定了ENTRYPOINT，那么使用下面的格式：
```dockerfile
CMD ["param1", "param2"] (as defualt paramenters to ENTRYPOINT)
```

- ENTRYPOINT，指定容器启动时执行的命令，可以多次设置，但是只有最后一个有效。

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"](like an exec, the preferred fomr)
ENTRYPOINT command param1 param2 (as a shell)
```

该指令的使用分为两种情况，一种是独自使用，另一种和CMD配合使用。  
当独自使用时，如果你还使用了CMD命令且CMD是一个完整的可执行的命令，那么CMD指令和ENTRYPOINT会相互覆盖只有最后一个CMD或者ENTRYPINT有效。
```dockerfile
# CMD指令将不会被执行，只有ENTRYPOINT指令被执行
CMD echo "Hello, World!"
ENTRYPOINT ls -l
```

另一种用法和CMD指令配合使用指定的ENTRYPIINT的默认参数，这时CMD指令不是一个完整的可执行命令，仅仅是参数部分；ENTRYPOINT指令只能使用JSON方式指定执行命令，而不能指定参数。
```dockerfile
FROM ubuntu
CMD ["-l"]
ENTRYPOINT ["/usr/bin/ls"]
```

- USER（设置container容器的用户）
设置指令，设置启动容器的用户，默认是root用户
```dockerfile
# 指定memcached的运行用户
ENTRYPOINT ["memcached"]
USER daemon
或
ENTRYPOINT ["memcached", "-u", "daemon"]
```

- EXPOSE（指定容器需要映射到宿主机器的端口）

设置指令，该指令会将容器中的端口映射成宿主机器中的某个端口。当你需要访问容器的时候，可以不是用容器的IP地址而是使用宿主机器的IP地址和映射后的端口。要完成整个操作需要两个步骤，首先在Dockerfile使用EXPOSE设置需要映射的容器端口，然后再运行容器的时候指定-p选项加上EXPOSE设置的端口，这样EXPOSE设置的端口号会被随机映射成宿主机器中的一个端口号。也可以指定需要映射到宿主机器的那个端口，这时要确保宿主机器上的端口号没有被使用。EXPOSE指令可以一次设置多个端口号，响应的运行容器的时候，可以配套的多次使用-p选项。
```dockerfile
# 映射一个端口
EXPOSE port1
# 相应的运行容器使用的命令（主机（端口）端口：容器端口）
docker run -p port image

# 映射多个端口
EXPOSE port1 port2 port3
# 相应的运行容器使用的命令
docker run -p port1 -p port2 -p port3 image
#还可以指定需要映射到宿主机器上的某个端口
docker run -p host_port1:port1 -p host_port2:port2 -p host_port3:port3 image
```
端口映射是docker比较重要的一个功能，原因在于我们每次运行容器的时候容器的IP地址不能指定而是在桥接网卡的地址范围内随机生成的。宿主机器的IP地址是固定的，我们可以将容器的端口映射到宿主机器上的一个端口，免去每次访问容器中的某个服务时都要查看容器的IP地址。对于一个运行的容器，可以使用docker port加上容器中需要映射的端口和容器的ID来查看该端口号在宿主机器上的映射端口。

- ENV（用于设置环境变量
构建指令，在image中设置一个环境变量。
```dockerfile
ENV <key> <value>
```

设置了后，后续的RUN命令都可以使用，container启动后，可以通过docker inspect查看这个环境变量，也可以通过在docker run --env key=value时设置或修改环境变量。
假如你安装了JAVA程序，需要设置JAVA_HOME，那么可以在Dockerfile中这样写：
```dockerfile
ENV JAVA_HOME /path/to/java/dirent
```

- ADD（从src复制文件到container的dest路径）

构建指令，所有拷贝到container中的文件和文件夹权限为0755，uid和gid为0；如果是一个目录，那么会将该目录下的所有文件添加到container中，不包括目录；如果文件是可识别的压缩格式，则docker会帮忙解压缩（注意压缩格式）；如果\<src>是文件且\<dest>中不使用斜杠结束，则会将\<dest>视为文件，\<src>的内容会写入\<dest>；如果\<src>是文件且\<dest>中使用斜杠结束，则会\<src>文件拷贝到\<dest>目录下。
```dockerfile
ADD <src> <dest>
```
`<src>`是相对被构建的源目录的相对路径，可以是文件或目录的路径，也可以是一个远程的文件url;  
`<dest>`是container中的绝对路径

- VOLUME（指定挂载点）

设置指令，使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用。我们知道容器使用的是AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。当容器中的应用有持久化数据的需求时可以在Dockerfile中使用该指令。

```dockerfile
FROM base
VOLUME ["/tmp/data"]
```

- WORKDIR（切换目录）  
设置指令，可以多次切换（相当于cd命令，对RUN,CMD,ENTRYPOINT生效
```dockerfile
# 在 /p1/p2下执行 vim a.txt
WORKDIR /p1 WORKDIR p2 RUN vim a.txt
```

**2.7 镜像导入导出**

```shell
docker save -o centos6.5.tar centos6.5 # 或
docker export f9c99092063c > centos6.5.tar
```

将本地镜像导入
```shell
docker load --input cnetos6.5.tar #或
docker load < centos6.5.tar
```

```shell
docker exec -it CONTAINER ID /bin/bash
```

## 三、存储

**3.1 数据盘**

docker的镜像使用一层一层文件组成的，docker的一些存储引擎可以处理怎么样存储这些文件。

```shell
docker inspect centos  # 查看容器详细信息
```
信息下方的Layers，就是centos的文件，这些东西都是只读的不能去修改，我们基于这个镜像去创建的镜像和容器也会共享这些文件层，而docker会在这些层上面去添加一个可读写的文件层。如果需要修改一些文件层里面的东西的话，docker会复制一份到这个可读写的文件层里面，如果删除容器的话，那么也会删除它对应的可读写的文件层的文件。如果有些数据你想一直保存的话，比如：web服务器上面的日志，数据库管理系统里面的数据，那么我们可以把这些数据放到data volumes数据盘里面。它上面的数据，即使把容器删掉，也还是会永久保留。创建容器的时候，我们可以去指定数据盘。其实就是去指定一个特定的目录。

```shell
docker run -i -t -v /mnt --name nginx docker.io/nginx /bin/bash
```
`-v`：指定挂载到容器内的目录  
使用docker insepect容器ID可以查看挂载目录对应于宿主机器的物理文件路径  
同样，我们可以使用将制定物理宿主机的目录挂载到容器的制定目录下：

将宿主机目录挂载到容器内：

```shell
docker run -d -p 80:80 --name nginx -v /webdata/wordpress:/usr/share/nginx/html docker.io/sergyzh/centos6-nginx
```
`-d`后台运行  
`--name`给运行的容器命名  
`-v`宿主机目录:容器目录 将宿主机目录挂载在容器内
`-p` 宿主机端口:容器监听端口 将容器内应用监听端口映射到物理宿主机的特定端口上

**3.2 数据容器**

可以创建一个数据容器，也就是再创建容器是指定这个容器的数据盘，然后让其他容器可以使用这个容器作为他们的数据盘，有点像继承了这个数据容器指定的数据盘作为数据盘。

首先创建一个数据容器命名为newnginx

```shell
docker create -v /mnt -it --name newnginx docker.io/nginx /bin/bash
```
利用此数据容器容器运行一个容器nginx1,在数据目录/mnt 下创建一个文件

```shell
docker run --volumes-from newnginx --name nginx1 -it docker.io/nginx /bin/bash
```

利用数据容器在创建一个容器nginx2，查看数据目录下容器nginx1创建的文件依旧存在，同理在nginx2的/mnt下创建文件，其他基于数据容器运行的新容器也可以看到文件

**3.3 数据盘管理**

在删除容器时，docker默认不会删除其数据盘。

```shell
docker volume ls   # 查看数据盘
docker volume ls -f dangling=true    # 查看未被容器使用的数据盘
docker volume rm VOLUME NAME    # 删除数据盘
```

如果想要删除容器时，同时删除掉其数据盘，那么可以使用`-v`参数。

```shell
docker rm -v newnginx
```

## 四、网络

docker提供几种网络，它决定容器之间和外界和容器之间如何去相互通信
```shell
docker newwork ls  # 查看网络
```
当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。

**4.1 bridge桥接网络**

除非创建容器的时候指定网络，不然容器就会默认的使用桥接网络。属于这个网络的容器之间可以相互通信，不过外界想要访问到这个网络的容器呢，需使用桥接网络，有点像主机和容器之间的一座桥，对容器有一点隔离作用。实际是在iptables做了DNAT规则，实现端口转发功能。可以使用iptables -t nat -vnL查看。


**4.2 host主机网络**

如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。只用这种网络的容器会使用主机的网络，这种网络对外界是完全开放的，能够访问到主机，就能访问到容器。

**4.3 使用none模式**

Docker容器拥有自己的Network Namespace，但是，并不为Docker容器进行任何网络配置。也就是说，这个Docker容器没有网卡、IP、路由等信息。需要我们自己为Docker容器添加网卡、配置IP等。使用此种网络的容器会完全隔离。

**4.4 简单演示**

```shell
for i in `docker ps | grep -v "CONTAINER" | awk '{print $1}'`; do docker inspect $i | grep 'IPAddress'; done
```

查看桥接模式下主机内部容器之间和容器与宿主机直接均可正常通信

```shell
docker inspect 容器ID
```

查看host创建的容器内部没有IP地址，它使用的为宿主机的地址

```shell
docker run -d --new host docker.io/sergeyzh/centos-nginx
```

查看host创建的容器内部没有IP地址，它使用的为宿主机的地址

```shell
docker run -d --new none docker.io/sergeyzh/centos-nginx
```
**4.5 容器端口**

如果想让外界可以访问到，基于bridge网络创建的容器提供的服务，那你可以告诉Docker你要使用哪些接口。如果想查看镜像会使用哪些端口，ExposedPorts，可以获悉镜像使用哪些端口。

```shell
docker run -d -p 80 docker.io/sergeyzh/centos-nginx
docker port 09648b2ff7f6
```
`-p` 参数会在宿主机随机映射一个高端口到容器内的指定端口

```shell
docker run -d -p 80:80 docker.io/sergeyzh/centos6-nginx  #将宿主机的80端口映射到容器的80端口
```
