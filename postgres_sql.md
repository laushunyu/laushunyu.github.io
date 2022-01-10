# postgres祖传性能SQL

## 连接占用

```
select count(*) as count,state , wait_event, query 
from pg_stat_activity group by query,wait_event, state order by count desc;
```


## 锁

```
select w1.pid     as w_pid,
       w2.query   as w_query,
       b1.pid     as b_pid,
       b2.query   as b_query
from pg_locks w1
         join pg_stat_activity w2 on w1.pid = w2.pid
         join pg_locks b1 on w1.transactionid = b1.transactionid and w1.pid != b1.pid
         join pg_stat_activity b2 on b1.pid = b2.pid
where not w1.granted;
```
## 查看各个表大小占用
```
SELECT table_schema, table_name, pg_total_relation_size('"' || table_schema || '"."' || table_name || '"')AS size
FROM information_schema.tables
ORDER BY size DESC
```

## 完全释放表空间（或许需要定时任务或者更频繁的autovacuum）
*导致的原因很多，比如频繁上报*
```
VACUUM FULL asset_startup;
```

## 加载 pg_stat_statements 模块
postgres.conf
```
shared_preload_libraries = 'pg_stat_statements'
```

活跃连接数 < sum(cpu)
```
select count(*) from pg_stat_activity where state='active';
```

超时请求
```
select count(*) from pg_stat_activity where state='active' and now()-query_start > interval '3 second';
```

等待🔓的语句查询
```
select count(*) from pg_stat_activity where wait_event_type is not null ; 
```











