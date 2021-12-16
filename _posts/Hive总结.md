### _hive 的概述_

**_hive 是构建在 HDFS 上的数据仓库_**

```
Hive是基于hadoop的一个数据仓库工具,可以将结构化的数据文件映射为一张数据库表,并提供简单的sql查询功能,可以将sql语句转换为MapReduce任务进行运行。其优点可以通过类SQL语句快速实现简单的MapReduce统计,不必开发专门的MapReduce应用,十分适合数据仓库的统计分析。Hive是一个数据仓库基础工具在Hadoop中用来处理结构化数据。它架构在Hadoop之上,总归为大数据,并使得查询和分析更加方便。
    在Hive中,Hive是SQL的解析引擎,它将SQL语句转译为M/R Job然后在hadoop中执行。Hive的表其实就是HDFS的目录/文件,按表名把文件夹分开。如果是分区表,则分区值是子文件夹,可以直接在M/R job中使用这些数据。
关系数据库里, 表的加载模式是在加载时强制确定的(表的介绍模式是指数据库存储数据的文件格式)
    如果加载数据时,发现加载的数据不符合模式,关系数据库则会拒绝加载数据,这个就叫"写模式",写模式会在数据加载时候对数据模型进行校验的操作.
    hive在加载数据时和关系数据库不同,hive在加载数据时不会对数据进行检查,也不会更改被加载的数据文件,而检查数据格式的操作是在查询操作时执行,
这种模式叫'读时模式'.
    在实际应用中,写模式在加载数据的时候会对列进行索引,对数据进行压缩,因此加载数据的速度很慢,但是当数据加载好了,我们查询数据的时候,速度很快.
    但是当我们的数据是非结构化,存储模式也是未知的时候,关系数据这种场景加麻烦多了,这时候hive就会发挥它的优势
什么是Hive
  Hive 是FaceBook开源用于解决还想结构化日志的数据统计
  Hive 是基于hadoop是一个数据仓库工具,可以将结构化数据文件映射为一张表,并提供类SQL查询功能
本质 "将HQL转换为MapReduce程序"

Hive可以理解为hadoop的一个客户端,安装一台就可以了,其本身没有集群的概念
```

### _hive 的系统架构_

```
hive的系统架构图
    用户接口cli   CLI即shell命令行
    JDBC/ODBC   Hive的java与传统数据库JDBC的方式类似
    WebGUI   通过浏览器访问Hive
    解释器,编译器,优化器完成HQL查询语句从词法分析,语法分析,编译,优化以及查询计划(plan)的生成。生成的查询计划存储在HDFS中,并随后又MapReduce调用执行。
    Hive的数据存储在HDFS中,大部分的查询由MapReduce完成( 包含*的查询,比如select * from table不会生成MR任务)
    Hive将元数据存储在数据库中(metastore),目前只支持mysql，derby.
    Hive中的元数据包括表的名字,表的列和分区以及其属性,表的属性(是否为外部表等),表的数据所在目录等.
```

##### _metadata 组件_

```
Hive的Metastore组件是hive元数据集中存放处。Metastor组件包含两部分
```

#### _hive 的优缺点_

```
优点
   1) 操作接口采用类SQL语法,提供快速开发能力(简单,容易上手)
   2) 避免了去写MapReduce,减少开发人员的学习成本
   3) Hive执行的延时比较高,因此Hive常用于数据分析,对实时性要求不高的场合
   4) Hive优势在于处理大数据,对处理小数据没有优势,因为Hive的执行延迟比较高
   5) Hive支持用户自定义函数,用户可以自己根据自己的需求来实现自己的函数
缺点
   1) Hive的 hql表达能力有限
     a) 迭代式算法无法表达
     b) 数据挖掘方面不擅长
   2) Hive的效率比较低
     a) Hive自动生成MapReduce作业,通常情况下不够智能化
     b) Hive调优比较困难,粒度较粗
```

### _hive 和 mysql 的对比_

```
由于Hive采用了类似SQL的查询语言HQL(Hive Query Language),因此很容易将Hive理解为数据库。
其实从结构上来看,hive和数据库处理拥有类似的查询语言,再无类似之处。
下面从几个方面介绍hive和myql的差异,数据库可以用在online的应用中,但是hive是为数据仓库设计的,清除这一点,有利于从应用角度理解hive的特性。
```

- **_查询语言_**

```
由于SQL被广泛的应用在数据仓库中,因此,专门针对hive的特性设计了类SQL的查询语言HQL。
熟悉SQL开发的开发者可以很方便的使用Hive进行开发
```

- **_数据存储位置_**

```
hive是建立hadoop之上的,所有hive的数据都是存储在hdfs中的。而数据库则可以将数据保存在块设备或者本地文件系统中
```

- **_数据更新_**

```
由于hive是针对数据仓库设计的,而数据仓库的内容是读多,写少的。因此,hive中不建议对数据的改写,所有的数据都是在加载的时候确定好的
而数据库中的数据通常是需要经常修改的,因此可以使用Insert into ...values添加数据,使用update...set修改数据
```

- **_索引_**

```
hive在加载数据过程中不会对数据进行任何处理,甚至不会数据进行扫描,因此也没有对数据中的某些key建立索引。
Hive要访问数据中满足条件的特定值时,需要暴力扫描整个数据,因此访问延迟较高。
由于MapReduce的引入,hive可以并行访问数据,因此即便没有索引,对于大数据量访问,Hive仍然可以体现出优势。
数据库中,通常会针对一个或者几个建立索引,因此对于少量的特定条件的数据访问,数据库可以有很高的效率,较低的延迟。
由于数据的访问延迟较高,决定了Hive不适合在线数据查询
```

- **_执行_**

```
Hive中大多数查询的执行时通过hadoop提供的MapReduce来实现的。而数据库通常有自己的执行引擎
```

- **_执行延迟_**

```
Hive在查询数据的时候,由于没有索引,需要扫描整个表,因此延迟较高。
另外一个导致Hive执行延迟高的因素是MapReduce框框。由于MapReduce本身具有较高的延迟,因此在利用MapReduce执行Hive查询时,也会有较高的延迟。
相对的,数据库的执行延迟较低.当然,这个低是有条件的,即数据规模下,当数据规模大到超过数据库处理能力的时候,Hive的并行计算显然能体现出优势。
```

- **_可扩展性_**

```
由于Hive是建立在Hadoop之上的,因此Hive可扩展性是和Hadoop的可扩展性是一致的。
而数据库由于ACID语义的严格限制,扩展行非常有效。目前最先进的并行数据库Oracle理论上扩展能力也只有100台左右
```

- **_数据规模_**

```
由于hive建立在集群上可以利用hadoop的MapReduce进行并行计算,因此可以支持很大规模的数据,对应的,数据库可以支持的数据规模较小
```

### _hive 的安装_

```
hive默认将元素保存在本地内嵌的Derby数据库中,Derby不支持多回话连接,所以本文将选择mysql作为元数据存储
下载mysql的jdbc<mysql-connector-java-5.1.28.jar>放到hive安装包的lib目录下
```

#### _安装步骤_

