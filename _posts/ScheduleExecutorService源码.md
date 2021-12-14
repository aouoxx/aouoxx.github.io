#### scheduleExecutorService 方法

```java
public <V> ScheduledFuture<V> schedule(Callable<V> callable,long delay, TimeUnit unit);
public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);
   创建并执行在给定延迟后启用的一次性操作

public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
	创建并执行一个在给定初始延迟后首次启用的定期操作, 后续操作具有给定的周期
    也就是将在 initialDelay 后开始执行,然后在 initialDelay + period 后执行
    接着在 initialDelay + 2*period后执行, 以此类推

 public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
    创建并执行一个在给定初始延迟后首次启用的定期操作,随后,每次执行终止和下一次执行开始之间都存在给定的延迟。
```

#### ScheduledExecutorService 总结

```java
schedule方法被用来延迟指定时间来执行某个指定任务。
如果你需要周期性重复执行定时任务可以使用scheduledAtFixedRate或者scheduleWithFixedDelay方法
	它们不同的是前者是以固定的频率执行，后者是以相对固定的频率执行。

不管任务执行耗时是否大于间隔时间，scheduledAtFixedRate和scheduleWithFixedDelay都不会导致同一个任务并发的被执行。
        唯一不同的是scheduleWithFixedDelay是当前一个任务结束的时刻，开始结算间隔时间
        如5秒开始执行第一次任务，任务耗时3秒那么，第二次任务执行的时间是在第8秒开始

ScheduledExecutorService的实现类，是ScheduledThreadPoolExecutor。
ScheduledThreadPoolExecutor对象包含的线路数量是没有可伸缩性的，只会有固定的数量的线程。
不过你可以通过其构造函数来设定线程的优先级，来降低定时任务线程的系统占用。
```

```java
*) ScheduledThreadPoolExecutor继承自ThreadPoolExecutor, 所以它也是一个线程池,也有corePoolSize和workQueue
   但ScheduledThreadPoolExecutor特殊之处在于,自己实现了优先工作队列 DelayedWorkQueue

*) ScheduledThreadPoolExecutor实现了ScheduledExecutorService, 所以就有了任务调度的方法, 如schedule, scheduleAtFixedRate, scheduleWithFixedDelay 同时注意他们之间的区别

*) 内部类ScheduledFutureTask继承自FutureTask 实现了任务的异步执行, 并且可以获取返回结果。同时实现Delayed接口,可以通过getDelay()方法获取将要执行的时间间隔。

*) 周期任务的执行其实是调用了FutureTask的runAndReset()方法, 每次执行完不设置结果和状态。

*) DelayedWorkQueue的数据结果,是一个基于最小堆结构的优先队列, 并且每次出队时,能够保证取出的任务是当前队列中执行时间最小的任务。 同时注意一下优先级队列中的堆的顺序,堆中的顺序并不是绝对的,但是要保证子节点的值要不父节点的值要大。
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1612246119718-9a2e403f-f39e-4dae-a464-17651a303826.png#crop=0&crop=0&crop=1&crop=1&height=170&id=TQ2LI&margin=%5Bobject%20Object%5D&name=image.png&originHeight=340&originWidth=724&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28146&status=done&style=none&title=&width=362)

#### ScheduledThreadPoolExecutor

```java
public class ScheduledThreadPoolExecutor extends ThreadPoolExecutor implements ScheduledExecutorService {

