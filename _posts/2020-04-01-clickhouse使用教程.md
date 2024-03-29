---
layout: post
title: clickhouse使用教程
categories: [olap, clickhouse]
description: clickhouse使用教程
keywords: clickhouse
---

 <meta name="referrer" content="no-referrer"/>

### clickhouse 数据类型

```java




UInt8，UInt16，UInt32，UInt64，Int8，Int16，Int32，Int64


Date
    一个Date。以1970-01-01（无符号）以来的天数存储在两个字节中。
    允许在Unix Epoch刚刚开始之后将值存储到编译阶段（目前，日期到2038年，但可能扩展到2106）由常量定义的上限。最小值输出为0000-00-00。
日期存储没有时区。

DateTime
	日期与时间。
    以四个字节存储为Unix时间戳（无符号）。
    允许将值存储在与日期类型相同的范围内。
    最小值输出为0000-00-00 00:00:00。时间储存精度高达1秒（不闰秒）。



Array(T)
	T 类型的数组。
    T型可以是任何类型，包括数组。
    我们不推荐使用多维数组，因为它们不被很好的支持（例如，除了内存表之外，你不能在多维数组中存储多维数组）。

元组（T1，T2，...）
	元组不能写入表（除了内存表）。
    它们用于临时列分组。
    在查询中使用IN表达式时，可以对列进行分组，并指定lambda函数的某些形式参数。
    有关更多信息，请参阅“IN运算符”和“高阶函数”。
元组可以作为运行查询的结果输出。在这种情况下，对于JSON *以外的文本格式，括号中的值用逗号分隔。在JSON *格式中，元组输出为数组（在方括号中）。

嵌套的数据结构
	嵌套（Name1 Type1，Name2 Type2，...）
	嵌套的数据结构就像一个嵌套的表。
	嵌套数据结构的参数 - 列名和类型 - 与在CREATE查询中的指定方式相同。
	每个表的行可以对应于嵌套数据结构中的任意数量的行。
    例如：
    CREATE TABLE test.visits(
      CounterID UInt32, StartDate Date,Sign Int8,IsNew UInt8, VisitID UInt64, UserID UInt64,
      Goals Nested (
          ID UInt32,
        Serial UInt32,
        EventTime DateTime,
        Price Int64,
        OrderID String,
        CurrencyID UInt32
       ),
     )ENGINE = CollapsingMergeTree(StartDate, intHash32(UserID), (CounterID, StartDate, intHash32(UserID), VisitID), 8192, Sign)

此示例声明了“目标”嵌套数据结构，其中包含有关转换的数据（达到的目标）。“访问”表中的每一行可以对应于零或任意数量的转换。
只支持一个嵌套级别。
在大多数情况下，使用嵌套数据结构时，会指定其各个列。为此，列名用点分隔。这些列组成一个匹配类型的数组。单个嵌套数据结构的所有列数组具有相同的长度。

例如：

SELECT Goals.ID, Goals.EventTime FROM test.visits WHERE CounterID = 101500 AND length(Goals.ID) < 5 LIMIT 10


链接：



```

> _参考:_[_https://www.jianshu.com/p/614f0c45d541_](https://www.jianshu.com/p/614f0c45d541)

```sql
1) UInt8, UInt16, UInt32, UInt64, Int8,Int16,Int32,Int64
    固定长度的整型, 包括有符号整型或无符号整型

2) Float32 Float64

3) Decimal(P,S)  Decimal32(S)  Decimal64(S) Decimal128(S)
    有符号的定点数,可在加,减和乘法运算过程中保持精度。
    十进制值范围
    Decimal32(S) - ( -1 * 10^(9 - S), 1 * 10^(9 - S) )
    Decimal64(S) - ( -1 * 10^(18 - S), 1 * 10^(18 - S) )
    Decimal128(S) - ( -1 * 10^(38 - S), 1 * 10^(38 - S) )

4) Boolean Values
    没有单独的类型来存储布尔值。可以使用UInt8类型, 取值限制为0或1

5) String
    字符串可以任意长度的。它可以包含任意的字节集, 包含空字节。
        因此,字符串类型可以代替其他DBMS的VARCHAR BLOB COLB等类型

    ClickHouse没有编码的概念,字符串可以是任务的字节集，按它们原本的方式进行存储和输出。
    若需存储文本, 我们建议使用UTF-8编码。这样我们在客户端使用UTF-8读写时就不需要任何的转换。
    对不同的编码文本clickhouse有不同的处理字符串的函数。

6) FixedString
    固定长度N的字符串 (N必须是严格的正自然数)
    当数据的长度恰好为N个字节时,FixedString类型是高效的。其他情况下可能会降低效率

    当向ClickHouse中插入数据时,
        如果字符串包含的字节数少于`N’,将对字符串末尾进行空字节填充。
        如果字符串包含的字节数大于N,将抛出Too large value for FixedString(N)异常。
    当做数据查询时，ClickHouse不会删除字符串末尾的空字节。 如果使用WHERE子句，则须要手动添加空字节以匹配FixedString的值。

7) UUID
    -- CREATE TABLE t_uuid (x UUID, y String) ENGINE=TinyLog
    -- insert into t_uuid select generateUUIDv4(),'ssgao';
    -- insert into t_uuid (y) values ('chenlin');

    select * from t_uuid ;
    40da4b79-9d14-4b60-b804-8ef0868669bb	ssgao
    00000000-0000-0000-0000-000000000000	chenlin //不使用generateUUIDv4 则为全0

8) Date
    日期类型,用两个字节存储,表示从1970-01-01(无符号)到当前的日期值。
    最小值为0000-00-00

9) DateTime64
    存储时间,时区信息存储在元数据中
    CREATE  table dt (`timestamp` DateTime64(3,'Europe/Moscow'), `event_id` UInt8) engine =TinyLog;
        timestamp	DateTime64(3, 'Europe/Moscow')
        event_id	UInt8
    insert into dt values (1546300800000, 1), ('2019-01-01 00:00:00', 2);
    select * from dt;
    2019-01-01 03:00:00.000	1
    2019-01-01 00:00:00.000	2
    *> 将datetime作为整数插入时,UTC时间戳。1546300800000表示'2019-01-0 00:00:00'由于指定了UTC+3时区显示为'2019-01-0 03:00:00'
    *> 当将字符串作为datetime插入时,它被视为位于列时区中2019-01-01 00:00:00'将被视为在UTC+3时区，并存储为1546290000000。


10) DateTime
    时间戳类型。用四个字节(无符号)存储Unix时间戳
    允许存储与日期类型相同的范围内的值。最小值为 0000-00-00 00:00:00。时间戳类型值精确到秒（没有闰秒）。


11) Enum8,Enum16
    Enum保存'string'=integer的对应关系
    所有含有Enum数据类型的操作都是按照包含整数的值来执行。这在性能方面比使用String数据类型更有效。
    create table t_enum
      (x Enum8('hello'=1,'world'=2) )
    engine=TinyLog
    insert into t_enum values ('hello'), ('world');
    select x,cast(x,'Int8') from t_enum;
    hello	1
    world	2

    Enum8类型的每个值范围是-128~127,Enum16类型的每个值范围是-32768~32767
    Enum中的字符串和数值都不能是NULL,如果可以为NULL,需要将Enum包含在Nullable类型中。
    create table t_enum_nullable
     ( x Nullable(Enum8('Hello'=1,'World'=2)))
    engine=TinyLog
    insert into t_enum_nullable values ('Hello'), ('World'),(NULL)
    select x,cast(x,'Int8') from t_enum_nullable where x is not null;
    Hello	1
    World	2
    1) 当以文本方式读取的时候，ClickHouse 将值解析成字符串然后去枚举值的集合中搜索对应字符串。
       如果没有找到，会抛出异常。
       当读取文本格式的时候，会根据读取到的字符串去找对应的数值。如果没有找到，会抛出异常。

    2) 当以文本形式写入时，ClickHouse 将值解析成字符串写入。
       如果列数据包含垃圾数据（不是来自有效集合的数字),则抛出异常。
       Enum 类型以二进制读取和写入的方式与 Int8 和 Int16 类型一样的。
    隐式默认值是数值最小的值。
    在 ORDER BY，GROUP BY，IN，DISTINCT 等等中，Enum 的行为与相应的数字相同。
    例如，按数字排序。对于等式运算符和比较运算符，Enum 的工作机制与它们在底层数值上的工作机制相同