```
1) 下载文件
http://mirror.bit.edu.cn/apache/hive/
2） 设置环境变量
export HIVE_HOME=/root/software/apache-hive-2.3.4-bin
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$REDIS_HOME:$MYSQL_HOME:$HIVE_HOME/bin
[root@master apache-hive-2.3.4-bin]# hive --version
Hive 2.3.4
Git git://daijymacpro-2.local/Users/daijy/commit/hive -r 56acdd2120b9ce6790185c679223b8b5e884aaf2
Compiled by daijy on Wed Oct 31 14:20:50 PDT 2018
From source with checksum 9f2d17b212f3a05297ac7dd40b65bab0
3） 修改配置文件
[root@master conf]# pwd
/root/software/apache-hive-2.3.4-bin/conf
[root@master conf]# cp hive-default.xml.template hive-site.xml
4） 配置文件 如下
5） 复制mysql的驱动到hive/lib下面
6)  定义hive日志信息
[root@master conf]# cp hive-log4j2.properties.template hive-log4j2.properties
property.hive.log.dir = /root/software/apache-hive-2.3.4-bin/logs
7)  修改hive-env.sh的信息
    HADOOP_HOME=/root/software/hadoop-2.7.4-cluster
8） 在mysql的hive schema(在此之前需要创建mysql下的hive数据库)
[root@master bin]#
[root@master bin]# schematool -dbType mysql -initSchema
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/software/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/root/software/hadoop-2.7.4-cluster/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:    jdbc:mysql://192.168.10.140:3306/hive?createDatabaseIfNotExist=true&characterEncoding=UTF-8
Metastore Connection Driver :    com.mysql.jdbc.Driver
Metastore connection User:   root
Starting metastore schema initialization to 2.3.0
Initialization script hive-schema-2.3.0.mysql.sql
Initialization script completed
schemaTool completed
9） 安装成功执行hive命令
[root@master bin]# hive
...(省略)...
hive> show databases;
OK
default
Time taken: 4.909 seconds, Fetched: 1 row(s)
hive> use default;
OK
Time taken: 0.018 seconds
hive> exit;
[root@master bin]#
```

#### _hive 的配置文件_

```
conf/hive-site.xml 文件
<configuration>
     <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
     </property>
     <property>
            <name>javax.jdo.option.ConnectionPassword</name>
            <value>ssgao</value>
     </property>
     <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://192.168.10.140:3306/hive1?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8</value>
     </property>
     <property>
            <name>javax.jdo.option.ConnectionDriverName</name>
            <value>com.mysql.jdbc.Driver</value>
     </property>

   <!---在这之前插入---->
   <property>
    <name>hive.exec.script.wrapper</name>
    <value/>
    <description/>
  </property>
</configuration>
javax.jdo.option.ConnectionURL 配置Hive元数据库链接信息
javax.jdo.option.ConnectionDriverName 配置链接Hive元数据库驱动类
javax.jdo.option.ConnectionUserName 配置链接Hive元数据库用户名
javax.jdo.option.ConnectionPassword 配置链接hive元数据密码
javax.jdo.option.Multithreaded 配置是否支持并发访问metastore
hive.metastore.local 配置metastore服务与元数据库主机是否保持一致
hive.map.aggr map端聚合是否开启(默认开启)
hive.exec.compress.output 配置mapreduce任务最终输出结果是否压缩(默认关闭,一般设置为开启,可以节省存储)
hive.intermediate.compression.codec 在开启压缩的时候指定,压缩格式
hive.exec.max.created.files 配置单个mapreduce作业能创建的hdfs文件最大数
```

```
默认情况下,Hive元数据保存在内嵌的Derby数据库中,只能允许一个回话连接,只适合简单的测试。
实际情况下生产环境中不使用,为了支持"多用户会话",则需要一个独立的元数据库,使用mysql作为元数据库,Hive内部对Mysql提供了很好的支持
参数配置方式
1) 查看当前所有的配置信息
    hive> set;
2) 参数的配置三种方式
   a) 配置文件方式
        默认配置文件: hive-default.xml
        用户自定义配置文件: hive-site.xml
        注意,用户自定义配置会覆盖默认配置,另外hive也会读入hadoop的配置,因为hive是作为hadoop的客户端启动的,hive的配置会覆盖hadoop的配置
        配置文件的设定对本机启动的多有hive进行都有效。
   b) 命令行参数方式
      启动hive时,可以在命令行添加-hiveconf param=value来设定参数
      bin/hive -hiveconf mapred.reduce.tasks=10; (仅对本次hive启动有效)
      查看参数设置
      hive> set mapred.reduce.tasks
   c) 参数声明方式
      可以在HQL中使用set关键字设定参数
      hive> set mapred.reduce.tasks=100; (仅对本次hive启动有效)
      参看参数设置
      hive> set mapred.reduce.tasks;
上述三种方式的优先级依次递增,即配置文件<命令行参数<参数声明。注意某些系统级参数,例如log4j相关的设定必须用前两种方式设定,因为那些参数的读取在会话建立之前已经完成了。
```

#### _hive 的属性介绍_

```
Hive数据仓库位置配置
    1) Default数据仓库的原始位置在hdfs上
    2) 在仓库目录下,没有对默认的数据库default创建文件夹。如果某张表属于default数据库,直接在数据仓库目录下创建一个文件夹。
    3) 修改default数据仓库原始位置(将hive-default.xml.template如下配置信息拷贝到hive-site.xml文件中)
        <property>
            <name>hive.metastore.warehouse.dir</name>
            <value>/user/hive/warehouse</value>
            <description>location of default database for the wirehouse</description>
        </property>
        配置同组用户有执行权限
        bin/hdfs dfs -chmod g+w /usr/hive/warehouse
查询后信息显示配置
    1) 在hive-site.xml文件中添加如下配置信息,就可以实现显示当前数据库,以及查询表的头信息配置
        <property>
            <name>hive.cli.print.header</name>
            <value>true</value>
        </property>
        <property>
            <name>hive.cli.print.current.db</name>
            <value>true</value>
        </property>
    2) 重启hive,对比配置前后差异
        再次查询,会输出对应的字段名称
hive运行日志信息配置(hive的启动和关闭日志,sql日志是mr程序的日志)
    1) hive的log默认存放在/tmp/当前用户名/hive.log目录下
    2) 修改hive的log存放日志到/opt/module/hive/logs
        a) 修改hive/conf/hive-log4j.properies.template文件名称为hive-log4j.properties
        b) 修改hive-log4j.properties文件中log存在的位置
            hive.log.dir=/opt/module/hive/logs
```

#### _hive 常用的交互命令_

```
1) -e 不进入hive交互窗口执行sql语句
bin/hive -e "select id from student;"
2) "-f" 执行脚本中的sql语句
    在/opt/module/datas目录下创建hivef.sql文件
    文件中写入正确sql语句
    select * from student;
    执行文件中的sql语句
    bin/hive -f /opt/module/datas/hivef.sql > ssgao.txt   (将查询结果可以放到ssgao.txt文件中)
交互命令的好处:
  可以设置一个任务在指定的时间进行执行

hive 其他的命令操作
  退出hive窗口
  hive > exit;  // 先隐形的提交数据,再退出
  hive > quit;  // 不提交数据,直接退出

hive cli 命令窗口中如何查看hdfs文件系统
hive > dfs -ls / ;
hive cli 命令窗口中如何查看本地文件系统
hive > ! ls /opt/module/datas ; ("!"不要省略 )
查看hive中输入的所有的历史命令
    1) 进入到当前用户的根目录/root或/home/ssgao
    2) 查看.hivehistory文件
    hive > cat .hivehistory
```

#### _问题集锦_

