---
source: https://openrouter.ai/docs/guides/features/message-transforms
fetched_at: 2026-05-19
api_version: N/A
---

# Message Transforms

> `transforms` 在请求发往上游模型前对 prompt 进行修改。当前文档明示的转换是 **middle-out**（上下文压缩）。

## `transforms` 参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `transforms` | array&lt;string&gt; | ✗ | — | 转换名数组，按顺序应用。 |

### 允许值

| 值 | 含义 |
| --- | --- |
| `middle-out` | 当 prompt 超过模型上下文窗口时，从消息序列中间移除/截断，保留首尾各一半。研究表明 LLM 对序列中段关注度较低。 |

## 默认行为：context-compression 插件

所有上下文长度 ≤ 8192 token 的 endpoint **默认启用** context-compression 插件，等价于自动应用 middle-out。可显式启用或关闭：

```json
{
  "plugins": [{"id": "context-compression"}]
}
```

```json
{
  "plugins": [{"id": "context-compression", "enabled": false}]
}
```

## 压缩选择机制

启用 context-compression 时，OpenRouter 优先选择上下文长度 ≥ 所需 token 总数一半的模型，然后从消息中间删除/截断内容直到满足窗口约束。对于有消息数量上限的模型（如部分 Claude），保留对话开始一半 + 结尾一半的消息。

## 与消息长度限制的交互

- 若 `context-compression` 关闭且 token 超限：请求直接失败（HTTP 400），错误提示建议减少长度或启用压缩。
- 若启用：先压缩再送达上游，不报错。

## 示例

```json
{
  "model": "openai/gpt-4o-mini",
  "transforms": ["middle-out"],
  "messages": [
    {"role": "system", "content": "..."},
    {"role": "user", "content": "..."}
  ]
}
```

显式关闭默认压缩：

```json
{
  "model": "openai/gpt-4o-mini",
  "plugins": [{"id": "context-compression", "enabled": false}],
  "messages": [...]
}
```

## 参考

- Message Transforms 指南：<https://openrouter.ai/docs/guides/features/message-transforms>
