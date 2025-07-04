了解了，act_ru_variable表没有time_列，导致你原先用ROW_NUMBER()时ORDER BY NULL在PostgreSQL不支持。

这里有几种处理方案：

1. 如果你只想取每个proc_inst_id_对应的任意一条记录（不关心具体哪条），可以用ORDER BY proc_inst_id_或者ORDER BY (SELECT NULL)的变通写法，但后者PostgreSQL不支持。

2. 你可以用ORDER BY proc_inst_id_或者ORDER BY (SELECT 1)，但这其实没什么排序意义，只是为了让ROW_NUMBER()能用。

3. 另一种思路是用DISTINCT ON，这在PostgreSQL中可以方便地取每组第一条记录，且不需要ROW_NUMBER()。

基于第3点，我帮你改写成用DISTINCT ON的写法，逻辑是：

- 先从运行时表取每个proc_inst_id_的第一条（任意一条）变量值
- 再从历史表取每个proc_inst_id_的第一条变量值
- 最终合并，优先返回运行时表的结果，没有再返回历史表的

改写SQL如下：

```sql
WITH run_var AS (
    SELECT DISTINCT ON (proc_inst_id_) proc_inst_id_, text_ AS variable_value
    FROM act_ru_variable
    WHERE proc_inst_id_ = ANY(:processInstanceIds)
      AND name_ = :variableName
      AND text_ IS NOT NULL
),
hist_var AS (
    SELECT DISTINCT ON (proc_inst_id_) proc_inst_id_, text_ AS variable_value
    FROM act_hi_varinst
    WHERE proc_inst_id_ = ANY(:processInstanceIds)
      AND name_ = :variableName
      AND text_ IS NOT NULL
    ORDER BY proc_inst_id_, time_ DESC
)
SELECT proc_inst_id_, variable_value
FROM (
    SELECT proc_inst_id_, variable_value, 1 AS priority FROM run_var
    UNION ALL
    SELECT proc_inst_id_, variable_value, 2 AS priority FROM hist_var
) t
WHERE NOT EXISTS (
    SELECT 1 FROM run_var r WHERE r.proc_inst_id_ = t.proc_inst_id_
)
OR priority = 1
ORDER BY proc_inst_id_, priority
;
```

说明：

- run_var用DISTINCT ON (proc_inst_id_)取每个流程实例的第一条变量值，顺序无所谓，所以没有ORDER BY。
- hist_var用DISTINCT ON (proc_inst_id_)并ORDER BY proc_inst_id_, time_ DESC，取时间最新的历史变量值。
- 最后合并结果，优先返回运行时表的变量值。

这样写更简洁且符合PostgreSQL语法。

你可以试试这个版本，看是否满足需求。需要我帮你进一步调整吗？