```java
状态: 一般是一个state属性, 基本是整个工具的核心, 通常整个工具都是在设置和修改状态, 很多方法的操作都依赖于当前状态是什么。由于状态是全局共享的, 一般会被设置成volatile类型, 以保证其修改的可见性。

队列: 队列通常是一个等待的集合, 大多数以链表的形式实现。队列的采用的是悲观锁的思想,表示当前所等待的资源,状态或者条件短时间内可能无法满足。因此,它会将当前线程包装成某种类型的数据结构,扔到一个等待队列中,当一定条件满足后,再从等待队列中取出。

CAS: cas操作是最轻量的并发处理,通常我们对于状态的修改都会使用CAS操作,因为状态可能被多个线程同时修改, CAS操作保证了同一个时刻,只有一个线程能修改成功, 从而保证了线程安全, CAS操作基本是由Unsafe工具类的compareAndSwapxxx来实现的。
```

#### DelayQueue

```java
DelayQueue是一个没有边界BlockingQueue实现,加入其中的元素必需实现Delayed接口。
当生产者线程调用put之类的方法加入元素时, 会触发Delayed接口中的compareTo方法进行排序, 也就是说队列中的元素顺序是按到期时间排序的,而非他们进入队列的顺序,排在队列头部的元素是最早到期的, 越往后到期时间越晚。


消费者线程查看队列头部的元素,注意是查看不是取出。然后调用元素的getDelay()方法,如果此方法返回的值小于0或者等于0,则消费者线程会从队列中取出此元素,并进行处理。如果getDelay方法返回的值大于0,则消费者元素wait返回的时间值后,再从队列头部取出元素,此时元素应该已经到期。

DelayQueue是Leader-Follower模式的变种,消费者线程处于等待状态时, 总是等待最先到期的元素,而不是长时间的等待。
消费者线程尽量把时间花在处理任务上, 最小化空等时间,以提高线程的利用效率。



```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611110925258-afef8d4e-511c-477e-8562-d8f751cb2015.png#height=790&id=b7LUO&margin=%5Bobject%20Object%5D&name=image.png&originHeight=923&originWidth=686&originalType=binary&ratio=1&size=104321&status=done&style=none&width=587)

```java
最优的消费者线程的个数与任务启动的时间间隔存在这样的关系, 单个任务处理时间的最大值/ 相邻任务的启动时间最小间隔 = 最优线程数。如果最优线程数是小数,则取整数后加+1, 比如1.3的话,那么最优线程数应该是2.

本例中,单个任务处理时间最大值固定为2s。相邻任务的启动时间最小间隔为1s。
则消费者线程数为2/1=2。
如果消费者线程数小于此值,则来不及处理到期的任务。如果大于此值,线程太多,在调度,同步上花更多的时间,无益于改善性能。
```

#### blockingqueue

```java
public class ArrayBlockingQueue<E> implements BlockingQueue<E>{
	final ReetrantLock lock;

    public ArrayBlockingQueue<E>{
    	//...
    }
    public E take() throws InterruptedException{
    	final ReentrantLock lock = this.lock; // 为什么全局字段lock复制到一个局部变量中 ？
        lock.lockInterruptibly();
        try{
        	while(count==0)
                notEmpty.await();
            return dequeue();
        }finally {
        	lock.unlock();
        }
    }
}
```

### unsafe

```java
Java最初被设计为一种安全的受控环境。尽管如此，Java HotSpot还是包含了一个“后门”，提供了一些可以直接操控内存和线程的低层次操作。
这个后门类——sun.misc.Unsafe——被JDK广泛用于自己的包中，如java.nio和java.util.concurrent。
但是丝毫不建议在生产环境中使用这个后门。因为这个API十分不安全、不轻便、而且不稳定。这个不安全的类提供了一个观察HotSpot JVM内部结构并且可以对其进行修改。
有时它可以被用来在不适用C++调试的情况下学习虚拟机内部结构，有时也可以被拿来做性能监控和开发工具

私有的构造器
工厂方法getUnsafe()的调用器只能被Bootloader加载，否则抛出SecurityException 异常
```

#### unsafe_api

