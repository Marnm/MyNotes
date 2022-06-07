---
title: FTP/TFTP Server的配置
tags:
  - FTP
  - TFTP
---

## TFTP
> 简单文件传输协议

**安装**
```bash
yum install tftp-server tftp xinetd
```
**配置**
```bash
vim /etc/xinetd.d/ftp
disable = no
```

> tftp服务文件默认存放目录
> /var/lib/tftpboot/


## FTP Server
### 简单配置
> `/etc/vsftpd/vsftpd.conf`  # ftp主要配置文件
> `/etc/vsftpd/ftpusers` # ftp黑名单
> `/etc/vsftpd/user_list` #白名单
> `/var/ftp` ftp默认根目录

**安装**
```bash
yum install -y vsftp lftp
```

**匿名访问FTP配置**
示例
```bash
# 启用匿名访问
anonymous_enable=YES
# 允许匿名用户上传
anon_upload_enable=YES
# 允许匿名用户创建目录
anon_mkdir_write_enable=YES
# 允许匿名用户修改
anon_other_write_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
```

### **多用户FTP**

> 关闭匿名访问

```bash
vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
...
anon_other_write_enable=NO
```
> 添加FTP用户
```bash
useradd -s /sbin/nologin ftp_user1
useradd -s /sbin/nologin ftp_user2
```

> 修改配置文件

```bash
vim /etc/vsftpd/vsftpd.conf

chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
allow_writeable_chroot=YES
local_enable=YES


touch /etc/vsftpd/chroot_list
vim /etc/vsftpd/chroot_list
ftp_user1
ftp_user2
```
**示例**
```bash
anonymous_enable=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=NO
local_root=/var/www/html
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
allow_writeable_chroot=YES
local_enable=YES
local_umask=022
write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=YES
#listen_ipv6=NO
```

### FTP虚拟用户
> https://access.redhat.com/solutions/54295
> 创建FTP登录用户

```bash
vim /etc/vsftpd/login.txt

user1
user1_passwd
user2
user2_passwd
```
> 将用户信息转换为数据库文件

```bash
db_load -T -t hash -f /etc/vsftpd/login.txt /etc/vsftpd/login.db
```

> 需修改用户数据库的文件权限

```bash
chmod 0600 /etc/vsftpd/login.db
```

> 更新vsftpd pam配置文件

```bash
vim /etc/pam.d/vsftpd
# 如果你允许ftp虚拟yoghurt访问，可以把其它的内容都注释掉
auth        required    pam_userdb.so   db=/etc/vsftpd/login
account    required   pam_userdb.so    db=/etc/vsftpd/login
```
> 为虚拟用户创建主目录

```bash
mkdir /home/ftp
mkdir -p /home/ftp/{user1,user2}
chown -R ftp:ftp /home/ftp
```

> **修改vsftpd配置文件**

```bash
vim /etc/vsftpd/vsftpd.conf
# 增加如下参数
guest_enable=YES
guest_username=virtftp
user_sub_token=$USER
local_root=/home/ftp/$USER
chroot_local_user=YES   # 强制虚拟用户只能访问自己的主目录中的文件
```

> 创建virtftp用户

```bash
useradd -d /var/ftp -M -s /sbin/nologin virtftp
```

> 创建ftp虚拟用户配置文件

```bash
vim /etc/vsftpd/vsftpd.conf
# 添加
user_config_dir=/etc/vsftpd/vsftpd_user_conf

mkdir /etc/vsftpd/vsftpd_user_conf
echo "anon_world_readable_only=NO" > /etc/vsftpd/vsftpd_user_conf/user1

vim /etc/vsftpd/vsftpd_user_conf/user2
anon_world_readable_only=NO
write_enable=YES
anon_upload_enable=YES
```

