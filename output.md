针对你这个查询需求，性能优化可以从以下几个方面考虑：

1. **索引设计**  
   - 对 `domain` 字段建立普通的B树索引，可以加速 `domain = 'sub.example.com'` 的精确匹配。  
   - 但是 `domain LIKE '.%'` 和 `'sub.example.com' LIKE '%' || domain` 这种后缀匹配，普通索引无法加速。

2. **利用反向索引（pg_trgm扩展）**  
   - PostgreSQL 的 pg_trgm 扩展支持对 LIKE '%xxx' 这种模糊匹配建立索引。  
   - 你可以对 `domain` 字段创建一个 `gin` 或 `gist` 类型的 trigram 索引，提升后缀匹配的性能。

3. **拆分查询逻辑**  
   - 先用索引快速定位全域名匹配的行。  
   - 再用 trigram 索引加速泛域名的后缀匹配。

4. **示例索引和查询**：

```sql
-- 开启pg_trgm扩展（如果还没启用）
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- 建立普通索引，加速精确匹配
CREATE INDEX idx_domain_exact ON domain_cookies(domain);

-- 建立trigram索引，加速LIKE匹配
CREATE INDEX idx_domain_trgm ON domain_cookies USING gin (domain gin_trgm_ops);
```

查询语句保持不变：

```sql
SELECT domain, cookies
FROM domain_cookies
WHERE domain = 'sub.example.com'
   OR (domain LIKE '.%' AND 'sub.example.com' LIKE '%' || domain);
```

5. **注意事项**  
- pg_trgm 索引对 LIKE '%xxx' 有效，但对 LIKE 'xxx%' 更有效率。这里是后缀匹配，pg_trgm 仍然适用。  
- 如果数据量非常大，且查询频繁，考虑把泛域名拆成单独字段或设计更适合的结构。  
- 也可以考虑把 domain 字段反转存储，方便用前缀匹配代替后缀匹配。

总结：  
- 建立普通索引加速精确匹配。  
- 建立 pg_trgm 索引加速后缀模糊匹配。  
- 保持查询逻辑不变。  

需要我帮你写具体的索引创建和测试脚本吗？