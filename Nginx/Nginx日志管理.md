---
title: Nginx日志管理
tags: []
---

---

title: Nginx日志管理
date: 2022-06-02 10:29:20
tags:
	- Nginx
	- web
categories: 运维开发

---

## Nginx日志管理

**Nginx日志主要分为两种：访问日志和错误日志**

日志开关在Nginx配置文件`nginx.conf`中设置，两种日志都可以选择性关闭，默认都是打开的。

**关闭日志的方式**

	access_log off;
	error_log /dev/null;

需要注意的是：Nginx进程设置的用户和用户组必须对日志路径有创建文件的权限，否则会报错！

## 访问日志access.log

访问日志主要记录客户端访问Nginx的每一个请求，格式可以自定义。通过访问日志，你可以得到用户地域来源、跳转来源、使用终端、某个URL访问量等相关信息。

能够使用access_log指令的字段包括：http、server、location

观察nginx的http段,可以看到如下类似信息(日志使用默认配置，如果需要修改日志配置，需要先取消注释)

	   日志类型       日志路径            日志格式
	#  access_log    logs/access.log     main;
	
这说明该http的访问日志的文件是logs/access.log ，使用的格式”main”格式，main格式是nginx定义好一种日志的格式，日志定义信息如下：

```
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
```

### 日志变量

`$server_name`：虚拟主机名称。  
`$remote_addr`：远程客户端IP地址。  
`-`：空白，用一个"-"占位符替代，历史原因导致还存在  
`$remote_user`：远程客户端用户名称，用于记录浏览者进行身份验证时提供的名字，如登录的用户名Jaking666，如果没有登录就是空白。  
`$time_local`：访问的时间与时区，比如18/Jul/2012:17:00:01 +0800。时间信息最后的"+0800"表示服务器所处时区位于UTC之后的8小时。  
`$request`：请求的URI和HTTP协议  
`$status`：记录请求返回的http状态码  
`$body_bytes_sent`：发送给客户端的文件主体内容的大小（可以将日志每条记录中的这个值累加起来以粗略估计服务器吞吐量）
`$http_referer`：记录从哪个页面链接访问过来的。  
`$http_user_agent`：客户端浏览器信息  
`$http_x_forwarded_for`：客户端的真实IP（通常web服务器放在反向代理的后面，这样就不能获取到客户端的IP地址了，通过`$request_addr`拿到的IP地址是反向代理服务器的IP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forward_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址）  
`uptream_status`upstream状态  
`upstream_addr`后端服务器的IP地址  
`ssl_protocol`SSL协议版本，比如TLSv1。  
`ssl_cipher`交换数据中的算法，比如RC4-SHA。   
`upstream_addr`upstream的地址，即真正提供服务的主机地址。  
`request_time`整个请求的总时间。  
`upstream_response_time`请求过程中，upstream的响应时间。  

**打开nginx.conf配置文件：** `vim /usr/local/nginx/nginx.conf`  
日志部分内容：  

	# access_log    logs/access.log    main;
	
日志生成的到Nginx根目录哦logs/access.log文件，默认使用"main"日志格式，也可以自定义格式。

除了main格式，还可以自定义其他格式，格式为：log_format name(格式名称) type(格式样式)

例如：
```
	#log format main 'remote_addr - $remote_user [$time_local] "$request" '
	#				 '$status $body_bytes_sent "$http_referer"'
	#				 '"$http_user_agent" "$http_x_forwarded_for"';

	# access_log logs/access.log    main;
```

**查看NGINX日志：**  
```shell
[root@rhel76 ~]& tail -f /usr/local/nginx/logs/access.log
'
192.168.73.20 - - [03/Jun/2022:06:09:06 +0800] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.29.0"
192.168.73.20 - - [03/Jun/2022:06:09:18 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0"
192.168.73.20 - - [03/Jun/2022:06:09:30 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0"
192.168.73.1 - - [03/Jun/2022:06:27:17 +0800] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36"
192.168.73.1 - - [03/Jun/2022:06:27:17 +0800] "GET /favicon.ico HTTP/1.1" 404 571 "http://192.168.73.24/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36"
192.168.73.1 - - [03/Jun/2022:06:28:17 +0800] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36"
192.168.73.1 - - [03/Jun/2022:06:28:17 +0800] "GET /favicon.ico HTTP/1.1" 404 571 "http://192.168.73.24/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36"
192.168.73.1 - - [03/Jun/2022:06:33:36 +0800] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.63 Safari/537.36 Edg/102.0.1245.30"
192.168.73.1 - - [03/Jun/2022:06:33:37 +0800] "GET /favicon.ico HTTP/1.1" 404 571 "http://192.168.73.24/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.63 Safari/537.36 Edg/102.0.1245.30"

'
```

