---
layout: post
title: zookeeper基础教程
categories: zookeeper
description: zookeeper基础教程使用
keywords: zookeeper
---

<meta name="referrer" content="no-referrer"/>

### zookeeper 基本概念

```java
通知机制
客户端注册监听它关心的目录节点，当目录节点发生变化(数据改变，被删除，子目录节点增加删除)时，zookeeper会通知客户端
```

​

```java
'leader'
leader服务器是整个ZK集群工作机制的核心
'Follower'
follower服务器是ZK集群状态的跟随者
'Observer'
Observer服务器充当一个观察者的角色
zookeeper 会话(Session)
```

```java
zookeeper的核心是原子广播,这个机制保证了各个Server之间的同步。
实现这个机制的协议叫做Zab协议.
Zab协议有两种模式,分为为恢复模式(选主)和广播模式(同步)


当服务启动或者领导者崩溃后,zab进入恢复模式(在此期间zk是不能对外提供服务的)
当领导者被选举出来,且大多数server完成了和leader的状态同步以后,恢复模式就结束了。状态同步保证了leader和server具有相同的系统状态.
(当领导者有数据写入之后,zab就进入广播模式,将更新的数据广播到所有节点,进行数据的同步)
```

```java
会话
	指的是客户端和ZK服务器的连接,ZK中的回话叫session，客户端与服务器建立一个TCP长连接。
来维护一个session，客户端在启动的时候首先会与服务器建立一个TCP长连接，通过这个连接，客户端能够通过心跳检测
与服务器保持有效会话，也能向ZK服务器发送请求并获得响应。

zk会为每个client分配一个session，session数据在每台zk节点上都有备份,server每隔ticktime向client发送一个心跳
session失效有两种可能:
	client主动调用close();server与client长时间(时间大于TIMEOUT)失去联系
session失效后与session关联的数据(比如 EPHEMERAL节点,watcher)会从内存中移除
如果Server和Client只是短暂（时间小于TIMEOUT ）地失去联系，则Session不会失效，Client再次请求Zookeeper时会把上次的SessionID带上。

Server和Client失去联系有两种情况：
	Server宕机（Zookeeper集群会让Client与另一台Server建立连接）；
    网络异常。

```

#### znode 节点

```java
ZK有两种节点
1）集群中的一台机器称为一个节点
2）数据模型中的数据单元znode，分为持久节点和临时节点
zookeeper的数据模型就是一颗树，树的节点就是znode,znode可以保存信息



持久节点(PERSISTENT)
    一旦创建,除非主动调用删除操作,否则一直存储在zk上
临时节点(EPHEMERAL)
    与客户端的回话绑定,一旦客户端回话失效,这个客户端创建的所有临时节点都被移除
顺序节点(SEQUENCE)
    创建子节点时,会自动在节点名后面追加一个整型数字,上限是整形的最大值顺序增加节点编号,
	比如clientA在zk server上创建一个znode为/name,
			指定了这种类型的节点后zookeeper会创建/name0000000000,
			clientB再去创建就是创建/name0000000001,
			clientC再创建就是/name0000000002
    以后任意client来创建这个znode都会得到一个比当前zookeeper命名空间最大znode编号+1的znode，
    也就说任意一个client去创建znode都是保证得到的znode是递增的,而且是唯一的

PERSISTENT|SEQUENTIAL 顺序自动编号的znode节点
     这种znode节点会根据当前已经存在的znode节点编号自动加1,而且不会随session断开而消失
EPHEMRAL|SEQUENTIAL 临时自动编号节点
    znode节点编号会自动增加,但是会随session消失而消失

```

​

#### Stat 以及节点版本

```java
zk的版本和svn与git中的版本含义不同，zk的版本表示节点,或子节点被修改的次数

cZxid：当前节点的事务id。
ctime：节点创建时间。
mZxid：该节点最近一次被更新时的事务id。
mtime：节点最后一次被更新时的时间。
pZxid： 子节点的最后版本。
cversion = x  节点的子节点变化次数,创建一个子节点会+1
dataVersion = x 数据的版本,每次修改会+1
aclVersion = x 节点ACL变化次数
ephemeralOwner：如果这个节点是临时节点，表示创建者的会话id。如果不是临时节点，这个值是0。
dataLength：节点存放的数据长度。
numChildren：节点的子节点个数。

节点数据发生变化dataVersion都会发生变化,即使更新的数据和原有的数据一样。
```

