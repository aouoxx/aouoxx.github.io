_hive 中排序介绍: order by , sort by ,distribute by ,cluster by_
\_

### _order by _

```sql
 order by 会对输入做全局排序,因此只有一个Reducer(多个Reducer无法保证全局有序),然而只有reducer,会导致输入规模较大时,消耗较长时间。
```

[_https://blog.csdn.net/lzm1340458776/article/details/43306115_](https://blog.csdn.net/lzm1340458776/article/details/43306115)

### 时间转换

```scala
from_unixtime 函数用法
	from_unixtime(int/bigint timestamp) 返回timestamp时间戳对应的日期,格式为yyyy-MM-dd HH:mm:ss

  from_unixtime(int/bigint timestamp,string format) 返回timestamp时间戳对应的日期,格式由format指定
  select from_unixtime(1000000000,'yyyy/MM/dd');

```

### 动态分区

```sql






```
