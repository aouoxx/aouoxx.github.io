---
layout: post
title: 观察者模式-Java提供Event机制
categories: [java, 设计模式]
description: Java 提供Event机制，Observeable observer
keywords: java, 设计模式
---

> _EventObject 事件对象，包含事件对应的数据，需要自定义类继承拓展它_
> _EventListener 事件监听器，需要定义类继承拓展它_
> _这只是 sun 的建议，其实我们完全不需要考虑 EventObject,EventListener。_ > *EventListener 只是一个标记接口，EventObject 也只是维护了一个 Source 的变量，提供 getSource 的方法*​

_\*\*\_Observer_**\_
\_**_Observeable_\*\*\_

> _这两个类才是总要的 Observable 提供一系列很有用的方法,Observer 也有一个重要的 update 方法_

```java
public class EventObserveable extends Observable {
    public void action(Object arg0){
    super.setChanged(); //setChanged是protected的方法
    	super.notifyObservers(arg0);
    }
    public void logicHandler(String name){
    	System. out.println("Object arg is ..." + name);
    }
}


```

​

```java
/**
 * 观察者,次update方法自动由observer来调用
 * 而update方法主要是调用业务方法，当然，我们也可以在这个方法中的直接业务逻辑处理，而不用
 * 调来调去需要继承observer是因为它和 Observale配套使用的
 */
public class EventObserver implements Observer {
    @Override
    public void update(Observable o, Object arg) {
        // TODO Auto-generated method stub
        ((EventObserveable)o).logicHandler(arg.toString());
    }
}
```

​

```java
/**
 * 此类可以 看做 事件源， 因为notifyEvent 就是发送事件的源头 EXACTLY! 专供外部调用，
 * 同时 notifyEvent 也是事件处理回调函数
 * 此类中，我们需要保持一个 Observeable的对象，它来代替我们一部分工作
 * 当然，首先要 完成 监听器的 添加、移除 等监听器相关操作
 *      ———— 当然，我们可以有其他方式添加监听器(参：http://rokily.iteye.com/blog/775395
 *           但是把addEventListener放在这个累里面感觉比较直接
 *  此类只是简单实现，实际中我们需要完善其功能，可以写的很复杂 ！
 */
public class EventSource {
         private EventObserveable ob = new EventObserveable();

           public void addEventListener(Observer listener) {
               ob.addObserver(listener);
           }

           public void removeEventListener(Observer listener) {
               ob.deleteObserver(listener);
           }

           public void notifyEvent(Object arg) {
               ob.action(arg);
           }
}
```

​

```java
public class Test {
          public static void main(String[] args) {
               EventSource ds = new EventSource();
               ds.addEventListener( new EventObserver());
               ds.notifyEvent( "Hello LuK ! ");
           }
}

```

​

public class Test {
public static void main(String[] args) {
EventSource ds = new EventSource();
ds.addEventListener( new EventObserver());
ds.notifyEvent( "Hello LuK ! ");
}
}