	 // 提交定时任务
     public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                           long delay,
                                           TimeUnit unit) {
        // 参数校验
        if (callable == null || unit == null)
            throw new NullPointerException();
        /** 这里是一个嵌套结果,首先把用户提交的任务封装为ScheduleFutureTask */
        /** 然后调用decorateTask进行包装, 该方法是留给用户去扩展的, 默认是一个空方法 */
        RunnableScheduledFuture<V> t = decorateTask(callable,
            new ScheduledFutureTask<V>(callable,
                                       triggerTime(delay, unit)));
        /** 包装号任务以后,就进行提交了 */
        delayedExecute(t);
        return t;
    }
    // 该方法是留给用户去扩展的, 默认是一个空方法
	protected <V> RunnableScheduledFuture<V> decorateTask(
        Callable<V> callable, RunnableScheduledFuture<V> task) {
        return task;
    }

    private void delayedExecute(RunnableScheduledFuture<?> task) {
        if (isShutdown()) //如果线程池已关闭,则使用拒绝策略把提交任务拒绝掉
            reject(task);
        else {
            //与ThreadPoolExecutor不同, 这里直接把任务加入延迟队列中
            super.getQueue().add(task);  //使用DelayedWorkQueue
            if (isShutdown() && !canRunInCurrentRunState(task.isPeriodic()) &&  remove(task))
                task.cancel(false);
            else
                /**
                 * 这里增加一个worker线程,避免提交的任务没有worker去执行
                 * 原因就是该类没有像ThreadPoolExecutor一样, worker满了才放入队列
                 */
                ensurePrestart();
        }
    }

    void ensurePrestart() {
        int wc = workerCountOf(ctl.get());
        if (wc < corePoolSize)
            addWorker(null, true);
        else if (wc == 0)
            addWorker(null, false);
    }

    /**
     * 调用reExecutePeriodic方法的时候已经执行过一次任务,所以并不会触发线程池拒绝策略
     * 传入reExecutePeriodic方法的任务一定是周期性任务
     */
    void reExecutePeriodic(RunnableScheduledFuture<?> task) {
        // 线程池当前状态下能够执行任务
        if (canRunInCurrentRunState(true)) {
            /** 与ThreadPoolExecutor不同, 这里直接把任务加入到延迟队列 */
            super.getQueue().add(task); //使用DelayedWorkQueue
            /** 线程池当前状态下不能执行任务,并且成功移除任务*/
            if (!canRunInCurrentRunState(true) && remove(task))
                task.cancel(false);
            else
                ensurePrestart();
        }
    }

}
```

#### scheduledFutureTask

```java
ScheduledFutureTask类的定义可以看出, ScheduleFutureTask类是ScheduleThreadPoolExecutor类的私有内部类, 继承了FutureTask类, 并实现了RunnableScheduleFuture接口。
  也就是说, ScheduleFutureTask具有FutureTask类的所有功能, 并实现了RunnableScheduleFuture接口的所有方法。

sequenceNumber:
	任务添加到scheduledThreadPoolExector中被分配的唯一序列号,可以根据这个序列号,确定唯一的一个任务。
    如果在定时任务中,如果一个任务是周期性执行的,但是它们的sequenceNumber的值相同,则被视为同一个任务。
 outerTask:
	ScheduledFutureTask对象,实际指向当前对象本身。
    此对象的引用会被传入到周期性执行任务的ScheduleThreadPoolExecutor类的reExecutePeriodic方法中。
```

```java
 private class ScheduledFutureTask<V> extends FutureTask<V> implements RunnableScheduledFuture<V> {

     // 任务开始的时间,下一次执行任务的时间
    private long time;
    // 任务添加到ScheduledThreadPoolExecutor中被分配的唯一序列号
    private final long sequenceNumber;
    // 任务执行的时间间隔,任务执行周期
    private final long period;
    //ScheduledFutureTask对象，实际指向当前对象本身
    RunnableScheduledFuture<V> outerTask = this;
    //当前任务在延迟队列中的索引,能够更加方便的取消当前任务
    int heapIndex;

    ScheduledFutureTask(Runnable r, V result, long ns) {
        super(r, result);
        this.time = ns;
        this.period = 0;
        this.sequenceNumber = sequencer.getAndIncrement();
    }
    ScheduledFutureTask(Runnable r, V result, long ns, long period) {
         super(r, result);
         this.time = ns;
         this.period = period;
         this.sequenceNumber = sequencer.getAndIncrement();
    }
    ScheduledFutureTask(Callable<V> callable, long ns) {
         super(callable);
         this.time = ns;
         this.period = 0;
         this.sequenceNumber = sequencer.getAndIncrement();
     }

     /** 获取下次执行任务的时间距离当前时间的纳秒数 */
     public long getDelay(TimeUnit unit) {
         return unit.convert(time - now(), NANOSECONDS);
     }

     /**
      * 判断是否是周期性任务
      * 这里只要判断运行任务的执行周期不等于0就能确定为周期性任务了。
      * 因为无论period的值是大于0还是小于0，当前任务都是周期性任务。
      */
     public boolean isPeriodic() {
       return period != 0;
     }


     public void run() {
         boolean periodic = isPeriodic(); // 当前任务是否是周期性任务
         if (!canRunInCurrentRunState(periodic))
             cancel(false);
         else if (!periodic)
             ScheduledFutureTask.super.run();
         else if (ScheduledFutureTask.super.runAndReset()) {
             setNextRunTime();
             reExecutePeriodic(outerTask);
         }
     }
 }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1612249002315-3c1876b7-6df8-41a2-b4b3-afb265180d8f.png#crop=0&crop=0&crop=1&crop=1&height=374&id=pWNmx&margin=%5Bobject%20Object%5D&name=image.png&originHeight=426&originWidth=571&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32756&status=done&style=none&title=&width=501)
