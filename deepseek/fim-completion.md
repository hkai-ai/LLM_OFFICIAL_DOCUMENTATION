---
source: https://api-docs.deepseek.com/zh-cn/api/create-completion
fetched_at: 2026-05-19
api_version: N/A（Beta）
---

# FIM 补全（Beta） · POST /beta/completions

> Fill-in-the-Middle 文本补全：给定前缀 `prompt` 与可选后缀 `suffix`，由模型生成中间部分。用于代码补全等场景。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DEEPSEEK_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |

Base URL 必须为 `https://api.deepseek.com/beta`。该端点不支持思考模式。

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID，例如 `deepseek-v4-pro` / `deepseek-v4-flash` / `deepseek-chat`。FIM 仅在非思考模式下可用。 |
| `prompt` | string | ✓ | `"Once upon a time,"` | 补全的前缀文本。 |
| `suffix` | string | ✗ | — | 补全的后缀文本。模型在 `prompt` 与 `suffix` 之间填充。 |
| `max_tokens` | integer | ✗ | — | 最大生成 token 数。本端点 completion 上限为 4000 token。 |
| `temperature` | number | ✗ | `1` | 取值 `[0, 2]`。 |
| `top_p` | number | ✗ | `1` | 取值 `(0, 1]`。 |
| `stop` | string \| array<string> | ✗ | — | 停止序列，最多 16 个。 |
| `stream` | boolean | ✗ | `false` | 是否流式返回。 |
| `stream_options` | object | ✗ | — | 见 `include_usage`。 |
| `logprobs` | integer | ✗ | — | 取值 `[0, 20]`；返回每个 token 的最可能候选的对数概率。注意此处为 integer，与 chat 端点 boolean 不同。 |
| `echo` | boolean | ✗ | `false` | 是否在响应 `text` 中回显 `prompt`。 |
| `frequency_penalty` | number | ✗ | — | 官方标注已弃用，保留兼容但不生效。 |
| `presence_penalty` | number | ✗ | — | 官方标注已弃用，保留兼容但不生效。 |

### `stream_options`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `include_usage` | boolean | ✗ | `true` 时在流末尾发送包含 `usage` 的 chunk。 |

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID。 |
| `object` | string | 固定 `text_completion`。 |
| `created` | integer | Unix 时间戳（秒）。 |
| `model` | string | 实际使用的模型 ID。 |
| `choices` | array | 候选补全列表。 |
| `usage` | object | token 使用统计。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选序号。 |
| `text` | string | 补全文本（`echo=true` 时包含 `prompt`）。 |
| `finish_reason` | string | `stop` / `length` / `content_filter` / `insufficient_system_resource`。 |
| `logprobs` | object \| null | 对数概率信息。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 数。 |
| `completion_tokens` | integer | 输出 token 数。 |
| `total_tokens` | integer | 总 token 数。 |

> FIM 不进入思考模式，因此 `reasoning_tokens` 通常为 0 或不返回。

## 示例

### 最小请求

```json
{
  "model": "deepseek-chat",
  "prompt": "def fib(n):\n    if n < 2:\n        return n\n    return ",
  "suffix": "\n\nprint(fib(10))",
  "max_tokens": 64
}
```

### 最小响应

```json
{
  "id": "cmpl-xxxxxxxx",
  "object": "text_completion",
  "created": 1747641600,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "text": "fib(n - 1) + fib(n - 2)",
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 24,
    "completion_tokens": 11,
    "total_tokens": 35
  }
}
```

## 错误码

与对话补全一致，详见 [errors.md](./errors.md)。

## 参考

- 端点文档：https://api-docs.deepseek.com/zh-cn/api/create-completion
- 指南：https://api-docs.deepseek.com/zh-cn/guides/fim_completion
