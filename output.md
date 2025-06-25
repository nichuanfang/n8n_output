在 PostgreSQL 中，提取字符串中的某个数据可以使用多种函数，具体取决于你想提取的内容和格式。常用的方法有：

1. 使用 substring 函数：
```sql
-- 提取字符串中符合正则表达式的部分
SELECT substring(your_column FROM '正则表达式') FROM your_table;
```
例如，提取字符串中第一个数字：
```sql
SELECT substring('abc123def' FROM '\d+') AS extracted_number;
-- 结果: 123
```

2. 使用 split_part 函数：
```sql
-- 按分隔符拆分字符串，提取指定部分
SELECT split_part(your_column, '分隔符', 部分编号) FROM your_table;
```
例如，提取以逗号分隔的第二部分：
```sql
SELECT split_part('apple,banana,cherry', ',', 2) AS second_part;
-- 结果: banana
```

3. 使用正则表达式函数 regexp_matches 或 regexp_replace：
```sql
-- 返回所有匹配的子串（数组）
SELECT regexp_matches(your_column, '正则表达式') FROM your_table;

-- 替换字符串中匹配的部分
SELECT regexp_replace(your_column, '正则表达式', '替换内容') FROM your_table;
```

如果你能提供具体的字符串样例和想提取的数据格式，我可以帮你写更精准的 SQL 语句。