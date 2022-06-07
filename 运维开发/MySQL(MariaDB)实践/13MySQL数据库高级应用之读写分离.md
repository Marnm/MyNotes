---
title: 13MySQL数据库高级应用之读写分离
tags:
  - mysql
  - mycat
---

# MySQL数据库高级应用之读写分离——MyCat篇

## 配置MySQL主从

### Master

修改配置
```ini
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

bind-address    = 0.0.0.0

user=mysql

innodb_flush_log_at_trx_commit=1
sync_binlog=1
log-bin=mysql-bin-master
server-id=1
binlog-do-db=ha
binlog-ignore-db=mysql
lower_case_table_names=1
skip-name-resolve

character-set-server=utf8
collation-server=utf8_general_ci
```

创建数据库，并设置授权用户
```sql
CREATE DATABASE ha;
use ha
CREATE TABLE test(id int, name char(16));

grant replication slave on *.* to root@'%' identified by '123456';

flush privileges;


show master status;
```

导出数据库
```shell
mysqldump -uroot -p -B ha > ha.sql
```


### Slave

修改配置文件
```ini
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

bind-address    = 0.0.0.0

log-bin=mysql-bin
server-id=3
binlog-do-db=ha
binlog-ignore-db=mysql
log_slave_updates=1
skip-name-resolve
lower_case_table_names=1

character-set-server=utf8
collation-server=utf8_general_ci
```

设置主服务
```sql
change master to master_host='master_ip_addr', master_user='root', master_password='123456', master_log_file='master_db_bin';
flush privileges;
start slave;
```

导入Master的数据库
```shell
mysql -uroot -p < ha.sql
```


## 安装配置MyCat

### 安装jdk，配置Java环境变量
```shell
mkdir /usr/java

tar -zxvf jdk-8u201-linux-x64.tar.gz -C /usr/java/
```

```shell
vim /etc/profile

JAVA_HOME=/usr/java/jdk1.8.0_201
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
export PATH JAVA_HOME CLASSPATH
MYCAT_HOME=/usr/local/mycat
PATH=$MYCAT_HOME/bin:$PATH

source /etc/profile
```

### 安装MyCat

```shell
tar -zxvf Mycat-server-1.6.7.3-release-20190927161129-linux.tar.gz -C /usr/local/
```

