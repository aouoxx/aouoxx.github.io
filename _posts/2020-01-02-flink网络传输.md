```java
在本地数据交换的情况下, 两个Task实际上是同一个JVM的两个线程, Task1产生的Buffer直接被Task2使用,
当Task2处理完之后这个Buffer就会被回收到本地缓冲池中。一旦Task2的处理速度比Task2产生Buffer的速度慢, 那么缓存池中Buffer
渐渐的就会被耗尽, Task1无法申请到新的Buffer自然就会阻塞,因而会导致Task1的降速。

在网络数据交换的情况下,如果下游Task的处理速度较慢,下游Task的接收缓存池也逐渐耗尽后就无法从网络中读取新的数据,
这会导致上游Task无法将缓冲池中的Buffer发送到网络中,因此上游Task的缓冲池也会被耗尽,进而导致上游任务的降速。
为了解决网络连接阻塞导致所有Task都无法处理数据的情况,Flink 还引入了 Credit-based Flow Control 算法。

在上游生产者下游消费者之间通过"信用点"来协调发送速度, 确保网络连接永远不会阻塞, 同时, Flink的网络栈基于Netty构建,
通过Netty水位线机制也可以控制发送端的发送速率。


```

### Credit-based-Flow-Control 具体机制

```java
1) 接收端向发送端声明可用的Credit (一个可用的Buffer对应一点credit)
2) 当发送端获得X点Credit, 表明它可以向网络中发送X个buffer,
	当接收端分配了X点Credit给发送端, 表明它由X个空闲的buffer可以接收数据。
3) 只有在Credit>0 的情况下,发送端才发送buffer, 发送端每发送一个buffer, credit也相应的减少一点
4) 由于CheckpointBarrier, EndOfPartitionEvent 等时间可以被立即处理, 因为事件可以立即发送, 无需使用Credit

5) 当发送端发送buffer的时候, 它同样把当前堆积的buffer数量 (backlog size)告知接收端, 接收端根据发送端堆积的数据来申请
	floating buffer。

 这种流量控制机制可以有效的改善网络的利用率,不会因为buffer长时间停留在网络链路中进而导致整个所有的Task都无法继续处理数据
 也无法进行checkpoint。但是它的一个潜在的缺点就是增加了上下游之间的通信成本(需要发送credit和backlog信息)。

 目前版本中可以通过 taskmanager.network.credit-model: false来禁用, 但是后续应该会移除整个配置项。
```
