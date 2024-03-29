---
layout: post
title: Apache Calcite
categories: Calcite
description: Apache Calcite
keywords: flink, Calcite
---

 <meta name="referrer" content="no-referrer"/>

### 什么是 Apache Calcite

```java
Apache Calcite 是一款开源SQL解析工具, 可以将各种SQL语句解析成抽象语法树AST(Abstract Syntax Tree)
之后通过操作AST就可以把SQL中所要表达的算法与关系体现在具体代码之中。


Calcite的生前为Optiq(即Farrago)为Java语言编写, 通过10多年的发展, 在2013年称为Apache下的顶级项目。

目前使用Calcite作为SQL解析与处理引擎的有Hive,Drill,Flink,Phoenix和Strom等,越来越多的数据处理引擎会采用
```

#### calcite 的主要功能

- _**SQL 解析**_
- _**SQL 校验**_
- _**查询优化**_
- _**SQL 生成器**_
- _**数据连接**_

#### calcite 解析 sql 步骤

```java
Parser
	Calicite通过JavaCC 将SQL解析成未经校验的AST

Validate
	校正Parser步骤中的AST是否合法,如验证SQL scheme, 字段, 函数等是否存在。
    SQL语句是否合法等,此步骤完成之后就生成了RelNode树

Optimize
	该步骤主要的作用优化RelNode树, 并将其转化成物理执行计划。
    主要涉及SQL规则优化如,基于规则优化(RBO)以及基于代价(CBO)优化。
    Optimze这一步原则上来说是可选的,通过Validate后的RelNode树,已经可以直接转化物理执行计划
    但现代的SQL解析器基本上都包括这一步,目的是优化SQL执行计划。此步得到的结果为物理执行计划。

Execute
	即执行阶段, 此节点主要做的是, 将物理执行计划转化成可在特定的平台执行的程序。
    如Hive与Flink都在此阶段将物理执行计划codeGen生成相应的可执行代码
```

#### calicite 相关组件

```java
Catalog 主要定义SQL语义相关的元数据与命名空间
SQL parser 主要是把SQL转化成AST
SQL validator 通过Catalog 来校正AST
Query optimizer 将AST转化成物理执行计划,优化物理执行计划。
SQL generator 反向将物理执行计划转化成SQL语句
```

### sql Parse

```java
Calcite 使用javaCC 作为SQL解析, javaCC 通过Calcite中定义的Parse.jj文件, 生成一系列的java代码
生成的java代码会把SQL转换成AST的数据结构 (SQL Node类型)
```
