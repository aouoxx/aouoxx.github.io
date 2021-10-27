---
layout: post
title: completableFuture对象
categories: [java, 多线程]
description: completableFuture对象
keywords: java, 多线程
---

```java
Future接口的局限性: 很难直接表述多个Future结果之间的依赖性.
实际开发中,我们经常需要达成以下目的:

1> 将两个异步计算合并为一个,这两个异步计算相互独立,同时第二个又依赖于第一个的结果.
2> 等待Future集合中的所有任务都完成
3> 仅等待Future集合中最快结束的任务完成(有可能因为它们试图通过不同的方式计算同一个值),并返回它的结果
4> 通过编程方式完成一个Future任务的执行(即以手工设定异步操作结果的方式)
5> 应对Future的完成事件(即当Future的完成事件发生时会受到通知,并能使用Future计算的结果进行下一步操作,不只是简单的进行阻塞等待操作的结果)


```

```java
completableFuture对象
   CompletableFuture.compleatedFuture是一个辅助方法,用来返回一个计算好的CompletableFuture
   public static <U> CompletableFuture<U> completedFutore(U value);

>>>下面四个方法可以用来为一段异步执行的代码创建CompletableFuture对象
   > public static CompletableFuture<Void> 	runAsync(Runnable runnable)
   > public static CompletableFuture<Void> 	runAsync(Runnable runnable, Executor executor)
   > public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
   > public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)

   以Async结尾并且没有指定Exector的方法会使用ForkJoinPool.commonPool()作为它的线程池执行异步代码.
    runAsync方法:   它以'Runnable函数式接口'类型为参数,所以CompletableFuture的计算结果为空
   supplyAsync方法: 它以Supplier<U>函数式接口类型为参数,CompletableFuture的计算结果类型为U
   -------------------------------
   方法的参数类型都是函数式接口,所以可以使用lambda表达式实现异步任务.
   CompletableFuture<String> future =
        CompletableFuture.supplyAsync(()->{
            //长时间的计算任务
            return ".00";
        });



>>> 计算结果完成时的处理
    当CompletableFuture的计算结果完成,或者抛出异常的时候,可以执行特定的Action
    public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
    public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
    public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
    public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
    可以看到Action的类型时BiConsumer<? super T, ? super Throwable>它可以处理正常计算结果,或者异常情况。
    方法不以Async结尾,意味着Action使用相同的线程执行,而Async可能会使用其他线程执行( 如果是使用相同的线程池也可能会被同一个线程选中执行)
    ------------------------------

    public class BasicFuture{
       private static Random rand = new Random();
       private static long t = System.currentTimeMillis();

       static int getMoreData(){
           try{
               TimeUnit.SECONDS.sleep(3);
           }catch(InterruptedException e){
               e.printStackTrace();
           }
           return rand.nextInt(1000);
       }
      public static void main(String[] args) throws Exception{
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(BasicFuture::getMoreData);

       Future<Integer> f = futrue.whenComplete((v,e) ->{
               System.out.println(v);
               System.out.println(e);
         });
           System.out.println(f.get());
      }
    }
>>> 转换信息
    CompletableFuture,我们可以不必等待一个计算完成而阻塞这调用线程,我们可以告诉CompletableFutrue当计算完成的时候执行某个Function,还可以串联起来
    CompletionStage是一个接口,从命名上看得知是一个完成的阶段,它里面的方法也标明是在某个运行阶段得到了结果之后要做的事情。
    >public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);
    >public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn);
    >public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn,Executor executor);
    上面方法首先说明一下以Async结尾的方法都是可以异步执行的
     *) 如果指定了线程池,会在指定的线程池中执行,如果没有指定,默认会在ForkJoinPool.commonPool()中执行.
     *) 关键的入参只有一个Function,它是函数式接口,所以使用Lamdba表示起来更加优雅,它的入参是上一个阶段的计算后的结果,返回值是经过转化后结果。
    @Test
    public voud thenApply(){
        String result = CompletableFufuture.supplyAsync(()->"hello").thenApply(s->s+"world").join();
        System.out.println(result);
    }
    //计算的结果为hello world;


>>> 结果计算
    只对结果执行Action,而不返回新的计算值,因此计算值为Void
    > public CompletionStage<Void> thenAccept(Consumer<? super T> action);
    > public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
    > public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action,Excutor executor);
    thenAccept是针对结果进行消耗,因为他的入参是Consumer,接口只有输入参数,没有返回值
    @Test
    public void thenAccept(){
        CompletableFuture.supplyAsync(()->"hello").thenAccept(s->System.out.println(s+"world"));
    }

```
