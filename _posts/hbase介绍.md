### lsm tree 的介绍

```java
LSM-tree 最大的特点是同时使用了两部分类树的数据结构来存储数据,并同时提供查询。

其中一部分数据结构存在于内存缓存(通常叫做memtable)中,负责接受新的数据插入更新以及读请求,并且直接在内存中对数据进行排序。 另一部分数据结构存在于磁盘(sstable)上, 它们是由存在于内存缓存中的数据冲写到磁盘而成的，主要负责提供读操作,特点是有序且不可被更好。


LSM-tree的另一大特点是处理使用两部分类树的数据结构外,还会使用日志文件(通常叫做commit log)来为数据恢复做保障。这三类数据结构的协作顺序一般: 所有的新插入与更新操作都会先被记录到commit log中,该操作叫做"WAL" (Write Ahead Log) ,然后在写到memtable,最后当达到一定条件时数据会从memtable冲写到sstable,并抛弃相关的log数据, memtable和sstable可同时提供查询。当memtable出问题时,可从commit log与sstable 中将memtable的数据恢复。


参考Hbase的结构来体会架构中基于LSM-tree的部分特点。按照WAL的原则,数据首先会写到Hbase的Hlog(相当于 commit log)里,然后再写到Memstore(相当于memtable)里。最后会冲写到磁盘StoreFile(相当于sstable)中。这样Hbase的HRegionServer就通过LSM-Tree实现了数据文件的生成。

LSM-tree的这种结构非常有利于数据的快速写入(理论上可以接近磁盘顺序写速度), 但是不利于读——因为理论上读的时候可能需要同时从memtable和所有硬盘上的sstable中查询数据,这样显然对性能造成比较大的影响。为解决这个问题,LSM-Tree的采取下面的措施:

  1) 定期将硬盘上小的sstable合并(通常叫做Merge或Compaction操作),成大的sstable,以减少sstable的数量。而且,平时的数据更新删除操作并不会更新原有的数据文件,只会将更新删除操作加到当前的数据文件末端,只有在sstable合并的时候才会真正将重复的操作或更新,去重,合并。

  2) 对每个sstable使用布隆过滤器(Bloom Filter),以加速对数据在该sstable的存在性进行判定,从而减少数据的总查询时间。




```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610237061246-9ccdfc82-fdfa-401b-bc93-c61f93b9aa59.png#height=433&id=mmQDU&margin=%5Bobject%20Object%5D&name=image.png&originHeight=463&originWidth=671&originalType=binary&ratio=1&size=43560&status=done&style=none&width=628)

### 物理存储

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1617249002726-ba0e7506-25f8-4252-8aa5-45c022b1b4e5.png#height=271&id=SiaxQ&margin=%5Bobject%20Object%5D&name=image.png&originHeight=406&originWidth=1108&originalType=binary&ratio=1&size=62867&status=done&style=none&width=739)

```java
每一张表从行键(RowKey)的方向上进行切分,切分成1个到多个HRegin- 也就意味着每一张表是由1到多个HRegion构成的。
每一个HRegion保存当前HRegion的startRowKey 和 endRowKey。
每一个HRegion分布到某一个HRegionServer节点上---进行切分的目的。
	1) 将数据分布在不同节点上以能够存储海量数据
    2) 提高读写效率-提高Hbase的吞吐量
    3) 负载均衡

由于行键是默认按照字典排序且每一个HRegion都记录了起始行键和结束行键,所以在添加数据的时候可以锁定一个唯一的HRegion

当HRegion的大小默认达到10G之后,HRegion会进行分裂,平均的分裂成2个HRegion,其中一个HRegion会挪到其他节点存储。
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1617946927538-80e4f704-328b-4d10-bd4b-5b65bedcda60.png#height=371&id=HAxZW&margin=%5Bobject%20Object%5D&name=image.png&originHeight=446&originWidth=913&originalType=binary&ratio=1&size=52132&status=done&style=none&width=760)

```java
Zookeeper
	HBase 通过zookeeper来做master的高可用(通过zookeeper来保证集群中只有1个master在运行,如果master异常,会通过竞争机制产生新的master提供服务)。RegionServer的监控(通过zookeeper来监控RegionServer的状态, 当RegionServer有异常的时候,通过回调的形式通知Master RegionServer上下线的信息)，元数据的入口以及集群配置等维护工作。
    (DML的请求通过ZK分发到HRegionServer不通过HMaster, HMaster是处理DDL的请求。HMaster宕机不会影响客户端的读写请求; 但是无法进行"create 'stu4','info'"的DDL操作。当原有meta元数据信息改变的时候也无法维护)
    DML: 数据的curd,比如select,insert,update,delete
    DDL: 表的操作,比如create,alter

