---
source: https://openrouter.ai/docs/api/reference/overview
fetched_at: 2026-05-19
api_version: N/A
---

# 文本补全（Legacy）· POST /api/v1/completions

> OpenRouter 对外宣称兼容 OpenAI 的 `/completions` 与 `/chat/completions` 两套规范。`/api/v1/completions` 即 OpenAI Legacy 文本补全风格，使用 `prompt` 字段。

## 状态

官方 API Reference 侧边栏对 `/api/v1/completions` 没有独立的页面，但在多处文档（FAQ、quickstart、OpenAPI spec）中将其与 `chat/completions` 并列声明为「OpenAI specification 兼容」。完整字段以 OpenAI Legacy Completions 规范为准；OpenRouter 在 chat 端点支持的扩展字段（`provider` / `models` / `route` / `transforms` / `reasoning` / `usage` / `plugins` / `cache_control` 等）在该端点同样适用。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json` |
| `HTTP-Referer` / `X-Title` | ✗ | 同 chat completions。 |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓* | — | 模型 ID。与 `models` 二选一。 |
| `models` | array&lt;string&gt; | ✗ | — | 候选模型回退数组。 |
| `prompt` | string \| array&lt;string&gt; | ✓ | — | 提示文本，或 token 数组（与 OpenAI 一致）。 |
| `max_tokens` | integer | ✗ | — | 生成上限。 |
| `temperature` | number | ✗ | `1.0` | 范围 `[0, 2]`。 |
| `top_p` | number | ✗ | `1.0` | 核采样阈值。 |
| `top_k` | integer | ✗ | `0` | OpenRouter 扩展。 |
| `min_p` | number | ✗ | `0.0` | OpenRouter 扩展。 |
| `top_a` | number | ✗ | `0.0` | OpenRouter 扩展。 |
| `frequency_penalty` | number | ✗ | `0.0` | 范围 `[-2, 2]`。 |
| `presence_penalty` | number | ✗ | `0.0` | 范围 `[-2, 2]`。 |
| `repetition_penalty` | number | ✗ | `1.0` | OpenRouter 扩展。 |
| `seed` | integer | ✗ | — | 确定性种子。 |
| `logit_bias` | object | ✗ | — | token 偏置映射。 |
| `logprobs` | integer \| boolean | ✗ | — | OpenAI Legacy 中为 integer（返回前 N 个候选）；OpenAI/OR 在 chat 中已切换为 boolean。 |
| `stop` | string \| array&lt;string&gt; | ✗ | — | 停止序列。 |
| `stream` | boolean | ✗ | `false` | SSE 流式。 |
| `stream_options` | object | ✗ | — | 如 `{"include_usage": true}`。 |
| `n` | integer | ✗ | `1` | 候选数量。 |
| `echo` | boolean | ✗ | `false` | 是否回显 prompt。 |
| `suffix` | string | ✗ | — | 拼接到生成内容后的文本（部分模型支持）。 |
| `user` | string | ✗ | — | 终端用户标识。 |
| `provider` | object | ✗ | — | provider 路由，见 [provider-routing.md](./provider-routing.md)。 |
| `route` | string | ✗ | — | `fallback` 启用模型回退。 |
| `transforms` | array&lt;string&gt; | ✗ | — | 同 chat。 |
| `usage` | object | ✗ | — | `{"include": true}` 时返回 `cost` 等。 |
| `plugins` | array | ✗ | — | 同 chat。 |

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID。 |
| `object` | string | `text_completion`。 |
| `created` | integer | Unix 秒。 |
| `model` | string | 实际使用的模型。 |
| `provider` | string | 实际使用的 provider。 |
| `choices` | array&lt;object&gt; | 候选数组。 |
| `usage` | object | 同 chat。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选序号。 |
| `text` | string | 生成文本。 |
| `finish_reason` | string | `stop` / `length` / `content_filter` / `error`。 |
| `logprobs` | object | 仅在请求开启时返回。 |

## 示例

### 最小请求

```json
{
  "model": "meta-llama/llama-3.1-8b-instruct",
  "prompt": "Once upon a time",
  "max_tokens": 64
}
```

### 最小响应

```json
{
  "id": "gen-...",
  "object": "text_completion",
  "created": 1715000000,
  "model": "meta-llama/llama-3.1-8b-instruct",
  "provider": "Together",
  "choices": [
    {"index": 0, "text": " in a small village...", "finish_reason": "stop"}
  ],
  "usage": {"prompt_tokens": 5, "completion_tokens": 12, "total_tokens": 17}
}
```

## 备注

- 端点的细粒度字段官方没有独立 Reference 页面，需以 OpenAI Legacy Completions 规范为基础，叠加 OpenRouter chat completions 的扩展字段。
- 多数现代模型（如 Anthropic / OpenAI o 系列）仅支持 chat 协议，使用 `/completions` 时可能被拒或自动转换。

## 参考

- API Reference 总入口：<https://openrouter.ai/docs/api/reference/overview>
- 参数索引：<https://openrouter.ai/docs/api/reference/parameters>