12) Array(T)
    T可以是任意类型,包含数组类型。但不推荐使用多维数组。
    SELECT array(1, 2) AS x, toTypeName(x)
    [1,2]	Array(UInt8)

    ClickHouse会自动检测数组元素 。
    如果在元素中存在 NULL 或存在 Nullable 类型元素，那么数组的元素类型将会变成 Nullable。
    如果 ClickHouse 无法确定数据类型，它将产生异常。
    SELECT array(1,null) AS x, toTypeName(x)
    [1,0]	Array(Nullable(UInt8))


14) Tuple(T1,T2,...)
    元组,其中每个元素都有单独的类型
    不能在表中存储元组(除了内存表), 他们可以使用临时列分组
    select tuple(1,'1') as x , toTypeName(x)
    (1,'1')	Tuple(UInt8, String)




15) Nullable(TypeName)
    允许用特殊标记 (NULL) 表示«缺失值», 可以与TypeName的正常值存放一起。
    例如，Nullable(Int8) 类型的列可以存储Int8类型值，而没有值的行将存储 NULL。
    对于TypeName 不能使用复合数据类型Array和Tuple。复合数据类型可以包含Nullable类型值,例如Array(Nullable(Int8))。
    Nullable类型字段不能包含在表索引中。除非在ClickHouse服务器配置中另有说明，否则NULL是任何 Nullable 类型的默认值。


```

```sql
lickhouse支持parquent格式的数据导入和导出

parquent data type(insert) | clickhouse data type | Parquent data type (select)

UINT8,BOOL  --> UInt8       --> Uint8
INT8        ——> Int8        ——> INT8
UINT16      ——> UInt16      ——> UINT16
INT16       ——> Int16       ——> INT16
UINT32      ——> UInt32      ——> UINT32
INT32       ——> Int32       ——> INT32
UINT64      ——> UInt64      ——> UINT64
INT64       ——> Int64       ——> INT64
FLOAT
HALF_FLOAT  ——> Float32     ——> FLOAT
DOUBLE      ——> Float64     ——> DOUBLE
DATE32      ——> Date        ——> UINT16
DATE64      ——> Date        ——> UINT32
STRING      ——> String      ——> STRING

clickhouse表的列名必须与parquent表的列名一致。
  clickhouse表的列数据类型可以不同于插入的parquent数据类型。
  在插入数据时,clickhouse根据上表解释数据类型,然后将数据类型转换为clickhouse表的列数据类型。


数据的导出
    clickhouse-client  --query="select * from xxx format Parquet" > parquent_demo.parquent

```

## 表引擎

```sql
引擎(表的类型)

 1) 数据的存储方式和位置,写到哪里以及从哪里读取数据
 2) 支持哪些查询以及如何支持
 3) 并发数据访问
 4) 索引的使用(如果存在)
 5) 是否可以执行多线程请求
 6) 数据复制参数


TinyLog引擎,表示一个小表,并且一般不会经常改动的表
   1) 存在磁盘中
   2) 不支持索引
   3) 没有并发控制, 多个写会有问题。
 使用场景: 小表,固定不变的表, 使用与枚举类,比如国家名,行业名称,兴趣列表

 >>>create table stu1 (id Int8, name String) engine=TinyLog;
--------------------------------------------------------------------------------------------

memory引擎
    内存引擎,数据以未压缩的原始形式直接保存在内存当中,服务器重启数据就会消失。
    读写操作不会相互阻塞,不支持索引。简单查询下有非常非常高的性能表现(超过10G/s)
    一般用到它的地方不多,除了用来测试,就是需要非常高的性能,同时数据量又不太大(上限大概1亿行)的场景。

    1) 数据在内存中
    2) 不支持索引
--------------------------------------------------------------------------------------------

Merge引擎
    1) 本身不存储数据,但是用于从多张其他的表中读取数据。
    2) 读数据是自动并行的,不支持写入。
    3) 读取时,哪些被真正读取到数据的表的索引(如果有的话)会被使用。

    ps: merge引擎的参数：一个数据库名,一个用于匹配表名的正则表达式
    实例
      先创建t1,t2,t3三个表,然后使用merge引擎的t表再把它们链接起来
       create table t1(id uint16, name String) engine=TinyLog;
       create table t2(id uint16, name String) engine=TinyLog;
       create table t3(id uint16, name String) engine=TinyLog;

    create table t (id UInt16,name String) ENGINE=Merge(currentDatabase(),'^t'); 所有以t开头的数据表


```

### clickhouse_file

```sql
file 引擎

    file表引擎按照支持格式(TabSeparated,CSV等) 将数据保存文件中。

使用场景:
    1) 数据从clickhouse导出文件
    2) 将数据从一种格式转换为另外一种格式
    3) 通过编辑磁盘上的文件更新clickhouse中的数据


指定表引擎
    engine=File(Format)
    Format参数指定了文件格式
    Clickhouse不支持为File引擎指定文件系统系统路径

    当使用File(Format) 创建表时,它会在该文件夹中创建空子目录。
    当数据写入该表时,它将数据写入子目录下的文件data.Format文件中。


dsp-bridge1.hz.163.org :) create table file_engine_t
			(name String,value UInt32) engine = File(CSV);


```

### clickhouse_hdfs

```java
使用hdfs引擎
create table hdfs_student_csv(
 id Int8,
 name String
)
Engine=HDFS('hdfs://hadoop102:9000/student.csv','csv');




create table student_local(
  id Int8,
  name String
) ENGINE=TinyLog;

insert into student_local select * from hdfs_student_csv;



```

### clickhouse_mysql

```java

mysql引擎可实现对mysql数据的表执行插入和查询操作
    clickhouse表结构,可以不同于原始的mysql表结构。
    列名应当与原始mysql表中的列名相同,但可以按任意顺序使用其中的一些列。
    列的数据类型可能与原始的mysql表中的列类型不同,clickhouse尝试进行数据类型转换。



指定表引擎
  engine=MySQL('host:port','database','table','user','password' [,replace_query, 'on_duplicate_clause']);
  引擎参数:
      host:port  mysqlserver的地址
      database   mysql数据库名称
      table      mysql表名
      user       mysql用户名
      password   mysql密码

      replace_query:
                将insert into查询转换为replace into查询的标识。如果replace_query=1,查询将被替换。
      on_duplicate_clause：
                将on duplicate key 'on_duplicate_clause'表达式添加到insert查询中。
                例如 insert into t(c1,c2) values('a',2) on duplicate key update c2=c2+1
                on_duplicate_clause的表达式为: update c2=c2+1;
                如果使用on_duplicate_clause,需要将参数replace_query=0
                如果同时传递replace_query=1和on_duplicate_clause,Clickhouse会抛出异常。

--------------------------------------------------------------------------------------------------------------
create table mysql_table(
    id Int32,
    cnt Int32
) engine=MySQL('127.0.0.1:3306','test_database','test_table','ad','123456',0,'UPDATE cnt=cnt+1');


```

### clickhouse_kafka

```java

Kafka引擎结合kafka使用,可实现订阅或发布数据流
engine=Kafka()
    settings
        kafka_broker_list='host:port',
        kafka_topic_list='topic1,topic2,topic3,...',
        kafka_group_name='group_name',
        kafka_format='data_format'[,]
        [kafka_row_delimiter='delimiter_symbol',]
        [kafka_schema='',]
        [kafka_num_consumers=N,]
        [kafka_skip_broken_messages=N]
