![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1612773702578-72d5ee8a-4ed4-4b4d-8b58-0d221ab2e88e.png#crop=0&crop=0&crop=1&crop=1&height=298&id=IeRUN&margin=%5Bobject%20Object%5D&name=image.png&originHeight=371&originWidth=913&originalType=binary&ratio=1&rotation=0&showTitle=false&size=97339&status=done&style=none&title=&width=733)

### park&unpark

#### 特征介绍

```java
	LockSupport 是非重入的,因为park的意思仅仅是阻塞某个线程而已, 并不是"锁",调用一次park方法,线程就被阻塞了。
	LockSupport的park阻塞, unpark唤醒的调用不需要任何条件对象, 也不需要先获取什么锁。在一定程度上降低代码的耦合度,即LuckSupport只与线程绑定, 并且被park的线程并不会释放之前获取到的锁。
	park阻塞与unpark唤醒的调用顺序可以颠倒,不会出现死锁,并且可以重复多次调用unpark; 而stop和resume方法如果顺序反了,就会出现死锁现象。

   park支持中断唤醒,但是不会抛出InterruptException异常,可以从isInterrupted(不会清除中断标记) interrupted(清除中断标记) 方法中获得中断标记。
  			 thread.isInterrupted();
        	 thread.interrupt();
```

```java

LockSupport类是Java6引入的一个类, 提供了基本的线程同步原语。
LockSupport实际上是调用了Unsafe类里的函数,归结到Unsafe里,只有两个函数:
  public native void unpark(Thread jdthead);
  public native void park(boolean isAbsolute,long time);
	isAbsolute参数 是指明时间是绝对的, 还是相对的。

unpark函数为线程提供"许可(permit)", 线程调用park函数则等待"许可"。有点像信号量,但这个'许可'是不能叠加的,'许可'是一次性的。
比如线程B,连续调用了三次unpark函数,当线程A调用park函数就使用掉这个'许可',如果线程A再次调用park,则进入等待状态。

ps: unpark函数可以先于park调用。比如线程B调用unpark函数,给线程A发一个'许可',那么当线程A调用park时,它发现已经有'许可',那么它会马上再继续执行


```

#### park&unpark 的灵活

```java

根据上面的介绍,unpark函数可以先于park调用,这正是它们的灵活之处。
一个线程它可能在别的线程unPark之前,或者之后,或者同时调用了park, 那么因为park的特性, 它可以不用担心自己的park时序问题。 否则，如果park必须要在unpark之前，就给编程带来很大的麻烦。

考虑一下,两个线程同步,要如何处理?
 在JDK 5里面，是用wait/notify/notifyAll来同步的。wait/notify机制的缺点, 比如线程B要用notify通知线程A, 那么线程B要确保线程A已经在wait调用上等待了,否则线程A可能永远都在等待。另外是调用notify, 还是notifyAll ?
 notify只会唤醒一个线程, 如果错误地有两个线程在同一个对象上wait等待,就又悲剧了,所以为了安全期间,好像只能用notifyAll。
>>> park/unpark模型真正解耦了线程之间的同步,线程之间不再需要一个Object或者其他变量来存储状态,不再需要关心对方的状态。




在JDK 5里面，是用wait/notify/notifyAll来同步的, 它没有LockSupport那样的容忍性，所以JDK7 JUC之后几乎都是采用park与unpark实现。

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610286135990-39491a87-fe70-4772-9109-45eea6fa80a4.png#crop=0&crop=0&crop=1&crop=1&height=137&id=LSKNP&margin=%5Bobject%20Object%5D&name=image.png&originHeight=137&originWidth=432&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8019&status=done&style=none&title=&width=432)