### Nginx自定义日志

```shell
[root@rhel76 ~]& vim /usr/local/nginx/conf/nginx.conf
```
```ini
http {
    include       mime.types;
    default_type  application/octet-stream;

	# 启用日志格式化
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
	# 不使用默认日志文件
    #########access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;
        
		# 启用自定义日志文件
        access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
# ---------------------------省略--------------------------------------
}
# ---------------------------省略--------------------------------------
```

**查看日志文件**
```shell
[root@rhel76 ~]& nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@rhel76 ~]& systemctl restart nginx
[root@rhel76 ~]& tail -f /usr/local/nginx/logs/host.access.log
```

**使用curl请求nginx信息**
```shell
[root@ctos74 ~]& curl -I 192.168.73.24
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Fri, 03 Jun 2022 11:55:27 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Thu, 02 Jun 2022 21:48:43 GMT
Connection: keep-alive
ETag: "6299303b-264"
Accept-Ranges: bytes

[root@ctos74 ~]&
```

**Nginx的日志变化：**  
```shell
[root@rhel76 ~]& tail -f /usr/local/nginx/logs/host.access.log  
192.168.73.20 - - [03/Jun/2022:19:55:27 +0800] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.29.0" "-"
```

**使用浏览器访问nginx服务：**  

> 这里以Microsoft Edge为例  
> Edge版本 102.0.1245.30 (正式版本) (64 位)

```shell
[root@rhel76 ~]& tail -f /usr/local/nginx/logs/host.access.log
192.168.73.20 - - [03/Jun/2022:19:55:27 +0800] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.29.0" "-"
192.168.73.1 - - [03/Jun/2022:20:02:33 +0800] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.63 Safari/537.36 Edg/102.0.1245.30" "-"
192.168.73.1 - - [03/Jun/2022:20:02:33 +0800] "GET /favicon.ico HTTP/1.1" 404 571 "http://192.168.73.24/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.63 Safari/537.36 Edg/102.0.1245.30" "-"

```

### Nginx日志切割  

创建nginx日志按每分钟自动切割脚本如下：  

```shell
[root@rhel76 ~]& vim /usr/local/nginx/conf/nginx_log.sh
```
```shell
#!/bin/bash

# 设置日志文件存放目录
LOG_HOME="/usr/local/nginx/logs/"

# 备份文件名称
LOG_PATH_BAK="$(date -d yesterday +%Y%m%d%H%M)".nginx666.access.log

# 重命名日志文件
mv ${LOG_HOME}/nginx666.access.log ${LOG_HOME}/${LOG_PATH_BAK}

# 向nginx主机进程发送信号，重新打开日志
kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
```
```shell
[root@rhel76 ~]& chmod +x /usr/local/nginx/conf/nginx_log.sh
```

**创建crontab设置作业**  

```shell
[root@rhel76 ~]& crontab -e
* * * * * /bin/bash /usr/local/nginx/conf/nginx_log.sh
crontab: installing new crontab
[root@rhel76 ~]&
 1L, 55C written
[root@rhel76 ~]& 
```

```shell
[root@rhel76 ~]& ls /usr/local/nginx/logs/
access.log  error.log  host.access.log  nginx666.access.log  nginx.pid
[root@rhel76 ~]&
[root@rhel76 ~]& ll /usr/local/nginx/logs/
total 16
-rw-r--r--. 1 root   root    0 Jun  3 20:27 202206022028.nginx666.access.log
-rw-r--r--. 1 root   root 1540 Jun  3 06:33 access.log
-rw-r--r--. 1 nobody root 2741 Jun  3 20:02 error.log
-rw-r--r--. 1 nobody root 2246 Jun  3 20:24 host.access.log
-rw-r--r--. 1 nobody root    0 Jun  3 20:28 nginx666.access.log
-rw-r--r--. 1 root   root    6 Jun  3 20:27 nginx.pid
[root@rhel76 ~]&
[root@rhel76 ~]& ll /usr/local/nginx/logs/
total 24
-rw-r--r--. 1 root   root    0 Jun  3 20:27 202206022028.nginx666.access.log
-rw-r--r--. 1 nobody root   94 Jun  3 20:28 202206022029.nginx666.access.log
-rw-r--r--. 1 root   root 1540 Jun  3 06:33 access.log
-rw-r--r--. 1 nobody root 2741 Jun  3 20:02 error.log
-rw-r--r--. 1 nobody root 2246 Jun  3 20:24 host.access.log
-rw-r--r--. 1 nobody root  188 Jun  3 20:29 nginx666.access.log
-rw-r--r--. 1 root   root    6 Jun  3 20:27 nginx.pid
[root@rhel76 ~]&
```

