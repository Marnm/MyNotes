---
title: 15MyCat全国省份分片枚举操作
tags: []
---

# MyCat分片枚举操作——全国省份分片

**机器**

|      IP       |    系统    |      角色         | 
|      ---      |    ---     |      ---         |
| 192.168.73.20 | CentOS 7.4 | MyCat Server     |
| 192.168.73.21 | RHEL 7.3   | MySQL Server 5.7 |
| 192.168.73.22 | CentOS 7.4 | MySQL Server 5.7 |
| 192.168.73.23 | RHEL 7.3   | MySQL Server 5.7 |

### 创建数据库

**192.168.73.21**
```sql
drop database jakingtestdb01;
drop database jakingtestdb02;
drop database jakingtestdb03;
create database jakingtestdb01 default character set utf8; 
create database jakingtestdb02 default character set utf8; 
create database jakingtestdb03 default character set utf8; 
create database jakingtestdb04 default character set utf8; 
create database jakingtestdb05 default character set utf8; 
create database jakingtestdb06 default character set utf8; 
create database jakingtestdb07 default character set utf8; 
create database jakingtestdb08 default character set utf8; 
create database jakingtestdb09 default character set utf8; 
create database jakingtestdb10 default character set utf8; 
create database jakingtestdb11 default character set utf8; 
create database jakingtestdb12 default character set utf8;
show databases;
```

**192.168.73.22**
```sql
drop database jakingtestdb04;
drop database jakingtestdb05;
drop database jakingtestdb06;
create database jakingtestdb13 default character set utf8; 
create database jakingtestdb14 default character set utf8; 
create database jakingtestdb15 default character set utf8; 
create database jakingtestdb16 default character set utf8; 
create database jakingtestdb17 default character set utf8; 
create database jakingtestdb18 default character set utf8; 
create database jakingtestdb19 default character set utf8; 
create database jakingtestdb20 default character set utf8; 
create database jakingtestdb21 default character set utf8; 
create database jakingtestdb22 default character set utf8; 
create database jakingtestdb23 default character set utf8; 
create database jakingtestdb24 default character set utf8;
show databases;
```

**192.168.73.23**
```sql
drop database jakingtestdb07;
drop database jakingtestdb08;
drop database jakingtestdb09;
create database jakingtestdb25 default character set utf8; 
create database jakingtestdb26 default character set utf8; 
create database jakingtestdb27 default character set utf8; 
create database jakingtestdb28 default character set utf8; 
create database jakingtestdb29 default character set utf8; 
create database jakingtestdb30 default character set utf8; 
create database jakingtestdb31 default character set utf8; 
create database jakingtestdb32 default character set utf8; 
create database jakingtestdb33 default character set utf8; 
create database jakingtestdb34 default character set utf8; 
create database jakingtestdb35 default character set utf8; 
create database jakingtestdb36 default character set utf8;
show databases;
```



### 配置MyCat服务

`server.xml`

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
        <property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为
关闭 -->
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

        <user name="root" defaultAccount="true">
                <property name="password">123456</property>
                <property name="schemas">mycatdb</property>

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

        <user name="user">
                <property name="password">ccc.456</property>
                <property name="schemas">mycatdb</property>
                <property name="readOnly">true</property>
        </user>