必选参数:
    kafka_broker_list   以逗号分隔的brokers列表
    kafka_topic_list    以逗号分隔的kafka的topic列表
    kafka_group_name    kafka消费组
    kafka_format        消息的格式,例如JSONEachRow
可选参数:
    kafka_row_delimiter 行之间的分隔符
    kafka_schema        按需定义schema
    kafka_num_consumers 消费者数量,默认1,最多不超过topic的分区数
    kafka_skip_broken_messages  每个block中,kafka消息解析器容忍schema不兼容消息的数量。默认为0
--------------------------------------------------------------------------------------------------------






create table queue(
    timestamp UInt64,
    level String,
    message String
) engine=kafka('dsp-bridge1.hz.163.org:9092','topic','group1','JSONEachRow');


create table kafka_queue_2(
    timestamp UInt64,
    level String,
    message String
) engine=kafka('dsp-bridge1.hz.163.org:9092','topic','group1')
    SETTINGS kafka_format='JSONEachRow',
             kafka_num_consumers=4;


create table kafka_queue_2(
    timestamp UInt64,
    level String,
    message String
) engine = Kafka SETTINGS
           kafka_broker_list='dsp-bridge1.hz.163.org:9092',
           kafka_topic_list='topic',
           kafka_group_name='group1',
           kafka_format='JSONEachRow',
           kafka_num_consumers=4;

----------------------------------------------------------------------------------------------------------------
kafka引擎
select查询对于读取消息并不是很有用(除了调试),因为每个消息只能读取一次。
通常,我们将该引擎结合物化视图一起使用, 使用方法如下:
    1) 使用kafka引擎创建一个kafka的消费者,并将其视为一个数据流
    2) 创建所需结构的表 (如MergeTree表)
    3) 创建一个物化视图,该视图转换来自引擎的数据并将其放入上一步创建的表中。

当物化视图添加至该引擎,它将会在后台收集数据。
这就允许你从kafka持续接收消息,并使用select将数据转换为所需的格式。它们不直接从kafka中读取数据,而是接收新记录,以block为单位,
这样就可以写入具有不同详细信息级别的多个表(分组聚合或无聚合)中。

为了提高性能,接收到的消息将被分组为大小为max_insert_block_size的block(块)。
如果block没有在stream_flush_interval_ms时间内形成,则不管block完整性如何,数据都将刷新到表中。


----------------------------------------------------------------------------------------------------------------
要停止接收topic数据,或更改转换逻辑,需要detach物化视图(和表的关系隔离开)。
    Detach table consumer; (detch用户将表)
    ATTACH MATERIALIZED VIEW consumer;

  如果要使用alter更改目标表,建议禁用物化视图, 以避免目标表和该视图中的数据之间出现差异。


----------------------------------------------------------------------------------------------------------------
kafka扩展配置

    kafka引擎支持使用clickhouse配置文件扩展配置
    用户可以使用两个配置key,全局的kafka和topic级别的kafka_*。首先应用全局配置,然后应用topic级别的配置
    具体的配置,在目录/etc/clickhouse-server/config.d/ 新建配置文件,配置文件名称任意指定,这里命令为kafka.xml.具体配置如下:
    <yandex>
        <!--Global configuration options for all tables of kafka engine type -->
        <kafka>
            <debug>cgrp</debug>
            <auto_offset_reset>smallest</auto_offset_reset>
        </kafka>

        <!--Configuration specific for topic 'logs'-->
        <kafka_logs>
            <retry_backoff_ms>250</retry_backoff_ms>
            <fetch_min_bytes>100000</fetch_min_bytes>
        </kafka_logs>

        <!-- Configuration specific for topic 'topic_ch' -->
        <kafka_topic_ch>
            <auto_offset_reset>lastest</auto_offset_reset>
            <retry_backoff_ms>250</retry_backoff_ms>
            <fetch_min_bytes>100000</fetch_min_bytes>
        </kafka_topic_ch>
    <yandex>

    更多的相关配置,参见librdkafka配置。
    clickhouse配置中使用下划线(_)代替点,例如,check.crcs=true将配置为<check_crcs>true</check_crcs>



----------------------------------------------------------------------------------------------------------------

create table kafka_stream_table (
    timestamp UInt64,
    level String,
    message String
) engine=Kafka('dsp-bridge1.hz.163.org:9092','topic_ch', 'group_ch','JSONEachRow');

创建物化明细表
create table kafka_table(
    timestamp UInt64,
    level String,
    message String
) engine=MergeTree()
  order by (timestamp);

创建物化视图
create MATERIALIZED VIEW kafka_topic_view to kafka_table
   AS select timestamp,level,message from kafka_stream_table;





创建物化聚合表
create table kafka_table_daily(
    day Date,
    level String,
    total UInt64
) engine=SummingMergeTree(day)
  order by (day,level);

create materialized view kafka_table_daily_view to kafka_table_daily
as select toDate(toDateTime(timestamp)) as day,level, count() as total
from topic_ch_kafka group by day,level;


create table kafka_action_test_demo2(
    act String,
    act_block_ad Int8,
    ad_id String,
    ad_serve_id String,
    ad_type String,
    api_type String,
    flight_id String,
    imei String,
    flight_name String,
    template_id String,
    murs String
) engine = Kafka SETTINGS kafka_broker_list='music-iad-bigdata-kafka1.jd.163.org:9092',
           kafka_topic_list='iad_adx_act_preformat_out_zeus_v2.online',
           kafka_group_name='group_act_test',
           kafka_format='JSONEachRow',
           kafka_num_consumers=2;


dsp-bridge1.hz.163.org :) select * from kafka_action_test_demo2 limit 10;
┌─act─┬─act_block_ad─┬─ad_id─┬─ad_serve_id─────────┬─ad_type─┬─api_type─┬─flight_id──┬─imei────────────┬─flight_name─────────────┬─template_id─┬─murs───────┐
│ 3   │            1 │ nil   │ 8374201774015529719 │ 1       │ nil      │ 90000053   │ A0000072BD6135  │ 云音乐搜索页ANDROID     │ 2004004     │ 3347095729 │
│ 3   │            1 │ nil   │ 3628227855272540506 │ 2       │ nil      │ 90000069   │ 860128040768119 │ 云音乐新评论页Android   │ 2009001     │ 1954913468 │
│ 3   │            1 │ nil   │ 1009452718697095055 │ 1       │ nil      │ 90000053   │ 864326037599218 │ 云音乐搜索页ANDROID     │ 2004004     │ 3441470982 │
│ 3   │            1 │ nil   │ 1845278961410040634 │ 2       │ nil      │ 90000053   │ 861537048701717 │ 云音乐搜索页ANDROID     │ 2004004     │ 2072321549 │
│ 3   │            1 │ nil   │ 7788552360257480656 │ 2       │ nil      │ 90000069   │ 860694049681728 │ 云音乐新评论页Android   │ 2009001     │ 1326285305 │
│ 3   │            1 │ nil   │ 8374201774015529719 │ 1       │ nil      │ 90000053   │ 869454032918780 │ 云音乐搜索页ANDROID     │ 2004004     │ 1435524234 │
│ 3   │            1 │ nil   │ 1382599935745634089 │ 2       │ nil      │ 90000053   │ 867063040306946 │ 云音乐搜索页ANDROID     │ 2004004     │ 1875309145 │
│ 3   │            1 │ nil   │ 6672601011427528016 │ 2       │ nil      │ 90000052   │                 │ 云音乐搜索页IOS         │ 2004004     │ 1846948399 │
│ 3   │            1 │ nil   │ 7300208293685399152 │ 2       │ nil      │ 1000004706 │ 860128040768119 │ 单曲评论页楼中楼Android │ 2009001     │ 1954913468 │
│ 3   │            1 │ nil   │ 1799958754807052259 │ 1       │ nil      │ 90000053   │ 868198046271170 │ 云音乐搜索页ANDROID     │ 2004004     │ 1474249658 │
└─────┴──────────────┴───────┴─────────────────────┴─────────┴──────────┴────────────┴─────────────────┴─────────────────────────┴─────────────┴────────────┘
10 rows in set. Elapsed: 2.487 sec. Processed 65.54 thousand rows, 12.08 MB (26.35 thousand rows/s., 4.86 MB/s.)


