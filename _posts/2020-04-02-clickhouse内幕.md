---
layout: post
title: clickhouse技术内幕
categories: [olap, clickhouse]
description: clickhouse技术内幕
keywords: clickhouse
---

 <meta name="referrer" content="no-referrer"/>

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639456263015-a0ba679b-41e4-4077-bc26-b5f6be48ff28.png#clientId=ucec0fbf4-97c7-4&from=paste&height=266&id=uc4ea83fc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=346&originWidth=731&originalType=binary&ratio=1&size=110405&status=done&style=none&taskId=ub2ccc667-64aa-48a4-bf2b-ae51980d2de&width=562.5)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639456182487-d594f1ab-fd37-45c3-ae43-2d9603fe0f8c.png#clientId=ua011a8a7-775a-4&from=paste&height=438&id=u70277df1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=461&originWidth=331&originalType=binary&ratio=1&size=27647&status=done&style=none&taskId=u370db9e7-535d-4c28-b188-3bf1fe876fb&width=314.5)

#### 分片间数据同步

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639456328299-2c90c7b7-ac04-46ef-ad75-b75b362bd0a4.png#clientId=ucec0fbf4-97c7-4&from=paste&height=251&id=u945a7432&margin=%5Bobject%20Object%5D&name=image.png&originHeight=300&originWidth=563&originalType=binary&ratio=1&size=28968&status=done&style=none&taskId=u963e5b65-ff29-4cf2-af44-5c408e433d6&width=470.5)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1639456374669-26bd6855-3991-45c6-90b4-c8676c986fff.png#clientId=ucec0fbf4-97c7-4&from=paste&height=618&id=u0ac4c82d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=812&originWidth=775&originalType=binary&ratio=1&size=134138&status=done&style=none&taskId=u40edf6f4-3298-4713-8093-3949f090253&width=589.5)

#### 分布式表 vs 本地表

```java
选择分布式表 还是本地表

分布式表
----------------------------------------------------------
     异步发送
     无一致性校验
     zk压力大
本地表
-----------------------------------------------------------
    数据通过各节点的本地表导入
    压力分散
    原子写入


数据入库优化
-------------------------------------------------------------
1) 使用复制表,保证数据原子写入和去重
2) 限制单个复制表 INSERT语句的执行频率 (建议每秒不超过1个)
3) 以相当大的数据块插入数据(默认为 1048576)
4) 将数据insert到clickhouse之前,通过分区键对数据进行分组
5) 增加入库的并发,入库性能线性提升
6) 自定义负载均衡策略, 控制数据均衡。

```

#### clickhouse 查询优化

```java
1) 小表放在join的右边
    clickhouse做join操作的时候,是将右边表写入内存,然后一行一行进行遍历和左表进行关联操作。

2) 使用子查询显式的设置数据处理的顺序
    可以提前过滤数据,减少数据量

3) 使用in替换join操作
    in相当于any left join ,相比其他的join效率高些

4) 使用字段代替join
    尤其是对于维表的join

5) 设置字段的单射属性
    类似于对数据先进行聚合,然后再进行数据关联

6) 使用Join引擎缓存表

7) 禁用分布式表的子查询,
	使用GLOBAL IN/JOIN 替换或将子查询替换分发到所有的server作为本地表(需要保证数据的完整性)

8) 使用PRWHERE 谓词下推

9) 避免使用select * from 查询

10）尽量少用或不用多表关联


```
