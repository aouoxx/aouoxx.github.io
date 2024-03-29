---
layout: post
title: zookeeper的原生,以及curator
categories: zookeeper
description: zookeeper的客户端使用介绍,curator
keywords: zookeeper
---

 <meta name="referrer" content="no-referrer"/>

### zookeeper 的应用场景

#### master 选举

```java
curator实现leader选举有两种当时,
第一种是LeaderLatch,这种是阻塞的。所有的client一起去竞争leader,没有选上的client会一直阻塞等待上一个leader退出或则挂掉。一旦leader位置空了,继续争夺leader
第二种是LeaderSelector监听器实现Leader选举功能,同一时刻，只有一个Listener会进入takeLeadership()方法，说明它是当前的Leader.
注意：当Listener从takeLeadership()退出时就说明它放弃了“Leader身份”， 这时Curator会利用Zookeeper再从剩余的Listener中选出一个新的Leader。
     autoRequeue()方法使放弃 Leadership的Listener有机会重新获得Leadership，如果不设置的话放弃了的Listener是不会再变成Leader的。

```

##### leaderlatch 实例

```java
// 选举Leader 启动
LeaderLatch latch = new LeaderLatch(client, "/leader");
latch.addListener(new LeaderLatchListener(){
    @Override
    public void isLeader() {
        System.out.println("I am leader");
    }
    @Override
    public void notLeader() {
        System.out.println("I not leader");
    }
});
latch.start();
// latch.await();
// System.out.println("I am leader");
System.out.println(latch.hasLeadership());
TimeUnit.SECONDS.sleep(20);
latch.close();

latch会参与竞选leader，如果被选举为leader，会调用isLeader()方法，hasLeadership()会返回true。
```

##### leaderlatch 的注意事项

```java
> 使用LeaderLatcher是,必须启动即必须要调用leaderLatch.start()方法。一旦启动LeaderLatch会和其他使用相同的path的LeaderLatch进行交涉,然后随机选一个用来作为leader。
> 查看一个给定的实例是否是leader 'public boolean hasLeadership()'
> 阻塞当前线直到获取leadership
   await()
   await(long timeout,TimeUnit unit)
> 一旦不使用LeaderLatch,必须调用close方法,如果它是leader,会释放leadership,其他的参与者将会选举一个leader
> 异常处理leaderLatch实例可以增加ConnectionStatsListener来监听网络连接。
```

##### leaderselector 示例

```java
final LeaderSelector leaderSelector = new LeaderSelector(client, "/leader",
        new LeaderSelectorListenerAdapter() {
            @Override
            public void takeLeadership(CuratorFramework client) throws Exception {
                System.out.println("I am leader, working...");
                TimeUnit.SECONDS.sleep(3);
                System.out.println("I am leader, end.");
            }
        });
leaderSelector.autoRequeue();
leaderSelector.start();
TimeUnit.SECONDS.sleep(1000);


除了LeaderSelectorListenerAdapter外，还可以使用LeaderSelectorListener，LeaderSelectorListener将会额外监控stateChanged。

LeaderSelectorListener selectorListener = new LeaderSelectorListener() {
    //此方法将会在Selector的线程池中的线程调用
    @Override
    public void takeLeadership(CuratorFramework client) throws Exception {
        System.out.println("I am leader...");
        //如果takeLeadership方法被调用,说明此selector实例已经为leader
        //此方法需要阻塞,直到selector放弃leader角色
    }
    //这个方法将会在Zookeeper主线程中调用---watcher响应时
    @Override
    public void stateChanged(CuratorFramework client, ConnectionState newState) {
        System.out.println("Connection state changed...");
        //对于LeaderSelector,底层实现为对leaderPath节点使用了"排他锁",
        //"排他锁"的本质,就是一个"临时节点"
        //如果接收到LOST,说明此selector实例已经丢失了leader信息.
        if (newState == ConnectionState.LOST) {
            //需要做特殊处理
        }
    }
};

```

##### leaderselector 的注意事项

```java
构造函数
public LeaderSelector(CuratorFramework client,String mutexPath,LeaderSelectotListener listener);
public LeaderSelector(CuratotFramework client,String mutexPath,ThreadFactory threadFactory,Executor executor,LeaderSelectorListener listener);

> leaderSelecto需要调用start进行启动。当实例取得领导权时你的listener的takeLeadership()方法被调用,而takeLeadership()方法只有领导权被释放才返回。
> 当不再使用leaderSelector实例时,应该调用它的close方法
> 异常处理 LeaderSelectorListener类继承ConnectionStateListener.LeaderSelector必须小心连接状态的改变。如果实例成为leader,它应该相应SUSPENDED或lost.
  当SUSPENDED状态出现时,实例必须假定在重新连接成功之前它可能不再是leader了。
  当LOS状态出现时,实例不再是leader，takeLeadership方法返回。
> leaderSelector.autoRequeue();保证在此实例释放领导权之后还可能获得领导权。
```

##### master 选举基本原理

```java
'实例场景'
    比如一个系统A向外提供一个服务B，服务B需要以7*24的方式向外提供服务,即提供服务的机器不能有单点故障,这种情况下我们考虑使用集群。
    集群中有一台主机,多台备机，主机向外提供服务,备机负责监听主机的状态,一旦主机down机,备机可以很迅速的接管主机,继续提供服务，这个过程
    从备机选出一台作为主机称为master选举

'zookeeper实现master选举'
   zookeeper的节点有两种类型'持久节点'和'临时节点'。
   临时节点有个特性,就是如果注册这个节点的机器失去连接(通常是down机),那么这个节点会被zookeeper删除。
   选主过程就是利用这个特性,在服务器启动的时候，去zookeeper特定的一个目录下注册一个临时节点(这个节点做为master,谁注册了这个节点就是就是master),注册的时候,如果发现这个节点已经存在,则说明别的服务器注册了(说明有别的服务器已经抢主成功),那么当前服务器只能放弃抢主,作为从机存在。

   同时抢主失败的当前服务器需要订阅该临时节点的删除事件,以便该节点删除时(也就是注册节点的服务器down机了或者网络断了之类的)进行再次抢主。
   从机具体需要去哪里注册服务器列表临时节点,节点保存了什么信息,需要根据业务不同自行约定。
   选主的过程,其实就是简单争抢在zookeeper注册临时节点的操作,谁注册了约定的临时节点,谁就是master
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635637851162-5416937e-9aa4-49ea-8349-0680053a0452.png#clientId=u1d4bf6c3-4907-4&from=paste&height=286&id=u9fa93261&margin=%5Bobject%20Object%5D&name=image.png&originHeight=572&originWidth=2336&originalType=binary&ratio=1&size=521747&status=done&style=none&taskId=u3aaa9a3e-2e0d-4147-b95c-5d18e3caff8&width=1168)

###### 应对网络抖动

```java
由于网络抖动,可能误删了master节点导致重新选举,如果master还未down机,而被其他节点抢到了,会造成可能有写数据重新生成等资源的浪费。
这里我们增加一个判断,如果上次自己不是一个master就等待5s在开始增强master，这样能保证没有down机的master能再次当选为master
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635637896375-d5491b7a-aa59-496f-91ee-1c439090044a.png#clientId=u1d4bf6c3-4907-4&from=paste&height=242&id=ubcfccc78&margin=%5Bobject%20Object%5D&name=image.png&originHeight=484&originWidth=1368&originalType=binary&ratio=1&size=191318&status=done&style=none&taskId=ua498c17e-75ce-4b48-bf88-f10100b0202&width=684)

#### 分布式锁

```java
分布式锁,这个主要得益于zookeeper为我们保证了数据的强一致性。锁服务可以分为两类,一个是保持独占,另一个控制时序。
'保持独占'
   就是所有试图来获取这个锁的客户端,最终只有一个可以成功获取这把锁。
   通常的做法就是把ZK上的znode看作是一把锁,通过create znode的方式来实现。
   所有客户端都去创建/distribute_lock节点,最终成功创建的那个客户端拥有了这把锁。

 分布式锁使用临时节点: 临时节点效率高,
 分布式锁为什么使用get: 因为get是存储在内存中,所以我们get的性能非常高

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635637401867-5d432bcc-c605-4b83-b42a-d268149af380.png#clientId=u1d4bf6c3-4907-4&from=paste&height=373&id=udc36eddb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=745&originWidth=1472&originalType=binary&ratio=1&size=110488&status=done&style=none&taskId=u71091467-16c7-409a-90d8-65ae5a0b7b3&width=736)

```java
我们通过zk来实现分布式锁。主要步骤是：
> 建立一个节点,假如名为lock。节点类型为持久节点(PERSISTENT)
> 每当进程需要访问共享资源时,会调用分布式lock()或trylock()方法获得锁,这个时候会在第一步创建的lock节点下建立相应的顺序节点
  节点类型为临时顺序节点(EPHMERAL_SEQUENTIAL),通过组成特定的名字name+lock+顺序号
> 在建立子节点后,对lock下面的所有以name开头的子节点进行排序,判断刚刚建立的子节点是否是最小的节点,假如是最小节点,则获得该锁对资源进行访问。
> 假如不是该节点,就获取该节点的上一顺序节点,并给该节点注册删除事件。同时在这里阻塞。等待监听事件的发生,获得锁控制权。
> 当调用完共享资源后,调用unlock()方法,关闭zk,进而可以引发监听事件,释放该锁。

