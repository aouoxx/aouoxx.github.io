#### 功能介绍

```java
DelayedWorkQueue也是一种设计为定时任务的延迟队列, 它的实现和DelayQueue一样,不过是将优先级队列和DelayQueue的实现过程迁移到本身方法体中,即重新实现了小顶堆的排序。


DelayedWorkQueue是用数组来存储队列中的元素, 核心数据结构是二叉最小堆的优先队列, 队列满时会自动扩容。


1) DelayedWorkQueue的数据结构是基于堆实现的
2) DelayedWorkQueue采用数组实现堆, 根节点出队, 用最后叶子节点替换, 然后下推至满足堆成立条件
	最后叶子节点入队,然后向上推至满足堆成立条件

3) DelayedWorkQueue 添加元素满了之后会自动扩容原来容量的1/2 即永远不会阻塞, 最大扩容可达Integer.MAX_VALUE, 所以线程池中至多有corePoolSize个工作线程正在运行。
4) DelayedWorkQueue 消费元素take, 在堆顶元素为空和delay > 0时, 阻塞等待。
5) DelayedWorkQueue 是一个生产永远不会阻塞, 消费可以阻塞的生产者消费则模式。
6) DelayedWorkQueue 有一个leader线程的变量, 是Leader-Follower模式的变种。
		当一个take线程变成leader线程时,只需要等待下一次的延迟时间,而不是leader线程的其他take线程则需要等待
        leader线程出队了才唤醒其他take线程。

```

```java
 DelayedWorkQueue 是ScheduleThreadPoolExector的静态内部类, 默认只有一个无参构造方法


static class DelayedWorkQueue extends AbstractQueue<Runnable> implements BlockingQueue<Runnable> {
	// 初始时，数组长度大小。
	private static final int INITIAL_CAPACITY = 16;
	// 使用数组来储存队列中的元素。
	private RunnableScheduledFuture<?>[] queue =  new RunnableScheduledFuture<?>[INITIAL_CAPACITY];
	// 使用lock来保证多线程并发安全问题。
	private final ReentrantLock lock = new ReentrantLock();
	// 队列中储存元素的大小
	private int size = 0;
	//特指队列头任务所在线程
	private Thread leader = null;
	// 当队列头的任务延时时间到了，或者有新的任务变成队列头时，用来唤醒等待线程
	private final Condition available = lock.newCondition();



}
```

#### offer 任务

```java
/**
 * DelayedWorkQueue 提供了put/add/offer(带时间) 三个插入元素方法。
 * 三个方法都是调用offer方法, 因为队列长度不限制, 可以不断的向DelayedWorkQueue添加元素
 * 当元素个数超过数组长度时, 会进行数组扩容
 */
public void put(Runnable e) { offer(e); }
public boolean add(Runnable e) { return offer(e);  }
public boolean offer(Runnable e, long timeout, TimeUnit unit) {
   return offer(e);
}
public boolean offer(Runnable x) {
    if (x == null)
        throw new NullPointerException();
    RunnableScheduledFuture<?> e = (RunnableScheduledFuture<?>)x;
    final ReentrantLock lock = this.lock; //使用lock保证并发操作安全
    lock.lock();
    try {
        int i = size;
        if (i >= queue.length) //如果超过数组长度,就要进行数组扩容
            grow(); // 数组扩容
        size = i + 1;
        if (i == 0) {
            queue[0] = e;
            setIndex(e, 0);
        } else {
            siftUp(i, e); //重新排序
        }
        /** 表示新插入的元素是队列头, 【更换了队列头】, 那么就要唤醒正在等待获取任务的线程 */
        if (queue[0] == e) {
            leader = null; //这里将leader置为null,因为队列头更换, 为当前最新插入的任务
            available.signal(); // 唤醒正在等待获取任务的线程
        }
    } finally {
        lock.unlock();
    }
    return true;
}

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1612231345098-24fb7dd0-fcc2-4753-98ed-dbdcce2b20d7.png#crop=0&crop=0&crop=1&crop=1&height=356&id=ZfccY&margin=%5Bobject%20Object%5D&name=image.png&originHeight=571&originWidth=666&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38764&status=done&style=none&title=&width=415)

#### take 任务

```java
DelayedWorkQueue提供下面几个出队方法
 1) take() 等待获取队列头元素
 2) poll() 理解获取队列头元素
 3) poll(long timeout, TimeUnit unit) 超时等待获取队列头元素
