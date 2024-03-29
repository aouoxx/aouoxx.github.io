---
layout: post
title: clickhouse的安装和运维
categories: [olap, clickhouse]
description: clickhouse的安装和运维
keywords: clickhouse
---

 <meta name="referrer" content="no-referrer"/>

## clickhouse 安装

### 安装前准备

```java
安装前的准备
centos取消打开文件的限制
    在/etc/security/limits.conf /etc/security/limites.d/90-nproc.conf在这两个文件的末尾加入内容
    vim /etc/security/limits.conf
       在文件末尾添加
       * soft nofile 65536
       * soft nofile 65536  --最大文件数
       * soft nproc 131072
       * soft nproc 131072  --最大进程数
     * 表示所有的框架


   查看是否生效
     ssgao:~ aouo$ ulimit -n
     2560


centos取消selinux
    修改/etc/selinux/config 中的SELINUX=disabled 后重启
    vim /etc/selinux/config
    SELINUX=disabled

关闭防火墙
    service iptables stop

安装依赖
   yum install -y libtool
   yum install -y *unixODBC*

```

```java


clickhouse的安装


命令行启动
root@dsp-bridge1:/etc/init.d# ./clickhouse-server stop
root@dsp-bridge1:/etc/init.d#
root@dsp-bridge1:/etc/init.d# ./clickhouse-server staus
Usage: ./clickhouse-server {start|stop|status|restart|forcestop|forcerestart|reload|condstart|condstop|condrestart|condreload|initdb}
root@dsp-bridge1:/etc/init.d# ./clickhouse-server status
clickhouse-server service is stopped



gaoshuoshuo@dsp-bridge1:~$ clickhouse-client
ClickHouse client version 20.5.4.40 (official build).
Connecting to localhost:9000 as user default.
Code: 516. DB::Exception: Received from localhost:9000. DB::Exception: default: Authentication failed: password is incorrect or there is no user with such name.

gaoshuoshuo@dsp-bridge1:~$ clickhouse-client --password
ClickHouse client version 20.5.4.40 (official build).
Password for user (default):
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 20.5.4 revision 54435.

dsp-bridge1.hz.163.org :)
dsp-bridge1.hz.163.org :)
dsp-bridge1.hz.163.org :) show tables;

SHOW TABLES

Ok.

0 rows in set. Elapsed: 0.002 sec.

dsp-bridge1.hz.163.org :)
dsp-bridge1.hz.163.org :)




clickhouse-client -m 可以支持多行SQL；

root@dsp-bridge1:/etc/init.d# ./clickhouse-server --help
Usage: ./clickhouse-server {start|stop|status|restart|forcestop|forcerestart|reload|condstart|condstop|condrestart|condreload|initdb}
root@dsp-bridge1:/etc/init.d#
root@dsp-bridge1:/etc/init.d# ./clickhouse-server status
clickhouse-server service is running



clickhouse配置文件设置
----------------------------------------------------------------------------------------------
root@dsp-bridge1:/var/log/clickhouse-server# cd /etc/clickhouse-server/
root@dsp-bridge1:/etc/clickhouse-server# ls -l
total 48
drwxrwxrwx 2 root root  4096 Aug  3 16:01 config.d
-rwxrwxrwx 1 root root 28822 Aug  3 17:02 config.xml
lrwxrwxrwx 1 root root    41 Aug  3 16:01 preprocessed -> /var/lib/clickhouse//preprocessed_configs
drwxrwxrwx 2 root root  4096 Aug  3 16:01 users.d
-rwxrwxrwx 1 root root  5328 Aug  1 18:46 users.xml

放开远程访问配置
    <!-- Listen specified host. use :: (wildcard IPv6 address),
           if you want to accept connections both with IPv4 and IPv6 from everywhere. -->
    <!-- <listen_host>::</listen_host> -->
    <!-- Same for hosts with disabled ipv6: -->
    <!-- <listen_host>0.0.0.0</listen_host> -->

修改日志路径
    <logger>
        <!-- Possible levels: https://github.com/pocoproject/poco/blob/poco-1.9.4-release/Foundation/include/Poco/Logger.h#L105 -->
        <level>trace</level>
        <log>/home/gaoshuoshuo/click_data/logs/clickhouse-server.log</log>
        <errorlog>/home/gaoshuoshuo/click_data/logs/clickhouse-server.err.log</errorlog>
        <size>1000M</size>
        ...
    </logger>

配置clickhouse的数据目录

    <!-- Path to data directory, with trailing slash. -->
    <path>/var/lib/clickhouse/</path>

    <!-- Path to temporary data for processing hard queries. -->
    <tmp_path>/var/lib/clickhouse/tmp/</tmp_path>


http_port/https_port：通过HTTP连接到服务器的端口。
    <http_port>8123</http_port>


http_server_default_response：
    访问ClickHouse HTTP服务器时默认显示的页面。默认值为“OK”
    <http_server_default_response>
      <![CDATA[<html ng-app="SMI2"><head><base href="http://ui.tabix.io/"></head><body><div ui-view="" class="content-ui"></div><script src="http://loader.tabix.io/master.js"></script></body></html>]]>
    </http_server_default_response>


启动指定配置文件

clickhouse-server --config-file=/etc/clickhouse-server/config.xml





Tabix安装
官方提供了五种安装方式，这里我们介绍下Embedded和Local这两种方式
Embedded
这种方式使用的clickhouse内置的服务，直接打开config.xml中http_server_default_response标签的注释就行

<http_server_default_response>
    <![CDATA[<html ng-app="SMI2">
        <head>
            <base href="http://ui.tabix.io/">
                </head><body><div ui-view="" class="content-ui"></div><script src="http    ://loader.tabix.io/master.js">
                </script></body></html>]]>
</http_server_default_response>
访问方式：
http://clickhouse:8123
使用默认的用户名default,密码不填，直接为空




Error on load database structure
Code: 516, e.displayText() = DB::Exception:
 Default: Authentication failed: password is incorrect or there is no user with such name (version 20.5.4.40 (official build))





```

