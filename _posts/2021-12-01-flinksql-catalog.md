---
layout: post
title: flinksql-catalog
categories: flink
description: flinksql-catalog
keywords: flink
---

 <meta name="referrer" content="no-referrer"/>

#### CatalogManager

![image.png](https://cdn.nlark.com/yuque/0/2022/png/659846/1641011840861-b71d7947-21a4-4ee2-90c0-4c3df6bbd471.png#clientId=u3fe9fb81-d90f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=189&id=u39272472&margin=%5Bobject%20Object%5D&name=image.png&originHeight=378&originWidth=1426&originalType=binary&ratio=1&rotation=0&showTitle=false&size=148748&status=done&style=none&taskId=u2d9176dd-d121-4a33-8b0b-919f8db8e8e&title=&width=713)

```java
public final class CatalogManager {

    private final Map<String, Catalog> catalogs;
    private final Map<ObjectIdentifier, CatalogBaseTable> temporaryTables;
    private String currentCatalogName;
    private String currentDatabaseName;

    private DefaultSchemaResolver schemaResolver;
    private final String builtInCatalogName;
    private final DataTypeFactory typeFactory;

    private CatalogManager(String defaultCatalogName, Catalog defaultCatalog, DataTypeFactory typeFactory) {
        catalogs = new LinkedHashMap<>();
        catalogs.put(defaultCatalogName, defaultCatalog);
        currentCatalogName = defaultCatalogName;
        currentDatabaseName = defaultCatalog.getDefaultDatabase();

        temporaryTables = new HashMap<>();
        builtInCatalogName = defaultCatalogName;

        this.typeFactory = typeFactory;
    }

    public static Builder newBuilder() {
        return new Builder();
    }
    ...
}
```

```java
table schema定义了flink table的数据表结构, 包括字段名称,类型,同时table schema和table format相匹配。
```

#### 两种计划器(Planner)的主要区别

```java
1) Blink将批处理作业视为流处理的一种特例。
2) Blink计划器不支持BatchTableSource,而是使用有界的StreamTableSource来替代。
3) 旧计划器和Blink计划器中FilterableTableSource的实现不是兼容的。
	旧计划器会将PlannerExpression下推至FilterableTableSource,而Blink计划器是将Expression下推
4) 基于字符串的键值配置选项仅在 Blink 计划器中使用。
5) Blink 计划器会将多sink（multiple-sinks）优化成一张有向无环图（DAG）,
	TableEnvironment 和 StreamTableEnvironment 都支持该特性。
    旧计划器总是将每个sink都优化成一个新的有向无环图，且所有图相互独立。
6) 旧计划器目前不支持 catalog 统计数据，而 Blink 支持。

```

```java
FlinkSQL 中, 元数据的管理分三层: catalog -> database -> table。

Flink SQL中是依托 calcite框架来进行SQL执行树生产, 校验, 优化等等。

我们这里主要看下flinksql是如何结合Calcite来进行元数据管理的。

```

```java
TableEnvironment维护着一个由标识符(identifier) 创建的表catalog的映射。
标识符由三个部分组成, catalog 名称, 数据库名称 以及对象名称, 如果catalog或者数据库没有指明, 就会使用当前默认值。
```

```java
Schema接口，可以通过table名来获得一张表， 可以通过schema名来获得一个子schema.

package org.apache.calcite.schema;
public interface Schema {
    Table getTable(String table);
    Set<String> getTableNames();
    RelProtoDataType getType(String type);
    Set<String> getTypeNames();
    Collection<Function> getFunctions(String name);
    Schema getSubSchema(String name);
    Set<String> getSubSchemaNames();
    ...
}


public class CatalogManagerCalciteSchema extends FlinkSchema {
    private final CatalogManager catalogManager;
    private final boolean isStreamingMode;
    public CatalogManagerCalciteSchema(CatalogManager catalogManager, boolean isStreamingMode) {
        this.catalogManager = catalogManager;
        this.isStreamingMode = isStreamingMode;
    }
    @Override
    public Table getTable(String name) {
        return null;
    }
    @Override
    public Set<String> getTableNames() {
        return Collections.emptySet();
    }
}





CatalogSchema返回DatabaseSchema, DatabaseSchema返回table, 这就是flink中元数据的三层结构了。
具体的元数据实际上都是在catalogManager中。

```

```java
public class CatalogCalciteSchema extends FlinkSchema {

    private final String catalogName;
    private final CatalogManager catalogManager;
    // Flag that tells if the current planner should work in a batch or streaming mode.
    private final boolean isStreamingMode;

    public CatalogCalciteSchema(
            String catalogName, CatalogManager catalog, boolean isStreamingMode) {
        this.catalogName = catalogName;
        this.catalogManager = catalog;
        this.isStreamingMode = isStreamingMode;
    }
    @Override
    public Schema getSubSchema(String schemaName) {
        if (catalogManager.schemaExists(catalogName, schemaName)) {
            return new DatabasecalciteSchema(schemaName, catalogNmae, catalogManager, isStreamingMode);
        }
    }
}
```

```java

class DatabaseCalciteSchema extends FlinkSchema {
    private final String catalogName;
    private final String databaseName;
    private final CatalogManager catalogManager;
    // Flag that tells if the current planner should work in a batch or streaming mode.
    private final boolean isStreamingMode;

    public DatabaseCalciteSchema(
            String catalogName,
            String databaseName,
            CatalogManager catalog,
            boolean isStreamingMode) {
        this.databaseName = databaseName;
        this.catalogName = catalogName;
        this.catalogManager = catalog;
        this.isStreamingMode = isStreamingMode;
    }

    @Override
    public Table getTable(String tableName) {
		ObjectIdentifier identifier = ObjectIdentifier.of(catalogName, databaseName, tableName);
		return catalogManager.getTable(identifier)
			.map(result -> {
				CatalogBaseTable table = result.getTable();
				FlinkStatistic statistic = getStatistic(result.isTemporary(), table, identifier);
				return new CatalogSchemaTable(identifier,
					table,
					statistic,
					catalogManager.getCatalog(catalogName)
						.flatMap(Catalog::getTableFactory)
						.orElse(null),
					isStreamingMode,
					result.isTemporary());
			})
			.orElse(null);
    }

    @Override
    public Schema getSubSchema(String name) {
        return null;
    }

}
```

#### FlinkSchema

```java
Calcite中的schema,主要是在validator过程中,获取对应的table字段信息, 对应的function的返回值信息,确保SQL的字段名,字段类型是正确的。
类的依赖关系为 validator -----> schemaReader -----> schema

```

```java
FlinkPlannerImpl.scala中

    private def createSqlValidator(catalogReader: CatalogReader) = {
        val validator = new FlinkCalciteSqlValidator(
          operatorTable,
          catalogReader,
          typeFactory)
        validator.setIdentifierExpansion(true)
        // Disable implicit type coercion for now.
        validator.setEnableTypeCoercion(false)
        validator
      }

PlanningConfigurationBuilder.java
		private CatalogReader createCatalogReader(	boolean lenientCaseSensitivity, String currentCatalog,	String currentDatabase) {
            SqlParser.Config sqlParserConfig = getSqlParserConfig();
            final boolean caseSensitive;
            if (lenientCaseSensitivity) {
                caseSensitive = false;
            } else {
                caseSensitive = sqlParserConfig.caseSensitive();
            }

            SqlParser.Config parserConfig = SqlParser.configBuilder(sqlParserConfig)
                .setCaseSensitive(caseSensitive)
                .build();

            return new CatalogReader(
                rootSchema,
                asList(
                    asList(currentCatalog, currentDatabase),
                    singletonList(currentCatalog)
                ),
                typeFactory,
                CalciteConfig.connectionConfig(parserConfig));
        }

```

#### 创建表的两种方式

```java
table 可以转换成DataStream或DataSet。
将table转换为DataStream或者DataSet时,需要指定生成的DataStream或者DataSet的数据类型,即Table的每行数据要转换成的数据类型。
       Row: 字段按位置映射, 字段数量任意, 支持null值, 无类型安全(type-safe)检查
      POJO: 字段按名称映射(POJO)必须按table中字段名称命名, 字段数量任务, 支持null值, 无类型安全检查。
Case Class: 字段按位置映射,不支持null值,有类型安全检查
	 Tuple: 字段按位置映射, 字段数量少于22(scala)或25(java)不支持null值,无类型安全检查
Automic Type: Table必须有一个字段,不支持null值,有类型安全检查。



将表转换成DataStream
 流式查询(streaming query)的结果表会动态更新,即,当新纪录到达查询的输入流时, 查询结果会改变。
因此,想这样将动态查询结果转换成DataStream需要对表的更新方式进行编码。

将Table转换成DataStream的有两种模式:
AppendMode: 仅当动态表Table仅通过insert更改进行修改时,才可以使用此模式,即仅仅是追加操作,并且之前输出的结果永远不会更新。
RetractMode: 任务情形都可以使用此模式。它使用boolean值对insert和delete操作的数据进行标记。


ps: 一旦Table被转化为DataStream,必须使用StreamExecutionEnvironment的execute方法执行该DataStream作业。
```

#### 数据类型与 Table Schema 映射

```java
Flink 的DataStream和DataSet APIs支持多样的数据类型, 例如Tuple(Scala内置以及Flink Java Tuple), POJO类型， scala的case calss
类型,以及Flink的Row类型等允许嵌套且有多个可在表的表达式中访问的字段的复合数据类型。

数据类型到 table schema的映射有两种方式: 基于字段位置或基于字段名称。

基于位置映射:
   基于位置的映射可在保持字段顺序的同时为字段提供更有意义的名称。
   这种映射方式可用于具有特定的字段顺序的复合数据类型以及原子类型。如tuple, row以及case class这些复合数据类型都有这样的字段顺序。
   然而POJO类型的字段则必须通过名称映射 (可以将字段投影出来,但不能使用as重命名)

StreamTableEnvironment tableEnv = ...; // see "Create a TableEnvironment" section;
DataStream<Tuple2<Long, Integer>> stream = ...
Table table = tableEnv.fromDataStream(stream);
Table table = tableEnv.fromDataStream(stream, $("myLong"));
Table table = tableEnv.fromDataStream(stream, $("myLong"), $("myInt"));


基于名称的映射
	基于名称的映射适用于任何数据类型包括POJO类型。这是定义table schema映射最灵活的方式。
    映射中所有字段均按名称引用,并且可以通过as重命名。 字段可以被重新排序和映射。
    如果没有指定任何字段名称,则使用默认的字段名和复合字段类型的字段顺序,或者使用f0表示原子类型。

StreamTableEnvironment tableEnv = ...;
DataStream<Tuple2<Long, Integer>> stream = ...
Table table = tableEnv.fromDataStream(stream);
Table table = tableEnv.fromDataStream(stream, $("f1"));
Table table = tableEnv.fromDataStream(stream, $("f1"), $("f0"));
Table table = tableEnv.fromDataStream(stream, $("f1").as("myInt"), $("f0").as("myLong"));

```

#### 原子类型

```java
Flink 将基础数据类型(Integer,Double,String)或者通用数据类型(不可拆分的数据类型)视为原子类型。
原子类型的DataStream或者DataSet会被转换成只有一条属性的Table。
属性的数据类型可以由原子类型推断出, 还可以重新命名属性。

StreamTableEnvironment tableEnv = ...; // see "Create a TableEnvironment" section
DataStream<Long> stream = ...
Table table = tableEnv.fromDataStream(stream);
//>>>> convert DataStream into Table with field name "myLong"
Table table = tableEnv.fromDataStream(stream, $("myLong"));

```

#### Tuple 类型

```java
StreamTableEnvironment tableEnv = ...; // see "Create a TableEnvironment" section
DataStream<Tuple2<Long, String>> stream = ...
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with renamed field names "myLong", "myString" (position-based)
Table table = tableEnv.fromDataStream(stream, $("myLong"), $("myString"));

// convert DataStream into Table with reordered fields "f1", "f0" (name-based)
Table table = tableEnv.fromDataStream(stream, $("f1"), $("f0"));

// convert DataStream into Table with projected field "f1" (name-based)
Table table = tableEnv.fromDataStream(stream, $("f1"));

// convert DataStream into Table with reordered and aliased fields "myString", "myLong" (name-based)
Table table = tableEnv.fromDataStream(stream, $("f1").as("myString"), $("f0").as("myLong"));

```

#### POJO 类型

```java
在不指定字段的情况下,将POJO类型的DataStream或DataSet转换成Table时, 将使用原始的POJO类型字段名称。
名称映射需要原始名称,并且不能按位置进行。
字段可以使用别名(带有as关键字)来重命名, 重新排序和投影

StreamTableEnvironment tableEnv = ...; // see "Create a TableEnvironment" section
DataStream<Person> stream = ...
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with renamed fields "myAge", "myName" (name-based)
Table table = tableEnv.fromDataStream(stream, $("age").as("myAge"), $("name").as("myName"));

// convert DataStream into Table with projected field "name" (name-based)
Table table = tableEnv.fromDataStream(stream, $("name"));

// convert DataStream into Table with projected and renamed field "myName" (name-based)
Table table = tableEnv.fromDataStream(stream, $("name").as("myName"));

```

#### Row 类型

```java
Row类型支持任意数量的字段以及具有null值的字段, 字段名称可以通过RowTypeInfo指定,
也可以在将Row的DataStream或DataSet转换为Table时指定。

Row类型的字段映射支持基于名称和基于位置两种方式,
	字段可以通过提供所有字段的名称的方式重命名(基于位置映射)
    或者分别选择进行投影/排序/重命名 (基于名称映射)

StreamTableEnvironment tableEnv = ...; // see "Create a TableEnvironment" section
DataStream<Row> stream = ...
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with renamed field names "myName", "myAge" (position-based)
Table table = tableEnv.fromDataStream(stream, $("myName"), $("myAge"));

// convert DataStream into Table with renamed fields "myName", "myAge" (name-based)
Table table = tableEnv.fromDataStream(stream, $("name").as("myName"), $("age").as("myAge"));

// convert DataStream into Table with projected field "name" (name-based)
Table table = tableEnv.fromDataStream(stream, $("name"));

// convert DataStream into Table with projected and renamed field "myName" (name-based)
Table table = tableEnv.fromDataStream(stream, $("name").as("myName"));

```