dsp-bridge1.hz.163.org :) show tables;
SHOW TABLES
┌─name────────────────────┐
│ kafka_action_test_demo2 │
└─────────────────────────┘

dsp-bridge1.hz.163.org :) detach table kafka_action_test_demo2;
DETACH TABLE kafka_action_test_demo2

Received exception from server (version 20.5.4):
Code: 60. DB::Exception: Received from dsp-bridge1.hz.163.org:9401. DB::Exception: Table myclickhouse.kafka_action_test_demo2 doesn't exist..

dsp-bridge1.hz.163.org :) show tables;
SHOW TABLES




```

### clickhouse_distribute

```java

分布式表基于Distributed引擎创建,在多个分片上运行分布式查询。
读取是自动并行化的,可使用远程服务器上的索引(如果有)
    数据在请求的本地服务器上尽可能的被部分处理。例如,对于group by 查询,数据将在远程服务器上聚合,
    聚合函数的中间状态将发送到请求服务器,然后数据将进一步聚合。


创建分布式表:
    engine=Distributed(cluster_name,db_name,table_name[,sharding_key[,policy_name]])
    参数:
    cluster_name: 集群名称
         db_name: 数据库名称,可使用常量表达式currentDatabase()
      table_name: 各个分片上的表名称
    sharding_key: 分片的key(可选),可设置为rand().
     policy_name: 策略名称,用户存储异步发送的临时文件。



---------------------------------------------------------------------------------------------------------------------

分布式表只是一个查询引擎,本身不存储任何数据,查询时将sql发送到所有集群分片,然后进行处理和聚合后将结果返回给客户端
因此clickhouse限制聚合结果大小,不能大于分布式表节点的内存,这个条件,以便都不会超过。

分布式表可以所有节点都创建,也可以只在一部分节点创建,一般建议设置多个,当某个节点挂掉时,可以查询其他节点上的表。

---------------------------------------------------------------------------------------------------------------------
创建本地复制表
    create table table_local_a  on cluster cluster_3shards_2replicas
       (event_date DateTime,
        counter_id UInt32,
        user_id UInt32
       ) engine = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/table_local_a','{replica}')
       partition by toYYYYMM(event_date)
       order by (counter_id,event_date,user_id);

创建分布式表
    create table table_distributed as table_local
        engine = Distributed(cluster_3shards_2replicas,myclick_house,table_local_a,rand());


dsp-bridge1.hz.163.org :) create table table_distributed as table_local  engine = Distributed(cluster_3shards_2replicas,myclick_house,table_local_a,rand());
CREATE TABLE table_distributed AS table_local
ENGINE = Distributed(cluster_3shards_2replicas, myclick_house, table_local_a, rand())
Ok.
0 rows in set. Elapsed: 0.002 sec.

dsp-bridge1.hz.163.org :)


```

### clickhouse_bitmap

```java

CREATE TABLE murs_tag_bitmap_test
(
    `tag_name` String,
    `tag_value` String,
    `user_bit` AggregateFunction(groupBitmap, UInt32),
    `day` String
)
ENGINE = AggregatingMergeTree()
PARTITION BY day
ORDER BY tag_name
SETTINGS index_granularity = 2


insert into murs_tag_bitmap_test
select 'gender',gender,groupBitmapState(murs),'2020-08-20' from ssgao_murs_test where day='2020-08-20' group by gender;
----------------------------------------------------------------------------------------------------------------------
iad-kafka1.jd.163.org :) select tag_name,tag_value,day, bitmapCardinality(user_bit) from murs_tag_bitmap_test;
    SELECT
        tag_name,
        tag_value,
        day,
        bitmapCardinality(user_bit)
    FROM murs_tag_bitmap_test
    ┌─tag_name─┬─tag_value─┬─day────────┬─bitmapCardinality(user_bit)─┐
    │ gender   │ female    │ 2020-08-20 │                    25352722 │
    └──────────┴───────────┴────────────┴─────────────────────────────┘
    ┌─tag_name─┬─tag_value─┬─day────────┬─bitmapCardinality(user_bit)─┐
    │ gender   │ nil       │ 2020-08-20 │                     7201214 │
    └──────────┴───────────┴────────────┴─────────────────────────────┘
    ┌─tag_name─┬─tag_value─┬─day────────┬─bitmapCardinality(user_bit)─┐
    │ gender   │ male      │ 2020-08-20 │                    29493771 │
    └──────────┴───────────┴────────────┴─────────────────────────────┘
----------------------------------------------------------------------------------------------------------------------
iad-kafka1.jd.163.org :) show create ssgao_murs_test;
 CREATE TABLE iad_big_data.ssgao_murs_test
(
    `murs` UInt32,
    `flight_name` String,
    `ad_type` String,
    `gender` String,
    `network_status` String,
    `age` String,
    `ip_city` String,
    `day` String
)
ENGINE = MergeTree()
PARTITION BY day
ORDER BY (murs, flight_name)
TTL now() + toIntervalDay(4)
SETTINGS index_granularity = 8192

iad-kafka1.jd.163.org :) select network_status,count(*) from ssgao_murs_test where day='2020-08-20' group by network_status;
┌─network_status─┬───count()─┐
│ wifi           │ 162115834 │
└────────────────┴───────────┘
┌─network_status─┬───count()─┐
│ wwan           │ 166025315 │
└────────────────┴───────────┘
----------------------------------------------------------------------------------------------------------------------
insert into murs_tag_bitmap_test
select 'network_status',network_status,groupBitmapState(murs),'2020-08-20' from ssgao_murs_test
    where day='2020-08-20' group by network_status;

insert into murs_tag_bitmap_test
select 'age',age,groupBitmapState(murs),'2020-08-20' from ssgao_murs_test
    where day='2020-08-20' group by age;

----------------------------------------------------------------------------------------------------------------------

1.bitmapCardinality
返回一个UInt64类型的数值，表示bitmap对象的基数。
用来计算不同条件下的用户数，可以粗略理解为count(distinct)

2.bitmapAnd
为两个bitmap对象进行与操作，返回一个新的bitmap对象。
可以理解为用来满足两个条件之间的and，但是参数只能是两个bitmap

3.bitmapOr
为两个bitmap对象进行或操作，返回一个新的bitmap对象。
可以理解为用来满足两个条件之间的or，但是参数也同样只能是两个bitmap。
如果是多个的情况，可以尝试使用groupBitmapMergeState

SELECT bitmapCardinality(bitmapAnd(
    (
        SELECT user_bit
        FROM murs_tag_bitmap_test
        WHERE (day = '2020-08-20') AND (tag_value = 'BETWEEN_26_35')
    ),
    (
        SELECT user_bit
        FROM murs_tag_bitmap_test
        WHERE (day = '2020-08-20') AND (tag_value = 'female')
    )))

┌─bitmapCardinality(bitmapAnd(_subquery3, _subquery4))─┐
│                                              4226329 │
└──────────────────────────────────────────────────────┘


select bitmapToArray( bitmapAnd(
    (select user_bit from murs_tag_bitmap_test where day='2020-08-20' and tag_value='BETWEEN_26_35')
    ,
    (select user_bit from murs_tag_bitmap_test where day='2020-08-20' and tag_value='female')
    )) ;


