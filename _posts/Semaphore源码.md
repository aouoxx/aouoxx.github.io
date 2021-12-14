```java
public class Semaphore implements java.io.Serializable {

    // 创建要传入许可次数, 默认使用非公平模式
    public Semaphore(int permits){
        sync=New NonfairSync(permits);
    }
    /**
     * 第二个构造方法可以指定时公平模式还是非公平模式,默认是非公平模式
     * Semaphore内部基于AQS共享模式,所以实现委托给了Sync类
     */
    public Semaphore(int permits,boolean fair){
        sync=fair?new FairSync(permits):new NonfairSync(permits);
    }


     abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;
        // 直接调用了父类的构造方法,这里的setState()方法,就是AQS中的资源,就是许可证的数量
        Sync(int permits) {
            setState(permits);
        }
        // 获取许可次数
        final int getPermits() {
            return getState();
        }
        // 非公平模式尝试获取许可
        final int nonfairTryAcquireShared(int acquires) {
            // 自旋模式, 获取许可
            for (;;) {
                // 获取剩余许可数量
                int available = getState();
                // 计算给完这次许可数量后的个数
                int remaining = available - acquires;
                // 如果许可不够或者可以将许可数量重置的话,返回
                if (remaining < 0 ||compareAndSetState(available, remaining) // cas方式更新剩余许可数量)
                    return remaining;
            }
        }

        // 释放许可
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                // 获取当前许可数量
                int current = getState();
                // 计算回收后的数量
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                // CAS改变许可数量成功,返回true
                if (compareAndSetState(current, next))
                    return true;
            }
        }

        final void reducePermits(int reductions) {
            for (;;) {
                // 获取当前剩余许可数量
                int current = getState();
                // 得到减完之后的许可数量
                int next = current - reductions;
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                // 如果CAS改变成功
                if (compareAndSetState(current, next))
                    return;
            }
        }

        /** 一次将剩余许可全部取走*/
        final int drainPermits() {
            for (;;) {
                int current = getState();
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }

    /**
     * 非公平锁的实现
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -2694183684443567898L;
        NonfairSync(int permits) {
            super(permits);
        }
        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
    }
    /** 公平锁的实现 */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = 2014338818796000944L;
        FairSync(int permits) {
            super(permits);
        }
		//  尝试获取许可
        //
        protected int tryAcquireShared(int acquires) {
            for (;;) {
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }


    /**
     * 获取许可
     * 先从获取一个许可看起,并且先看非公平模式下的实现。首先看acquire方法,acquire方法有几个重载,主要是下面这个方法
     */
    public void acquire(int permits) throws InterruptedException {
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireSharedInterruptibly(permits);
    }

}
```