实现分布式锁是严格按照顺序访问的并发锁。
```

##### curator 实现分布式锁

```java
public class ZkLock{
    // zookeeper地址
    static final String CONNECT_ADDR = "192.168.0.4:2181,192.168.0.9:2181,192.168.0.6:2181";
    // session超时时间
    static final int SESSION_OUTTIME=5000;
    static int count =10;
    public static void genarNo(){
        try{
            count--;
            System.out.println(count);
        }finally{
            //...
        }
    }

    public static void main(String[] args) throws Exception{
      //重试策略:初试时间为1s,重试10次
      RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000,10);
      //通过工厂创建连接
      CuratorFramework cf = CuratorFrameworkFactory.builder()
                              .connectString(CONNECT_ADDR)
                              .sessionTimeoutMs(SESSION_OUTTIME)
                              .retryPolicy(retryPolicy)
                              .build();
      //开启连接
      cf.start();
      /**
       * 分布式锁
       * 本实例演示使用InterProcessMutex做分布式锁,通过zookeeper节点不能重复的原理来让分布式环境的服务队列处理同一个数据
       */
      final InterProcessMutex lock = new InterProcessMutex(cf,"/aouo");
      final CountdownLatch countdown = new CountDownLatch(1);

     for(int i=0;i<10;i++){
         new Thread(new Runnable(){
             @Override
             public void run(){
                 try{
                     countdown.await();
                     //加锁
                     lock.acquire();
                     //------------业务开始处理
                     genarNo();
                     //------------业务处理结束
                 }catch(Exception e){
                     e.printStackTrace();
                 }finally{
                     try{
                         //释放
                         lock.release();
                     }catch(Exception e){
                         e.printStackTrace();
                     }
                 }
             }
         },"t"+i).start();
     }
     Thread.sleep(100);
     countdown.countDown();
    }
}
```

##### zookeeper 负载均衡

```java
zk的负载均衡是指软负载均衡。在分布式环境中,为了保证高可用性,
	通常同一个应用或同一个服务的提供方都会部署多份,达到对等服务。
消费者就必须在这些对等服务器中选择一个,来执行相关的业务逻辑,
	其中比较典型的是
    	消息中间件中的生产者,消费者负载均衡。
 		消息中间件中发布者和订阅者的负载均衡


```

#### 命名服务

```java
命名服务也是分布式系统中比较常见的一类场景。在分布式系统中,通常使用命名服务,客户端应用能够根据制定名字来获取资源或服务的地址,提供者等信息。
被命名的实体通常可以是集群中的机器,提供的服务地址,远程对象等等——这些我们可以统称它们为名字(Name)。
其中较为常见的就是一些分布式服务框架中的服务地址列表。
通过调用ZK提供的创建节点API，能够很容易的创建一个全局唯一的path,这个path就可以作为一个名称。

Dubbo中就是使用ZK来做命名服务,维护全局的服务地址列表。
dubbo实现如下：
> 服务提供者启动的时候,向ZK上指定的节点/dubbo/${serviceName}/providers目录下写入自己的URL地址,这个操作完成了服务的发布
> 服务消息者启动的时候,订阅/dubbo/{serviceName}/providers目录下的提供者URL地址,并向/dubbo/${serviceName}/consumers目录下写入自己的URL地址

Note:
'所有向ZK上注册的地址上都是临时节点,这样就能够保证服务提供者和消费者能够自动感应资源的变化。'

```

#### 数据发布与订阅

```java
发布和订阅模式是1对多的关系,多个订阅者对象同时监听某一主题对象,这个主题对象在自身状态发生变化时会通知所有订阅者对象。
使它们能自动的更新自己的状态。发布/订阅可以使得发布方和订阅方独立封装,独立改变。
当一个对象的改变需要同时改变其他对象,而且它不知道具体有多少对象需要改变时可以使用发布、订阅模式。
发布/订阅模式在分布式系统中典型应用有'配置管理'和'服务发现,注册'

'配管管理'
  如果集群中的机器拥有某些相同的配置,并且这些配置信息需要动态的改变,我们可以使用发布/订阅模式把配置做统一集中管理,让这些机器各自订阅配置信息的改变,当配置信息发送改变时,这些机器就可以得到通知并更新为最新的配置
'服务发现,注册'
  服务发现,注册是指对集群中的服务上下线做统一管理。每个工作服务器都可以作为数据的发布方,向集群注册自己的基本信息,而往某些监控服务器作为订阅方,订阅工作服务器的基本信息
当工作服务器基本信息发生改变如上下线,服务器角色或服务范围变更,监控服务器可以得到通知并响应这些变化。

'系统中有些信息需要动态获取,并且还存在人工手动去修改这个信息访问'
通常是暴露出接口,例如JMX接口,来获取一些运行时信息。进入ZK之后,只要将这些信息放到指定的ZK节点上即可

'数据发布和订阅使用场景'
一般用于数据量很小,但是数据更新可能比较快的场景
```

##### 配置管理

```java
所谓的配置中心,就是发布者将数据发布到ZK的一个或一系列的节点上,供订阅者进行数据订阅,进而达到动态获取数据的目的,实现配置信息的集中管理和数据的动态更新。

发布/订阅系统一般有两种设计模式,分别是推模式(push)和拉模式(pull)
> 推模式
服务端主动将数据更新发送给所有订阅的客户端
> 拉模式
客户端通过采用定时轮询拉取

zookeeper采用推拉相结合的方式:
客户端向服务端注册自己需要关注的节点,一旦该节点的数据发生变更,那么服务端就会向相应的客户端发送watcher事件通知，
客户端接收到这些消息通知后,需要主动到服务端获取最新的数据。


如果将配置信息存放到ZK上进行集中管理,那么通常情况下,应用在启动的时候会主动到ZK服务器上进行一次配置信息的获取,同时,在指定的节点上注册一个Watche监听,这样一来,
但凡配置信息发生变化,服务器都会实时通知到所订阅的客户端,从而达到实时获取最新配置信息的目的。

```

### curator 的应用

#### curator 使用

```sql
Curator是Netflix公司开源的一套zookeeper客户端框架,curator解决了很多zookeeper非常 底层的细节开发工作
包括连接重连，反复注册watcher等实现了Fluent风格的API接口 目前已经成为Apache的顶级项目是全世界范围内使用最广泛的zookeeper客户端之一.
```

#### curator 的基本使用

```java
使用的依赖

@Test
    public void createSession(){
        //使用Fluent分割
        CuratorFramework zkclient =
                CuratorFrameworkFactory
                        .builder()
                .connectString("192.168.10.104:2181")
                .canBeReadOnly(false)
                .retryPolicy(new ExponentialBackoffRetry(1000,Integer.MAX_VALUE)) //重试策略
                .sessionTimeoutMs(5000) //回话超时时间
                .connectionTimeoutMs(5000) //连接超时时间
                .build();

        zkclient.start();
    }