#### 悲观所和乐观锁

```java
悲观锁又叫悲观并发锁,是数据库中一个非常严格的锁策略,具有强烈的排他性,能够避免不同事物对同一数据并发更新造成的
数据不一致性,在上一个事物没有完成之前,下一个事物不能访问相同的资源，适合数据更新竞争非现激烈的场景。

相比悲观锁乐观锁使用的场景会更多，悲观锁认为事物访问相同的数据时一定会出现相互干扰,所以简单粗暴使用排他访问的方式。
而乐观锁认为不同事物访问相同资源很少出现相互干扰的情况,因此在事务处理期间不需要进行并发控制,当然乐观锁还是锁还是会有并发的控制。
对于数据库我们通常的做法是在每个表中增加一个version版本字段,事务修改数据之前先读出数据，当然版本号也顺势读取出来，
然后把这个读取出来的版本号加入到更新语句的条件中,比如读取出来的版本号是1，我们修改数据的语句可以这样写
'update xxx表 set 字段1=xxx where id=1 and version=1'
那如果更新失败了说明以后其他事物已经修改过数据了,那系统需要抛出异常给客户端,让客户端自行处理，客户端可以选择重试
```

#### zk 中的 watcher

##### 事件监听器

```java
ZK允许用户子在指定的节点上注册一些watcher，当数据节点发生变化的时候,ZK服务器会吧这个变化的通知发送给感兴趣的客户端

客户端向zk服务器注册wathcer的同时,会将watcher对象存储在客户端的watchManager
ZK服务器触发watcher事件后,会向客户端发送通知,客户端线程从watchManager中唤起watcher执行
```

##### watcher 中的事件

```java
NodeDataChanged事件
	无论节点数据变化还是数据版本发生变化都会触发该事件。
NodeChildrenChanged事件
	新增节点或删除节点
AuthFiled
	使用错误的scheme进行权限检查,SASL权限检查失败。

客户端只能收到相关的事件通知,但是并不能获取到对应的数据节点的原始数据内容以及变更后的新数据内容；
所以,如果业务需要知道变更前的数据或者变更后的新数据,需要业务系统保存变更前的数据和调用接口获取新的数据。
'watcher设置后,一旦触发一次即会失效,如果需要一直监听,就需要在注册,一个客户端如zkclient或zk curator 默认实现再次注册'
```

#### ZK 中的 ACL 权限控制

```java
ACL是Access Control Lists的简写
	Zookeeper采用ACL策略来进行权限控制,通常有如下权限

'CREATE'
创建子节点权限
'READ'
获取节点数据和子节点列表的权限
'WRITE'
更新节点数据的权限
'DELETE'
删除子节点的权限
'ADMIN'
设置节点ACL的权限
```

##### zk 的权限管理

```java
ACL(Access Control List 访问控制列表)
>>>> Scehme:id:permission

'Schema---->zookeeper的访问策略(验证过程中使用的检验策略)'
* 基于IP白名单
* 基于digest(用户名和密码)
'授权对象(ID)'
* ip权限模式：具体的ip地址
* digest权限模式：username:Base64(SHA-1(username:password))
'权限(permission)----> crdwa'
create(c) delete(d) read(r) write(w) admin(a)
只有一个权限为单个权限
含有所有权限为完全权限
还有多个权限为复合权限
权限组合：scheme+ID+permission
```

##### zk 客户端 ACL

```java
'创建节点时使用IP策略'
create -e /acl_a ip:192.168.11.69:crwda
表示从192.168.11.69登录的客户端可以对acl_a节点进行crwda操作(所有的权限)
'创建节点时使用digest策略'
create -e /acl_b digest:ssgao:7uNr/g9Sz3uCOHuYDrgn/EDA0kE=:crwda
客户端使用，用户名密码进行访问

产生密码方式
public class PasswordGenerate {
    public static void main(String[] args) throws NoSuchAlgorithmException {
        System.out.println(DigestAuthenticationProvider.generateDigest("ssgao:ssgao1987"));
    }
}
--输出：ssgao:7uNr/g9Sz3uCOHuYDrgn/EDA0kE=
```

##### 获取 ACL 权限控制节点

