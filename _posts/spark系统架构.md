### spark 术语

```
应用程序 Application
    基于spark的用户程序,包含了一个Driver Program和集群中的多个executor。
驱动 Driver
		运行Application的main()函数并且创建sparkContext
执行单元 Executor
	  为某Application运行在worker node上的一个进程,该进程独立运行task,并且负责将数据存在内存或者磁盘上,每个Application都有各自独立的Executors。

集群管理程序(Cluster Manager)
		在集群上获取资源的外部服务(例如 Local, Standalone, Mesos或Yarn等集群管理系统)
操作(Operation)
		作用于RDD的各种操作,分为transformation 和 action



```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/659846/1604307755333-1ac67e3d-cfdd-4b01-973d-c2cb0602bc0e.png#align=left&display=inline&height=180&margin=%5Bobject%20Object%5D&name=image.png&originHeight=221&originWidth=501&size=15174&status=done&style=none&width=407)

```
Driver部分
	Driver部分主要对SparkContext进行配置,初始化以及关闭。初始化sparkContext是为了构建Spark应用程序的运行环境，在初始化SparkContext,要先导入一些Spark类和隐式转换; 在Executor部分运行完毕后,需要将SparkContext关闭

Executor部分
	Spark
```

#### spark 各种概念之间的关系

```
在spark中,一个应用(Application)由一个任务控制节点(Driver)和若干个作业(Job)构成,一个作业由多个阶段(Stage)构成,一个阶段由多个Task组成。当执行一个应用时,任务控制节点会向集群管理器(Cluster Manager)申请资源,启动Executor, 并向Executor发送应用程序代码和文件。然后在Executor上执行任务,运行结束后,执行结果会返回给任务控制节点,或者写到HDFS或者其他数据库中。

```

#### jobs

```
spark job处于spark执行层结构中的最高层,每个spark job对应一个action, 每个action被spark应用中的驱动程序调用。
可以把action理解成把数据从RDD的数据带到其他存储系统的组件(通常是带到驱动程序所在的位置或者写到稳定的存储系统中)。
>>> 主要一个action被调用,spark就不会再向这个job增加新的东西
```

#### stages

```sql
RDD转换是懒加载,直到调用一个action才开始执行RDD转换。
一个job是由一个action来定义的,一个action可能会包含一个或多个转换(transformation) spark根据宽依赖把job分解成stage。
从整体看,一个stage是"计算task"的集合,这些task在各自的Executor中进行运算,而不需要同其他的执行器或者驱动进行网络通讯。换句话说,当任何两个workers之间开始需要网络通讯的时候,这时候一个新的stage就产生了，例如shuffle的时候。
```

#### Tasks

```sql
stage有tasks组成。在执行层集中,task是最小执行单位。每一个task表现为一个本地计算。
一个stage中的所有tasks会对不同的数据执行相同的代码。(程序代码一样,只是作用在不同的数据上)

一个task不能被多个执行器来执行,但是,每个执行器会动态的分配多个slots来执行tasks, 并且在整个生命周期内会并行的运行多个task。每个stage的task数量对应着分区的数量,即每个Partition都被分配一个Task
```

### 运行原理

```
	  使用spark-submit提交一个spark作业之后,这个作业就会启动一个对应的Driver进程。
    根据我们使用的部署模式(deploy-mode)不同, Driver进程可能在本地启动,也可能在集群中某个工作节点启动。而Driver进程要做的第一件事情,就是向集群管理器(yarn)申请运行spark作业需要的使用的资源, 这里的资源指的是Executor进程。yarn集群管理器会根据我们为spark作业设置的资源参数,在各个工作节点上,启动一定数量的executor进程。每个executor进程都占用一定数量的内存的和CPU core。

    在申请到了作业执行所需要的资源之后,Driver进程就会开始调度和执行我们编写的作业代码了。Driver进程会将我们编写的spark作业拆分为多个stage。每个stage执行一部分代码片段,并为每个stage创建一批task,然后将这些task分配到各个executor进程中进行执行。task是最小的计算单元,负责执行一模一样的计算逻辑(也就是我们自己编写的某个代码片段),只是每个task处理的数据不同而已。一个stage的所有task都执行完毕之后,会在各个节点本地的磁盘文件中写入计算中间结果,然后Driver就会调度运行写一个stage。下一个stage的task的输入数据就是上一个stage输出的中间结果。如此循环往复,直到将我们自己编写的代码逻辑全部执行完,并且计算完所有的数据，得到我们想要的结果为止。

    spark根据shuffle类算子,来进行stage划分。如果代码中执行了某个shuffle类算子(例如reduceByKey,join等),那么就会在该算子处,划分出一个stage界限来。可以大致理解为,shuffle算子执行之前的代码会被划分为一个stage, shuffle算子执行以及之后的代码会被划分为下一个stage。因此一个stage刚开始执行的时候,它的每个task可能都会从上一个stage的task所在的节点,去通过网络传输拉取需要自己处理的所有key,然后对拉取的所有相同的key使用我们自己编写的算子函数执行聚合操作(比如 reduceByKey()算子接收的函数)。这个过程就是shuffle.

   当我们在代码中执行了cache/persist等持久化操作时,根据我们选择的持久化级别不同,每个task计算出来的数据也会保存到executor进程的内存所在节点磁盘文件中。
   Executor的内存主要分为三块:
   	1) 让Task执行我们自己编写的代码时使用,默认占Executor总内存的20%
    2) 让Task通过shuffle过程拉取上一个stage的Task的输出后,进行聚合等操作时使用,默认也是占Executor的20%
    3) 让RDD持久化时使用,默认占Executor总内存的60%

	Task的执行速度跟每个Executor进程的CPU core数量有直接的关系。一个CPU core同一时间只能执行一个线程。而每个Executor进程上分配到多个task, 都是以每个task一条线程的方式, 多线程并发运行的。如果CPU core数量比较充足,而且分配到的task数量比较合理,那么通常来说,可以比较快速和高效的执行完这些Task线程

```