```java
 // 这个类提供了一个更底层的操作并且应该在受信任的代码中使用。可以通过内存地址
 // 存取fields,如果给出的内存地址是无效的那么会有一个不确定的运行表现。
public class Unsafe{
  private static Unsafe unsafe = new Unsafe();
  //使用私有默认构造器防止创建多个实例
  private Unsafe()
  {}
  public static Unsafe getUnsafe()
  {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null)
      sm.checkPropertiesAccess();
    return unsafe;
  }
  //分配var1字节大小的内存，返回起始地址偏移量
  public native long allocateMemory(long var1);
  //重新给var1起始地址的内存分配长度为var3字节大小的内存，返回新的内存起始地址偏移量
  public native long reallocateMemory(long var1, long var3);
  //释放起始地址为var1的内存
  public native void freeMemory(long var1);


  /* 获取字节对象中非静态方法的偏移量,这个值对于给定的field是唯一的，并且后续对该方法的调用都应该返回相同的值。
   * 参数：需要返回偏移量的field
   * 返回值：指定field的偏移量
   */
  public native long objectFieldOffset(Field field);
  /**
  * 在obj的offset位置比较integer field和期望的值，如果相同则更新。
  * 这个方法的操作应该是原子的，因此提供了一种不可中断的方式更新integer field。
  * obj: 包含要修改field的对象
  * offset: object型field的偏移量
  * 如果期望值expect与field的当前值相同,设置field的值为这个新值,如果filed被改变返回true.
  */
  public native boolean compareAndSwapInt(Object obj, long offset,int expect, int update);
  public native boolean compareAndSwapLong(Object obj, long offset,long expect, long update);
  public native boolean compareAndSwapObject(Object obj, long offset,Object expect, Object update);
  /**
   * 设置obj对象中offset偏移地址对应的long型field的值为指定值。这是一个有序或者有延迟的方法，并且不保证值的改变被其他线程立即看到。
   * obj,包含需要修改field的对象
   * offset field的偏移量
   * value 被设置的新值
   */
  public native void putOrderedInt(Object obj, long offset, int value);
  public native void putOrderedLong(Object obj, long offset, long value);
  public native void putOrderedObject(Object obj, long offset, Object value);
  //获取给定数组中第一个元素的偏移地址。
  public native int arrayBaseOffset(Class arrayClass);
  public native int arrayIndexScale(Class arrayClass);
  /**
   * 阻塞一个线程直到出现、线程被中断或者timeout时间到期。
   * timeout为0表示永不过期
   */
  public native void park(boolean isAbsolute, long time);
  /**
   * 释放被创建的在一个线程上的阻塞.这个方法也可以被使用来终止一个先前调用park导致的阻塞.
   * 这个操作操作是不安全的,因此线程必须保证是活的.
   */
  public native void unpark(Thread thread);

  /**
   * 设置obj对象中offset偏移地址对应的整型field的值为指定值。
   * 这是一个有序或者有延迟的putIntVolatile方法，并且不保证值的改变被其他线程立即看到。
   * 只有在field被volatile修饰并且期望被意外修改的时候使用才有用。
   * obj: 包含要修改field的对象
   * offset: obj中整型field的偏移量
   * value: field将被设置的新值
   */
  public native void putOrderedInt(Object obj, long offset, int value);
  public native void putOrderedLong(Object obj, long offset, long value);
  public native void putOrderedObject(Object obj, long offset, Object value);
  /**
   * 设置obj对象中offset偏移地址对应的整型field的值为指定值。支持volatile store语义
   * obj: 包含需要修改field的对象
   * offset:<code>obj</code>中整型field的偏移量
   * value:field将被设置的新值
   */
  public native void putIntVolatile(Object obj, long offset, int value);
  public native void putLongVolatile(Object obj, long offset, long value);
  public native void putObjectVolatile(Object obj, long offset, Object value);
  /**
   * 获取obj对象中offset偏移地址对应的整型field的值,支持volatile load语义。
   * obj: 包含需要去读取的field的对象
   * offset:<code>obj</code>中整型field的偏移量
  */
  public native int getIntVolatile(Object obj, long offset);
  public native long getLongVolatile(Object obj, long offset);
  public native Object getObjectVolatile(Object obj, long offset);

  /**
   * 设置obj对象中offset偏移地址对应的long型field的值为指定值。
   * obj:包含需要修改field的对象
   * offset: <code>obj</code>中long型field的偏移量
   * value:field将被设置的新值
   */
  public native void putLong(Object obj, long offset, long value);
  public native long getLong(Object obj, long offset);
}
```

#### unsafe 功能

```java
'内存管理'
    包括内存分配和释放
    主要的方法
        allocateMemory(分配内存),
        reallocateMemory(重新分配内存),
        copyMemory(拷贝内存),
        freeMemory(释放内存)
        getAddress(获取内存地址)
        addressSize
        pageSize
        getInt(获取内存地址指向的整数)
        getIntVolatile(获取内存地址指向的整数,并支持volatile语义)
        putInt(将整数写入指定内存地址)
        putIntVolatile(将整数写入指定内存地址,并支持volatile语义)
        putOrderedit(将整数写入指定内存地址,有序或者有延迟的方法)

  getXXXX和putXXX包含了各种基本类型的操作。
  利用copyMemory方法,我们可以实现一个通用的对象拷贝方法,无需在对每一个对象都实现clone方法,当然这通用的方法只能做到对象浅拷贝。

'非常规的对象实例化'
  allocateInstance()方法提供了另一种创建实例的途径。我们可以用new或者反射实例化对象,使用allocateInstance()方法可以直接生成对象实例
  无需调用构造方法和其他初始化方法。
  在对象反序列化的时候很有用,能够重建和设置final字段,而不需要调用构造方法。
'操作类,对象,变量'
   staticFieldOffset(静态域偏移),
   defineClass(定义类)
   defineAnonymousClass(定义匿名类)
   ensureClassInitialized(确保类初始化)
   objectFieldOffset(对象域偏移)
  通过上面的这些方法,我们可以获取对象的指针,通过对指针进行偏移,我们不仅可以直接修改指向的数据(即使它们是私有的)甚至可以找到JVM已经认定为垃圾,可以回收的对象
'数组操作'
  该部分包含了arrayBaseOffset(获取数组第一个元素的偏移地址),arrayIndexScale(获取数组中元素的增量地址等方法)
  arrayBaseOffset与arrayIndexScale配合起来使用，就可以定位数组中每个元素在内存中的位置
  由于Java的数组最大值为Integer.MAX_VALUE，使用Unsafe类的内存分配方法可以实现超大数组。
'多线程同步'
  锁机制和cas操作等
'挂起与恢复'
  park和unpark方法
  park挂起线程，线程将一直阻塞直到超时或者中断等条件出现
  unpark可以终止一个挂起的线程，使其恢复正常
  整个并发框架中对线程的挂起操作被封装在LockSupport类中，LockSupport类中有各种版本pack方法，但最终都调用了Unsafe.park()方法
'内存屏障'
  这部分包括了loadFence、storeFence、fullFence等方法
  loadFence() 表示该方法之前的所有load操作在内存屏障之前完成
  storeFence()表示该方法之前的所有store操作在内存屏障之前完成
  fullFence()表示该方法之前的所有load、store操作在内存屏障之前完成。
```

