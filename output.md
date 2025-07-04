如果只需要查询字符串类型的变量值，Flowable中字符串类型的变量一般存储在字段 `text_` 中（而非 `long_` 或 `double_` 等字段）。

基于此，针对你要根据流程实例ID（execution_id_）和变量名称（name_）查询字符串类型变量值的需求，推荐的高性能SQL示例如下：

```sql
SELECT text_ AS variable_value
FROM act_ru_variable
WHERE execution_id_ = :executionId
  AND name_ = :variableName
  AND text_ IS NOT NULL
UNION ALL
SELECT text_ AS variable_value
FROM act_hi_varinst
WHERE execution_id_ = :executionId
  AND name_ = :variableName
  AND text_ IS NOT NULL
ORDER BY CASE WHEN variable_value IS NOT NULL THEN 0 ELSE 1 END
LIMIT 1;
```

说明：
- 只查询 `text_` 字段不为空的记录，确保是字符串类型变量。
- 先查运行时变量表 `act_ru_variable`，查不到再查历史变量表 `act_hi_varinst`。
- 使用 `(execution_id_, name_)` 组合索引，确保查询效率。
- `LIMIT 1` 保证只返回一个结果。

索引建议：

```sql
CREATE INDEX idx_ru_variable_execution_name_text ON act_ru_variable (execution_id_, name_) WHERE text_ IS NOT NULL;
CREATE INDEX idx_hi_varinst_execution_name_text ON act_hi_varinst (execution_id_, name_) WHERE text_ IS NOT NULL;
```

这样索引只覆盖字符串类型变量，进一步提升查询性能。

如果你需要，我可以帮你写具体的参数绑定示例或Java代码调用示例。