### 单机配置参数

```java
<?xml version="1.0"?>
<yandex>
   <!-- 日志 -->
   <logger>
       <level>trace</level>
       <log>/data1/clickhouse/log/server.log</log>
       <errorlog>/data1/clickhouse/log/error.log</errorlog>
       <size>1000M</size>
       <count>10</count>
   </logger>

   <!-- 端口 -->
   <http_port>8123</http_port>
   <tcp_port>9000</tcp_port>
   <interserver_http_port>9009</interserver_http_port>

   <!-- 本机域名 -->
   <interserver_http_host>这里需要用域名，如果后续用到复制的话</interserver_http_host>

   <!-- 监听IP -->
   <listen_host>0.0.0.0</listen_host>
   <!-- 最大连接数 -->
   <max_connections>64</max_connections>

   <!-- 没搞懂的参数 -->
   <keep_alive_timeout>3</keep_alive_timeout>

   <!-- 最大并发查询数 -->
   <max_concurrent_queries>16</max_concurrent_queries>

   <!-- 单位是B -->
   <uncompressed_cache_size>8589934592</uncompressed_cache_size>
   <mark_cache_size>10737418240</mark_cache_size>

   <!-- 存储路径 -->
   <path>/data1/clickhouse/</path>
   <tmp_path>/data1/clickhouse/tmp/</tmp_path>

   <!-- user配置 -->
   <users_config>users.xml</users_config>
   <default_profile>default</default_profile>

   <log_queries>1</log_queries>

   <default_database>default</default_database>



   <remote_servers incl="clickhouse_remote_servers" />
   remote_servers: 远程服务器，分布式表引擎和集群表功能使用的集群的配置。
                   使用说明：分布式集群的配置metrika.xml中使用

   ClickHouse与ZooKeeper群集进行交互的设置。使用复制表时，ClickHouse使用ZooKeeper来存储副本的元数据。如果不使用复制表，则可以忽略此参数。
   <zookeeper incl="zookeeper-servers" optional="true" />

    复制表的参数替换，如果不使用复制表，则可以省略。
   <macros incl="macros" optional="true" />

   <!-- 重新加载内置词典的时间间隔（以秒为单位），默认3600。可以在不重新启动服务器的情况下“即时”修改词典。 -->
   <builtin_dictionaries_reload_interval>3600</builtin_dictionaries_reload_interval>

   <!-- 控制大表的删除 -->
   <max_table_size_to_drop>0</max_table_size_to_drop>

   <include_from>/data1/clickhouse/metrika.xml</include_from>
</yandex>

```