```
Metastore connection URL:    jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:   APP
Starting metastore schema initialization to 2.3.0
Initialization script hive-schema-2.3.0.mysql.sql
Error: Syntax error: Encountered "<EOF>" at line 1, column 64. (state=42X01,code=30000)
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
Underlying cause: java.io.IOException : Schema script failed, errorcode 2
Use --verbose for detailed stacktrace.
先确定hive-site.xml文件名是否正确，如果不对则必须改为hive-site.xml否则不生效。然后查看其中的mysql数据连接信息是否正确修改。我这里犯了个错误就是直接从网上拷贝后粘贴到文件的上方了，后来检查文件时发现文件中其实是有这四个标签的并且都有默认值，估计执行时后面标签的内容把我添加到前面的标签内容给覆盖掉了所以才没有生效。
```

```
[root@master bin]# hive
which: no hbase in (/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/software/jdk1.8.0_191/bin:/root/software/hadoop-2.7.4-cluster/bin:/root/software/redis-2.8.3:/root/bin:/root/software/jdk1.8.0_191/bin:/root/software/hadoop-2.7.4-cluster/bin:/root/software/redis-2.8.3:/usr/local/mysql/bin:/root/software/jdk1.8.0_191/bin:/root/software/hadoop-2.7.4-cluster/bin:/root/software/redis-2.8.3:/usr/local/mysql/bin:/root/software/apache-hive-2.3.4-bin/bin)
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/software/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/root/software/hadoop-2.7.4-cluster/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Logging initialized using configuration in file:/root/software/apache-hive-2.3.4-bin/conf/hive-log4j2.properties Async: true
Exception in thread "main" java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
    at org.apache.hadoop.fs.Path.initialize(Path.java:205)
    at org.apache.hadoop.fs.Path.<init>(Path.java:171)
    at org.apache.hadoop.hive.ql.session.SessionState.createSessionDirs(SessionState.java:659)
    at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:582)
    at org.apache.hadoop.hive.ql.session.SessionState.beginStart(SessionState.java:549)
    at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:750)
    at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:686)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.apache.hadoop.util.RunJar.run(RunJar.java:221)
    at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
Caused by: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
    at java.net.URI.checkPath(URI.java:1823)
    at java.net.URI.<init>(URI.java:745)
    at org.apache.hadoop.fs.Path.initialize(Path.java:202)
解决方法：建立相应目录并在hive-site.xml文件中修改
```

### _hive 的数据库和表_

> _Hive 的数据都是存储在 hdfs 上的,默认有一个根目录,在 hive-site.xml 中,由参数 hive.metastore.warehouse.dir 指定_

**_hive 的数据库(DataBase)_**

```
进入Hive 命令行,执行show databases; 命令,可以列出hive中的所有数据库,默认有一个default数据库,进入hive-cli后即到default数据库下
使用use databasename; 可以切换到某个数据库下,同mysql
hive>
    > show databases;
OK
default
Time taken: 4.728 seconds, Fetched: 1 row(s)
-- 创建database--------------------------------------------------------
创建数据库是用来创建数据库在Hive中语句.在Hive中数据库是一个命名空间或表的集合
create database|schema [if not exists] <database name>
这里,if not exists是一个可选子句,通知用户已经存在相同名称的数据库。
删除数据库 drop database|schema [if exists] database_name;
hive>
    > create database userdb;
OK
Time taken: 0.228 seconds
hive> create schema mydb;
OK
Time taken: 0.052 seconds
hive>
    > show databases;
OK
default
mydb
userdb
Time taken: 0.008 seconds, Fetched: 3 row(s)
hive>
    >
    > use mydb;
OK
Time taken: 0.018 seconds
hive>
    >  create table t1( sid int, name string, age int) location '/mytable/hive/t2';
OK
Time taken: 0.469 seconds
hive>
    > desc t1;
OK
sid                     int
name                    string
age                     int
Time taken: 0.157 seconds, Fetched: 3 row(s)
hive> show create table t1;
OK
CREATE TABLE `t1`(
  `sid` int,
  `name` string,
  `age` int)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://master.ssgao:9000/mytable/hive/t2'
TBLPROPERTIES (
  'transient_lastDdlTime'='1548341034')
Time taken: 0.139 seconds, Fetched: 14 row(s)
```

### _数据类型_

```
基本数据类型
    > tinyint/smallint/int/bigint :整数类型
    > float/double 浮点数类型
    > boolean 布尔类型
    > string 字符串类型
 复杂数据类型
    > Array 数组类型,由一系列相同数据类型的元素组成
      create table student (sid int, sname string, grade array<float>);
      数据:{1,Tom,[80,90,75]}
    > Map 集合类型,包含key->value键值对,可以通过key来访问元素
      create table student1(sid int,sname string,grade map<string,float>);
      数据:{1,Tom,<'语文成绩',132>}
    > Struct 结构类型, 可以包含不同数据类型的元素。这些元素可以通过"点语法"的方式来得到所需要的元素。
      create table student4( sid int,info struct<name:string,age:int,sex:string> );
      数据:{1, {'Tom',10,'男'}}

 时间类型
    > Date 从hive0.12.0 开始支持
    > Timestramp 从Hive0.8.0 开始支持
```

#### _map,array,struct_

```
map,string,struct类型数据解析
> map类型: 表中的数据类型为map<string,string>
  env
  {"uid":12345,"city":"unknow"}
 解析出city
  select env['city'] from table;
> string类型
 表中的数据
 exdata
 {"env_networkType":"4G","env_province":"Unknow","env_countryType":"0","childnum":"0","env_ouid":"",
  "filterlist":"[{\"filtertype\":\"8\",\"filtername\":\"小寨\\/省体育场\",\"filtersubtype\":
 解析出env_province,env_countryType
 select * from tablename a
 lateral view json_tuple(a.exdata,'env_networkType','env_province') as env_networkType,env_province
>struct类型
 struct<isp:string,longitude:double>
 geo
 ----------------------------------------
 {"isp":"电信","longitude":108.948024}
 要解析出isp
 select geo.isp from tablename;
```

#### _复杂数据类型查询_

```
数据类型
 `filter_ads` map<string,array<struct<dsp_id:string,deal_id:string,material_id:string,ad_filter_code:int>>>

 select filter_ads from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 => {"1231":[{"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}]}

 select filter_ads['1231'] from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 =>  [{"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}]

 select filter_ads['1231'][0] from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 => {"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}

select filter_ads['1231'][0],filter_ads['1231'][0].dsp_id, filter_ads['1231'][0].deal_id from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 => {"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231} 123 1231
```

#### _类型转换_

> _hive 类型转换支持向下转型,不支持向上转型_

```
Hive的原子数据类型是可以进行隐式转换的,类似于Java的类型转换,例如某表达式使用INT类型,TINYINT会自动转换为INT类型,但是Hive不会进行反向转化
例如某表达式使用TINYINT类型,INT不会自动转换为TINYINT类型,它会返回错误,除非使用CAST操作
隐式类型转换规则如下:
  1) 任何整数类型都可以隐式转换为一个范围更广的类型,例如TINYINT可以转换为INT类型,INT类型可以转换为BIGINT类型
  2) 所有整数类型,FLOAF,DUOBLE都可以转换为隐式的转换为STRING类型
  3) TINYINT,SMALLINT,INT都可以转换为FLOAT类型
  4) BOOLEAN类型不可以转换为任何其他类型

可以使用CAST操作显示进行数据类型转换
例如 CAST('1' as INT) 把字符串'1'转换为整数1,如果强制类型转换失败,如执行CAST('X' AS INT),表达式返回NULL
```

#### _数据类型修改_

> **_Alter table 表名 change column 原字段名称 现字段名称 数据类型_**

### _hive 的存储格式_

