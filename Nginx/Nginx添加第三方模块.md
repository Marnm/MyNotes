---
title: Nginx添加第三方模块
tags: []
---

## Nginx添加第三方模块

> 以会话保持nginx-stiky模块为例

下载地址：https://github.com/bymaximus/nginx-sticky-module-ng  
```shell
[root@rhel76 nginx]$ nginx -V
nginx version: nginx/1.12.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --with-http_v2_module --with-http_ssl_module --with-http_sub_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre
[root@rhel76 nginx]$

[root@rhel76 Packages]$ tar zxf master.tar.gz
[root@rhel76 Packages]$ mv nginx-goodies-nginx-sticky-module-ng-08a395c66e42 /usr/local/nginx-stiky-module
[root@rhel76 ~]$ ls /usr/local/nginx-stiky-module/
basic.t  Changelog.txt  config  docs  LICENSE  ngx_http_sticky_misc.c  ngx_http_sticky_misc.h  ngx_http_sticky_module.c  patches  README.md
[root@rhel76 ~]$ cd /usr/local/src/nginx-1.12.2/
[root@rhel76 nginx-1.12.2]$ ./configure --prefix=/usr/local/nginx --add-module=/usr/local/nginx-stiky-module/ --with-http_stub_status_module --with-http_sub_module --with-http_ssl_module --with-pcre
[root@rhel76 nginx-1.12.2]$ make

[root@rhel76 nginx-1.12.2]$ nginx -V
nginx version: nginx/1.12.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --with-http_v2_module --with-http_ssl_module --with-http_sub_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre

[root@rhel76 nginx-1.12.2]$ nginx -s stop

[root@rhel76 nginx-1.12.2]$ pwd
/usr/local/src/nginx-1.12.2
[root@rhel76 nginx-1.12.2]$ ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
[root@rhel76 nginx-1.12.2]$ cp ./objs/nginx /usr/local/nginx/sbin/
cp: overwrite ‘/usr/local/nginx/sbin/nginx’? yes

[root@rhel76 nginx-1.12.2]$ nginx
[root@rhel76 nginx-1.12.2]$ nginx -V
nginx version: nginx/1.12.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --add-module=/usr/local/nginx-stiky-module/ --with-http_stub_status_module --with-http_sub_module --with-http_ssl_module --with-pcre
[root@rhel76 nginx-1.12.2]$
```