</mycat:server>
```

`schema.xml`

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

<schema name="mycatdb" checkSQLschema="false" sqlMaxLimit="100">
    <table name="jaking_member" dataNode="dnmycatdb$1-35" rule="sharding-by-intfile-mycatdb-jakingtest_member"></table>
</schema>


<!-- 设定数据结点 dnmycatdb1-dnmycatdb12 对应的 192.168.73.21 服务 以及对应的物理 schema -->
<dataNode name="dnmycatdb1" dataHost="192.168.73.21" database="jakingtestdb01" />
<dataNode name="dnmycatdb2" dataHost="192.168.73.21" database="jakingtestdb02" />
<dataNode name="dnmycatdb3" dataHost="192.168.73.21" database="jakingtestdb03" />
<dataNode name="dnmycatdb4" dataHost="192.168.73.21" database="jakingtestdb04" />
<dataNode name="dnmycatdb5" dataHost="192.168.73.21" database="jakingtestdb05" />
<dataNode name="dnmycatdb6" dataHost="192.168.73.21" database="jakingtestdb06" />
<dataNode name="dnmycatdb7" dataHost="192.168.73.21" database="jakingtestdb07" />
<dataNode name="dnmycatdb8" dataHost="192.168.73.21" database="jakingtestdb08" />
<dataNode name="dnmycatdb9" dataHost="192.168.73.21" database="jakingtestdb09" />
<dataNode name="dnmycatdb10" dataHost="192.168.73.21" database="jakingtestdb10" />
<dataNode name="dnmycatdb11" dataHost="192.168.73.21" database="jakingtestdb11" />
<dataNode name="dnmycatdb12" dataHost="192.168.73.21" database="jakingtestdb12" />


<!-- 设定数据结点 dnmycatdb13-dnmycatdb24 对应的 192.168.73.22 服务 以及对应的 物理 schema -->
<dataNode name="dnmycatdb13" dataHost="192.168.73.22" database="jakingtestdb13" />
<dataNode name="dnmycatdb14" dataHost="192.168.73.22" database="jakingtestdb14" />
<dataNode name="dnmycatdb15" dataHost="192.168.73.22" database="jakingtestdb15" />
<dataNode name="dnmycatdb16" dataHost="192.168.73.22" database="jakingtestdb16" />
<dataNode name="dnmycatdb17" dataHost="192.168.73.22" database="jakingtestdb17" />
<dataNode name="dnmycatdb18" dataHost="192.168.73.22" database="jakingtestdb18" />
<dataNode name="dnmycatdb19" dataHost="192.168.73.22" database="jakingtestdb19" />
<dataNode name="dnmycatdb20" dataHost="192.168.73.22" database="jakingtestdb20" />
<dataNode name="dnmycatdb21" dataHost="192.168.73.22" database="jakingtestdb21" />
<dataNode name="dnmycatdb22" dataHost="192.168.73.22" database="jakingtestdb22" />
<dataNode name="dnmycatdb23" dataHost="192.168.73.22" database="jakingtestdb23" />
<dataNode name="dnmycatdb24" dataHost="192.168.73.22" database="jakingtestdb24" />


<!-- 设定数据结点 dnmycatdb25-dnmycatdb35 对应的 192.168.73.23 服务 以及对应的 物理 schema -->
<dataNode name="dnmycatdb25" dataHost="192.168.73.23" database="jakingtestdb25" />
<dataNode name="dnmycatdb26" dataHost="192.168.73.23" database="jakingtestdb26" />
<dataNode name="dnmycatdb27" dataHost="192.168.73.23" database="jakingtestdb27" />
<dataNode name="dnmycatdb28" dataHost="192.168.73.23" database="jakingtestdb28" />
<dataNode name="dnmycatdb29" dataHost="192.168.73.23" database="jakingtestdb29" />
<dataNode name="dnmycatdb30" dataHost="192.168.73.23" database="jakingtestdb30" />
<dataNode name="dnmycatdb31" dataHost="192.168.73.23" database="jakingtestdb31" />
<dataNode name="dnmycatdb32" dataHost="192.168.73.23" database="jakingtestdb32" />
<dataNode name="dnmycatdb33" dataHost="192.168.73.23" database="jakingtestdb33" />
<dataNode name="dnmycatdb34" dataHost="192.168.73.23" database="jakingtestdb34" />
<dataNode name="dnmycatdb35" dataHost="192.168.73.23" database="jakingtestdb35" />

<!-- 设定 192.168.73.21 目前只配了一台物理机，若要做读写分离可以参考上面的部分 -->
<dataHost name="192.168.73.21" maxCon="1000" minCon="10" balance="1"
writeType="0" dbType="mysql" dbDriver="native">
<heartbeat>select user()</heartbeat>
<!-- can have multi write hosts -->
<writeHost host="192.168.73.21" url="192.168.73.21:3306" user="root" password="ccc.456" />
</dataHost>

<dataHost name="192.168.73.22" maxCon="1000" minCon="10" balance="1"
writeType="0" dbType="mysql" dbDriver="native">
<heartbeat>select user()</heartbeat>
<!-- can have multi write hosts -->
<writeHost host="192.168.73.22" url="192.168.73.22:3306" user="root" password="ccc.456" />
</dataHost>

<dataHost name="192.168.73.23" maxCon="1000" minCon="10" balance="1"
writeType="0" dbType="mysql" dbDriver="native">
<heartbeat>select user()</heartbeat>
<!-- can have multi write hosts -->
<writeHost host="192.168.73.23" url="192.168.73.23:3306" user="root" password="ccc.456" />
</dataHost>
</mycat:schema>
```

`rule.xml`

