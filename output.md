在PostgreSQL中，可以通过查询系统视图pg_locks和pg_stat_activity来查看当前被锁定的SQL语句和锁信息。以下是一个常用的查询示例，用于查看哪些SQL语句持有锁或正在等待锁：

```sql
SELECT
    pg_stat_activity.pid,
    pg_stat_activity.query,
    pg_locks.locktype,
    pg_locks.mode,
    pg_locks.granted,
    pg_stat_activity.state,
    pg_stat_activity.query_start
FROM
    pg_stat_activity
JOIN
    pg_locks ON pg_stat_activity.pid = pg_locks.pid
WHERE
    pg_stat_activity.pid <> pg_backend_pid()  -- 排除当前会话
    AND pg_stat_activity.query <> '<IDLE>'
ORDER BY
    pg_stat_activity.query_start;
```

这个查询会显示持有锁或等待锁的进程ID（pid）、对应的SQL语句、锁类型、锁模式、是否已授予锁、会话状态以及查询开始时间。

如果您需要进一步帮助，或者想要查询特定表的锁信息，也可以告诉我。