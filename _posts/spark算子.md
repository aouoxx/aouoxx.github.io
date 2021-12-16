spark 中的算子的介绍

```scala
rdd 的计算一个分区内的数据是一个一个执行逻辑
	只有前面的一个数据全部执行完毕后,才会执行下一个数据
  分区内数据的执行是有序的

不同分区数据计算是无须的。
```

### 创建 RDD

#### 并行度

```scala
// parallel 设置并行度度
    val scRDD = sc.parallelize(List(1,2,3,4),2);
    val dataRDD = sc.makeRDD(List(1,3,4,1,2,3,4),2);

 def makeRDD[T: ClassTag](
      seq: Seq[T],
      numSlices: Int = defaultParallelism): RDD[T] = withScope {
    parallelize(seq, numSlices)
  }
```

#### 创建 RDD 和分区执行指定

```scala
/**  RDD的并行度&&分区
 * makeRDD方法可以传递第二个参数,这个参数表示分区的数量
 * 第二个参数可以不传递的,那么makeRDD方法会使用默认值.
 *   scheduler.cong.getInt("spark.default.parallelism",totalCores)
 * spark在默认情况下,从配置对象获取配置参数: spark.default.parallelism
 * 如果获取不到,那么使用totalCores属性,取值为当前运行环境的最大可用core数
 */

val dataRDD = sc.makeRDD(List(1,3,4,1,2,3,4)


/**
 * path路径可以是文件的具体路径,也可以是目录名称
 * textFile可以将文件作为数据处理的数据源,默认也可以设定分区
 *       minPartitions: 最小分区数量
 *       math.min(defaultParallelism,2)
 *
 * 如果不想使用默认的分区数量,可以通过第二个参数指定分区数
 * spark读取文件,底层其实使用就是hadoop的读取方式
 * 分区数量的计算方式
 *   totalSize = 7 (字节数,包含换行)
 *   goalSize = 7/2 = 3（byte)
 *   7/3=2...1(1.1) + 1=3(分区)
 */
val scRDD = sc.textFile("/user/ssgao/text.txt");

textFile 以行为单位来读取数据
wholeTextFiles 以文件为单位读取数据
val rdd = sc.wholeTextFile("datas")
rdd.collect().foreach(println)


```

#### 文件数据源分区分配

```scala
1) 数据以行尾单位进行读取
	spark读取文件,采用的是以hadoop的方式读取,所以一行一行读取,和字节数没有关系
2) 数据读取时以偏移量为单位,偏移量不会重复读取
/*
 1@@ => 012
 2@@ => 345
 3 	 => 6
 */
3) 数据分区的偏移量范围的计算
   0 => [0,3]  => 12
   1 => [3,6]  => 3
   2 => [6,7]

var rdd = sc.textFile("datas/1.txt",2);  // minPartitions = 2;

```

### 转换算子

#### sample

```scala

    val scRDD = context.makeRDD(List(1,3,4,5,2,6,7,8,9));
    /**
     * sample需要传递三个参数
     *  第一个参数: 表示抽取数据是否将数据返回 true(放回) false(丢弃)
     *  第二个参数:
     *      如果抽取不放回, 数据源中每条数据被抽取的概率,基准值的概念
     *      如果收取返回, 数据源中的每条数据被抽取的可能次数
     *  第三个参数: 抽取数据时随机的算法种子
     *      如果不传递第三个参数,那么使用的是当前系统时间
     */
    val sampleRDD = scRDD.sample(false,0.4,1);
    sampleRDD.collect().foreach(println(_))
```

#### distinct

```sql
  将数据集中重复的数据去重
  def distinct()
  def distinct(numPartition: Int)

  val dataRDD = sc.makeRDD(List(1,3,4,1,2,3,4));
    val distinctRDD = dataRDD.distinct(2);
    distinctRDD.collect().foreach(println(_));
```

#### coalesce

```scala
 def coalesce(numPartitions: Int, shuffle: Boolean = false,
               partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
根据数据量缩减分区, 用于大数据集过滤后,提高小数据集的执行效率。
当spark程序中,存在过多的小任务的时候,可以通过coalesce方法,收缩合并分区,减少分区个数,减少任务调度成本

```

