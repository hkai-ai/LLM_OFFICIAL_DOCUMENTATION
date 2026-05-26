---
source: https://platform.minimaxi.com/docs/api-reference/text-chat-openai
fetched_at: 2026-05-26
api_version: N/A（OpenAI 兼容，路径 /v1）
---

# MiniMax Chat Completions · POST /v1/chat/completions（OpenAI 兼容）

> 根据 messages 列表生成模型回复。与 OpenAI Chat Completions 协议兼容，可直接用 OpenAI SDK 通过更换 `base_url` 调用；同时返回了 MiniMax 特有的 `base_resp` / `input_sensitive*` / `output_sensitive*` 字段。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json` |

完整 URL：`POST https://api.minimaxi.com/v1/chat/completions`

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID。允许值见 [models.md](./models.md)。 |
| `messages` | array | ✓ | — | 对话消息数组，详见 `messages[]`。 |
| `stream` | boolean | ✗ | `false` | 是否启用 SSE 流式返回。 |
| `max_completion_tokens` | integer | ✗ | — | 生成内容上限 token，范围 `[1, 2048]`。 |
| `temperature` | number | ✗ | `1.0` | 采样温度，范围 `(0, 1]`。 |
| `top_p` | number | ✗ | `0.95` | nucleus 采样阈值，范围 `(0, 1]`。 |

### `messages[]`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `role` | string | ✓ | — | 取值：`system` / `user` / `assistant` / `user_system` / `group` / `sample_message_user` / `sample_message_ai`。后四种为 MiniMax 特有，详见下文。 |
| `content` | string | ✓ | — | 消息内容（纯文本）。 |
| `name` | string | ✗ | — | 同类型消息多条时区分发送者。 |

**MiniMax 特有 role 含义**：

- `user_system` — 在对话场景中代表"用户人设"，用于角色扮演时设定 user 一侧的身份背景。
- `group` — 多人 / 群组对话场景中的群组标识。
- `sample_message_user` / `sample_message_ai` — 示例对话对，用于 Few-Shot 学习。

## 响应

### 顶层对象（非流式）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID。 |
| `object` | string | 固定 `chat.completion`。 |
| `created` | integer | Unix 时间戳。 |
| `model` | string | 实际使用的模型 ID。 |
| `choices` | array | 候选结果数组，结构见下。 |
| `usage` | object | token 使用统计。 |
| `input_sensitive` | boolean | 输入是否命中敏感词。 |
| `input_sensitive_type` | integer | 1 严重违规 / 2 色情 / 3 广告 / 4 违禁 / 5 谩骂 / 6 暴恐 / 7 其他。 |
| `output_sensitive` | boolean | 输出是否命中敏感词。 |
| `output_sensitive_type` | integer | 取值同 `input_sensitive_type`。 |
| `base_resp` | object | MiniMax 通用响应封装。`status_code` 为 0 表示成功，其他值见 [errors.md](./errors.md)。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选索引。 |
| `message.role` | string | 通常为 `assistant`。 |
| `message.content` | string | 生成内容。 |
| `finish_reason` | string | `stop` / `length`。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 数。 |
| `completion_tokens` | integer | 输出 token 数。 |
| `total_tokens` | integer | 合计。 |

## 流式响应（SSE）

`stream: true` 时按 OpenAI 风格分块下发：

```json
{
  "id": "...",
  "object": "chat.completion.chunk",
  "created": 1776838946,
  "model": "MiniMax-M2.7",
  "choices": [
    {
      "index": 0,
      "delta": {"role": "assistant", "content": "..."},
      "finish_reason": null
    }
  ],
  "usage": null
}
```

最后一个 chunk 的 `finish_reason` 为 `stop` 或 `length`，并携带完整 `usage`。

## 上下文缓存

支持 **被动自动缓存**（默认开启，无需调整请求）。命中后在 `usage` 中以 `cached_tokens`（OpenAI SDK 风格字段）体现。如需显式 `cache_control` 控制写入，请改用 Anthropic 兼容路径 [messages.md](./messages.md)。

最小触发阈值：input ≥ 512 tokens。详见官方 [Prompt Caching](https://platform.minimaxi.com/docs/api-reference/text-prompt-caching)。

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
  "id": "06379b8377842dc0108975f159dc3e7c",
  "object": "chat.completion",
  "created": 1776838788,
  "model": "MiniMax-M2.7",
  "choices": [
    {
      "index": 0,
      "message": {"role": "assistant", "content": "你好！有什么我可以帮助你的吗？"},
      "finish_reason": "stop"
    }
  ],
  "usage": {"prompt_tokens": 22, "completion_tokens": 21, "total_tokens": 43},
  "input_sensitive": false,
  "input_sensitive_type": 0,
  "output_sensitive": false,
  "output_sensitive_type": 0,
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 参考

- 端点文档：https://platform.minimaxi.com/docs/api-reference/text-chat-openai
- SDK 接入：https://platform.minimaxi.com/docs/api-reference/text-openai-api
- Prompt 缓存：https://platform.minimaxi.com/docs/api-reference/text-prompt-caching
- 错误码：[errors.md](./errors.md)
