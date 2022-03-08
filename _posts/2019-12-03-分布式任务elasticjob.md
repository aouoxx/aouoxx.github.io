---
layout: post
title: 分布式任务elasticjob
categories: [分布式, elasticjob]
description: 分布式任务elasticjob
keywords: 分布式, elasticjob
---

<meta name="referrer" content="no-referrer"/>

### elasticjob 基本介绍

```java
传统任务的痛点
    使用quartz和spring-task
    1> 不敢轻易的跟着应用多节点部署,可能会重复多次执行,引发系统逻辑的错误
    2> quartz的集群仅仅只是用来HA,节点数量的增加并不能给我们每次执行效率代码提升,即不能实现水平扩展。、

ElasticJob 主要功能
    > 定时任务: 基于成熟的定时任务作业框架Quartz cron表达式执行定时任务
    > 作业注册中心: 基于zookeeper和其客户端Curator实现的全局作业注册控制中心.用于注册,控制和协调分布作业执行
    > 作业分片: 将一个任务分片成多个小任务项在多服务器上同时执行
    > 弹性扩容缩容: 运行中的作业服务器崩溃,或新增n台作业服务器,作业框架将在下次做特执行前重新分片,不影响当前作业执行
    > 支持多种作业执行模式
    > 失效转移: 运行中的作业服务器崩溃不会导致重新分片,只会在下次作业启动的时候分片.启用失效转移功能可以在本次作业执行过程中,检测其他作业服务器空闲,抓取未完成的孤儿分片项执行
    > 运行时状态收集: 监控作业运行时状态,统计最近一段时间处理的数据成功和失败数量,记录作业上次运行开始时间,结束时间和下次运行时间
    > 作业停止,恢复和禁用: 用于操作作业启停,并可以禁止某作业运行(上线时常用)

    > 被错过执行的作业重触发: 自动记录错过执行的作业,并在上次作业完成后自动触发.
    > 多线程快速处理数据: 使用多线程处理抓取到的数据,提升吞吐量
    > spring支持: 支持spring容器,自定义命名空间,支持占位符
    > 运维平台: 提供运维界面,可以管理作业和注册中心

目录结构说明
    elastic-job-core
    elastic-job核心模块，只通过Quartz和Curator就可执行分布式作业。
    elastic-job-spring
    elastic-job对spring支持的模块，包括命名空间，依赖注入，占位符等。
    elastic-job-console
    elastic-job web控制台，可将编译之后的war放入tomcat等servlet容器中使用。
    elastic-job-example
    elastic-job-test
    测试elastic-job使用的公用类，使用方无需关注。
```

```java
使用限制
    > 作业一旦启动成功就不能修改作业名称,如果修改名称则视为新的作业
    > 同一台作业服务器只能运行一个相同的作业实例,因为作业运行时是按照IP注册和管理的
```

#### elasticjob 的原理

```java
elastic-job-lite 原理
 一个典型的job场景,比如余额宝里的昨日收益,系统需要job在每天的某个时间点开始,给所有余额宝用户计算收益。
 如果用户数量不多,我们可以轻易使用quartz来完成,我们让计息job在某个时间点开始执行,循环遍历所有用户计算利息,这没问题。
 可是,如果用户体量特别大,我们可能会面临着在第二天之前处理不完这么多用户.另外,我们部署job的时候也得注意,我们可能会把job直接放在我们webapp里。
 webapp通常是多个节点部署,这样我们的job也就是多个节点,多个job同时执行,很容易造成重复执行,比如用户重复计息,为了避免这种情况,我们可能会对job执行加锁。
 加锁来保证始终只有一个节点能执行,或者干脆让job从webapp里剥离出来,独自部署一个节点。

 elastic-job就可能帮组我们解决上面的问题,elastic底层的任务调度还是使用quartz,通过zookeerper来动态给job节点分片.
 我们来看,很大体量的用户需要在特定时间段内计息完成,我们肯定是希望我们的任务可以通过集群水平扩展,集群里的每个节点都处理部分用户,
 不管用户数量有多庞大,我们只要增加机器就可以了,比如单台机器热定时间能处理n个用户,2台机器处理2n个用户,3台3n,4台4n,再多的用户也不怕了。

 使用elastic-job开发的作业都是zookeeper的客户端,比如我们希望三台机器跑job,我们将任务分成3片,框架通过zk的协调,最终会让3台机器分别
 分配到0,1,2的任务片,比如
     server0->0
     server1->1
     server2->2
 当server0执行时,可以只查询id%3==0的用户,server1执行时只查询id%3==1的用户,server2执行时只查询到id%3==2的用户


 任务部署多节点
   > 在上面的基础上,我们再增加server3,此时,server3分不到任务分片,因为只有3片,已经分完了。没有分到任务的分片作业程序将不执行.
   > 如果此时server2挂了,那么server2的分片项会分配为server3,server3有了分片,就会替代server2执行.
   > 如果此时server3也挂了,只剩下server0和server1了,框架也会自动把server3的分片随机分配为server0或者server1,可能会server0->0;server1->1,2
   > 这种特定称为弹性扩容,即elastic-job名称的由来.




```

