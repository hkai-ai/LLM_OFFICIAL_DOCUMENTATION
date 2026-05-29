---
source: https://www.volcengine.com/docs/82379/1528783
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 批量(Chat) API · POST /api/v3/batch/chat/completions

> 批量推理调用，享受更高限流配额与更低价格（约为在线推理的 50%），适合大批量数据处理。请求 / 响应字段与[对话(Chat) API](./chat-completions.md) 基本一致。

完整 URL：`POST https://ark.cn-beijing.volces.com/api/v3/batch/chat/completions`

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

## 与 Chat API 的差异

- `model`：**需用 Endpoint ID** 调用（参考获取 Endpoint ID）。
- 走批量推理配额（更高限流、更低价格），定价见 [pricing.md](./pricing.md) 的「批量推理」一节。
- 响应 `service_tier` 仅返回 `default`（批量不使用 TPM 保障包）。

## 请求参数（与 Chat API 通用）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | Endpoint ID。 |
| `messages` | object[] | ✓ | — | 消息列表，结构同 [Chat API messages[]](./chat-completions.md#messages)。 |
| `thinking` | object | ✗ | `{"type":"enabled"}` | 深度思考开关。 |
| `max_tokens` | integer \| null | ✗ | `4096` | 回答最大长度。 |
| `max_completion_tokens` | integer \| null | ✗ | — | 范围 `[0, 64k]`，仅 `doubao-seed-1-6-250615` / `doubao-seed-1-6-flash-250615` 支持。不可与 `max_tokens` 同设。 |
| `stop` | string \| string[] \| null | ✗ | `null` | 最多 4 个停止词。深度思考能力模型不支持。 |
| `frequency_penalty` | float \| null | ✗ | `0` | 范围 `[-2.0, 2.0]`。 |
| `presence_penalty` | float \| null | ✗ | `0` | 范围 `[-2.0, 2.0]`。 |
| `temperature` | float \| null | ✗ | `1` | 范围 `[0, 2]`。 |
| `top_p` | float \| null | ✗ | `0.7` | 范围 `[0, 1]`。 |
| `logprobs` | boolean \| null | ✗ | `false` | 是否返回对数概率。 |
| `top_logprobs` | integer \| null | ✗ | `0` | 范围 `[0, 20]`，仅 `logprobs=true` 时可设。 |
| `logit_bias` | map \| null | ✗ | `null` | token 偏差，值范围 `[-100, 100]`。 |
| `tools` | object[] \| null | ✗ | `null` | 工具列表，结构同 [Chat API tools[]](./chat-completions.md#tools)。 |

## 响应（与 Chat API 通用）

顶层字段 `id` / `model` / `service_tier` / `created` / `object`（`chat.completion`）/ `choices` / `usage`，结构与含义同 [Chat API 响应](./chat-completions.md#响应)。

## 参考

- 批量(Chat) API：https://www.volcengine.com/docs/82379/1528783
- 批量推理介绍：https://www.volcengine.com/docs/82379/1399517
- 对话(Chat) API：https://www.volcengine.com/docs/82379/1494384
