---
layout: post
title: 高并发Disruptor
categories: [多线程, Disruptor]
description: 高并发Disruptor
keywords: 多线程,Disruptor
---

 <meta name="referrer" content="no-referrer"/>

### disruptor 介绍

```java
disruptor是一个开源的并发框架,能够在无锁的情况下实现网络的Queue的并发操作。
  disruptor模式是 生产者-消费者模式，而介于两者时间的是RingBuffer环，生产者（例如请求事件）不断向RingBuffer中放入事件object，而消费者不断读取RingBuffer
  Disruptor是一个高性能的异步处理框架,或者可以认为是最快的消息框架(轻量的JMS),也可以认为是一个观察者模式的实现,或者事件监听模式的实现

  "ringBuffer的优点"
  传统消息框架使用Queue队列，如JDK LinkedList等数据结构实现，RingBuffer比Linked之类数据结构要快，因为没有锁，是CPU友好型的。
  另外一个不同的地方是不会在清除RingBuffer中数据，只会覆盖，这样降低了垃圾回收机制启动频率。


  maven依赖
  <dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.3.7</version>
 </dependency>
```

### 队列对比

```java
Disruptor是一款高性能的有界内存队列, 目前应用非常广泛, Log4J2, SpringMessaging, Hbase
Storm都用到了Disruptor。disruptor的优势:
  1>> 内存分配合理, 使用RingBuffer数据结构, 数组元素在初始化时一次性全部创建
  	  提升缓存命中率, 对象循环利用, 避免频繁GC
  2>> 能够避免伪共享,提升缓存利用率
  3>> 采用无锁算法,避免频繁加锁, 解锁的性能消耗。
  4>> 支持批量消费,消费者可以无锁方式消费多个消息。

```

### disruptor 术语

```java
RingBuffer
  被看作Disruptor最主要的组件,然而在3.0以后RingBuffer仅用来负责存储和更新在Disruptor中流通的数据。
  对一些特殊的使用场景能够被用户(其他数据结构)完全代替。
Producer
   由用户实现,它调用RingBuffer来插入数据(Event)，在Disruptor中没有相应的实现代码，由用户实现。

WaitStrategy
   决定一个消费者将如何等待生产者将Event(数据)置入Disruptor

Event
  从生产者到消费者过程中所处理的数据单元。Dispatcher把数据认为是event
  Disruptor中没有代码表示Event，因为它完全是由用户定义的。
"EventProcessor"
   主要事件循环(相当于生产者生产数据所需要的线程)，处理Disruptor中的Event，并且拥有消费者的Sequence。
   它有一个实现类是BatchEventProcessor，包含了event loop有效的实现，并且将回调到一个EventHandler接口的实现对象。
EventHandler
   由用户实现并且代表了Disruptor中的一个消费者的接口。
   提供onEvent()方法，负责具体业务逻辑实现。
EventHandlerGroup
   业务处理器分组,管理多个业务处理器的依赖关系,提供then(),before(),after()等api


"WorkProcessor"
  也是为生产者生产数据的线程
  确保每个sequence只被一个processor消费，在同一个WorkPool中的处理多个WorkProcessor不会消费同样的sequence。
WorkerPool
   一个WorkProcessor池，其中WorkProcessor将消费Sequence，所以任务可以在实现WorkHandler接口的worker吃间移交

LifecycleAware
   当BatchEventProcessor启动和停止时，于实现这个接口用于接收通知。


```

```java
eventfactory 用于创建Event(消费数据),该工厂在RingBuffer被初始化时,在fill中调用,用于初始化RingBuffer的单元。
通过调用newInstance方法创建Event(数据对象实体)填入到RingBuffer的槽位
```

#### 序号

```java
Sequence 序号
  可以认为是RingBuffer的槽位
  生产者生产数据放在RingBuffer中的哪个位置, 消费者应该消费哪个位置的数据。
  这个序号可以简单的理解为一个Automic Long类型的变量。其使用了Padding的方法去消除缓存的伪共享问题。


Sequencer 序号生产器
  这是Disruptor真正的核心。
  实现了这个接口的两种生产者(SingleProducerSequencer,MultiProducerSequencer)均实现了所有的并发算法,为了在生产者和消费者之间进行准确快速的数据传递。

SequenceBarrier  序号屏障
  我们知道消费者,在消费数据的时候,需要知道消费哪个位置的数据。消费者总不能自己想取哪个数据消费,就取哪个数据消费(这样就混乱了) 这个SequenceBarrier起到就是这样一个"栅栏"般的阻隔作用。你消费者想消费数据,需要被告知一个序号(sequence), 然后消费者去指定的位置消费数据。要是没有数据需要根据waitStrategy的配置进行等待。

Sequence、Sequencer、SequencerBarrier这三个概念无非就是Ringbuffer中哪里都需要用到序号(Sequence)，而Sequencer用于生产者生产的时候产生序号，SeqencerBarrier就是协调生产者与消费者，并且告诉消费者一个可以用于消费的序号！
```

