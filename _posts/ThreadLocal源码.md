![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1611872041838-781f0efb-a058-41f9-ac37-6e33dbf424a6.png#crop=0&crop=0&crop=1&crop=1&height=232&id=dNeDR&margin=%5Bobject%20Object%5D&name=image.png&originHeight=251&originWidth=561&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14727&status=done&style=none&title=&width=518)

```
ThreadLocal底层是通过ThreadLocalMap 整个静态内部类来存储数据的,
ThreadLocalMap 就是一个键值对的 Map，
		它的底层是 Entry 对象数组，Entry 对象中存放的键是 ThreadLocal 对象，值是 Object 类型的具体存储内容 。
```

​

_**ThreadLocalMap 是 Thread 类的一个属性即\*\***ThreadLocalMap 是属于线程的\*\*_
_**​**_

####

```java
ThreadLocal 很多地方叫做线程本地变量,有些地方叫做线程本地存储。ThreadLocal为变量在每个线程中创建一个副本, 那么每个线程可以访问自己的副本变量。

	假如每个线程中都有一个connect变量, 各个线程之间对connect变量的访问实际上是没有依赖关系的, 即一个线程不需要关心其他线程是否为这个connect进行了修改的。ThreadLocal在每个线程中对该变量会创建一个副本, 即每个线程内都会有一个该变量,且在线程内部任何地方都可以使用, 线程之间互不影响, 这样一来就不存在线程安全问题, 也不会严重影响程序执行性能。
	但是要注意, 虽然ThreadLocal能够解决上面说的问题, 但是由于在每个线程中都创建了副本, 所以要考虑它对资源的消耗, 比如内存的占用会比不使用ThreadLocal要大。



```

#### api 介绍

```java
T get()
    返回当前线程的此线程局部变量的副本中的值。

protected T	initialValue()
	返回此线程局部变量的当前线程的"初始值"。

void remove()
	删除此线程局部变量的当前线程的值。

void set(T value)
	将当前线程的此线程局部变量的副本设置为指定的值。

static <S> ThreadLocal<S>	withInitial(Supplier<? extends S> supplier)
	创建线程局部变量。

```

```java
public class ThreadLocal<T> {

	 public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        //如果获取成功,则返回value值, 如果map为空,则调用setInitialValue方法返回value
        if (map != null) {
            // 然后接着下面获取到<key,value>键值对, 注意这里获取键值对传进去的是this, 而不是当前线程t。
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
    /**
     * getMap(t) 方法获取到一个map， map的类型为ThreadLocalMap。
     * getMap中是调用当前线程t, 返回当前线程t中的一个成员变量 threadLocals.
     * 成员threadLocals是什么, ThreadLocal.ThreadLocalMap threadLocals=null; 实际上就是一个
     * ThreadLocalMap,这个类型是ThreadLocal类的一个内部类。
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    /**
     * setInitialValue的具体实现:
     *    如果map不为空,就设置键值对, 为空,再创建Map
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

[_https://blog.csdn.net/qq_35029061/article/details/86495625_](https://blog.csdn.net/qq_35029061/article/details/86495625)

#### threadLocal 为每个线程创建副本

```java
ThreadLocal是如何为每个线程创建变量的副本的:

	首先, 在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals, 这个threadLocals就是用来存储实际的变量副本。键值为当前ThreadLocal变量, value为变量副本(即 T类型的变量)。

    初始时, 在Thread里面,ThreadLocals为空, 当通过ThreadLocal变量调用get()方法或set()方法,就会对Thread类中的threadLocals进行初始化,并且以当前ThreadLocal变量为键值, 以ThreadLocal要保存的副本变量value,存到threadLocals。

    然后在当前线程里面,如果要使用副本变量,就可以通过get()方法在ThreadLocals里面查找。
    	1) 实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的。
        2) 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象,因为每个线程中可有多个threadLocal变量,就像下面代码中的LongLocal和StringLocal
        3) 在进行get之前,必须先set, 否则会报空指针异常。
    如果没有先set的话,即在map中查找不到对应的存储, 则会通过调用setInitialValue方法返回i, 而在setInitialValue方法中,有一个语句是T value = initialValue,默认情况下, initialValue方法返回的是null。

```

#### threadLocal 源码

```java
public class ThreadLocalDemo {

    /** 重写了initialValue方法 */
    private static ThreadLocal<Long> threadLocalLong = new ThreadLocal<Long>(){
        @Override
        public Long initialValue(){
            return new Long(34);
        }
    };

    private static ThreadLocal<String> threadLocalString = new ThreadLocal<String>(){
        @Override
        public String initialValue(){
            return new String("ssgao");
        }
    };

    public void set(){
        threadLocalLong.set(Thread.currentThread().getId());
        threadLocalString.set(Thread.currentThread().getName());
    }

    public Long getLong(){
        return threadLocalLong.get();
    }

    public String getString(){
        return threadLocalString.get();
    }
}

```

#### threadLocal 内存泄漏

```java
ThreadLocal中内存泄漏是指ThreadLocalMap中的Entry中的key为null,
```

##### 预防内存泄漏

```java
ThreadLocal源码中其实已经对内存泄漏问题做了很多优化,
	在set, get, remove方法中都会对key为null, 但value不为null的Entry进行value置null操作, 使得value引用为null, 可达性失败, 在gc时可以回收value为null的内存。

在日常使用的时候, 最好在最后使用完ThreadLocal后, 记得remove一次。

因为如果不remove, 当一次gc执行, 这个value就会造成内存泄漏直到当前线程结束。
(线程结束, ThreadlLocalMap会被置为null, 而ThreadLocalMap中的Entry自己也就不可达, 会被回收, 一切都被回收)

```

#### threadLocal 解决 dataFormat 问题

```java
{
	public static final ThreadLocal<DateFormat> rfcFormat = new ThreadLocal<DateFormat>(){
    	@Override
        public DateFormat initialValue(){
        	DateFormat result = new SimpleDateFormat(RFC1123_PATTERN, Locale.US);
            result.setTimeZone(GMT_ZONE);
            return result;
        }
    }
}


弊端:
  threadlocal在线程数不固定的场景下,如何使用,在创建线程时,为线程的threadLocals赋值。
```

> [_https://hongjiang.info/simpledateformat-performance/_](https://hongjiang.info/simpledateformat-performance/)