**修改wrapper.conf配置文件**
```ini
#********************************************************************
# Wrapper Properties
#********************************************************************
# Java Application
# 改成JDK安装目录
wrapper.java.command=/usr/java/jdk1.8.0_201/bin/java
wrapper.working.dir=..

# Java Main class.  This class must implement the WrapperListener interface
#  or guarantee that the WrapperManager class is initialized.  Helper
#  classes are provided to do this for you.  See the Integration section
#  of the documentation for details.
wrapper.java.mainclass=org.tanukisoftware.wrapper.WrapperSimpleApp
set.default.REPO_DIR=lib
set.APP_BASE=.

# Java Classpath (include wrapper.jar)  Add class path elements as
#  needed starting from 1
wrapper.java.classpath.1=lib/wrapper.jar
wrapper.java.classpath.2=conf
wrapper.java.classpath.3=%REPO_DIR%/*

# Java Library Path (location of Wrapper.DLL or libwrapper.so)
wrapper.java.library.path.1=lib

# Java Additional Parameters
#wrapper.java.additional.1=
wrapper.java.additional.1=-DMYCAT_HOME=.
wrapper.java.additional.2=-server
wrapper.java.additional.3=-XX:MaxPermSize=64M
wrapper.java.additional.4=-XX:+AggressiveOpts
wrapper.java.additional.5=-XX:MaxDirectMemorySize=2G
wrapper.java.additional.6=-Dcom.sun.management.jmxremote
wrapper.java.additional.7=-Dcom.sun.management.jmxremote.port=1984
wrapper.java.additional.8=-Dcom.sun.management.jmxremote.authenticate=false
wrapper.java.additional.9=-Dcom.sun.management.jmxremote.ssl=false
# 如果MyCat还是起不来调整一下性能要求
wrapper.java.additional.10=-Xmx640M
wrapper.java.additional.11=-Xms256M

# Initial Java Heap Size (in MB)
#wrapper.java.initmemory=3

# Maximum Java Heap Size (in MB)
#wrapper.java.maxmemory=64

# Application parameters.  Add parameters as needed starting from 1
wrapper.app.parameter.1=io.mycat.MycatStartup
wrapper.app.parameter.2=start

#********************************************************************
# Wrapper Logging Properties
#********************************************************************
# Format of output for the console.  (See docs for formats)
wrapper.console.format=PM

# Log Level for console output.  (See docs for log levels)
wrapper.console.loglevel=INFO

# Log file to use for wrapper output logging.
wrapper.logfile=logs/wrapper.log

# Format of output for the log file.  (See docs for formats)
wrapper.logfile.format=LPTM

# Log Level for log file output.  (See docs for log levels)
wrapper.logfile.loglevel=INFO

# Maximum size that the log file will be allowed to grow to before
#  the log is rolled. Size is specified in bytes.  The default value
#  of 0, disables log rolling.  May abbreviate with the 'k' (kb) or
#  'm' (mb) suffix.  For example: 10m = 10 megabytes.
wrapper.logfile.maxsize=512m

# Maximum number of rolled log files which will be allowed before old
#  files are deleted.  The default value of 0 implies no limit.
wrapper.logfile.maxfiles=30

# Log Level for sys/event log output.  (See docs for log levels)
wrapper.syslog.loglevel=NONE

#********************************************************************
# Wrapper Windows Properties
#********************************************************************
# Title to use when running as a console
wrapper.console.title=Mycat-server

#********************************************************************
# Wrapper Windows NT/2000/XP Service Properties
#********************************************************************
# WARNING - Do not modify any of these properties when an application
#  using this configuration file has been installed as a service.
#  Please uninstall the service before modifying this section.  The
#  service can then be reinstalled.

# Name of the service
wrapper.ntservice.name=mycat

# Display name of the service
wrapper.ntservice.displayname=Mycat-server

# Description of the service
wrapper.ntservice.description=The project of Mycat-server

# Service dependencies.  Add dependencies as needed starting from 1
wrapper.ntservice.dependency.1=

# Mode in which the service is installed.  AUTO_START or DEMAND_START
wrapper.ntservice.starttype=AUTO_START

# Allow the service to interact with the desktop.
wrapper.ntservice.interactive=false

wrapper.ping.timeout=120
configuration.directory.in.classpath.first=conf
```

