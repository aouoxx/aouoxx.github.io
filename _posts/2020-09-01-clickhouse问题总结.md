### clickhouse 查询使用客户端时间

```java

默认情况下,客户端连接到服务端的时候会使用服务端时区。
  我们可以通过启动客户端命令行选项 --use_client_time_zone来设置使用客户端时间。
```

### 各个节点同名的表数据不一致导致 distribute 查询结果不一致

```sql
Distribute查询规则是在每个shard里找一个replica。所以如果replica(节点)的同名数据表内容不一致,使用distribute查询的时会出问题。



例如: 一个shard有2个备份(节点)
SELECT
    cluster,
    shard_num,
    replica_num,
    host_name,
    port
FROM system.clusters

┌─cluster─────┬─shard_num─┬─replica_num─┬─host_name─────────────┬─port─┐
│ iad_cluster │         1 │           1 │ iad-kafka1.jd.163.org │ 9000 │ 备份也可以认为是一个节点
│ iad_cluster │         1 │           2 │ iad-kafka2.jd.163.org │ 9001 │
│ iad_cluster │         2 │           1 │ iad-kafka2.jd.163.org │ 9000 │
│ iad_cluster │         2 │           2 │ iad-kafka3.jd.163.org │ 9001 │
│ iad_cluster │         3 │           1 │ iad-kafka3.jd.163.org │ 9000 │
│ iad_cluster │         3 │           2 │ iad-kafka4.jd.163.org │ 9001 │
│ iad_cluster │         4 │           1 │ iad-kafka4.jd.163.org │ 9000 │
│ iad_cluster │         4 │           2 │ iad-kafka5.jd.163.org │ 9001 │
│ iad_cluster │         5 │           1 │ iad-kafka5.jd.163.org │ 9000 │
│ iad_cluster │         5 │           2 │ iad-kafka1.jd.163.org │ 9001 │
└─────────────┴───────────┴─────────────┴───────────────────────┴──────┘

iad-kafka1.jd.163.org  9000 :) select * from ssgao_tin_log;
┌──date_time─┬─id─┬─name──┐
│ 2020-01-01 │  1 │ ssgao │
│ 2020-01-02 │  1 │ xx    │
│ 2020-01-03 │  1 │ cc    │
└────────────┴────┴───────┘
iad-kafka1.jd.163.org 9100:) select * from ssgao_tin_log;
┌──date_time─┬─id─┬─name──┐
│ 2019-01-01 │  1 │ ssgao │
│ 2019-01-02 │  1 │ xx    │
│ 2019-01-03 │  1 │ cc    │
│ 2019-01-04 │  1 │ aa    │
└────────────┴────┴───────┘

CREATE TABLE iad_big_data_test.ssgao_tin_log_all (
    `date_time` Date COMMENT '时间', `id` UInt16, `name` String
) ENGINE = Distributed('iad_cluster', 'iad_big_data_test', 'ssgao_tin_log', rand());
'iad_cluster' 集群名
'iad_big_data_test' 数据库名
'ssgao_tin_log' 表名


iad-kafka1.jd.163.org :) select * from ssgao_tin_log_all;
┌──date_time─┬─id─┬─name──┐
│ 2020-01-01 │  1 │ ssgao │
│ 2020-01-02 │  1 │ xx    │
│ 2020-01-03 │  1 │ cc    │
└────────────┴────┴───────┘
┌──date_time─┬─id─┬─name──┐  查询shard2 shard2的replica 为
│ 2019-01-01 │  1 │ ssgao │     iad-kafka2.jd.163.org 9000 上面无数据
│ 2019-01-02 │  1 │ xx    │			iad-kafka1.jd.163.org 9001 上面有数据
│ 2019-01-03 │  1 │ cc    │  distribute表查询的时候,请求有可能会达到两个副本中的一个,就会到查询结果返回不一致
│ 2019-01-04 │  1 │ aa    │
└────────────┴────┴───────┘
iad-kafka1.jd.163.org :) select * from ssgao_tin_log_all;
┌──date_time─┬─id─┬─name──┐
│ 2020-01-01 │  1 │ ssgao │
│ 2020-01-02 │  1 │ xx    │
│ 2020-01-03 │  1 │ cc    │
└────────────┴────┴───────┘


```

### clickhose 时间超越

![image.png](https://cdn.nlark.com/yuque/0/2020/png/659846/1606354787260-79f59038-762f-44b3-a4ef-8d5e97b342a8.png#height=356&id=TZsLs&margin=%5Bobject%20Object%5D&name=image.png&originHeight=712&originWidth=2228&originalType=binary&ratio=1&size=316158&status=done&style=none&width=1114)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/659846/1606354821636-265f7e2a-058b-4cba-984f-e78998c40b78.png#height=333&id=dvjWD&margin=%5Bobject%20Object%5D&name=image.png&originHeight=666&originWidth=2064&originalType=binary&ratio=1&size=293943&status=done&style=none&width=1032)

```java
iad-kafka1.jd.163.org :) select length( splitByString('.',outer_ip))  from adx_serve_table_distributed_all  where day='2021-01-15' and outer_ip is not null and outer_ip!='nil' limit 1; \G
SELECT length(splitByString('.', outer_ip))
FROM adx_serve_table_distributed_all
WHERE (day = '2021-01-15') AND isNotNull(outer_ip) AND (outer_ip != 'nil')
LIMIT 1
Received exception from server (version 20.6.3):
Code: 43. DB::Exception: Received from iad-kafka1.jd.163.org:9000. DB::Exception: Nested type Array(String) cannot be inside Nullable type.

0 rows in set. Elapsed: 0.004 sec.
```

```sql
select assumeNotNull(outer_ip) AS outer_ip,     splitByChar('.', outer_ip) from adx_serve_table_distributed_all  where day='2021-01-15' and outer_ip is not null and outer_ip!='nil' limit 1; \G

SELECT
    assumeNotNull(outer_ip) AS outer_ip,
    splitByChar('.', outer_ip)
FROM adx_serve_table_distributed_all
WHERE (day = '2021-01-15') AND isNotNull(outer_ip) AND (outer_ip != 'nil')
LIMIT 1

Row 1:
──────
outer_ip:                                  14.157.86.161
splitByChar('.', assumeNotNull(outer_ip)): ['14','157','86','161']
```

```sql
SELECT
    len,
    count(*)
FROM
(
    SELECT
        assumeNotNull(outer_ip) AS outer_ip,
        length(splitByChar('.', outer_ip)) AS len
    FROM adx_serve_table_distributed_all
    WHERE (day = '2021-01-15') AND isNotNull(outer_ip) AND (outer_ip != 'nil') AND (media_flight_request = 1) AND (flight_id IN ('90000001', '90000002'))
) AS t
GROUP BY len
```