### sparkonyarn 的运行

```scala
driver 运行在集群中(cluster模式)
	1) client向yarn提交一个job
  2) ResourceManager为该job在某个NodeManager上分配一个ApplicationMaster,NM启动ApplicationMaster,ApplicationMaster启动driver（sparkContext）
  3）ApplicationMaster启动后完成初始化作业,driver生成DAG图, DAG scheduler将DAG拆分成stage(TaskSet) 发送给Task Scheduler
	4) AM向RM申请资源,RM返回Executor信息
  5) AM通过rpc启动相应的Executor
  6) Exector向Driver申请task, TaskScheduler将Task发送给Executor
  7) Exector执行task,执行结果写入外部或返回driver端


driver 运行在client端
	1、客户端启动后直接运行应用程序，直接启动 driver，并初始化
	2、客户端将 job 发布到 yarn 上
	3、RM 为该job 在某个 NM 分配一个 AM
	4、AM 向 RM 申请资源，RM 返回Executor 信息
	5、AM 通过 RPC 启动相应的 Executor
	6、driver 生成DAG 图，DAG scheduler 将DAG 拆分成 stage （TaskSet）发送给 Task Scheduler
	7、Driver 向 Executor 分配 task
	8、Executor 执行task 并将结果写入第三方存储系统或者 Driver 端

```

### RDD

```scala
RDD (Resilient Distributed Dataset) 是一个弹性, 可复原的分布式数据集,是spark中最基本的抽象是一个不可变的, 有多个分区的, 并可以并行计算的集合。RDD中并不装真正要计算的数据,而装的是描述信息, 描述以后从哪里读取数据,调用了什么方法,传入了什么函数,以及依赖关系等。
```

#### RDD 特点

```scala
有一些列连续的分区: 分区编号从0开始,分区的数量决定了对应阶段Task的并行度
有一个函数作用在每个输入切片上: 每一个分区都会生成一个Task,对该分区的数据进行计算,这个函数就是具体的计算逻辑

RDD和RDD之间存在一些列依赖关系, RDD调用Transformation后会生成一个新的RDD,子RDD会记录父RDD的依赖关系,包含宽依赖（有shuffle)和窄依赖(没有shuffle)
		K-V的RDD在Shuffle会有分区器,默认使用HashPartitioner
    如果从HDFS中读取数据,会有一个最优位置,spark在调度任务之前会读取NameNode的元数据,获取数据的位置,移动计算而不是移动数据,这样可以提高计算效率
```

### 宽窄依赖

```
RDD 只支持粗粒度转换,即在大量记录上执行单个操作。
将创建RDD的一系列(血统)记录下来,以便恢复丢失的分区。RDD的血缘会记录RDD的元数据信息和转换行为。
当RDD的部分分区数据丢失时,它可以根据这些信息来重新运算和恢复丢失的数据分区。

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609671140374-7c51a686-9254-4ecf-a05b-e3eab0e7cb8e.png#align=left&display=inline&height=56&margin=%5Bobject%20Object%5D&name=image.png&originHeight=76&originWidth=552&size=8455&status=done&style=none&width=408)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609673594862-11e63db7-9129-442d-b4d0-87346d6b150c.png#align=left&display=inline&height=387&margin=%5Bobject%20Object%5D&name=image.png&originHeight=451&originWidth=551&size=34868&status=done&style=none&width=473)

### stage 划分

```
shuffleMapStage 和 ResultStage