Hmaster
	监控RegionServer, 为RegionServer分配Region (维护整个集群的负载均衡, 在空闲时间进行数据的负载均衡, 维护集群的元数据信息, 处理Region的分配或转移) (发现失效的Region, 并将失效的Region分配到正常的RegionServer上; RegionServer失效的时候,协调对应Hlog的拆分。)

HregionServer
	HregionServer直接对接用户的读写请求,是真正的"干活"的节点。
    它的功能概括如下:
		>> 管理master为其分配的Region,处理来自客户端的读写请求
        >> 负责和底层HDFS的交互(存储数据到HDFS)
        >> 负责Region变大以后的拆分
        >> 负责StoreFile的合并工作,刷新缓存到HDFS,维护Hlog。
```

```java
Write-Ahead Logs
	hbase的修改记录, 当对hbase读写数据的时候,数据不是直接写进磁盘,它会在内存中保留一段时间(时间和数据量阈值可以设定)。但把数据保存在内存中可能有更高的概率引起数据丢失,为了解决这个问题,数据会先写在一个write-Ahead LogFile的文件中然后再写入内存中。
 	所以在系统出现故障的时候,数据可以通过这个日志文件重建。

Region
	Hbase表的分片, Hbase表会根据rowkey值被分成不同Region存储在RegionServer中
    在一个RegionServer中可以有多个不同的Region

Store
	HFile存储在store中,一个store对应hbase表中一个列族

MemStore
	内存存储,位于内存中,用来保存当前数据操作, 所以数据保存在WAL中之后,RegionServer会在内存存储键值对。

HFile
	在磁盘上保存原始数据的实际物理文件,是实际存储文件
    StoreFile是以HFile的形式存储在HDFS的。
```

### meta 表

```java
有了Region标识符,就可以唯一标识每个Region。 为了定位每个Region所在的位置,可以构建一张映射表

映射表的每个条目包含两项内容:
	1) Region标识符
    2) Region服务器标识
这个条目就表示 Region 和 Region 服务器之间的对应关系，从而就可以使用户知道某个 Region 存储在哪个 Region 服务器中。这个映射表包含了关于Region的元数据,因此也被称为"元数据表",又名"meta表"。

meta表中的每一行记录了一个Region的信息。RowKey=[tablename],[startrow],[timestamp].[regionid]
第一个region的起始行键为空,时间戳之间用'.'隔开的为分区名称的编码字符串, 该信息是由前面的表名、起始行键和时间戳进行字符串编码后形成的。

meta表里有一个列族info.info包含了如下列。
    info:regioninfo --->> 记录了Region的详细信息,包括行键范围startKey和endKey,列族列表和属性
    info:server --->> server记录了管理该Region的Region服务器的地址.如，localhost:16020
    info:serverstartcode --->> 记录了Region服务器开始托管该Region的时间。
    info:seqnumDuringOpen
    info:sn
    info:state