```

```java
 public RunnableScheduledFuture<?> take() throws InterruptedException {
            final ReentrantLock lock = this.lock;
            lock.lockInterruptibly();
            try {
                for (;;) {
                    // 如果没有任务，就让线程在available条件下等待。
                    RunnableScheduledFuture<?> first = queue[0];
                    if (first == null)
                        available.await(); // 说明没有任务,需要等待任务插入
                    else {
                        long delay = first.getDelay(NANOSECONDS); // 获取任务的剩余时间
                        if (delay <= 0) // delay<0 说明延时时间到了,就返回这个任务,去执行
                            return finishPoll(first);
                        first = null; // don't retain ref while waiting
                        if (leader != null)
                            available.await();
                        else {
                            // 记录下当前等待队列头任务的线程
                            Thread thisThread = Thread.currentThread();
                            leader = thisThread;
                            try {
                                // 当任务延时时间到了时, 能够自动超时唤醒
                                available.awaitNanos(delay);
                            } finally {
                                if (leader == thisThread)
                                    leader = null;
                            }
                        }
                    }
                }
            } finally {
                if (leader == null && queue[0] != null)
                    available.signal(); // 唤醒等待任务的线程
                lock.unlock();
            }
        }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1612232357711-95e11872-71b3-4349-b97c-c206dcc79e9b.png#crop=0&crop=0&crop=1&crop=1&height=326&id=zs1gN&margin=%5Bobject%20Object%5D&name=image.png&originHeight=652&originWidth=629&originalType=binary&ratio=1&rotation=0&showTitle=false&size=54258&status=done&style=none&title=&width=314.5)

#### finishPoll 出队列

```java
// 移除队列头元素
private RunnableScheduledFuture<?> finishPoll(RunnableScheduledFuture<?> f) {
    // 将队列中元素个数减一
    int s = --size;
    // 获取队列末尾元素x
    RunnableScheduledFuture<?> x = queue[s];
    // 原队列末尾元素设置为null
    queue[s] = null;
    if (s != 0)
        // 因为移除了队列头元素，所以进行重新排序。
        siftDown(0, x);
    setIndex(f, -1);
    return f;
}

堆的删除方法主要分为三步:
1) 先将队列中元素个数减一
2) 将原队列末尾元素设置成队头元素,将蒋队列末尾元素设置为null
3）调用setDown(O,x)方法,保证按照元素的优先级排序
```

#### poll

```java
立即获取队列头元素，当队列头任务是null，或者任务延时时间没有到，表示这个任务还不能返回，因此直接返回null。
否则调用finishPoll方法，移除队列头元素并返回。

public RunnableScheduledFuture<?> poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
     RunnableScheduledFuture<?> first = queue[0];
        // 队列头任务是null，或者任务延时时间没有到，都返回null
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
         // 移除队列头元素
            return finishPoll(first);
    } finally {
     lock.unlock();
    }
}
```

#### poll(long timeout,TimeUnit unit)

```java
超时等待获取队列头元素，与take方法相比较，就要考虑设置的超时时间，
	如果超时时间到了，还没有获取到有用任务，那么就返回null。
    其他的与take方法中逻辑一样。

public RunnableScheduledFuture<?> poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
         RunnableScheduledFuture<?> first = queue[0];
            // 如果没有任务。
            if (first == null) {
             // 超时时间已到，那么就直接返回null
                if (nanos <= 0)
                    return null;
                else
                 // 否则就让线程在available条件下等待nanos时间
                    nanos = available.awaitNanos(nanos);
            } else {
                // 获取任务的剩余延时时间
                long delay = first.getDelay(NANOSECONDS);
                // 如果延时时间到了，就返回这个任务，用来执行。
                if (delay <= 0)
                    return finishPoll(first);
                // 如果超时时间已到，那么就直接返回null
                if (nanos <= 0)
                    return null;
                // 将first设置为null，当线程等待时，不持有first的引用
                first = null; // don't retain ref while waiting
                // 如果超时时间小于任务的剩余延时时间，那么就有可能获取不到任务。
                // 在这里让线程等待超时时间nanos
                if (nanos < delay || leader != null)
                 nanos = available.awaitNanos(nanos);
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        // 当任务的延时时间到了时，能够自动超时唤醒。
                        long timeLeft = available.awaitNanos(delay);
                        // 计算剩余的超时时间
                        nanos -= delay - timeLeft;
                    } finally {
                        if (leader == thisThread)
                         leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && queue[0] != null)                    // 唤醒等待任务的线程
         available.signal();
        lock.unlock();
    }
}
```

#### remove 删除指定元素

```java
删除指定元素一般用于取消任务时，任务还在阻塞队列中，则需要将其删除。
当删除的元素不是堆尾元素时，需要做堆化处理。

public boolean remove(Object x) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int i = indexOf(x);
        if (i < 0)
            return false;
        //维护heapIndex
        setIndex(queue[i], -1);
        int s = --size;
        RunnableScheduledFuture<?> replacement = queue[s];
        queue[s] = null;
        if (s != i) {
            //删除的不是堆尾元素，则需要堆化处理
            //先向下堆化
            siftDown(i, replacement);
            if (queue[i] == replacement)
                //若向下堆化后，i位置的元素还是replacement，说明四无需向下堆化的，
                //则需要向上堆化
                siftUp(i, replacement);
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```