```scala
  /**
     * coalesce 方法默认情况下不会将分区的数据打乱重新组合
     * 这种情况的缩减数据可能会导致数据不均衡, 出现数据倾斜
     * 如果想让数据均衡,可以进行shuffle处理
     */
    val scRDD = context.makeRDD(List(1,2,3,4,5,6,7,8),4)
      .coalesce(2)
      .saveAsTextFile(ContanstUtil.path)

   coalesce算子可以扩大分区,但是如果不进行shuffle操作,是没有意义的,不起作用。
   所以如果想要实现扩大分区的效果,需要使用shuffle操作
   val scRDD=sc.makeRDD(List(1,2,3,4,5,6),2)
   scRDD.coalesce(3,true)
   spark提供一个简化的操作,repartition
   scRDD.repartition(3) // reparition底层实现还是使用的coalesce
```

#### sortBy

```scala
sortby 可以根据指定的规则进行排序,默认为升序

sortBy方法可以根据指定的跪着对数据源中的数据进行排序,默认为升序,第二个参数可以设置为false
sortBy默认情况下,不会改变分区。但是中间存在shuffle操作

val scRDD = sc.makeRDD(List(("a",2),("b",4),("c",3)),2)
    val newRDD = scRDD.sortBy(t=>t._2)
    newRDD.collect().foreach(println(_))


val scRDD = sc.makeRDD(List(6,2,4,5,1,3),2)
    val newRDD = scRDD.sortBy(num=>num,true) // 升序排序
    newRDD.collect().foreach(println(_))
排序前 分区1:6 2 4 分区2:5 1 3
排序后 分区1:1 2 3 分区2:4 5 6
排序前后有数据的shuffle的过程
```

#### 交并差集&拉链

```scala
 /**
     * 交集,并集,差集
     * 问题: 数据类型不一致的情况 会怎样, 编译不过,类型不匹配。
     * 交集要求两个RDD的数据类型一致,返回的结果也是相同的类型。
     */
    val scRDDnew = scRDDA.intersection(scRDDC)
    // 并集
    val unionRDD = scRDDA.union(scRDDC);
    // 差集
    val subtractRDD = scRDDA.subtract(scRDDC)
    /**
     * 拉链: 将相同位置的数据拉在一块
     * 拉链操作,两个数据源的类型可以不一致
     */
    val zipRDD1 = scRDDA.zip(scRDDC);
    val zipRDD2 = scRDDA.zip(scRDDB);

    val scRDDE = sc.makeRDD(List(1,2,3,4,5,6),2);
    val scRDDF = sc.makeRDD(List(2,3,4,5),3);
    /**
     * 抛出异常
     * cannot zip RDDs with unequal numbers of parititions List(2,4)
     * 说明: 拉链操作要求两个数据源的分区数量保持一致
     * cannot zip RDDs with same number of elements in each parition.
     * 说明: 要求分区中数据的数量保持一致。
     */
    val zipRDDC = scRDDE.zip(scRDDF);
```

#### partitionBy

```scala
 // 这种方式是均衡的进行分区,1 2在一个分区, 3 4在另一个分区
    val scRDDA = sc.makeRDD(List(1,2,3,4,4,4,4,9),2);
    /**
     * RDD => PairRDDFunctions 隐式转换
     * partitionBy 根据指定的分区规则进行重分区, 和 reparition和coaleace不同,
     *  reparition和coaleace指的是分区数量的大小改变,比如从3=>2 5=>8
     *  partitionBy 指的是,对数据进行分区位置的改变
     */
    val newRDD =scRDDA.map((_,1))
    /**
     * 分区1 (1,1)(2,1)(3,1)(4,1)
     * 分区2 (4,1)(4,1)(4,1)(9,1)
     */
    newRDD.saveAsTextFile("output1")
    /**
     * 分区1 (2,1)(4,1)(4,1)(4,1)(4,1)
     * 分区2 (1,1)(3,1)(9,1)
     */
    newRDD.partitionBy(new HashPartitioner(2)).saveAsTextFile("output2")


问题总结:
 /**
     * 如果重分区的分区器和当前RDD的分区器一样怎么办?
     *  如果一样就不再进行分区,返回还是原来的分区器
     */
    val partitionRDD = newRDD.partitionBy(new HashPartitioner(2));
    partitionRDD.partitionBy(new HashPartitioner(2));
```

