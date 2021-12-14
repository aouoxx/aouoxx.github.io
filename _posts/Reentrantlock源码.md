### reentrantlock 独占锁

```java
ReentrantLock是一个比较常用的锁,它是一个互斥锁,互斥锁的含义就是只能由某个线程进行操作,其他线程等到释放锁资源之后才能竞争锁。
ReentrantLock是一个可重入锁,可以被单个线程多次使用。
ReentrantLock可以获取公平锁和非公平锁,同一个时间只能被一个线程获取,其他的线程通过FIFO的队列来管理。

ReentrantLock的实现机制:
		首先提供一个volatile修饰的state,当state=0时表明没有线程获取锁,一个线程想要获取锁时,如果state=0,则将state=1,此时线程获取到了锁。其他线程来获取锁时,会判断state,此时state不等于0,将此线程添加到FIFO队列中,并将线程阻塞,当一个线程释放锁时会通知阻塞的线程,使阻塞线程变成可执行线程来竞争锁(判断state是否为0),当一个线程已经获得锁并再次获取锁时,虽然此时state=1此时只需要将state++就好,这样这个线程就可以获取多次锁,实现机制在AbstractQueuedSynchronized的有详细的介绍。

ReentrantLock的lock机制有2种，忽略中断锁和响应中断锁，这给我们带来了很大的灵活性。
	比如：如果A、B2个线程去竞争锁，A线程得到了锁，B线程等待，但是A线程一直不返回，B线程等不及了，想中断自己，不再等待这个锁了，转而处理其他事情。这个时候ReentrantLock就提供了2种机制
		第一，B线程中断自己（或者别的线程中断它），但是ReentrantLock不去响应，继续让B线程等待，你再怎么中断，我全当耳边风（synchronized原语就是如此）
		第二，B线程中断自己（或者别的线程中断它），ReentrantLock处理了这个中断，并且不再等待这个锁的到来，完全放弃。

lock()
	如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁
tryLock()
    如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false；
tryLock(long timeout,TimeUnit unit)
	如果获取了锁定立即返回true,如果别的线程正持有锁等待参数给定的时间,在等待的过程中,如果获取了锁定,返回true;如果等待超时,返回false
lockInterruptibly()
	如果获取了锁定立即返回，如果没有获取锁定，当前线程处于休眠状态，直到或者锁定，或者当前线程被别的线程中断
```

> synchronized 和 reentrantlock 用途区别

```
synchronized原语与reentrantlock在一般情况下没有什么区别,但是在非常复杂的同步应用中,需要考虑使用ReentrantLock。
	某个线程在等待一个锁的控制权的这段时间需要中断
	需要分开处理一些wait-notify,ReentrantLock里面的Condition应用,能够控制notify哪个线程
	具有公平锁功能,每个到来的线程都将排队等候。


```

synchronized 原语与 reentrantlock 在一般情况下没有什么区别,但是在非常复杂的同步应用中,需要考虑使用 ReentrantLock。

> 某个线程在等待一个锁的控制权的这段时间需要中断
> 需要分开处理一些 wait-notify,ReentrantLock 里面的 Condition 应用,能够控制 notify 哪个线程
> 具有公平锁功能,每个到来的线程都将排队等候。

ReentraantLock 是通过一个 FIFO 的等待队列来管理获取该锁所有线程的。
“公平锁”机制: 线程依次排队获取锁；
“非公平锁”在锁是可获取状态时,不管自己是不是在队列的开头都会获取锁。

#### 使用 synchronized 加锁

```java
public class SynchronizedLockDemo {
	private Object lock;
	public void write(){
		synchronized (lock){
            System.out.println("开始写数据..");
            long begin = System.currentTimeMillis();
            //模拟很长时间
            for(;;){
                if(System.currentTimeMillis()-begin>Integer.MAX_VALUE)
                break;
            }
            System.out.println("数据写完了");
		}
	}
    public void read(){
        synchronized (lock){
            System.out.println("从buf中读取信息...");
        }
    }
}
程序进入write()方法，在write()方法没有执行完之前,read()是一直得不到执行的,
	因为锁一直被write方法占用,read()会一直处于等待状态
```

#### reentrantlock 加锁

```java
public class ReentrantLockDemo {
private ReentrantLock lock=new ReentrantLock();
public void write(){
lock.lock();
try {
System.out.println("开始写数据..");
long begin = System.currentTimeMillis();
//模拟很长时间
for(;;){
if(System.currentTimeMillis()-begin>Integer.MAX_VALUE)
break;
}
System.out.println("数据写完了");
}finally {
lock.unlock();
}
}
public void read() throws InterruptedException {
//这里可以响应中断
lock.lockInterruptibly();
try {
System.out.println("从buf中读取信息...");
}finally {
lock.unlock();
}
}
}
```