简单来说 DAG的最后一个阶段会为每个结果的partition生成一个ResultTask, 即每个Stage里面的Task的数量是由该Stage中最后一个RDD的Partition的数量所决定的。

而其余所有阶段都会生成ShuffleMapTask,之所以成为shuffleMapTask是因为它需要将自己的计算结果通过shuffle到下一个stage中
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609674703584-1e3a477a-e90f-4c85-8063-35c63987e422.png#align=left&display=inline&height=305&margin=%5Bobject%20Object%5D&name=image.png&originHeight=326&originWidth=591&size=37587&status=done&style=none&width=553)

### 任务(job)划分

### RDD 持久化

```
RDD 通过cache或persist方法将前面的计算结果缓存,默认情况下,会把数据缓存在JVM的堆内存中。
但是并不是这两个方法被调用时立即缓存,而是触发后面的action算子时,该RDD将会被缓存在计算节点的内存中,并供后面重用。

// cache操作会增加血缘关系,不改变原有的血缘关系
println(wordToOneRdd.toDebugString)
// 数据缓存
wordToOneRdd.cache()
// 可以更改存储级别
mapRDD.persist(StorageLevel.MEMORY_AND_DISK_2)

```

### 检查点

```scala

object SparkCheckPointDemo {
  def main(args: Array[String]): Unit = {
    val sc = SparkContextUtil.getSparkContext(null)
    sc.setCheckpointDir("checkpoint")
    val scRDD = sc.makeRDD(List("a","b","c","d"))
    val mapRDD :RDD[(String, Int)] = scRDD.map(t=>t->1)
    /**
     * checkpoint 需要落盘,需要指定检查点保存路径
     * 检查点路径保存的文件,当作业执行完毕后, 不会被删除
     * 一般保存路径在分布式存储系统中如: HDFS
     */
    mapRDD.cache()
    mapRDD.checkpoint();
    val reduceRDD:RDD[(String, Int)] = mapRDD.reduceByKey(_+_);
    reduceRDD.collect().foreach(println(_))
    println("---------------------------------------")
    val groupRDD:RDD[(String, Iterable[Int])] = mapRDD.groupByKey();
    groupRDD.collect().foreach(println(_))
    sc.stop()
  }
}
```

#### 检查点&持久化的区别

```scala
cache
	将数据临时存储在内存中进行数据重用
  会在血缘关系中添加新的依赖。一旦,出现问题,可以从头读取数据

persist
	将数据临时存储在磁盘文件中进行数据重用, 涉及到磁盘IO,性能较低,但是数据安全
  如果作业执行完毕,临时保存的数据文件就会丢失

checkpoint
	将数据长久保存在磁盘文件中进行数据重用
  涉及到磁盘IO,性能较低,但是数据安全
  为了保证数据安全,所以一般情况下,会独立执行作业
  为了能够提高效率,一般情况下,是需要和cache联合使用
  执行过程中,会切断血缘关系。重新建立新的血缘关系。
  ""checkpoint等同于改变了数据源""

```

### 分区器

```scala
spark目前支持hash分区和range分区,以及用户自定义分区。
hash分区为当前的默认分区。分区器直接决定了RDD中分区的个数,RDD中每条数据经过shuffle后进入那个分区,进而决定了Reduce的个数。
 > 只有key-value类型的RDD才有分区器, 非key-value类型的RDD分区的值是None
 > 每个RDD的分区ID范围: (0~(numPartitions-1)) 决定这个值是属于哪个分区的。

```

### RDD 文件读取与保存

```scala
  spark的数据读取及数据保存可以从两个维度来做区分: 文件格式以及文件系统。
  文件格式分为,text文件, csv文件, sequence文件以及Object文件
  文件系统分为: 本地文件系统,hdfs,hbase以及数据库

text文件
	读取输入文件:val inputRDD: RDD[String] = sc.textFile("input/1.txt")
  保存数据: inputRDD.saveAsTextFile("output")
sequence文件
	SequenceFile文件是Hadoop用来存储二进制形式的key-value对,而设计的一种平面文件(Flat File)
  保存数据为SequenceFile: dataRDD.saveAsSequenceFile("output")
  读取sequenceFile文件: sc.sequenceFile[Int,Int]("output").collect().foreach(println(_))

Object对象文件
	对象文件是将对象序列化后保存的文件,采用Java的序列化机制。
  可以通过objectFile[T:classTag](path)函数接收一个路径,读取对象文件,返回对应的RDD。也可以通过调用saveAsObjectFile实现对对象文件的输出。序列化要指定类型
  保存数据: dataRDD.saveAsObjectFile("output")
  读取数据: sc.objectFile[Int]("output").collect().foreach(println(_))



object SparkIODemo {
  def main(args: Array[String]): Unit = {
    val sc = SparkContextUtil.getSparkContext(null);
    val scRDD = sc.makeRDD(List(("a",1),("a",2),("b",3),("b",4),("b",5),("a",6)),2)

    scRDD.saveAsTextFile("output")
    scRDD.saveAsObjectFile("objFile")
    scRDD.saveAsSequenceFile("seqFile")

    sc.textFile("output")
    sc.objectFile("objFile")
    sc.sequenceFile("seqFile")
    sc.stop;
  }
}
```