##### hashParition

```scala
class HashPartitioner(partitions: Int) extends Partitioner {
  require(partitions >= 0, s"Number of partitions ($partitions) cannot be negative.")

  def numPartitions: Int = partitions
  def getPartition(key: Any): Int = key match {
    case null => 0
    case _ => Utils.nonNegativeMod(key.hashCode, numPartitions)
  }
  override def equals(other: Any): Boolean = other match {
    case h: HashPartitioner =>
      h.numPartitions == numPartitions // 类型相同, 数量相同就是同一个分区
    case _ =>
      false
  }
  override def hashCode: Int = numPartitions
}

```

#### reduceByKey

```scala
 def main(args: Array[String]): Unit = {
    val sc = SparkContextUtil.getSparkContext(null);
    val scRDD = sc.makeRDD(List(("a"->2),("a"->2),("c"->2),("a"->3),("a"->1)))
    /**
     * reduceByKey 相同的key数据进行value数据的聚合操作
     * scala语言中一般聚合操作都是两两聚合,spark基于scala开发,所以它的聚合也是两两几个
     * a [2,2,3,1]
     * c [3]  reduceByKey 如果key的数据只有一个,就直接返回,不用参与计算
     */
    val reduceRDD = scRDD.reduceByKey((x:Int,y:Int)=>{x+y})
    reduceRDD.collect().foreach(println(_))
    sc.stop();
  }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609556205019-6279e8d8-4c3f-4cd4-8424-732495fc6db8.png#align=left&display=inline&height=133&margin=%5Bobject%20Object%5D&name=image.png&originHeight=175&originWidth=746&size=21601&status=done&style=none&width=566)

#### groupByKey

```scala
val scRDD = sc.makeRDD(List(("a"->2),("a"->2),("c"->2),("a"->3),("a"->1)))
    /**
     * groupByKey: 将分区的数据直接转换为相同类型的内存数据进行后续处理
     * 将数据源中的数据,相同key的数据分在一个组中,形成一个对偶元组:
     *      元组中的第一个元素就是key, 第二个元素就是相同key的value集合
     */
    val groupRDD:RDD[(String, Iterable[Int])] = scRDD.groupByKey()
    groupRDD.collect().foreach(println(_))
    /**
     * groupBy
     * 1) groupByKey分组,固定是按key进行分组,而groupBy分组可以指定
     * 2) groupByKey按key进行分组,value可以独立出来,而groupBy没有固定按key进行,所以对整组元素进行分组
     */
    val groupByRDD:RDD[(String, Iterable[(String, Int)])] = scRDD.groupBy(t=>t._1)

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609555915889-7c8170a1-93f6-4b61-94f0-3d67e8477274.png#align=left&display=inline&height=191&margin=%5Bobject%20Object%5D&name=image.png&originHeight=219&originWidth=616&size=27830&status=done&style=none&width=537)
**reduceBykey 和 groupBykey 对比**

```scala
groupBykey
  会导致数据打乱重组,存在shuffle操作
reduceBykey
  也会导致数据shuffle,但reducebykey可以进行预聚合,减少shuffle的IO操作
  从性能上来看reduceByKey比groupBykey性能更优一些

ps:
  从shuffle的角度:  reduceBykey和groupBykey都存在shuffle的操作,但是reduceBykey可以在shuffle前对分区内相同key的数据进行预聚合(combine)操作,这样会减少落盘的数据量, 而groupByKey只能进行分组,不存在数据量减少的问题。所以reduceByKey性能更高。
  从功能的角度: reduceByKey其实包含分组和聚合的功能。GroupByKey只能分组,不能聚合,所以在分组聚合的场合下,推荐使用reduceByKey。如果仅仅是分组而不需要聚合。那么还是只能使用GroupByKey。
```

#### aggregateBykey&foldByKey