```

##### 非 fluent 的写法

```java
public class Person {
    private String name;
    private int age;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

##### fluent 的写法

```java
public class FPerson {
    private String name;
    private int age;
    //设置name的值,并且返回实体
    public FPerson setName(String name){
        this.name=name;
        return this;
    }
    //设置age的值,并且返回实体
    public FPerson setAge(int age){
        this.age=age;
        return this;
    }
    public String getName() {
        return name;
    }
    public int getAge() {
        return age;
    }
    //返回学生实体,可以做成单例
    public static  FPerson build(){
        return new FPerson();
    }
    @Override
    public String toString() {
        return "FPerson{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

public static void main(String[] args){
  FPerson fperson = FPerson.build().setName("ssgao").setAge(30);
  System.out.println(fperson.getName());
}
```

#### CuratorFramework 提供的方法

```java
create()
 开始创建操作,可以调用额外的方法(比如方式mode或者后台执行background),并在最后调用forPath()指定要操作的ZNode
delete()
 开始删除操作,可以调用额外的方法(版本或者后台处理version or background)并在最后调用forPath()指定要操作的Znode
checkExists()
 开始检查ZNode是否存在的操作,可以调用额外的方法(监控或者后台处理)并在最后调用forPath()指定要操作的ZNode
getData()
  开始获得ZNode节点数据的操作,可以调用额外的方法(监控,后台处理或者获取状态watch,background or get stat)并在最后调用forPath()指定要操作的ZNode
setData()
  开始设置ZNode节点的数据操作,可以调用额外的方法(版本或者后台处理)并最后调用forPath()指定要操作的ZNode
getChildren()
  开始获取ZNode()的子节点列表。以调用额外的方法(监控,后台处理或者获取状态watch,background ot get stat)并在最后调用forPath()指定要操作的ZNode
inTransaction()
  开始是原子ZooKeeper事务,可以复合create,setData,check,and/or delete 等操作然后调用commit()作为一个原子操作提交



```

####

#### CuratorFrameworkFactory

```java
CuratorFrameworkFactory有四种重连策略,均实现了RetryPolicy接口
    zkServer端timeout参数
    tickTime： zk的心跳间隔(heart beat,interval) 也是session timeout的基本单位,单位为毫秒
    minSessionTimeout: 最小超时时间,zk设置的默认值为2*tickTime
    maxSessionTimeout: 最大超时时间,zk设置的默认值为20*tickTime

客户端实际超时时间
    int minSessionTimeout = zk.getMinSessionTimeout();
    if(sessionTimeout 小于 minSessionTimeout){
          sessionTimeout = minSessionTimeout;
    }
    int maxSessionTimeout = zk.getMinSessionTimeout();
    if(sessionTimeout 大于  maxSessionTimeout ){
          sessionTimeout = maxSessionTimeout;
    }
如果sessionTimeout的设置小于minSessionTimeout，则采用服务器的minSessionTimeout配置
如果sessionTimeout的设置大于maxSessionTimeout，则采用服务器的maxSessionTimeout配置
客户端的超时设置,不能超过服务器端配置的范围

```

#### curator 的客户端超时设置

```java
CuratorFrameworkFactory.newClient
连接超时15秒
Session超时60s
    private static final int DEFAULT_SESSION_TIMEOUT_MS = Integer.getInteger("curator-default-session-timeout",60*1000);
    private static final int DEFAULT_CONNECTION_TIMEOUT_MS = Integer.getInteger("curator-default-connection-timeout",15*1000);
    public static CuratorFramework newClient(String connectionString,RetryPolicy retryPolicy){
         return newClient(connectionString,DEFAULT_SESSION_TIMEOUT_MS,DEFAULT_CONNECTION_TIMEOUT_MS,retryPolicy)
    }
```

####

#### curator 的重连策略

```java
RetryUtilElspsed(int maxElapsedTimeMs, int sleepMsBetweenRetries)
    > 以slesspMsBetweenRetries的间隔重连,直到超过maxElapsedTimeMs的时间设置
    > maxElapsedTimeMs 表示最大的重试时间
    > sleepMsBetweenRetries 两次重试之间的间隔时间

RetryNTimes(int n ,int sleepMsBetweenRetries)
    > 指定最大重连次数
    > n表示最大重试次数
    > sleepMsBetweenRetries 表示两次重试之间的间隔

RetryOneTime(int sleepMsBetweenRetry)
    > 重连一次,简单粗暴

ExponentialBackoffRetry(int baseSleepTimeMs, int maxRetries);
    > 重试指定的次数,并且在每一次重试之间,重试的时间逐渐增加
    > ExponentialBackoffRetry(int baseSleepTimeMs,int maxReties,int maxSleepMs);
    > 时间间隔 = baseSleepTimeMs*Math.max(1,random.nextInt(1 《 （retryCount+)));
```

#####

#### curator 的节点操作

##### 创建节点操作

```java
创建数据节点模式
  PERSISTENT 持久化
 PERSISENT_SEQUENTIAL持久化带序列号
 EPHEMERAL 临时
 EPHEMETAL_SEQUENTIAL 持久化并且带序列号

> 创建一个节点,初始化内容为空
  client.create().forPath("/name"); // 如果没有设置节点属性,节点创建模式默认为持久化节点,内容默认为空

> 创建一个节点,附带初始化内容
  client.create().forPath("/name","ssgao".getBytes());

> 创建一个节点,指定创建模式(临时节点),内容为空
  client.create().withMode(CreateMode.EPHEMERAL).forPath("path")

> 创建一个节点,指定创建模式(临时节点),附带初始化内容
  client.create().withMode(CreateMode.EPHEMERAL).forPath("name","ssgao".getBytes());

> 创建一个节点,指定创建模式(临时节点)，附带初始化内容,并且自动递归创建父节点
  client.create()
   .creatingParentContainersIfNeeded()
   .withNode(CreateMode.EPHEMERAL)
   .forPath("/name"，"ssgao".getBytes());
   这里creatingParentContainersIfNeeded()方法非常有用,该方法不用去判断当前创建节点的父节点是否存在,可以进行递归创建所需要的父节点。
   如果不使用该方法可能抛出NoNodeException异常
```

##### 删除数据节点

```java
> 删除一个节点
  client.delete().forPath("/name"); //只能删除叶子节点(否则抛出异常)

> 删除一个节点,并且递归删除所有子节点
  client.delete().deletingChildrenIfNeeded().forPath("path");

> 删除一个节点，强制指定版本进行删除
  client.delete().withVersion(10086).forPath("path");

> 删除一个节点，强制保证删除
  client.delete().guaranteed().forPath("path");
  guaranteed()接口是一个保障措施，只要客户端会话有效，那么Curator会在后台持续进行删除操作，直到删除节点成功。

> 上面的多个流式接口是可以自由组合的
  client.delete().guaranteed().deletingChildrenIfNeeded().withVersion(10086).forPath("path");
```

##### 读取数据节点

```java
> 读取一个节点的数据内容
  client.getData().forPath("path"); 注意，此方法返的返回值是byte[ ];

> 读取一个节点的数据内容，同时获取到该节点的stat
  Stat stat = new Stat();
  client.getData().storingStatIn(stat).forPath("path");
```

##### 更新数据节点

```java
> 更新一个节点的数据内容
 client.setData().forPath("/path","data".getBytes());  //注意：该接口会返回一个Stat实例

>更新一个节点的数据内容，强制指定版本进行更新
 client.setData().withVersion(10086).forPath("path","data".getBytes());
```

##### 检查节点是否存在

```java
> client.checkExists().forPath("path");
  注意：
  该方法返回一个Stat实例
  用于检查ZNode是否存在的操作. 可以调用额外的方法(监控或者后台处理)并在最后调用forPath( )指定要操作的ZNode
```

##### 获取节点下子节点信息

```java
> client.getChildren().forPath("path");
  注意：该方法的返回值为List<String>,获得ZNode的子节点Path列表。
  可以调用额外的方法(监控、后台处理或者获取状态watch, background or get stat) 并在最后调用forPath()指定要操作的父ZNode
```

#### curator 的异步使用

```java
zk_curate节点操作,章节主要介了创建,删除,更新,读取等方法都是同步的,Curator提供了异步接口.

引入了"BackgroundCallBack"接口用于处理异步接口调用之后服务端返回的结果信息。
BackgroundCallback接口中一个重要的回调值为CuratorEvent 里面包含事件类型,响应码和节点的详细信息。\\

public interface BackgroundCallback {
    /**
     * 参数 curatorEvent 包含书剑类型,响应吗,和节点的详细信息
     */
    void processResult(CuratorFramework var1, CuratorEvent curatorEvent) throws Exception;
}

```

```java
@Test
    public void aysnOperate() throws Exception{
        CuratorFramework zkclient =
                CuratorFrameworkFactory
                        .builder()
                        .connectString("192.168.10.104:2181")
                        .canBeReadOnly(false)
                        .retryPolicy(new ExponentialBackoffRetry(1000,Integer.MAX_VALUE))
                        .sessionTimeoutMs(5000)
                        .connectionTimeoutMs(5000)
                        .build();
        zkclient.start();
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        zkclient.checkExists()
                .inBackground(
                /**
                 * 传入回调,系统执行完成后会自动的调用回调
                 * curater的回调是实现了BackgroundCallback接口的实例
                 * curatorFramework 客户端对象
                 * curatorEvent curator的事件对象
                 *
                 * curate的异步调用会放到一个单独的线程去执行,如果同一时刻存在大量的异步调用
                 * 系统就会创建大量的线程,开销会很大,为了优化系统性能,我们使用线程池
                 * 我们创建一个线程池并把该线程池传递给异步调用.线程池使用完后注意关闭
                 */
                new BackgroundCallback() {
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        //curatorEvent.getType();获取curate的事件类型
                        //curatorEvent.getResultCode();异步操作的返回码 失败非0 成功为0
                        //curatorEvent.getContext();获取异步操作的上下文,上下文是通过参数传递进行来的,比如参数值"123"
                        //curatorEvent.getPath();触发该事件的节点路径
                        //curatorEvent.getData();获取节点的数据内容
                        //curatorEvent.getChildren();获取子节点列表
                        curatorEvent.getStat();//获取节点状态
                    }
                },"123",executorService)
                .forPath("/name");
    }

注意：如果inBackground()方法不指定executor，那么会默认使用Curator的EventThread去进行异步处理。

/**
 * 异步创建时进行回调
 */
 client.create().withMode(CreateMode.EPHEMERAL)
                .inBackground(new BackgroundCallback() {
                    @Override
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        System.out.println("节点路径:" + curatorEvent.getPath());
                       // 注意下面这句语句在执行的时候报错,以为这时还有没内容所有无法获取节点数据信息
                       // System.out.println("节点内容:" + new String(curatorEvent.getData()));
                        System.out.println("节点名称:" + curatorEvent.getName());
                    }
                })
                .forPath("/async","ssgao".getBytes());

```

##### **CuratorEventType 事件类型**

```java
事件类型 CREATE     对应curatorFramework的实例方法 create()
事件类型 DELETE     对应curatorFramework的实例方法 delete()
事件类型 EXISTS     对应curatorFramework的实例方法 checkExists()
事件类型 GET_DATA   对应curatorFramework的实例方法 getData()
事件类型 SET_DATA   对应curatorFramework的实例方法 setData()
事件类型 CHILDREN   对应curatorFramework的实例方法 getChildren()
事件类型 SYNC       对应curatorFramework的实例方法 sync(String,Object)
事件类型 GET_ACL    对应curatorFramework的实例方法 getACL()
事件类型 SET_ACL    对应curatorFramework的实例方法 setACL()

事件类型 WATCHED    对应curatorFramework的实例方法 watcher(Watcher watcher)
事件类型 CLOSING    对应curatorFramework的实例方法 close()
```

##### **CuratorEventType 返回响应码**

```java
0    Ok,调用成功
-4   ConnectionLoss 客户端与服务端断开连接
-110 NodeExist节点已经存在
-112 SessionExpired 回话过时
```

#### curator 的事件监听

```java
zookeeper原生支持通过注册watcher来进行事件监听,但是开发者需要反复注册(Watcher只能单次注册单次使用)。
Cache是Curator中对事件监听的包装,可以看成是对事件监听的本地缓存视图,能够自动为开发者处理反复注册监听。
Curator提供了三种Cache来监听结点的变化
需要依赖
<dependency>
  <groupId>org.apache.curator</groupId>
  <artifactId>curator-recipes</artifactId>
  <version>2.12.0</version>
</dependency>

cache是一种缓存机制,可以借助cache来实现监听
cache客户端缓存了znode的各种状态,当感知到zk集群的znode状态变化,会触发event事件,注册的监听器会处理这些事件。

```

##### pathcache

```java
PathCache用来监控一个ZNode的子节点
   当一个子节点增加,更新,删除时,path Cach会改变他的状态包含最新子节点,子节点的数据和状态,而状态的更变将通过PathChildrenCacheListener通知
使用时主要涉及的四个类
PathChildrenCache //节点缓冲信息
PathChildrenCacheEvent  //节点事件类型
PathChildrenCacheListener //节点监听接口对象
ChildData //节点数据信息

"构造函数创建PathCache"
public PathChildCache(CuratorFramework client,String path,boolean cacheData);
> client curator的客户端对象
> path 监听的节点路径
> cacheDate 一个布尔值,
   > true 表示子节点列表发生变化的时候同时获取子节点列表的数据内容
   > false 表示cache将不会缓存节点数据,下面pathChildrenCacheEvent.getData().getData()将返回null
 "想使用cache,必须调用它的start()方法,使用完成后调用close()方法,可以设置StartMode来实现启动模式。"

'启动模式'
   NORMAL 正常初始化
   BUILD_INITIAL_CACHE 在调用start()之前会调用rebuild()
   POST_INITIALIZED_EVENT 当cache初始化数据后发送一个PathChildrenCacheEvent.Type#INITIALIZED事件

final PathChildrenCache pathChildrenCache = new PathChildrenCache(zkclient,"/name",true);
        /**
         * 设置启动模式, 直接调用start()方法相当于采用了StartMode.NORMAL的模式
         */
        childrenCache.start(PathChildrenCache.StartMode.NORMAL);
        //从事件对象中拿到事件的类型 addListener 增加listener监听缓存的变化
        pathChildrenCache.getListenable().addListener(
                //增加listener监听缓存的变化
          new PathChildrenCacheListener() {
           public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
                        //获取子节点的事件类型3种
                        switch (pathChildrenCacheEvent.getType()){
                            case CHILD_ADDED:  //添加子节点
                                System.out.println(pathChildrenCacheEvent.getData().getData());//得到子节点具体信息
                                break;
                            case CHILD_UPDATED: //子节点数据内容改变
                                break;
                            case CHILD_REMOVED: //子节点删除
                                break;
                            default:
                                break;
                    }
            }
        });
        //返回一个List<ChildData>对象,遍历所有的子节点
        pathChildrenCache.getCurrentData();
```

```java
public void PathCacheDemo {
        String PATH = "/example/pathCache";
        TestingServer server = new TestingServer();
        CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
        client.start();
        PathChildrenCache cache = new PathChildrenCache(client, PATH, true);
        cache.start();
        //子节点监听器
        PathChildrenCacheListener cacheListener = (client1, event) -> {
            System.out.println("事件类型：" + event.getType());
            if (null != event.getData()) {
                System.out.println("节点数据：" + event.getData().getPath() + " = " + new String(event.getData().getData()));
            }
        };
        //添加子节点监听器
        cache.getListenable().addListener(cacheListener);
        client.create().creatingParentsIfNeeded().forPath("/example/pathCache/test01", "01".getBytes());
        Thread.sleep(10);
        client.create().creatingParentsIfNeeded().forPath("/example/pathCache/test02", "02".getBytes());
        Thread.sleep(10);
        client.setData().forPath("/example/pathCache/test01", "01_V2".getBytes());
        Thread.sleep(10);
        //返回所有子节点列表
        for (ChildData data : cache.getCurrentData()) {
            System.out.println("getCurrentData:" + data.getPath() + " = " + new String(data.getData()));
        }
        //删除子节点
        client.delete().forPath("/example/pathCache/test01");
        Thread.sleep(10);
        client.delete().forPath("/example/pathCache/test02");
        Thread.sleep(1000 * 5);
        cache.close();
        client.close();
        System.out.println("OK!");
    }
}

```

##### nodecache

```java
只能监听一个节点的变化
    与pathCache类似，NodeCache只是监听某个特定节点,涉及到如下三个类
    NodeCache NodeCache实现类
    NodeCacheListener 节点监听器
    ChildData 节点数据
    note: 使用cache,依然调用它的start方法(),使用完成后调用close()方法。
    getCurrentData()将得到节点当前的状态,通过它的状态可以得到当前的值。

@Test
public void NodeCacheDemo {
        String PATH = "/example/cache";
        TestingServer server = new TestingServer();
        CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
        client.start();
        client.create().creatingParentsIfNeeded().forPath(PATH);

        //建立一个nodeCahe
        final NodeCache cache = new NodeCache(client, PATH);
        cache.start();


        NodeCacheListener listener = () -> {
            ChildData data = cache.getCurrentData();
            if (null != data) {
                System.out.println("节点数据：" + new String(cache.getCurrentData().getData()));
            } else {
                System.out.println("节点被删除!");
            }
        };
        cache.getListenable().addListener(listener);


        client.setData().forPath(PATH, "01".getBytes());
        Thread.sleep(100);
        client.setData().forPath(PATH, "02".getBytes());
        Thread.sleep(100);
        client.delete().deletingChildrenIfNeeded().forPath(PATH);
        Thread.sleep(1000 * 2);
        cache.close();
        client.close();
        System.out.println("OK!");
}


```

##### treecache

```java
Tree Cache可以监控整个树上的所有节点,类似于PathCache和NodeCache的组合,主要涉及下面4个类：
TreeCache treeCache实现类
TreeCacheListener 监听器类
TreeCacheEvent 触发的事件类
ChildData 节点数据

@Test
public void TreeCacheDemo {
        String PATH = "/example/cache";
        TestingServer server = new TestingServer();
        CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
        client.start();
        client.create().creatingParentsIfNeeded().forPath(PATH);
        TreeCache cache = new TreeCache(client, PATH);
        TreeCacheListener listener = (client1, event) ->
                System.out.println("事件类型：" + event.getType() +
                        " | 路径：" + (null != event.getData() ? event.getData().getPath() : null));
        cache.getListenable().addListener(listener);
        cache.start();
        client.setData().forPath(PATH, "01".getBytes());
        Thread.sleep(100);
        client.setData().forPath(PATH, "02".getBytes());
        Thread.sleep(100);
        client.delete().deletingChildrenIfNeeded().forPath(PATH);
        Thread.sleep(1000 * 2);
        cache.close();
        client.close();
        System.out.println("OK!");
}

注意：
TreeCache在初始化(调用start()方法)的时候会回调TreeCacheListener实例一个事TreeCacheEvent
而回调的TreeCacheEvent对象的Type为INITIALIZED，ChildData为null，
此时event.getData().getPath()很有可能导致空指针异常，这里应该主动处理并避免这种情况。

```

#### zookeeper 会话过期

```java
Curator提供了对zookeeper客户端的封装, 并监控链接状态和会话session, 特别是会话session过期后, curator能够重新链接zookeeper, 并且创建一个新的session。

使用ZK时, session的概念非常重要.
```

##### 什么事 zk 的会话过期

```java
一般来说我们使用zk都是采用集群形式,如下图, client 和zk集群(3个实例)建立一个会话session。
这个会话session当中, client其实是随机与其中一个 zk provider建立的链接, 并且互发心跳heartbeat。zk集群负责管理这个session, 并且在所有的provider上维护这个session的信息, 包括这个session中定义的临时数据和监视点watcher。

如果在网络不佳或者zk集群中某一台provider挂掉的情况下,有可能出现connection loss的情况, 例如client和zk provider1连接断开, 这时候client不需要任何操作(zookeeper api已经给我们做好了) 只需要等待client与其他provider重新链接即可。
这个过程可能导致两个结果:

 1) 在session timeout 之内连接成功
 	这个时候cLient 成功切换到连接另一个provider 例如provider2, 由于zk在所有provider上同步了session相关的数据,此时可以认为无缝迁移了。

 2) 在session timeout 之内没有重新连接
    这就是session expore的情况, 这时候zookeeper集群会认为任务会话已经结束, 并清楚和这个session有关的所有数据,包括临时节点和注册的监视点watcher。
    在session 超时之后, 如果client 重新连接上了zookeeper集群, 很不幸, zookeeper会发出session expired异常,且不会重新session, 也就是不会重建临时数据和watcher。
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1636942983401-8a43ab0a-63b9-4b7b-923d-ca899a347f0c.png#clientId=u34fa5c1c-c817-4&from=paste&height=86&id=u844ab7a3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=172&originWidth=322&originalType=binary&ratio=1&size=6751&status=done&style=none&taskId=u84f50324-16d7-4305-b986-d70e860e60b&width=161)

```java
public class SessionExceptionDemo {

