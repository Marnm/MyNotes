---
title: 14MyCat垂直分表操作
tags: []
---

`schema.xml`

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

	<!-- 逻辑库配置 mycatdb1,mycatdb2,mycatdb3 是逻辑库并不是真的数据库-->
	
	<!-- 其中 checkSQLschema 表明是否检查并过滤 SQL 中包含 schema 的情况:
	如逻辑库为 mycatdb1，则可能写为 select * from mycatdb1.jakingtestdb11，
	此时会自动过滤 mycatdb1，SQL 变为 select * from jakingtestdb11，若不会出现上述写法，则可以关闭属性为 false	
	checkSQLschema="false"    select * from mycatdb1.jakingtestdb11;
	checkSQLschema="true"     select * from jakingtestdb11;   --> 

	<!-- sqlMaxLimit 默认返回的最大记录数限制，MyCat1.4 版本里面，用户的 Limit 参数会 覆盖掉 MyCat 的 sqlMaxLimit 默认设置-->
	

<!-- 每个逻辑库对应一个 schema ,每个schema 对应一个数据节点 -->
<schema name="mycatdb1" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb1"/>
<schema name="mycatdb2" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb2"/> 
<schema name="mycatdb3" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb3"/> 
<schema name="mycatdb4" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb4"/> 
<schema name="mycatdb5" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb5"/> 
<schema name="mycatdb6" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb6"/> 
<schema name="mycatdb7" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb7"/> 
<schema name="mycatdb8" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb8"/> 
<schema name="mycatdb9" checkSQLschema="false" sqlMaxLimit="100" dataNode="dnmycatdb9"/>


	<!-- 表分片配置在这里 --> 
	<!--</schema> --> 
	<!-- 定义 MyCat 的数据节点，节点配置 jakingtestdb01,jakingtestdb02,jakingtestdb03 才是真正的数据库， 
		dataNode 中的 name 数据表示节点名称， dataHost 表示数据主机名称， 
		database 表示该节点要路由的数据库的名称 -->

<!-- 设定数据结点 dnmycatdb1-3 对应的 192.168.10.12 服务 以及对应的物理 schema --> 
<dataNode name="dnmycatdb1" dataHost="jaking12" database="jakingtestdb01" /> 
<dataNode name="dnmycatdb2" dataHost="jaking12" database="jakingtestdb02" /> 
<dataNode name="dnmycatdb3" dataHost="jaking12" database="jakingtestdb03" />

<!-- 设定数据结点 dnmycatdb4-6 对应的 192.168.10.13 服务 以及对应的物理 schema --> 
<dataNode name="dnmycatdb4" dataHost="jaking13" database="jakingtestdb04" /> 
<dataNode name="dnmycatdb5" dataHost="jaking13" database="jakingtestdb05" /> 
<dataNode name="dnmycatdb6" dataHost="jaking13" database="jakingtestdb06" /> 

<!-- 设定数据结点 dnmycatdb7-9 对应的 192.168.10.14 服务 以及对应的物理 schema --> 
<dataNode name="dnmycatdb7" dataHost="jaking14" database="jakingtestdb07" /> 
<dataNode name="dnmycatdb8" dataHost="jaking14" database="jakingtestdb08" /> 
<dataNode name="dnmycatdb9" dataHost="jaking14" database="jakingtestdb09" />


	<!-- 读写分离的配置 -->
	<!-- dataHost 配置的是实际的后端数据库集群（当然，也可以是非集群） -->
	<!-- 注意： schema 中的每一个 dataHost 中的 host 属性值必须唯一，否则会出现主从在所有 dataHost 中全部切换的现象 -->
	<!-- 定义数据主机 dtHost1，只连接到 MySQL 读写分离集群中的 Master 节点，不使用 MyCat 托管 MySQL 主从切换 -->
	<!-- 
		<dataHost name="dtHost1" maxCon="500" minCon="20" balance="0" 
		writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100"> 
		<heartbeat>select user()</heartbeat> 
		<writeHost host="hostM1" url="192.168.10.63:3306" user="root" password="lyz" /> 
		</dataHost> 
	-->
	

<!-- 设定 192.168.10.12 目前只配了一台物理机，若要做读写分离可以参考上面的部分 --> 
<dataHost name="jaking12" maxCon="1000" minCon="10" balance="1" 
writeType="0" dbType="mysql" dbDriver="native"> 
<heartbeat>select user()</heartbeat> 
<!-- can have multi write hosts --> 
<writeHost host="192.168.10.12" url="192.168.10.12:3306" user="root" password="123456" /> 
</dataHost>

<dataHost name="jaking13" maxCon="1000" minCon="10" balance="1"
writeType="0" dbType="mysql" dbDriver="native"> 
<heartbeat>select user()</heartbeat> 
<!-- can have multi write hosts --> 
<writeHost host="192.168.10.13" url="192.168.10.13:3306" user="root" password="123456" /> 
</dataHost>

<dataHost name="jaking14" maxCon="1000" minCon="10" balance="1" 
writeType="0" dbType="mysql" dbDriver="native"> 
<heartbeat>select user()</heartbeat> 
<!-- can have multi write hosts --> 
<writeHost host="192.168.10.14" url="192.168.10.14:3306" user="root" password="123456" /> 
</dataHost> 

</mycat:schema>
```

`server.xml`

```xml
<!-- 开通root用户访问mycatdb访问权限 mycatdbx是虚拟的schema -->
<user name="root"> 
	<property name="password">123456</property> 
	<property name="schemas">mycatdb1,mycatdb2,mycatdb3,mycatdb4,mycatdb5,mycatdb6,mycatdb7,mycatdb8,mycatdb9</property> 
</user>

<!-- 开通user普通用户访问mycatdb访问权限 mycatdbx是虚拟的schema -->
<user name="user"> 
	<property name="password">123456</property> 
	<property name="schemas">mycatdb1,mycatdb2,mycatdb3,mycatdb4,mycatdb5,mycatdb6,mycatdb7,mycatdb8,mycatdb9</property> 
	<property name="readOnly">true</property> 
</user>
```