### 线上配置

```java

iad-kafka1.jd.163.org :) select * from system.clusters;

SELECT *
FROM system.clusters

┌─cluster─────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name─────────────┬─host_address─┬─port─┬─is_local─┬─user─┬─default_database─┬─errors_count─┬─estimated_recovery_time─┐
│ iad_cluster │         1 │            1 │           1 │ iad-kafka1.jd.163.org │ 10.196.8.37  │ 9000 │        1 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         1 │            1 │           2 │ iad-kafka2.jd.163.org │ 10.196.8.36  │ 9001 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         2 │            1 │           1 │ iad-kafka2.jd.163.org │ 10.196.8.36  │ 9000 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         2 │            1 │           2 │ iad-kafka3.jd.163.org │ 10.196.8.35  │ 9001 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         3 │            1 │           1 │ iad-kafka3.jd.163.org │ 10.196.8.35  │ 9000 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         3 │            1 │           2 │ iad-kafka4.jd.163.org │ 10.196.8.34  │ 9001 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         4 │            1 │           1 │ iad-kafka4.jd.163.org │ 10.196.8.34  │ 9000 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         4 │            1 │           2 │ iad-kafka5.jd.163.org │ 10.196.8.33  │ 9001 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         5 │            1 │           1 │ iad-kafka5.jd.163.org │ 10.196.8.33  │ 9000 │        0 │ ad   │                  │            0 │                       0 │
│ iad_cluster │         5 │            1 │           2 │ iad-kafka1.jd.163.org │ 10.196.8.37  │ 9001 │        0 │ ad   │                  │            0 │                       0 │
└─────────────┴───────────┴──────────────┴─────────────┴───────────────────────┴──────────────┴──────┴──────────┴──────┴──────────────────┴──────────────┴─────────────────────────┘

```