### disruptor 的使用

#### 基本使用

```java
> 建立一个Event类
> 建立一个工厂Event类,用于创建Event类实例对象
> 需要有一个监听事件类,用于处理数据Event类
> 我们需要惊醒测试代码编写,实例化Dispatcher实例,配置一系列参数,然后我们对Dispatcher实例绑定监听事件类,接受并处理数据
> 在Disruptor中,真正存储数据的核心叫做RingBuffer,我们通过Disruptor实例拿到它,然后把数据生产出来,把数据加入到RingBuffer的实例对象中

```

```java
"event数据类"
public class FileData {
    private String line;
    public String getLine() {
        return line;
    }
    public void setLine(String line) {
        this.line = line;
    }
}

"主消费线程"
public class DemoTest {
    public static void main(String[] args) {
        //创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        //创建buffer的大小
        int ringBufSize=1024*1024;
        //创建数据工厂,将创建的Event数据填充到ringBuffer中
        EventFactory<FileData> eventFactory = new EventFactory<FileData>() {
            public FileData newInstance() {
                return new FileData();
            }
        };
        /**
         * 第一个参数: 工厂类对象,用于创建一个个FileData,FileData是实际的消费数据
         * 第二个参数: 缓冲区大小
         * 第三个参数: 线程池,进行Disruptor的内部调度
         * 第四个参数: ProducerType.SINGLE,MULTI
         * 第五个参数: 是一种策略,为生产者和消费者做平衡策略
         */
        Disruptor<FileData> disruptor = new Disruptor<FileData>(
                eventFactory,
                ringBufSize,
                executorService,
                ProducerType.SINGLE,
                new YieldingWaitStrategy()
        );
        //设置消费者,可以设置多个消费者
        disruptor.handleEventsWith(new DisruptorConsumer());

        //启动消费
        disruptor.start();
        /**
         * disruptor获取ringbuffer,用于生产数据,供消费者消费
         */
        RingBuffer<FileData> ringBuffer = disruptor.getRingBuffer();
        try {
            File file = new File("/Users/ssgao/ssgao/log.log");
            BufferedReader bufferedReader = new BufferedReader(new FileReader(file));
            String line ;
            while ((line = bufferedReader.readLine())!=null){
                //可以把ringBuffer看做一个事件队列，那么next就是得到下面一个事件槽(放数据的槽位)
                long seq = ringBuffer.next();
                try {
                   //用上面的索引取出一个空的事件(数据对象)用于填充（获取该序号对应的事件对象）
                   FileData fileData=ringBuffer.get(seq);
                   fileData.setLine(line);
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    //使用ringbuffer把数据发布
                    ringBuffer.publish(seq);
                }
            }
            bufferedReader.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        //关闭 disruptor，方法会堵塞，直至所有的事件都得到处理；
        disruptor.shutdown();
        //关闭 disruptor 使用的线程池;如果需要的话，必须手动关闭， disruptor 在 shutdown 时不会自动关闭；
        executorService.shutdown();
    }
}

"消费者使用了EventHandler的接口"
public class DisruptorConsumer implements EventHandler<FileData>{
    public void onEvent(FileData event, long sequence, boolean endOfBatch) throws Exception {
        System.out.print("-->");
        System.out.println(event.getLine());
    }
}

```

#### 等待策略