```java
[zk: 192.168.11.69:2181(CONNECTED) 14] getAcl /acl_b
'world,'anyone
: cdrwa
[zk: 192.168.11.69:2181(CONNECTED) 15] getAcl /acl_a
'world,'anyone
: cdrwa
```

##### 设置节点 ACL 权限

```java
[zk: 192.168.11.69:2181(CONNECTED) 22] create /acl_3 123
Created /acl_3
[zk: 192.168.11.69:2181(CONNECTED) 23] setAcl /acl_3 ip:192.168.11.69:w
cZxid = 0xb00000009
ctime = Tue Aug 15 07:14:12 CST 2017
mZxid = 0xb00000009
mtime = Tue Aug 15 07:14:12 CST 2017
pZxid = 0xb00000009
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: 192.168.11.69:2181(CONNECTED) 24] get /acl_3
Authentication is not valid : /acl_3
```

##### 添加认证信息

```java
[zk: 192.168.11.69:2181(CONNECTED) 27] addauth digest ssgao:7uNr/g9Sz3uCOHuYDrgn/EDA0kE=:crwda
[zk: 192.168.11.69:2181(CONNECTED) 28] get /acl_b
digest:ssgao:7uNr/g9Sz3uCOHuYDrgn/EDA0kE=:crwda
cZxid = 0xb00000006
ctime = Tue Aug 15 07:08:50 CST 2017
mZxid = 0xb00000006
mtime = Tue Aug 15 07:08:50 CST 2017
pZxid = 0xb00000006
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x5de2eb65450000
dataLength = 46
numChildren = 0

```

#### zookeeper 脑裂问题

