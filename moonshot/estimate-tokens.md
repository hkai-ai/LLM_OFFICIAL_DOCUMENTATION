---
source: https://platform.kimi.com/docs/api/estimate
fetched_at: 2026-05-20
api_version: N/A
---

# 估算 Token 数 · POST /v1/tokenizers/estimate-token-count

按指定模型的分词器估算一段消息序列的 token 数；与 OpenAI 不同，Moonshot 提供官方在线端点而非本地 tokenizer 库。

## 请求 Body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 模型 ID，决定使用哪套分词器。 |
| `messages` | `array<Message>` | ✓ | 与 `chat/completions` 同结构，支持多模态 `content` 数组与 `partial` 字段。 |

> 文档未明确该端点的速率限制；惯例上与 chat 请求共享 RPM 配额，建议节流。

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `data.total_tokens` | integer | 整个 `messages` 序列经该模型分词器计算得到的总 token 数。 |

## 示例

请求：

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {"role": "system", "content": "你是 Kimi。"},
    {"role": "user", "content": "你好"}
  ]
}
```

响应：

```json
{
  "data": {"total_tokens": 12}
}
```

## 参考

- 端点：https://platform.kimi.com/docs/api/estimate
