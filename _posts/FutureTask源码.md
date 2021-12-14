#### Future

```java
public interface Future<V>{
	// 取消任务
    boolean cancel(boolean mayInterruptIfRunning);
    // 判断任务是否已经取消
    boolean isCancelled();
    // 判断任务是否结束(执行完或取消)
    boolean isDone();
    // 阻塞式获取任务执行结果
    V get() throws InterruptedException, ExecutionException;
    // 支持超时获取任务执行结果
    V get(long timeout,TimeUnit unit) throws InterruptedException,ExecutionException,TimeoutException;

}
```

#### FutureTask

```
FutureTask是一个支持取消行为的异步任务执行器。该类实现了Future接口的方法如:
   1) 取消任务执行
   2) 查询任务是否执行完成
   3) 获取任务执行结果("get"任务必须得执行完成才能获取结果,否则会阻塞直到任务完成)

FutureTask实现了Runnable接口和Future接口,因此FutureTask可以传给到线程对象Thread或Executor(线程池)来执行。
如果在当前线程中需要执行比较耗时的操作，但又不想阻塞当前线程时，可以把这些作业交给FutureTask，另开一个线程在后台完成，当前线程将来需要时，就可以通过FutureTask对象获得后台作业的计算结果或者执行状态。
```

```java



```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1612741713399-a2558681-1994-4573-9fd7-55f21977b213.png#crop=0&crop=0&crop=1&crop=1&height=180&id=gOC8E&margin=%5Bobject%20Object%5D&name=image.png&originHeight=211&originWidth=581&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17998&status=done&style=none&title=&width=497)

#### waitnode

```java
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```