```scala
reduceByKey 在分区内和分区间操作是一样的都是聚合操作, 如果需要在分区内和分区键的操作不一致就需要使用aggregateByKey

 val scRDD = sc.makeRDD(List(("a",1),("a",2),("b",3),("b",4),("b",5),("a",6)),2)
    /**
     * 实例: 分区内去最大值,分区间聚合
     * aggregateByKey 存在函数柯力化,有两个参数列表
     *  第一个参数列表: 需要传递一个参数,表示初始值
     *  第二个参数列表需要传递2个参数
     *    第一个参数表示分区内计算规则
     *    第二个参数表示分区间计算规则
     */
    val aggreRDD = scRDD.aggregateByKey(0)((x,y)=>math.max(x,y),(x,y)=> x+y)
    aggreRDD.collect().foreach(println(_))


 /**
     * 分区内和分区间的计算可以是相同, 如下就是wordcount另一种表述方式
     */
    val aggRDD = scRDD.aggregateByKey(0)(_+_,_+_)
    aggRDD.collect().foreach(println(_));
 /**
     * 如果聚合计算,分区内和分区间是相同的时候,可以使用foldBykey代替
     */
    scRDD.foldByKey(0)(_+_).collect().foreach(println(_))
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609558329781-b99ba232-11f7-4583-bd56-9badbe6322e9.png#align=left&display=inline&height=107&margin=%5Bobject%20Object%5D&name=image.png&originHeight=145&originWidth=810&size=19699&status=done&style=none&width=600)

#### aggregateByKey&mapValues

```scala
 /** 获取相同key的平均值=> (a,3) (b,4) */
val scRDD = sc.makeRDD(List(("a",1),("a",2),("b",3),("b",4),("b",5),("a",6)),2)
var newRDD:RDD[(String, (Int, Int))]  = scRDD.aggregateByKey((0,0))(
       (t,v)=>{
         (t._1+v,t._2+1)
       },
       (t1,t2)=>{
         (t1._1+t2._1,t1._2+t2._2)
       }
)
/** 对key的处理保持不变,只对value的数值进行处理 (参数就是value的类型 tuple(Int,Int))*/
var resRDD:RDD[(String,Int)] = newRDD.mapValues {
       case (num, cnt) => {
         num / cnt
       }
     }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1609559671540-7a5683a5-c8e2-43e1-93b2-4fc15183ab3a.png#align=left&display=inline&height=117&margin=%5Bobject%20Object%5D&name=image.png&originHeight=145&originWidth=810&size=21374&status=done&style=none&width=651)

#### combineByKey

```scala

val scRDD = sc.makeRDD(List(("a",1),("a",2),("b",3),("b",4),("b",5),("a",6)),2)

/** combineByKey 替换了aggregateByKey的初始值
     * combineByKey 方法需要三个参数
     * 第一个参数表示: 将相同key的第一个数据进行结构的转换,实现操作
     * 第二个参数表示: 分区内的计算规则
     * 第三个参数表示: 分区间的计算规则
     */
    scRDD.combineByKey(
      v=>(v,1),
      (t:(Int,Int),v)=>{
        (t._1+v,t._2+1)
      },
      (t1:(Int,Int),t2:(Int,Int))=>{
        (t1._1+t2._1,t1._2+t2._2)
      }
    )
```

#### 聚合算子的区别

```scala

val rdd = sc.makeRDD(List(("a",1),("a",2),("b",3),("b",4),("b",5),("a",6)),2)
rdd.reduceByKey(_+_)
rdd.aggregateByKey(0)(_+_,_+_)
rdd.foldByKey(0)(_+_)
rdd.combineByKey(v=>v,(x:Int,y)=>x+y,(x:Int,y:Int)=>x+y)
rdd.countByKey()

底层都是使用的combineByKeyWithClassTag

```

#### join 的操作

```scala

在类型为(K,v)和(K,w)的RDD上调用,返回一个相同key对应的所有元素连接在一起的(K,(v,w))的RDD

    val scRDDA = sc.makeRDD(List(("a",1),("b",2),("c",3)))
    val scRDDB = sc.makeRDD(List(("a",5),("c",6),("a",3)))
    /**
     * join: 两个不同数据源的数据,相同的key会连接在一起,形成元组
     *      如果两个数据源中key没有匹配上,那么数据不会出现在结果中
     *      如果两个数据源中key有多个相同的,会依次匹配,可能会出现笛卡尔乘积,数据量会几何形增长
     *      导致性能降低
     */
    val joinRDD:RDD[(String, (Int, Int))]= scRDDA.join(scRDDB)
    val leftRDD:RDD[(String, (Int, Option[Int]))] =  scRDDA.leftOuterJoin(scRDDB);
    val rightRDD:RDD[(String, (Option[Int], Int))] = scRDDA.rightOuterJoin(scRDDB);

```

