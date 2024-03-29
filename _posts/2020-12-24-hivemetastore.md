---
layout: post
title: hivemetastore
categories: [数据生态, hive]
description: hivemetastore
keywords: 数据生态,hive
---

 <meta name="referrer" content="no-referrer"/>

```java
hive中metastore(元数据存储)的三种方式:
  内嵌Derby方式
  Local方式
  Remote方式

```

#### 内嵌 Derby

```java
hive的默认启动方式,一般用于单元测试
缺点:
在同一时间只能有一个进程(一个hive客户端使用数据库)连接使用数据库,测试使用,无法适用于生产环境


```

```java
hive-site.xml的配置

<configuration>
    <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:derby:;databaseName=metastore_db;create=true</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>org.apache.derby.jdbc.EmbeddedDriver</value>
    </property>
    <property>
      <name>hive.metastore.local</name>
      <value>true</value>
    </property>
    <property>
      <name>hive.metastore.warehouse.dir</name>
      <value>/user/hive/warehouse</value>
    </property>
</configuration>
```

#### 本地模式

```java
本地安装Mysql替代derby存储元数据,不再使用内嵌的derby作为元数据的存储介质,而是使用其他数据库比如mysql来存储元数据
hive服务和metastore服务运行在同一进程中, mysql是单独的进程, 可以是同一台机器也可以分开部署

```

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hive.metastore.warehouse.dir</name>  //数据库储存路径
    <value>/user/hive_remote/warehouse</value>
  </property>
  <property>
    <name>hive.metastore.local</name>  //本地模式
    <value>true</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>  //所连接的 MySQL 数据库的地址
    <value>jdbc:mysql://localhost/hive_local?createDatabaseIfNotExist=true</value>  //问号后固定格式防止定时任务报错
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>  // MySQL 驱动
    <value>com.mysql.jdbc.Driver</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>  //数据库账户
    <value>hive</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>  //数据库密码
    <value>password</value>
  </property>

</configuration>
```

#### 远程模式

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1640311495378-1b9e707b-2d8f-4ffe-94f6-7cb2b2f9ded9.png#clientId=u3dc1c14c-6f6e-4&from=paste&height=145&id=u8b46d8e7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=171&originWidth=561&originalType=binary&ratio=1&size=21044&status=done&style=none&taskId=ud0583afe-a009-4548-bbea-2717ebcc03a&width=475.5)

```xml
hive服务和metastore在不同的进程内, 可以在不同的机器上部署,比如 A机器部署 hive服务, B机器部署metastore服务

将hive.metastore.uris设置为metastore服务器URL
  <property>
   <name>hive.metastore.uris</name>
   <value>thrift://10.69.1.17:9083</value>
  </property>
远程元数据存储使用metastore服务, 每个客户端都是在配置文件里配置连接到该metastore服务。
将metadata作为一个单独的服务进行启动。各种客户端无需要知道数据库的密码即可连接。

所谓的远程模式, 只是metastore和hive服务是否在同一进程内, 而不仅是连接远程的mysql
```

```xml
服务端hive-site.xml配置
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
      <name>hive.metastore.warehouse.dir</name>
      <value>/user/hive/warehouse</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://192.168.1.214:3306/hive_remote?createDatabaseIfNotExist=true</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>hive</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>password</value>
    </property>
    <property>
      <name>hive.metastore.local</name> //但是在0.10 ，0.11或者之后的HIVE版本 hive.metastore.local 属性不再使用。
      <value>false</value>
    </property>
    <property>
      <name>hive.metastore.uris</name>
      <value>thrift://192.168.1.188:9083</value>
    </property>
</configuration>
```

```xml
客户端hive-site.xml配置
<configuration>

 <property>
   <!--  hive表的默认存储路径 -->
    <name>hive.metastore.warehouse.dir</name>
    <value>hdfs://flashHadoopUAT/user/hive/warehouse</value>
  </property>
  <property>
   <name>hive.metastore.uris</name>
   <value>thrift://10.69.1.17:9083</value>
  </property>
</configuration>

```

#### mysql 中元数据信息

[_mysql 中库表元数据信息_](https://blog.csdn.net/weixin_38924500/article/details/107021763?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.opensearchhbase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.opensearchhbase)

#### 元数据操作 API

```java
public class HiveMetaStoreTest {

    public static void main(String[] args) throws Exception {
        HiveConf hiveConf = new HiveConf();
        hiveConf.addResource("hive-site.xml");

        HiveMetaStoreClient client = new HiveMetaStoreClient(hiveConf);

        // 获取数据库信息
        List<String> dbs =  client.getAllDatabases();
        for(String db:dbs){
            System.out.println("db "+db);
        }
        // 获取库表信息
        List<String> tbs = client.getAllTables("iceberg");
        for(String tb:tbs){
            System.out.println("tb "+tb);
        }
        // 获取table的信息
        Table table = client.getTable("iceberg","user_msg_iceberg_a");
        System.out.println(JSON.toString(table));
    }
}
```
