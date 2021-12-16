_**MPP 大规模并行计算**_
![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610690997808-11535ae9-9045-4da5-b26a-448a6e679328.png#align=left&display=inline&height=495&margin=%5Bobject%20Object%5D&name=image.png&originHeight=601&originWidth=839&size=82989&status=done&style=none&width=691)

### impala 特点

```java
优点:
  1) 基于内存运算, 不需要把中间结果写入磁盘, 省掉大量的I/O开销
  2) 无需转换为MapReduce, 直接访问存储在HDFS, Hbase中的数据进行作业调度, 速度快。
  3) 使用了支持Data Locality 的I/O调度机制, 尽可能的将数据和计算分配在同一台机器上进行,减少网络开销。
  4) 支持各种文件格式, 如TEXTFILE, SEQUENCEFILE, RCFILE, Parquet
  5) 可以访问hive的metastore, 对hive数据直接做数据分析。

缺点:
  1) 对内存的依赖大,且完全依赖于Hive
  2) 实践中,分区超过一万,性能严重下降
  3) 只能读取文本文件,而不能直接读取自定义二进制文件
  4) 每当新的记录/文件被添加到HDFS中的数据目录时, 该表需要被刷新。
      比如同一张student表,现在有一条数据, 通过hive又插入一条数据, 通过hive查询是2条数据, 但是impala查询还是显示一条,这时候需要多元数据进行刷新,必须手动刷新,不会自动刷新。


 impala无法查询含有hive表中的map<String,String>类型的字段
 		Impala 只用parquet格式存储时，才能使用复杂数据类型
```

### **impala 的组件**

**![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610683119971-f9b55842-062f-4904-abe3-e42eeda99565.png#align=left&display=inline&height=333&margin=%5Bobject%20Object%5D&name=image.png&originHeight=367&originWidth=741&size=66196&status=done&style=none&width=672)**

#### impala Daemon

```java
Impala的核心组件是运行在各个节点上面的impalad这个守护进程(Impala daemon) 它负责读写数据文件, 接收从impala-shell hue jdbc, odbc 等接口发送的查询语句, 并行化查询语句和分发工作任务到impala集群的各个节点上,同时负责将本地计算好的查询结果,发送给协调节点(coordinator node). coordinator node负责构建最终的结果数据返回给用户。

impala在提交任务的时候,采用round-robin 算法来实现负载均衡, 将任务提交到不同的节点上。
impalad进程通过持续和statestore通信来确认自己所在的节点是否健康 和是否可以接受新的任务请求。

```

#### impala statestore

```java
状态管理进程, 定时检查 the impala Daemon的健康状况, 协调各个运行impalad的实例之间的信息关系, impala正是通过这些信息去定位查询请求所要的数据, 进程名叫做statestored, 在集群中只需要启动一个这样的进程, 如果impala节点由于物理原因,网络原因,软件原因或者其他原因而下线, statestore会通知其他节点, 避免查询任务分发到不可用的节点上。
```

#### impala catalog service

```java
元数据管理服务, 进程名叫做 catalogd, 将数据表变化的信息分发给各个进程。
接收来自statestore的所有请求, 每个impala节点在本地缓存所有元数据。当处理极大量的数据和/或许多分区时,获得表特定的元数据可能需要大量的时间。因此,本地存储的元数据缓存有助于提供这样的信息。
当表定义或表数据更新时, 其他impala后台进程必须通过检索最新元数据来更新其元数据缓存,然后对相关表发出新查询。
```

### impala 的查询流程

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610688976895-4339577e-c174-4573-8017-e7f931e7424a.png#align=left&display=inline&height=302&margin=%5Bobject%20Object%5D&name=image.png&originHeight=382&originWidth=781&size=57279&status=done&style=none&width=618)

```java
select
t1.n1,
t2.n2,
count(1) as c
from t1 join t2 on t1.id = t2.id
join t3 on t1.id = t3.id
where t3.n3 between ‘a’ and ‘f’
group by t1.n1, t2.n2
order by c desc
limit 100;
```

```java
1)T1和T2使⽤Hash join，此时需要按照id的值分别将T1和T2分散到不同的Impalad进程，但是相同的id会散列到相同的Impalad进程，这样每⼀个Join之后是全部数据的⼀部分。
2)T1与T2Join之后的结果数据再与T3表进⾏Join,此时T3表采⽤Broadcast⽅式把⾃⼰全部数据(id列)⼴播到需要的Impala节点上。
3) T1,T2,T3Join之后再根据Group by执⾏本地的预聚合，每⼀个节点的预聚合结果只是最终结果的⼀部分（不同的节点可能存在相同的group by的值），需要再进⾏⼀次全局的聚合。
4） 全局的聚合同样需要并⾏，则根据聚合列进⾏Hash分散到不同的节点执⾏Merge运算（其实仍然是⼀次聚合运算），⼀般情况下为了较少数据的⽹络传输， Impala会选择之前本地聚合节点做全局聚合⼯作。
5) 通过全局聚合之后，相同的key只存在于⼀个节点，然后对于每⼀个节点进⾏排序和TopN计算，最终将每⼀个全局聚合节点的结果返回给Coordinator进⾏合并、排序、limit计算，返回结果给⽤户。

```

#### impala 元数据的刷新

```java
mpala采用了比较奇葩的多个impalad同时提供服务的方式，并且它会由catalogd缓存全部元数据，再通过statestored完成每一次的元数据的更新到impalad节点上，Impala集群会缓存全部的元数据，这种缓存机制就导致通过其他手段更新元数据或者数据对于Impala是无感知的，例如通过hive建表，直接拷贝新的数据到HDFS上等,Impala提供了两种机制来实现元数据的更新，分别是INVALIDATE METADATA和REFRESH操作
```

##### invalidate metadata

```java
INVALIDATE METADATA是用于刷新全库或者某个表的元数据，包括表的元数据和表内的文件数据，它会首先清楚表的缓存，然后从metastore中重新加载全部数据并缓存，该操作代价比较重，主要用于在hive中修改了表的元数据，需要同步到impalad，例如create table/drop table/alter table add columns等。

INVALIDATE METADATA;                   //重新加载所有库中的所有表
INVALIDATE METADATA [table]            //重新加载指定的某个表
```

##### refresh

```java
REFRESH是用于刷新某个表或者某个分区的数据信息，它会重用之前的表元数据，仅仅执行文件刷新操作，它能够检测到表中分区的增加和减少，主要用于表中元数据未修改，数据的修改，例如INSERT INTO、LOAD DATA、ALTER TABLE ADD PARTITION、LLTER TABLE DROP PARTITION等，如果直接修改表的HDFS文件（增加、删除或者重命名）也需要指定REFRESH刷新数据信息。


REFRESH [table]                             //刷新某个表
REFRESH [table] PARTITION [partition]       //刷新某个表的某个分区
```

##### refresh&invalidate metadata 的区别

```java
invalidate metadata操作比refresh要重量级
如果涉及到表的schema改变，使用invalidate metadata [table]
如果只是涉及到表的数据改变，使用refresh [table]
如果只是涉及到表的某一个分区数据改变，使用refresh [table] partition [partition]
禁止使用invalidate metadata什么都不加，宁愿重启catalogd。
```

### impala shell

```java
-h, --help显示帮助信息
-v or --version显示版本信息
-i hostname, --impalad=hostname指定连接运行impalad 守护进程的主机。默认端口是21000。
-q query, --query=query从命令行中传递一个shell 命令。执行完这一语句后shell 会立即退出。
-f   query_file, --query_file= query_file传递一个文件中的SQL 查询。文件内容必须以分号分隔
-o filename or --output_file filename保存所有查询结果到指定的文件。通常用于保存在命令行使用-q 选项执行单个查询时的查询结果。
-c查询执行失败时继续执行
-d default_dbor --database=default_db指定启动后使用的数据库，与建立连接后使用use语句选择数据库作用相同，如果没有指定，那么使用default数据库
-r or --refresh_after_connect建立连接后刷新Impala 元数据
-p, --show_profiles对shell 中执行的每一个查询，显示其查询执行计划
-B（--delimited）去格式化输出
--output_delimiter=character 指定分隔符
--print_header打印列名

> impala-shell -i hadoop103
  连接指定hadoop103的impala主机

> impala-shell -q 'select * from student' -o output.txt
  查询数据并输出到文件
```

### impala 的数据类型

```java
Hive数据类型	Impala数据类型		长度
TINYINT			TINYINT			1byte有符号整数
SMALINT			SMALINT			2byte有符号整数
INT				INT				4byte有符号整数
BIGINT			BIGINT			8byte有符号整数
BOOLEAN			BOOLEAN			布尔类型，true或者false
FLOAT			FLOAT			单精度浮点数
DOUBLE			DOUBLE			双精度浮点数
STRING			STRING			字符系列。可以指定字符集。可以使用单引号或者双引号。
TIMESTAMPT		IMESTAMP		时间类型
BINARY			不支持			  字节数


注意：Impala虽然支持array，map，struct复杂数据类型，
但是支持并不完全，一般处理方法，将复杂类型转化为基本类型，通过hive创建表
```

#### 数据存储和压缩

### impala sql

```java
创建database
CREATE DATABASE [IF NOT EXISTS] database_name
		[COMMENT database_comment]
        [LOCATION hdfs_path];

显示数据库
> show databases;
查看数据库描述
> desc databases hive_db;
删除数据库
> drop database hive_db;
Impala不支持alterdatabase语法当数据库被USE 语句选中时，无法删除

创建内部表
 create table if not exits student2(
    id int,name string
   ) row format delimited fields terminated by '\t'
     stored as textfile
     location '/user/hive/warehouse/student2'

创建外部表
create external table stu_external(
   id int,
   name string
 )row format delimited fields terminated by '\t';

创建分区表
create table stu_par(id int, name string)
    partitioned by (month string)
    row format delimited
    fields terminated by '\t';
    如果分区没有，load data导入数据时，不能自动创建分区
> load  data  inpath  '/student.txt'  into  table  stu_par partition(month='201810')

增加多个分区
alter table stu_par add partition (month='201812') partition (month='201813')

show partitions stu_par
```

#### impala 查询

```java
1) 基本语法和hive的查询语句基本一样
2) impala不支持 cluster by, distribute by ,sort by
3) impala中不支持分桶表
4) impala不支持collect_set(col) 和 explode(col) 函数
5) impala 支持开窗函数
    select name,orderdata,cost,sum(cost) over (partition by month(orderdate)) from business;
```

#### 自定义函数

```java
maven 依赖
<dependency>
  <groupId>org.apache.hive</groupId>
  <artifactId>hive-exec</artifactId>
  <version>1.2.1</version>
</dependency>

创建UDF函数

import org.apache.hadoop.hive.ql.exec.UDF;
public class Lower extends UDF{
	public String evaluate(final String s){
    	if(s==null){
        	return null;
        }
        return s.toLowerCase();
    }
}

将jar上传到hdfs hdfs dfs -put xxx.jar hz://cluster-12/user/ad/

使用自定义函数

> create function myLower(string) returns string location 'hz://cluster-12/user/ad/xxx.jar' symbol='com.aouo.Lower';
> select Lower(name) from db.test;

show functions; 查看函数
```

### impala 和 hive 对比

```java
https://blog.csdn.net/wyqwilliam/article/details/81073381
```

```java
https://blog.csdn.net/Gemini_two/article/details/78992484
```