```java
clickhouse高可用配置主要用到metrika.xml 默认路径 /etc/metrika.xml

<yandex>
    <!--配置分片信息 -->
    <clickhouse_remote_servers>
        <cluster_3shards_2replicas>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>dsp-bridge1.hz.163.org</host>
                    <port>9401</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
                <replica>
                    <host>dsp-bridge2.hz.163.org</host>
                    <port>9402</port>
                    <user>default</user>
                    <password>clickhouse</password>
                 </replica>
            </shard>

            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>dsp-bridge3.hz.163.org</host>
                    <port>9402</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
                <replica>
                    <host>dsp-bridge2.hz.163.org</host>
                    <port>9401</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
            </shard>

             <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>dsp-bridge3.hz.163.org</host>
                    <port>9401</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
                <replica>
                    <host>dsp-bridge1.hz.163.org</host>
                    <port>9402</port>
                     <user>default</user>
                    <password>clickhouse</password>
                </replica>
            </shard>
        </cluster_3shards_2replicas>
    </clickhouse_remote_servers>
    <!-- 配置分片信息 -->
    <zookeeper-servers>
        <node index="1">
            <host>dsp-bridge1.hz.163.org</host>
            <port>2182</port>
        </node>
        <node index="2">
            <host>dsp-bridge2.hz.163.org</host>
            <port>2182</port>
        </node>
        <node index="3">
            <host>dsp-bridge3.hz.163.org</host>
            <port>2182</port>
        </node>
    </zookeeper-servers>
    <!-- 配置宏参数 -->
    <macros>
        <layer>01</layer>
        <shard>02</shard>
        <replica>dsp-bridge2.hz.163.org_9401</replica>
    </macros>

    <networks>
        <ip>::/0</ip>
    </networks>
    <!--  配置压缩算法 -->
    <clickhouse_compression>
        <case>
          <min_part_size>10000000000</min_part_size>
          <min_part_size_ratio>0.01</min_part_size_ratio>
          <method>lz4</method>
        </case>
    </clickhouse_compression>
</yandex>





参数: internal_replication属性,这个属性是true或者false,默认是false;
    如果设置为


gaoshuoshuo@dsp-bridge1:~$ clickhouse-client --password -hdsp-bridge1.hz.163.org --port 9401 -m
ClickHouse client version 20.5.4.40 (official build).
Password for user (default):
Connecting to dsp-bridge1.hz.163.org:9401 as user default.
Connected to ClickHouse server version 20.5.4 revision 54435.

dsp-bridge1.hz.163.org :) select * from system.clusters where cluster='cluster_3shards_2replicas';

SELECT *
FROM system.clusters
WHERE cluster = 'cluster_3shards_2replicas'

┌─cluster───────────────────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name──────────────┬─host_address──┬─port─┬─is_local─┬─user────┬─default_database─┬─errors_count─┬─estimated_recovery_time─┐
│ cluster_3shards_2replicas │         1 │            1 │           1 │ dsp-bridge1.hz.163.org │ 10.130.46.202 │ 9401 │        1 │ default │                  │            0 │                       0 │
│ cluster_3shards_2replicas │         1 │            1 │           2 │ dsp-bridge2.hz.163.org │ 10.130.46.203 │ 9402 │        0 │ default │                  │            0 │                       0 │
│ cluster_3shards_2replicas │         2 │            1 │           1 │ dsp-bridge3.hz.163.org │ 10.130.46.204 │ 9402 │        0 │ default │                  │            0 │                       0 │
│ cluster_3shards_2replicas │         2 │            1 │           2 │ dsp-bridge2.hz.163.org │ 10.130.46.203 │ 9401 │        0 │ default │                  │            0 │                       0 │
│ cluster_3shards_2replicas │         3 │            1 │           1 │ dsp-bridge3.hz.163.org │ 10.130.46.204 │ 9401 │        0 │ default │                  │            0 │                       0 │
│ cluster_3shards_2replicas │         3 │            1 │           2 │ dsp-bridge1.hz.163.org │ 10.130.46.202 │ 9402 │        0 │ default │                  │            0 │                       0 │
└───────────────────────────┴───────────┴──────────────┴─────────────┴────────────────────────┴───────────────┴──────┴──────────┴─────────┴──────────────────┴──────────────┴─────────────────────────┘

6 rows in set. Elapsed: 0.002 sec.




dsp-bridge1.hz.163.org :) CREATE DATABASE IF NOT EXISTS myclick_house on cluster cluster_3shards_2replicas;
CREATE DATABASE IF NOT EXISTS myclick_house ON CLUSTER cluster_3shards_2replicas
┌─host───────────────────┬─port─┬─status─┬─error─┬─num_hosts_remaining─┬─num_hosts_active─┐
│ dsp-bridge3.hz.163.org │ 9401 │      0 │       │                   5 │                0 │
│ dsp-bridge3.hz.163.org │ 9402 │      0 │       │                   4 │                0 │
│ dsp-bridge2.hz.163.org │ 9401 │      0 │       │                   3 │                0 │
│ dsp-bridge2.hz.163.org │ 9402 │      0 │       │                   2 │                0 │
│ dsp-bridge1.hz.163.org │ 9401 │      0 │       │                   1 │                0 │
│ dsp-bridge1.hz.163.org │ 9402 │      0 │       │                   0 │                0 │
└────────────────────────┴──────┴────────┴───────┴─────────────────────┴──────────────────┘


```

### 参考