    public static void main(String[] args) throws Exception {
        String rootPath = "/ssgao";
        String hostAddress = InetAddress.getLocalHost().getHostAddress();
        String serviceInstance = "/prometheus" + "-" +  hostAddress + "-";
        String path = rootPath + serviceInstance;

        CuratorFramework client = buildClient(1000,2,"127.0.0.1:2181",100,200);

        /**
         * todo
         *  这里我们创建了一个临时有序节点node, 这个节点将会在session expired触发的时候被自动书暗处。
         *  当session又重新恢复的时候, client只会收到session expired 异常 并不会自动将临时节点添加到zk中,
         *  为了解决这个问题,我们增加一个监听器 :  client.getConnectionStateListenable().addListener(sessionConnectionListener);
         *
         *  这个监听器监听session expired事件, 并且在事件发生的时候进行处理, 监听器处理的流程如下:
         *      ps: 这个监听器注册是可以复用的,即如果多次session expired,不用重复注册监听器
         */
        SessionConnectionListener sessionConnectionListener = new SessionConnectionListener(path, "");
        client.getConnectionStateListenable().addListener(sessionConnectionListener);
        client.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(path);

    }

    public static CuratorFramework buildClient(int baseSleepTimeMs, int maxRetries,
                                        String zookeeperServer,int sessionTimeoutMs,int connectionTimeoutMs){
        // todo 设置重试策略 retryPolicy 和会话超时时间 sessionTimeoutMs
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(baseSleepTimeMs,maxRetries);
        CuratorFramework client =  CuratorFrameworkFactory
                    .builder()
                    .connectString(zookeeperServer)
                    .retryPolicy(retryPolicy)
                    .sessionTimeoutMs(sessionTimeoutMs)
                    .connectionTimeoutMs(connectionTimeoutMs)
                    .build();

        client.start();
        return client;
    }
}
```

```java
public class SessionConnectionListener implements ConnectionStateListener {