## vsftpd.conf参数说明
> 原文 [http://vsftpd.beasts.org/vsftpd_conf.html#index](http://vsftpd.beasts.org/vsftpd_conf.html#index)

### NAME
vsftpd.conf - vsftpd配置文件

### DESCRIPTION
> `vsftpd.conf` 用于控制vsftpd行为。默认情况下，vsftpd在`/etc/vsftpd.conf` 位置查找此文件。但是，您可以通过vsftpd指定命令行参数来覆盖它。

### FORMAT
> `vsftpd.conf` 的格式非常简单。每行要么是注释，要么是指令。注释行以`#` 开头并被vsftpd忽略。

`options=value`  选项=值
**注意！**在每个=和左右不能有空格


### 布尔选项
> 下面是Boolean选项列表。布尔选项的值可以设置为YES或NO

**allow_anon_ssl**
只有**ssl_enable=YES**时才适用。如果设置为YES，匿名用户将被允许使用安全的SSL连接。
    - 默认值：NO
    
**anon_mkdir_write_enable**
    如果设置为YES，将允许匿名用户在特定条件下创建新目录。为此，必须激活**write_enable**，并且ftp匿名用户必须对父目录具有写权限
    - 默认值：NO

**anon_other_write_enable**
\# 允许匿名用户被执行除上传和创建目录之外的写操作，例如删除和重命名

**anon_upload_enable**
\# 允许匿名用户在特定条件下上传文件，在**write_enable**激活的条件下。并且匿名用户上传位置具有写入权限
\# 虚拟用户上传 也需要此设置；默认情况下，虚拟用户具有匿名（即最大限制）权限
- 默认值：NO

**anon_world_readable_only**
\# 允许匿名用户下载可读文件，（不能在ftp中打开）
- 默认值：YES

**anonymous_enable**
\# 允许匿名用户登录，默认用户名：**ftp** 、**anonymous**
默认值：YES

**ascii_download_enable**
\# 下载时采用ASCII模式传输数据


**ascii_upload_enable**
\# 上传时接收ASCII模式数据传输

**async_abor_enable**
\# 启用`async ABOR` 特殊FTP命令。不建议启用

**background**
\# FTP以`listen`模式启动，

**check_shell**
\# 该选项仅对非PAM构建的vsftpd有效。如果禁用，vsftpd将不会检查/etc/shell以获取本地登录的有效用户shell
默认值：YES

**chmod_enable**
\# 允许使用SITE CHMOD命令。仅使用与本地用户。匿名用户无法使用SITE CHMOD。
默认值：YES

**chown_uploads**
\# 所有匿名上传文件的所有权都将更改为**chown_username**选项指定的用户。


**chroot_list_enable**
\# 只允许用户访问自己的目录。
\# 如果启用**chroot_local_user**，与上述相反

**chroot_local_user**
\# 本地用户登录后只能访问自己的主目录

**connect_from_port_20**
\# 控制端口数据连接是否使用服务器上的端口ftp。

**debug_ssl**
\# OpenSSL连接诊断将转存到vsftpd日志文件。

**delete_failed_uploads**
\# 删除任何失败的上传文件

**deny_email_enable**
\# 如果激活，您可以提供导致登录被拒绝的匿名密码电子邮件响应列表。默认情况下，包含此列表的文件是 /etc/vsftpd.banned_emails，但您可以使用banned_email_file 设置覆盖它 。

**dirlist_enable**
\# 所有目录列表命令都将拒绝授权
默认值：YES

**dirmessage_enable**
\# 用户在他们第一次进入新目录时看到消息。默认情况下会扫描目录中的.message文件，可以通过message_file修改

**download_enable**
\# 允许下载。如果设置为NO，所有下载请求都将被拒绝
默认值：YES

**dual_log_enable**
\# 双日志文件。启用将生成 **/var/log/xferlog** 和 **/var/log/vsftpd.log**。前者是wu-ftpd风格的传输日志，可由标准工具解析。后者是vsftpd的风格日志。

**force_dot_files**
\# 

**force_anon_data_ssl**
**force_local_logins_ssl**
**guest_enable**
**hide_ids**
**implicit_ssl**
**listen**
**listen_ipv6**
**local_enable**
**lock_upload_files**
**log_ftp_protocol**
**ls_recurse_enable**
**mdtm_write**
**no_anon_password**
**no_log_lock**
**one_process_model**
**passwd_chroot_enable**
**pasv_addr_resolv**
**pasv_enable**
**pasv_promiscuous**
**port_enable**
**port_promiscuous**
**require_cert**
**require_ssl_reuse**
**run_as_launching_user**
**secure_email_list_enable**
**session_support**
**setproctitle_enable**
**ssl_enable**
**ssl_request_cert**
**ssl_sslv2**
**ssl_sslv3**
**ssl_tlsv1**
**strict_ssl_read_eof**
**strict_ssl_write_shutdown**
**syslog_enable**
**tcp_wrappers**
**text_userdb_names**
**tilde_user_enable**
**use_localtime**
**use_sendfile**
**userlist_deny**
**userlist_enable**
**calidate_cert**
**virtual_use_local_privs**
**write_enable**
**xferlog_enable**
**xferlog_std_format**



### 数值选项

**accept_timeout**
**anon_max_rate**
**anon_umask**
**chown_upload_mode**
**connect_timeout**
**data_connection_timeout**
**delay_failed_login**
**delay_successful_login**
**file_open_mode**
**ftp_data_port**
**idle_session_timeout**
**listen_port**
**local_max_rate**
**local_umask**
**max_clients**
**max_login_fails**
**max_per_ip**
**pasv_max_port**
**pasv_min_port**
**trans_chunk_size**

### 字符串选项

**anon_root**
**banned_email_file**
**banner_file**
**ca_certs_file**
**chown_username**
**chroot_list_file**
**cmds_allowed**
**cmds_denied**
**deny_file**
**dsa_cert_file**
**dsa_private_key_file**
**email_password_file**
**ftp_username**
**ftpd_banner**
**guest_username**
**hide_file**
**listen_address**
**listen_address6**
**local_root**
**message_file**
**nopriv_user**
**pam_service_name**
**pasv_address**
**rsa_cert_file**
**rsa_private_key_file**
**secure_chroot_dir**
**ssl_ciphers**
**user_config_dir**
**user_sub_token**
**userlist_file**
**vsftpd_log_file**
**xferlog_file**
