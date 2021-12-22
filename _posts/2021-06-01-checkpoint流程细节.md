```java
Flink 任务从checkpoint或savepoint处,恢复的整体流程概述:

  1) 客户端提供checkpoint或savepoint的目录
  2) JM从给定的目录中找到_metadata文件 (checkpoint的元数据文件)
  3) JM解析元数据文件,做一些校验, 将信息写入到zk中,然后准备从这一次checkpoint中恢复任务
  4) JM拿到所有算子对应的state, 给各个subtask分配 StateHandler 状态句柄
  5) TM启动时, 也就是StreamTask的初始化阶段会创建KeyedStateBackend && OperatorStateBackend
  6) 创建过程中会根据JM分配给自己的StateHandle从hdfs上恢复State

Flink任务从checkpoint恢复不只是说TM去dfs拉状态文件即可, 需要JM先给各个TM分配state
  (由于涉及到修改并发)


```