    private final Logger logger = LoggerFactory.getLogger(SessionConnectionListener.class);

    private String path;
    private String data;

    public SessionConnectionListener(String path, String data) {
        this.path = path;
        this.data = data;
    }

    @Override
    public void stateChanged(CuratorFramework client, ConnectionState newState) {
        // todo 这里的ConnectionState.LOST 等同于session expired 时间, 对整个事件的处理是在一个死循环中重试连接zk,直到链接成功才退出
        if(newState==ConnectionState.LOST){
            logger.error("[负载均衡失败] zk session 超时 ~ ");
            while (true){
                try{
                    if(client.getZookeeperClient().blockUntilConnectedOrTimedOut()){
                        client.create().creatingParentsIfNeeded()
                                       .withMode(CreateMode.EPHEMERAL_SEQUENTIAL)
                                       .forPath(path,data.getBytes(StandardCharsets.UTF_8));
                        logger.info("[负载均衡修复] zk reconnect success ");
                        break;
                    }
                }catch (InterruptedException interruptedException){
                    break;
                }catch (Exception e){

                }

            }
        }
    }
}
```

> _**特别提示: watcher 仅仅是一次性的, zookeeper 通知 watcher 事件后, 就会将这个 watcher 从 session 中删除, 因此如果要想继续监控,就要添加新的 watcher**_

> [_https://cloud.tencent.com/developer/article/1050407_](https://cloud.tencent.com/developer/article/1050407)

### zookeeper 的原生 API

```java
org.apache.zookeeper
org.apache.zookeeper.data
org.apache.zookeeper.server
org.apache.zookeeper.server.quorum
org.apache.zookeeper.server.upgrade

org.apache.zookeeper包含zookeeper类,它是我们编程时最常使用的类文件.
这个类是zookeeper客户端的主要类文件，如果要使用zookeeper服务,应用程序首先必须创建一个zookeeper实例，这时就需要使用此类
一旦客户端和zookeeper服务端建立起了连接，zookeeper系统将会 给此连接会话分配一个ID值,并且客户端将会周期性向服务器端发送心跳来维持会话连接。
只要连接有效,客户端就可以使用zookeeper api来进行相应的处理。
```

```java
zkClient是GitHub上一个开源的zookeeper客户端
zkClient在zookeeper原生API接 口之上进行包装,是一个 更加易用的zookeeper客户端
zkClient在内部实现了session连接超时,watcher反复注册等功能
```

#### zookeeper 主要的方法

```java
创建回话的方法:客户端可以创建一个zookeeper实例来连接zookeeper服务器
 zookeeper(Arguments)方法,一共4个构造方法,根据参数不同
 connectionString 连接服务器列表,使用","分割
 sessionTimeout 心跳检测时间周期
 wather 事件处理通知器
 canBeReadOnly 标识当前回话是否支持只读
 sessionId和SessionPassword 提供连接zk的sessionId和密码,通过这个可与确定唯一一台客户端,目的是可以提供重复回话
 注意:zk客户端和服务端回话建立是一个异步过程,也就是说在程序中,我们程序方法处理完客户端初始化后立即返回
 (也就是说程序往下执行代码,这样,大多数情况下我们并没有真正构建好一个可用回话,在会话生命周期处于"connectioning"时才算真正建立完毕)

public class TestConnect implements Watcher {
    public static CountDownLatch signal = new CountDownLatch(1);
    public static void main(String[] args) {
        String hostport="172.19.40.118:2181";
        try {
            ZooKeeper zooKeeper = new ZooKeeper(hostport,10000,new TestConnect());
            //阻塞在这里,直到CountDownLatch计数为0
            signal.await();
            //ZooDefs.Ids.OPEN_ACL_UNSAFE 开放权限
            zooKeeper.create("/ssgao","ssgao".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL);
            System.out.println("启动成功");
            zookeeper.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public void process(WatchedEvent event) {
        if(event.getState().equals(Event.KeeperState.SyncConnected)){
            System.out.println("zk连接成功!!");
            signal.countDown();
        }
    }
}

String create (final string path,byte data[],List acl,CreateMode createMode)
void create(final String path, byte data[], List<ACL> acl,CreateMode createMode,  StringCallback cb, Object ctx)
    创建一个znode节点
    参数1：path 路径，
    参数2：data[] znode内容，
    参数3: acl acl(访问控制权限),
    参数4：znode创建类型
            节点权限,使用Ids.OPEN_ACL_UNSAFE开放权限设置
            CreateMode.PERSISTENT 持久节点
            CreateMode.PERSISTENT_SEQUENTIAL 持久顺序节点
            CreateMode.EPHEMERAL 临时节点
            CreateMode.SEQUENTIAL 临时顺序节点
    异步方式(在同步方式的基础上增加两个参数)
    参数5：注册一个异步回调函数,要实现AsynCallBack,StringCallBack接口,重写processResult(int rc,String path,Object ctx,String name)
         rc:为服务端响应码0表示调用成功, -4表示端口连接 -110表示指定节点存在  -112表示会话已过期
         path:接口调用时传入API数据节点的路径参数
         ctx:为调用接口传入API的context值
         name: 实际在服务器端创建节点的名称
    参数6：传递给回调函数的参数,一般为上下文信息(context)



void delete(final String path ,int version)
void delete(final String path, int version, VoidCallback cb,Object ctx)
    删除一个znode节点
    参数1：path 路径，
    参数2：version 版本号，如果版本号与znode的版本号不一致，将无法删除,是一种乐观加锁,version=-1 表示跳过版本检查
    zooKeeper.delete("/ssgao", -1, new AsyncCallback.VoidCallback() {
                public void processResult(int rc, String path, Object ctx) {
                    System.out.println("rc:"+rc);
                    System.out.println("删除的节点:"+path);
                    System.out.println("ctx上下文信息:"+ctx);
                }
            },"delete");


public Stat exists(String path, boolean watch)
public Stat exists(final String path, Watcher watcher)
public void exists(final String path, Watcher watcher, StatCallback cb, Object ctx)
    判断某个znode节点是否存在
    参数:路径，watcher(监视器）当znode节点被改变时，将会触发当前watcher

Stat exists(String path,boolean watch)
    判断某个znode节点是否存在,节点不存在返回null
    参数1: path 路径并设置是否监控这个目录节点，这里的watch 是在创建zookeeper实例时指定的watcher

Stat setData(String path,byte[] data,int version)
    设置节点的数据
    参数1:path 路径
    参数2:version,版本号 如果版本号为-1,跳过版本检查

byte[] getData(final String path,Watcher watcher,Stat stat)
 void getData(String path, boolean watch, DataCallback cb, Object ctx)
    获取某个znode节点上的数据
    参数：路径,监视器，数据版本等信息

public List<String> getChildren(String path, boolean watch)
public void getChildren(final String path, Watcher watcher,ChildrenCallback cb, Object ctx)
    获取某个节点下所有直接子节点(第一级子节点,不可以递归)
    参数：路径，监视器，该方法有多个重载
```

#### zookeeper 的使用介绍

```java
创建一个与zookeeper服务器的连接
参数1 服务器地址与端口号
参数2 连接会话超时时间
参数3 观察者,连接成功会触发该观察者,不过只会触发一次，该watcher会获取所有事件的通知
Zookeeper zk = new ZooKeeper("127.0.0.1:2181",5000,new Watcher(){
  public void process(WatchedEvent event){
   System.out.print("监控所有被触发的事件:EVENT:"+event.getType());
  }
});

public class CreateSession implements Watcher
{
    public static void main( String[] args ) throws Exception {
        //实例化客户端
        ZooKeeper zooKeeper = new ZooKeeper("192.168.10.125:2181",5000, new CreateSession());
        System.out.println(zooKeeper.getState());
        Thread.sleep(5000);
        System.out.println(zooKeeper.getState());
    }
    public void process(WatchedEvent event) {
        System.out.println("接收到事件:"+event);
        //获取事件的状态
        KeeperState keepState = event.getState();
        //获取事件的类型
        EventType eventType = event.getType();
        //如果是建立连接
        if(KeeperState.SyncConnected == KeeperState){
            if(EventType.None == eventType){
              //如果建立连接成功,则发送信号量,让后续阻塞程序向下执行
               connectedSemaphore.CountDown();
               System.out.println("zk 建立连接");
            }
        }
    }
}
```

##### 创建 zookeeper 客户端端

```java
创建客户端方法ZKClient(Arguments)
    参数1 zkServers,zookeeper服务器地址,用","分隔
    参数2 sessionTimeout 会话超时时间,单位为毫秒
    参数3 connectionTimeout 连接超时时间
    参数4 IZKConnection 接口实现类
    参数5 zkSerializer 自定义序列化实现

zkClient给开发人员提供一套监听方式,我们可以使用监听节点的方式操作，剔除了繁琐的反复watcher操作，简化代码复杂度
----监听节点的变化
subscribeChildChanges方法(只监听节点的变化）
参数1 path路径
参数2 实现了IZKChildListener接口的类,只需要重写handleChildChanges(String parentPath,List<String>currentChilds)方法
其中parentPath为所监听节点的全路径，currentChilds为最新的子节点列表(相对路径）
    IZKClien事件针对下面三个事件触发
    新增子节点
    减少子节点
    删除节点
----监听节点数据的变化
subscriteDataChanges方法(只监听数据的变化)
参数1 path路径
参数2 实现了IZKDataListener接口的类
    需要重载handlerDataDeleted(String path)方法(节点删除时触发)和handlerDataChanges(String path,Object data)方法
    经测试，该接口只会对所有监控的path的数据变化，子节 点数据变化不会监控到

```

```java
create,createEphemeral,createEphermeralSequential,createPersistent,createPersistentSequential
    参数1 path 路径
    参数2 data 数据内容 可以传入null
    参数3 mode 节点类型,为一个枚举类型
    参数4 acl策略
    参数5 callback 回调函数
    参数6 context 上下文对象
    参数7 createParents是否创建父节点

删除节点方法 delete,deleteRecursive
    参数1 path路径
    参数2 callback回调函数
    参数3 context上下文对象

读取子节点数据方法 getChildren
    参数1 path路径

读取节点数据方法 readData
    参数1 path路径
    参数2 returnNullIfPathNotExist(避免空节点抛出异常,直接返回null)
    参数3 节点状态

```

##### zookeeper 创建连接

```java
ZkClient zkClient = new ZkClient("192.168.11.69:2181",10000,100000,new SerializableSerializer());
System.out.println("connection ok!");
```

##### zk 客户端节点操作

```java
public static void main(String[] args) {                                                                                                       ZkClient zkClient = new ZkClient("192.168.11.69:2181",10000,100000,new SerializableSerializer());
    User user = new User();
    user.setName("ssgao");
    user.setAge(30);
    //创建节点
    zkClient.create("/zkc_a",user, CreateMode.EPHEMERAL);
    //读取节点
    User res = zkClient.readData("/zkc_a",true);
    System.out.println(res.toString());
}

//todo 其他节点操作
public static void main(String[] args) {
   ZkClient zkClient = new ZkClient("192.168.11.69:2181",10000,100000,new SerializableSerializer());
   //获取节点状态
   Stat stat = new Stat();
   User res = zkClient.readData("/zkc_a",stat);
   //获取节点子节点
   List<String> nodes = zkClient.getChildren("/zkc_a");
   //检查节点是否存在
   zkClient.exists("/zkc_a");
   //删除节点,无子节点
   zkClient.delete("/zkc_a");
   //循环删除,有子节点
   zkClient.deleteRecursive("/zkc_a");
}


public class ZkClient_Demo {
    static final String connect_dir="172.19.40.118:2181";
    static final int session_outtime=5000;
    public static void main(String[] args) {
        ZkClient client = new ZkClient(ZkClient_Demo.connect_dir,ZkClient_Demo.session_outtime);
        //create and delete 方法
        client.createEphemeral("/ssgao");
        //递归创建
        client.createPersistent("/xiaoxiao/a",true);
        //递归删除
        client.deleteRecursive("/xiaoxiao");
        //设置path和data,并且读取子节点的内容
        client.createPersistent("/root","root");
        client.createPersistent("/root/a","root/a");
        client.createPersistent("/root/b","root/b");
        for(String p : client.getChildren("/root")){
            System.out.println("节点:"+p);
            String data = client.readData("/root/"+p);
            System.out.println("数据信息:"+data);
        }
        client.deleteRecursive("/root");
        //更新节点数据信息
        client.create("/temp","temp", CreateMode.EPHEMERAL);
        client.writeData("/temp","newData");
        System.out.println("节点数据:"+client.readData("/temp"));
        client.deleteRecursive("/temp");
        //关闭连接
        client.close();
    }
}

```

##### zookeeper 创建节点

```java
"zookeeper 采用同步方式创建节点"
public class CreateNode implements Watcher{
    public  static ZooKeeper zooKeeper;
    public static void main( String[] args ) throws Exception {
        zooKeeper = new ZooKeeper("192.168.10.125:2181",5000, new CreateNode());
        Thread.sleep(5000);
    }
    public void process(WatchedEvent event) {
        System.out.println("接收到事件:"+event);
        //判断是否已经连接
        if(event.getState()== Event.KeeperState.SyncConnected){
            try {
                String res = zooKeeper.create("/test-a","123".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
                System.out.println("节点创建完成!"+res);
            } catch (KeeperException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

"zookeeper 采用异步的方式创建节点"
 public class CreateAysncNode implements Watcher{
    public  static ZooKeeper zooKeeper;
    public static void main( String[] args ) throws Exception {
        zooKeeper = new ZooKeeper("192.168.10.125:2181",5000, new CreateAysncNode());
        Thread.sleep(5000);
    }

    public void process(WatchedEvent event) {
        System.out.println("接收到事件:"+event);
        //判断是否已经连接
        if(event.getState()== Event.KeeperState.SyncConnected) {
           zooKeeper.create("/test-b",
                    "123".getBytes(),
                    ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.PERSISTENT,
                    new CallBack(),"create");

            System.out.println("节点创建完成!");
        }
    }

    static class CallBack implements AsyncCallback,AsyncCallback.StringCallback{
        public void processResult(int rc, String path, Object ctx, String name) {
            StringBuffer res = new StringBuffer();
            res.append("rc:"+rc+"\n");
            res.append("path:"+path+"\n");
            res.append("ctx:"+ctx+"\n");
            res.append("name:"+name);
            System.out.println(res.toString());
        }
    }
}

输出结果：
接收到事件:WatchedEvent state:SyncConnected type:None path:null
节点创建完成!
rc:0
path:/test-b
ctx:create
name:/test-b
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1635638096870-69815ad5-871d-4f3c-a002-61102427afe8.png#clientId=u1d4bf6c3-4907-4&from=paste&height=617&id=u969c4a7d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1234&originWidth=1554&originalType=binary&ratio=1&size=1364876&status=done&style=none&taskId=u2521eb20-6c28-4023-a5ce-91ef680b15d&width=777)

##### 获取指定节点子节点

```java
"同步获取节点"

public class GetChildNode implements Watcher {
    public  static ZooKeeper zooKeeper;
    public static void main( String[] args ) throws Exception {
        zooKeeper = new ZooKeeper("192.168.10.125:2181",5000, new GetChildNode());
        Thread.sleep(15000*10);
    }
    public void process(WatchedEvent event) {
        System.out.println("接收到事件:"+event);
        //获取所有子节点
        if(event.getState()== Event.KeeperState.SyncConnected){
            if(event.getType()== Event.EventType.None&&null==event.getPath()){
                getChildNode();
            }else{
                //如果节点子节点变化
                if(event.getType()== Event.EventType.NodeChildrenChanged){
                    getChildNode();
                }
            }

        }
    }
    public void getChildNode() {
        try {
            List<String> list = zooKeeper.getChildren("/", true);
            System.out.println(list);
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

"异步节点获取"
 public class GetAysncChildNode implements Watcher{
    public  static ZooKeeper zooKeeper;
    public static void main( String[] args ) throws Exception {
        zooKeeper = new ZooKeeper("192.168.10.125:2181",5000, new GetAysncChildNode());
        Thread.sleep(15000*10);
    }

    public void process(WatchedEvent event) {
        System.out.println("接收到事件:"+event);
        //获取所有子节点
        if(event.getState()== Event.KeeperState.SyncConnected){
            if(event.getType()== Event.EventType.None&&null==event.getPath()){
                getChildNode();
            }else{
                //如果节点子节点变化
                if(event.getType()== Event.EventType.NodeChildrenChanged){
                    getChildNode();
                }
            }

        }
    }
    public void getChildNode() {
       zooKeeper.getChildren("/", true, new IChildren2CallBack(),null );
    }
    public static class IChildren2CallBack implements AsyncCallback,AsyncCallback.Children2Callback{

        public void processResult(int rc, String path, Object ctx, List<String> children, Stat stat) {
            System.out.println(rc);
            System.out.println(path);
            System.out.println(ctx);
            System.out.println(children);
            System.out.println(stat);
        }
    }
}
```

##### zookeeper 修改数据

```java
public class EditData implements Watcher{
    public  static ZooKeeper zooKeeper;
    public static void main( String[] args ) throws Exception {
        zooKeeper = new ZooKeeper("192.168.10.125:2181",5000, new EditData());
        Thread.sleep(15000*10);
    }
    //监听回调接口
    public void process(WatchedEvent event) {
        System.out.println("接收到事件:"+event);
        //获取所有子节点
        if(event.getState()== Event.KeeperState.SyncConnected){
            if(event.getType()== Event.EventType.None&&null==event.getPath()){
                dataprocess();
                asyncdataprocess();
            }else{
                //如果节点子节点变化
                if(event.getType()== Event.EventType.NodeChildrenChanged){
                    dataprocess();
                    asyncdataprocess();
                }
            }
        }
    }
    //同步处理
    public void dataprocess() {
        try {
            Stat stat = zooKeeper.setData("/test-a","123".getBytes(),-1);
            System.out.println(stat);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    //异步处理
    public void asyncdataprocess() {
        try {
           zooKeeper.setData("/test-a","124".getBytes(),-1,new IStatCallBack(),null);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    //异步处理需要实现的接口
    public static class IStatCallBack implements AsyncCallback.StatCallback{
        public void processResult(int rc, String path, Object ctx, Stat stat) {
            System.out.println(rc+"|"+path+"|"+ctx+"|"+stat);
        }
    }
}
```

#### zookeeper 权限管理

```java
ACL(Access Control List)
    zookeeper作为一个分布式协调框架,其内部存储的都是一些关于分布式系统运行时状态的元数据,尤其是涉及到一些分布式锁
    master选举和协调等应用场景。我们需要有效的保障zk数据安全。
ZK提供了三种模式"权限模式、授权对象、权限"

'zookeeper的访问策略'
* 基于IP白名单
* 基于digest(用户名和密码)


'授权对象(ID)'
* ip权限模式：具体的ip地址
   ip模式是通过ip地址粒度来进行控制权限,例如配置了ip:192.168.1.107
       即表示权限控制都是针对这个ip地址的
       同时也支持按网段进行分配,比如 192.168.1.*


* digest权限模式：username:Base64(SHA-1(username:password))
    Digest是最常用的权限控制模式,也符合我们对权限控制的认知,类似username:password 形式的权限标识进行权限配置
    zk会对形成的权限标识先后进行两次编码处理,分别是SHA-加密算法 / BASE64编码

'权限(permission)'
create(c) delete(d) read(r) write(w) admin(a)
只有一个权限为单个权限
含有所有权限为完全权限
还有多个权限为复合权限
权限组合：scheme+ID+permission
```

##### zookeeper 权限信息

```java
public class AclSession {
    public  static ZooKeeper zooKeeper;
    public static void main( String[] args ) throws Exception {
        zooKeeper = new ZooKeeper("192.168.11.69:2181",5000, new GetChildNode());
        Thread.sleep(15000*10);
    }

    public void process(WatchedEvent event) throws Exception {
        System.out.println("接收到事件:"+event);
        //创建节点
        if(event.getState()== Watcher.Event.KeeperState.SyncConnected){
            //基于IP的规则
            ACL ipacl = new ACL(ZooDefs.Perms.READ,new Id("ip","192.168.11.65"));
            //基于用户名密码规则
            ACL digestacl = new ACL(ZooDefs.Perms.READ|ZooDefs.Perms.WRITE,new Id("digest",
                    DigestAuthenticationProvider.generateDigest("ssgao:ssgao1987")));
            ArrayList<ACL> acls = new ArrayList<ACL>();
            String path= zooKeeper.create("/acl_aa","124".getBytes(),acls,CreateMode.EPHEMERAL);
            System.out.println("节点创建成功:"+path);

            //等价于zookeeper自带的ID认证信息
            zooKeeper.addAuthInfo("digest","ssgao:ssgao1987".getBytes());
            String patha= zooKeeper.create("/acl_aa","124".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL,CreateMode.EPHEMERAL);
        }
    }
}
```

```java
public class TestAcl implements Watcher {
    private static String hostport = "172.19.40.118:2181";
    private static String path="/path";
    private static String path_del="/path/del";
    private static String authentication_type="digest";
    private static String corrent = "123456";
    private static String incorrect="654321";

    static ZooKeeper zooKeeper = null;
    AtomicInteger atomicInteger = new AtomicInteger();
    private CountDownLatch countDownLatch = new CountDownLatch(1);
    //创建连接
    public void createConnection(String connectString,int timeout){
        try{
            zooKeeper = new ZooKeeper(connectString,timeout,this);
            //添加授权,zookeeper已经添加了认证,这时客户端创建的各个节点都是添加了认证的信息的
            zooKeeper.addAuthInfo(authentication_type,corrent.getBytes());
            countDownLatch.await();
        }catch(Exception e){
            e.printStackTrace();
        }
    }
    //释放连接
    public void releaseConnection(){
        try {
            if(zooKeeper!=null)
                zooKeeper.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    /**
     * 处理监听事件
     * @param event
     */
    public void process(WatchedEvent event) {
        //连接状态
        Event.KeeperState state = event.getState();
        //事件类型
        Event.EventType eventType = event.getType();
        if(event.getState().equals(Event.KeeperState.SyncConnected)) {
            if (eventType.equals(Event.EventType.None)) {
                System.out.println("成功连接zk");
                countDownLatch.countDown();
            }
        }
    }
    public void aclOption(List<ACL> acls) throws Exception{
        System.out.println("测试认证acl");
        //这里的acls 和客户可的authInfo是两种不同的改变
        // acl 可以认为是创建节点时,指定的一种认证方式
        zooKeeper.create("/ssgao","ssgao".getBytes(),acls, CreateMode.EPHEMERAL);
        //没有认证
        getByNoAcl();
        //错误认证
        getByErrorAcl();
        //正确认证
        getByCorrectAcl();
    }

    //org.apache.zookeeper.KeeperException$NoAuthException: KeeperErrorCode = NoAuth for /ssgao
    //提示没有认证
    public void getByNoAcl(){
        ZooKeeper zooKeeper=null;
        try {
            zooKeeper = new ZooKeeper(TestAcl.hostport,1000,this);
            zooKeeper.getData("/ssgao",false,null);

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                zooKeeper.close();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    //org.apache.zookeeper.KeeperException$NoAuthException: KeeperErrorCode = NoAuth for /ssgao
    //错误的认证信息
    public void getByErrorAcl(){
        try {
            ZooKeeper zooKeeper = new ZooKeeper(TestAcl.hostport,1000,this);
            zooKeeper.addAuthInfo(authentication_type,incorrect.getBytes());
            zooKeeper.getData("/ssgao",false,null);
            zooKeeper.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    //正确的认证,才可以获取信息
    public void getByCorrectAcl(){
        try {
            System.out.println(new String(zooKeeper.getData("/ssgao",false,null)));
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        TestAcl testAcl = new TestAcl();
        testAcl.createConnection(TestAcl.hostport,10000);
        List<ACL> list = new ArrayList<ACL>(1);
        for(ACL acl: ZooDefs.Ids.CREATOR_ALL_ACL){
            list.add(acl);
        }
        try {
            testAcl.aclOption(list);
        }catch(Exception e){
            e.printStackTrace();
        }
        testAcl.releaseConnection();
    }
}
```

#### zookeeper 监听器

```java
ZKclient里面并没有类似的watcher,watch参数,这也就是说我们无需关心注册Watcher的问题,ZKClient给我们提供一套监听方式。
我们可以使用监听节点的方式进行操作,剔除了繁琐的反复watcher操作,大大简化代码复杂度

subscribeChildChanges方法
    参数1 path路径
    参数2 实现了IZKChildListener接口的类
         (如: 实例化IZKChildListener类)只需要重写其handleChildChanges(String parentPath,List<String> currentChilds)方法
         其中parentPath为监听节点的全路径
             currentChilds为最新的子节点列表(相对路径)
    "监听：新增子节点,减少子节点,删除节点都会触发该事件,节点数据改变不触发该监听事件"

IZKChildListener的特点如下:
  1> 客户端可以对一个不存在的节点进行变更的监听
  2> 一旦客户端对一个节点注册子节点列表变更监听后,那么当前节点的子节点列表发生变更的时候,服务端都会通知客户端,并将最新的子节点列表发送个客户端
  3> 该节点本身创建和删除也会通知客户端
  4> 这个监听时一直存在的,不是单词监听,从而相比较原生的API提供的方式要简单的多了。



public static void main(String[] args) {
        ZkClient zkClient = new ZkClient("192.168.11.69:2181",10000,100000,new SerializableSerializer());

        User user = new User();
        user.setName("ssgao");
        user.setAge(30);
        //节点数据修改 -1表示数据版本号
        zkClient.writeData("/zkc_a",user,-1);
    }

```

##### 订阅节点子节点数量变化

```java
public class SubNodeCount {
    public static void main(String[] args) {
        ZkClient zkClient = new ZkClient("192.168.11.69:2181",10000,100000,new SerializableSerializer());
        zkClient.subscribeChildChanges("/zkc_a",new ZKChildListener());
    }

    public static class ZKChildListener implements IZkChildListener{
        //s 节点路径 ,list 子节点列表
        public void handleChildChange(String s, List<String> list) throws Exception {
            System.out.println(s);
            System.out.println(list);
        }
    }
}

```

##### 订阅节点内容变化

```java
public class SubNodeContent {
    public static void main(String[] args) {
        //序列化器 BytesPushThroughSerializer 兼容zkCli 字符串
        ZkClient zkClient = new ZkClient("192.168.11.69:2181",10000,100000,new BytesPushThroughSerializer());
        zkClient.subscribeDataChanges("/zkc_a",new ZKChildListener());
    }
    public static class ZKChildListener implements IZkDataListener{
        //节点数据变化
        public void handleDataChange(String s, Object o) throws Exception {
            System.out.println(o);
        }
        //节点数据被删除
        public void handleDataDeleted(String s) throws Exception {
            System.out.println(s);
        }
    }
}
```

```java
public class ZkClient_watch {
    static final String connect_dir="172.19.40.118:2181";
    static final int session_outtime=5000;
    public static void main(String[] args) throws InterruptedException {
        //初始化客户端
        ZkClient client = new ZkClient(ZkClient_watch.connect_dir, ZkClient_watch.session_outtime);
        //监听父节点的子节点的变化,子节点内容发生改变的时候不会触发该事件
        client.subscribeChildChanges("/ssgao", new IZkChildListener() {
            public void handleChildChange(String s, List<String> list) throws Exception {
                System.out.println("父节点:"+s);
                System.out.println("子节点:"+list.toString());
            }
        });
        //监听节点的数据变化,创建和改变节点"/ssgao"都会触发该事件
        client.subscribeDataChanges("/ssgao", new IZkDataListener() {
            public void handleDataChange(String s, Object o) throws Exception {
                System.out.println("节点数据改变:"+s+",改变的内容为:"+o);
            }
            public void handleDataDeleted(String s) throws Exception {
                System.out.println("节点删除:"+s);
            }
        });
        client.create("/ssgao","ssgao",CreateMode.PERSISTENT);
        Thread.sleep(1000);
        client.create("/ssgao/a","ssgao/a",CreateMode.EPHEMERAL);
        Thread.sleep(1000);
        client.writeData("/ssgao/a","newData");
        Thread.sleep(1000);
        client.writeData("/ssgao","newData");
        Thread.sleep(1000);
        //删除节点
        client.deleteRecursive("/ssgao");
        Thread.sleep(2000);
        //关闭连接
        client.close();
    }
}
```

####

#### watcher 数据变更通知

```java
 zk 提供了分布式数据的发布/订阅功能,一个典型的发布/订阅模型系统定义了一种一对多的订阅关系,能够让多个订阅者同时监听某一个主题对象,
当该主题对象自身状态变化时,会通知所有订阅者,使得他们能够做出相应的处理。在zk中通过watch机制来实现这种分布式通知的功能。
zk 允许客户端向服务端注册一个watcher监听,当服务端的一些事件触发了这个watcher,那么就会向指定客户端发送一个事件通知来实现分布式通知的功能。


"Watcher接口"
 在zookeeper中,接口类Watcher用于标识一个标准的事件处理器,其定义了事件通知相关的逻辑,包含"KeeperState"和"EventType"两个枚举类
 分别代表了通知状态和事件类型,同时定义了事件的回调方法process(),当zk向客户端发送一个Watcher事件通知时,客户端就会对相应的process方法进行回调
     process(WatchedEvent event)
 WatchedEvent包含了每一个事件的三个基本属性：
         通知状态（keeperState）
         事件类型（EventType）
         节点路径（path）
 当/zk-book这个节点的数据发生变更时，服务端会发送给客户端一个"ZNode数据内容变更"事件，客户端只能够接收到如下信息：
     keeperState：SyncConnected
     EventType：NodeDataChanged
     Path：/zk-book


"watcher事件"
 keeperState状态类型(与客户端实例相关),EventType事件类型(与znode节点相关)。
 keeperState        EventType         触发条件
 ------------------------------------------------------------------------------------------------
                   None(-1)              客户端与服务端成功建立连接
 SyncConnected(0)  NodeCreader(1)        watcher监听的对象数据节点被创建
                   NodeDeleted(2)        watcher监听的对应数据节点被删除
                   NodeDataChanged(3)    watcher监听的对应数据节点的数据内容发生变更
                   NodeChildChanged(4)   watcher监听的对应数据节点的子节点列表发生变更
 Disconnected(0)   None(-1)              客户端与zookeeper服务器断开连接
 Expired(-112)     Node(-1)              回话超时
 AuthFailed(4)     None(-1)              通常有两种情况,1:使用错误的schema进行权限检查,2:SASL权限检查失败


"watcher的特性: 一次性,客户端串行执行,轻量"
 一次性: 对于ZK的watcher,只需要记住一点,zookeeper有watch事件,是一次性触发的
        当watch监视的数据发生变化的时候,通知设置了该watcher的client,即watcher,由于zookeeper的监控都是一次性的所以,每次都必须设置监控
 客户端串行执行: 客户端watcher回调的过程是一个串行同步过程
        这为我们保证了顺序,同时需要开发人员注意一点,千万不要因为一个watcher的处理逻辑影响了整个客户端的watcher回调
 轻量: WatchedEvent是zookeeper整个watcher通知机制最小的通知单元。
       整个通知结构只包含三个部分:通知状态,事件类型和节点路径.也就是说watcher通知非现的简单,只会告诉客户端发生了事件而不会告知其具体内容
       需要客户自己去进行获取,比如NodeDataChanged事件,zookeeper只会通知客户端指定节点的数据发生了变更,而不直接提供具体的数据内容



```

##### 客户端注册 watch

```java
创建一个zk客户端的实例时可以像构造方法中传入一个默认的watcher
    public Zookeeper (String connectString,int sessionTimeout, Watcher watcher);
 这个watcher将作为这个zk回话期间的默认的watcher,会一直被保存在客户端的ZKWatchManager的defaultWatcher中。
 另外，ZooKeeper客户端也可以通过getData，getChildren和exist三个接口来向ZooKeeper服务器注册Watcher。

 以getData接口为例
  getData接口用户获取指定节点的数据内容,主要有两个方法:
    public byte[] getData(String path,boolean watch, Stat stat);
    public byte[] getData(final String path,Watcher watcher, Stat stat);

```

```java
public class TestWatch implements Watcher {
    private AtomicInteger atomicInteger = new AtomicInteger(0);
    public  CountDownLatch signal = new CountDownLatch(1);
    public  void init() {
        String hostport="192.168.10.106:2181";
        try {
            ZooKeeper zooKeeper = new ZooKeeper(hostport,10000,this);
            signal.await();
            zooKeeper.delete("/ssgao",-1);
            //zk监控节点都是一次性,即process只执行一次,
            // 这里即使节点"/ssgao"不存在,watch也是生效的,可以提前对节点进行监控
            // 节点还不存在的时候就可以对节点进行监控,当节点创建的时候,就能收到节点创建的事件
            zooKeeper.exists("/ssgao",true);
            //创建节点时,监听节点
            zooKeeper.create("/ssgao","ssgao".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            //更新数据,这时并不触发watch,因为setData之间没有对节点再次进行监控
            zooKeeper.setData("/ssgao","newData".getBytes(),-1);
            /**
             * 如果需要对/ssgao节点的更新事件,进行监控需要再次注册watch,如下
             */
            zooKeeper.getData("/ssgao",true,null);
            zooKeeper.setData("/ssgao","ssgao".getBytes(),-1);
            /**
             * 监控子节点的变换,注意事项
             *  1> 临时节点下面不可以创建子节点
             *  2> 子节点的创建,更新,删除,相对于父节点来说都是 "NodeChildrenChanged"
             */
            zooKeeper.getChildren("/ssgao",true);
            zooKeeper.create("/ssgao/a","a".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL);
            zooKeeper.getChildren("/ssgao",true);
            zooKeeper.setData("/ssgao/a","b".getBytes(),-1);
            zooKeeper.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public void process(WatchedEvent event) {
        System.out.println("进入process: "+event);
        System.out.println("第几次进入:"+atomicInteger.incrementAndGet());
        //连接状态
        Event.KeeperState state = event.getState();
        //事件类型
        Event.EventType eventType = event.getType();
        if(event.getState().equals(Event.KeeperState.SyncConnected)){
            if(eventType.equals(Event.EventType.None)){
                System.out.println("成功连接zk");
                signal.countDown();
            }else if(eventType.equals(Event.EventType.NodeCreated)){
                System.out.println("创建节点!");
            }else if(eventType.equals(Event.EventType.NodeDataChanged)){
                System.out.println("节点数据改变!");
            }else if(eventType.equals(Event.EventType.NodeChildrenChanged)){
                /**
                 * 这种情况无论,子节点是create,update,delete 都是触发父节点的NodeChildrenChanged
                 * 无法判断,子节点具体是什么操作
                 */
                System.out.println("子节点改变");
            }else if(eventType.equals(Event.EventType.NodeDeleted)){
                System.out.println("节点删除!");
            }
        }else if(state.equals(Event.KeeperState.Disconnected)){
            System.out.println("与服务器断开连接!");
        }else if(state.equals(Event.KeeperState.AuthFailed)){
            System.out.println("权限检车失败!");
        }else if(state.equals(Event.KeeperState.Expired)){
            System.out.println("回话失效!");
        }
        System.out.println("-----------------");
        System.out.println();
    }
     public static void main(String[] args) {
         TestWatch watch = new TestWatch();
         watch.init();
     }
}

```