```

### mergeTree

```java
MergeTree
  clickhouse中最强大的表引擎当属MergeTree(合并树)引擎及该系列(*MergeTree)中的其他引擎。
  MergeTree引擎系列的基本理念如下:
        当我们有巨量数据要插入到表中,我们要高效的一批批写入数据片段,并希望这些数据片段在后台按照一定规则合并。
        相比在插入时不断修改(重写)数据进行存储,这种策略会高效很多。
        特点如下:
           1) 数据按照主键排序 //创建mergeTree表时,需要指定主键
           2) 可以使用分区(如果指定了主键),类似hive表的分区
           3) 支持数据副本
           4) 支持数据采样


  格式:
    ENGINE=MergeTree() //引擎名和参数
    [PARTITION BY exor] //PARTITION BY 分区键,要按月分区,可以使用表达式 toYYYYMM(date_column) 只能按照日期分区,必须有个时间字段,到月分区
    [ORDER BY expr] // 表的排序键,可以是一组列的元组或任意的表达式,例如ORDER BY (id,name)
    [PRIMARY KEY expr] // 主键,不写的话就和order by的键一样, 如果写的话需要与排序键字段不同
    [SAMPLE BY expr] // 抽样表达式,如果要用抽样表达式,主键中必须包含这个表达式
    [SETTINGS name=value,...] // 影响MergeTree性能的额外参数
     (1) index_granularity: 索引粒度,即索引中相邻[标记]键的数据行数,默认值,8192
     (2) use_minimalistic_part_header_in_zookeeper: 数据片段,在zookeeper中的存储方式
     (3) min_merge_byte_to_use_direct_io: 使用直接IO,来操作磁盘合并操作要求的最小数据量

    create table m_table(data Date,id UInt8, name String)
    engine=MergeTree()
    partition by date
    order by (id,name) settings index_granularity=8192;

示例:注意大小写
------------------------------------------------------------------------------------------
create table merge_table(
 id Int16 comment 'id',
 date_time Date comment '日期',
 name String comment '名字',
 row_key String comment '排序'
)engine = MergeTree()
partition by toYYYYMM(date_time)
order by row_key
settings
	index_granularity=8192
-------------------------------------------------------------------------------------------


ENGINE 引擎名和参数  engine=MergeTree()  MergeTree引擎没有参数

PARTITION BY  分区键
    要按月分区,可以使用表达式`toYYYYMM(date_column)`, 这里的`data_column`是一个Date类型的列

ORDER BY 表的排序键
    可以是一组列的元组或任意的表达式。例如order by (CounterID,EventDate)

PRIMARY KEY 主键
    如果设置,必须和ORDER BY的键不同
    默认情况下主键要设成跟排序键相同 (由'order by '子句指定)

index_granularity 索引粒度, 索引中相邻标记间的数据行数。默认值 8192
    index_granularity_bytes 索引粒度, 以字节位单位, 默认值10MB. 如果仅按数据行数限制索引粒度,需要设置为0



```

#### 分区规则

```java
不指定分区键
	如果不使用分区键, 即不使用partition by声明任何分区表达式, 则分区ID默认取名为all, 所有的数据都会被写入到
    这个all分区。

使用整型
	如果分区键取值属于整型(兼容UInt64, 包括有符合整型和无符号整型), 且无法转换为日期类型YYYYMMDD格式,则直接按照该整型的字符形式输出作为分区ID的取值。

使用日期类型
	如果分区键取值属于日期类型,或者是能够转换为YYYYMMDD日期格式的整型,则使用按照YYYYMMDD日期格式化后的字符形式输出作为分区ID的取值。

使用其他类型
	如果分区键取值即不属于整型,也不属于日期类型, 例如String,Float 等。则通过128位Hash算法取其Hash值,作为分区ID的取值。

```

```java
PartitionID 分区ID
MinBlockNum&&MaxBlockNum
	最小数据块编号与最大数据块编号, 这里BlockNum 是一个整型的自增长编号。
    如果将其设为n的话,那么计数n在单张MergeTree数据表内全局累加, n从1开始,每当新创建一个分区目录时,计数n就会累积+1。对于一个新的分区目录而言, MinBlockNum与MaxBlockNum取值一样, 同等于n。

Level
	合并的层级, 可以理解为某个分区被合并过的次数, Level计数与BlockNum有所不同,它并不是全局累加的。
    对于每一个新创建的分区目录而言,其初始值均为0。之后, 以分区为单位
```

## 排序键

```java
选择和主键不一样的排序键

默认情况下,主键(由PRIMARY KEY指定)和排序键(ORDER BY)相同,因此,大部分情况下，不需要专门指定PRIMARY KEY子句
当使用summingMergeTree和AggregatingMergeTree时,可考虑选择和主键不一样的排序键。

1) 排序键可以修改,主键不能修改
2) 预聚合/增量聚合的key是由排序键指定的。业务逻辑后期可能会更改。
3) 查询条件无需包含排序键的所有字段。

---------------------------------------------------------------------------------------------------------
添加排序键需要注意的:
  > 主键字段是排序键字段的子集。
    例如主键为(A,B) 则排序键以(A,B)开头,排序键为(A,B)或(A,B,C)等
  > 旧的排序键是新排序键的前缀
    修改排序键只能增加排序字段,不能减少排序键字段,例如可修改排序键(A,B)为(A,B,C),但不能修改为(A,C)或(A,C,B)等
  > 排序键只能添加新加入表的列,表中已存在数据的列不能添加到排序键中。


---------------------------------------------------------------------------------------------------------



dsp-bridge1.hz.163.org :) create table t_merge_sum( id UInt32, name String, value UInt32) engine=SummingMergeTree() order by id;
CREATE TABLE t_merge_sum
(
    `id` UInt32,
    `name` String,
    `value` UInt32
)
ENGINE = SummingMergeTree()
ORDER BY id;

// 尝试将name列新增为排序键 (表中新增的列并不能添加排序键)
dsp-bridge1.hz.163.org :) alter table t_merge_sum modify order by (id, name);
ALTER TABLE t_merge_sum
    MODIFY ORDER BY (id, name)
Received exception from server (version 20.5.4):
Code: 36. DB::Exception: Received from dsp-bridge1.hz.163.org:9401. DB::Exception: Existing column name is used in the expression that was added to the sorting key. You can add expressions that use only the newly added columns.


// 添加新列new_name并将其新增为排序键 (只能通过新加的列来提供排序键)
dsp-bridge1.hz.163.org :) alter table t_merge_sum add column new_name String , modify order by (id,new_name);
ALTER TABLE t_merge_sum
    ADD COLUMN `new_name` String,
    MODIFY ORDER BY (id, new_name)
Ok.
0 rows in set. Elapsed: 0.002 sec.
// 查看表信息
dsp-bridge1.hz.163.org :) show create t_merge_sum;
┌─statement───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ CREATE TABLE myclick_house.t_merge_sum
(
    `id` UInt32,
    `name` String,
    `value` UInt32,
    `new_name` String
)
ENGINE = SummingMergeTree()
PRIMARY KEY id // 主键索引没有修改
ORDER BY (id, new_name)  // 排序键添加了新的列new_name
SETTINGS index_granularity = 8192 │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘





```

## 物化视图

```java
在用于插入数据的表上,创建多个物化视图, 每个物化视图根据业务需求对数据做转换。

1) 物化视图存储通过由相应的select查询转换的数据
2) 在数据插入期间做查询转换,压力分散
3) 仅在插入的单个数据块中聚合,数据不会进一步聚合 (物化视图存储的是单个数据块聚合后的数据)
4) 数据表的引擎可为NULL



create table t_table( date Date,id UInt8, name String)
engine =MergeTree()
partition by date
order by id;

------创建物化视图---------------------------------------------------------------------------
create materialized view t_materialized
engine=MergeTree() partition by date order by id
as select date,id,count(*) as cnt from t_table group by date,id;


