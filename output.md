在 PostgreSQL 中，联合索引（多列索引）是作为一个整体创建的，不能直接修改索引来删除其中的某一个字段。如果你想要“删除”联合索引中的某个字段，实际上需要做的是：

1. 删除原有的联合索引。
2. 重新创建一个不包含该字段的新联合索引。

具体步骤如下：

假设你有一个联合索引名为 idx_example，索引字段是 (column1, column2, column3)，你想删除其中的 column2。

1. 删除原有索引：

```sql
DROP INDEX idx_example;
```

2. 创建新的联合索引，不包含 column2：

```sql
CREATE INDEX idx_example_new ON your_table (column1, column3);
```

注意事项：

- 删除索引会影响数据库查询性能，建议在业务低峰期操作。
- 创建新索引时，可以根据需要选择合适的索引类型（默认是 B-tree）。
- 如果索引名被其他对象引用，删除时需要谨慎。

总结：PostgreSQL 不支持直接修改索引字段，必须删除旧索引并重新创建新索引。