当用户表特别大时,用户表的Region也会非常多。Meta表存储了这些Region信息,也会变得非常大。
Meta表存储了这些Region信息,也会变得非常大。Meta表也需要划分成多个Region, 每个Meta分区记录一部分用户表和分区管理的情况。
```

```java
2.2.7 :004 >   scan 'hbase:meta'
ROW                                                       	COLUMN+CELL
 ssgao-demo                                               	column=table:state, timestamp=1617867197038, value=\x08\x00
 ssgao-demo,,1617866771919.ffda5947a0013e4ee3e13acdd0c2f6 column=info:regioninfo, timestamp=1617952113961, value={ENCODED => ffda5947a0013e4ee3e13acdd0c2f688, NAME => 'ssgao-demo,,1617866771919.ffda5947a0013e4ee3e13acdd0c2f6
 88.                                                      88.', STARTKEY => '', ENDKEY => '10'}
 ssgao-demo,,1617866771919.ffda5947a0013e4ee3e13acdd0c2f6 column=info:seqnumDuringOpen, timestamp=1617934372215, value=\x00\x00\x00\x00\x00\x00\x00\x05
 88.
 ssgao-demo,,1617866771919.ffda5947a0013e4ee3e13acdd0c2f6 column=info:server, timestamp=1617934372215, value=localhost:16020
 88.
 ssgao-demo,,1617866771919.ffda5947a0013e4ee3e13acdd0c2f6 column=info:serverstartcode, timestamp=1617934372215, value=1617934278307
 88.
 ssgao-demo,,1617866771919.ffda5947a0013e4ee3e13acdd0c2f6 column=info:sn, timestamp=1617952113961, value=localhost,16020,1617952101954
 88.
 ssgao-demo,,1617866771919.ffda5947a0013e4ee3e13acdd0c2f6 column=info:state, timestamp=1617952113961, value=OPENING
 88.
 ssgao-demo,10,1617866771919.b7c8ce3ce9d2cbd85c48e5cf9b75 column=info:regioninfo, timestamp=1617952114110, value={ENCODED => b7c8ce3ce9d2cbd85c48e5cf9b751040, NAME => 'ssgao-demo,10,1617866771919.b7c8ce3ce9d2cbd85c48e5cf9b75
 1040.                                                    1040.', STARTKEY => '10', ENDKEY => '20'}
 ssgao-demo,10,1617866771919.b7c8ce3ce9d2cbd85c48e5cf9b75 column=info:seqnumDuringOpen, timestamp=1617934372215, value=\x00\x00\x00\x00\x00\x00\x00\x02
 1040.
 ssgao-demo,10,1617866771919.b7c8ce3ce9d2cbd85c48e5cf9b75 column=info:server, timestamp=1617934372215, value=localhost:16020
 1040.
 ssgao-demo,10,1617866771919.b7c8ce3ce9d2cbd85c48e5cf9b75 column=info:serverstartcode, timestamp=1617934372215, value=1617934278307
 1040.
 ssgao-demo,10,1617866771919.b7c8ce3ce9d2cbd85c48e5cf9b75 column=info:sn, timestamp=1617952114110, value=localhost,16020,1617952101954
 1040.
 ssgao-demo,10,1617866771919.b7c8ce3ce9d2cbd85c48e5cf9b75 column=info:state, timestamp=1617952114110, value=OPENING
 1040.
 ssgao-demo,20,1617866771919.5fa40813448c6feea442deac220b column=info:regioninfo, timestamp=1617952114110, value={ENCODED => 5fa40813448c6feea442deac220b296e, NAME => 'ssgao-demo,20,1617866771919.5fa40813448c6feea442deac220b
 296e.                                                    296e.', STARTKEY => '20', ENDKEY => '30'}
 ssgao-demo,20,1617866771919.5fa40813448c6feea442deac220b column=info:seqnumDuringOpen, timestamp=1617934612512, value=\x00\x00\x00\x00\x00\x00\x00\x05
 296e.
 ssgao-demo,20,1617866771919.5fa40813448c6feea442deac220b column=info:server, timestamp=1617934612512, value=localhost:16020
 296e.
 ssgao-demo,20,1617866771919.5fa40813448c6feea442deac220b column=info:serverstartcode, timestamp=1617934612512, value=1617934278307
 296e.
 ssgao-demo,20,1617866771919.5fa40813448c6feea442deac220b column=info:sn, timestamp=1617952114110, value=localhost,16020,1617952101954
 296e.
 ssgao-demo,20,1617866771919.5fa40813448c6feea442deac220b column=info:state, timestamp=1617952114110, value=OPENING
 296e.
```

```java
meta是一种思想概念, 一种抽象思维, 用来描述数据的数据, 比如有一张学生表,记录着学生的基本信息, 我们通过表可以获取学生信息(数据), 但是有时候也要得到表本身的信息数据(比如表结构信息, 字段信息, 字段数据类型, 长度等信息)。对于这种基础信息的描述,就会使用META的概念, 使用meta元数据来描述表本身。

在HTML中也是一样的,HTML用来描述网页信息,但是HTML自己也有一些信息(比如网页标题,网页描述,搜索关键字),这些信息也就是HTML META信息, 并且HTML也定义了专门的META标记。
```

### hbase 读写流程

#### hbase 读数据

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1617183125322-380e47c6-b4f0-491a-8178-3abb6d3c7115.png#height=578&id=tJ4Ns&margin=%5Bobject%20Object%5D&name=image.png&originHeight=620&originWidth=671&originalType=binary&ratio=1&size=72524&status=done&style=none&width=626)

```java
读的时候会同时读取memStore和HFile, 然后做merge
```

**​**

#### hbase 写数据

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1617183176601-f76c8a54-d33c-4247-9b80-97b7f0a6c802.png#height=353&id=xHvxT&margin=%5Bobject%20Object%5D&name=image.png&originHeight=429&originWidth=811&originalType=binary&ratio=1&size=65345&status=done&style=none&width=668)

#### hbaseflush 过程

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1618830927262-4839cfb0-7b72-4e5c-9d7b-9534585bf6ed.png#height=271&id=OWlob&margin=%5Bobject%20Object%5D&name=image.png&originHeight=271&originWidth=403&originalType=binary&ratio=1&size=13808&status=done&style=none&width=403)

```java
上图中,一个Region中有两个store(两个列族),在flush的时候会往hdfs的不同datanode中写入。每一个列族中的列具有高的'查询同时性',不同列族中的列再一次查询中不同时查询,所以可以存放在hdfs的不同DataNode节点上。

1) 当MemStore数据达到阈值(默认是128M,老版本是64M),将数据刷到硬盘, 将内存的数据删除,同时删除HLog中的历史数据。
2) 并将数据存储到HDFS中;
3) 在HLog中做标记点


