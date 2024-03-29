---
layout: post
title: calcite自定义sql
categories: calcite
description: calcite自定义sql
keywords: calcite
---

 <meta name="referrer" content="no-referrer"/>

### 名词介绍

```java
Adapter
	适配器主要作用是把SQL查询下推到实际数据源执行.
	Calcite本身实现了一些常用的Adapter, 例如jdbc,csv,elasticsearch

SchemaFactory
	驱动类, 作用是生成schema

Schema
	模式类
    对应关系数据库里的表空间概念,逻辑上包含多张表.
    根据对SQL查询的支持程度, 分为Scannable,Filterable等多个类型。 作用是生成Table。

Table
	表类
    对应关系数据库里的二维表概念,可在其上执行SQL查询。
    主要作用是处理表接口, 把不同数据源支持的数据类型映射到通用SQL类型。

Enumerator
	枚举器
    与java中的iterator类似。作用是从数据源里取出数据。
```

适配器

```java
https://blog.csdn.net/Hadoop_SC/article/details/104592779?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&utm_relevant_index=2
```

#### sql 解析阶段

```java
SqlParser parser = SqlParser.create(sql,SqlParser.Config.DEFAULT);
SqlNode sqlNode = parser.parseStmt();

Calcite使用javacc做SQL解析, javacc根据calicite中定义的parser.jj文件,生成一系列的java代码,
生成的java代码会把SQL转换为SqlNode.

Javacc实现了一个SQL Parser, 它的功能有以下两个 (这里都是需要在jj文件中定义的)
    > 设计词法和语法, 定义SQL中具体的元素
    > 实现 词法分析器(Lexer)和语法分析器(Parser),完成对SQL的解析,完成相应的转换。
```

#### SqlDialect

```java
运算操作符
	org.apache.calcite.sql.SqlOperator
厂商
	org.apache.calcite.sql.SqlDialect.DatabaseProduct
规范
	org.apache.calcite.sql.validate.SqlConformance
系统类型
	org.apache.calcite.rel.type.RelDataTypeSystem


SqlDialect工厂
	通过java.sql.DatabaseMetaData创建org.apache.calcite.sql.SqlDialect
    org.apache.calcite.sql.SqlDialectFactoryImpl
```

#### schema

```java
Schema
  是table和function的名称空间, 它是一个可嵌套的结构
 	Schema还可以有subSchema,理论上可以无限嵌套, 但一般不会这么做。Schema可以理解成DataBase,Database下面有table。
    这样就和传统数据库的概念联系起来了,
  在Calcite中顶层的schema是root, 自定义的schema是root的subSchema,同时还可以设置defaultSchema,类似我们使用数据库时,
    使用use databases命令以后就不用在输入database名字前缀。
```

#### table

```java
table就是数据库中的表, 在table描述了字段名以及相应的类型,表统计信息,例如表有多少纪录等等。
另外重要的是数据文件的存储以及如何扫描读取数据文件。
```

```java
{
    "version": "1.0",
    "defaultSchema": "SALES",
    "schemas":
    [
        {
            "name": "SALES",
            "type": "custom",
            "factory": "org.apache.calcite.adapter.csv.CsvSchemaFactory",
            "operand":
            {
                "directory": "sales"
            }
        }
    ]
}


model文件
它描述了在数据库中有多少个Schema、每个Schema如何创建以及默认的Schema，这里的Schema可以理解成database。
defaultSchema属性设置默认Schema，schemas是数组类型，
	每一项代表一个Schema描述信息，在描述信息中有一个关键的属性factory，它是创建Schema的工厂类，
    在这个例子中factory是org.apache.calcite.adapter.csv.CsvSchemaFactory，它实现了SchemaFactory接口。
————————————————
原文链接：https://blog.csdn.net/weixin_39953236/article/details/113195755
```

### 实现自定义全表扫描

#### 自定义 schemaFactory

```java
public interface SchemaFactory {
  Schema create(
      SchemaPlus parentSchema,  // 父节点,一般为root SchemaPlus rootSchema = Frameworks.createRootSchema(true);
      String name, // schema的名字, 在model中定义的
      Map<String, Object> operand // 用于传入自定义参数
    );
}

```

### table

#### ScannableTable

```java
ScannableTable: 用于简单的全表扫描。

public interface ScannableTable extends Table {
  Enumerable<Object[]> scan(DataContext root);
}
```

```java
FilterableTable 用于谓词下推

public interface FilterableTable extends Table {
  Enumerable<Object[]> scan(DataContext root, List<RexNode> filters);
}

```

```java
ProjectableFilterableTable 既能支持谓词下推,又能支持project下推。

public interface ProjectableFilterableTable extends Table {

  Enumerable<Object[]> scan(DataContext root, List<RexNode> filters,
      int[] projects);
}
```

#### Enumerable&&Enumerator

```java
Enumerable支持linq和java的迭代器

// 返回java的迭代器
Iterator<T> it = enumerable.iterator();
// LINQ风格的迭代器
Enumerator<T> enumerator = enumerable.enumerator();

如果要使用这两种迭代器之前, 必须要实现它。AbstractEnumerable借助Linq4j实现了enumerator和iterator的转换
public Iterator<T> iterator(){ return Linq4j.enumeratorIterator(enumerator())}
// 所以我们仅需实现enumerator方法

public abstract class AbstractEnumerable<T> extends DefaultEnumerable<T> {
    public AbstractEnumerable() {
    }
    public Iterator<T> iterator() {
        return Linq4j.enumeratorIterator(this.enumerator());
    }
}


@Covariant(0)
public interface Enumerator<T> extends AutoCloseable {
  T current(); // 返回游标所指的当前纪录,current并不会改变游标位置
  boolean moveNext(); // 将游标指向下一条纪录,并获取当前纪录供current方法调用,如果没有下一条纪录返回false
  void reset();
  @Override void close();
}

```

```java
例:
 CsvEnumerator是读取csv文件的迭代器, 它还得需要一个RowConverter, 因为csv中都是String类型
 使用RowConverter转化成相应的类型。 在moreNext方法中, 有Stream和谓词下推filter部分的实现.
 这里仅关注下面几行代码
 final String[] strs = reader.readNext();
 if(strs ==null){
   this.current = null;
   return false;
 }
 ...
 this.current = rowConverter.convertRow(strings);
 return true;
```

```java
Jinq写法:
  Jinq为开发者提供轻松自然的方式在Java应用中编写数据库查询。
  我们可以把数据库当成普通的存储在集合里的对象,然后通过迭代器来访问这些对象,而代码会自动转成经过优化的数据库查询

---------------
 PreparedStatement ps = con.prepareStatement("select * "
                                             + " from Custom C "
                                             + " where C.name = ? ");
 ps.setString(1,"Alice");
 ResultSet rs = ps.executeQuery();

如果用Jinq来写的话是这样:
database.getCustomers().where( customer-> customer.getName().equals("Alice"));
```