```xml
<!DOCTYPE mycat:rule SYSTEM "rule.dtd">
<mycat:rule xmlns:mycat="http://io.mycat/">

<tableRule name="sharding-by-intfile-mycatdb-jakingtest_member">
<rule>
    <columns>region</columns>
    <algorithm>hash-int</algorithm>
</rule>
</tableRule>

<function name="hash-int" class="io.mycat.route.function.PartitionByFileMap">
    <property name="mapFile">partition-hash-int-mycatdb-jakingtest_member.txt</property>
    <property name="type">1</property>
    <property name="defaultNode">0</property>
<!-- type 默认值为0，0表示integer，非零表示string,所有节点配置都是从0开始，及0代表节点1 -->
</function>
</mycat:rule>
```

`partition-hash-int-mycatdb-jakingtest_member.txt`

	vim /mysql/app/mycat/conf/partition-hash-int-mycatdb-jakingtest_member.txt
	
```
北京市=0
上海市=1
云南省=2
内蒙古=3
贵州省=4
重庆市=5
台湾省=6
吉林省=7
四川省=8
天津市=9
宁夏省=10
安徽省=11
山东省=12
山西省=13
广东省=14
广西省=15
新疆省=16
江苏省=17
江西省=18
河北省=19
河南省=20
浙江省=21
海南省=22
湖北省=23
湖南省=24
澳门=25
甘肃省=26
福建省=27
西藏=28
辽宁省=29
陕西省=30
青海省=31
香港=32
黑龙江省=33
DEFAULT_NODE=34
```

### 检查MyCat

```sql
mysql> show @@databases;
+----------+
| DATABASE |
+----------+
| mycatdb  |
+----------+
1 row in set (0.01 sec)

mysql> show @@datasource;
+-------------+---------------+-------+---------------+------+------+--------+------+------+---------+-----------+------------+
| DATANODE    | NAME          | TYPE  | HOST          | PORT | W/R  | ACTIVE | IDLE | SIZE | EXECUTE | READ_LOAD | WRITE_LOAD |
+-------------+---------------+-------+---------------+------+------+--------+------+------+---------+-----------+------------+
| dnmycatdb22 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb23 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb20 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb21 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb4  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb19 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb5  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb2  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb17 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb3  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb18 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb15 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb1  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb16 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb13 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb35 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb14 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb8  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb9  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb6  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb7  | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb11 | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb33 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb12 | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb34 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb31 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb10 | 192.168.73.21 | mysql | 192.168.73.21 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb32 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb30 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb28 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb29 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb26 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb27 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb24 | 192.168.73.22 | mysql | 192.168.73.22 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
| dnmycatdb25 | 192.168.73.23 | mysql | 192.168.73.23 | 3306 | W    |      0 |    0 | 1000 |       0 |         0 |          0 |
+-------------+---------------+-------+---------------+------+------+--------+------+------+---------+-----------+------------+
35 rows in set (0.01 sec)

mysql> show @@heartbeat;
+---------------+-------+---------------+------+---------+-------+--------+---------+--------------+---------------------+-------+
| NAME          | TYPE  | HOST          | PORT | RS_CODE | RETRY | STATUS | TIMEOUT | EXECUTE_TIME | LAST_ACTIVE_TIME    | STOP  |
+---------------+-------+---------------+------+---------+-------+--------+---------+--------------+---------------------+-------+
| 192.168.73.21 | mysql | 192.168.73.21 | 3306 |      -1 |     1 | idle   |   30000 | 14,2443,2443 | 2022-05-10 00:17:53 | false |
| 192.168.73.22 | mysql | 192.168.73.22 | 3306 |      -1 |     1 | idle   |   30000 | 15,2443,2443 | 2022-05-10 00:17:53 | false |
| 192.168.73.23 | mysql | 192.168.73.23 | 3306 |      -1 |     1 | idle   |   30000 | 16,2444,2444 | 2022-05-10 00:17:53 | false |
+---------------+-------+---------------+------+---------+-------+--------+---------+--------------+---------------------+-------+
3 rows in set (0.01 sec)

mysql>
```

### 导入数据库

注意：一定要把3个节点的MySQL 自动提交的参数打开！
```sql
set global autocommit=1;
show variables like "%autocommit%";
```

	mysql -h192.168.73.20 -P8066 -uroot -p123456 mycatdb < jakingtest_member.sql


### 查询数据