使用hbase的命令【flush + 表名】来强制memstore中的内存数据强制写入到磁盘中(hdfs)。 --flush '表名'
hdfs中的文件可以使用命令 hbase hfile -p -f 文件名的方式来查看文件内容。

```

##### flush 的触发条件

```xml
从容易的大小来看, 从小到大可以分为以下3个级别

memstore 级别限制
   ==>> 当region中的任意一个Memstore达到上限(hbase.hregion.memestore.flush.size 默认为128M), 就会触发memstore刷新
   ==>> 可以通过更改配置文件来更改默认大小
   <property>
     <name>hbase.hregion.memstore.flush.size</name>
     <value>134217728</value>
	 </property>


region级别限制
	==>> 当Region中所有memstore的大小总和达到了上限
				(hbase.hregion.memstore.block.multiplier*hbase.hregion.memstrore.flush.size 默认 2*128M=256M)
			 会触发memstore刷新
  ==>> 可以通过更改配置文件来更改默认大小



```

### hbase 数据模型

#### rowkey

```java
访问Hbase table中的行, 有三种方式:
 1) 通过单个rowKey访问
 2) 通过rowKey的range(正则)
 3) 全表扫描

 rowkey行键可以是任意字符串(最大长度是64kb, 实际应用中长度一般为10~100bytes)。
 在Hbase内部,rowkey保存为字节数组,存储时数据按照rowkey的字典顺序排序存储