[_https://www.cnblogs.com/kevingrace/p/12433503.html_](https://www.cnblogs.com/kevingrace/p/12433503.html)

### zk 常用的配置

#### zookeeper 基本配置

```java
zookeeper的配置

zookeeper的功能特性是通过zookeeper配置文件来进行控制管理的<zoo.cfg>

zookeeper基本配置

下面是在最低配置要求中必须配置的参数
client 监听客户端连接的端口
'tickTime
    基本事件单元,这个时间是作为zookeepe服务器之间客户端与服务器之间维持心跳时间间隔，每隔tickTime时间就会发送一个心跳,最小session过期时间为2倍tickTime
'clientPort'
	clientPort这个端口就是客户端连接zookeeper服务器的端口,zookeeper会监听这个端口，接受客户端的访问请求
'dataDir'
    存储内存中数据快照的位置,如果不设置参数,更新事务的日志将存储到默认位置(默认情况下zookeeper将写数据的日志文件也保存这个目录里)
'应该谨慎的选择日志的位置,使用专用的日志存储设备能够大大提高系统的性能'
'如果将日志存储在比较繁忙的存储设备上,那么将很大程度上响应系统性能'
```

#### zookeeper 高级配置

```java
zookeeper高级配置

高级配置参数中可选配置
'dataLogDir'
    这个操作让管理机器吧事务日志写入'dataLogDir'所指定的目录中,而不是dataDir所指定的目录。
    这将允许使用一个专用的日志设备,帮组我们避免日志和快照的竞争。
    配置如下:
        dataLogDir=/usr/local/zk/datalog
'maxClientCnxns'
    这个操作将限制连接到zookeeper的客户端数量,并限制并发连接的数量,通过IP来区分不同的客户端。
    该配置可以阻止某些类别的Dos攻击。
    将它设置为零或忽略不进行设置将会取消对并发连接的限制
    例如:我们将maxClientCnxns的值设置为1,如下所示
        maxClientCnxns=1
   启动zookeeper后,首先用一个客户端连接到zookeeper服务器上
                  然后第二个客户端尝试对zookeeper进行连接或者有某些隐式的对客户端的连接操作
                  将会触发zookeeper的上述配置
'minSessionTimeout和maxSessionTimeout'
    最小回话超时和最大回话超时
    默认minSessionTimeout=2*tickTime;maxSession=20*tickTime
zookeeper集群配置
'initLimit'
该配置表示,允许follower连接并同步到Leader的初始化连接时间,以tickTime为单位
当初始化连接时间超过该值，则表示连接失败
'syncLimit'
该配置项表示Leader和Follower之间发送消息时,请求和应答时间长度。
如果follower在设置时间内不能与Leader通信,那么follower将会被丢弃。
'server.id=host:port:port解析'
每一行此配置表示一个集群中的一台服务器,其中id为ServerID,用来标识该机器在集群中的编号。同时,在所有服务器的数据目录(/tmp/zookeeper)下创建一个myid文件,
该文件只有一行内容,并且是一个数字,就是对应每台服务的Server ID数字

比如 server.1=IP1:2888:3888的myid的内容就是1. 不同服务器的ID需要保持不同,并且和zoo.cfg文件中的server.id中的保持一致
第一个port是集群中其他机器与Leader之间通信的端口
第二个port为当Leader宕机或其他故障时，集群进行重新选举Leader时使用的端口。
每个集群的zoo.cfg文件都是相同的，可通过版本控制或其他工具保证每台zookeeper服务器的配置文件相同。集群中每台机器唯一不同的是server.id对应的myid文件中的数字不同。
```

#### 建立与 zookeeper 的连接

```java
./zkCli.sh -timeout 0 -r -server ip:port
timeout:当前会话的超时时间
如果服务器在timeout时间内没收到客户端的心跳包认为该客户端失效  --单位是ms
-r :表示只读
如果一个服务器和几个集群中过半机器失去联系以后，该服务器就不再处理客户端的请求,我们希望故障发送的时候该机器还可以提供对外读服务,即只能读不可以写
[root@ssgao1987  bin]# ./zkCli.sh -timeout 5000  -server 192.168.10.231
.....
192.168.10.231:2181, initiating session
2017-08-14 11:09:48,439 [myid:] - INFO  [main-SendThread(192.168.10.231:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server 192.168.10.231/192.168.10.231:2181, sessionid = 0x5dde84cd6e0001, negotiated timeout = 5000
WATCHER::
```

```java
WatchedEvent state:SyncConnected type:None path:null
zookeepler中命令
[zk: 192.168.10.231(CONNECTED) 2] help
ZooKeeper -server host:port cmd args
stat path [watch]
set path data [version]
ls path [watch]
delquota [-n|-b] path
ls2 path [watch]
setAcl path acl
setquota -n|-b val path
history
redo cmdno
printwatches on|off
delete path [version]
sync path
listquota path
rmr path
get path [watch]
create [-s] [-e] path data acl
addauth scheme auth
quit
getAcl path
close
connect host:port
```

```java
列出某一节点下所有子节点信息
	ls path [watch]
        path 所要列出节点的完整路径
        [zk: 192.168.10.231(CONNECTED) 3] ls /   --根目录下所有子节点
        [zookeeper, node_3]


查询某一个节点的状态
    stat path [watch]
        [zk: 192.168.10.231(CONNECTED) 4] stat /
        cZxid = 0x0  '节点被创建时的事务ID'
        ctime = Thu Jan 01 08:00:00 CST 1970 '节点被创建时的时间'
        mZxid = 0x0 '节点最后一次修改的事务ID'
        mtime = Thu Jan 01 08:00:00 CST 1970 '节点最后一次修改的时间'
        pZxid = 0x100000004 '该节点的子节点列表最后一次被修改的事务ID,为当前节点添加子节点,或从当前节点删除子节							点，修改子节点的数据内容不计算在内'
        cversion = 0
        dataVersion = 0
        aclVersion = 0
        ephemeralOwner = 0x0 '临时节点的事务ID，如果当前节点是持久的节点该值默认为0'
        dataLength = 0 '当前节点数据长度'
        numChildren = 2 '当前节点子节点个数'
        zookeeper中每次对数据节点的操作，认为是一次事务，每一个事务，系统分配一个事务ID


获取节点数据内容
    get path [watch]
        [zk: 192.168.10.231(CONNECTED) 5] get /node_3
        123 -数据信息
        cZxid = 0x100000004
        ctime = Sun Aug 13 04:20:14 CST 2017
        mZxid = 0x100000004
        mtime = Sun Aug 13 04:20:14 CST 2017
        pZxid = 0x100000004
        cversion = 0
        dataVersion = 0
        aclVersion = 0
        ephemeralOwner = 0x0
        dataLength = 3
        numChildren = 0
列出当前节点的子节点
    ls2 path [watch]
        [zk: 192.168.10.231(CONNECTED) 7] ls2 /
        [zookeeper, node_3]  --子节点
        cZxid = 0x0
        ctime = Thu Jan 01 08:00:00 CST 1970
        mZxid = 0x0
        mtime = Thu Jan 01 08:00:00 CST 1970
        pZxid = 0x100000004
        cversion = 0
        dataVersion = 0
        aclVersion = 0
        ephemeralOwner = 0x0
        dataLength = 0
        numChildren = 2
创建节点
    create [-s] [-e] path data acl
        -s 顺序节点
        -e 临时节点
        -s和-e可以同时使用
        path 完整路径
        data 节点数据
        acl 节点权限
        [zk: 192.168.10.231(CONNECTED) 8] create  /node_2 234
        Created /node_2
        [zk: 192.168.10.231(CONNECTED) 9] create /node_2/ssgao shuoailin
        Created /node_2/ssgao
        [zk: 192.168.10.231(CONNECTED) 10] create -e /node_2/aouo shuoailin  --注意-s和-e的位置
        Created /node_2/aouo
        临时节点会自动消失(启动重启后)
        [zk: 192.168.10.231(CONNECTED) 0] create -s /node_1 121
        Created /node_10000000002
        [zk: 192.168.10.231(CONNECTED) 2] create -s /node_11 121
        Created /node_110000000003
修改节点数据
    set path data [version]
    --设置节点数据的时候，要么不添加版本号，要么版本就要和设置前查询出来的版本号一致，否则会报错
        [zk: 192.168.10.231(CONNECTED) 7] set /node_3 123456
        cZxid = 0x100000004
        ctime = Sun Aug 13 04:20:14 CST 2017
        mZxid = 0x900000010
        mtime = Mon Aug 14 12:44:10 CST 2017
        pZxid = 0x100000004
        cversion = 0
        dataVersion = 2
        aclVersion = 0
        ephemeralOwner = 0x0
        dataLength = 6
        numChildren = 0
删除指定节点
    delete path [version] --只能删除没有子节点的数据
    rmr path --可以删除含有子节点的节点
        [zk: 192.168.10.231(CONNECTED) 16] delete /node_2
        Node not empty: /node_2
        [zk: 192.168.10.231(CONNECTED) 17] rmr /node_2
        [zk: 192.168.10.231(CONNECTED) 18] ls2 /
        [node_110000000003, zookeeper, node_1110000000004, node_3, node_10000000002]
        cZxid = 0x0
        ctime = Thu Jan 01 08:00:00 CST 1970
        mZxid = 0x0
        mtime = Thu Jan 01 08:00:00 CST 1970
        pZxid = 0x900000014
        cversion = 5
        dataVersion = 0
        aclVersion = 0
        ephemeralOwner = 0x0
        dataLength = 0
        numChildren = 5
```

```java
设置节点数据长度和字节点个数
setquota -n|-b val path
	-n 表示子节点个数
	-b 数据值得长度
    [zk: 192.168.10.231(CONNECTED) 38] setquota -n 2 /aouo_1
    Comment: the parts are option -n val 2 path /aouo_1
    [zk: 192.168.10.231(CONNECTED) 39] create /aouo_1/a a
    Created /aouo_1/a
    [zk: 192.168.10.231(CONNECTED) 40] create /aouo_1/b a
    Created /aouo_1/b
    [zk: 192.168.10.231(CONNECTED) 41] create /aouo_1/c a
    Created /aouo_1/c
    查看zookeeper的 日志输出信息
    2017-08-14 13:33:11,498 [myid:0] - WARN  [CommitProcessor:0:DataTree@301] - Quota exceeded: /aouo_1 count=4 limit=2

查看节点限制和当前状况
	listquota path

    [zk: 192.168.10.231(CONNECTED) 44] listquota /aouo_1
    absolute path is /zookeeper/quota/aouo_1/zookeeper_limits
    Output quota for /aouo_1 count=2,bytes=-1  当前节点限制节点 个数为2,节点数据值长度没有限制
    Output stat for /aouo_1 count=4,bytes=6    当前节点节点个数为4(包含当前节点)，数据长度值12(节点本身长度和子节点数据长度和)

删除节点限制
	delquota [-n|-b] /aouo_1

	[zk: 192.168.10.231(CONNECTED) 47] delquota -n  /aouo_1
    [zk: 192.168.10.231(CONNECTED) 48] listquota /aouo_1
    absolute path is /zookeeper/quota/aouo_1/zookeeper_limits
    Output quota for /aouo_1 count=-1,bytes=-1
    Output stat for /aouo_1 count=4,bytes=6
```