### 累加器

```scala
累加器用来把Executor端变量信息聚合到Driver端。
在Driver程序中定义的变量,在Exector端的每个Task都会得到这个变量一个新的副本,每个task更新这些副本的值后传回Driver端进行merge
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609644111331-f1bbcf1e-84b4-4e75-a8c2-44c7f6c9aca9.png#align=left&display=inline&height=124&margin=%5Bobject%20Object%5D&name=image.png&originHeight=147&originWidth=674&size=20113&status=done&style=none&width=570)

```scala

object SparkAccDemo {
  def main(args: Array[String]): Unit = {
    val sc = SparkContextUtil.getSparkContext(null);
    val scRDD = sc.makeRDD(List(1,2,3,4))
    // reduce 分区内计算,分区间计算
    //val res = scRDD.reduce(_+_);
    //println(res)
    var sum=0;
    scRDD.foreach(num=>{
      sum += num
    })
    println(sum) // 没有使用累加输出为0
    /**
     * 获取系统累加器
     * spark默认提供了简单的数据聚合累加器
     */
    val sumAcc = sc.longAccumulator("SUM")
    scRDD.foreach(num=>{
      // 使用累加器
      sumAcc.add(num)
    })
    println(sumAcc.value) // 没有使用累加输出为0
  }
}
```

```scala

object SparkAccDemob {
  def main(args: Array[String]): Unit = {
    val sc = SparkContextUtil.getSparkContext(null);
    val scRDD = sc.makeRDD(List(1,2,3,4))

    val sumAcc = sc.longAccumulator("SUM")
    val mapRDD = scRDD.map(num=>{
      sumAcc.add(num)
      num
    })

    /**
     * 获取累加器的值
     * 少加: 转换算子调用累加器,如果没有行动算子,那么不执行
     */
    println(sumAcc.value)
    /** 多加,mapRDD执行了两次, 可以通过mapRDD.cache()避免 */
    mapRDD.collect().foreach(println(_))
    mapRDD.collect().foreach(println(_))
    /** 一般情况下累加器放置在行动算子中进行操作*/
  }
}
```

#### 自定义累加器

```scala
/**
 * 自定义累加器
 * 继承: AccumulatorV2, 定义泛型
 *  IN: 累加器输入的数据类型
 *  OUT: 累加器返回的数据类型
 *
 * 重写方法
 */
class MyAccumulator extends AccumulatorV2[String,mutable.Map[String,Long]] {

  private var myMap = mutable.Map[String,Long]()

  /**
   * isZero判断是否为初始状态
   */
  override def isZero: Boolean = {
    myMap.isEmpty
  }
  // 复制一个新的累加器
  override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] = new MyAccumulator
  // 重置累加器
  override def reset(): Unit = myMap.clear()

  /**
   * 获取累加器需要计算的值
   * @param word
   */
  override def add(word: String): Unit = {
     val newCnt = myMap.getOrElse(word,0L) + 1
     myMap.update(word,newCnt)
  }

  /**
   * Driver 合并多个累加器
   * @param other
   */
  override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]): Unit = {
    val map1 = this.myMap;
    val map2 = other.value;
    map2.foreach{
      case(word,count)=>{
        val newCount = map1.getOrElse(word,0L)+count;
        map2.update(word,newCount)
      }
    }
  }

  /**
   * 获取累加器的结果
   * @return
   */
  override def value: mutable.Map[String, Long] = {
    myMap
  }
}
```

```scala
object SparkMyAccumulator {

  def main(args: Array[String]): Unit = {
    val sc = SparkContextUtil.getSparkContext(null);
    val scRDD = sc.makeRDD(List("hello","hi","hi","hi"))

    val myAcc = new MyAccumulator;
    /** 向spark 进行注册自定义累加器 */
    sc.register(myAcc,"myAcc")
    scRDD.foreach(t=>{
      // 数据的累加(使用累加器)
      myAcc.add(t)
    })
    // 获取累加器的结果
    println(myAcc.value)
  }
}
```

### 广播变量

```scala
广播变量用来高效分发较大的对象。
向所有工作节点发送一个较大的只读值,以供一个或多个spark操作使用。
比如,如果你的应用需要向所有节点发送一个较大的只读查询表,广播变量用起来都很顺手。
在多个并行操作中使用同一个变量,但是spark会为每个任务分别发送。



```