##

#### cogroup

```scala
coGroup = connect + group

val scRDDA = sc.makeRDD(List(("a",1),("b",2),("c",3)))
    val scRDDB = sc.makeRDD(List(("a",5),("c",6),("a",3)))
    /**
     * (a,1-53)
     * (b,2-)
     * (c,3-6)
     */
    val coGroupRDD:RDD[(String, (Iterable[Int], Iterable[Int]))]  = scRDDA.cogroup(scRDDB);
    coGroupRDD.mapValues(t=>{
      t._1.mkString+'-'+t._2.mkString
    }).collect().foreach(println(_))

```

## action

#### collect

```scala
val rdd=sc.makeRDD(List(1,2,3,4))
/**
 * 所谓的行动算子,其实就是触发作业(job)执行的方法
 * 底层代码调用的是环境对象的runJob方法
 * 底层代码中会创建ActiveJob,并提交执行
 */
rdd.collect()

collect方法的介绍
   val rdd = sc.makeRDD(List(1,2,3,4));
    /** collect
     *  将不同分区的数据,按照分区顺序采集到Driver端内存中,形成数组
     */
    val res:Array[Int] = rdd.collect();

collect()
	在驱动程序中,以数组的形式返回数据集的所有元素
  collect 收集一个弹性分布式数据集的所有元素到一个数组中,这样便于我们观察,毕竟分布式数据集比较抽象。
  spark的collect方法,是action类型的一个算子,会从远程集群拉取数据到driver端。最后,将大量数据汇集到一个driver
  节点上,将数据用数组存放,占用了jvm堆内存,非常容易造成内存溢出,只用做小型数据的观察
    val lines : RDD[String] = context.textFile("/Users/ssgao/Downloads/test.txt");
    val words: RDD[String] = lines.flatMap(_.split(" "))
    val wordGroup =  words.groupBy(words=>words)
    val wordCount = wordGroup.map {
      case (word, list) => word->list.size
    }
    var result = wordCount.collect().toList;
    result.foreach(println(_))
```

#### reduce

```scala
reduce聚合RDD中的所有元素,先聚合分区内数据,在聚合分区间数据

def main(args: Array[String]): Unit = {
    val sc = SparkContextUtil.getSparkContext(null);
    val rdd = sc.makeRDD(List(1,2,3,4));
    val res = rdd.reduce(_+_);
    println(res)
  }
```

#### count&first&take

```scala
count()
	返回RDD的元素个数
first()
	返回RDD的第一个元素
take(n)
  返回一个由数据集的前n个元素组成的数组


```

#### aggregate&&flod

```scala
def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U
	分区的数据通过初始值和分区内的数据进行聚合,然后再和初始值进行分区间的数据聚合

/**
 *  aggregateByKey 初始值只会参与分区内计算
 *  aggregate 初始值 会参与分区内计算, 并且也会参与分区间计算
 *  10 + 1 +2  10 +3 + 4 => 10 + 13 + 17 => 40
 */
val res = scrdd.aggregate(10)(_+_,_+_);
// 分区间和分区内计算公式一样的情况下,可以使用fold来代替aggregate
val res2 = scrdd.fold(10)(_+_);
    println(res2)
```

#### countByKey&countByValue

```scala
    val scrdd = sc.makeRDD(List(1,2,1,4),2);
    /** 统计数值出现的次数 */
    val resRDD1: collection.Map[Int, Long] = scrdd.countByValue()
    println(resRDD1) //Map(4 -> 1, 2 -> 1, 1 -> 2)

    val scrdd2= sc.makeRDD(List(("a",1),("a",2),("b",3),("a",4)),2);
    val resRDD2:collection.Map[String, Long] = scrdd2.countByKey()
    println(resRDD2) //Map(b -> 1, a -> 3)
```

#### save 算子

```scala
 val scrdd= sc.makeRDD(List(("a",1),("a",2),("b",3),("a",4)),2);

scrdd.saveAsTextFile("output")
scrdd.saveAsTextFile("objectput")
// 要求数据的格式必须为key->value类型
scrdd.saveAsSequenceFile("seqput")
```