#### elasticjob 作业类型

```java
作业类型
    elastic-job 提供了三种类型的作业,simple类型作业,Dataflow类型作业,script类型作业.
    其中DataFlow类型用于处理数据流,它又提供2种作业类型,分别是"ThroughDataFlow"和"SequenceDataFlow".需要继承相应的抽象类

    方法参数'shardingContext'包含作业配置,分片和运行时信息.
    可以通过'getShardingTotalCount()','getShardingItems()'等方法分别获取分片总数,运行在本作业服务器的分片序列号集合等。


>>SimpleJob
    simpleJob需要实现SimpleJob接口,意为简单实现,为经过任何封装,与quartz原生接口相似.
    需要继承'AbstractSimpleElasticJob',该类只提供一个方法用于覆盖,此方法将被定时执行.用于执行普通定时任务。
    与Quartz原生接口相似,只是增加了弹性扩缩容和分片等功能
    public class MyElasticJob extends AbstractSimpleElasticJob{
        @Override
        public void process(JobExecutionMultipleShardingContext context){
            //do something by sharding items;
        }
    }

>>Dataflow
    dataFolw类型用于处理数据流,需要实现DataflowJob接口,该接口提供了2个方法可供覆盖,分别用于抓取(fetchData)和处理(processData)数据
    可以通过"DataFlowJobConfiguration"配置是否流式处理
    流式处理数据只有fetchData方法的返回值为null或集合长度为空时,作业才停止抓取,否则作业将一直运行下去.
    非流失处理数据则只会在每次作业执行过程中执行一次fetchData方法和processData方法,随机完成本次作业。

    SequenceDataFlow类型作业和ThroughputDataFlow作业类型极为相似,所不同的是ThroughDataFlow做业务类型可以将获取的数据多线程处理,但不会保证多线程处理数据的顺序。
    如:从2个分片共获取100条数据,第一个分片40条,第二个分片60条,配置为两个线程处理. 则第一个线程处理前50条数据,第二个线程处理后50条数据,无视分片项;

    SequenceDataFlow类型作业则根据当前服务器所分配的分片项数量进行多线程处理,每个分片项使用同一个线程处理,防止同一个分片的数据被多线程处理
    从而导致的顺序问题.
    如:从2个分片共获取到100条数据,第一个分片40条,第二个分片60条,则系统自动分配两个线程处理.
    第一个线程处理第一个分片的40条数据,第二个线程处理第二个分片的60条数据.
    由于ThroughputDataFlow作业可以使用对于分片项的任意线程数处理,所以性能调优的可能会优于SequenceDataFlow作业.



```

#### elasticjob 的使用限制

```java
使用限制
    > 作业一旦启动成功后不能修改作业名称,如果修改名称则视为新的作业
    > 同一台作业服务器只能运行一个相同的作业实例,因为作业运行的时候按照IP注册和管理的
    > 作业根据/etc/hosts文件获取Ip地址,如果获取的ip地址为127.0.0.1而非真实ip地址,应正确配置此文件

    > 一旦有服务器波动,或者修改分片项,将会触发重新分片;
       触发重新分片将会导致运行中的Perpetual以及SequencePerpetual作业再执行完本次作业后不再继续执行,等待分片结束后再恢复正常.
    > elastic-job没有自动删除作业服务器的功能,因为无法区分是服务器崩溃还是正常下线.
       所以如果要下线服务器,需要手工删除zookeeper中相关的服务器节点.
       由于直接删除服务器节点风险较大,暂时不考虑在运维平台增加此功能


```

### elasticjob 的基本配置

```java
job:simple
    id                         作业名称
    class                      作业实现类,实现ElasticJob接口
    registry-center-ref        注册中心Bean的引用,需应用reg:zookeeper的声明
    cron                       cron表达式,用于配置作业触发时间
    sharding-total-count       作业分片总数
    sharding-item-parameters   分片序列号和参数用等号分隔,多个键值对用逗号分隔,分片序列号从0开始,不可大于或等于作业分片总数,如0=a,1=b,2=c
    job-parameter              作业自定义参数,可以配置多个相同的作业,单用不同的调度实例


    failover                   是否开启失效转移,仅moniterExecution开启,实现转移才有效
    misfire                    是否开启错误任务重新执行
    job-sharding-strategy-class 作业分片策略实现类全路径,默认使用平均分配策略
    description                作业描述信息
    disabled                   作业是否禁止启动,可用于部署时先禁止启动,部署结束后统一启动
    overwrite                   本地配置是否可覆盖注册中心配置,如果可覆盖,每次启动作业都以本地配置为准

```