dsp-bridge1.hz.163.org :) insert into t_table values ('2020-08-17',1,'ssgao'),('2020-08-17',1,'sssss') ;
dsp-bridge1.hz.163.org :) select * from t_table;
┌───────date─┬─id─┬─name──┐
│ 2020-08-17 │  1 │ ssgao │
│ 2020-08-17 │  1 │ sssss │
└────────────┴────┴───────┘
dsp-bridge1.hz.163.org :) select * from t_materialized;
┌───────date─┬─id─┬─cnt─┐
│ 2020-08-17 │  1 │   2 │
└────────────┴────┴─────┘

dsp-bridge1.hz.163.org :) insert into t_table values ('2020-08-17',1,'xxxx'),('2020-08-18',2,'sss'),('2020-08-18',2,'a'),('2020-08-18',3,'c'),('2020-08-18',3,'d') ;
dsp-bridge1.hz.163.org :) select * from t_table;
┌───────date─┬─id─┬─name──┐
│ 2020-08-17 │  1 │ ssgao │
│ 2020-08-17 │  1 │ sssss │
└────────────┴────┴───────┘
┌───────date─┬─id─┬─name─┐
│ 2020-08-17 │  1 │ xxxx │
└────────────┴────┴──────┘
┌───────date─┬─id─┬─name─┐
│ 2020-08-18 │  2 │ sss  │
│ 2020-08-18 │  2 │ a    │
│ 2020-08-18 │  3 │ c    │
│ 2020-08-18 │  3 │ d    │
└────────────┴────┴──────┘
dsp-bridge1.hz.163.org :) select * from t_materialized;
┌───────date─┬─id─┬─cnt─┐
│ 2020-08-17 │  1 │   2 │
└────────────┴────┴─────┘
┌───────date─┬─id─┬─cnt─┐
│ 2020-08-18 │  2 │   2 │
│ 2020-08-18 │  3 │   2 │
└────────────┴────┴─────┘
┌───────date─┬─id─┬─cnt─┐
│ 2020-08-17 │  1 │   1 │
└────────────┴────┴─────┘


```

## 二级索引

```java
CREATE TABLE iad_big_data.ssgao_murs_test_index2
(
    `murs` UInt32,
    `flight_name` String,
    `ad_type` String,
    `gender` String,
    `network_status` String,
    `age` String,
    `ip_city` String,
    `day` String,
    index in_f_n flight_name type tokenbf_v1(256,2,0) granularity 5,
    index in_age age type minmax granularity 5
)
ENGINE = MergeTree()
PARTITION BY day
ORDER BY (murs, flight_name)
TTL now() + toIntervalDay(4)
SETTINGS index_granularity = 8192;

ad@iad-kafka1:/data/clickhouse_node1_data/clickhouse/data/iad_big_data/ssgao_murs_test_index2/9c9278c8502718d4b114d1ee385a3ddc_1_1_0$ ls -l
total 108
-rw-r----- 1 ad netease  50 Aug 22 17:19 age.bin
-rw-r----- 1 ad netease  48 Aug 22 17:19 age.mrk2
-rw-r----- 1 ad netease  11 Aug 22 17:19 partition.dat
-rw-r----- 1 ad netease  16 Aug 22 17:19 primary.idx
-rw-r----- 1 ad netease  39 Aug 22 17:19 skp_idx_in_age.idx
-rw-r----- 1 ad netease  24 Aug 22 17:19 skp_idx_in_age.mrk2
-rw-r----- 1 ad netease  50 Aug 22 17:19 skp_idx_in_f_n.idx
-rw-r----- 1 ad netease  24 Aug 22 17:19 skp_idx_in_f_n.mrk2

------------------------------------------------------------------------------------------------------------------------
1) minmax:
    minmax索引记录了一段数据内的最小和最大极值
    其索引的作用类似分区目录的minmax索引,能够快速跳过无用的数据区间.
        index a id type minmax granularity 5
    minmax索引会记录这段数据区间内ID字段的极值。极值的计算涉及每5个index_granularity区间中的数据

2) set 索引直接记录了声明字段或表达式的取值(唯一值,无重复),其完整形式为set(max_rows)
    其中max_rows是一个阈值,表示在一个index_granularity内,索引最多记录的数据行数。
    如果max_rows=0,则表示无限制,例如
        index b (length(ID)*8) TYPE set(2) GRANULARITY 5;
        示例中set索引会记录数据中ID的长度*8后的取值。

3)  ngrambf_v1:ngrambf_v1索引记录的是数据短语的布隆过滤器,只支持String和FixedString数据类型。
      ngrambf_v1只能提升in,notin,like,equals和notEquals查询的性能,其完整形式为
        ngrambf_v1(n,size_of_bloom_fliter_in_bytes,number_of_hash_functions,random_seed)
      这些参数是一个布隆过滤器的标准输入,
            n: token长度, 依据n的长度,将数据切割为token短语
            size_of_bloom_filter_in_bytes: 布隆过滤器的大小
            number_of_hash_functions: 布隆过滤器中使用hash函数的个数
            random_seed: hash函数的随机种子

      示例: index c (id,code) type ngrambf_v1(3,256,2,0) GRANULARITY 5;
            ngrambf_v1索引会依照3的粒度将数据切割成短语token，token会经过2个hash函数映射后再被写入,布隆过滤器大小为256字节。

4) tokenbf_v1: tokenbf_v1索引是ngrambf_v1的变种,同样也是一种布隆过滤器索引。
               tokenbf_v1除了短语token的处理方法外,其他与ngrambf_v1是完全一样的。
               tokenbf_v1会自动按照非字符的,数字的字符串分割token,具体用法如下所示:
               index index_name id(要索引的字段) type tokenbf_v1(256,2,0) GRANULARITY  5；

------------------------------------------------------------------------------------------------------------


```

## clickhouse sql

### group

```java


dsp-bridge1.hz.163.org :) select id,count(*) from t_left_a group by id with totals;
┌─id─┬─count()─┐
│ 2  │       2 │
│ 3  │       1 │
│ 1  │       1 │
└────┴─────────┘

Totals:
┌─id─┬─count()─┐
│    │       4 │
└────┴─────────┘



with rollup修饰符,用于分组统计
    dsp-bridge1.hz.163.org :) select id,event,count(*) from t_left_a group by id,event with rollup;

    ┌─id─┬─event──────────┬─count()─┐
    │ 3  │ event_left_3   │       1 │
    │ 1  │ event_left_1   │       1 │
    │ 2  │ event_left_2_1 │       1 │
    │ 2  │ event_left_2_2 │       1 │
    └────┴────────────────┴─────────┘
    ┌─id─┬─event─┬─count()─┐
    │ 1  │       │       1 │
    │ 3  │       │       1 │
    │ 2  │       │       2 │
    └────┴───────┴─────────┘
    ┌─id─┬─event─┬─count()─┐
    │    │       │       4 │
    └────┴───────┴─────────┘

    类似
    select id,event,count(*) from t_left_a group by id,event
    union all
    select id,null,count(*) from t_left_a group by id,null
    union all
    select null,null,count(*) from t_left_a group by null,event

with cube修饰符,用于分组统计
    cube中文是立方体,用户聚合统计时,每个维度之间自由组合计算的聚合值
    例如当需要对维度(A,B,C)计算CUBE的聚合,则会生成如下维度的组合(A,B,C),(A,B),(A,C),(B,C),(A),(B),(C)和全表聚合

    dsp-bridge1.hz.163.org :) select id,event,count(*) from t_left_a group by id,event with cube;

    SELECT
        id,
        event,
        count(*)
    FROM t_left_a
    GROUP BY
        id,
        event
        WITH CUBE

    ┌─id─┬─event──────────┬─count()─┐
    │ 3  │ event_left_3   │       1 │
    │ 1  │ event_left_1   │       1 │
    │ 2  │ event_left_2_1 │       1 │
    │ 2  │ event_left_2_2 │       1 │
    └────┴────────────────┴─────────┘
    ┌─id─┬─event─┬─count()─┐
    │ 1  │       │       1 │
    │ 3  │       │       1 │
    │ 2  │       │       2 │
    └────┴───────┴─────────┘
    ┌─id─┬─event──────────┬─count()─┐
    │    │ event_left_2_1 │       1 │
    │    │ event_left_3   │       1 │
    │    │ event_left_1   │       1 │
    │    │ event_left_2_2 │       1 │
    └────┴────────────────┴─────────┘
    ┌─id─┬─event─┬─count()─┐
    │    │       │       4 │
    └────┴───────┴─────────┘