## Nginx错误日志error.log
错误日志主要记录客户端访问Nginx出错时的日志，格式不支持自定义，通过错误日志，可以得到系统某个服务或server的性能瓶颈等。   

**Nginx错误日志的几个等级**  
error_ log 指令的第1个参数用于存放错误日志的路径。第2个参数用于指定错误记录详细程度的等级,默认值为error  
`debug`、`info`、`notice`、`warn`、`error`和`crit`, 日志记录详细程度依次递减,debug记录的内容最详细,crit记录的内容最简洁。  

```shell
[root@rhel76 ~]& cat /usr/local/nginx/logs/error.log
```
```
2022/06/03 05:54:01 [emerg] 13522#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:01 [emerg] 13522#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:01 [emerg] 13522#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:01 [emerg] 13522#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:01 [emerg] 13522#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:01 [emerg] 13522#0: still could not bind()
2022/06/03 05:54:09 [notice] 13533#0: signal process started
2022/06/03 05:54:09 [error] 13533#0: open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
2022/06/03 05:54:17 [emerg] 13549#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:17 [emerg] 13549#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:17 [emerg] 13549#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:17 [emerg] 13549#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:17 [emerg] 13549#0: bind() to 0.0.0.0:80 failed (98: Address already in use)
2022/06/03 05:54:17 [emerg] 13549#0: still could not bind()
2022/06/03 05:54:37 [notice] 13571#0: signal process started
2022/06/03 05:54:37 [error] 13571#0: open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
2022/06/03 05:54:41 [notice] 13575#0: signal process started
2022/06/03 05:54:41 [error] 13575#0: open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
2022/06/03 05:55:01 [alert] 13417#0: unlink() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
2022/06/03 06:27:17 [error] 13613#0: *4 open() "/usr/local/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.73.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.73.24", referrer: "http://192.168.73.24/"
```

## 第三方日志分析工具————GoAccess

GoAccess 是一款开源的且具有交互视图界面的实时 Web 日志分析工具，通过你的 Web 浏览器或者 *nix 系统下的终端程序(terminal)即可访问。

能为系统管理员提供快速且有价值的 HTTP 统计，并以在线可视化服务器的方式呈现。  

**下载安装**  
官网：https://goaccess.io/  
中文站：https://www.goaccess.cc/  
最新版下载地址：https://goaccess.io/download  
其它版本下载地址：https://github.com/allinurl/goaccess/tags  

```shell
[root@rhel76 Packages]& [root@rhel76 Packages]& wget https://github.com/allinurl/goaccess/archive/refs/tags/v1.2.tar.gz
[root@rhel76 Packages]& tar -zxvf v1.2.tar.gz
[root@rhel76 Packages]& cd goaccess-1.2
[root@rhel76 goaccess-1.2]& 
[root@rhel76 goaccess-1.2]& yum install -y ncurses-devel geoip-devel libmaxminddb-devel openssl-devel
[root@rhel76 goaccess-1.2]& ./configure --enable-utf8 --enable-geoip=mmdb
[root@rhel76 goaccess-1.2]& make && make install
[root@rhel76 ~]& goaccess

GoAccess - 1.2

Usage: goaccess [filename] [ options ... ] [-c][-M][-H][-q][-d][...]
The following options can also be supplied to the command:

Log & Date Format Options

  --date-format=<dateformat>      - Specify log date format. e.g., %d/%b/%Y
  --log-format=<logformat>        - Specify log format. Inner quotes need to be
                                    escaped, or use single quotes.
  --time-format=<timeformat>      - Specify log time format. e.g., %H:%M:%S

User Interface Options
---------------------------------省略----------------------------------------------------
```

**基本使用**
```
[root@rhel76 ~]& goaccess -f /usr/local/nginx/logs/access.log --log-format="%h %^[%d:%t %^] \"%r\" %s %b \"%R\" \"%u\"" --date-format="%d/%b/%Y" --time-format=%H:%M:%S
```
![XUSh7t.png](https://s1.ax1x.com/2022/06/03/XUSh7t.png)

生成HTML统计表  
```shell
[root@rhel76 nginx]$ goaccess -f /usr/local/nginx/logs/access.log --log-format="%h %^[%d:%t %^] \"%r\" %s %b \"%R\" \"%u\"" --date-format="%d/%b/%Y" --time-format=%H:%M:%S --hour-spec=min -o  /usr/local/nginx/html/nginx-log.html
[root@rhel76 nginx]$
```
![XUG0tP.png](https://s1.ax1x.com/2022/06/03/XUG0tP.png)