#### unsafe 的实例

```java
public static void main(String[] args){
    Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
    theUnsafe.setAccessible(true);
    Unsafe UNSAFE = (Unsafe) theUnsafe.get(null);
    System.out.println(UNSAFE);

        byte[] data = new byte[10];
        System.out.println(Arrays.toString(data));
        byteArrayBaseOffset = UNSAFE.arrayBaseOffset(byte[].class);

        System.out.println(byteArrayBaseOffset);
        UNSAFE.putByte(data, byteArrayBaseOffset, (byte) 1);
        UNSAFE.putByte(data, byteArrayBaseOffset + 5, (byte) 5);
        System.out.println(Arrays.toString(data));
}
sun.misc.Unsafe@6af62373
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
---修改后的输出----
[1, 0, 0, 0, 0, 5, 0, 0, 0, 0]
```

```java
'Unsafe大数组'
Integer.MAX_VALUE是数组元素个数的最大值。使用直接内存分配我们可以创建一个任意大的数组
class SuperArray {
    private final static int BYTE = 1;
    private long size;
    private long address;
    public SuperArray(long size) {
        this.size = size;
        address = getUnsafe().allocateMemory(size * BYTE);
    }
    public void set(long i, byte value) {
        getUnsafe().putByte(address + i * BYTE, value);
    }
    public int get(long idx) {
        return getUnsafe().getByte(address + idx * BYTE);
    }
    public long size() {
        return size;
    }
}

long SUPER_SIZE = (long)Integer.MAX_VALUE * 2;
SuperArray array = new SuperArray(SUPER_SIZE);
System.out.println("Array size:" + array.size()); // 4294967294
for (int i = 0; i < 100; i++) {
    array.set((long)Integer.MAX_VALUE + i, (byte)3);
    sum += array.get((long)Integer.MAX_VALUE + i);
}
这种方式的内存分配不是在堆里，不受GC的管理,使用Unsafe.freeMemory()来照管它。它也不进行任何边界检查，因此任何非法访问都有可能造成虚拟机崩溃。
```

```java
'获取类的某个对象的某个field偏移地址'
try {
    f = SampleClass.class.getDeclaredField("i");
} catch (NoSuchFieldException e) {
    e.printStackTrace();
}
long iFiledAddressShift = unsafe.objectFieldOffset(f); //获取Sample类field=i的偏移地址
SampleClass sampleClass = new SampleClass();
//获取对象的偏移地址，需要将目标对象设为辅助数组的第一个元素（也是唯一的元素）。
//由于这是一个复杂类型元素（不是基本数据类型,它的地址存储在数组的第一个元素。
//然后，获取辅助数组的基本偏移量。数组的基本偏移量是指数组对象的起始地址与数组第一个元素之间的偏移量。
Object helperArray[]    = new Object[1]; //使用数组辅助获取对象地址
helperArray[0]  = sampleClass;
long baseOffset = unsafe.arrayBaseOffset(Object[].class); //通过数组来获取对象的地址
long addressOfSampleClass = unsafe.getLong(helperArray, baseOffset); //获取对象的field=i的地址
int i = unsafe.getInt(addressOfSampleClass + iFiledAddressShift); //获取i的值

```

```java
'unsafe的内存分配'
//在堆外分配一个byte
long allocatedAddress = unsafe.allocateMemory(1L);
unsafe.putByte(allocatedAddress, (byte) 100);
byte shortValue = unsafe.getByte(allocatedAddress);
//重新分配一个long
allocatedAddress = unsafe.reallocateMemory(allocatedAddress, 8L);
unsafe.putLong(allocatedAddress, 1024L);
long longValue = unsafe.getLong(allocatedAddress);
//Free掉,这个数据可能脏掉
unsafe.freeMemory(allocatedAddress);
longValue = unsafe.getLong(allocatedAddress);
```