```

#### column family 列族 && qualifier(列)

```java
column family
	列族: hbase表中的每个列,都归属于某个列族

    列族是表的schema的一部分(而列不是) 必须在使用表之前定义
    列表都是以列族为前缀。例如 course:history, course:math都属于courses这个列族。

 >> 列名以列族作为前缀, 每个"列族"都可以有多个列成员(column); 如course:math, course:english
    新的列族成员(列) 可以按需,动态加入。
 >> 权限控制, 存储以及调优都在列族层面进行的。

 >> Hbase把同一列族里面的数据存储在同一目录下,由几个文件保存。
 		一个列族是1~N个文件, 不是文件夹
```

#### cell 单元格

```java
>> 单元格是有版本的。
>> 单元格的内容是未解析的字节数组
	由{rowkey, column (=<family:列族>+<column:列名>), version}唯一确定的单元。
	cell中的数据是没有类型的, 全部是字节码(字节数据)形式存储。

关键字: 无类型,字节码
```

#### timestamp

```java
>> Hbase中通过rowkey 和columns确定为一个存储单位为cell。 每个cell都保存着同一份数据的多个版本。 根据唯一的时间戳来区分每个版本之间的差异, 不同版本的数据按照时间倒序排序, 最新的数据版本排在最前面。

>> 时间戳的类型是64位整型。时间戳可以由HBASE(在数据写入时自动)赋值, 此时时间戳是精确到毫秒的当前系统时间。
>> 时间戳也可以由客户显示赋值。如果应用程序要避免数据版本冲突, 就必须自己生成具有唯一性的时间戳。

每个cell中, 不同版本的数据按照时间倒序排序,即最新的数据排在最前面。

```

```java
每个cell都保存着同一份数据的多个版本。版本通过时间戳来索引, 时间戳的类型是64位整型。时间戳可以由HBASE(在数据写入时自动)赋值,此时时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显示赋值。如果应用程序要避免数据版本冲突,就必须自己生成具有唯一性的时间戳。每个cell中,不同版本的数据按照时间倒序排序, 即最新的数据排在最前面。
为了避免数据存在过多版本造成的管理(包括存贮和索引)负担, HBASE提供了两种数据版本回收方式。
	一是保存数据的最后n个版本
    二是保存最近一段时间内的版本, 比如最近7天。

用户可以针对每个列族进行设置。在HBase中,timestamp是一个很重要的概念,它记录着往HBase进行增删改操作的时间(系统自动赋值),它的值越大,说明这个操作就越新, 通常我们从HBase得到只是那个最新的操作结果, 但之前的操作(时间戳小的)会保留知道达到一定的版本数或者设定时间。
```

### HFile

```java
HFile HBase中keyValue数据的存储形式, HFile是hadoop的二进制文件, 实际上StoreFile就是对HFile做了轻量级保障, storeFile的底层是Hfile。

HLog File
	Hbase中WAL(Write Ahead Log)的存储格式, 物理上是Hadoop的Sequence File
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1617170820377-6b99a073-62e3-46bf-95f8-d6bb72b75d83.png#height=479&id=qcldk&margin=%5Bobject%20Object%5D&name=image.png&originHeight=617&originWidth=909&originalType=binary&ratio=1&size=80547&status=done&style=none&width=706)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1617158060222-389249f8-d298-49ce-bdc2-10b15637b4fa.png#height=168&id=fP9Mh&margin=%5Bobject%20Object%5D&name=image.png&originHeight=168&originWidth=608&originalType=binary&ratio=1&size=12039&status=done&style=none&width=608)

```java
HFile结构解读

Data 数据块, 保存表中的数据(可被压缩)
Meta(Optional) 元数据块, 保存用户自定义的kv对(可被压缩)
FileInfo  HFile的元数据,用户也可以在这一部分添加自己的元信息。

DataIndex 存储Dats块索引信息的块文件,每条索引的key是被索引的block的第一条记录的key
MetaIndex 存储Meta块索引信息的块文件
Trailer 它存储了FileInfo, DataIndex, MetaIndex块的偏移值和寻址信息。

Trailrer中有指针指向其他数据块的起始点。
```