```

### in

```java
in 操作符

in 操作符: 包括 in,not in,global in, not global in;
in 操作符左边是单个列或多个列组成的元组。
in 操作符的右边可以是常量表达式,常量表达式组成的元组,数据库表名和子查询。

select user_id in (123,456) from ...
select (count_id,user_id) in ((34,123),(101500,456)) from ...
select user_id in default.user_info from ...
select (count_id,user_id) in (select count_id,user_id from ...) from ...

使用in操作的注意事项:

1> 如果in操作符右侧的数据集很大(数百万) 将其放在临时表中,通过子查询放在操作符右侧。
2> 如果操作符右侧是表名,则它等同于选择该表所有列的子查询。例如:"user_id in users" 等同于
               user_id in (select * from users);
3> in操作符左右两列的类型应该相同
4> in操作符和子查询可以出现在查询的任何部分,包括聚合函数和lambda函数
5> null不参与in运算, 所有与null运算的结果返回false，不管null在in操作符的左边还是右边。

```

### join

```java

join子句用于多表关联
 clickhouse支持的join类型
 > inner join(或join)
 > left join( left outer join)
 > right join ( right outer join)
 > full join (full outer join)
 > cross join

1) 多表之间是两两join 然后将结果与第三张表join,依次类推
2) 如果查询包含where子句,clickhouse将谓词(条件)下推至表,从而提前过滤不必要的数据,提升查询性能
3) 多表之间的连接查询建议使用join on 或join using 语法
4) 多表join支持在from后将多张表使用逗号分隔, 需要配合设置参数
       allow_experimental_cross_to_join_conversion=1(默认为1),不推荐。

create table t_first(id String, area_id String, score UInt8) engine=TinyLog;
create table t_second(id String, name String, age UInt8) engine=TinyLog;
create table t_third(area_id String, city String) engine=TinyLog;


insert into t_first values ('id001','025',100);
insert into t_first values ('id002','0551',100);
insert into t_first values ('id003','010',100);

insert into t_second values ('id001','xiaohe',22);
insert into t_second values ('id002','xiaojiang',23);
insert into t_second values ('id003','xiaohai',34);

insert into t_third values ('025','南京');
insert into t_third values ('0551','合肥');
insert into t_third values ('010','北京');

------------------------------------------------------
strictness 匹配逻辑
    strictness用于关联数据的匹配逻辑,clickhouse提供了三种stricness, ALL,ANY 和ASOF
    ALL
        如果右表具有多个匹配的行,clickhouse将从匹配的行创建笛卡尔积。这是SQL中的标准JOIN行为。
    ANY
        如果右表具有多个匹配的行,则仅连接找到第一个行。如果右表只有匹配行,则使用ANY和ALL关键值查询的结果是相同
    ASOF
        用于连接不完全匹配的序列。
        用于ASOF JOIN的表必须具有满足有序序列的特征列,这些数据类型包括UInt32, UInt64,Float32, Float64,Date和DateTime.
        ASOF JOIN不支持JOIN引擎表
        ASOF JOIN支持任意个(1个或以上)相等条件和有且仅有一个的不等匹配条件(最接近的匹配条件)
        (不等匹配条件(最接近的匹配条件)支持 >/>=/</<= )
-------------------------------------------------------

dsp-bridge1.hz.163.org :) create table t_right ( id String , name String, ev_time DateTime, event String) engine=TinyLog;

dsp-bridge1.hz.163.org :) create table t_left ( id String , name String, ev_time DateTime, event String) engine=TinyLog;

dsp-bridge1.hz.163.org :) select * from t_left_a;
┌─id─┬─name──────┬─────────────ev_time─┬─event──────────┐
│ 1  │ xiaojiang │ 2020-03-25 23:55:55 │ event_left_1   │
│ 2  │ xiaojiang │ 2020-03-25 23:25:55 │ event_left_2_1 │
│ 2  │ xiaojiang │ 2020-03-25 23:27:50 │ event_left_2_2 │
│ 3  │ xiaojiang │ 2020-03-25 23:26:22 │ event_left_3   │
└────┴───────────┴─────────────────────┴────────────────┘
dsp-bridge1.hz.163.org :) select * from t_right_a;
┌─id─┬─name──────┬─────────────ev_time─┬─event──────────┐
│ 1  │ xiaojiang │ 2020-03-25 23:25:20 │ event_righ_1_1 │
│ 1  │ xiaojiang │ 2020-03-25 23:25:55 │ event_righ_1_2 │
│ 2  │ xiaojiang │ 2020-03-25 23:25:55 │ event_righ_2_1 │
│ 2  │ xiaojiang │ 2020-03-25 23:26:55 │ event_righ_2_2 │
│ 2  │ xiaojiang │ 2020-03-25 23:27:55 │ event_righ_2_3 │
└────┴───────────┴─────────────────────┴────────────────┘

dsp-bridge1.hz.163.org :) select t1.id as id, t1.event as e1,t2.event e2 from t_left_a as t1 all left join t_right_a as t2 on t1.id=t2.id order by id asc;
SELECT
    t1.id AS id,
    t1.event AS e1,
    t2.event AS e2
FROM t_left_a AS t1
ALL LEFT JOIN t_right_a AS t2 ON t1.id = t2.id
ORDER BY id ASC
┌─id─┬─e1─────────────┬─e2─────────────┐
│ 1  │ event_left_1   │ event_righ_1_1 │
│ 1  │ event_left_1   │ event_righ_1_2 │
│ 2  │ event_left_2_1 │ event_righ_2_1 │
│ 2  │ event_left_2_1 │ event_righ_2_2 │
│ 2  │ event_left_2_1 │ event_righ_2_3 │
│ 2  │ event_left_2_2 │ event_righ_2_1 │
│ 2  │ event_left_2_2 │ event_righ_2_2 │
│ 2  │ event_left_2_2 │ event_righ_2_3 │
│ 3  │ event_left_3   │                │
└────┴────────────────┴────────────────┘

dsp-bridge1.hz.163.org :) select t1.id as id, t1.event as e1,t2.event e2 from t_left_a as t1 any left join t_right_a as t2 on t1.id=t2.id order by id asc;
SELECT
    t1.id AS id,
    t1.event AS e1,
    t2.event AS e2
FROM t_left_a AS t1
ANY LEFT JOIN t_right_a AS t2 ON t1.id = t2.id
ORDER BY id ASC

┌─id─┬─e1─────────────┬─e2─────────────┐
│ 1  │ event_left_1   │ event_righ_1_1 │
│ 2  │ event_left_2_1 │ event_righ_2_1 │
│ 2  │ event_left_2_2 │ event_righ_2_1 │
│ 3  │ event_left_3   │                │
└────┴────────────────┴────────────────┘

SELECT
    t1.id AS id,
    t1.event AS e1,
    t1.ev_time,
    t2.ev_time,
    t2.event AS e2
FROM t_left_a AS t1
ASOF LEFT JOIN t_right_a AS t2 ON (t1.id = t2.id) AND (t1.name = t2.name) AND (t1.ev_time > t2.ev_time)
ORDER BY id ASC

┌─id─┬─e1─────────────┬─────────────ev_time─┬──────────t2.ev_time─┬─e2─────────────┐
│ 1  │ event_left_1   │ 2020-03-25 23:55:55 │ 2020-03-25 23:25:55 │ event_righ_1_2 │
│ 2  │ event_left_2_1 │ 2020-03-25 23:25:55 │ 0000-00-00 00:00:00 │                │
│ 2  │ event_left_2_2 │ 2020-03-25 23:27:50 │ 2020-03-25 23:26:55 │ event_righ_2_2 │
│ 3  │ event_left_3   │ 2020-03-25 23:26:22 │ 0000-00-00 00:00:00 │                │
└────┴────────────────┴─────────────────────┴─────────────────────┴────────────────┘

