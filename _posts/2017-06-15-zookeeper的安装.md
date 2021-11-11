#### zookeeper 的搭建方式

```java
单机模式 zookeeper值运行在一台服务器上，适合测试环境
伪集群模式 在一台物理机上运行多个zookeeper实例
集群模式 zookeeper运行于一个集群上,适合生成环境,这个计算机集群被称为一个集合体

zookeeper通过复制实现高可用性,只要集合体上,半数以上的机器处于可用状态，它就能够保证服务继续。
zookeeper复制策略有关:
zookeeper确保对znode树的每一个修改都会被复制到集合体中超过半数的机器上
```

#### zookeeper 的单机模式

```java
下载解压tar -zxvf zookeeper-3.4.5.tar.gz
配置文件:将conf目录下zoo_sample.cfg文件修改为zoo.cfg文件
tickTime=2000
dataDir=/usr/local/zk/data
dataLogDir=/usr/local/zk/dataLog
clientPort=2181
配置环境变量: 为了操作方便对zookeeper的操作环境进行如下配置
修改 /etc/profile
export ZOOKEEPER_HOME = /usr/data/zookeeper
export PATH=$ZOOKEEPER/bin:$PATH

'启动zookeeper的server'
zkServer.sh start
'关闭zookeeper的server'
zkServer.sh stop

```

#####

#### zookeeper 伪集群模式

```java
zookeeper不但可以在单台机器上运行单模式的zookeeper,而且还可以在单机模拟集群模式Zookeeper的运行,也就将不同节点运行在同一机器上。
我们知道伪分布模式下hadoop的操作和分布式模式下有很大的不同,但是在集群伪分布式模式下对zookeeper的操作和集群模式下没有本质区别。
显然,集群伪分布式模式为我们体验zookeeper和做一些尝试性的实验提供了很大的便利。
```

##### **伪集群注意事项**

```java
在一台机器部署了3个server，需要注意在集群伪分布式模式下我们使用每个配置文档模式一台机器,也就是说单台机器以及以上运行多个zookeeper实例。
但是必须每个配置文档的各个端口不能冲突，处理clientPort不同之处,dataDir也不同
另外还需要在dataDir所在目录中创建myid文件来指定对应zookeeper服务器实例
'clientPort': 如果一台机器上部署多个server,那么每台机器都不同的clientport，比如server1是2181,server2是2182，server3是2183
'dataDir和dataLogDir'
dataDir和dataLogDir也需要区分开,将数据文件和日志文件分开放,同时每个server的这两变量所对应路径都是不同的
'server.x和myid'
server.X这个数字就是对应data/myid中的数字。
在3个server的myid文件中分别写入0，1，2那么每个server中的zoo.cfg都配server0,server1,server2就行了。

'同一台机器上,后面连着的3个端口,3个server都不要一样,否则端口冲突'
server.0=127.0.0.1:2881:3881
server.1=127.0.0.1:2882:3882
server.2=127.0.0.1:2883:3883
'伪集群模式下的端口不可一致 '
```

##### zookeeper 的伪集群启动

```java
在集群为分布式下，我们只有一台机器。同时要运行三个zookeeper实例。
此时需要通过下面三条：命令就能运行前面所配置的zookpeer服务
zkServer.sh start zoo-a.cfg
zkServer.sh start zoo-b.cfg
zkServer.sh start zoo-c.cfg
[root@ssgao1987 bin]# ./zkServer.sh start zoo-a.cfg
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.10/bin/../conf/zoo-a.cfg
Starting zookeeper ... STARTED
[root@ssgao1987 bin]# ./zkServer.sh start zoo-b.cfg
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.10/bin/../conf/zoo-b.cfg
Starting zookeeper ... STARTED
[root@ssgao1987 bin]# ./zkServer.sh start zoo-c.cfg
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.10/bin/../conf/zoo-c.cfg
Starting zookeeper ... STARTED

在运行完第一条指令之后,可能会出现一些错误异常,原因是由于zookeeper服务的每个实例都拥有全局配置信息,他们在启动的时候会随时随地进行Leader选举操作。
此时第一个启动的zookeeper需要和另外两个zookeeper实例进行通信,但是另外,两个zookeeper实例还没有启动起来，因此就产生异常信息。这种异常可以忽略,待所有zookeeper实例都启动起来之后,相应的异常信息自然就会消失。
'查看zookeeper的运行状态:
[root@ssgao1987 bin]# ./zkServer.sh status zoo-a.cfg
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.10/bin/../conf/zoo-a.cfg
Mode: follower
[root@ssgao1987 bin]# ./zkServer.sh status zoo-b.cfg
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.10/bin/../conf/zoo-b.cfg
Mode: leader
[root@ssgao1987 bin]# ./zkServer.sh status zoo-c.cfg
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.10/bin/../conf/zoo-c.cfg
Mode: follower
```

#### zookeeper 的管理指令

```java
zookeeper支持某些特定的四字命令字母与其交互。他们大多数是查询命令,用来获取zookeeper服务的当前状态以及相关信息。
在客户端可以通过'telnet'或'nc'向zookeeper提交相应的命令
```

##### zookeeper 常见命令

```java
'conf'
输出相关服务配置的详细信息
'cons'
详细信息。包括“接受/发送”的包数量,回话id,操作延迟, 最后的操作执行等等信息
'dump'
列出未经处理的会话和临时节点
'envi'
输出关于服务 环境的详细信息(区别于conf命令)
'reqs'
列出未经处理的请求
'ruok'
测试服务是否处于正确状态。如果确实如此,那么服务返回'imok' 否则不做任何响应
'stat'
输出关于性能和连接的客户端的列表
'wchs'
列出服务器watch的详细信息
'wchc'
通过session列出服务器watch的详细信息,它的输出是一个与watch相关的会话列表
'wchp'
通过路径列出服务器watch的详细信息。它输出一个与session相关的路径
```

##### 使用 telnet 进行测试

```java
ssgao:~ aouo$ telnet 192.168.10.231 2181
Trying 192.168.10.231...
Connected to 192.168.10.231.
Escape character is '^]'.
ruok
imokConnection closed by foreign host.
```

##### 使用 telnet 进行四字命令查询

```java
ssgao:~ aouo$ telnet 192.168.10.231 2181
Trying 192.168.10.231...
Connected to 192.168.10.231.
Escape character is '^]'.
stat
Zookeeper version: 3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
Clients:
 /192.168.11.65:52204[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 0/0/0
Received: 7
Sent: 6
.....
```

##### 使用 nc 命令进行查询

```java
ssgao:~ aouo$ echo conf | nc 192.168.11.69 2181
clientPort=2181
dataDir=/root/data/zookeeper-a/version-2
dataLogDir=/root/data/log-a/version-2
tickTime=2000
maxClientCnxns=60
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=0
initLimit=10
syncLimit=5
electionAlg=3
electionPort=3888
quorumPort=2888
peerType=0

' 查询当前连接'
ssgao:~ aouo$ echo cons | nc 192.168.11.69 2181
/192.168.11.65:57615[0](queued=0,recved=1,sent=0)
/192.168.11.69:49270[1](queued=0,recved=665,sent=665,sid=0x5de2eb65450000,lop=PING,est=1502750772792,to=30000,lcxid=0x1b,lzxid=0xffffffffffffffff,lresp=1502757257596,llat=0,minlat=0,avglat=0,maxlat=14)
```
