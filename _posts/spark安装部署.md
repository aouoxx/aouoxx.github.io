```
配置spark的JAVA_HOME的路径
export JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera

export SPARK_MASTER_IP=10.130.2.20 #1
export SPARK_MASTER_PORT=7077

export SPARK_MASTER_WEBUI_PORT=8089   配置master节点的UI端口
---------------------------------------- standalone模式下 ------------------------------------
export SPARK_WORKER_CORES=24  //每个Worker进程所需要的CPU核的数目, 默认为CPU的核数
export SPARK_WORKER_INSTANCES = 1 // 每台机器上运行的worker数量,默认为1
export SPARK_WORKER_MEMORY=48g //每个worker从节点能够支配的内存数,默认为1G
export SPARK_DAEMON_MEMROY=512M //分配给Spark master和worker守护进程的内存空间 512M

SPARK_WORKER_CORES*SPARK_WORKER_INSTANCES = 每台机器节点总cores
SPARK_WORKER_CORES*SPARK_WORKER_MEMORY = 每台机器节点可以使用的内存容量
-------------------------------------------------------------------------------------------

-----------------------------------on yarn模式----------------------------------------------

export SPARK_EXECUTOR_INSTANCES=1 // 在yarn集群中启动的worker的数目,默认为2个
SPARK_EXECUTOR_CORES //每个Worker所占用的CPU核的数目
SPARK_EXECUTOR_MEMORY //每个worker所占用的内存大小
SPARK_DRIVER_MEMORY Spark应用程序所占的内存大小,这里的Driver对应Yarn中的ApplicationMaster
SPARK_YARN_APP_NAME Spark Application在Yarn中的名字


-------------------------------------------------------------------------------------------

HADOOP_HOME=/opt/cloudera/parcels/CDH/lib/hadoop 当前节点HDFS的部署路径
HADOOP_CONF_DIR=/etc/hadoop/conf/ hdfs的conf配置文件路径,正常情况下此目录为$HADOOP_HOME/etc/hadoop



#export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=bdc40.hexun.com:2181,bdc41.hexun.com:2181,bdc46.hexun.com:2181,bdc53.hexun.com:2181,bdc54.hexun.com:2181 -Dspark.deploy.zookeeper.dir=/spark" #2
#export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=FILESYSTEM -Dspark.deploy.recoveryDirectory=/opt/modules/spark/recovery" #3

export JAVA_LIBRARY_PATH=$JAVA_LIBRARY_PATH:$HADOOP_HOME/lib/native
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HADOOP_HOME/lib/native
export SPARK_LIBRARY_PATH=$SPARK_LIBRARY_PATH:$HADOOP_HOME/lib/native
export SPARK_CLASSPATH=$SPARK_CLASSPATH:$HADOOP_HOME/lib/snappy-java-1.0.4.1.jar
```

### spark-defaults.conf

```
spark.local.dir /diskb/sparktmp  配置本地日志文件
spark.eventLog.enabled true
spark.eventLog.dir  hdfs://nameservice/spark-log  配置基于hdfs的历史日志文件存储
```

### 配置 historyserver

### standalone 模式高可用

```
-Dspark.deploy.recoveryMode=ZOOKEEPER
	说明整个集群状态是通过zookeeper来维护的,整个集群状态的恢复也是通过zookeeper来维护的。
  就是说用zookeeper做spark的HA配置,Master(Active)挂掉的话,Master(standby)要想编程Master(Active)的话,Master(Standby)就要向zookeeper读取整个集群状态信息,然后进行恢复所有worker和driver的状态信息和所有的Application状态信息。
-Dspark.deploy.zookeeper.url=hadoop1:2181,hadoop2:2181,hadoop3:2181
-Dspark.deploy.zookeeper.dir=/spark  保存spark的元数据,spark的作业状态

zookeeper会保存spark集群的所有状态信息,包括所有workers信息,所有Applications信息,所有Driver信息。




 export SPARK_DAEMON_JAVA_OPTS="
  -Dspark.deploy.recoveryMode=ZOOKEEPER
  -Dspark.deploy.zookeeper.url=zk1,zk2,zk3
  -Dspark.deploy.zookeeper.dir=/spark" zk上的路径
```

### kerberous

```
spark-submit
      --principal hdfs/hostname@jast.COM
      --keytab hdfs-hostname.keytab
      --jars $(echo lib/*.jar | tr ' ' ',')
      --class com.jast.test.Test test.jar

```

### 提交任务

```
一旦打包号,就可以使用bin/spark-submit脚本启动应用了。这个脚本负责设置spark使用的classpath和依赖,支持不同类型的管理器和发布模式

   bin/spark-submit
   			-- class <main-class>  应用的启动类
        -- master <master-url>  集群的master URL (如spark://23.195.26.18:7077)
        -- deploy-mode <deploy-mode>  是否发布驱动到worker节点(cluster) 或作为一个本地客户端(client:默认)
        -- conf <key>=<value> 任意的spark配置属性,格式key=value,如果值包含空格可以加引号"key=value"
        ... #other options
        <application-jar> \ 打包好的应用jar, 包含依赖。这个url在集群中全局可见。
        										比如hdfs:// ,file://
        [application-arguments] 传给main()方法的参数
 --executor-memory 1g
 --total-executor-cores 5


master url的格式
	local       本地以一个worker线程运行
  local[k]    本地以K worker线程(理想情况下,K设置为机器的CPU核数)
  local[*]    本地以本机同样核数的线程运行
  spark://host:port  连接到指定的spark standalone cluster master 端口是master集群配置的端口,默认为7077
  mesos://host:port
  yarn-client  以client模式连接到yarn cluster,集群的位置基于HADOOP_CONF_DIR变量找到
  yarn-cluster 以cluster模式连接到yarn cluster,集群的位置基于HADOOP_CONF_DIR变量找到
```

## 单机部署

```scala
安装scala



ssgao:conf aouo$ cat spark-env.sh
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home
export SCALA_HOME=/Users/ssgao/workspace/software_space/scala-2.13.3
export SPARK_MASTER_HOST=localhost
export SPARK_MASTER_IP=localhost
export SPARK_MASTER_PORT=7079

export SPARK_WORKER_CORES=1     # worknode给spark分的核数
export SPARK_WORKER_MEMORY=1G    #worknode给spark的内存


export SPARK_WORKER_INSTANCES=1   # worknode使用spark实例数


export SPARK_WORKER_PORT=8888    #指定spark运行是的端口

export HADOOP_HOME=/Users/ssgao/workspace/software_space/hadoop-2.9.2-1.0.0
export HADOOP_CONF_DIR=/Users/ssgao/workspace/software_space/hadoop-2.9.2-1.0.0/etc/hadoop
export SPARK_LOCAL_DIRS=/Users/ssgao/workspace/software_space/sf_data/spark/tmp
export SPARK_PID_DIR=/Users/ssgao/workspace/software_space/sf_data/spark/pid

命令启动
ssgao:sbin aouo$ ./start-all.sh
org.apache.spark.deploy.master.Master running as process 41542.  Stop it first.
Password:
localhost: starting org.apache.spark.deploy.worker.Worker, logging to /Users/ssgao/workspace/software_space/spark-3.0.1-bin-hadoop2.7/logs/spark-aouo-org.apache.spark.deploy.worker.Worker-1-ssgao.local.out
ssgao:sbin aouo$

查看进程
ssgao:sbin aouo$ jps
33345 KotlinCompileDaemon
41542 Master
30742 RemoteMavenServer36
41927 Jps
1482
41902 Worker

## 查看spark页面
http://localhost:8080/
```