> _Hive 会为每个数据库在 HDFS 上创建一个目录,该数据库的表会以子目录形式存储,表中的数据会在表目录下以文件形式存储_

```
textFile
    textfile为默认格式，存储方式为行存储。数据不做压缩,磁盘开销大,数据解析开销大
SequeneceFile
    sequenceFile是Hadoop API提供的一种二进制文件支持,其具有使用方便,可分割,可压缩的特点
    sequenceFile支持三种压缩选择,NONE,RECORD,BLOCK。Record压缩率低,一般建议使用BLOCK压缩。

RCFile
    一种行列存储相结合的存储方式
ORCFile
    数据按照行分块,每个块按照列存储,其中每个块都存储有一个索引。
    Hive给出的新格式,属于RCFile的升级版,性能有大幅度的提升,而数据可以压缩存储,压缩块, 快速列存取
```

### _HIVE 相关 DDL 的操作_

```
删除空数据库
hive> drop database hive_db2;
如果删除的数据库不存在,最好采用if exists 判断数据库是否存在
hive> drop database if exits hive_db2;
如果数据库不为空,可以采用cascade命令,强制删除
hive> drop database db_hive cascade;
```

### _hive 的数据存储_

```
>>> hive的数据存储
     1) hive数据存储基于HDFS
     2) 没有专门的数据存储格式
     3) 存储结构主要包括:数据库,文件,表,视图
     4) 可以直接加载文本文件(.txt文件等)
     5) 创建表的时候,指定hive数据的列分隔符,与行分隔符
>>> hive的数据模型(表)
    —— Table 内部表
    —— Partition 分区表
    —— External Table 外部表
    —— Bucket Table 桶表
>>> hive中的视图
```

#### _hive 内部表_

```
>>> 内部表(Table)
    1) 内部表Load数据有两种方式:
        a） load data ***
         b）hdfs dfs -put *** 因为在Metastore文件,即mysql的hive数据库的"SDs"表中,保存着Hive表对应的hdfs路径信息
    2) 每一个Table在hive中都有一个相应的目录存储数据， 内部表在Load数据时,如果使用LOCAL关键字,Hive会把本地文件系统的数据文件复制到Hive的/wareHouse目录。反之,则是将HDFS上的数据文件剪切到/warehourse目录
    3) Hive在Load数据时,并不检查目录中的文件是否符合表所声明的模式。只有通过Select查询返回空值NULL,才能确定不匹配
    4) 内部表drop的时候,表的元数据(mysql中)和数据文件(hdfs中)都会被删除
  create table t1( sid int, name string, age int) location '/mytable/hive/t2'; //指定表存在的位置
  -- 指定分隔符
  create table t1( sid int, name string, age int) row format delimited fields terminated by ',';
  create table t4 as select * from sample_data;
  --------------------------------------------------
  create table t5
  --指定文件中列与列之间的分割符
  row format delimited fields terminated by ','
  as
  select * from sample_data;
  --------------------------------------------------
```

#### _hive 分区表_

```
>>> 分区表(Partition)
   1) Partition 对应于数据库中的Partition列的密集索引
   2) 在Hive中,表中的一个Partition对应于表下的一个目录,所有的Partition的数据都存储在对应的目录中.
分区实质,在数据表文件夹下再次创建分区文件夹,分区在创建表是用partitioned By定义,创建表后,可以使用Alter Table来增加和移除分区。
    create table logs (ts bigint,line string)
        partitioned by (dt string ,country string)
        row format delimited fields terminated by '\t';
> load数据时,显示指定分区值
    load data local inpath '/root/hive/file2' into table table_name partition (dt='xxx');
> 创建以性别为分区的表
    create table partition_table
    (sid int,sname string,partition by (gender string))
    row format delimited fields terminated by ',';
   -------------------------------------------------------
   insert into table partition_table partition(gender='M')
   select sid,sname,from sample_data where gender='M';
> 查看表的分区数
    show partitions table_name 命令,获取表中有哪些分区
```

##### _分区介绍_

```
分区建表分为2种，
    一种是单分区，也就是说在表文件夹目录下只有一级文件夹目录。
    一种是多分区，表文件夹下出现多文件夹嵌套模式。
a、单分区建表语句：
    create table day_table (id int, content string) partitioned by (dt string);
    单分区表，按天分区，在表结构中存在id，content，dt三列。
b、双分区建表语句：
    create table day_hour_table (id int, content string) partitioned by (dt string, hour string);
    双分区表，按天和小时分区，在表结构中新增加了dt和hour两列。
c、一次增加一个分区
    alter table testljb add partition (age=2);

d、一次增加多个分区,中间使用空格分开
    alter table testljb add partition(age=3) partition(age=4)
e、一次增加两个分区不可以写成 alter table testljb add partition (age=5,age=6)
   这种写法实际上是: 具有多个分区字段表的分区添加,我们两次写同一个字段,而系统中并没有两个age分区字段,就会随机添加其中一个分区
   有个表具有两个分区字段：age分区和sex分区。那么我们添加一个age分区为1，sex分区为male的数据，可以这样添加：
   alter table testljb add partition (age=1,sex='male')；

f、删除分区，使用逗号分隔VC
   alter table testljb drop partition(age=1),partition(age=2)；
   加入表testljb有两个分区字段（上文已经提到多个分区先后顺序类似于windows的文件夹的树状结构）
   partitioned by(age int ,sex string)，那么我们删除age分区（第一个分区）时，会把该分区及其下面包含的所有sex分区一起删掉。

分区中如何刷出新的字段
alter table table_name [partition] add|replace columns(col_name data_type comment xx) [cascade|restricts]
ddl语句最后添加cascade,否则新增的列在旧的分区中不可见,查询数据时为null,重新刷新数据仍为null
```

##### _分区表插入数据_

```
向分区表home插入新的数据,该数据的分区为partition(province='shandong')
> 使用load
   load data local inpath '/opt/module/datas/student.txt' into table stu_partition partition(month='2022623');
> 使用insert
hive> insert into home partition(province='shandong') select id,address,no,area from home_local;
Query ID = root_20190125232233_f78aa357-c5d1-440d-a246-bc44c1220f7d
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1548418373210_0002, Tracking URL = http://master.ssgao:8088/proxy/application_1548418373210_0002/
Kill Command = /root/software/hadoop-2.7.4-cluster/bin/hadoop job  -kill job_1548418373210_0002
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2019-01-25 23:22:39,985 Stage-1 map = 0%,  reduce = 0%
2019-01-25 23:22:44,262 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.07 sec
MapReduce Total cumulative CPU time: 1 seconds 70 msec
Ended Job = job_1548418373210_0002
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to: hdfs://master.ssgao:9000/mydb/home/province=shandong/.hive-staging_hive_2019-01-25_23-22-33_545_7568553728674045364-1/-ext-10000
Loading data to table mydb.home partition (province=shandong)
Partition mydb.home{province=shandong} stats: [numFiles=1, numRows=1, totalSize=23, rawDataSize=22]
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 1.07 sec   HDFS Read: 3664 HDFS Write: 106 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 70 msec
OK
Time taken: 13.211 seconds
hive> select * from home ;
OK
1   杭州  3   江南国际城   hangzhou
1   shandong    4   gaozhuang   shandong
Time taken: 0.077 seconds, Fetched: 2 row(s)
查看该分区表的数据路径会发现多了province=shandong的文件夹
[root@slavea ~]# hadoop fs -ls /mydb/home/
Found 2 items
drwxr-xr-x   - root supergroup          0 2019-01-25 23:02 /mydb/home/province=hangzhou
drwxr-xr-x   - root supergroup          0 2019-01-25 23:22 /mydb/home/province=shandong
```