------------------------------------------------------------------------------------------------------------
global inner/left/right join

SELECT
    a.req_uid,
    a.murs,
    a.gender,
    a.network_status,
    a.age,
    a.ip_city,
    b.exp_uid,
    b.material_id,
    b.exposure
FROM
(
    SELECT
        req_uid,
        murs,
        flight_name,
        ad_type,
        gender,
        network_status,
        age,
        ip_city
    FROM adx_serve_table_distributed_all
    WHERE day = '2020-08-20'
) AS a
GLOBAL INNER JOIN
(
    SELECT
        req_uid,
        exp_uid,
        material_id,
        1 AS exposure
    FROM adx_exp_clk_act_table_distributed_all
    WHERE day = '2020-08-20'
) AS b ON a.req_uid = b.req_uid



```

### limit

```java
limit by子句
 limit n by expressions 子句用于为每个不同的expressions值选择前n行,limit by的键可以包含任意数量的表达式
 clickHouse支持如下两种语法:
    limit[offset_value,]n by expressions
    limit n offset offset_value by expressions
 如果指定了OFFSET,则对于属于不同表达式组合的每个数据块,clickhouse跳过该数据块的开头offset_value行数,结果最多返回n行。

 如果offset_value大于数据块中的行数,则clickhouse从该数据块返回0行。


dsp-bridge1.hz.163.org :) select * from t_left_a;
┌─id─┬─name──────┬─────────────ev_time─┬─event──────────┐
│ 1  │ xiaojiang │ 2020-03-25 23:55:55 │ event_left_1   │
│ 2  │ xiaojiang │ 2020-03-25 23:25:55 │ event_left_2_1 │
│ 2  │ xiaojiang │ 2020-03-25 23:27:50 │ event_left_2_2 │
│ 3  │ xiaojiang │ 2020-03-25 23:26:22 │ event_left_3   │
└────┴───────────┴─────────────────────┴────────────────┘


dsp-bridge1.hz.163.org :) select * from t_left_a limit 1 by id;
 // 去每个id下的前1行
SELECT *
FROM t_left_a
LIMIT 1 BY id

┌─id─┬─name──────┬─────────────ev_time─┬─event──────────┐
│ 1  │ xiaojiang │ 2020-03-25 23:55:55 │ event_left_1   │
│ 2  │ xiaojiang │ 2020-03-25 23:25:55 │ event_left_2_1 │
│ 3  │ xiaojiang │ 2020-03-25 23:26:22 │ event_left_3   │
└────┴───────────┴─────────────────────┴────────────────┘


dsp-bridge1.hz.163.org :) select * from t_left_a limit 1,1 by id;
 // 根据id 偏移一行,然后再去一行
SELECT *
FROM t_left_a
LIMIT 1, 1 BY id
┌─id─┬─name──────┬─────────────ev_time─┬─event──────────┐
│ 2  │ xiaojiang │ 2020-03-25 23:27:50 │ event_left_2_2 │
└────┴───────────┴─────────────────────┴────────────────┘

dsp-bridge1.hz.163.org :) select * from t_left_a limit 1 offset 1 by id;
SELECT *
FROM t_left_a
LIMIT 1, 1 BY id
┌─id─┬─name──────┬─────────────ev_time─┬─event──────────┐
│ 2  │ xiaojiang │ 2020-03-25 23:27:50 │ event_left_2_2 │
└────┴───────────┴─────────────────────┴────────────────┘


select ip,device_type,murs,count(*) cnt
    from adx_exposure
    group by murs,ip,device_type
    order by cnt desc
    limit 5 by murs  //取出每个用户下前五使用的IP信息
    limit 100

------------------------------------------------------------------------------------------------------------

limit 子句用于限制从查询结果返回的函数
    limit m : 从查询结果中返回前m行
    limit n,m  : 从查询结果中跳过前n行,选择m条数据
    limit m offset n : 从查询结果中跳过前n行,选择m条数据,和 limit n,m一致
```

### arrayJoin 语句

```java
SELECT
    arrayJoin([1, 2, 3] AS src) AS dst,
    'Hello',
    src

┌─dst─┬─'Hello'─┬─src─────┐
│   1 │ Hello   │ [1,2,3] │
│   2 │ Hello   │ [1,2,3] │
│   3 │ Hello   │ [1,2,3] │
└─────┴─────────┴─────────┘
```

```java
SELECT
    arrayJoin(range(1, 8) AS src) AS dst,
    'Hello',
    src
┌─dst─┬─'Hello'─┬─src─────────────┐
│   1 │ Hello   │ [1,2,3,4,5,6,7] │  range(1,8) 取值范围 1,2,3,4,5,6,7
│   2 │ Hello   │ [1,2,3,4,5,6,7] │
│   3 │ Hello   │ [1,2,3,4,5,6,7] │
│   4 │ Hello   │ [1,2,3,4,5,6,7] │
│   5 │ Hello   │ [1,2,3,4,5,6,7] │
│   6 │ Hello   │ [1,2,3,4,5,6,7] │
│   7 │ Hello   │ [1,2,3,4,5,6,7] │
└─────┴─────────┴─────────────────┘
```

### bitmat

```java
bitmapCardinality
返回一个UInt64类型的数值，表示位图对象的基数。 SELECT bitmapCardinality(bitmapBuild([1, 2, 3, 4, 5])) AS res


bitmapToArray
将位图转换为整数数组。 SELECT bitmapToArray(bitmapBuild([1, 2, 3, 4, 5])) AS res
┌─res─────────┐
│ [1,2,3,4,5] │
└─────────────┘
```

###

### 开窗函数

```java
  在clickhouse中实现ROW_NUMBER OVER 和 DENSE_RANK OVER 等同效果的查询, 它们在一些其他数据库中可应用于RANK排序

  CH中并没有直接提供对应的开窗函数, 需要利用一些特殊函数变相实现, 主要会用到下面几个数组函数, 它们分别是:
     arrayEnumerate
     arrayEnumerateDense
     arrayEnumerateUniq

```

### 函数

#### groupArray

```java
groupArray 查询结果转换为数组
```

#### arrayIntersect

```java
arrayIntersect 返回所有数组元素的交集。
```

###

## 导出数据

gaoshuoshuo@iad-kafka3:~$ clickhouse-client -h 127.0.0.1 --port 9000 -m -u ad --password dtrMePXW --query="select \* from table_a " >/home/gaoshuoshuo/7day_active.txt

```java
into outfile子句用于将查询结果输出到指定文件,文件是保存在客户端主机上。
    如果已经存在相同文件名的文件,则查询将失败
    该功能在命令行客户端和clickhouse-local中可使用
    默认的输出格式为Tab, 可以通过format子句修改输出格式

-------------------------------------------------------------------------------------------

dsp-bridge1.hz.163.org :) select * from table_distributed;
┌──────────event_date─┬─counter_id─┬─user_id─┐
│ 2018-12-11 11:12:13 │          1 │      33 │
└─────────────────────┴────────────┴─────────┘
┌──────────event_date─┬─counter_id─┬─user_id─┐
│ 2020-08-12 09:00:00 │         23 │     100 │
└─────────────────────┴────────────┴─────────┘
dsp-bridge1.hz.163.org :)
dsp-bridge1.hz.163.org :) select * from table_distributed into outfile '/home/gaoshuoshuo/tmp/table.txt' ;

gaoshuoshuo@dsp-bridge1:~/tmp$ cat table.txt
2018-12-11 11:12:13     1       33
2020-08-12 09:00:00     23      100
```

## 字典表
