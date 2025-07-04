你使用Flowable结合PostgreSQL，想根据流程变量和变量名称查询变量值，确实通常的逻辑是：

1. 先从运行时变量表 `act_ru_variable` 查询，因为这是当前正在运行的流程实例的变量，数据是最新的且查询速度快。
2. 如果在运行时变量表中没有查到，再去历史变量表 `act_hi_varinst` 查询，因为历史表中保存了已经完成或已结束流程实例的变量。

关于如何保证SQL查询的高性能，建议如下：

### 1. 查询逻辑示例
```sql
-- 先查运行时变量
SELECT text_ AS variable_value
FROM act_ru_variable
WHERE execution_id_ = :executionId
  AND name_ = :variableName
UNION ALL
-- 如果运行时变量没查到，再查历史变量
SELECT text_ AS variable_value
FROM act_hi_varinst
WHERE execution_id_ = :executionId
  AND name_ = :variableName
ORDER BY CASE WHEN variable_value IS NOT NULL THEN 0 ELSE 1 END
LIMIT 1;
```
这里用UNION ALL合并两个表的查询，优先返回运行时变量，如果没有再返回历史变量。

### 2. 索引优化
- 确保 `act_ru_variable` 和 `act_hi_varinst` 表上有索引，特别是 `(execution_id_, name_)` 组合索引，这样能快速定位变量。
- PostgreSQL可以使用如下索引：
```sql
CREATE INDEX idx_ru_variable_execution_name ON act_ru_variable (execution_id_, name_);
CREATE INDEX idx_hi_varinst_execution_name ON act_hi_varinst (execution_id_, name_);
```

### 3. 其他建议
- 只查询必要的字段，避免SELECT *。
- 如果变量值存储在不同字段（如text_, long_, double_等），根据变量类型选择对应字段查询。
- 如果业务允许，可以考虑缓存热点变量，减少数据库查询压力。

总结：先查 `act_ru_variable`，查不到再查 `act_hi_varinst`，并且确保两个表的 `(execution_id_, name_)` 有合适的索引，SQL写法上用UNION ALL并LIMIT 1保证只返回一个结果，能兼顾准确性和性能。需要我帮你写具体的SQL示例吗？