**修改server.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- - - Licensed under the Apache License, Version 2.0 (the "License");
        - you may not use this file except in compliance with the License. - You
        may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0
        - - Unless required by applicable law or agreed to in writing, software -
        distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT
        WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the
        License for the specific language governing permissions and - limitations
        under the License. -->
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
        <system>
        <property name="nonePasswordLogin">0</property> <!-- 0为需要密码登陆、1为不需要密码登陆 ,默认为0，设置为1则需要指定默认账户-->
        <property name="useHandshakeV10">1</property>
        <property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->
        <property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为 关闭 -->
                <property name="sqlExecuteTimeout">300</property>  <!-- SQL 执行超时 单位:秒-->
                <property name="sequnceHandlerType">2</property>
                <!--<property name="sequnceHandlerPattern">(?:(\s*next\s+value\s+for\s*MYCATSEQ_(\w+))(,|\)|\s)*)+</property>-->
                <!--必须带有MYCATSEQ_或者 mycatseq_进入序列匹配流程 注意MYCATSEQ_有空格的情况-->
                <property name="sequnceHandlerPattern">(?:(\s*next\s+value\s+for\s*MYCATSEQ_(\w+))(,|\)|\s)*)+</property>
        <property name="subqueryRelationshipCheck">false</property> <!-- 子查询中存在关联查询的情况下,检查关联字段中是否有分片字段 .默认 false -->
      <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
        <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--设置模拟的MySQL版本 号-->
        <!-- <property name="processorBufferChunk">40960</property> -->
        <!--
        <property name="processors">1</property>
        <property name="processorExecutor">32</property>
         -->
        <!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena | type 2 NettyBufferPool -->
                <property name="processorBufferPoolType">0</property>
                <!--默认是65535 64K 用于sql解析时最大文本长度 -->
                <!--<property name="maxStringLiteralLength">65535</property>-->
                <!--<property name="sequnceHandlerType">0</property>-->
                <!--<property name="backSocketNoDelay">1</property>-->
                <!--<property name="frontSocketNoDelay">1</property>-->
                <!--<property name="processorExecutor">16</property>-->
                <!--
                        <property name="serverPort">8066</property> <property name="managerPort">9066</property>
                        <property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property>
                        <property name="dataNodeIdleCheckPeriod">300000</property> 5 * 60 * 1000L; //连接空闲检查
                        <property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
                <!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内 只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
                <property name="handleDistributedTransactions">0</property>

                        <!--
                        off heap for merge/order/group/limit      1开启   0关闭
                -->
                <property name="useOffHeapForMerge">0</property>

                <!--
                        单位为m
                -->
        <property name="memoryPageSize">64k</property>

                <!--
                        单位为k
                -->
                <property name="spillsFileBufferSize">1k</property>

                <property name="useStreamOutput">0</property>

                <!--
                        单位为m
                -->
                <property name="systemReserveMemorySize">384m</property>


                <!--是否采用zookeeper协调切换  -->
                <property name="useZKSwitch">false</property>

                <!-- XA Recovery Log日志路径 -->
                <!--<property name="XARecoveryLogBaseDir">./</property>-->

                <!-- XA Recovery Log日志名称 -->
                <!--<property name="XARecoveryLogBaseName">tmlog</property>-->
                <!--如果为 true的话 严格遵守隔离级别,不会在仅仅只有select语句的时候在事务中切换连接-->
                <property name="strictTxIsolation">false</property>

                <property name="useZKSwitch">true</property>

        </system>

        <!-- 全局SQL防火墙设置 -->
        <!--白名单可以使用通配符%或着*-->
        <!--例如<host host="127.0.0.*" user="root"/>-->
        <!--例如<host host="127.0.*" user="root"/>-->
        <!--例如<host host="127.*" user="root"/>-->
        <!--例如<host host="1*7.*" user="root"/>-->
        <!--这些配置情况下对于127.0.0.1都能以root账户登录-->
        <!--
        <firewall>
           <whitehost>
              <host host="1*7.0.0.*" user="root"/>
           </whitehost>
       <blacklist check="false">
       </blacklist>
        </firewall>
        -->
		<!-- fnaichu:修改相关数据库配置： -->
        <user name="root" defaultAccount="true">
                <property name="password">ccc.456</property>
                <property name="schemas">ha</property>

                <!-- 表级 DML 权限设置 -->
                <!--
                <privileges check="false">
                        <schema name="TESTDB" dml="0110" >
                                <table name="tb01" dml="0000"></table>
                                <table name="tb02" dml="1111"></table>
                        </schema>
                </privileges>
                 -->
        </user>

		<!-- fnaichu:修改这里 -->
        <user name="user">
                <property name="password">user</property>
                <property name="schemas">ha</property>
                <property name="readOnly">true</property>
        </user>

</mycat:server>
```

**修改schema.xml**
```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
<schema name="ha" checkSQLschema="false" sqlMaxLimit="100" dataNode='dn1'>
        </schema>
        <dataNode name="dn1" dataHost="dthost" database="ha"/>
        <dataHost name="dthost" maxCon="500" minCon="10" balance="3" writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="rhel7" url="192.168.73.21:3306" user="root" password="ccc.456">
        <readHost host="rhel_sqlsrv" url="192.168.73.22:3306" user="root" password="ccc.456" />
        </writeHost>
        </dataHost>
</mycat:schema>
```

**修改log4j2.xml文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d [%-5p][%t] %m %throwable{full} (%C:%F:%L) %n"/>
        </Console>

        <RollingFile name="RollingFile" fileName="${sys:MYCAT_HOME}/logs/mycat.log"
                     filePattern="${sys:MYCAT_HOME}/logs/$${date:yyyy-MM}/mycat-%d{MM-dd}-%i.log.gz">
        <PatternLayout>
                <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %5p [%t] (%l) - %m%n</Pattern>
            </PatternLayout>
            <Policies>
                <OnStartupTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="250 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
    </Appenders>
    <Loggers>
        <!--<AsyncLogger name="io.mycat" level="info" includeLocation="true" additivity="false">-->
            <!--<AppenderRef ref="Console"/>-->
            <!--<AppenderRef ref="RollingFile"/>-->
        <!--</AsyncLogger>-->
		
		<!-- fnaichu:调整日志级别 -->
        <asyncRoot level="debug" includeLocation="true">

            <!--<AppenderRef ref="Console" />-->
            <AppenderRef ref="RollingFile"/>

        </asyncRoot>
    </Loggers>
</Configuration>
```

