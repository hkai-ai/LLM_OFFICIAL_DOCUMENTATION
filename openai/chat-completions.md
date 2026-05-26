---
source: https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create
fetched_at: 2026-05-19
api_version: N/A
---

# Chat Completions · POST /v1/chat/completions

> 根据消息列表生成模型回复。仍是 OpenAI 当下最广为兼容的接口，但新特性（内置工具、推理摘要、后台模式等）优先上 Responses API。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <OPENAI_API_KEY>` |
| `Content-Type` | ✓ | `application/json` |
| `OpenAI-Organization` | ✗ | 多 organization 账户显式指定计费 organization。 |
| `OpenAI-Project` | ✗ | 显式指定 project ID。 |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID。允许值见 [models.md](./models.md)。 |
| `messages` | array | ✓ | — | 对话消息数组，详见 [`messages[]`](#messages)。 |
| `modalities` | array&lt;string&gt; | ✗ | `["text"]` | 期望输出模态，可选 `text` / `audio`（需 audio-preview 模型）。 |
| `audio` | object | ✗ | — | 当 `modalities` 含 `audio` 时配置语音输出，见 [`audio`](#audio)。 |
| `frequency_penalty` | number | ✗ | `0` | 范围 `[-2.0, 2.0]`。正值降低重复 token 的出现概率。 |
| `presence_penalty` | number | ✗ | `0` | 范围 `[-2.0, 2.0]`。正值鼓励引入新话题。 |
| `logit_bias` | object | ✗ | — | token id → 偏置（`[-100, 100]`）。`-100` 屏蔽，`100` 强制选择。 |
| `logprobs` | boolean | ✗ | `false` | 返回每个输出 token 的 log probability。 |
| `top_logprobs` | integer | ✗ | — | 返回每个位置 top-N 候选（0–20），需 `logprobs: true`。 |
| `max_tokens` | integer | ✗ | — | **已弃用**：生成的最大 token 数，且对 reasoning 模型不生效。新代码请用 `max_completion_tokens`。 |
| `max_completion_tokens` | integer | ✗ | — | 模型生成可见 token + 推理 token 的上限。 |
| `n` | integer | ✗ | `1` | 候选数量。每个候选独立计费。 |
| `parallel_tool_calls` | boolean | ✗ | `true` | 是否允许一次响应中并行调用多个工具。 |
| `prediction` | object | ✗ | — | 推测解码（speculative decoding）配置，见 [`prediction`](#prediction)。 |
| `reasoning_effort` | string | ✗ | — | 仅 reasoning 模型：`minimal` / `low` / `medium` / `high`。 |
| `verbosity` | string | ✗ | — | 仅 GPT-5 系列：`low` / `medium` / `high`，控制输出详尽程度。 |
| `response_format` | object | ✗ | `{ "type": "text" }` | 输出格式，见 [`response_format`](#response_format)。 |
| `seed` | integer | ✗ | — | 实验性，尽力做可复现解码。配合 `system_fingerprint` 检验。 |
| `service_tier` | string | ✗ | `auto` | `auto` / `default` / `flex` / `priority`。priority/flex 需账户启用。 |
| `stop` | string \| array&lt;string&gt; | ✗ | — | 最多 4 个停止序列。 |
| `store` | boolean | ✗ | `false` | 是否把对话存储用于后续 distillation / evals。 |
| `stream` | boolean | ✗ | `false` | 是否启用 SSE 流式。 |
| `stream_options` | object | ✗ | — | `stream: true` 下的额外选项，见 [`stream_options`](#stream_options)。 |
| `temperature` | number | ✗ | `1` | 范围 `[0, 2]`。建议只调一项（temperature / top_p）。 |
| `top_p` | number | ✗ | `1` | nucleus sampling。 |
| `tools` | array | ✗ | — | 函数工具定义数组，见 [`tools[]`](#tools)。 |
| `tool_choice` | string \| object | ✗ | `auto` | `none` / `auto` / `required` 或 `{ "type": "function", "function": { "name": "..." } }`。 |
| `user` | string | ✗ | — | **遗留字段**，等同 `safety_identifier` 的早期形态。新代码用 `safety_identifier`。 |
| `metadata` | object | ✗ | — | 最多 16 对 K/V，键 ≤ 64 字符，值 ≤ 512 字符。 |
| `web_search_options` | object | ✗ | — | 启用 web 搜索（限指定模型，如 `gpt-4o-search-preview`），见 [`web_search_options`](#web_search_options)。 |
| `prompt_cache_key` | string | ✗ | — | 提示缓存路由 key，相同 key 命中同一缓存 shard。 |
| `safety_identifier` | string | ✗ | — | 终端用户的稳定哈希标识，用于风控。 |

### `messages[]`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `role` | string | ✓ | — | `system` / `developer` / `user` / `assistant` / `tool`。`developer` 用于 GPT-5/o-系列替代 `system`。 |
| `content` | string \| array | 通常 ✓ | — | 字符串或内容块数组。`assistant` 仅含 `tool_calls` 时可省略。 |
| `name` | string | ✗ | — | 发送者标识，便于区分多个 user/assistant。 |
| `tool_calls` | array | ✗ | — | 仅 `role: "assistant"`：模型发起的工具调用。 |
| `tool_call_id` | string | ✗（`role: "tool"` 时 ✓） | — | 对应 assistant 工具调用的 id。 |
| `refusal` | string | ✗ | — | `role: "assistant"` 的拒答文本（与 `content` 互斥）。 |

#### `messages[].content[]`（用户 / 系统 / 开发者侧）

| `type` | 字段 | 说明 |
| --- | --- | --- |
| `text` | `text` | 文本片段。 |
| `image_url` | `image_url.url`、`image_url.detail` | URL 或 `data:` base64；`detail` 取 `auto` / `low` / `high`。 |
| `input_audio` | `input_audio.data`、`input_audio.format` | base64 音频；format 取 `wav` / `mp3`。 |
| `file` | `file.file_id` 或 `file.file_data` + `file.filename` | 引用已上传的 `/v1/files` 对象或内联 base64。 |

#### `messages[].content[]`（assistant 侧）

| `type` | 字段 | 说明 |
| --- | --- | --- |
| `text` | `text` | 文本输出。 |
| `refusal` | `refusal` | 模型拒答字符串。 |

#### `tool_calls[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 工具调用 id，回传时通过 `tool_call_id` 引用。 |
| `type` | string | 固定 `function`。 |
| `function.name` | string | 调用的函数名。 |
| `function.arguments` | string | JSON 字符串形式的参数。 |

### `audio`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `voice` | string | ✓ | `alloy` / `ash` / `ballad` / `coral` / `echo` / `sage` / `shimmer` / `verse` 等（随模型变化）。 |
| `format` | string | ✓ | `wav` / `mp3` / `flac` / `opus` / `pcm16`。 |

### `tools[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `function`（Chat Completions 不支持 Responses 的其它内置工具）。 |
| `function.name` | string | ✓ | 1–64 字符，匹配 `^[a-zA-Z0-9_-]+$`。 |
| `function.description` | string | ✗ | 给模型看的描述。 |
| `function.parameters` | object | ✗ | JSON Schema。 |
| `function.strict` | boolean | ✗ | 启用严格模式：保证 arguments 满足 schema。 |

### `response_format`

| `type` | 附加字段 | 说明 |
| --- | --- | --- |
| `text` | — | 默认。 |
| `json_object` | — | 强制 JSON。需要在 prompt 中明确要求 JSON，否则可能空字符串。 |
| `json_schema` | `json_schema.name`、`json_schema.schema`、`json_schema.strict`、`json_schema.description` | 结构化输出。`strict: true` 时按 schema 解码。 |

### `stream_options`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `include_usage` | boolean | 末事件包含 `usage`。 |
| `include_obfuscation` | boolean | 控制 token 混淆相关字段。 |

### `prediction`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | 固定 `content`。 |
| `content` | string \| array | 期望的延续内容，用于推测解码加速。 |

### `web_search_options`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `search_context_size` | string | `low` / `medium` / `high`。 |
| `user_location` | object | `type: approximate` + `approximate.{country, city, region, timezone}`。 |

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `chatcmpl-...` |
| `object` | string | 固定 `chat.completion`。 |
| `created` | integer | Unix 时间戳（秒）。 |
| `model` | string | 实际使用模型 ID（可能带日期版本）。 |
| `choices` | array | 候选数组。 |
| `usage` | object | token 用量统计。 |
| `system_fingerprint` | string | 与 `seed` 配合检验可复现性。 |
| `service_tier` | string | 实际命中的层级。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选索引。 |
| `message.role` | string | 固定 `assistant`。 |
| `message.content` | string \| null | 文本回复，含工具调用时可能为 null。 |
| `message.refusal` | string \| null | 拒答文本（与 content 互斥）。 |
| `message.tool_calls` | array | 模型发起的工具调用，结构同请求侧。 |
| `message.audio` | object | 音频输出：`{id, data, expires_at, transcript}`。 |
| `finish_reason` | string | `stop` / `length` / `tool_calls` / `content_filter` / `function_call`（legacy）。 |
| `logprobs` | object \| null | 见 [`logprobs`](#logprobs)。 |

### `logprobs`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `content[].token` | string | token 文本。 |
| `content[].logprob` | number | log 概率。 |
| `content[].bytes` | array&lt;integer&gt; \| null | UTF-8 字节。 |
| `content[].top_logprobs` | array | `{ token, logprob, bytes }` 数组。 |
| `refusal[]` | array | 当 `message.refusal` 非空时给出对应 logprobs。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 数。 |
| `completion_tokens` | integer | 输出 token 数。 |
| `total_tokens` | integer | 总 token 数。 |
| `prompt_tokens_details.cached_tokens` | integer | 命中提示缓存的 token 数。 |
| `prompt_tokens_details.audio_tokens` | integer | 输入音频 token。 |
| `completion_tokens_details.reasoning_tokens` | integer | 推理模型消耗的隐藏推理 token。 |
| `completion_tokens_details.audio_tokens` | integer | 输出音频 token。 |
| `completion_tokens_details.accepted_prediction_tokens` | integer | 推测解码命中的 token。 |
| `completion_tokens_details.rejected_prediction_tokens` | integer | 推测解码失败的 token。 |

## 流式响应

`stream: true` 时返回 `text/event-stream`，事件为多次 `data: { ... }`，最后一行 `data: [DONE]`。每个 chunk 形如：

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion.chunk",
  "created": 1730000000,
  "model": "gpt-5",
  "choices": [
    {
      "index": 0,
      "delta": { "role": "assistant", "content": "Hel" },
      "logprobs": null,
      "finish_reason": null
    }
  ]
}
```

- 工具调用通过 `delta.tool_calls` 增量下发（`id` / `function.name` 仅首块出现，`function.arguments` 持续追加）。
- `stream_options.include_usage: true` 时末尾追加一个含 `usage` 字段的 chunk。

## 示例

### 最小请求

```json
{
  "model": "gpt-5",
  "messages": [
    { "role": "user", "content": "Hello" }
  ]
}
```

### 最小响应

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1730000000,
  "model": "gpt-5-2026-01-15",
  "choices": [
    {
      "index": 0,
      "message": { "role": "assistant", "content": "Hi there!", "refusal": null },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 8,
    "completion_tokens": 3,
    "total_tokens": 11,
    "prompt_tokens_details": { "cached_tokens": 0 },
    "completion_tokens_details": { "reasoning_tokens": 0 }
  }
}
```

### 工具调用响应片段

```json
{
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
          {
            "id": "call_abc",
            "type": "function",
            "function": { "name": "get_weather", "arguments": "{\"city\":\"Beijing\"}" }
          }
        ]
      },
      "finish_reason": "tool_calls"
    }
  ]
}
```

## 错误码

| HTTP | error.type | 触发原因 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 参数缺失、格式错误、JSON 不合法。 |
| `401` | `invalid_api_key` / `authentication_error` | API key 无效或权限不足。 |
| `403` | `permission_error` | 区域不支持或被风控。 |
| `404` | `not_found_error` | 模型 ID 不存在或对当前账号不可见。 |
| `429` | `rate_limit_error` / `insufficient_quota` | 触发 RPM/TPM 或余额耗尽。 |
| `500` | `server_error` | 服务端异常，可重试。 |
| `503` | `engine_overloaded` | 模型过载。 |

## 参考

- 端点文档：<https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create>
- 上级目录：<https://developers.openai.com/api/reference/resources/chat>
- 推理参数：<https://developers.openai.com/api/docs/guides/reasoning>