### elasticjob 的基本使用

```java
    // 定义作业核心配置
    JobCoreConfiguration simpleCoreConfig = JobCoreConfiguration.newBuilder("demoSimpleJob", "0/15 * * * * ?", 10).build();
    // 定义SIMPLE类型配置
    SimpleJobConfiguration simpleJobConfig = new SimpleJobConfiguration(simpleCoreConfig, SimpleDemoJob.class.getCanonicalName());
    // 定义Lite作业根配置
    JobRootConfiguration simpleJobRootConfig = LiteJobConfiguration.newBuilder(simpleJobConfig).build();

    // 定义作业核心配置
    JobCoreConfiguration dataflowCoreConfig = JobCoreConfiguration.newBuilder("demoDataflowJob", "0/30 * * * * ?", 10).build();
    // 定义DATAFLOW类型配置
    DataflowJobConfiguration dataflowJobConfig = new DataflowJobConfiguration(dataflowCoreConfig, DataflowDemoJob.class.getCanonicalName(), true);
    // 定义Lite作业根配置
    JobRootConfiguration dataflowJobRootConfig = LiteJobConfiguration.newBuilder(dataflowJobConfig).build();

    // 定义作业核心配置配置
    JobCoreConfiguration scriptCoreConfig = JobCoreConfiguration.newBuilder("demoScriptJob", "0/45 * * * * ?", 10).build();
    // 定义SCRIPT类型配置
    ScriptJobConfiguration scriptJobConfig = new ScriptJobConfiguration(scriptCoreConfig, "test.sh");
    // 定义Lite作业根配置
    JobRootConfiguration scriptJobRootConfig = LiteJobConfiguration.newBuilder(scriptCoreConfig).build();
```

```java
<dependency>
    <groupId>com.dangdang</groupId>
    <artifactId>elastic-job-lite-core</artifactId>
    <version>2.0.0</version>
</dependency>

/**
 * 可以通过DataflowJobConfiguratuon配置是否流式处理
 *  流式处理数据只有fetchData方法的返回值为null或集合长度为空时,作业才停止抓取,否则作业将一直运行下去
 *  非流式处理数据则只会在每次作业执行过程中执行一次fetchData方法和processData方法,随机完成本次作业
 *
 *  如果采用流式作业处理方式，建议processData处理数据后更新其状态，避免fetchData再次抓取到，从而使得作业永不停止。
 *  流式数据处理参照TbSchedule设计，适用于不间歇的数据处理。
 */
public class ApiJobDataFlow {
    public static void main(String[] args) {
        new JobScheduler(registryCenter(),configuration()).init();
    }
    private static CoordinatorRegistryCenter registryCenter(){
        /**
         * 配置zookeeper
         * <配置注册中心,任务的信息都会在zk中存储>
         * <reg:zookeeper id="regCenter" server-lists="192.168.56.102:2181"
         *  namespace="test-job" base-sleep-time-milliseconds="1000" max-sleep-time-millseconds="3000" max-retries="4" />
         *
         * id用于给该注册中心命名。
         * server-lists用于指定使用的zookeeper的地址，多个地址之间用英文的逗号分隔。
         * namespace用于指定注册中心在zookeeper中的命名空间，属于zookeeper的概念。
         * base-sleep-time-milliseconds用于指定等待重试的间隔时间的初始值，单位是毫秒。
         * max-sleep-time-milliseconds用于指定等待重试的间隔时间的最大值，单位是毫秒。
         * max-retries用于指定最大的重试次数。
         */
        CoordinatorRegistryCenter registryCenter = new ZookeeperRegistryCenter( new ZookeeperConfiguration("192.168.56.102:2181","ssgao-job", 1000, 3000,4));
        registryCenter.init();
        return registryCenter;
    }
    private static LiteJobConfiguration configuration(){
        //定义作业核心配置
        String shardingItemParameter = "0:shandong,1=hangzhou";
        JobCoreConfiguration dataflowCoreConfig = JobCoreConfiguration
                .newBuilder("dataFlowJob","0/20 * * * * ?",3)
                .build();
        //定义DATAFLOW类型配置
        DataflowJobConfiguration dataflowJobConfiguration
                = new DataflowJobConfiguration(dataflowCoreConfig,ApiJobDataFlow.class.getCanonicalName(),true);
        //定义Lite作业根配置
        String jobSharedStrategyClass = null;
        LiteJobConfiguration dataflowJobRootConfiguration = LiteJobConfiguration.
                    newBuilder(dataflowJobConfiguration).
                    jobShardingStrategyClass(jobSharedStrategyClass).build();
        return dataflowJobRootConfiguration;
    }
}

```

