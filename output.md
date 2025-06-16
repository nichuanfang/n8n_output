根据流程实例ID获取当前任务的候选组ID，可以通过查询 Flowable 的 ACT_RU_IDENTITYLINK 表，筛选出对应流程实例ID且类型为候选人的记录，并且候选组ID不为空。

SQL 查询示例（请将 '流程实例Id' 替换为实际的流程实例ID）：

```sql
SELECT TASK_ID_, GROUP_ID_
FROM ACT_RU_IDENTITYLINK
WHERE PROC_INST_ID_ = '流程实例Id'
  AND TYPE_ = 'candidate'
  AND GROUP_ID_ IS NOT NULL;
```

这条语句会返回该流程实例下所有任务的候选组ID。如果你只想获取当前活动任务的候选组，可以先通过 ACT_RU_TASK 表获取当前活动任务，再结合上面的查询。

例如，先查当前活动任务：

```sql
SELECT ID_ AS TASK_ID
FROM ACT_RU_TASK
WHERE PROC_INST_ID_ = '流程实例Id';
```

然后用得到的 TASK_ID 去查询候选组：

```sql
SELECT GROUP_ID_
FROM ACT_RU_IDENTITYLINK
WHERE TASK_ID_ = '任务ID'
  AND TYPE_ = 'candidate'
  AND GROUP_ID_ IS NOT NULL;
```

如果你需要，我可以帮你写完整的查询语句。