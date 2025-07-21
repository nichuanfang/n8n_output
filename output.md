你说的表结构是 domain_cookies(domain, cookies)，其中 domain 字段存储的是域名信息，可能是泛域名（以点开头，如 `.example.com`）或者是全域名（如 `example.com`）。

你想根据某一个具体域名（比如 `sub.example.com`）获取所有匹配的 cookies，匹配规则是：

- 如果表中 domain 是全域名，且等于该域名，则匹配。
- 如果表中 domain 是泛域名（以点开头），且该域名是泛域名的子域名，则匹配。

举例：查询 `sub.example.com`，要匹配 `sub.example.com` 和 `.example.com`。

对应的 SQL 可以写成：

```sql
SELECT cookies
FROM domain_cookies
WHERE domain = 'sub.example.com'
   OR (domain LIKE '.%' AND 'sub.example.com' LIKE '%' || domain);
```

解释：

- `domain = 'sub.example.com'`：匹配全域名完全相等的情况。
- `domain LIKE '.%'`：筛选出泛域名（以点开头）。
- `'sub.example.com' LIKE '%' || domain`：判断传入的域名是否以泛域名结尾，比如 `sub.example.com` LIKE `%.example.com`。

如果你想查询其他域名，只需要替换 `'sub.example.com'` 即可。

如果你需要返回所有匹配的 cookies，可以用上面的查询。如果需要返回 domain 和 cookies，可以改成：

```sql
SELECT domain, cookies
FROM domain_cookies
WHERE domain = 'sub.example.com'
   OR (domain LIKE '.%' AND 'sub.example.com' LIKE '%' || domain);
```

这样就能获取该域名对应的所有 cookies 了。需要我帮你写成函数或者更复杂的查询吗？