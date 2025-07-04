你当前的SQL是针对单个流程实例ID和变量名查询变量值的。现在需求是根据多个流程实例ID列表，批量获取每个流程实例对应的变量值。

思路是将单个流程实例ID的查询改写为针对多个流程实例ID的查询，返回每个流程实例ID对应的变量值。需要注意：

- 变量名仍然是单个变量名，还是多个变量名？假设变量名是单个变量名。
- 对每个流程实例ID，优先返回运行时变量，否则返回历史变量。
- 返回结果中包含流程实例ID和对应的变量值。

下面是基于你原SQL的改写示例，假设传入的流程实例ID列表是 :processInstanceIds（数组或逗号分隔字符串），变量名是 :variableName：

```sql
WITH run_var AS (
    SELECT proc_inst_id_, text_ AS variable_value,
           ROW_NUMBER() OVER (PARTITION BY proc_inst_id_ ORDER BY NULL) AS rn
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

说明：

- run_var和hist_var分别查询运行时和历史变量，使用ROW_NUMBER()按流程实例分组排序，取每个流程实例的第一条变量值。
- 最终结果中，优先返回运行时变量（priority=1），如果运行时变量不存在，则返回历史变量（priority=2）。
- 通过WHERE NOT EXISTS保证只有当运行时变量不存在时才返回历史变量。
- 返回字段包含proc_inst_id_和对应的variable_value。

如果你的数据库或驱动不支持数组参数，可以将 :processInstanceIds 替换成临时表或用IN列表。

如果你需要支持多个变量名，逻辑会更复杂，可以告诉我，我帮你调整。