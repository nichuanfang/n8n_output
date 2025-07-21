你想要的SQL查询结果只返回两项：

1. 拼接的cookie字符串（用分号`;`分割所有匹配行的cookie_string）
2. 最大的超时时间（expiration_date）

基于你的表结构和查询条件，修正后的SQL示例如下（假设使用PostgreSQL）：

```sql
SELECT
    STRING_AGG(cookie_string, ';') AS cookies,
    MAX(expiration_date) AS max_expiration_date
FROM domain_cookies
WHERE
    domain = '{{ $json.domain }}'
    OR domain = '.' || '{{ $json.domain }}'
    OR (
        domain LIKE '.%' AND
        '{{ $json.domain }}' LIKE '%.' || SUBSTRING(domain FROM 2)
    );
```

说明：
- `STRING_AGG(cookie_string, ';')` 用于将所有符合条件的cookie_string字段用分号拼接起来。
- `MAX(expiration_date)` 用于取最大超时时间。

如果你的数据库不是PostgreSQL，拼接函数可能不同，请告诉我具体数据库，我帮你调整。