## 测试读写分离

目前MyCAT有两个端口，8066数据端口，9066管理端口，测试读写操作用8066数据端口

```shell
[root@ctos7mini ~]& netstat -pantul | grep 66
tcp6       0      0 :::46667                :::*                    LISTEN      4136/java

tcp6       0      0 :::8066                 :::*                    LISTEN      4136/java

tcp6       0      0 :::9066                 :::*                    LISTEN      4136/java

[root@ctos7mini ~]#
```

```sql
[root@ctos7mini ~]# mysql -uroot -pccc.456 -h192.168.73.20 -P8066
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.6.29-mycat-1.6.7.3-release-20190927161129 MyCat Server (OpenCloudDB)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+----------+
| DATABASE |
+----------+
| ha       |
+----------+
1 row in set (0.00 sec)

mysql>




[root@ctos7mini ~]# mysql -uroot -pccc.456 -h192.168.73.20 -P9066
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.6.29-mycat-1.6.7.3-release-20190927161129 MyCat Server (monitor)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+----------+
| DATABASE |
+----------+
| ha       |
+----------+
1 row in set (0.00 sec)

mysql>
```

**观察日志变化**

```shell
tail -f /usr/local/mycat/logs/mycat.log
```

```
2022-05-08 01:19:04.198 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.server.NonBlockingSession.releaseConnection(NonBlockingSession.java:386)) - release connection MySQLConnection [id=11, lastTime=1651943944189, user=root, schema=ha, old shema=ha, borrowed=true, fromSlaveDB=true, threadId=15, charset=utf8, txIsolation=3, autocommit=true, attachment=dn1{select * from test}, respHandler=SingleNodeHandler [node=dn1{select * from test}, packetId=6], host=192.168.73.22, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]


2022-05-08 01:22:56.420 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.server.NonBlockingSession.releaseConnection(NonBlockingSession.java:386)) - release connection MySQLConnection [id=10, lastTime=1651944176407, user=root, schema=ha, old shema=ha, borrowed=true, fromSlaveDB=false, threadId=23, charset=utf8, txIsolation=3, autocommit=true, attachment=dn1{insert into test values (2,'superman')}, respHandler=SingleNodeHandler [node=dn1{insert into test values (2,'superman')}, packetId=1], host=192.168.73.21, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=true]


2022-05-08 01:36:04.542 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.server.NonBlockingSession.execute(NonBlockingSession.java:126)) - ServerConnection [id=3, schema=ha, host=192.168.73.20, user=root,txIsolation=3, autocommit=true, schema=ha, executeSql=insert into test values (2,'superman')]insert into test values (2,'superman'), route={
   1 -> dn1{insert into test values (2,'superman')}
} rrs
2022-05-08 01:36:04.542 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.mysql.nio.handler.SingleNodeHandler.execute(SingleNodeHandler.java:170)) - rrs.getRunOnSlave() default
2022-05-08 01:36:04.542 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.mysql.nio.handler.SingleNodeHandler.execute(SingleNodeHandler.java:172)) - node.getRunOnSlave()  default
2022-05-08 01:36:04.542 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.mysql.nio.handler.SingleNodeHandler.execute(SingleNodeHandler.java:182)) - node.getRunOnSlave() null
2022-05-08 01:36:04.542 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.mysql.nio.handler.SingleNodeHandler.execute(SingleNodeHandler.java:184)) - node.getRunOnSlave() null
2022-05-08 01:36:04.542 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.datasource.PhysicalDBNode.getConnection(PhysicalDBNode.java:102)) - rrs.getRunOnSlave()  default
2022-05-08 01:36:04.542 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.datasource.PhysicalDBNode.getConnection(PhysicalDBNode.java:133)) - rrs.getRunOnSlave()  default
2022-05-08 01:36:04.554 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.server.NonBlockingSession.releaseConnection(NonBlockingSession.java:386)) - release connection MySQLConnection [id=10, lastTime=1651944964528, user=root, schema=ha, old shema=ha, borrowed=true, fromSlaveDB=false, threadId=23, charset=utf8, txIsolation=3, autocommit=true, attachment=dn1{insert into test values (2,'superman')}, respHandler=SingleNodeHandler [node=dn1{insert into test values (2,'superman')}, packetId=1], host=192.168.73.21, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=true]
2022-05-08 01:36:04.554 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.datasource.PhysicalDatasource.releaseChannel(PhysicalDatasource.java:633)) - release channel MySQLConnection [id=10, lastTime=1651944964528, user=root, schema=ha, old shema=ha, borrowed=true, fromSlaveDB=false, threadId=23, charset=utf8, txIsolation=3, autocommit=true, attachment=null, respHandler=null, host=192.168.73.21, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]

2022-05-08 01:36:40.600 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.backend.datasource.PhysicalDBPool.getRWBanlanceCon(PhysicalDBPool.java:551)) - select read source rhel_sqlsrv for dataHost:dthost
2022-05-08 01:36:40.601 DEBUG [$_NIOREACTOR-0-RW] (io.mycat.server.NonBlockingSession.releaseConnection(NonBlockingSession.java:386)) - release connection MySQLConnection [id=17, lastTime=1651945000587, user=root, schema=ha, old shema=ha, borrowed=true, fromSlaveDB=true, threadId=20, charset=utf8, txIsolation=3, autocommit=true, attachment=dn1{select * from test}, respHandler=SingleNodeHandler [node=dn1{select * from test}, packetId=10], host=192.168.73.22, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]

2022-05-08 01:53:32.188 DEBUG [Timer0] (io.mycat.sqlengine.SQLJob.connectionAcquired(SQLJob.java:89)) - con query sql:select user() to con:MySQLConnection [id=1, lastTime=1651946012188, user=root, schema=ha, old shema=ha, borrowed=true, fromSlaveDB=false, threadId=22, charset=utf8, txIsolation=3, autocommit=true, attachment=null, respHandler=null, host=192.168.73.21, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]
2022-05-08 01:53:32.188 DEBUG [Timer0] (io.mycat.sqlengine.SQLJob.connectionAcquired(SQLJob.java:89)) - con query sql:select user() to con:MySQLConnection [id=14, lastTime=1651946012188, user=root, schema=ha, old shema=ha, borrowed=true, fromSlaveDB=true, threadId=18, charset=utf8, txIsolation=3, autocommit=true, attachment=null, respHandler=null, host=192.168.73.22, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]
```
读操作是在Slave 192.168.73.22上进行的