```java
Disruptor生产者和消费者之间是通过什么策略进行同步的，Disruptor提供的策略如下

BlockingWaitStrategy  一般应用从
   默认等待策略。
   和BlockingQueue的实现很类似，通过使用锁和条件（Condition）进行线程同步和唤醒。
   此策略对于线程切换来说，最节约CPU资源(对CPU的消耗最小)，但在高并发场景下性能有限。


SleepingWaitStrategy
   CPU友好型策略。
   会在循环中不断等待数据。
   首先进行自旋等待，若不成功，则使用Thread.yield()让出CPU，并使用LockSupport.parkNanos(1)进行线程睡眠。
   所以，此策略数据处理数据可能会有较高的延迟，适合用于对延迟不敏感的场景。优点是对生产者线程影响小，典型应用场景是异步日志。

YieldingWaitStrategy
   低延时策略。
   消费者线程会不断循环监控RingBuffer的变化，在循环内部使用Thread.yield()让出CPU给其他线程。
    YieldingWaitStrategy：尝试自旋 100 次，然后调用 Thread.yield() 让出 cpu。cpu 占用高，延迟低。
   ps: 'YieldingWaitStrategy'这个策略一定要非常谨慎的使用, 会是的CPU飙到100%

   YieldingWaitStrategy(放弃等待) 性能是最好的,适合用于低延迟的系统, 在要求极高性能且事件处理线程数小于CPU逻辑核心数的场景下(即消费线程数要小于Cpu的core数),推荐使用此策略。


 BusySpinWaitStrategy
   死循环策略。消费者线程会尽最大可能监控缓冲区的变化，会占用所有CPU资源。



waitStrategy是WaitStrategy接口的对象，通过名称可以确认这是一个策略模式，使用waitFor()和signalAllWhenBlocking()两个方法。

waitFor()方法，
   用于让EventProcessor（事件的处理器）等待可用的sequence个数
   如果没有生产者生成的产品或者依赖的处理队列没有处理完，就会一直等待；一般在SequenceBarrier的waitFor方法中调用。

signalAllWhenBlocking()方法，用于激活等待的EventProcessor
 一般在Sequencer继承的Sequenced接口的publish()方法中使用，当生产者有产出后，会激活等待的EventProcessor。
```

#### ringbuffer

```java
如名所述
  它是一个环(首尾相接的环),我们可以用做在不同上下文(线程)间传递数据的buffer
  基本来说,ringbuffer拥有一个序号,这个序号指向数组中下一个可用的元素。随着我们不停的填充这个buffer(可能会有相应的读取),这个需要会一直增长,直到绕过这个环。

  要找到数组中当前序号指向的元素,我们可以通过mod操作
  如下图ringbuffer 12%10=2
  实际使用是一般将ringBuffer的槽位设置为2的N次方更有利于二进制的计算机进行计算

```

```java
/** 创建RingBuffer*/
RingBuffer<Order> ringBuffer = RingBuffer.create(ProducerType.MULTI,new EventFactory<Order>(){
		@Override
    	public Order newInstance(){
        	return new Order();
        }
	}, 1024*1024, new YieldingWaitStrategy());

ProducerType.MULTI: 表示多生产者,还是单生产者
Order: 表示数据

public class Order{
	private String id;
    private String name;
    private double price;
}
```

### disruptor 使用示例

#### 单生产者模式