#### 流式作业

```java
https://blog.csdn.net/elim168/article/details/78938300

/** 数据流分布式作业接口 */
public interface DataflowJob<T> extends ElasticJob{
    /**
     * 获取待处理数据
     * @param shardingContext 分片上下文
     * @return 待处理的数据集合
     */
    List<T> fetchData(ShardingContext shardingContext);
    /**
     * 处理数据
     * @param shardingContext分片上下文
     * @param data待处理数据集合
     */
    void processData(ShardingContext shardingContext,List<T> data);
}

    流式作业,每次调度触发的时候会先调用fetchData获取数据,如果获取到了数据再调度processData方法处理数据。
DataflowJob在运行时有两种方式,流式和非流式,通过属性streamingProcess控制,如果是基于spring xml的配置方式,则通过配置属性'streaming-process'(boolean 类型)
    当作业配置为流式的时候,每次触发作业后调度一次fetchData获取数据,如果获取到数据会调度processData方法处理数据,处理完后有继续调fetchData获取数据,在调processData处
理.如此循环,像流水一样.一直到fetchData没有获取到数据或者发生了重新分片才会停止.
    具体的实现可以参考com.dangdang.ddframe.job.executor.type.DataflowJobExecutor.下面是DataflowJob的一个简单实现,该实现中每次调度触发都会连续调度processData十次

public class MyDataflowJob implements DataflowJob<String>{
    private static final TreadLocal<Integer> LOOP_COUNTER = new ThreadLocal<>();
    private static final int LOOP_TIMES =10; //每次获取流处理循环次数
    private static final AtomicInteger COUNTER = new AtomicInteger(1); //计数

    public List<String> fetchData(ShardingContext shardingContext){
        Integer current = LOOP_COUNTER.get();
        if(current==null){
            current=1;
        }else{
            current +=1;
        }

        LOOP_COUNTER.set(current);
        System.out.println(Thread.currentThread()+"----current----"+current);
        if(current>LOOP_TIMES){
            System.out.println("\n\n\n");
            return null;
        }else{
            int shardingItem = shardingContext.getShardingItem();
            List<String> datas = Arrays.asList(getData(shardingItem),getData(shardingItem),getData(shardingItem));
            return datas;
        }
    }

    private String getData(int shardingItem){
        return shardingItem + "_"+COUNTER.getAndIncrement();
    }

    @Override
    public void processData(ShardingContext shardingContext,List<String> data){
        System.out.println(Thread.currentThread()+"------"+data);
    }

}

spring中的对应的配置
<job:dataflow id="myDataflowJob" class="com.aouo.MyDataflowJob" registry-center-ref="regCenter"
     cron="0 0/2 * * * ?" sharding-total-count="2" sharding-item-paramters="0=hangzhou,1=shanghai"
    failover="true" overwrite="true" streaming-process="true" />
```

### elasticjob 的运维管理

```java
<dependency>
    <groupId>com.cxytiandi</groupId>
    <artifactId>elastic-job-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>

运维平台提供两种接口,管理员及访客,管理员拥有全部操作权限,访客仅拥有查看权限.
默认管理员用户名和密码是root/root,访客用户名和密码是guest/guest,
    可通过conf/auth.properties修改管理员以及访客用户名以及密码.


```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639275339357-a664bc46-9c31-4d3f-a33d-81bf2eb059af.png#clientId=ub891aa2b-dbf6-4&from=paste&height=405&id=uacef91f3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=810&originWidth=1914&originalType=binary&ratio=1&size=511330&status=done&style=none&taskId=u1ccb24a7-1b05-45a2-b38a-225dc811e89&width=957)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639275363871-10c478d6-636e-40cc-8dea-16b16856d01c.png#clientId=ub891aa2b-dbf6-4&from=paste&height=423&id=u355bf91c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=846&originWidth=1926&originalType=binary&ratio=1&size=423443&status=done&style=none&taskId=u3686845a-b8de-43ef-8f48-fec7cac6e85&width=963)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639275383859-13571931-bf40-43a5-b1b0-3d61f7ed20bf.png#clientId=ub891aa2b-dbf6-4&from=paste&height=342&id=uac6355e7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=1908&originalType=binary&ratio=1&size=550527&status=done&style=none&taskId=u73754bff-0d73-493c-8bba-a7a8821c6a0&width=954)
