```java
在JAVA中一个完整的定时任务需要由Timer，TimerTask两个类来配合完成。API中是这样定义的。
Timer：一种工具，线程用其安排以后再后台线程中执行的任务。可安排任务执行或者定期重复执行。

由TimerTask：Timer安排为一次执行执行或重复执行的任务，我们可以这样理解Timer是一种定时器工具，用来在一个后台线程计划执行指定任务，而TimerTask一个抽象类，它的子类代表一个可以被Timer计划的任务。


```

#### timer

```java
 在工具类Timer中提供4个构造方法，每个构造方法都启动了计时器线程，同时Timer类可以保证多个线程可以共享Timer对象而无需进行外部同步，所以Timer类是线程安全的，但是由于每个Timer对象对应的是单个后台线程，用于顺序执行所有的计时器任务。

 一般情况下，我们的线程任务执行所消耗时间应该非常短，但是由于特殊情况导致某个定时器任务执行执行时间太长，
 那么它就会"独占"计时器的任务执行线程，其后所有的线程都必须等待它执行完，
 这就会导致任务执行的延后，使这些任务堆积在一起。
```

```java
当程序初始化完成Timer后
定时任务就会按照我们设定的时间去执行，Timer提供了schedule方法，该方法有多中重载方式来适应不同的情况，如下：
schedule(TimerTask task, Date time)
	安排在指定的时间执行指定的任务。
schedule(TimerTask task, Date firstTime, long period)
	安排指定的任务在指定的时间开始进行重复的固定延迟行。
schedule(TimerTask task, long delay)
	安排在指定延迟后执行指定的任务。
schedule(TimerTask task, long delay, long period)
	安排指定的任务从指定的延迟后开始进行重复的固定延迟执行。

同时也重载了scheduleAtFixedRate方法，scheduleAtFixedRate方法与schedule相同
		只不过他们的侧重点不同，区别后面分析。
scheduleAtFixedRate(TimerTask task, Date firstTime, long period)
	安排指定的任务在指定的时间开始进行重复的固定速率执行。
scheduleAtFixedRate(TimerTask task, long delay, long period)
	安排指定的任务在指定的延迟后开始进行重复的固定速率执行。
cancel()
    终止此计时器，丢弃所有当前已安排的任务
purge()
    从此计时器的任务队列中移除所有已取消的任务
```

#### timertask

```java
TimerTask类是一个抽象类，由Timer安排为一次执行或重复执行的任务。
它有一个抽象方法run()方法，该方法用于执行相应的计时器任务要执行的操作。
	因为每一个具体的任务都必须继承TimerTask,然后重写run方法。

boolean cancel(); 取消此计时器任务
long scheduledExecutionTime(); 返回此任务最近实际执行的安排执行时间

Timer和TimerTask类中都有一个cacel()方法
TimerTask类中的作用是：将自身的任务队列清除(一个Time对象可以执行多个Timertask任务）
Timer类中的作用是：将任务队列中的全部任务清空
```

```java
schedule(TimerTask task, Date time)
schedule(TimerTask task, long delay)
这两个方法，如果执行的计划执行时间scheduledExecutionTime<= systemCurrentTime，则task会被立即执行。
scheduledExecutionTime不会因为某一个task的过度执行而改变。

schedule(TimerTask task, Date firstTime, long period)
schedule(TimerTask task, long delay, long period)
	这两个方法与上面两个就有点儿不同的，前面提过Timer的计时器任务会因为前一个任务执行时间较长而延时。
	在这两个方法中，每一次执行的task的计划时间会随着前一个task的实际时间而发生改变，
也就是scheduledExecutionTime(n+1)=realExecutionTime(n)+periodTime。
	也就是说如果第n个task由于某种情况导致这次的执行时间过程，
	最后导致systemCurrentTime>= scheduledExecutionTime(n+1)，这是第n+1个task并不会因为到时了而执行，
	他会等待第n个task执行完之后再执行，
	那么这样势必会导致n+2个的执行实现scheduledExecutionTime发生改变
即scheduledExecutionTime(n+2) = realExecutionTime(n+1)+periodTime。
	所以这两个方法更加注重保存间隔时间的稳定。

scheduleAtFixedRate(TimerTask task, Date firstTime, long period)
scheduleAtFixedRate(TimerTask task, long delay, long period)
	在前面也提过scheduleAtFixedRate与schedule方法的侧重点不同，schedule方法侧重保存间隔时间的稳定，
	而scheduleAtFixedRate方法更加侧重于保持执行频率的稳定。
	原因如下。
在schedule方法中会因为前一个任务的延迟而导致其后面的定时任务延时，而scheduleAtFixedRate方法则不会，
如果第n个task执行时间过长导致systemCurrentTime>= scheduledExecutionTime(n+1)，
则不会做任何等待他会立即执行第n+1个task，所以scheduleAtFixedRate方法执行时间的计算方法不同于schedule，
而是scheduledExecutionTime(n)=firstExecuteTime +n*periodTime，该计算方法永远保持不变。
所以scheduleAtFixedRate更加侧重于保持执行频率的稳定。
```

#### **Timer 定时器的缺陷**

```java
Timer计时器可以
	定时(指定时间执行任务）
    延迟（延迟5S执行任务）
    周期性的执行任务(每隔1秒执行任务)  但是，Timer存在一些缺陷。

1) Timer对调度的支持是基于绝对时间的，而不是相对时间，所以它对系统时间的改变非常敏感。
2) Timer线程不捕获异常，如果TimerTask抛出了未检查异常则会导致Timer线程终止，也不会重新恢复线程，而是整个Timer线程都会取消。
同时，已经安排但尚未执行的TimerTask也不会执行了，新的任务不能被调度。
故如果TimerTask抛出未检查的异常，Timer将会产生无法预料的行为。
```

```java
----------------Timer管理时间延迟缺陷-----------------
public class TimerTask04 {
    private Timer timer;

    public long start;

    public TimerTask04(){
        this.timer = new Timer();
        start=System.currentTimeMillis();
    }

    public void timeOne(){
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("timerOne invoked , the time:"+(System.currentTimeMillis()-start));
                try {
                    java.lang.Thread.sleep(4000);
                }catch (Exception E){
                    E.printStackTrace();
                }
            }
        },1000);
    }

    public void timeTwo(){
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("timerOne invoked , the time:"+(System.currentTimeMillis()-start));

            }
        },3000);
    }

    public static void main(String[] args) {
        TimerTask04 timerTask04 = new TimerTask04();
        timerTask04.timeOne();
        timerTask04.timeTwo();
    }
}
---输出结果：
timerOne invoked , the time:1003
timerOne invoked , the time:5008
```

```java
Timer抛出异常缺陷
public class TimerTask05 {
    private Timer timer;
    public long start;

    public TimerTask05(){
        this.timer = new Timer();
        start=System.currentTimeMillis();
    }
    public void timeOne(){
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                throw new RuntimeException();
            }
        },1000);
    }
    public void timeTwo(){
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("timerOne invoked , the time:"+(System.currentTimeMillis()-start));

            }
        },3000);
    }

    public static void main(String[] args) {
        TimerTask05 timerTask05 = new TimerTask05();
        timerTask05.timeOne();
        timerTask05.timeTwo();
    }
}
timeTwo() 将得不到执行，因为定时器在执行timeOne的时候抛出异常
```