ReentrantLock 给了一种机制让我们来响应中断，让“读”能伸能屈，勇敢放弃对这个锁的等待。

Reentactlock 和 Condition 配合使用
ReentrantLock 可以与 Condition 配合使用,condition 为 ReentrantLock 锁的等待和释放提供控制逻辑。
例如,使用 ReentrantLock 加锁之后,可以通过它自身的 Condition.await()方法释放该锁,线程在此等待 Condition.signal 方法然后继续执行下去。
await 方法需要放在 while 循环中,因此,在不同线程之间实现并发控制,还需要一个 volatile 的变量,boolean 是原子性的变量。
因此,一般的并发控制的操作逻辑如下所示：
volatile boolean isProcess = false;
ReentrantLock lock = new ReentrantLock();
Condtion processReady = lock.newCondtion();
thead:run(){
lock.lock();
isProcess=true;
try{
//isProcessReady 是另外一个线程的控制变量
while(!isProcessReady){
processReady.await();//释放了 lock，在此等待 signal
}
}catch((InterruptedException e){
Thread.currentThread().interrupt();
}finally{
lock.unlock();
isProcess = false;
}
}
reentrantlock 可重入锁

可重入锁,也叫做递归锁,指的是同一线程外层函数获取锁之后,内层递归函数仍能获取还锁的代码,并不受影响。
ReentrantLock 和 synchronized 都是可重入锁

public class ReentrantLockDemoA {
public static void main(String[] args) {
final SyschronizedTest syschronizedTest = new SyschronizedTest();
new Thread(new Runnable() {
public void run() {
syschronizedTest.get();
}
}).start();

```
    final LockTest lockTest = new LockTest();
    new Thread(new Runnable() {
        public void run() {
            lockTest.get();
        }
    }).start();
}

static class SyschronizedTest{
    public synchronized  void get(){
        System.out.println("SYS->"+Thread.currentThread().getId());
        set();
    }
    public synchronized void set(){
        System.out.println("SYS->"+Thread.currentThread().getId());
    }
}
static class LockTest{
    ReentrantLock lock = new ReentrantLock();
    //在get方法中调用set()实现可重入锁
    public void get(){
        lock.lock();
        System.out.println("LOCK->"+Thread.currentThread().getId());
        set();
        lock.unlock();
    }
    public void set(){
        lock.lock();
        System.out.println("LOCK->"+Thread.currentThread().getId());
        lock.unlock();
    }
}
```

}

### ReentrantLock 原有 API

```java
public class ReentrantLock implements Lock, java.io.Serializable {
        // 创建一个 ReentrantLock ，默认是“非公平锁”。
        ReentrantLock()
        // 创建策略是fair的 ReentrantLock。fair为true表示是公平锁，fair为false表示是非公平锁。
        ReentrantLock(boolean fair)
        // 查询当前线程保持此锁的次数。
        int getHoldCount()
        // 返回目前拥有此锁的线程，如果此锁不被任何线程拥有，则返回 null。
        protected Thread getOwner()
        // 返回一个 collection，它包含可能正等待获取此锁的线程。
        protected Collection getQueuedThreads()
        // 返回正等待获取此锁的线程估计数。
        int getQueueLength()
        // 返回一个 collection，它包含可能正在等待与此锁相关给定条件的那些线程。
        protected Collection getWaitingThreads(Condition condition)
        // 返回等待与此锁相关的给定条件的线程估计数。
        int getWaitQueueLength(Condition condition)
        // 查询给定线程是否正在等待获取此锁。
        boolean hasQueuedThread(Thread thread)
        // 查询是否有些线程正在等待获取此锁。
        boolean hasQueuedThreads()
        // 查询是否有些线程正在等待与此锁有关的给定条件。
        boolean hasWaiters(Condition condition)
        // 如果是“公平锁”返回true，否则返回false。
        boolean isFair()
        // 查询当前线程是否保持此锁。
        boolean isHeldByCurrentThread()
        // 查询此锁是否由任意线程保持。
        boolean isLocked()
        // 获取锁。
        void lock()
        // 如果当前线程未被中断，则获取锁。
        void lockInterruptibly()
        // 返回用来与此 Lock 实例一起使用的 Condition 实例。
        Condition newCondition()
        // 仅在调用时锁未被另一个线程保持的情况下，才获取该锁。
        boolean tryLock()
        // 如果锁在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁。
        boolean tryLock(long timeout, TimeUnit unit)
        // 试图释放此锁。
        void unlock()
        public ReentrantLock(){
            sync = new NonfairSync();
        }
        public ReentrantLock(boolean fair){
           sync=fair? new FairSync:new NonfairSync();
        }
}
```
