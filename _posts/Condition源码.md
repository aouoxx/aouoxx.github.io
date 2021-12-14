### condition 的特性

```java
条件(也称为条件队列或条件变量)为一个线程暂停执行("等待")直到另一线程通知某些状态条件现在可能为真提供一种方法。

由于对该共享状态信息的访问发生在不同的线程中,因此必须对其进行保护,因此某种形式的锁与该条件相关联。
等待条件提供的提供的关键属性是它自动释放关联的锁并挂起当前线程,就像Object.wait一样。

Condition实例从本质上看需要绑定到锁, 要获取特定Lock实例的Condition实例, 需要使用其newCondition()方法。
```

**每创建一个 Condition 对象就会对应一个 Condtion 队列, 每一个调用了 Condition 对象的 await()方法的线程都会被包装成 Node 扔进一个条件队列中。** 
![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611811981328-87e9b25d-2c24-4bc0-bcd6-360ac734f31f.png#crop=0&crop=0&crop=1&crop=1&height=195&id=XPid7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=281&originWidth=1005&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32640&status=done&style=none&title=&width=697)

```java
包装成的Node信息和AQS中的同步队列(CLH)的节点使用的是相同的类。
条件队列是一个单向链表,在该链表中我们使用nextWaiter属性来串联链表。但是,就像在同步队列中不会使用nextWaiter属性来串联链表一样,在条件队列中也不会使用到prev, next等属性, 它们的值都为null。

lock.newCondition()
    每创建一个Condition对象,就会对应一个Condition队列(条件队列),每一个调用了Condition对象的await方法的线程都会包装成Node扔进一个条件队列中。
```

#### 条件队列&&同步队列

```java
一般情况下,等待锁的同步队列和条件队列是相互独立的, 彼此之间并没有任何关系。
但是, 当我们调用某个条件队列的signal方法时, 会将某个或所有等待这个条件队列中的线程唤醒, 被唤醒的线程和普通线程一样需要竞争锁, 如果没有抢到, 则同样要被加到等待锁的同步队列中去,此时节点就从条件队列中被转移到同步队列中
```

### condition 源码

#### condition

```java
public interface Condition {
  /**
   * 使当前线程加入 await() 等待队列中，并释放当锁，与Object.wait()类似。
   * 当前线程会一直阻塞,直到他线程调用signal()会重新请求锁。
   */
  void await() throws InterruptedException;

  //调用该方法的前提是，当前线程已经成功获得与该条件对象绑定的重入锁，否则调用该方法时会抛出   IllegalMonitorStateException。
  //调用该方法后，结束等待的唯一方法是其它线程调用该条件对象的signal()或signalALL()方法。
  //等待过程中如果当前线程被中断，该方法仍然会继续等待，同时保留该线程的中断状态。
  void awaitUninterruptibly();

  // 调用该方法的前提是，当前线程已经成功获得与该条件对象绑定的重入锁，否则调用该方法时会抛出IllegalMonitorStateException。
  //nanosTimeout指定该方法等待信号的的最大时间（单位为纳秒）。若指定时间内收到signal()或signalALL()则返回nanosTimeout减去已经等待的时间；
  //若指定时间内有其它线程中断该线程，则抛出InterruptedException并清除当前线程的打断状态；若指定时间内未收到通知，则返回0或负数。
  long awaitNanos(long nanosTimeout) throws InterruptedException;

  /**
   * await()基本一致，唯一不同点在于，指定时间之内(await()会一直阻塞)没有收到
   *     signal()
   *     signalAll()
   *     线程中断信号
   * 该方法会返回false,否则会返回true
   */
  boolean await(long time, TimeUnit unit) throws InterruptedException;

  //适用条件与行为与awaitNanos(long nanosTimeout)完全一样，唯一不同点在于它不是等待指定时间，而是等待由参数指定的某一时刻。
  boolean awaitUntil(Date deadline) throws InterruptedException;

  //唤醒一个在 await()等待队列中的线程。与Object.notify()相似
  void signal();

  //唤醒 await()等待队列中所有的线程。与object.notifyAll()相似
  void signalAll();

}

```

#### conditionobject#await()

```java


 public class ConditionObject implements Condition, java.io.Serializable {
     private static final long serialVersionUID = 1173984872572414699L;

     private transient Node firstWaiter; //队列头部节点
     private transient Node lastWaiter; //队列尾部节点

     /**  Creates a new {@code ConditionObject} instance.*/
     public ConditionObject() { }

     public final void await() throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        // 添加节点到条件队列中
        Node node = addConditionWaiter();
         // 释放当前线程所占用的锁，保存当前的锁状态
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        // 如果当前队列不在同步队列中，说明刚刚被await, 还没有人调用signal方法，
        // 则直接将当前线程挂起
        while (!isOnSyncQueue(node)) {
            LockSupport.park(this); // 线程挂起的地方
            // 线程将在这里被挂起，停止运行
            // 能执行到这里说明要么是signal方法被调用了，要么是线程被中断了
            // 所以检查下线程被唤醒的原因，如果是因为中断被唤醒，则跳出while循环
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
       // 线程将在同步队列中利用进行acquireQueued方法进行“阻塞式”争锁，
       // 抢到锁就返回，抢不到锁就继续被挂起。因此，当await()方法返回时，
       // 必然是保证了当前线程已经持有了lock锁
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // 移除非CONDITION的节点
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }

    /** 封装一个节点将该节点放入条件队列中*/
   private Node addConditionWaiter() {
        Node t = lastWaiter;
        // 如果尾节点被cancel了，则先遍历整个链表，清除所有被cancel的节点
        if (t != null && t.waitStatus != Node.CONDITION) {
            unlinkCancelledWaiters();
            t = lastWaiter;
        }
        // 将当前线程包装成Node扔进条件队列
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        // 如果当前节点为空值那么新创建的node节点就是第一个等待节点
        if (t == null)
            firstWaiter = node;
        // 如果当前节点不为空值那么新创建的node节点就加入到当前节点的尾部节点的下一个
        else
            t.nextWaiter = node;
        lastWaiter = node; // 尾部节点指向当前节点
        return node; // 返回新加入的节点
    }

    /**
     * 如果入队时发现尾节点已经取消等待了, 那么我们就不应该接在它后面, 通过调用unlinkCancelledWaiters来
     * 清除哪些已经取消等待的线程 (条件队列从头部进行遍历的, 同步队列是从尾部开始遍历的)
     */
    private void unlinkCancelledWaiters() {
        // 获取队列的头节点
        Node t = firstWaiter;
        Node trail = null;
        // 当前节点不为空
        while (t != null) {
           // 获取下一个节点
            Node next = t.nextWaiter;
            // 如果当前节点不是条件节点
            if (t.waitStatus != Node.CONDITION) {
                // 在队列中取消当前节点
                t.nextWaiter = null;
                if (trail == null)
                    // 队列的头节点是当前节点的下一个节点
                    firstWaiter = next;
                else
                    // trail的 nextWaiter 指向当前节点t的下一个节点
                    // 因为此时t节点已经被取消了
                    trail.nextWaiter = next;
                    // 如果t节点的下一个节点为空那么lastWaiter指向trail
                if (next == null)
                    lastWaiter = trail;
            }
            else
                // 如果是条件节点 trail 指向当前节点
                trail = t;
            // 循环赋值遍历
            t = next;
        }
    }

    /** 释放当前线程所占用的锁 */
    final int fullyRelease(Node node) {
         boolean failed = true;
         try {
             int savedState = getState();
             // 如果释放成功
             if (release(savedState)) {
                 failed = false;
                 return savedState;
             } else {
                 throw new IllegalMonitorStateException();
             }
         } finally {
             if (failed)
                 // 节点的状态被设置成取消状态，从同步队列中移除
                 node.waitStatus = Node.CANCELLED;
         }
     }

     public final boolean release(int arg) {
        // 尝试获取锁，如果获取成功，唤醒后续线程
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
 }
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611863614559-da3c925f-ceec-4754-8be3-150bc58ed308.png#crop=0&crop=0&crop=1&crop=1&height=126&id=ddn3g&margin=%5Bobject%20Object%5D&name=image.png&originHeight=151&originWidth=845&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25791&status=done&style=none&title=&width=705)

#### condition#中断处理

```java
   /**
    *  检查是否有中断, 如果在发出信号之前被中断, 则返回THROW_IE
    *  在发出信号之后, 则返回REINTERRUPT, 如果没有被中断,则返回0
    */
   private int checkInterruptWhileWaiting(Node node) {
        return Thread.interrupted() ?
            (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
    }
   /**
    * 只要一个节点的waitStatus是Node.CONDITION, 说明它还没有被signal过。
    *
    */
   final boolean transferAfterCancelledWait(Node node) {
        if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
            enq(node);  //加入同步队列
            return true;
        }
        //
        while (!isOnSyncQueue(node))
            Thread.yield();
        return false;
    }
```

#### conditionobject#signal()

_**调用 signal()方法会从当前条件队列中取出第一个没有被 cancel 的节点添加到 sync 队列的末尾** _

```java
public final void signal() {
 // getExclusiveOwnerThread() == Thread.currentThread(); 当前线
 // 程是不是独占线程
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 获取第一个阻塞线程节点
    Node first = firstWaiter;
    // 条件队列是否为空
    if (first != null)
        doSignal(first);
}
// 遍历整个条件队列，找到第一个没有被canceled的节点，并将它添加到条件队列的末尾
// 如果条件队列里面已经没有节点了，则将条件队列清空
private void doSignal(Node first) {
    do {
        // 将firstWaiter指向条件队列队头的下一个节点
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        // 将条件队列原来的队头从条件队列中断开，则此时该节点成为一个孤立的节点
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611863241450-82ef83b3-0bd5-478b-a8d9-089f3565e4d6.png#crop=0&crop=0&crop=1&crop=1&height=187&id=nVltS&margin=%5Bobject%20Object%5D&name=image.png&originHeight=211&originWidth=829&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30235&status=done&style=none&title=&width=734)

```java
public final void signalAll() {
 // getExclusiveOwnerThread() == Thread.currentThread(); 当前线
 // 程是不是独占线程
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 获取第一个阻塞线程节点
    Node first = firstWaiter;
   // 条件队列是否为空
    if (first != null)
        doSignalAll(first);
}
// 移除并转移所有节点
private void doSignalAll(Node first) {
    // 清空队列中所有数据
    lastWaiter = firstWaiter = null;
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;
        transferForSignal(first);
        first = next;
    } while (first != null);
}

// 将条件队列中的节点一个一个的遍历到同步队列中
final boolean transferForSignal(Node node) {
    // 如果该节点在调用signal方法前已经被取消了，则直接跳过这个节点
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
   // 利用enq方法将该节点添加至同步队列的尾部
    Node p = enq(node);
    // 返回的是前驱节点，将其设置SIGNAL之后，才会挂起
    // 当前节点
    int ws = p.waitStatus;
    // 前节点的状态>0 即为cancelled状态,或者前节点的状态设置为singal失败
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread); // 唤醒当前线程
    return true;
}
```

### 源码参考

> [_https://www.toutiao.com/i6922284247281680904/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1611724610&app=news_article&utm_source=weixin&utm_medium=toutiao_android&use_new_style=1&req_id=202101271316490102040230311E04CDFD&share_token=44f8f313-2295-4e9c-8845-7f3015e7954f&group_id=6922284247281680904_](https://www.toutiao.com/i6922284247281680904/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1611724610&app=news_article&utm_source=weixin&utm_medium=toutiao_android&use_new_style=1&req_id=202101271316490102040230311E04CDFD&share_token=44f8f313-2295-4e9c-8845-7f3015e7954f&group_id=6922284247281680904)
