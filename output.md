根据流程实例Id和流程变量名称更新历史流程变量的值（字符串类型），一般是在流程引擎（如Activiti、Flowable、Camunda等）中操作历史数据。不同的流程引擎有不同的API和数据库表结构，下面以常见的Activiti/Flowable为例，说明思路和方法：

1. 理解历史流程变量存储
- 历史流程变量通常存储在历史表中，如ACT_HI_VARINST（Activiti）或ACT_HI_VARINST（Flowable）。
- 该表中包含流程实例ID（PROC_INST_ID_）、变量名称（NAME_）、变量值（TEXT_、TEXT2_、DOUBLE_、LONG_等字段）等。

2. 查询历史变量记录
- 根据流程实例ID和变量名称查询对应的历史变量记录。

3. 更新变量值
- 将对应记录的变量值字段更新为新的字符串值。

4. 注意事项
- 直接修改历史数据可能影响流程审计和追踪，需谨慎操作。
- 有些引擎不建议直接修改历史数据，推荐通过流程变量接口修改运行时变量。
- 如果确实需要修改历史数据，建议备份数据。

示例SQL（假设表名为ACT_HI_VARINST，变量值存储在TEXT_字段）：

```sql
UPDATE ACT_HI_VARINST
SET TEXT_ = '新的字符串值'
WHERE PROC_INST_ID_ = '流程实例Id'
  AND NAME_ = '变量名称';
```

如果是通过Java API操作，示例思路：

```java
// 伪代码，具体API根据引擎版本调整
HistoryService historyService = processEngine.getHistoryService();
List<HistoricVariableInstance> vars = historyService.createHistoricVariableInstanceQuery()
    .processInstanceId(processInstanceId)
    .variableName(variableName)
    .list();

for (HistoricVariableInstance var : vars) {
    // 这里通常HistoricVariableInstance是只读的，不能直接修改
    // 需要通过底层数据库操作或自定义扩展实现更新
}
```

总结：
- 直接更新历史流程变量值通常需要操作数据库。
- 先确认变量存储字段和表结构。
- 执行SQL更新语句。
- 注意数据一致性和流程审计影响。

如果你使用的是具体的流程引擎，请告诉我，我可以帮你提供更具体的操作方法。