#### 动态分区\_

```
https://blog.csdn.net/yeweiouyang/article/details/52560078
动态分区
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.max.dynamic.partitions=20480;
set hive.exec.max.dynamic.partitions.pernode=20480;
当insert往分区插入数据时,我们可以指定分区字段名和值,分区字段的值是可选的,如果指定分区字段值,我们称为静态分区,否则称为动态分区。动态分区创建根据插入数据的列决定。 关于动态分区插入,有几个相关的配置:
hive.exec.dynamic.partition=true; 设置为true表示启用动态分区插入
hive.exec.dynamic.partition.mode=nonstrict;
            strict表示用户必须指定一个静态分区,nonstrict所有分区都可能是动态的。
hive.exec.max.dynamic.partitions.pernode=100; 每个mapper/reducer节点允许创建的最大动态分区数
hive.exec.max.dynamic.partitions=1000; 总共允许创建的最大动态分区数
hive.exec.max.created.files=100000;  一个MR job中允许创建多少个HDFS文件
hive.error.on.empty.partition=false; 当动态分区插入产生空值时是否抛出异常
```

```sql
create table t_partitioned(url string )
	partitioned by (month string,day string)
  stored as textfile;

将数据按照时间插入到表中,需要使用动态分区来实现,动态分区需要注意设定如下参数:
 'hive.exec.dynamic.partition 默认false'
 		是否开启动态分区功能,默认false关闭
    使用动态分区的时候需要设置为true
 'hive.exec.dynamic.partition.mode 默认值strict'
		动态分区的模式,默认值为strict,表示必须指定至少一个分区为静态分区
    nonstrict模式表示运行所有的分区字段都可以使用动态分区,一般需要设置为nonstrict
 'hive.exec.max.dynamic.paritions.pernode 默认值100'
 		在每个执行MR的节点上,最大可以创建多少个动态分区
    该参数需要根据实际的数据来设定
    比如: 源数据中包含了一年的数据,即day字段有365个值,那么该参数就需要设置成大于365,如果用默认值100,则会报错。
'hive.exec.max.dynamic.partitions 默认值 1000'
		在所有执行MR的节点上,最大一共可以创建多少动态分区
'hive.exec.max.created.files 默认值100000'
 	  整个MR job中,最大可以创建多少个hdfs文件
    一般默认值就足够了,除非数据量特别的大,需要创建的文件数大于100000
'hive.error.on.empty.partition 默认值 false'
	 当有空分区生成时,是否抛出异常
   一般不需要设置


SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.max.dynamic.partitions.pernode = 1000;
SET hive.exec.max.dynamic.partitions=1000;
insert overwrite table t_partitioned partition(month,day)
select url,substr(url,1,7) as month,day from t_partition
-- 查询子句中需要包含 month,day 字段信息
```

\_

#### _hive 外部表_

```
>>> 外部表(External table)
    1) 外部表只需在创建表时显示指定表中数据存储位置即可。Hive不会把数据剪切到自己的目录
    2) 实际上,在创建表时,Hive甚至不会检查这一外部位置是否存在。这是一个非常重要的特性,意味着可以先创建表,然后再上传数据文件
    3) 外部表只有一个过程,加载数据和创建表同时完成,并不会移动到数据仓库目录中,只是与外部数据建立一个链接.当删除一个外部表的时候,仅仅删除该链接.
    4) 外部表在drop时,不会碰数据文件,而只会删除元数据信息,即HDFS上的'/book'目录依旧存在。
> 通过指定外部文件创建外部表
    create external table external_student
    (sid int, sname string,age int)
    row format delimited fields terminated by ','
    location '/input';
```

- 内部表和外部表的使用场景

```
每天将收集的网站日志定期流入HDFS文件。
在外部表(原始日志表)的基础上做大量的分析,用到的中间表,临时表,结果表使用内部表存储,数据通过select+insert进入内部表
```

- 内部表和外部表的互相转换

```
查询表的类型
desc formatted student2;
修改内部表student2为外部表
alter table student2 set tblproperties('EXTERNAL'='TRUE');
修改外部表student2为内部表
alter table student2 set tblproperties('EXTERNAL'='FALSE');
注意:
('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法,区分大小写
```

- 外部表修改 location

```
alter table table_name set location 'hdfs://xxx/xxx/xx'
```

#### _hive 桶表_

```
>>> 桶表(Bucket table)
    -桶表是对数据进行哈希取值,然后放到不同文件中存储.
> 创建桶表
  create table bucket_table
  (sid int, sname string, age int)
  clustered by (sname) into 5 buckets;  //对sname进行哈希运算, 创建5个桶表
```

#### _hive 视图_

```
>>> 视图(view)
    1) 视图是一种虚表,是一个逻辑概念,可以跨越多张表
    2) 视图建立在已有表的基础上,视图赖以建立的这些表称为基表
    3) 视图是不存储数据的
    4) 视图可以简化复杂查询
> 创建视图信息
   -- 视图员工信息: 员工号,姓名,月薪,年薪,部门名称
      create view empinfo
      as
      select e.empno,e.ename,e.sal,e.sal*12 allSary, d.dname
      from emp e, dept d
      where e.deptno=d.no;
```

### _hive 建表详述_

#### _详细建表语句_

```
标准hql建表语法
create [external] table [if not exists] table_name
[(col_name data_type [comment col_comment],...)]
[comment table_comment]
[partitioned by (col_name data_type [comment col_comment]),...]
[clustered by (col_name,col_name,...)]
[sorted by (col_name [asc|desc],...)] into num_buckets BUCKETS]
[row format delimited fields terminated by ',']
[collection items terminated by '-']
[map keys terminated by ':']
[lines terminated by '\n']
[stored as file_format]
[location hdfs_path]
1> create table 创建一个指定名字的表,如果相同名字表已经存在,则抛出异常。
    我们可以用if not exists 选项来忽略这个异常,一般也可以不加这个if not exits语句,最多抛出错误

2> external 关键字让用户创建一个外部表,默认是内部表,创建外部表必须同时指定一个指定实际数据的路径(location)
    hive创建内部表时,会将数据移动到数据仓库指向的路径,若创建外部表仅记录数据所在路径,不对数据的位置进行任何该表。

3> comment 表示为表字段或表内容添加注释说明
4> partitioned by 为表做分区,hive中所谓的分区表就是对表新增一个字段,就是分区的名字,这样我们在操作表中的数据时可以按分区字段进行过滤。
5> [row format ] 指定表存储各列数据的划分格式,这里指定的是逗号分隔符,可以是其他分隔符
6> stored as sequencefile|textfile|rcfile 如果文件数据存文本,可以使用stored as textfile.如果数据要压缩使用stored as sequencefile
7> clustered by 对于每个表(table)或者分区,hive可以进一步组织成桶,也就是说桶是更为细粒度的数据范围划分。
    hive也针对某一列进行桶的组织
    hive采用对列值哈希,然后除以桶的个数求余的方式决定该条记录存放在哪个桶中
8> location 其实是定义hive表的数据在hdfs上的存储路径,一般内部表不需要自定义，但是如果是外部表最好直接指定一个路径,否则会使用默认路径
```

##### _inputformt/outputformat/serde_

```
https://blog.csdn.net/tianyeshiye/article/details/79822986
```

