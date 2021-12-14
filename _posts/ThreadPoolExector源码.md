#### 构造函数

```java

 private volatile boolean allowCoreThreadTimeOut;
 private int largestPoolSize;
 private volatile int maximumPoolSize;
 private final ReentrantLock mainLock = new ReentrantLock();
 private HashSet<Worker> workers; // workers是非线程安全的,所以在addworkers的时候会加锁

 public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    // 这几个参数都是必须要有的
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();

    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
 }

使用一个整形,表示线程池的状态和线程池线程数。
	高位表示状态
    低位表示线程数
"可以深入了解下位操作"
// 这里 COUNT_BITS 设置为 29(32-3)，意味着前三位用于存放线程状态，后29位用于存放线程数
private static final int COUNT_BITS = Integer.SIZE - 3;
// 000 11111111111111111111111111111
// 这里得到的是 29 个 1，也就是说线程池的最大线程数是 2^29-1=536870911
// 以我们现在计算机的实际情况，这个数量还是够用的
private static final int CAPACITY   = (1 << COUNT_BITS) - 1; 000 11111111111111111111111111111
// 将整数 c 的低 29 位修改为 0，就得到了线程池的状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
// 将整数 c 的高 3 为修改为 0，就得到了线程池中的线程数
private static int workerCountOf(int c)  { return c & CAPACITY; }
// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS; 111 0000000..29个


```

#####

#### execute()

```java
public void execute(Runnable command) {
     if (command == null)
            throw new NullPointerException();
     int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611784627810-818db7d8-a47f-4fba-9524-f50367f2ce52.png#crop=0&crop=0&crop=1&crop=1&height=416&id=fnUlE&margin=%5Bobject%20Object%5D&name=image.png&originHeight=521&originWidth=691&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49164&status=done&style=none&title=&width=552)

#### runworker

```java

 final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) {
                /** 这里上锁的原因
                 *   执行之前要先获取锁, 避免在执行期间被其他线程中断
                 *   谁会中断的正则执行的Worker， 在InterruptIdleWorkers()方法中可以看到。
                 */
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                // shutdown 时状态是CANCEL
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611286444416-d31bf156-212c-4d60-b43a-2cfcfbe8249d.png#crop=0&crop=0&crop=1&crop=1&height=318&id=Rkt7q&margin=%5Bobject%20Object%5D&name=image.png&originHeight=431&originWidth=643&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58557&status=done&style=none&title=&width=474)

```java
对Worker对象的lock使用的代码排查
1) runWorker在执行一个task之前会获取该锁, 避免任务执行是被中断
2) 中断空闲线程的interruptIdleWorkers会获取该资源确保线程并没有在执行任务而是阻塞在getTask方法。
	 从这个这里可以得出lock的一个作用是表示,当前线程是否在执行任务,还是阻塞在等待任务。
3) interruptWorkers 方法会调用Worker对象内部的方法InterruptIfStarted来设置线程中断状态。
	通过getState()>=0来判断线程是启动了的(初始值=-1, lock=1 ,unlock=0 )


Worker为甚不能使用ReentrantLock来实现(而是直接继承AQS)

    ReentrantLock的tryAcquire方法它是允许重入的,但线程正在执行是不允许其他锁重入进来的。
	为什么不使用ReentrantLock,而是使用AQS, 另外一个目的就是为了实现不可重入的特性去反映线程现在的执行状态。

```

#### getTask()

```java
private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);
            // 1.线程池状态是STOP, TIDYING, TERMINATED
            // 2.线程池shutdown并且队列是空的
            // 满足以上两个条件之一,则工作线程数wc-1, 然后直接返回null
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }
            int wc = workerCountOf(c);

            // 允许核心工作线程对象销毁淘汰 或者工作线程数 > 最大核心线程数 corePoolSize
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
            // allowCoreThreadTimeOut默认为false

            // 1. 工作线程数 > 最大线程数 maximumPoolSize或者timed=true && timeOut==true
            // 2. 工作线程数 > 1 或者队列为空
            // 同时满足以上两个条件通过CAS把线程数减去1,同时返回null。
            // CAS把线程数减去1失败会进入下一轮循环做重试
            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }
            try {
                // 如果timed为true, 通过poll()方法做超时拉取, keepAliveTime时间内没有等待到有效任务,返回null
                // 如果timed为false, 通过take()做阻塞拉取,会阻塞到下一个有效的任务时候在返回(一般不会是null)
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }

默认的情况下核心线程数量会一直保持，即使这些线程是空闲的它也是会一直存在的，而当设置为 true 时，线程池中 corePoolSize 线程空闲时间达到 keepAliveTime 也将销毁关闭。
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611304134815-b8ec16f6-8614-45f7-9901-746fae02c4f7.png#crop=0&crop=0&crop=1&crop=1&height=374&id=bmSnn&margin=%5Bobject%20Object%5D&name=image.png&originHeight=503&originWidth=711&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60246&status=done&style=none&title=&width=529)

#### worker

```java
private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

        //构造函数，初始化AQS的state值为-1
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }
}

	firstTask用它来初始化时传入第一个任务, 这个任务可以有也可以为null。 如果这个值非空,那么线程就会在启动初期立即执行这个任务; 如果这个值为null, 那么就需要创建一个线程去执行任务列表(workQueue)中的任务。
```

##### 参考

> [_https://www.cnblogs.com/jajian/p/11442929.html_](https://www.cnblogs.com/jajian/p/11442929.html)\_ _
> [\_https://baijiahao.baidu.com/s?id=1679318315704715777&wfr=spider&for=pc_](https://baijiahao.baidu.com/s?id=1679318315704715777&wfr=spider&for=pc)
> [_https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html_](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
