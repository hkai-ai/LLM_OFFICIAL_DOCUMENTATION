---
source: https://platform.minimaxi.com/docs/api-reference/text-chat-anthropic
fetched_at: 2026-05-26
api_version: N/A（Anthropic 兼容，路径 /anthropic/v1，不需要 anthropic-version header）
---

# MiniMax Messages · POST /anthropic/v1/messages（Anthropic 兼容）

> 根据 messages 列表生成模型回复。与 Anthropic Messages API 协议兼容，可直接用 `@anthropic-ai/sdk` 通过更换 `base_url` 调用。**注意 base 路径不是 `/v1`，而是 `/anthropic/v1`；鉴权 header 是 `X-Api-Key` 不是 `Authorization: Bearer`**。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `X-Api-Key` | ✓ | MiniMax 平台 API key |
| `Content-Type` | ✓ | `application/json` |

> 与官方 Anthropic 不同，**不需要** `anthropic-version` header。

完整 URL：`POST https://api.minimaxi.com/anthropic/v1/messages`

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID。见 [models.md](./models.md)。 |
| `messages` | array | ✓ | — | 对话消息数组，结构见下。 |
| `system` | string \| array | ✗ | — | 系统提示词，可纯字符串或 content block 数组。 |
| `stream` | boolean | ✗ | `false` | 是否启用 SSE 流式返回。 |
| `max_tokens` | integer | ✗ | — | 生成上限，范围 `[1, 204800]`。 |
| `temperature` | number | ✗ | `1.0` | 范围 `(0, 1]`。 |
| `top_p` | number | ✗ | `0.95` | 范围 `(0, 1]`。 |

### `messages[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | `user` / `assistant` / `user_system` / `group` / `sample_message_user` / `sample_message_ai`（后四种为 MiniMax 特有，含义同 [chat-completions.md](./chat-completions.md) §messages）。 |
| `content` | string \| array | ✓ | 文本或 content block 数组。 |

## 响应

### 顶层对象（非流式）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID。 |
| `type` | string | 固定 `message`。 |
| `role` | string | 固定 `assistant`。 |
| `model` | string | 实际使用的模型 ID。 |
| `content` | array | 响应内容块数组，见 `content[]`。 |
| `stop_reason` | string | `end_turn` / `max_tokens` / `stop_sequence`。 |
| `usage.input_tokens` | integer | 输入 token 数。 |
| `usage.output_tokens` | integer | 输出 token 数。 |
| `base_resp` | object | MiniMax 通用响应封装（`status_code` / `status_msg`），见 [errors.md](./errors.md)。 |

### `content[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | `text` / `thinking`。 |
| `text` | string | `type=text` 时的文本内容。 |
| `thinking` | string | `type=thinking` 时模型的思考过程。 |
| `signature` | string | 思考块签名，用于完整性校验。 |

## 流式响应（SSE）

`stream: true` 时按 Anthropic 命名事件下发：

| 事件 | 说明 | 关键字段 |
| --- | --- | --- |
| `message_start` | 消息开始 | `message`（含元数据） |
| `ping` | 心跳 | — |
| `content_block_start` | 内容块开始 | `index`、`content_block` |
| `content_block_delta` | 内容块增量 | `index`、`delta`（含 `type` / `text` 等） |
| `content_block_stop` | 内容块结束 | `index` |
| `message_delta` | 消息级增量 | `delta.stop_reason`、`usage` |
| `message_stop` | 消息结束 | — |

## 显式 Prompt Caching（cache_control）

Anthropic 兼容路径支持显式 **`cache_control: { type: "ephemeral" }`** 内联在 content block 上，TTL 为 **5 分钟**，命中后自动续期。计费字段：

- `cache_creation_input_tokens` — 缓存创建消耗的 input token 数（计费高于普通 input）。
- `cache_read_input_tokens` — 缓存命中读取的 input token 数（计费低）。

详见官方 [Anthropic 主动缓存](https://platform.minimaxi.com/docs/api-reference/anthropic-api-compatible-cache) 与 [pricing.md](./pricing.md) §计费要点。

最小命中阈值：input ≥ 512 tokens。

## 与 Anthropic 标准的差异

| 项 | MiniMax | 官方 Anthropic |
| --- | --- | --- |
| Base path | `/anthropic/v1/messages` | `/v1/messages` |
| 鉴权 header | `X-Api-Key` | `x-api-key` + `anthropic-version` |
| `anthropic-version` | 不需要 | 必需 |
| `role` 扩展 | 支持 `user_system` / `group` / `sample_message_*` | 仅 `user` / `assistant` |
| `content` 思考块 | `type: "thinking"` 块 + `signature` | 同 |
| `base_resp` 字段 | 顶层返回 | 无（Anthropic 错误结构不同） |

## 示例

### 最小请求

```json
{
  "model": "MiniMax-M2.7",
  "messages": [
    {"role": "user", "content": "你好"}
  ]
}
```

### 最小响应

```json
{
  "id": "06379dbe27b33d7c58d8410a8efe6394",
  "type": "message",
  "role": "assistant",
  "model": "MiniMax-M2.7",
  "content": [
    {"type": "text", "text": "你好！有什么我可以帮助你的吗？"}
  ],
  "stop_reason": "end_turn",
  "usage": {"input_tokens": 42, "output_tokens": 30},
  "base_resp": {"status_code": 0, "status_msg": ""}
}
```

## 参考

- 端点文档：https://platform.minimaxi.com/docs/api-reference/text-chat-anthropic
- SDK 接入：https://platform.minimaxi.com/docs/api-reference/text-anthropic-api
- 主动缓存：https://platform.minimaxi.com/docs/api-reference/anthropic-api-compatible-cache
- 错误码：[errors.md](./errors.md)