```
hive中,默认使用的是TextInputFormat,一行表示一条记录。在每条记录(一行中),默认使用^A分割各个字段
有些时候,我们往往面对多行,结构化的文档,并需要将其导入Hive处理,这时我们就需要自定义InputFormat,OutputFormat以及SerDe了
三者之间的关系,我们直接使用Hive的官网的说法
SerDe is short name for “Serializer and Deserializer”
Hive uses SerDe(and !FileFormat) to read and write table rows
HDFS files -> InputFileFormat -> <key,value> ->Deserializer -> Row object
Row object ->Serializer -> <Key,Value> -> OutputFileFormat -> HDFS files
即: 当面临一个HDFS上的文件时,Hive将如下处理(以读为例)
        a> 调用InputFormat,将文件切成不同的文档。即每篇文档即一行(Row)
        b> 调用SerDe的Derserializer,将一行(Row),切分为各个字段

    当Hive指定insert操作,将Row写入文件时,主要调用OutputFormat,SerDe的Seriliazer,顺序与读取相反
```

#### _建表语句详细_

```
CREATE TABLE `t1`(
  `sid` int,
  `name` string,
  `age` int)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://master.ssgao:9000/mytable/hive/t2'
TBLPROPERTIES (
  'COLUMN_STATS_ACCURATE'='true',
  'numFiles'='0',
  'numRows'='3',
  'rawDataSize'='35',
  'totalSize'='0',
  'transient_lastDdlTime'='1548346852')


创建外部表
CREATE EXTERNAL TABLE `home`(
  `id` int COMMENT 'id',
  `address` string COMMENT '地址',
  `no` int COMMENT '人数',
  `area` string COMMENT '小区')
PARTITIONED BY ( `province` string COMMENT '省份')
ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://master.ssgao:9000/mydb/home'
TBLPROPERTIES (
  'transient_lastDdlTime'='1548428115')
hive>
    > alter table home add partition (province='hangzhou');
OK
Time taken: 0.275 seconds
hive> select * from home;
OK
1   杭州  3   江南国际城   hangzhou
Time taken: 0.372 seconds, Fetched: 1 row(s)
hive>
```

```
create table student1 as select * from student;
这种方式创建的表,除了表结构同时也把数据复制过来
create table student2 like student;
这种方式只有表结构,没有表数据
desc extended student;
desc formatted student;
```

#### _可能出现的问题_

```
hive>
    > alter table home add partition (province='hangzhou');
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:hdfs://master.ssgao:9000/mydb/home/province=hangzhou is not a directory or unable to create one)
说明没有分区被添加
```

### _hive 数据导入_

```
> 使用load执行数据导入
  load data [local] inpath 'filepath' [overwrite] into table tableName
  [patition (partcol1=val1,partcol2=val2..)]
  -----------------------------------------------------------------------------
     [local]: 表示从操作系统的目录进行导入，如果不写local表示从HDFS数据目录进行数据导入
      inpath: 表示导入的路径'filepath'文件存在的目录
hive> load data inpath '/root/inner_table.dat' into table t1;
      移动hdfs中数据到t1表中
hive> load data local inpath '/root/inner_table.dat' into table t1;
      上传本地数据到hdfs中
hive> !ls 查询当前linux文件夹下的文件
hive> dfs -ls / 查询当前hdfs文件系统下'/'目录下的文件
```

### _hive 表相关操作_

```
Hive 创建表加上对字段的描述信息
    create table table_name (column1 int comment '第一列',...); //comment表示字段描述
    create table test_table(id bigint comment '序号',name string comment '姓名');
> 查看表信息
    desc person;
> 查看表分区
    show partitions t1; (查看表有哪些分区)
> 删除表信息
    drop table t1;
    drop table t1 if exits t1;
> 新增字段(新增列信息)
    添加列 alter table t1 add columns(english,int);
    alter table tablename add columns (列名 类型 [comment '注释']); --comment 部分可选
    alter table table_test add columns (data string);
    alter table table_test add columns (
        order_source string comment '订单来源',
        bank_name string comment '银行行名'
    );

  alter table `iad_dwd_prod.dwd_adx_dsp_bid_range_ads_win` add columns (log_day string comment '日志记录日期')

> alter table table_name [partition] add|replace columns(col_name data_type comment xx) [cascade|restricts]
    ddl语句最后添加cascade,否则新增的列在旧的分区中不可见,查询数据时为null,重新刷新数据仍为null
> 修改hive表字段注释
    alter hive 表名 change 列名 新的列名 新的列名类型 comment '列注释'
> 修改hive表名
    alter table table_name rename to new_table_name;
> 修改hive表字段名
    alter table 表名 change 旧字段 新字段 类型;
```

#### _hive insert_

```
insert into overwrite local directory '/home/hadoop/ssgao/test_demo.txt'
 row format delimited
 fields terminated by '\t'
 select * from test;
insert into 和 insert overwrite的区别
> insert into 语句
    insert into table account select id,age,name form account_tmp;
> insert overwrite 语句
    insert overwrite table account2 select id,age,name from account_tmp;

> 两者的区别:
    insert overwrite 会覆盖已经存在的数据,先将原始表的数据remove,如果存在分区的情况下,insert overwrite 会只重写当前分区数据
    insert into 只是简单的插入,不考虑原始表数据,直接追加到表中,最后表中的数据是原始数据和新插入的数据。
使用insert overwrite 字段数量要一致
insert into ... select
> insert into table2(字段1,字段2) select 字段1,字段2 from table1;
> insert into table2(字段1,字段2) partition(field=xx,fieldb=xx) select 字段1,字段2 from table1 where partition=xx (插入指定的分区)
```

```
hive>
    > insert into t1 (sid,name,age) values (1,'ssgao',30);
Query ID = root_20190125001415_864c1887-6aa9-4942-b638-5f747c11ca84
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1548345485107_0001, Tracking URL = http://master.ssgao:8088/proxy/application_1548345485107_0001/
Kill Command = /root/software/hadoop-2.7.4-cluster/bin/hadoop job  -kill job_1548345485107_0001
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2019-01-25 00:14:27,294 Stage-1 map = 0%,  reduce = 0%
2019-01-25 00:14:34,692 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 2.38 sec
MapReduce Total cumulative CPU time: 2 seconds 380 msec
Ended Job = job_1548345485107_0001
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to: hdfs://master.ssgao:9000/mytable/hive/t2/.hive-staging_hive_2019-01-25_00-14-15_359_8894983812698269023-1/-ext-10000
Loading data to table mydb.t1
Table mydb.t1 stats: [numFiles=0, numRows=1, totalSize=0, rawDataSize=10]
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 2.38 sec   HDFS Read: 3811 HDFS Write: 74 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 380 msec
OK
Time taken: 20.807 seconds
hive>
    >
    > select * from t1;
OK
1   ssgao   30
Time taken: 0.121 seconds, Fetched: 1 row(s)
hive>
hive>
    >
    >
    > insert into t1 values (2,'chenclin','30');
Query ID = root_20190125001556_e1d4d2d3-d62f-49b6-9e3a-3b74e7c6935a
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1548345485107_0002, Tracking URL = http://master.ssgao:8088/proxy/application_1548345485107_0002/
Kill Command = /root/software/hadoop-2.7.4-cluster/bin/hadoop job  -kill job_1548345485107_0002
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2019-01-25 00:16:02,912 Stage-1 map = 0%,  reduce = 0%
2019-01-25 00:16:08,261 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.73 sec
MapReduce Total cumulative CPU time: 1 seconds 730 msec
Ended Job = job_1548345485107_0002
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to: hdfs://master.ssgao:9000/mytable/hive/t2/.hive-staging_hive_2019-01-25_00-15-56_504_6233551623849223257-1/-ext-10000
Loading data to table mydb.t1
Table mydb.t1 stats: [numFiles=0, numRows=2, totalSize=0, rawDataSize=23]
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1   Cumulative CPU: 1.73 sec   HDFS Read: 3813 HDFS Write: 77 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 730 msec
OK
Time taken: 13.118 seconds
hive>
    > select * from t1;
OK
1   ssgao   30
2   chenclin    30
Time taken: 0.08 seconds, Fetched: 2 row(s)
hive>
hive> desc home_local ;
OK
id                      int
address                 string
no                      int
area                    string
Time taken: 0.045 seconds, Fetched: 4 row(s)
hive> insert into home_local (id,address,no) values (3,'jining',3);
hive> select * from home_local;
OK
1   shandong    4   gaozhuang
3   jining  3   NULL
Time taken: 0.058 seconds, Fetched: 2 row(s)
```

