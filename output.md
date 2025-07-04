你的SQL逻辑整体上是合理的，目标是先从运行时表（act_ru_variable）中查找变量值，如果找不到再从历史表（act_hi_varinst）中查找。查询结构和思路也符合需求。

不过，有几点细节可以优化或确认：

1. ROW_NUMBER() OVER (PARTITION BY proc_inst_id_ ORDER BY NULL)  
   - PostgreSQL中ORDER BY NULL是不被支持的，应该改成ORDER BY某个确定的列，或者如果不关心顺序，可以用ORDER BY proc_inst_id_，但这样每个分组内的行号可能没有意义。  
   - 你这里的目的是取每个proc_inst_id_的第一条记录，建议明确指定排序列，比如运行时表中可以用某个时间戳列（如果有）或者主键列。

2. WHERE NOT EXISTS 子句的逻辑  
   - 你用来排除已经在运行时表中存在的proc_inst_id_，这部分逻辑是对的。

3. 参数绑定  
   - :processInstanceIds 和 :variableName 是参数占位符，确保调用时传入正确格式的数组和字符串。

4. 性能建议  
   - 如果processInstanceIds数量很大，ANY(:processInstanceIds)可能性能不佳，可以考虑临时表或JOIN。

5. 语法检查  
   - 你可以尝试执行SQL，确认是否有语法错误。

基于以上，我帮你调整了ORDER BY部分，假设运行时表和历史表都有时间列time_，用它排序：

```sql
WITH run_var AS (
    SELECT proc_inst_id_, text_ AS variable_value,
           ROW_NUMBER() OVER (PARTITION BY proc_inst_id_ ORDER BY time_ DESC) AS rn
    FROM act_ru_variable
    WHERE proc_inst_id_ = ANY(:processInstanceIds)
      AND name_ = :variableName
      AND text_ IS NOT NULL
),
hist_var AS (
    SELECT proc_inst_id_, text_ AS variable_value,
           ROW_NUMBER() OVER (PARTITION BY proc_inst_id_ ORDER BY time_ DESC) AS rn
    FROM act_hi_varinst
    WHERE proc_inst_id_ = ANY(:processInstanceIds)
      AND name_ = :variableName
      AND text_ IS NOT NULL
)
SELECT proc_inst_id_, variable_value
FROM (
    SELECT proc_inst_id_, variable_value, rn, 1 AS priority FROM run_var WHERE rn = 1
    UNION ALL
    SELECT proc_inst_id_, variable_value, rn, 2 AS priority FROM hist_var WHERE rn = 1
) t
WHERE NOT EXISTS (
    SELECT 1 FROM run_var r WHERE r.proc_inst_id_ = t.proc_inst_id_
)
OR priority = 1
ORDER BY proc_inst_id_, priority
;
```

如果运行时表没有time_列，可以用其他合适列替代ORDER BY。

总结：你的SQL思路没问题，主要是ORDER BY NULL在PostgreSQL不支持，建议改成明确的排序列。其他逻辑看起来正确。你可以根据实际表结构调整排序列。需要我帮你根据表结构进一步优化吗？