写操作是在Master 192.168.73.21上进行的

## 替换log4j

参考：https://www.yuque.com/ccazhw/tuacvk/doq6pc

log4j-2.15-jar包，替换即可，最新的log4j的jar请到阿里云maven仓库下载

1. 替换log4j为log4j-1.15,1.15前的版本有漏洞
```shell
[root@ctos7mini mycat]& ll ./lib/log4j*
-rwxrwxrwx. 1 mycat mycat  489884 Sep 27  2019 ./lib/log4j-1.2.17.jar
-rwxrwxrwx. 1 mycat mycat   36947 Sep 27  2019 ./lib/log4j-1.2-api-2.5.jar
-rwxrwxrwx. 1 mycat mycat  146761 Sep 27  2019 ./lib/log4j-api-2.5.jar
-rwxrwxrwx. 1 mycat mycat 1106090 Sep 27  2019 ./lib/log4j-core-2.5.jar
-rwxrwxrwx. 1 mycat mycat   22935 Sep 27  2019 ./lib/log4j-slf4j-impl-2.5.jar
[root@ctos7mini mycat]#
```
删掉`log4j-1.2.17.jar`，  
替换lib文件夹上述的4个文件,替换为下面4个文件 

它们的下载地址是  
https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-1.2-api  
https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api  
https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core  
https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl  

这是缓存下来的打包好的,不保证最新    
http://dl.mycat.org.cn/1.6.7.6/20211221142218/log4j-2.17.zip


## MyCat故障解决
