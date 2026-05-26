---
source: https://openrouter.ai/docs/api/api-reference/generations/get-generation
fetched_at: 2026-05-19
api_version: N/A
---

# 查询请求元数据 · GET /api/v1/generation

> 用 generation ID 异步查询一次请求的 token 数、费用、provider、延迟等详细信息，用于审计与计费对账。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |

## 请求参数（query string）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `id` | string | ✓ | — | generation ID。即 chat completions 响应中的 `id` 字段。 |

## 响应

成功时返回 `{"data": {...}}`，`data` 对象包含约 40+ 字段。

### `data`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | generation 唯一 ID。 |
| `request_id` | string | 同一次 API 请求下产生的所有输出共享。 |
| `session_id` | string | 跨多次 generation 的会话分组。 |
| `model` | string | 实际使用的模型 ID。 |
| `provider_name` | string | 实际服务该请求的 provider 名称。 |
| `created_at` | string | ISO 8601 时间戳。 |
| `generation_time` | integer | 生成耗时（毫秒）。 |
| `latency` | integer | 总响应延迟（毫秒）。 |
| `moderation_latency` | integer | moderation 处理耗时。 |
| `streamed` | boolean | 该次响应是否走了 SSE 流。 |
| `api_type` | string | 端点类型，如 `completions` / `chat_completions` / `embeddings` / `responses`。 |
| `origin` | string | 请求来源（HTTP-Referer 等）。 |
| `app_id` | integer | app 实体 ID。 |
| `tokens_prompt` | integer | OpenRouter 归一化后的提示 token 数。 |
| `tokens_completion` | integer | OpenRouter 归一化后的完成 token 数。 |
| `native_tokens_prompt` | integer | provider 上报的原始提示 token 数。 |
| `native_tokens_completion` | integer | provider 上报的原始完成 token 数。 |
| `native_tokens_cached` | integer | provider 上报的缓存命中 token 数。 |
| `native_tokens_reasoning` | integer | provider 上报的推理 token 数（若有）。 |
| `num_media_prompt` | integer | 提示中媒体项数量。 |
| `num_media_completion` | integer | 完成中媒体项数量。 |
| `num_input_audio_prompt` | integer | 提示中音频输入数量。 |
| `total_cost` | number | 该次请求总费用（USD）。 |
| `usage` | number | 使用金额（USD），通常等同 total_cost。 |
| `cache_discount` | number | 缓存折扣金额。 |
| `upstream_inference_cost` | number | 上游 provider 实际收取的推理成本。 |
| `finish_reason` | string | 完成原因。 |
| `cancelled` | boolean | 是否被取消。 |
| `is_byok` | boolean | 是否走 BYOK key。 |

> 注：上述字段以官方 OpenAPI schema 为准；OpenRouter 表示响应共 41 个字段，本表列出常用字段。完整字段建议查阅 <https://openrouter.ai/openapi.json>。

## 示例

### 请求

```
GET /api/v1/generation?id=gen-1726100000-abc123 HTTP/1.1
Host: openrouter.ai
Authorization: Bearer sk-or-...
```

### 响应

```json
{
  "data": {
    "id": "gen-1726100000-abc123",
    "model": "anthropic/claude-3.5-sonnet",
    "provider_name": "Anthropic",
    "streamed": false,
    "created_at": "2026-05-19T08:00:00Z",
    "generation_time": 1245,
    "tokens_prompt": 128,
    "tokens_completion": 512,
    "native_tokens_prompt": 128,
    "native_tokens_completion": 512,
    "native_tokens_cached": 0,
    "total_cost": 0.0089,
    "cache_discount": 0,
    "origin": "https://example.com",
    "finish_reason": "stop"
  }
}
```

## 错误码

| HTTP | 含义 |
| --- | --- |
| `401` | 未授权。 |
| `404` | 未找到该 generation ID。 |
| `500` | 服务器错误。 |

## 参考

- 端点文档：<https://openrouter.ai/docs/api/api-reference/generations/get-generation>
- API Reference 总入口：<https://openrouter.ai/docs/api/reference/overview>
