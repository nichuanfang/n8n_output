好的，下面给你一个详细的 n8n 工作流设计方案，包含全局参数设置、缓存查询、调用 CookieCloud 接口、解密、筛选 cookie、更新缓存的完整流程说明和示例代码。你可以根据此方案在 n8n 中搭建。

---

# 一、准备工作

1. **全局参数**（在 n8n 的“设置”->“变量”中配置）：

| 参数名               | 示例值                         | 说明                     |
|----------------------|-------------------------------|--------------------------|
| `uuid`               | your-uuid-string               | CookieCloud 的 uuid       |
| `password`           | your-password-string           | CookieCloud 的密码        |
| `cookiecloud_base_url`| http://your-cookiecloud-host:8088 | CookieCloud 服务器地址    |
| `cache_ttl_minutes`  | 10                            | 缓存有效期，单位分钟      |

2. **PostgreSQL 缓存表**

执行以下 SQL 创建缓存表：

```sql
CREATE TABLE cookie_cache (
  cache_key VARCHAR(255) PRIMARY KEY,
  cookie_data TEXT NOT NULL,
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

---

# 二、n8n 工作流节点设计

---

### 1. **输入参数节点**（可选）

- 用于接收外部传入的 URL 参数，字段名假设为 `inputUrl`。

---

### 2. **生成缓存键（Function 节点）**

```javascript
const crypto = require('crypto');

const uuid = $global.uuid;
const password = $global.password;
const inputUrl = $json.inputUrl;

function generateCacheKey(uuid, password, url) {
  const keyString = uuid + password + url;
  return crypto.createHash('md5').update(keyString).digest('hex');
}

const cacheKey = generateCacheKey(uuid, password, inputUrl);

return { json: { cacheKey, inputUrl } };
```

---

### 3. **查询缓存（PostgreSQL 节点）**

- SQL 查询：

```sql
SELECT cookie_data, updated_at FROM cookie_cache WHERE cache_key = '{{ $json.cacheKey }}';
```

- 连接参数配置你的 PostgreSQL 数据库。

---

### 4. **判断缓存是否有效（Function 节点）**

```javascript
const cacheData = $json.cookie_data;
const updatedAt = $json.updated_at ? new Date($json.updated_at) : null;
const now = new Date();

const CACHE_TTL = ($global.cache_ttl_minutes || 10) * 60 * 1000; // 转毫秒

if (cacheData && updatedAt && (now - updatedAt) < CACHE_TTL) {
  // 缓存有效，直接返回缓存数据
  return [{ json: { fromCache: true, cookieHeader: cacheData } }];
} else {
  // 缓存无效，继续后续流程
  return [{ json: { fromCache: false, cacheKey: $json.cacheKey, inputUrl: $json.inputUrl } }];
}
```

---

### 5. **条件判断（IF 节点）**

- 判断 `fromCache` 是否为 true。
- 如果是 true，直接输出缓存的 `cookieHeader`，流程结束。
- 如果是 false，进入下一步调用 CookieCloud 接口。

---

### 6. **调用 CookieCloud 获取加密 cookie（HTTP Request 节点）**

- Method: GET
- URL: `{{$global.cookiecloud_base_url}}/get/{{$json.cacheKey.split('-')[0]}}`  
  （注意：这里的 uuid 需要从缓存键中拆出来，或者你也可以直接用全局变量 `$global.uuid`）

- 这里建议直接用 `$global.uuid`，URL 写成：

```
{{$global.cookiecloud_base_url}}/get/{{$global.uuid}}
```

- 返回结果示例：

```json
{
  "encrypted": "加密字符串"
}
```

---

### 7. **解密 Cookie（Function 节点）**

```javascript
const crypto = require('crypto');

const uuid = $global.uuid;
const password = $global.password;
const encryptedData = $json.encrypted;

function md5(data) {
  return crypto.createHash('md5').update(data).digest('hex');
}

const key = md5(uuid + password);

function decrypt(encrypted, key) {
  const iv = Buffer.alloc(16, 0);
  const decipher = crypto.createDecipheriv('aes-128-cbc', Buffer.from(key, 'hex'), iv);
  decipher.setAutoPadding(true);
  let decrypted = decipher.update(encrypted, 'base64', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}

try {
  const decryptedJsonStr = decrypt(encryptedData, key);
  const decryptedJson = JSON.parse(decryptedJsonStr);
  return { json: { decryptedJson } };
} catch (error) {
  throw new Error('解密失败: ' + error.message);
}
```

---

### 8. **筛选对应 URL 的 Cookie（Function 节点）**

```javascript
const url = $json.inputUrl;
const cookies = $json.decryptedJson.cookies;

function getDomainFromUrl(url) {
  try {
    return new URL(url).hostname;
  } catch {
    return null;
  }
}

const domain = getDomainFromUrl(url);

if (!domain) {
  throw new Error('无效的 URL');
}

// 过滤出 domain 匹配的 cookie
const matchedCookies = cookies.filter(cookie => {
  return domain.endsWith(cookie.domain.replace(/^\./, ''));
});

// 转换成 Cookie 请求头字符串
const cookieHeader = matchedCookies.map(c => `${c.name}=${c.value}`).join('; ');

return { json: { cookieHeader, cacheKey: $json.cacheKey } };
```

---

### 9. **更新缓存（PostgreSQL 节点）**

- SQL 语句（使用 UPSERT）：

```sql
INSERT INTO cookie_cache (cache_key, cookie_data, updated_at)
VALUES ('{{ $json.cacheKey }}', '{{ $json.cookieHeader }}', NOW())
ON CONFLICT (cache_key) DO UPDATE SET
cookie_data = EXCLUDED.cookie_data,
updated_at = EXCLUDED.updated_at;
```

---

### 10. **输出最终结果**

- 输出字段：`cookieHeader`，即对应 URL 的 Cookie 请求头字符串。

---

# 三、流程总结图（逻辑顺序）

```
输入 URL
   ↓
生成缓存键
   ↓
查询缓存
   ↓
缓存有效? ── 是 ──> 返回缓存 cookieHeader，结束
   ↓ 否
调用 CookieCloud 接口获取加密 cookie
   ↓
解密 cookie 数据
   ↓
筛选对应 URL 的 cookie
   ↓
更新缓存
   ↓
返回 cookieHeader
```

---

# 四、补充说明

- 你可以根据需要调整缓存有效期。
- 如果你有多个 uuid 或密码，可以扩展缓存键生成逻辑。
- 以上代码示例均基于 Node.js 环境，n8n Function 节点支持。
- PostgreSQL 节点需要正确配置数据库连接。

---

如果你需要，我可以帮你导出一个完整的 n8n 工作流 JSON 文件，包含上述所有节点和连接，你看需要吗？