```java
"使用eventProcessor"
public class MainTestA {
    public static void main(String[] args) throws Exception{
        int BUFFER_SIZE=1024;
        int THREAD_NUMBERS=4;
        /**
         * createSingleProducer 创建一个单生产者的RingBuffer
         * 第一个参数叫EventFactory, 用于创建事件,即数据信息(生产数据填充RingBuffer区块)
         * 第二个参数是RingBuffer的大小,必须是2的指数倍,将求模运算转换为&运算提高效率
         * 第三个参数是等待策略
         */
        final RingBuffer<FileData> ringBuffer = RingBuffer.createSingleProducer(new EventFactory<FileData>() {
                public FileData newInstance() {
                    return new FileData();
                }
        },BUFFER_SIZE,new YieldingWaitStrategy());
        //创建线程池
        ExecutorService executors = Executors.newFixedThreadPool(THREAD_NUMBERS);
        //创建SequenceBarrier
        //生产者生产块或消费块的时候,使用Barrier(屏障)来平衡sequence
        SequenceBarrier sequenceBarrier = ringBuffer.newBarrier();

        //创建消息处理器
        BatchEventProcessor<FileData> eventProcessor = new BatchEventProcessor<FileData>(
                ringBuffer,sequenceBarrier,new DisruptorConsumer()
        );
        //把消息者的位置信息引用注入到生产者,如果只有一个消费者可以省略
        ringBuffer.addGatingSequences(eventProcessor.getSequence());
        //把消息处理器提交到线程池
        executors.submit(eventProcessor);
        /**
         * 如果存在多个消费者,那么重复执行上面的三行代码,把DisruptorConsumer换成对应的消息处理类
         */

        //生产数据
        Future future = executors.submit(new Callable<Void>() {
            public Void call() throws Exception {
                long seq;
                for(int i=0;i<10;i++){
                    seq = ringBuffer.next();
                    ringBuffer.get(seq).setLine("ssgao");
                    ringBuffer.publish(seq);
                }
                return null;
            }
        });

        future.get();//等待生产结束
        Thread.sleep(1000); //等1s,等消费者处理完成
        eventProcessor.halt(); //通知事件处理器,可以结束了(并不是马上结束)
        executors.shutdown(); //终止线程
    }
}

"使用workPool"
public class MainTestB {
    public static void main(String[] args) throws Exception{
        int BUFFER_SIZE=1024;
        int THREAD_NUMBERS=4;
        /**
         * createSingleProducer 创建一个单生产者的RingBuffer
         * 第一个参数叫EventFactory, 用于创建事件,即数据信息(生产数据填充RingBuffer区块)
         * 第二个参数是RingBuffer的大小,必须是2的指数倍,将求模运算转换为&运算提高效率
         * 第三个参数是等待策略
         */
        final RingBuffer<FileData> ringBuffer = RingBuffer.createSingleProducer(new EventFactory<FileData>() {
                public FileData newInstance() {
                    return new FileData();
                }
        },BUFFER_SIZE,new YieldingWaitStrategy());
        //创建线程池
        ExecutorService executors = Executors.newFixedThreadPool(THREAD_NUMBERS);
        //创建SequenceBarrier
        //生产者生产块或消费块的时候,使用Barrier(屏障)来平衡sequence
        SequenceBarrier sequenceBarrier = ringBuffer.newBarrier();

        //创建消费者, 单消费者
        WorkerPool<FileData> workerPool = new WorkerPool<FileData>(ringBuffer, sequenceBarrier, new IgnoreExceptionHandler(), new MyWorkHandler());

        workerPool.start(executors);
        //生产数据
        long seq;
        for(int i=0;i<10;i++){
            seq = ringBuffer.next();
            ringBuffer.get(seq).setLine("ssgao");
            ringBuffer.publish(seq);
        }
        Thread.sleep(1000); //等1s,等消费者处理完成
        workerPool.halt(); //通知事件处理器,可以结束了(并不是马上结束)
        executors.shutdown(); //终止线程
    }
}

public class MyWorkHandler implements WorkHandler<FileData> {
    public void onEvent(FileData event) throws Exception {
        System.out.println(event.getLine());
    }
}

```

#### 多生产者模式

```java
public class TestMain {
    public static void main(String[] args) throws Exception {
        //创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        //创建buffer的大小
        int ringBufSize=1024*1024;
        //创建数据工厂,将创建的Event数据填充到ringBuffer中
        EventFactory<FileData> eventFactory = new EventFactory<FileData>() {
            public FileData newInstance() {
                return new FileData();
            }
        };
        /**
         * 第一个参数: 工厂类对象,用于创建一个个FileData,FileData是实际的消费数据
         * 第二个参数: 缓冲区大小
         * 第三个参数: 线程池,进行Disruptor的内部调度
         * 第四个参数: ProducerType.SINGLE,MULTI
         * 第五个参数: 是一种策略,为生产者和消费者做平衡策略
         */
        final Disruptor<FileData> disruptor = new Disruptor<FileData>(
                eventFactory,
                ringBufSize,
                executorService,
                ProducerType.SINGLE,
                new YieldingWaitStrategy()
        );
        //设置消费者
        EventHandlerGroup<FileData> fileDataEventHandlerGroup
                 = disruptor.handleEventsWith(new CustomeHandlerA(), new CustomeHandlerB());
        fileDataEventHandlerGroup.handleEventsWith(new CustomeHandlerC());
        //启动
        disruptor.start();

        //开始产生数据,使用countDownLatch用于设置数据生产完后再进行消费
        final CountDownLatch latch = new CountDownLatch(1);
        for(int i=0;i<10;i++){
          final Procuder procuder=  new Procuder(disruptor.getRingBuffer());
           new Thread(new Runnable() {
               public void run() {
                   try {
                       latch.await();
                       for(int j=0;j<10;j++){
                           procuder.pushData("ssgao"+Thread.currentThread().getName()+"--->"+j);
                       }
                   } catch (Exception e) {
                       e.printStackTrace();
                   }
               }
           }).start();
        }
        Thread.sleep(1000);
        latch.countDown();
        Thread.sleep(5000);

        disruptor.shutdown();
        executorService.shutdown();
        System.out.println("数据处理完成!");
    }
}
```

