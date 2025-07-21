你的需求是：  
给定一个域名参数（如 `foo.a.com`），需要在 domain_cookies 表中查找所有“适用”的 cookie。  
- 如果 domain 字段为 `a.com`，只匹配 `a.com` 本身。  
- 如果 domain 字段为 `.a.com`，则匹配 `a.com` 及其所有子域（如 `b.a.com`, `foo.a.com` 等）。

## 查询思路

假设你的输入参数为 `{{ $json.domain }}`，比如 `foo.a.com`。

你需要查找：
1. domain = `foo.a.com`（全域名精确匹配）
2. domain = `.a.com`，且 `foo.a.com` 以 `a.com` 结尾（泛域名匹配）

### SQL 查询

```sql
SELECT *
FROM domain_cookies
WHERE
    -- 精确匹配
    domain = '{{ $json.domain }}'
    -- 泛域名匹配
    OR (
        domain LIKE '.%' -- 以点开头的泛域名
        AND (
            -- 去掉 domain 字段的开头的点
            right('{{ $json.domain }}', length(domain) - 1) = right(domain, length(domain) - 1)
            OR
            '{{ $json.domain }}' LIKE '%' || right(domain, length(domain) - 1)
        )
        AND (
            -- 只匹配以 domain 去掉点后的结尾
            '{{ $json.domain }}' ~ ('(^|\\.)' || right(domain, length(domain) - 1) || '$')
        )
    )
;
```

### 更简洁实用的写法

通常只需要两种情况：

```sql
SELECT *
FROM domain_cookies
WHERE
    domain = '{{ $json.domain }}'
    OR (
        domain LIKE '.%' -- 泛域名
        AND (
            '{{ $json.domain }}' = right(domain, length(domain) - 1)
            OR
            '{{ $json.domain }}' LIKE '%.' || right(domain, length(domain) - 1)
        )
    )
;
```

#### 说明
- `right(domain, length(domain) - 1)` 取得去掉开头点的域名部分。
- `LIKE '%.' || ...` 保证只有以 `.` 分隔的子域才会匹配。
- 这样 `foo.a.com` 会匹配 `.a.com`，`a.com` 也会匹配 `.a.com`。

---

## 示例

假设表中有如下数据：

| id | domain   | ... |
|----|----------|-----|
| 1  | a.com    | ... |
| 2  | .a.com   | ... |
| 3  | b.com    | ... |
| 4  | .b.com   | ... |

输入 `foo.a.com`，会匹配 id=2（.a.com）；  
输入 `a.com`，会匹配 id=1（a.com）和 id=2（.a.com）。

---

## 总结

你可以直接用如下 SQL：

```sql
SELECT *
FROM domain_cookies
WHERE
    domain = '{{ $json.domain }}'
    OR (
        domain LIKE '.%'
        AND (
            '{{ $json.domain }}' = right(domain, length(domain) - 1)
            OR
            '{{ $json.domain }}' LIKE '%.' || right(domain, length(domain) - 1)
        )
    )
;
```

如需进一步优化或有特殊边界情况，欢迎补充说明！