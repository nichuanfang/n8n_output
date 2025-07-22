你的原始代码每次循环都生成一个新的正则表达式对象，这会影响性能。可以通过以下方式优化：

- 只生成一次正则表达式（使用字符集），而不是每个字符生成一次。
- 用 replace 和回调函数批量转义所有需要的字符。
- 只在需要转义的字符前加 \，避免多余操作。

优化后的代码如下：

```javascript
function escapeMarkdown(text) {
  // 需要转义的字符
  const escapeChars = '_[]()~>#+-=|{}.!';
  // 构建正则表达式，使用字符集
  const regex = new RegExp(`[${escapeChars.replace(/[-/\\^$*+?.()|[\]{}]/g, '\\$&')}]`, 'g');
  // 替换并转义
  return text.replace(regex, '\\$&');
}
```

**说明：**
- `[${escapeChars.replace(/[-/\\^$*+?.()|[\]{}]/g, '\\$&')}]` 这段代码会自动转义 escapeChars 中的特殊正则字符，确保正则表达式安全。
- `replace(regex, '\\$&')` 会把所有匹配到的字符前加上 `\`，实现转义。

这样性能更优，代码也更简洁。