### _hive 函数_

[https://www.cnblogs.com/jifengblog/p/9286800.html](https://www.cnblogs.com/jifengblog/p/9286800.html)

#### _get_json_object_

#### _json_tuple_

| data                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| _{"store":{"fruit":[{"weight":8,"type":"apple"},{"weight":9,"type":"pear"}]，“bicycle”:{"price":19.95,"color":"red"}},"email":"amy@only_for_json_udf_test.net","owner":"amy"}_ |

```
get_json_object(string json_string,string path)
返回值 string
说明 解析json的字符串json_string,返回path指定的内容,如果输入的json字符串无效,那么返回null
注意 输入的字段需要必须为string 类型,如果是array,struct 类型则无效
select  get_json_object(data,'$.owner') from dual;  --输出结果为amy
json_tuple(jsonstr,k1,k2,...)
参数为一组键 k1,k2...和json字符串,返回值为元组。该方法比get_json_object高效,因为可以在一次调用中输入多个键
select a.timestamp,b.* from log a lateral view json_tuple(a.appevent,'eventid','eventname') b as f1,f2;
ps: get_jsob_object和jsob_tuple 解析的对象为string类型,而非结构类型(如：map,array,struct)等
```

```
select body from table_test where d='2018-10-23'
{"1231":[{"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}]}
-- get_json_object的嵌套使用, 解析数组中的元数据
select get_json_object(body,'$.1231'),get_json_object(get_json_object(body,'$.1231[0]'),'$.dsp_id') ,body from table_test
-- json_tuple和get_json_object 联合使用
select json_tuple(get_json_object(body,'$.adContentList[0]'),'ad_type','location','app') from iad_ods_test.tmp_body_sloth_dms_biddings_di_20181106
```

#### _laterval view_

```
https://blog.csdn.net/youziguo/article/details/6837368
行转列
laterval view 用于和split,explode 等UDTF一起使用,可以将一行数据拆分成多行数据,并在此基础上对拆分数据进行聚合。
laterval view 首先为原始表的每行调用UDTF,UDTF会把一行拆分成一行或者多行,laterval view 把结果组合,产生一个支持别名表的虚拟表
1) 单个laterval view
原表 table_1
A      B
190     [100,101]
191     [200,201]
select A, B from table_1 laterval view expolde(B) mytable as B
A       B
190 100
190 101
191 200
191 201
2) 多个laterval view
from 语句后面可以带有多个laterval view 语句
Array<int>col1 Array<string>col2
  [1,2]                 ['a','b','c']
  [3,4]                 ['d','e','f']
select mycol1,mycol2 from basetable
laterval view explode(col1) mytable1 as myclo1
laterval view explode(col2) mytable2 as myclo2

3) 复杂方式
20141018  aa|bb 7|9|0|3
20141019    cc|dd 6|1|8|5
select datenu,des,type from tb_split
    laterval view explode(split(des,"\\|")) tb1 as des
  laterval view explode(split(type,"\\|")) tb2 as type
```

#### _explode 行转列_

```
explode(array(or map))
explode(array)列表中的每个元素生成一行
explode(map) map中的每个key-value对,生成一行,key为一列,value为一列
将输入的一行数组或者map转换为列输出
```

#### _coalesce 函数_

```
coalesce函数: 返回列表中第一个非空表达式
coalesce(expression,expression[,...])
下面的语句返回值 34; SELECT COALESCE( NULL, 34, 13, 0 )
备注: 如果所有参数均为 NULL，则 COALESCE 返回 NULL。
COALESCE(expression1,...n) 与此 CASE 函数等效：
CASE   WHEN (expression1 IS NOT NULL) THEN expression1
...
WHEN (expressionN IS NOT NULL) THEN expressionN   ELSE NULLEND
尽管 ISNULL 等效于 COALESCE，但它们的行为是不同的。
包含具有非空参数的 ISNULL 的表达式将视为 NOT NULL，而包含具有非空参数的 COALESCE 的表达式将视为 NULL。
使用COALESCE的秘密在于大部分包含空值的表达式最终将返回空值（连接操作符“||” 是一个值得注意的例外）。
例如，空值加任何值都是空值，空值乘任何值也都是空值， 依此类推。
```

#### _array_contains 函数_

```
select * from login where dt=20130101 and !array_contains(split('3.1,3.2,4.0,5.2',','),ver);
```

### _hive 的数值操作函数_

- _指定精度取整函数 round_

```
round(double a, int d) 返回值类型为double
select round(3.1415926,4) from dual;
3.1415
```

- _向下取整函数 floor_

```
floor(double a)
返回等于或者小于该double变量的最大整数
select floor(3.1415) from dual;
3
select floor(25) from dual;
25
```

- _向上取整函数 ceil_

```
ceil(double a)
返回等于或者大于该double变量的最大整数
slect ceil(3.14) from dual;
4
select ceil(46) from dual;
46
可以使用ceiling命令来代替ceil
```

- _取随机函数 rand_

```
rand() rand(int seed)
返回一个0~1范围内的随机数。如果指定种子,则会等到一个稳定的随机数序列
select rand() from dual;
0.557
```

- _绝对值函数 abs_

```
abs(double a) abs(int a)
返回double int (返回数值的绝对值)
select abs(-3.9) from dual;
3.9
select abs(-4) from dual;
4
```

### _hive 的字符串操作函数_

- _substr_

```
截取字符串常用的就是substr函数或者substring函数
substr(String a, int start), substring(String a,int start)  --两者用法一样
返回值:string
说明: 返回字符串A,从start位置到结尾的字符串
```

- _concat_ws_

```
concat_ws(string SEP,string A,string B,...)
SEP: 返回输入字符串连接后的结果,SEP表示各个字符串之间的分隔符
返回值: string
```

- _split 字符串分割函数_

```
split(str,regex)  --split str arround occurances that match regex;
返回一个数组
split('a,b,c',',')
结果: ["a","b","c"]
截取字符串中的某个值,我们可以指定获取结果数组中的某一项
split['a,b,c',','][0] = a
特殊字符的处理
split(str,regex) --当regex为字符串匹配的参数,所以遇到特殊字符的时候需要进行特殊处理
split('192.168.10.1','\\.') = ["192","168","10","1"]
split('name|age|address','\\|') = ["name","age","address"]
注意,当split包含在""之中时,需要加4个 \\
hive -e "...split('192.168.0.1','\\\\.')..."
```

-

### _hive 的语句_

> _select count(\*) from user ”去查询 user 表的大小，因为 HIVE 会将这个语句翻译为 MR 作业在 HADOOP 上运行，效率非常低_。

#### _show tables_

```
hive>
    > show tables;
OK
t1
Time taken: 0.023 seconds, Fetched: 1 row(s)
```

#### _show create table tbname_

#### _show partitions_

### _hive 自定义函数_

#### UDF(用户定义函数)

> _当 hive 提供的内置函数无法满足我们业务处理需要时,就需要考虑使用用户自定义函数(UDF: user-defined function)_

```
新建Java maven项目
 添加依赖
 <dependencies>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.4</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.2</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

```
/** hive自定义函数 */
package hive.udf;
public class ItcastFunc extends UDF{
    // 重载
    public String evaluate(String input){
        return input.toLowerCase(); //将大写字母转换为小写
    }
    public int evaluate(int a,int b){
        return a+b; //计算两个数之和
    }
}
*> 打成jar包上传到服务器
*> 将jar包添加到hive的classpath
  hive> add jar hdfs://master.ssgao:9000/root/hivedata/udf.jar;
*> 创建临时函数与开发好的java class关联
  create temporary function udffunc as 'hive.udf.ItcastFunc'
    temporary 表示临时方法
    udffunc 表示hive中定义的函数名
    hive.udf.ItcastFunc 为自定义方法的全类路径

 *> 在hive中使用自定义的函数
    select udffunc("ABC") from dual;  //输出abc
    select udffunc(2,3) from dual; //输出 5
```

\_

#### UDAF(用户定义聚集函数)

```java
输入多行数据,输出一行数据, 一般在group by中使用。
 一般情况使用不到
```

#### UDTF(用户定义表生成函数)

```java
UDTF 操作作用于单个数据行, 产生多个数据行。 1对多 如 lateral view explore()
/**
 * UDTF 实现一行输入, 多行输出
 */
public class UDTFDemo extends GenericUDTF {

    /**
     * 初始化校验参数是否正确
     * @return
     * @throws UDFArgumentException
     */
    @Override
    public StructObjectInspector initialize(ObjectInspector[] args) throws UDFArgumentException {
        if (args.length != 1) {
            throw new UDFArgumentLengthException("ExplodeMap takes only one argument");
        }
        if (args[0].getCategory() != ObjectInspector.Category.PRIMITIVE) {
            throw new UDFArgumentException("ExplodeMap takes string as a parameter");
        }
        /**添加字段名col1,col2 和其对应的类型 */
        ArrayList<String> fieldNames = new ArrayList<String>();
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<ObjectInspector>();
        fieldNames.add("col1");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        fieldNames.add("col2");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        /**添加字段名col1,col2 和其对应的类型 */
        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,fieldOIs);
    }

    /**
     * process 处理返回结果
     * @param args
     * @throws HiveException
     */
    @Override
    public void process(Object[] args) throws HiveException {
        String input = args[0].toString();
        String[] test = input.split(";");
        for (int i = 0; i < test.length; i++) {
            try {
                String[] result = test[i].split(":");
                /**  将结果返回 */
                forward(result);
            } catch (Exception e) {
                continue;
            }
        }

    }

    @Override
    public void close() throws HiveException {
    }
}
```

\_

```java
UDTF的使用
  add jar /user/ad/test_udtf.jar
  create temporary function explode_map as 'com.aouo.test.TestUDTF'
  select explode_map(mapstrs) as (col1,col2) from app.t_maps;