```java
HFile 会被切分为多个大小相等的block块, 每个block块的大小在创建表列簇的时候通过参数 (blocksize=>65535)进行指定, 默认为64K
大号的Block有利于顺序scan, 小号的block有利于随机查询, 因而需要权衡。而且所有block块都拥有相同的数据结构。

Hbase将block块抽象为一个统一的HFileBlock。
	HFileBlock支持两种类型,一种类型支持checksum, 一种不支持。
```

#### _HFileBlock_

```java
	HFileBlock主要包括两部分, BlockHeader和BlockData。 其中BlockHeader主要存储block元数据, BlockData用来存储具体数据。Block元数据中最核心的字段是BlockType字段,用来标示该block块的类型, Hbase中定义了8中BlockType, 每种BlockType对应的block都存储不同的数据内容,有的存储用户数据,有的存储索引数据, 有的存储meta元数据.
	对于任意一种类型的HFileBlock,都拥有相同结构的BlockHeader, 但是BlockData结构却不相同。
```

```java
TrailerBlock  记录HFile的基本信息,各个部分的偏移值和寻址信息
Meta Block 主要记录Bloom Filter信息
Data Block 主要存储用户的KeyValue数据
Root Index DataBlock的跟索引以及MetaBlock和BloomFilter的索引
IntermediateLevelIndex DataBlock的第二层索引
LeafLevelIndex DataBlock的第三层索引, 即索引的叶子节点

Bloom MetaBlock Bloom相元数据
Bloom Block Bloom相关数据

```

##### _Trailer Block_

> _traniler 这部分主要记录了 HFile 的基本信息,各个部分的偏移值和寻址信息_

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1617155958379-cb6f6e5b-0530-43ed-8c53-a594b381df1a.png#height=105&id=Ti1bv&margin=%5Bobject%20Object%5D&name=image.png&originHeight=121&originWidth=711&originalType=binary&ratio=1&size=20513&status=done&style=none&width=615)

```java
开始是两个固定长度的数值,分别表示Key的长度和Value的长度。
紧接着是Key
	开始是固定长度的数值,表示RowKey的长度,紧接着是RowKey
    然后是固定长度的数值,表示列族的长度,紧接着是列族Column Family (列族),Column Qualifier (列)
    最后是两个固定长度的数值,表示TimeStamp 和Key Type(put/delete/deletecolumn 和 deleteFamily)。

    Value部分没有这么复杂的结构,就是纯粹的二进制数据。

```

### RowKey 的设计

```java
>>越短越好, 提高效率
 	1> 数据的持久化文件HFile是按照KeyValue存储的, 如果rowKey过长,比如操作100字节, 1000W行数据,单存储rowKey就需要1亿Byte,接近1G数据, 这样会影响HFile的存储效率。
 	2> HBase中包含缓存机制,每次会将查询的结果缓存到HBase的内存中,如果rowKey字段过长,内存的利用率就会降低, 系统不能缓存更多的数据,这样会降低检索效率。

>>散列原则,实现负载均衡
    如果rowKey是按时间戳的方式递增,不要将时间放在二进制码的前面, 建议将RowKey的高位作为散列字段,由程序循环生成,低位放时间字段,这样将提高效率均衡分布在每个RegionServer实现负载均衡的几率。
    如果没有散列字段,首字段直接是时间信息将产生所有数据都在一个RegionServer上堆积容易出现数据热点的问题。这样在做数据检索的时候负载将会集中个别RegionServer,降低查询效率。
    1> 加盐: 添加随机值
    2> hash: 采用md5散列算法取前4为做前缀
    3> 反转: 将手机号反转

>> 唯一原则,字典需排序存储
	必须在设计上保证其唯一性, rowKey是按照字段顺序排序存储的, 因此, 设计rowKey的时候,要充分利用这个排序特点, 将经常读取的数据存储到一块, 将最近可能被访问的数据放到一块。


```
