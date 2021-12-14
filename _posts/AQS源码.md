![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610588280640-5025f3f1-42d2-4d27-bc6b-43ce809f5b8c.png#crop=0&crop=0&crop=1&crop=1&height=393&id=Sa5yk&margin=%5Bobject%20Object%5D&name=image.png&originHeight=786&originWidth=1718&originalType=binary&ratio=1&rotation=0&showTitle=false&size=181032&status=done&style=none&title=&width=859)

#### node

```java
public abstract class AbstractOwnableSynchronizer implements java.io.Serializable {
    //当前节点的序列化号
    private static final long serialVersionUID = 3737899427754241961L;

    protected AbstractOwnableSynchronizer() { }
    // 当前资源的独占资源
    private transient Thread exclusiveOwnerThread;
    // 设置当前独占资源的线程信息
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
    // 获取当前线程的独占资源的线程信息
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }

    static final class Node {
        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;
        // 因为超时或者中断,结点处于取消状态(线程已被取消),被取消的结点(线程),不应该去竞争锁
        // 处于这种状态的结点(线程)会被踢出队列,被GC回收
        static final int CANCELLED =  1;
        // 表示结点的后续结点被阻塞了,需要被唤醒(unpark)
        static final int SIGNAL    = -1;
        // 表示结点(线程)在条件队列中(处于condition休眠状态),在等待Condition唤醒
        static final int CONDITION = -2;
        // 使用共享模式下结点有可能处于这种状态,表示锁的下一次获取可以被无条件传播
        static final int PROPAGATE = -3;
        // 为0时,新节点会处于这种状态

        // waitState 一个32位的整型常量,该状态为与线程状态密切相关。
        volatile int waitStatus;
        volatile Node prev; // 前继结点
        volatile Node next; //后继结点
        volatile Thread thread;
        Node nextWaiter;
        ...
        Node() {}
        Node(Thread thread, Node mode) {
            this.nextWaiter = mode;
            this.thread = thread;
        }
        Node(Thread thread, int waitStatus) {
            this.waitStatus = waitStatus;
            this.thread = thread;
        }

        // 当前节点的前驱节点
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }
    }
}
}
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610588426533-9a39a750-3831-4315-9267-d106184e0eda.png#crop=0&crop=0&crop=1&crop=1&height=326&id=Mh4P8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=652&originWidth=1838&originalType=binary&ratio=1&rotation=0&showTitle=false&size=104258&status=done&style=none&title=&width=919)

#### 独占模式

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {

    /**
     * 此方法是独占模式下,线程获取共享资源的顶层入口。
     * 如果获取到资源,线程直接返回,否则进入等待队列,直到获取资源位置,整个过程忽略中断的影响。
     * 获取资源后,线程就可以去执行临界区代码了。
     *
     * 函数执行流程:
     *  tryAcquire()尝试直接去获取资源,如果成功则直接返回
     *  addWaiter() 该线程加入等待队列的尾部,并标记为独占模式
     *  acquireQueued() 使线程在等待队列中获取资源,一直获取到资源后才返回. 如果整个等待过程中被中断过,则返回true,否则返回false
     *  如果线程在等待过程中被中断过,它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()将中断补上。
     *
     */
    public final void acquire(int arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

    /**
     * 尝试去获取独占资源。如果获取成功,则直接返回true,否则直接返回false。
     * AQS中是一个框架,具体的资源的获取/释放方式交由自定义同步器去实现。
     */
     protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
    /**
     * 用于将当前线程加入等待队列的队尾,并返回当前线程所在的节点。
     */
    private Node addWaiter(Node mode) {
        // 以给定模式构造结点,mode有两种,EXCLUSIVE(独占)和SHARED(共享)
        Node node = new Node(Thread.currentThread(), mode);
        // 快速直接放在队尾
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        //上一步失败则通过enq入队
        enq(node);
        return node;
    }

    private Node enq(final Node node) {
        // CAS自旋,直到成功加入队尾
        for (;;) {
            Node t = tail;
            if (t == null) {
                //队列为空,创建一个为空的标志结点作为head节点,并将tail也指向它
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else { //正常流程,放入队尾
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

    /**
     *  通过tryAcquire和addWriter()该线程获取资源失败,已经被放入等待队列尾部了。
     *  主要目的: 进入等待状态休息,直到其他线程彻底释放资源唤醒自己,自己再拿到资源,然后就可以去干自己想干的事情了。
     *  类似案例: 类似去医院排队拿号(在等待队列中排队拿号(中间没有其他事干可以休息),直到拿到号在返回。)
     */
    final boolean acquireQueued(final Node node, int arg) {
        // 标记是否成功拿到资源
        boolean failed = true;
        try {
            // 标记等待过程中对否被中断过
            boolean interrupted = false;
            //"自旋"信息
            for (;;) {
                //拿到前驱
                final Node p = node.predecessor();
                // 如果前驱是head,即该节点已成老二,那么便有资格去尝试获取资源(可能是老大释放完资源唤醒自己的,也有可能被interrupt了)
                if (p == head && tryAcquire(arg)) {
                    // 拿到资源后,将head指向该节点。所以head所指的标杆结点,就是当前获取到资源的那个节点或null
                    setHead(node);
                    // setHead中的node.prev已置为null,此处在将head.next置为null,就是为了方便GC回收之前head节点
                    // 也就意味着之前拿完资源的结点出队了。
                    p.next = null;
                    failed = false;
                    // 返回等待过程中是否被中断过
                    return interrupted;
                }
                // 如果自己可以休息了,就进入waiting状态,直到被unpark()
                if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                    // 如果等待过程中被中断过,哪怕只有一次就将interrpted标记为true
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    /**
     * 用于检查状态: 看看自己是否可以真的去休息了(进入waiting(等待)状态),万一队列前面的线程都放弃了只是瞎站着。
     */
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        // 拿到前驱结点的状态
        int ws = pred.waitStatus;
        // 如果已经告诉前驱节点拿到号后通知自己一下,就可以安心休息了
        if (ws == Node.SIGNAL)
            return true;
        // 如果前驱结点状态大于0,说明前驱结点放弃了,就一直往前找,直到找到最近一个正常等待的状态,并排在它的后边。
        // 注意,哪些放弃的结点,由于被自己"加塞"到它们前边,它们就相当于形成了一个无引用连,稍后就会被GC回收
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            // 如果前驱正常,那就把前驱的状态设置SIGNAL,告诉它拿完号后通知自己一下。
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

    private final boolean parkAndCheckInterrupt() {
        // 调用park()使得线程进入waiting状态
        // park() 会让当前线程进入waiting状态,在此状态下,有两种方式唤醒该线程: "被unpark()" "被interrupt()"
        LockSupport.park(this);
        // 如果被唤醒,查看自己是不是被中断的，Thread.interrupted会清除当前线程中的中断标记位
        return Thread.interrupted();
    }

    /**
     * 独占模式下,线程释放公共资源的顶层入口。用于释放指定量的资源。如果彻底释放了(即state=0),它会唤醒等待队列里的其他线程来获取资源。
     * 需要注意的一点是: 它是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了,所以自定义同步器在设计tryRelease()的时候需要明确这一点
     */
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            // 找到头结点
            Node h = head;
            if (h != null && h.waitStatus != 0)
                // 唤醒等待队列里的下一个线程
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    /**
     * 具体的释放资源的方法由自定义的同步器来实现
     *  跟tryAcquire()一样,这个方法是需要独占模式的自定义同步器去实现的。正常来说,tryRelease()都会成功的,因为这是独占模式
     *  该线程来释放资源,那么它肯定已经拿到独占资源了,直接减掉相应量的资源即可(state-=arg),也不需要考虑线程安全的问题。
     *
     *  上面提到了,release()是根据tryRelease()的返回值来判断该线程是否已经完成了释放掉资源了
     *   所以自定义同步器在实现时,如果已经彻底释放资源(state=0)要返回true,否则返回false
     */
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }
    /**
     * 用于唤醒等待队列中下一个线程
     *  用unpark()唤醒等待队列中最前边的哪个未放弃的线程,这里我们也用s来表示。
     */
    private void unparkSuccessor(Node node) {
        // 这里,node一般为当前线程所在的节点
        int ws = node.waitStatus;
        // 置零当前线程所在的结点状态,允许失败
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        // 找到下一个需要唤醒的节点s
        Node s = node.next;
        // 如果为空或已取消
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                // 从这里可以看出,<=0的结点,都是还有效的结点
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            // 唤醒
            LockSupport.unpark(s.thread);
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611304248660-24897841-70a7-4355-ae81-350fd318fe1f.png#crop=0&crop=0&crop=1&crop=1&height=447&id=rGTfd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=661&originWidth=665&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70545&status=done&style=none&title=&width=450)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610592901413-c0760220-2a40-4029-9379-0c2af2445dd0.png#crop=0&crop=0&crop=1&crop=1&height=609&id=Fbf0b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1218&originWidth=1842&originalType=binary&ratio=1&rotation=0&showTitle=false&size=377658&status=done&style=none&title=&width=921)

#### 共享模式

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
    ....
    /**
     *  共享模式线程获取共享资源的顶层入口,用于获取指定量的资源,成功直接返回,失败进入等待队列,直到被unpark()/interrupt()并成功获取资源才返回
     *  整个过程忽略中断。
     *  tryAcquireShared() 需要自定义同步器去实现,用于获取资源,成功则直接返回,失败则通过doAcquireShared()进入等待队列,直到获取到资源为止才返回。
     *      AQS已经把返回值的语义定义好了:
     *        > 负值代表获取失败
     *        > 0代表获取成功,但没有剩余资源
     *        > 正数表示获取成功,还有剩余资源,其他线程还可以去获取。
     *
     */
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }
    /**
     * 此方法用于将当前线程加入等待队列尾部休息,直到其他线程释放资源唤醒自己,自己成功拿到相应量的资源后才返回。
     */
    private void doAcquireShared(int arg) {
        // 加入队列尾部
        final Node node = addWaiter(Node.SHARED);
        // 是否成功的标志
        boolean failed = true;
        try {
            // 等待过程中是否被中断过的标志
            boolean interrupted = false;
            for (;;) {
                //前驱
                final Node p = node.predecessor();
                // 如果到head的下一个,因为head是拿到资源的线程,此时node被唤醒,很可能是head用完资源来唤醒自己的
                if (p == head) {
                    // 尝试获取资源
                    int r = tryAcquireShared(arg);
                    // 成功
                    if (r >= 0) {
                        // 将head指向自己,还剩余资源可以再唤醒之后的线程
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        // 如果等待过程中被打断过,此时将中断补上
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                // 判断状态,寻找安全点,进入waiting状态,等着被unpark()或interrupt()
                if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head;
        // head指向自己
        setHead(node);
        // 如果还有剩余量,继续唤醒下一个邻居线程
        if (propagate > 0 || h == null || h.waitStatus < 0 ||(h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }


    /**
     * 共享模式下线程释放共享资源的入口,释放指定量的资源,如果成功释放且允许唤醒等待线程,它会唤醒等待队里的其他线程来获取资源
     */
    public final boolean releaseShared(int arg) {
        // 尝试释放资源
        if (tryReleaseShared(arg)) {
            // 唤醒后继结点
            doReleaseShared();
            return true;
        }
        return false;
    }
    /**
     * 唤醒后继
     */
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;
                    // 唤醒后继
                    unparkSuccessor(h);
                }
                else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;
            }
            // head发生变化
            if (h == head)
                break;
        }
    }
}


分析独占模式和共享模式下资源释放的问题:
    独占模式下的tryRelease()在完全释放掉资源(state=0)之后,才返回true去唤醒其他线程,这主要基于独占模式下的考量。
    共享模式下的releaseShared()则没有这种要求,共享模式实质就是控制一定量的线程并发执行,那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待节点。

    例如资源总量是13(state=13),线程A和B分别占用了5和7,那么此时剩余资源为1,线程C需要4个资源，此时就需要等待。
    A在运行过程中释放掉了2个资源量,然后tryReleaseShared(2)返回true,唤醒C线程,C线程一看只有3个仍然不够继续等待,随后B线程又释放了2个
    tryReleaseShared(2)返回true唤醒C,C一看5个够自己用了,然后C就可以跟A和B一起运行。

    如ReentrantReadWriteLock读锁的tryReleaseShared()只有在完全释放掉资源(state=0)才返回true
        所以自定义同步器可以根据实际需要决定tryReleaseShared()的返回值
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611304213493-d9ba0429-3a3e-4f38-9cbf-2c59de847c63.png#crop=0&crop=0&crop=1&crop=1&height=624&id=ueyKN&margin=%5Bobject%20Object%5D&name=image.png&originHeight=781&originWidth=494&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55509&status=done&style=none&title=&width=395)

#### 其他 API

```java
acquire()和acquireSahred()两种方法下，线程在等待队列中都是忽略中断的。
AQS也支持响应中断的,acquireInterruptibly()/acquireSharedInterruptibly()，这里相应的源码跟acquire()和acquireSahred()差不多.
```

[_https://juejin.cn/post/6844903732061159437#heading-1_](https://juejin.cn/post/6844903732061159437#heading-1)