> [_https://www.jianshu.com/p/20639fdfdc99_](https://www.jianshu.com/p/20639fdfdc99)

## 权限管理

```java
profile
    用户角色,可以实现最大内存,负载方式等配置的复用
users
    设置包括用户名,密码,权限等
quotas
    限制一段时间内的资源使用等
```

### users.xml

```java

root@dsp-bridge1:/etc/clickhouse-server# vim users.xml
<?xml version="1.0"?>
<yandex>
    <profiles>
        <!-- default组是必须的 -->
        <default>
            <max_memory_usage>10000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <load_balancing>random</load_balancing>
        </default>
        <!--自定义readonly角色-->
        <readonly>
            <readonly>1</readonly>
        </readonly>
    </profiles>

    <!-- Users and ACL. -->
    <users>
        <default>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
        </default>
        <myclickhouse>
            <password>myclickhouse</password>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
        </myclickhouse>
    </users>

    <quotas>
        <default>
            <interval>
                <duration>3600</duration>
                <queries>0</queries>
                <errors>0</errors>
                <result_rows>0</result_rows>
                <read_rows>0</read_rows>
                <execution_time>0</execution_time>
            </interval>
        </default>
    </quotas>
</yandex>



user.xml配置文件中的quota标签是限制了单位之间的系统资源使用量,而不是限制单个查询的系统资源使用量。
    (server的配置可以设置限制单个查询的系统资源使用量)
    值为0表示不限制,如下面示例所示,
    表示仅跟踪每小时的资源消耗,而不限制使用



quotas属性详情

duration设置
   duration表示累计时间周期,单位为秒,达到该时间周期后,清除所有收集的值,接下来的周期将重新开始计算,当服务重启时,也会清除所有的值,重新开始新的周期
    <duration>3600</duration>
queris设置
    quries表示在该周期内,允许执行的查询次数,0为不限制
    <!--在duration设置周期时间内只允许查询1000次-->
    <queries>1000</queries>

errors设置
    errors表示在该周期内,允许引发异常的查询次数,0为不限制
    <errors>0</errors>

result_rows设置
    result_rows表示在周期内，允许查询返回的结果行数，0为不限制。
    <result_rows>0</result_rows>

read_rows设置
    read_rows表示在周期内,允许远程节点读取的数据行数，0表示不限制
    <read_rows>0</read_rows>

execution_time设置
    execution_time表示允许查询的总执行时间(又叫wall time)单位秒, 0表示不限制。
    <exection_time>0</exection_time>



```

### profile

```java
profile
  profile是一组设置的集合,类似于角色的概念,每个用户都有一个profile
  可以有两种方式使用profile
  在命令行行设置: set profile='xxxx'
  在user.xml的user部分为某个用户指定profile。profile在users.xml配置文件的profiles标签下配置


```

```java
<profiles>
        <!-- 读写用户设置  -->
        <default>
            <max_memory_usage>40000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <load_balancing>random</load_balancing>
        </default>

        <!-- 只写用户设置  -->
        <readonly>
            <max_memory_usage>40000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <load_balancing>random</load_balancing>
            <readonly>2</readonly>
        </readonly>

         <web>
            <max_rows_to_read>1000000000</max_rows_to_read>
            <max_bytes_to_read>1000000000</max_bytes_to_read>

            在profile部分定义对设置的约束,禁止用户使用set语句更改某些设置
            如果用户尝试修改,则会引发异常
            支持三种类型的约束, min,max(使用数值约束的上限和下限) 和readonly 指定了用户无法更改响应的设置
            <constraints>
                <max_memory_usage>
                    <min>500000000</min>
                    <max>20000000000</max>
                </max_memory_usage>
                <force_index_by_date>  属性的含义是强制使用date类型来进行索引,查询是必须添加日期类型
                    <readonly />
                </force_index_by_date>
            </constraints>
         </web>

        <readonly_profile>  使用该profile --> set profile=readonly_profile
            <readonly>1</readonly>只读
        </readonly_profile>

        <limit_rows>  profile的嵌套,继承
            <profile>readonly_profile</profile>
            <max_columns_to_read>25</max_columns_to_read>
        </limit_rows>



    </profiles>
```