```java
Disruptor 它为什么快的原因。

多线程下其他队列之所以慢,是因为互斥锁的原因。
Distuptor就解决了这个问题, Disruptor它是通过valatile关键字实现。Disruptor的volatile变量都会不会出现同时写的操作,所以volatile就可以达到要求。

Disruptor的RingBuffer的指针是一个volatile对象,消费者中的序列号也是volatile类型。



```

#### 伪共享 cacheline

> [_https://www.pianshen.com/article/19491078659/_](https://www.pianshen.com/article/19491078659/)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611471726431-e557d862-11b2-40a1-8ce7-08bde2a914c1.png#height=173&id=knNzp&margin=%5Bobject%20Object%5D&name=image.png&originHeight=345&originWidth=593&originalType=binary&ratio=1&size=19822&status=done&style=none&width=296.5)

```java
这个里面的Value就是Sequence的实际值，是个volatile long，RhsPadding便是右边的填充，LhsPadding表示左边的填充，左右两边各填充7个long，这样在缓存行中，无论Sequence的value在缓存行什么位置，都可以保证不会有别的缓存影响到它自己(自己独占一个缓存行，空间换时间)，这就是缓存行填充，消除伪共享。
value最坏的情况就是位于cache line的头或者为, 所以value左右个填充7个long类型数据。
```

```java
class LhsPadding{
    protected long p1, p2, p3, p4, p5, p6, p7;
}

class Value extends LhsPadding{
    protected volatile long value;   "注意volatile的位置"
}

class RhsPadding extends Value{
    protected long p9, p10, p11, p12, p13, p14, p15;
}

/** Sequence是一个做了缓存行填充优化的原子序列 */
public class Sequence extends RhsPadding{
    static final long INITIAL_VALUE = -1L;
    private static final Unsafe UNSAFE;
    private static final long VALUE_OFFSET;

    static
    {
        UNSAFE = Util.getUnsafe();
        try
        {
            VALUE_OFFSET = UNSAFE.objectFieldOffset(Value.class.getDeclaredField("value"));
        }
        catch (final Exception e)
        {
            throw new RuntimeException(e);
        }
    }

     public Sequence(final long initialValue){
        UNSAFE.putOrderedLong(this, VALUE_OFFSET, initialValue);
    }
    public void set(final long value){
        UNSAFE.putOrderedLong(this, VALUE_OFFSET, value);
    }
    public void setVolatile(final long value){
        UNSAFE.putLongVolatile(this, VALUE_OFFSET, value);
    }
    public long incrementAndGet(){
        return addAndGet(1L);
    }
    /**CAS自旋 */
    public long addAndGet(final long increment){
        long currentValue;
        long newValue;

        do
        {
            currentValue = get();
            newValue = currentValue + increment;
        }
        while (!compareAndSet(currentValue, newValue));

        return newValue;
    }

}
```

```java
注：除了填充法，还可以使用@Contended注解消除伪共享问题，要注意的是user classpath使用此注解默认是无效的，需要在jvm启动时设置-XX:-RestrictContended。

```

#### 无锁算法

### 性能优化点

#### 序号栅栏

### 总结

#### disreptor 为什么快

```java
环形数组结构
	数组初始化时,就分配好内存,避免垃圾回收,采用数组而非链表, 同时数组对处理器的缓存机制更加友好。
    采用缓存行填充的形式解决伪共享问题。

元素位置定位
	数组长度为2^n, 通过位运算, 加快定位的速度
    所以ringbuffer的大小必须是要2^n

无锁设计
	每个生产者或者消费者线程,会先申请可以操作的元素在数组中的位置, 申请到了之后,直接在该位置写入或者读取数据。
	线程通过sequence访问ringBuffer, 通过cas取代了加锁, 这也是并发编程的原则,把同步块最小化到一个变量上。

ps: Disruptor是基于JVM的, 是单JVM下的无锁队列技术, 不是分布式下的。
```

### 源码解读

> [_https://blog.csdn.net/xorxos/article/details/80503358_](https://blog.csdn.net/xorxos/article/details/80503358)

> [_https://www.alicharles.com/article/disruptor/disruptor-ringbuffer-muti-write/_](https://www.alicharles.com/article/disruptor/disruptor-ringbuffer-muti-write/)
