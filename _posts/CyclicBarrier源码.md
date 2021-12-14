#### cyclicBarrier 简单总结

_**CyclicBarrier 是依靠一个计数器实现的，内部有一个 count 变量，每次调用都会减一。当一次完整的栅栏活动结束后，计数器重置，这样，就可以重复利用了。**_

```java
CyclicBarrier的思路:
	设置一个计数器, 线程每调用一次计数器,就减一, 并使用Condition阻塞线程。
    当计数器是0的时候,就唤醒所有线程,并尝试执行构造函数中的任务。由于CyclicBarrier是可重复执行的,所以需要重置计数器

CyclicBarrier还有一个重要的点,就是generation的概念, 由于每一个线程可以使用多个CyclicBarrier, 每个CyclicBarrier又都可以唤醒线程, 那么就需要代来控制, 如果代不匹配,就需要重新休眠。
  如果线程中断了, 代就会损坏(broken=true),同时唤醒该代对应的所有等待的线程,抛出BrokenBarrierException异常。
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1612315830132-cfb6e18a-47ce-45eb-98eb-84c5c94b3de4.png#crop=0&crop=0&crop=1&crop=1&height=292&id=wPxrt&margin=%5Bobject%20Object%5D&name=image.png&originHeight=301&originWidth=512&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24897&status=done&style=none&title=&width=497)

```java


public class CyclicBarrier {


	 public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }

    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException, TimeoutException {
        final ReentrantLock lock = this.lock;
        /**
         * 加锁, CyclicBarrier都有一个Lock, 想执行await方法,就必须获得这把锁。
         * 由于这把锁, CyclicBarrier在并发情况下的性能是不高的
         */
        lock.lock();
        try {
            final Generation g = generation; // 当前代
            if (g.broken) // 如果这代损坏了,抛出异常
                throw new BrokenBarrierException();
			/** 线程中断了,抛出异常。*/
            if (Thread.interrupted()) {
                breakBarrier(); // 将损坏状态设置为true, 并通知其他阻塞在此栅栏上的线程
                throw new InterruptedException();
            }
			// 获取下标
            int index = --count;
            // 如果是0, 说明到头了
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    // 执行栅栏任务
                    if (command != null)
                        command.run();
                    ranAction = true;
                    /** 更新一代, 将count重置, 将generation重置 */
                    nextGeneration();
                    return 0; //结束
                } finally {
                    if (!ranAction) // 如果执行栅栏任务的时候失败了,就将栅栏失效
                        breakBarrier();
                }
            }
            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {
                    if (!timed) // 如果没有时间限制,则直接等待,直到被唤醒
                        trip.await();
                    else if (nanos > 0L) //如果有时间限制,则等待指定时间
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    /**
                     * CyclicBarrier中只要有一个线程中断了,其余线程也会抛出中断异常。
                     * 并且这个CyclicBarrier就不能在使用了
                     *
                     * g==generation >> 当前代   !g.broken >>> 没有损坏
                     */
                    if (g == generation && ! g.broken) {
                        breakBarrier(); // 让栅栏失效
                        throw ie;
                    } else {
                        // 上面条件不满足, 说明这个线程不是这代的
                        // 就不会影响当前这代栅栏执行逻辑。所以,就打个标记就好了
                        Thread.currentThread().interrupt();
                    }
                }
				/**
                 * 当有任务一个线程中断, 会调用breakBarrier方法
                 * 就会唤醒其他线程,其他线程醒来后,也要抛出异常
                 */
                if (g.broken)
                    throw new BrokenBarrierException();
				/**
                 * g != generation >>> 正常换代了
                 * 一切正常,返回当前线程所在栅栏的下标
                 * 如果g==generation 说明还没有换代, 那为什么为醒了呢 ?
                 * 因为一个线程可以使用多个栅栏, 当别的栅栏唤醒这个线程,就会走到这里,所以需要判断是否是当前待
                 * 正是因为这个原因,才需要generation 来保证正确
                 */
                if (g != generation)
                    return index;
				/** 如果有时间限制,且时间小于等于0, 销毁栅栏,并抛出异常*/
                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();
        }
    }

    private void breakBarrier() {
        // 设置状态
        generation.broken = true;
        // 恢复正在等待进入屏障的线程数量
        count = parties;
        trip.signalAll(); // 唤醒所有线程
    }

    private void nextGeneration() {
        // signal completion of last generation
        trip.signalAll();
        // set up next generation
        count = parties;
        generation = new Generation();
    }
}
```

#### 内部类 Generation

```java
/**
 * CyclicBarrier类存在一个内部类Generation, 每一次使用的CycBarrier可以当成Generation的实例
 * CyclicBarrier是可以复用的, 每次所有的线程都通过了栅栏, 就表示一代过去, 就像我们的新年一样。
 *   比如:当所有人跨过了元旦, 日历就更新了。
 */
 private static class Generation{
 	 boolean broken = false;
 }
```

#### 和 countDownLatch 的区别

```java
CountDownLatch只能使用一次,就over了,yclicBarrier 能使用多次，可以说功能类似，CyclicBarrier 更强大一点。
并且 CyclicBarrier 携带了一个在栅栏处可以执行的任务, 更加灵活。
```