```sql
mysql> use mycatdb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables in mycatdb |
+-------------------+
| jaking_member     |
+-------------------+
1 row in set (0.00 sec)

mysql> select count(*) from jaking_member;
+--------+
| COUNT0 |
+--------+
|  50000 |
+--------+
1 row in set (0.03 sec)

mysql> select * from jaking_member where id = 20000210;
+----------+----------------+-----+-----+-------------+---------------------+-----------+-------------+------------------+--------------------------+------------------------------------------------------------------------------------------------------+
| ID       | NAME           | AGE | SEX | CARDID      | JOINDATE            | REGION    | TEL         | EMAIL            | RECOMMEND                | IDENTIFIER
                                         |
+----------+----------------+-----+-----+-------------+---------------------+-----------+-------------+------------------+--------------------------+------------------------------------------------------------------------------------------------------+
| 20000210 | jaking20000210 |  31 | 男  | 10050307335 | 2015-12-25 11:50:51 | 上海市    | 13108379727 | 123359043@qq.com | 彭理邦小鲜肉老师         | klbd muygjo xs iyz hqlghz duucn fjlap dgknrq kkz js ogsga hq cedyjm nzgmy unjpy kky jjh zevor juctq  |
+----------+----------------+-----+-----+-------------+---------------------+-----------+-------------+------------------+--------------------------+------------------------------------------------------------------------------------------------------+
1 row in set (0.02 sec)



-- 查询数据量
mysql> select table_schema,table_name as "Tables",ROUND(((data_length + index_length)/1024/1024), 2) "Size in MB" from information_schema.TABLES where TABLE_NAME = "jaking_member" order by (data_length + index_length) desc;
+----------------+---------------+------------+
| table_schema   | Tables        | Size in MB |
+----------------+---------------+------------+
| jakingtestdb01 | jaking_member |       1.52 |
| jakingtestdb02 | jaking_member |       0.41 |
| jakingtestdb05 | jaking_member |       0.39 |
| jakingtestdb03 | jaking_member |       0.39 |
| jakingtestdb11 | jaking_member |       0.39 |
| jakingtestdb08 | jaking_member |       0.39 |
| jakingtestdb07 | jaking_member |       0.39 |
| jakingtestdb06 | jaking_member |       0.38 |
| jakingtestdb04 | jaking_member |       0.38 |
| jakingtestdb12 | jaking_member |       0.38 |
| jakingtestdb09 | jaking_member |       0.38 |
| jakingtestdb10 | jaking_member |       0.36 |
+----------------+---------------+------------+
12 rows in set (0.01 sec)

mysql>
```

### 查看分片数据

**192.168.73.21**
```sql
mysql> select count(*) ,region from jakingtestdb10.jaking_member group by region;
+----------+-----------+
| count(*) | region    |
+----------+-----------+
|     1363 | 天津市    |
+----------+-----------+
1 row in set (0.01 sec)

mysql>
```
**192.168.73.22**
```sql
mysql> select count(*) ,region from jakingtestdb20.jaking_member group by region;
+----------+-----------+
| count(*) | region    |
+----------+-----------+
|     1448 | 河北省    |
+----------+-----------+
1 row in set (0.00 sec)
```
**192.168.73.23**
```sql
mysql> select count(*) ,region from jakingtestdb30.jaking_member group by region;
+----------+-----------+
| count(*) | region    |
+----------+-----------+
|     1448 | 辽宁省    |
+----------+-----------+
1 row in set (0.00 sec)

mysql>
```

## 故障

```mbox
2022-05-10 01:28:18.943  INFO [$_NIOREACTOR-0-RW] (io.mycat.backend.datasource.PhysicalDatasource$1$1.connectionError(PhysicalDatasource.java:508)) - connection connectionError
2022-05-10 01:28:18.944  INFO [$_NIOREACTOR-0-RW] (io.mycat.net.AbstractConnection.close(AbstractConnection.java:520)) - close connection,reason:stream closed ,MySQLConnection [id=53, lastTime=1652117298929, user=root, schema=jakingtestdb04, old shema=jakingtestdb04, borrowed=false, fromSlaveDB=false, threadId=634, charset=utf8, txIsolation=3, autocommit=true, attachment=null, respHandler=null, host=192.168.73.21, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]
2022-05-10 01:28:18.944  INFO [$_NIOREACTOR-0-RW] (io.mycat.backend.datasource.PhysicalDatasource$1$1.connectionError(PhysicalDatasource.java:508)) - connection connectionError
2022-05-10 01:28:18.944  INFO [$_NIOREACTOR-0-RW] (io.mycat.net.AbstractConnection.close(AbstractConnection.java:520)) - close connection,reason:stream closed ,MySQLConnection [id=54, lastTime=1652117298929, user=root, schema=jakingtestdb35, old shema=jakingtestdb35, borrowed=false, fromSlaveDB=false, threadId=566, charset=latin1, txIsolation=3, autocommit=true, attachment=null, respHandler=null, host=192.168.73.23, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]
2022-05-10 01:28:18.944  INFO [$_NIOREACTOR-0-RW] (io.mycat.backend.datasource.PhysicalDatasource$1$1.connectionError(PhysicalDatasource.java:508)) - connection connectionError
```