UDTF两种使用方式:
	1) 直接放在select 后面
    	select explode_map(properties) as (col1,col2) from src;
        不可以添加其他字段使用
        	select a, explode_map(properties) as (col1,col2) from src;
        不可以和groupby/cluster by/distribute by/sort by 一起使用
        select explode_map(properties) as (col1,col2) from src group by col1,col2;
		不可以嵌套调用
        select a,explode_map(explode_map(properties)) from src;
    2) 和lateral view一起使用
       select src.id, mytable.col1, mytable.col2 from src
       			lateral view explode_map(properties) mytable as col1,col2;
```

\_
\_
\_
\_
\_

### 开窗函数

_普通的聚合函数聚合的行集是组,开窗函数聚合的行集是窗口。因此,普通的聚合函数每组(group by)只返回一个值,而开窗函数则可为窗口中每行都返回一个值。_
_简单的理解，就是对查询的结果多出一列，这一列可以是聚合值，也可以是排序值。_
_开窗函数一般分为两类,聚合开窗函数和排序开窗函数。_

#### sum 开窗函数

```sql
-- sum开窗函数

select studentId,math,departmentId,classId,
-- 以符合条件的所有行作为窗口
sum(math) over() as sum1,
-- 以按classId分组的所有行作为窗口
sum(math) over(partition by classId) as sum2,
 -- 以按classId分组、按math排序后、按到当前行(含当前行)的所有行作为窗口
sum(math) over(partition by classId order by math) as sum3,
 -- 以按classId分组、按math排序后、按当前行+往前1行+往后2行的行作为窗口
sum(math) over(partition by classId order by math rows between 1 preceding and 2 following) as sum4
from student_scores where departmentId='department1';


结果
studentid   math    departmentid    classid sum1    sum2    sum3    sum4
111         69      department1     class1  718     410     69      223
113         74      department1     class1  718     410     143     316
112         80      department1     class1  718     410     223     341
115         93      department1     class1  718     410     316     267
114         94      department1     class1  718     410     410     187
124         70      department1     class2  718     308     70      222
121         74      department1     class2  718     308     144     308
123         78      department1     class2  718     308     222     238
122         86      department1     class2  718     308     308     164

结果解释:
    同count开窗函数
```

#### lag 开窗函数

```sql
lag(col,n,default) 用于统计窗口内往上第n个值。
    col:列名
    n:往上第n行
    default:往上第n行为NULL时候，取默认值,不指定则取NULL

select studentId,math,departmentId,classId,
 --窗口内 往上取第二个 取不到时赋默认值60
lag(math,2,60) over(partition by classId order by math) as lag1,
 --窗口内 往上取第二个 取不到时赋默认值NULL
lag(math,2) over(partition by classId order by math) as lag2
from student_scores where departmentId='department1';
```

#### 排序函数: rank 开窗函数

```sql
-- rank 开窗函数

select *,
-- 对全部学生按数学分数排序
rank() over(order by math) as rank1,
-- 对院系 按数学分数排序
rank() over(partition by departmentId order by math) as rank2,
-- 对每个院系每个班级 按数学分数排序
rank() over(partition by departmentId,classId order by math) as rank3
from student_scores;

结果
id  studentid   language    math    english     classid departmentid    rank1   rank2   rank3
1   111         68          69      90          class1  department1     1       1       1
3   113         90          74      75          class1  department1     3       3       2
2   112         73          80      96          class1  department1     9       6       3
5   115         99          93      89          class1  department1     15      8       4
4   114         89          94      93          class1  department1     17      9       5
9   124         76          70      76          class2  department1     2       2       1
6   121         96          74      79          class2  department1     3       3       2
8   123         70          78      61          class2  department1     7       5       3
7   122         89          86      85          class2  department1     14      7       4

```

#### 参考连接

[_https://blog.csdn.net/wangpei1949/article/details/81437574_](https://blog.csdn.net/wangpei1949/article/details/81437574)

### \_

### _hive 表索引_

```java
https://blog.csdn.net/zzq900503/article/details/79391071
```

### \_

### _hive 谓词下推_

```java
https://blog.csdn.net/weixin_33894640/article/details/93843917?utm_medium=distribute.pc_relevant.none-task-blog-OPENSEARCH-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-1.control
```

### _总结_
