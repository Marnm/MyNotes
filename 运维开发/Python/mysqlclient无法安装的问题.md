---
title: mysqlclient无法安装的问题
tags:
  - mysqlclient
---

```shell


sudo apt install -y mysql-client 

#基于python安装mysqlclient需要依次安装以下库: 
sudo apt-get install libmysqlclient-dev 
sudo apt install libssl-dev 
sudo apt install libcrypto++-dev 
sudo pip3 install mysqlclient 
# 最后这项也可以在虚拟环境中安装 
pip install mysqlclient
```
